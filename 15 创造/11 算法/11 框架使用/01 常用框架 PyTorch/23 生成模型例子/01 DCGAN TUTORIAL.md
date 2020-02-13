---
title: 01 DCGAN TUTORIAL
toc: true
date: 2019-06-29
---
# DCGAN TUTORIAL

**Author**: [Nathan Inkawhich](https://github.com/inkawhich)

## Introduction

This tutorial will give an introduction to DCGANs through an example. We will train a generative adversarial network (GAN) to generate new celebrities after showing it pictures of many real celebrities. Most of the code here is from the dcgan implementation in [pytorch/examples](https://github.com/pytorch/examples), and this document will give a thorough explanation of the implementation and shed light on how and why this model works. But don’t worry, no prior knowledge of GANs is required, but it may require a first-timer to spend some time reasoning about what is actually happening under the hood. Also, for the sake of time it will help to have a GPU, or two. Lets start from the beginning.

## Generative Adversarial Networks

### What is a GAN?

GANs are a framework for teaching a DL model to capture the training data’s distribution so we can generate new data from that same distribution. GANs were invented by Ian Goodfellow in 2014 and first described in the paper [Generative Adversarial Nets](https://papers.nips.cc/paper/5423-generative-adversarial-nets.pdf). They are made of two distinct models, a *generator* and a*discriminator*. The job of the generator is to spawn ‘fake’ images that look like the training images. The job of the discriminator is to look at an image and output whether or not it is a real training image or a fake image from the generator. During training, the generator is constantly trying to outsmart the discriminator by generating better and better fakes, while the discriminator is working to become a better detective and correctly classify the real and fake images. The equilibrium of this game is when the generator is generating perfect fakes that look as if they came directly from the training data, and the discriminator is left to always guess at 50% confidence that the generator output is real or fake.

Now, lets define some notation to be used throughout tutorial starting with the discriminator. Let x be data representing an image. D(x) is the discriminator network which outputs the (scalar) probability that x came from training data rather than the generator. Here, since we are dealing with images the input to D(x) is an image of HWC size 3x64x64. Intuitively, D(x) should be HIGH when x comes from training data and LOW whenx comes from the generator. D(x) can also be thought of as a traditional binary classifier.

For the generator’s notation, let z be a latent space vector sampled from a standard normal distribution. G(z)represents the generator function which maps the latent vector z to data-space. The goal of G is to estimate the distribution that the training data comes from (p_{data}) so it can generate fake samples from that estimated distribution (p_g).

So, D(G(z)) is the probability (scalar) that the output of the generator G is a real image. As described in [Goodfellow’s paper](https://papers.nips.cc/paper/5423-generative-adversarial-nets.pdf), D and G play a minimax game in which D tries to maximize the probability it correctly classifies reals and fakes (logD(x)), and G tries to minimize the probability that D will predict its outputs are fake (log(1-D(G(x)))). From the paper, the GAN loss function is

\underset{G}{\text{min}} \underset{D}{\text{max}}V(D,G) = \mathbb{E}_{x\sim p_{data}(x)}\big[logD(x)\big] + \mathbb{E}_{z\sim p_{z}(z)}\big[log(1-D(G(z)))\big]

In theory, the solution to this minimax game is where p_g = p_{data}, and the discriminator guesses randomly if the inputs are real or fake. However, the convergence theory of GANs is still being actively researched and in reality models do not always train to this point.

### What is a DCGAN?

A DCGAN is a direct extension of the GAN described above, except that it explicitly uses convolutional and convolutional-transpose layers in the discriminator and generator, respectively. It was first described by Radford et. al. in the paper [Unsupervised Representation Learning With Deep Convolutional Generative Adversarial Networks](https://arxiv.org/pdf/1511.06434.pdf). The discriminator is made up of strided [convolution](https://pytorch.org/docs/stable/nn.html#torch.nn.Conv2d) layers, [batch norm](https://pytorch.org/docs/stable/nn.html#torch.nn.BatchNorm2d) layers, and[LeakyReLU](https://pytorch.org/docs/stable/nn.html#torch.nn.LeakyReLU) activations. The input is a 3x64x64 input image and the output is a scalar probability that the input is from the real data distribution. The generator is comprised of [convolutional-transpose](https://pytorch.org/docs/stable/nn.html#torch.nn.ConvTranspose2d) layers, batch norm layers, and [ReLU](https://pytorch.org/docs/stable/nn.html#relu) activations. The input is a latent vector, z, that is drawn from a standard normal distribution and the output is a 3x64x64 RGB image. The strided conv-transpose layers allow the latent vector to be transformed into a volume with the same shape as an image. In the paper, the authors also give some tips about how to setup the optimizers, how to calculate the loss functions, and how to initialize the model weights, all of which will be explained in the coming sections.

```
from __future__ import print_function
#%matplotlib inline
import argparse
import os
import random
import torch
import torch.nn as nn
import torch.nn.parallel
import torch.backends.cudnn as cudnn
import torch.optim as optim
import torch.utils.data
import torchvision.datasets as dset
import torchvision.transforms as transforms
import torchvision.utils as vutils
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.animation as animation
from Ipython.display import HTML

# Set random seem for reproducibility
manualSeed = 999
#manualSeed = random.randint(1, 10000) # use if you want new results
print("Random Seed: ", manualSeed)
random.seed(manualSeed)
torch.manual_seed(manualSeed)
```

Out:

```
Random Seed:  999
```

## Inputs

Let’s define some inputs for the run:

- **dataroot** - the path to the root of the dataset folder. We will talk more about the dataset in the next section
- **workers** - the number of worker threads for loading the data with the DataLoader
- **batch_size** - the batch size used in training. The DCGAN paper uses a batch size of 128
- **image_size** - the spatial size of the images used for training. This implementation defaults to 64x64. If another size is desired, the structures of D and G must be changed. See [here](https://github.com/pytorch/examples/issues/70) for more details
- **nc** - number of color channels in the input images. For color images this is 3
- **nz** - length of latent vector
- **ngf** - relates to the depth of feature maps carried through the generator
- **ndf** - sets the depth of feature maps propagated through the discriminator
- **num_epochs** - number of training epochs to run. Training for longer will probably lead to better results but will also take much longer
- **lr** - learning rate for training. As described in the DCGAN paper, this number should be 0.0002
- **beta1** - beta1 hyperparameter for Adam optimizers. As described in paper, this number should be 0.5
- **ngpu** - number of GPUs available. If this is 0, code will run in CPU mode. If this number is greater than 0 it will run on that number of GPUs

```
# Root directory for dataset
dataroot = "data/celeba"

# Number of workers for dataloader
workers = 2

# Batch size during training
batch_size = 128

# Spatial size of training images. All images will be resized to this
#   size using a transformer.
image_size = 64

# Number of channels in the training images. For color images this is 3
nc = 3

# Size of z latent vector (i.e. size of generator input)
nz = 100

# Size of feature maps in generator
ngf = 64

# Size of feature maps in discriminator
ndf = 64

# Number of training epochs
num_epochs = 5

# Learning rate for optimizers
lr = 0.0002

# Beta1 hyperparam for Adam optimizers
beta1 = 0.5

# Number of GPUs available. Use 0 for CPU mode.
ngpu = 1
```

## Data

In this tutorial we will use the [Celeb-A Faces dataset](http://mmlab.ie.cuhk.edu.hk/projects/CelebA.html) which can be downloaded at the linked site, or in [Google Drive](https://drive.google.com/drive/folders/0B7EVK8r0v71pTUZsaXdaSnZBZzg). The dataset will download as a file named *img_align_celeba.zip*. Once downloaded, create a directory named *celeba* and extract the zip file into that directory. Then, set the *dataroot* input for this notebook to the *celeba* directory you just created. The resulting directory structure should be:

```
/path/to/celeba
    -> img_align_celeba
        -> 188242.jpg
        -> 173822.jpg
        -> 284702.jpg
        -> 537394.jpg
           ...
```

This is an important step because we will be using the ImageFolder dataset class, which requires there to be subdirectories in the dataset’s root folder. Now, we can create the dataset, create the dataloader, set the device to run on, and finally visualize some of the training data.

```
# We can use an image folder dataset the way we have it setup.
# Create the dataset
dataset = dset.ImageFolder(root=dataroot,
                           transform=transforms.Compose([
                               transforms.Resize(image_size),
                               transforms.CenterCrop(image_size),
                               transforms.ToTensor(),
                               transforms.Normalize((0.5, 0.5, 0.5), (0.5, 0.5, 0.5)),
                           ]))
# Create the dataloader
dataloader = torch.utils.data.DataLoader(dataset, batch_size=batch_size,
                                         shuffle=True, num_workers=workers)

# Decide which device we want to run on
device = torch.device("cuda:0" if (torch.cuda.is_available() and ngpu > 0) else "cpu")

# Plot some training images
real_batch = next(iter(dataloader))
plt.figure(figsize=(8,8))
plt.axis("off")
plt.title("Training Images")
plt.imshow(np.transpose(vutils.make_grid(real_batch[0].to(device)[:64], padding=2, normalize=True).cpu(),(1,2,0)))
```

<center>

![](http://images.iterate.site/blog/image/20190629/fpXM6cNat468.png?imageslim){ width=55% }
</center>

## Implementation

With our input parameters set and the dataset prepared, we can now get into the implementation. We will start with the weigth initialization strategy, then talk about the generator, discriminator, loss functions, and training loop in detail.

### Weight Initialization

From the DCGAN paper, the authors specify that all model weights shall be randomly initialized from a Normal distribution with mean=0, stdev=0.02. The `weights_init` function takes an initialized model as input and reinitializes all convolutional, convolutional-transpose, and batch normalization layers to meet this criteria. This function is applied to the models immediately after initialization.

```
# custom weights initialization called on netG and netD
def weights_init(m):
    classname = m.__class__.__name__
    if classname.find('Conv') != -1:
        nn.init.normal_(m.weight.data, 0.0, 0.02)
    elif classname.find('BatchNorm') != -1:
        nn.init.normal_(m.weight.data, 1.0, 0.02)
        nn.init.constant_(m.bias.data, 0)
```

### Generator

The generator, G, is designed to map the latent space vector (z) to data-space. Since our data are images, converting z to data-space means ultimately creating a RGB image with the same size as the training images (i.e. 3x64x64). In practice, this is accomplished through a series of strided two dimensional convolutional transpose layers, each paired with a 2d batch norm layer and a relu activation. The output of the generator is fed through a tanh function to return it to the input data range of [-1,1]. It is worth noting the existence of the batch norm functions after the conv-transpose layers, as this is a critical contribution of the DCGAN paper. These layers help with the flow of gradients during training. An image of the generator from the DCGAN paper is shown below.

<center>

![](http://images.iterate.site/blog/image/20190629/JkLPchvFTN0L.png?imageslim){ width=55% }
</center>

Notice, the how the inputs we set in the input section (*nz*, *ngf*, and *nc*) influence the generator architecture in code. *nz* is the length of the z input vector, *ngf* relates to the size of the feature maps that are propagated through the generator, and *nc* is the number of channels in the output image (set to 3 for RGB images). Below is the code for the generator.

```
# Generator Code

class Generator(nn.Module):
    def __init__(self, ngpu):
        super(Generator, self).__init__()
        self.ngpu = ngpu
        self.main = nn.Sequential(
            # input is Z, going into a convolution
            nn.ConvTranspose2d( nz, ngf * 8, 4, 1, 0, bias=False),
            nn.BatchNorm2d(ngf * 8),
            nn.ReLU(True),
            # state size. (ngf*8) x 4 x 4
            nn.ConvTranspose2d(ngf * 8, ngf * 4, 4, 2, 1, bias=False),
            nn.BatchNorm2d(ngf * 4),
            nn.ReLU(True),
            # state size. (ngf*4) x 8 x 8
            nn.ConvTranspose2d( ngf * 4, ngf * 2, 4, 2, 1, bias=False),
            nn.BatchNorm2d(ngf * 2),
            nn.ReLU(True),
            # state size. (ngf*2) x 16 x 16
            nn.ConvTranspose2d( ngf * 2, ngf, 4, 2, 1, bias=False),
            nn.BatchNorm2d(ngf),
            nn.ReLU(True),
            # state size. (ngf) x 32 x 32
            nn.ConvTranspose2d( ngf, nc, 4, 2, 1, bias=False),
            nn.Tanh()
            # state size. (nc) x 64 x 64
        )

    def forward(self, input):
        return self.main(input)
```

Now, we can instantiate the generator and apply the `weights_init` function. Check out the printed model to see how the generator object is structured.

```
# Create the generator
netG = Generator(ngpu).to(device)

# Handle multi-gpu if desired
if (device.type == 'cuda') and (ngpu > 1):
    netG = nn.DataParallel(netG, list(range(ngpu)))

# Apply the weights_init function to randomly initialize all weights
#  to mean=0, stdev=0.2.
netG.apply(weights_init)

# Print the model
print(netG)
```

Out:

```
Generator(
  (main): Sequential(
    (0): ConvTranspose2d(100, 512, kernel_size=(4, 4), stride=(1, 1), bias=False)
    (1): BatchNorm2d(512, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
    (2): ReLU(inplace)
    (3): ConvTranspose2d(512, 256, kernel_size=(4, 4), stride=(2, 2), padding=(1, 1), bias=False)
    (4): BatchNorm2d(256, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
    (5): ReLU(inplace)
    (6): ConvTranspose2d(256, 128, kernel_size=(4, 4), stride=(2, 2), padding=(1, 1), bias=False)
    (7): BatchNorm2d(128, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
    (8): ReLU(inplace)
    (9): ConvTranspose2d(128, 64, kernel_size=(4, 4), stride=(2, 2), padding=(1, 1), bias=False)
    (10): BatchNorm2d(64, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
    (11): ReLU(inplace)
    (12): ConvTranspose2d(64, 3, kernel_size=(4, 4), stride=(2, 2), padding=(1, 1), bias=False)
    (13): Tanh()
  )
)
```

### Discriminator

As mentioned, the discriminator, D, is a binary classification network that takes an image as input and outputs a scalar probability that the input image is real (as opposed to fake). Here, D takes a 3x64x64 input image, processes it through a series of Conv2d, BatchNorm2d, and LeakyReLU layers, and outputs the final probability through a Sigmoid activation function. This architecture can be extended with more layers if necessary for the problem, but there is significance to the use of the strided convolution, BatchNorm, and LeakyReLUs. The DCGAN paper mentions it is a good practice to use strided convolution rather than pooling to downsample because it lets the network learn its own pooling function. Also batch norm and leaky relu functions promote healthy gradient flow which is critical for the learning process of both G and D.

Discriminator Code

```
class Discriminator(nn.Module):
    def __init__(self, ngpu):
        super(Discriminator, self).__init__()
        self.ngpu = ngpu
        self.main = nn.Sequential(
            # input is (nc) x 64 x 64
            nn.Conv2d(nc, ndf, 4, 2, 1, bias=False),
            nn.LeakyReLU(0.2, inplace=True),
            # state size. (ndf) x 32 x 32
            nn.Conv2d(ndf, ndf * 2, 4, 2, 1, bias=False),
            nn.BatchNorm2d(ndf * 2),
            nn.LeakyReLU(0.2, inplace=True),
            # state size. (ndf*2) x 16 x 16
            nn.Conv2d(ndf * 2, ndf * 4, 4, 2, 1, bias=False),
            nn.BatchNorm2d(ndf * 4),
            nn.LeakyReLU(0.2, inplace=True),
            # state size. (ndf*4) x 8 x 8
            nn.Conv2d(ndf * 4, ndf * 8, 4, 2, 1, bias=False),
            nn.BatchNorm2d(ndf * 8),
            nn.LeakyReLU(0.2, inplace=True),
            # state size. (ndf*8) x 4 x 4
            nn.Conv2d(ndf * 8, 1, 4, 1, 0, bias=False),
            nn.Sigmoid()
        )

    def forward(self, input):
        return self.main(input)
```

Now, as with the generator, we can create the discriminator, apply the `weights_init` function, and print the model’s structure.

```
# Create the Discriminator
netD = Discriminator(ngpu).to(device)

# Handle multi-gpu if desired
if (device.type == 'cuda') and (ngpu > 1):
    netD = nn.DataParallel(netD, list(range(ngpu)))

# Apply the weights_init function to randomly initialize all weights
#  to mean=0, stdev=0.2.
netD.apply(weights_init)

# Print the model
print(netD)
```

Out:

```
Discriminator(
  (main): Sequential(
    (0): Conv2d(3, 64, kernel_size=(4, 4), stride=(2, 2), padding=(1, 1), bias=False)
    (1): LeakyReLU(negative_slope=0.2, inplace)
    (2): Conv2d(64, 128, kernel_size=(4, 4), stride=(2, 2), padding=(1, 1), bias=False)
    (3): BatchNorm2d(128, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
    (4): LeakyReLU(negative_slope=0.2, inplace)
    (5): Conv2d(128, 256, kernel_size=(4, 4), stride=(2, 2), padding=(1, 1), bias=False)
    (6): BatchNorm2d(256, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
    (7): LeakyReLU(negative_slope=0.2, inplace)
    (8): Conv2d(256, 512, kernel_size=(4, 4), stride=(2, 2), padding=(1, 1), bias=False)
    (9): BatchNorm2d(512, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
    (10): LeakyReLU(negative_slope=0.2, inplace)
    (11): Conv2d(512, 1, kernel_size=(4, 4), stride=(1, 1), bias=False)
    (12): Sigmoid()
  )
)
```

### Loss Functions and Optimizers

With D and G setup, we can specify how they learn through the loss functions and optimizers. We will use the Binary Cross Entropy loss ([BCELoss](https://pytorch.org/docs/stable/nn.html#torch.nn.BCELoss)) function which is defined in PyTorch as:

\ell(x, y) = L = \{l_1,\dots,l_N\}^\top, \quad l_n = - \left[ y_n \cdot \log x_n + (1 - y_n) \cdot \log (1 - x_n) \right]

Notice how this function provides the calculation of both log components in the objective function (i.e. log(D(x)) and log(1-D(G(z)))). We can specify what part of the BCE equation to use with the y input. This is accomplished in the training loop which is coming up soon, but it is important to understand how we can choose which component we wish to calculate just by changing y (i.e. GT labels).

Next, we define our real label as 1 and the fake label as 0. These labels will be used when calculating the losses of D and G, and this is also the convention used in the original GAN paper. Finally, we set up two separate optimizers, one for D and one for G. As specified in the DCGAN paper, both are Adam optimizers with learning rate 0.0002 and Beta1 = 0.5. For keeping track of the generator’s learning progression, we will generate a fixed batch of latent vectors that are drawn from a Gaussian distribution (i.e. fixed_noise) . In the training loop, we will periodically input this fixed_noise into G, and over the iterations we will see images form out of the noise.

```
# Initialize BCELoss function
criterion = nn.BCELoss()

# Create batch of latent vectors that we will use to visualize
#  the progression of the generator
fixed_noise = torch.randn(64, nz, 1, 1, device=device)

# Establish convention for real and fake labels during training
real_label = 1
fake_label = 0

# Setup Adam optimizers for both G and D
optimizerD = optim.Adam(netD.parameters(), lr=lr, betas=(beta1, 0.999))
optimizerG = optim.Adam(netG.parameters(), lr=lr, betas=(beta1, 0.999))
```

### Training

Finally, now that we have all of the parts of the GAN framework defined, we can train it. Be mindful that training GANs is somewhat of an art form, as incorrect hyperparameter settings lead to mode collapse with little explanation of what went wrong. Here, we will closely follow Algorithm 1 from Goodfellow’s paper, while abiding by some of the best practices shown in [ganhacks](https://github.com/soumith/ganhacks). Namely, we will “construct different mini-batches for real and fake” images, and also adjust G’s objective function to maximize logD(G(z)). Training is split up into two main parts. Part 1 updates the Discriminator and Part 2 updates the Generator.

**Part 1 - Train the Discriminator**

Recall, the goal of training the discriminator is to maximize the probability of correctly classifying a given input as real or fake. In terms of Goodfellow, we wish to “update the discriminator by ascending its stochastic gradient”. Practically, we want to maximize log(D(x)) + log(1-D(G(z))). Due to the separate mini-batch suggestion from ganhacks, we will calculate this in two steps. First, we will construct a batch of real samples from the training set, forward pass through D, calculate the loss (log(D(x))), then calculate the gradients in a backward pass. Secondly, we will construct a batch of fake samples with the current generator, forward pass this batch through D, calculate the loss (log(1-D(G(z)))), and *accumulate* the gradients with a backward pass. Now, with the gradients accumulated from both the all-real and all-fake batches, we call a step of the Discriminator’s optimizer.

**Part 2 - Train the Generator**

As stated in the original paper, we want to train the Generator by minimizing log(1-D(G(z))) in an effort to generate better fakes. As mentioned, this was shown by Goodfellow to not provide sufficient gradients, especially early in the learning process. As a fix, we instead wish to maximize log(D(G(z))). In the code we accomplish this by: classifying the Generator output from Part 1 with the Discriminator, computing G’s loss *using real labels as GT*, computing G’s gradients in a backward pass, and finally updating G’s parameters with an optimizer step. It may seem counter-intuitive to use the real labels as GT labels for the loss function, but this allows us to use the log(x) part of the BCELoss (rather than the log(1-x) part) which is exactly what we want.

Finally, we will do some statistic reporting and at the end of each epoch we will push our fixed_noise batch through the generator to visually track the progress of G’s training. The training statistics reported are:

- **Loss_D** - discriminator loss calculated as the sum of losses for the all real and all fake batches (log(D(x)) + log(D(G(z)))).
- **Loss_G** - generator loss calculated as log(D(G(z)))
- **D(x)** - the average output (across the batch) of the discriminator for the all real batch. This should start close to 1 then theoretically converge to 0.5 when G gets better. Think about why this is.
- **D(G(z))** - average discriminator outputs for the all fake batch. The first number is before D is updated and the second number is after D is updated. These numbers should start near 0 and converge to 0.5 as G gets better. Think about why this is.

**Note:** This step might take a while, depending on how many epochs you run and if you removed some data from the dataset.

```
# Training Loop

# Lists to keep track of progress
img_list = []
G_losses = []
D_losses = []
iters = 0

print("Starting Training Loop...")
# For each epoch
for epoch in range(num_epochs):
    # For each batch in the dataloader
    for i, data in enumerate(dataloader, 0):

        ############################
        # (1) Update D network: maximize log(D(x)) + log(1 - D(G(z)))
        ###########################
        ## Train with all-real batch
        netD.zero_grad()
        # Format batch
        real_cpu = data[0].to(device)
        b_size = real_cpu.size(0)
        label = torch.full((b_size,), real_label, device=device)
        # Forward pass real batch through D
        output = netD(real_cpu).view(-1)
        # Calculate loss on all-real batch
        errD_real = criterion(output, label)
        # Calculate gradients for D in backward pass
        errD_real.backward()
        D_x = output.mean().item()

        ## Train with all-fake batch
        # Generate batch of latent vectors
        noise = torch.randn(b_size, nz, 1, 1, device=device)
        # Generate fake image batch with G
        fake = netG(noise)
        label.fill_(fake_label)
        # Classify all fake batch with D
        output = netD(fake.detach()).view(-1)
        # Calculate D's loss on the all-fake batch
        errD_fake = criterion(output, label)
        # Calculate the gradients for this batch
        errD_fake.backward()
        D_G_z1 = output.mean().item()
        # Add the gradients from the all-real and all-fake batches
        errD = errD_real + errD_fake
        # Update D
        optimizerD.step()

        ############################
        # (2) Update G network: maximize log(D(G(z)))
        ###########################
        netG.zero_grad()
        label.fill_(real_label)  # fake labels are real for generator cost
        # Since we just updated D, perform another forward pass of all-fake batch through D
        output = netD(fake).view(-1)
        # Calculate G's loss based on this output
        errG = criterion(output, label)
        # Calculate gradients for G
        errG.backward()
        D_G_z2 = output.mean().item()
        # Update G
        optimizerG.step()

        # Output training stats
        if i % 50 == 0:
            print('[%d/%d][%d/%d]\tLoss_D: %.4f\tLoss_G: %.4f\tD(x): %.4f\tD(G(z)): %.4f / %.4f'
                  % (epoch, num_epochs, i, len(dataloader),
                     errD.item(), errG.item(), D_x, D_G_z1, D_G_z2))

        # Save Losses for plotting later
        G_losses.append(errG.item())
        D_losses.append(errD.item())

        # Check how the generator is doing by saving G's output on fixed_noise
        if (iters % 500 == 0) or ((epoch == num_epochs-1) and (i == len(dataloader)-1)):
            with torch.no_grad():
                fake = netG(fixed_noise).detach().cpu()
            img_list.append(vutils.make_grid(fake, padding=2, normalize=True))

        iters += 1
```

Out:

```
Starting Training Loop...
[0/5][0/1583]   Loss_D: 1.7410  Loss_G: 4.7765  D(x): 0.5343    D(G(z)): 0.5771 / 0.0136
[0/5][50/1583]  Loss_D: 0.0970  Loss_G: 7.3876  D(x): 0.9762    D(G(z)): 0.0500 / 0.0017
[0/5][100/1583] Loss_D: 0.7465  Loss_G: 11.8024 D(x): 0.9807    D(G(z)): 0.3861 / 0.0000
[0/5][150/1583] Loss_D: 0.9122  Loss_G: 10.3798 D(x): 0.9154    D(G(z)): 0.4858 / 0.0002
[0/5][200/1583] Loss_D: 0.7143  Loss_G: 3.9219  D(x): 0.6308    D(G(z)): 0.0256 / 0.0407
[0/5][250/1583] Loss_D: 0.5147  Loss_G: 3.8340  D(x): 0.7791    D(G(z)): 0.1282 / 0.0370
[0/5][300/1583] Loss_D: 1.7211  Loss_G: 3.3890  D(x): 0.2996    D(G(z)): 0.0018 / 0.0783
[0/5][350/1583] Loss_D: 0.9044  Loss_G: 2.0067  D(x): 0.5382    D(G(z)): 0.0712 / 0.1965
[0/5][400/1583] Loss_D: 0.3062  Loss_G: 5.7195  D(x): 0.8823    D(G(z)): 0.1241 / 0.0062
[0/5][450/1583] Loss_D: 0.4525  Loss_G: 4.7466  D(x): 0.8692    D(G(z)): 0.2217 / 0.0180
[0/5][500/1583] Loss_D: 1.1330  Loss_G: 3.8996  D(x): 0.4620    D(G(z)): 0.0123 / 0.0572
[0/5][550/1583] Loss_D: 0.8309  Loss_G: 5.3636  D(x): 0.7611    D(G(z)): 0.3329 / 0.0125
[0/5][600/1583] Loss_D: 0.5957  Loss_G: 4.2152  D(x): 0.6512    D(G(z)): 0.0295 / 0.0320
[0/5][650/1583] Loss_D: 0.9307  Loss_G: 7.8156  D(x): 0.9415    D(G(z)): 0.5124 / 0.0011
[0/5][700/1583] Loss_D: 0.2999  Loss_G: 4.4143  D(x): 0.8703    D(G(z)): 0.1176 / 0.0193
[0/5][750/1583] Loss_D: 0.7836  Loss_G: 8.3069  D(x): 0.9385    D(G(z)): 0.4544 / 0.0007
[0/5][800/1583] Loss_D: 0.6383  Loss_G: 6.4706  D(x): 0.9433    D(G(z)): 0.3672 / 0.0049
[0/5][850/1583] Loss_D: 0.4683  Loss_G: 3.9057  D(x): 0.8098    D(G(z)): 0.1307 / 0.0339
[0/5][900/1583] Loss_D: 0.2767  Loss_G: 5.0191  D(x): 0.9273    D(G(z)): 0.1599 / 0.0134
[0/5][950/1583] Loss_D: 0.3336  Loss_G: 3.5024  D(x): 0.8711    D(G(z)): 0.1438 / 0.0506
[0/5][1000/1583]        Loss_D: 0.3906  Loss_G: 4.3941  D(x): 0.8509    D(G(z)): 0.1591 / 0.0237
[0/5][1050/1583]        Loss_D: 0.5886  Loss_G: 2.6354  D(x): 0.7153    D(G(z)): 0.1316 / 0.1244
[0/5][1100/1583]        Loss_D: 0.9326  Loss_G: 5.7653  D(x): 0.8840    D(G(z)): 0.4504 / 0.0139
[0/5][1150/1583]        Loss_D: 0.5095  Loss_G: 3.8890  D(x): 0.7879    D(G(z)): 0.1758 / 0.0366
[0/5][1200/1583]        Loss_D: 0.5811  Loss_G: 3.4397  D(x): 0.7072    D(G(z)): 0.1080 / 0.0560
[0/5][1250/1583]        Loss_D: 0.6703  Loss_G: 6.7806  D(x): 0.8745    D(G(z)): 0.3416 / 0.0025
[0/5][1300/1583]        Loss_D: 1.3335  Loss_G: 2.7742  D(x): 0.3929    D(G(z)): 0.0066 / 0.1293
[0/5][1350/1583]        Loss_D: 0.6162  Loss_G: 2.3932  D(x): 0.6696    D(G(z)): 0.0825 / 0.1368
[0/5][1400/1583]        Loss_D: 0.4152  Loss_G: 4.5638  D(x): 0.9046    D(G(z)): 0.2279 / 0.0183
[0/5][1450/1583]        Loss_D: 0.5327  Loss_G: 6.0527  D(x): 0.9580    D(G(z)): 0.3354 / 0.0056
[0/5][1500/1583]        Loss_D: 0.3840  Loss_G: 3.5863  D(x): 0.8170    D(G(z)): 0.1248 / 0.0425
[0/5][1550/1583]        Loss_D: 0.3435  Loss_G: 3.8842  D(x): 0.8631    D(G(z)): 0.1483 / 0.0336
[1/5][0/1583]   Loss_D: 0.4277  Loss_G: 4.4370  D(x): 0.9382    D(G(z)): 0.2615 / 0.0197
[1/5][50/1583]  Loss_D: 0.4324  Loss_G: 3.3108  D(x): 0.8827    D(G(z)): 0.2319 / 0.0528
[1/5][100/1583] Loss_D: 0.5938  Loss_G: 2.3543  D(x): 0.6510    D(G(z)): 0.0556 / 0.1429
[1/5][150/1583] Loss_D: 0.8280  Loss_G: 1.7701  D(x): 0.5450    D(G(z)): 0.0644 / 0.2431
[1/5][200/1583] Loss_D: 0.4149  Loss_G: 3.4936  D(x): 0.8413    D(G(z)): 0.1813 / 0.0444
[1/5][250/1583] Loss_D: 0.4307  Loss_G: 3.1091  D(x): 0.8062    D(G(z)): 0.1431 / 0.0678
[1/5][300/1583] Loss_D: 0.6110  Loss_G: 5.5691  D(x): 0.9092    D(G(z)): 0.3468 / 0.0089
[1/5][350/1583] Loss_D: 2.2207  Loss_G: 2.0806  D(x): 0.1738    D(G(z)): 0.0021 / 0.2002
[1/5][400/1583] Loss_D: 0.2937  Loss_G: 4.2036  D(x): 0.8998    D(G(z)): 0.1484 / 0.0236
[1/5][450/1583] Loss_D: 1.4573  Loss_G: 1.3451  D(x): 0.3551    D(G(z)): 0.0152 / 0.3649
[1/5][500/1583] Loss_D: 0.4528  Loss_G: 3.7265  D(x): 0.8373    D(G(z)): 0.2072 / 0.0382
[1/5][550/1583] Loss_D: 0.5096  Loss_G: 4.0309  D(x): 0.8700    D(G(z)): 0.2713 / 0.0269
[1/5][600/1583] Loss_D: 1.0324  Loss_G: 6.3865  D(x): 0.9754    D(G(z)): 0.5834 / 0.0034
[1/5][650/1583] Loss_D: 0.3631  Loss_G: 3.4525  D(x): 0.8153    D(G(z)): 0.1063 / 0.0509
[1/5][700/1583] Loss_D: 0.3041  Loss_G: 2.7197  D(x): 0.8887    D(G(z)): 0.1526 / 0.0932
[1/5][750/1583] Loss_D: 0.6470  Loss_G: 3.9104  D(x): 0.9106    D(G(z)): 0.3645 / 0.0352
[1/5][800/1583] Loss_D: 0.7648  Loss_G: 2.0570  D(x): 0.6723    D(G(z)): 0.2086 / 0.1725
[1/5][850/1583] Loss_D: 0.4573  Loss_G: 3.1397  D(x): 0.8334    D(G(z)): 0.2058 / 0.0657
[1/5][900/1583] Loss_D: 0.4113  Loss_G: 2.8877  D(x): 0.7586    D(G(z)): 0.0784 / 0.0823
[1/5][950/1583] Loss_D: 0.8829  Loss_G: 2.0020  D(x): 0.5786    D(G(z)): 0.1585 / 0.1886
[1/5][1000/1583]        Loss_D: 0.5947  Loss_G: 4.6668  D(x): 0.9236    D(G(z)): 0.3552 / 0.0153
[1/5][1050/1583]        Loss_D: 0.5024  Loss_G: 2.5094  D(x): 0.6922    D(G(z)): 0.0749 / 0.1077
[1/5][1100/1583]        Loss_D: 0.4957  Loss_G: 2.3835  D(x): 0.7133    D(G(z)): 0.0908 / 0.1351
[1/5][1150/1583]        Loss_D: 0.5783  Loss_G: 2.4495  D(x): 0.7520    D(G(z)): 0.1976 / 0.1201
[1/5][1200/1583]        Loss_D: 0.4757  Loss_G: 3.0061  D(x): 0.9168    D(G(z)): 0.2940 / 0.0672
[1/5][1250/1583]        Loss_D: 0.5674  Loss_G: 3.1954  D(x): 0.9093    D(G(z)): 0.3245 / 0.0670
[1/5][1300/1583]        Loss_D: 0.6865  Loss_G: 4.2065  D(x): 0.9371    D(G(z)): 0.4189 / 0.0222
[1/5][1350/1583]        Loss_D: 0.8484  Loss_G: 4.7369  D(x): 0.9063    D(G(z)): 0.4681 / 0.0187
[1/5][1400/1583]        Loss_D: 0.4484  Loss_G: 4.0615  D(x): 0.9368    D(G(z)): 0.2882 / 0.0238
[1/5][1450/1583]        Loss_D: 0.8247  Loss_G: 5.5296  D(x): 0.9475    D(G(z)): 0.4893 / 0.0100
[1/5][1500/1583]        Loss_D: 0.6002  Loss_G: 2.7162  D(x): 0.8361    D(G(z)): 0.3019 / 0.0893
[1/5][1550/1583]        Loss_D: 0.4514  Loss_G: 2.5365  D(x): 0.7804    D(G(z)): 0.1485 / 0.1046
[2/5][0/1583]   Loss_D: 0.5371  Loss_G: 2.2121  D(x): 0.7208    D(G(z)): 0.1405 / 0.1486
[2/5][50/1583]  Loss_D: 1.4293  Loss_G: 1.5383  D(x): 0.3270    D(G(z)): 0.0225 / 0.2751
[2/5][100/1583] Loss_D: 0.7791  Loss_G: 1.1362  D(x): 0.5179    D(G(z)): 0.0328 / 0.3974
[2/5][150/1583] Loss_D: 0.5094  Loss_G: 2.6882  D(x): 0.8045    D(G(z)): 0.2144 / 0.0924
[2/5][200/1583] Loss_D: 1.6749  Loss_G: 1.8245  D(x): 0.2644    D(G(z)): 0.0094 / 0.2217
[2/5][250/1583] Loss_D: 1.5443  Loss_G: 1.1897  D(x): 0.2853    D(G(z)): 0.0259 / 0.3743
[2/5][300/1583] Loss_D: 0.6269  Loss_G: 3.6420  D(x): 0.9050    D(G(z)): 0.3743 / 0.0369
[2/5][350/1583] Loss_D: 0.6075  Loss_G: 2.5398  D(x): 0.6959    D(G(z)): 0.1525 / 0.1169
[2/5][400/1583] Loss_D: 0.4172  Loss_G: 2.5260  D(x): 0.8347    D(G(z)): 0.1915 / 0.1066
[2/5][450/1583] Loss_D: 0.7049  Loss_G: 1.9934  D(x): 0.5611    D(G(z)): 0.0248 / 0.1802
[2/5][500/1583] Loss_D: 1.2043  Loss_G: 1.4110  D(x): 0.3794    D(G(z)): 0.0452 / 0.3063
[2/5][550/1583] Loss_D: 1.2075  Loss_G: 0.8934  D(x): 0.3682    D(G(z)): 0.0523 / 0.4610
[2/5][600/1583] Loss_D: 0.4658  Loss_G: 2.8172  D(x): 0.8749    D(G(z)): 0.2553 / 0.0803
[2/5][650/1583] Loss_D: 0.9255  Loss_G: 3.9089  D(x): 0.9073    D(G(z)): 0.5049 / 0.0308
[2/5][700/1583] Loss_D: 0.4909  Loss_G: 2.6780  D(x): 0.7493    D(G(z)): 0.1482 / 0.0945
[2/5][750/1583] Loss_D: 0.7156  Loss_G: 0.8278  D(x): 0.5596    D(G(z)): 0.0441 / 0.4794
[2/5][800/1583] Loss_D: 0.5110  Loss_G: 2.0730  D(x): 0.6862    D(G(z)): 0.0770 / 0.1656
[2/5][850/1583] Loss_D: 0.7568  Loss_G: 3.5540  D(x): 0.8115    D(G(z)): 0.3736 / 0.0394
[2/5][900/1583] Loss_D: 0.7316  Loss_G: 3.6348  D(x): 0.9147    D(G(z)): 0.4332 / 0.0355
[2/5][950/1583] Loss_D: 0.4500  Loss_G: 3.0410  D(x): 0.8614    D(G(z)): 0.2346 / 0.0642
[2/5][1000/1583]        Loss_D: 0.8289  Loss_G: 3.6698  D(x): 0.8887    D(G(z)): 0.4719 / 0.0340
[2/5][1050/1583]        Loss_D: 0.5876  Loss_G: 1.9556  D(x): 0.6640    D(G(z)): 0.0991 / 0.1841
[2/5][1100/1583]        Loss_D: 0.4856  Loss_G: 2.6838  D(x): 0.7739    D(G(z)): 0.1720 / 0.0926
[2/5][1150/1583]        Loss_D: 0.5126  Loss_G: 3.1566  D(x): 0.8738    D(G(z)): 0.2852 / 0.0565
[2/5][1200/1583]        Loss_D: 0.5474  Loss_G: 2.4431  D(x): 0.7070    D(G(z)): 0.1349 / 0.1178
[2/5][1250/1583]        Loss_D: 0.6033  Loss_G: 3.8954  D(x): 0.9400    D(G(z)): 0.3878 / 0.0283
[2/5][1300/1583]        Loss_D: 0.5561  Loss_G: 2.7264  D(x): 0.6888    D(G(z)): 0.1181 / 0.0897
[2/5][1350/1583]        Loss_D: 0.5960  Loss_G: 3.3690  D(x): 0.8653    D(G(z)): 0.3341 / 0.0461
[2/5][1400/1583]        Loss_D: 0.4791  Loss_G: 2.7906  D(x): 0.8520    D(G(z)): 0.2482 / 0.0774
[2/5][1450/1583]        Loss_D: 1.0043  Loss_G: 1.3384  D(x): 0.4523    D(G(z)): 0.0518 / 0.3312
[2/5][1500/1583]        Loss_D: 1.1454  Loss_G: 0.8031  D(x): 0.4097    D(G(z)): 0.0778 / 0.4831
[2/5][1550/1583]        Loss_D: 0.7281  Loss_G: 1.3043  D(x): 0.6083    D(G(z)): 0.1468 / 0.3158
[3/5][0/1583]   Loss_D: 0.6874  Loss_G: 2.9105  D(x): 0.8977    D(G(z)): 0.4015 / 0.0718
[3/5][50/1583]  Loss_D: 0.8232  Loss_G: 1.3518  D(x): 0.5537    D(G(z)): 0.1145 / 0.3063
[3/5][100/1583] Loss_D: 1.1111  Loss_G: 3.8186  D(x): 0.9300    D(G(z)): 0.5787 / 0.0338
[3/5][150/1583] Loss_D: 2.1501  Loss_G: 5.7021  D(x): 0.9629    D(G(z)): 0.8391 / 0.0053
[3/5][200/1583] Loss_D: 0.4407  Loss_G: 3.5393  D(x): 0.8657    D(G(z)): 0.2358 / 0.0375
[3/5][250/1583] Loss_D: 1.6148  Loss_G: 1.0549  D(x): 0.2869    D(G(z)): 0.0419 / 0.4505
[3/5][300/1583] Loss_D: 0.5261  Loss_G: 2.6280  D(x): 0.8517    D(G(z)): 0.2754 / 0.0898
[3/5][350/1583] Loss_D: 0.5353  Loss_G: 2.1975  D(x): 0.7648    D(G(z)): 0.1981 / 0.1443
[3/5][400/1583] Loss_D: 0.9490  Loss_G: 3.4311  D(x): 0.9184    D(G(z)): 0.5260 / 0.0461
[3/5][450/1583] Loss_D: 0.7868  Loss_G: 0.8195  D(x): 0.5740    D(G(z)): 0.1387 / 0.4873
[3/5][500/1583] Loss_D: 0.3867  Loss_G: 2.5272  D(x): 0.8300    D(G(z)): 0.1614 / 0.1005
[3/5][550/1583] Loss_D: 0.8742  Loss_G: 3.8900  D(x): 0.9277    D(G(z)): 0.5043 / 0.0286
[3/5][600/1583] Loss_D: 0.7140  Loss_G: 1.3940  D(x): 0.5711    D(G(z)): 0.0663 / 0.3066
[3/5][650/1583] Loss_D: 0.3957  Loss_G: 3.5908  D(x): 0.8164    D(G(z)): 0.1498 / 0.0440
[3/5][700/1583] Loss_D: 0.8354  Loss_G: 4.1706  D(x): 0.8636    D(G(z)): 0.4574 / 0.0230
[3/5][750/1583] Loss_D: 0.5680  Loss_G: 2.1763  D(x): 0.7363    D(G(z)): 0.1816 / 0.1369
[3/5][800/1583] Loss_D: 0.6351  Loss_G: 2.0249  D(x): 0.6974    D(G(z)): 0.1999 / 0.1659
[3/5][850/1583] Loss_D: 0.7132  Loss_G: 1.1100  D(x): 0.5758    D(G(z)): 0.0708 / 0.3776
[3/5][900/1583] Loss_D: 0.7673  Loss_G: 1.2005  D(x): 0.5955    D(G(z)): 0.1491 / 0.3399
[3/5][950/1583] Loss_D: 0.8552  Loss_G: 3.5365  D(x): 0.9107    D(G(z)): 0.4866 / 0.0381
[3/5][1000/1583]        Loss_D: 1.8766  Loss_G: 0.8448  D(x): 0.2140    D(G(z)): 0.0308 / 0.4826
[3/5][1050/1583]        Loss_D: 0.5885  Loss_G: 1.6747  D(x): 0.6481    D(G(z)): 0.1010 / 0.2281
[3/5][1100/1583]        Loss_D: 0.6429  Loss_G: 2.7670  D(x): 0.8102    D(G(z)): 0.3196 / 0.0807
[3/5][1150/1583]        Loss_D: 1.1353  Loss_G: 0.7328  D(x): 0.4536    D(G(z)): 0.1377 / 0.5201
[3/5][1200/1583]        Loss_D: 0.4691  Loss_G: 3.1328  D(x): 0.8038    D(G(z)): 0.1865 / 0.0598
[3/5][1250/1583]        Loss_D: 0.7917  Loss_G: 3.9241  D(x): 0.9253    D(G(z)): 0.4647 / 0.0281
[3/5][1300/1583]        Loss_D: 0.8187  Loss_G: 1.2049  D(x): 0.5380    D(G(z)): 0.0799 / 0.3457
[3/5][1350/1583]        Loss_D: 0.7272  Loss_G: 2.7083  D(x): 0.7990    D(G(z)): 0.3482 / 0.0827
[3/5][1400/1583]        Loss_D: 1.1593  Loss_G: 3.0037  D(x): 0.8417    D(G(z)): 0.5491 / 0.0818
[3/5][1450/1583]        Loss_D: 0.6652  Loss_G: 2.2496  D(x): 0.6993    D(G(z)): 0.2241 / 0.1321
[3/5][1500/1583]        Loss_D: 0.7449  Loss_G: 3.8253  D(x): 0.9254    D(G(z)): 0.4531 / 0.0290
[3/5][1550/1583]        Loss_D: 0.6512  Loss_G: 2.6410  D(x): 0.7928    D(G(z)): 0.3067 / 0.0894
[4/5][0/1583]   Loss_D: 1.6244  Loss_G: 4.9382  D(x): 0.9480    D(G(z)): 0.7389 / 0.0130
[4/5][50/1583]  Loss_D: 0.8133  Loss_G: 1.3301  D(x): 0.5335    D(G(z)): 0.0738 / 0.3255
[4/5][100/1583] Loss_D: 0.7493  Loss_G: 3.8537  D(x): 0.9069    D(G(z)): 0.4435 / 0.0286
[4/5][150/1583] Loss_D: 1.3529  Loss_G: 3.5755  D(x): 0.8691    D(G(z)): 0.6404 / 0.0410
[4/5][200/1583] Loss_D: 0.6289  Loss_G: 3.6080  D(x): 0.9148    D(G(z)): 0.3908 / 0.0350
[4/5][250/1583] Loss_D: 0.5749  Loss_G: 2.6441  D(x): 0.8344    D(G(z)): 0.2863 / 0.0914
[4/5][300/1583] Loss_D: 2.1005  Loss_G: 0.0192  D(x): 0.2142    D(G(z)): 0.0788 / 0.9813
[4/5][350/1583] Loss_D: 0.6545  Loss_G: 2.6521  D(x): 0.7929    D(G(z)): 0.3107 / 0.0903
[4/5][400/1583] Loss_D: 0.6472  Loss_G: 3.3982  D(x): 0.8644    D(G(z)): 0.3526 / 0.0466
[4/5][450/1583] Loss_D: 0.7619  Loss_G: 2.2528  D(x): 0.7262    D(G(z)): 0.3056 / 0.1354
[4/5][500/1583] Loss_D: 0.5958  Loss_G: 3.7782  D(x): 0.9559    D(G(z)): 0.3864 / 0.0303
[4/5][550/1583] Loss_D: 0.7567  Loss_G: 3.2539  D(x): 0.8241    D(G(z)): 0.3904 / 0.0525
[4/5][600/1583] Loss_D: 0.5094  Loss_G: 2.9529  D(x): 0.8504    D(G(z)): 0.2559 / 0.0713
[4/5][650/1583] Loss_D: 1.0401  Loss_G: 3.4868  D(x): 0.8320    D(G(z)): 0.5098 / 0.0454
[4/5][700/1583] Loss_D: 0.5789  Loss_G: 2.8586  D(x): 0.8645    D(G(z)): 0.3110 / 0.0785
[4/5][750/1583] Loss_D: 0.6133  Loss_G: 3.0673  D(x): 0.8090    D(G(z)): 0.2972 / 0.0601
[4/5][800/1583] Loss_D: 0.7776  Loss_G: 1.8326  D(x): 0.5637    D(G(z)): 0.0989 / 0.2031
[4/5][850/1583] Loss_D: 0.6367  Loss_G: 1.6715  D(x): 0.6683    D(G(z)): 0.1617 / 0.2290
[4/5][900/1583] Loss_D: 0.6799  Loss_G: 1.7540  D(x): 0.6689    D(G(z)): 0.1882 / 0.2090
[4/5][950/1583] Loss_D: 0.5481  Loss_G: 1.6366  D(x): 0.6749    D(G(z)): 0.0971 / 0.2302
[4/5][1000/1583]        Loss_D: 0.9257  Loss_G: 3.5284  D(x): 0.9337    D(G(z)): 0.5150 / 0.0457
[4/5][1050/1583]        Loss_D: 0.5365  Loss_G: 2.0475  D(x): 0.7565    D(G(z)): 0.1905 / 0.1527
[4/5][1100/1583]        Loss_D: 0.5168  Loss_G: 3.3355  D(x): 0.8811    D(G(z)): 0.2938 / 0.0464
[4/5][1150/1583]        Loss_D: 0.4738  Loss_G: 2.9217  D(x): 0.8533    D(G(z)): 0.2460 / 0.0718
[4/5][1200/1583]        Loss_D: 0.5412  Loss_G: 3.2415  D(x): 0.8575    D(G(z)): 0.2889 / 0.0547
[4/5][1250/1583]        Loss_D: 1.1634  Loss_G: 0.8186  D(x): 0.3897    D(G(z)): 0.0721 / 0.4896
[4/5][1300/1583]        Loss_D: 0.4392  Loss_G: 2.8852  D(x): 0.8208    D(G(z)): 0.1895 / 0.0814
[4/5][1350/1583]        Loss_D: 0.6328  Loss_G: 3.4688  D(x): 0.8023    D(G(z)): 0.3092 / 0.0432
[4/5][1400/1583]        Loss_D: 0.4807  Loss_G: 2.8489  D(x): 0.7959    D(G(z)): 0.1891 / 0.0820
[4/5][1450/1583]        Loss_D: 0.5848  Loss_G: 1.3567  D(x): 0.6604    D(G(z)): 0.1060 / 0.2945
[4/5][1500/1583]        Loss_D: 0.7616  Loss_G: 1.8773  D(x): 0.6014    D(G(z)): 0.1454 / 0.1846
[4/5][1550/1583]        Loss_D: 1.1731  Loss_G: 0.5555  D(x): 0.3834    D(G(z)): 0.0433 / 0.6155
```

## Results

Finally, lets check out how we did. Here, we will look at three different results. First, we will see how D and G’s losses changed during training. Second, we will visualize G’s output on the fixed_noise batch for every epoch. And third, we will look at a batch of real data next to a batch of fake data from G.

**Loss versus training iteration**

Below is a plot of D & G’s losses versus training iterations.

```
plt.figure(figsize=(10,5))
plt.title("Generator and Discriminator Loss During Training")
plt.plot(G_losses,label="G")
plt.plot(D_losses,label="D")
plt.xlabel("iterations")
plt.ylabel("Loss")
plt.legend()
plt.show()
```

<center>

![](http://images.iterate.site/blog/image/20190629/rjN6OVxY587i.png?imageslim){ width=55% }
</center>

**Visualization of G’s progression**

Remember how we saved the generator’s output on the fixed_noise batch after every epoch of training. Now, we can visualize the training progression of G with an animation. Press the play button to start the animation.

```
#%%capture
fig = plt.figure(figsize=(8,8))
plt.axis("off")
ims = [[plt.imshow(np.transpose(i,(1,2,0)), animated=True)] for i in img_list]
ani = animation.ArtistAnimation(fig, ims, interval=1000, repeat_delay=1000, blit=True)

HTML(ani.to_jshtml())
```
<center>

![](http://images.iterate.site/blog/image/20190629/XDbt4cAaHAeY.png?imageslim){ width=55% }
</center>


**Real Images vs. Fake Images**

Finally, lets take a look at some real images and fake images side by side.

```
# Grab a batch of real images from the dataloader
real_batch = next(iter(dataloader))

# Plot the real images
plt.figure(figsize=(15,15))
plt.subplot(1,2,1)
plt.axis("off")
plt.title("Real Images")
plt.imshow(np.transpose(vutils.make_grid(real_batch[0].to(device)[:64], padding=5, normalize=True).cpu(),(1,2,0)))

# Plot the fake images from the last epoch
plt.subplot(1,2,2)
plt.axis("off")
plt.title("Fake Images")
plt.imshow(np.transpose(img_list[-1],(1,2,0)))
plt.show()
```
<center>

![](http://images.iterate.site/blog/image/20190629/tpunMJgNwWvE.png?imageslim){ width=55% }
</center>


## Where to Go Next

We have reached the end of our journey, but there are several places you could go from here. You could:

- Train for longer to see how good the results get
- Modify this model to take a different dataset and possibly change the size of the images and the model architecture
- Check out some other cool GAN projects [here](https://github.com/nashory/gans-awesome-applications)
- Create GANs that generate [music](https://deepmind.com/blog/wavenet-generative-model-raw-audio/)




# 相关

- [DCGAN TUTORIAL](https://pytorch.org/tutorials/beginner/dcgan_faces_tutorial.html)
