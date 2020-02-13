---
title: 01 Creating Extensions Using numpy and scipy
toc: true
date: 2019-06-29
---
# CREATING EXTENSIONS USING NUMPY AND SCIPY

**Author**: [Adam Paszke](https://github.com/apaszke)

**Updated by**: [Adam Dziedzic](https://github.com/adam-dziedzic)

In this tutorial, we shall go through two tasks:

1. Create a neural network layer with no parameters.

   > - This calls into **numpy** as part of its implementation

2. Create a neural network layer that has learnable weights

   > - This calls into **SciPy** as part of its implementation

```
import torch
from torch.autograd import Function
```

## Parameter-less example

This layer doesn’t particularly do anything useful or mathematically correct.

It is aptly named BadFFTFunction

**Layer Implementation**

```
from numpy.fft import rfft2, irfft2


class BadFFTFunction(Function):

    def forward(self, input):
        numpy_input = input.detach().numpy()
        result = abs(rfft2(numpy_input))
        return input.new(result)

    def backward(self, grad_output):
        numpy_go = grad_output.numpy()
        result = irfft2(numpy_go)
        return grad_output.new(result)

# since this layer does not have any parameters, we can
# simply declare this as a function, rather than as an nn.Module class


def incorrect_fft(input):
    return BadFFTFunction()(input)
```

**Example usage of the created layer:**

```
input = torch.randn(8, 8, requires_grad=True)
result = incorrect_fft(input)
print(result)
result.backward(torch.randn(result.size()))
print(input)
```

Out:

```
tensor([[ 6.0109, 10.1737,  1.6463,  7.0552,  9.4915],
        [ 2.3966, 17.3371,  7.7631,  1.5111,  4.0304],
        [11.9083,  4.6854,  3.2047,  7.1667, 19.0580],
        [ 4.0054,  4.6734,  4.0319,  9.5528,  9.5643],
        [ 3.7321,  3.5226, 14.7737,  4.1725,  5.1574],
        [ 4.0054,  5.8267, 12.5935, 11.0283,  9.5643],
        [11.9083,  7.0578,  8.9575,  0.8138, 19.0580],
        [ 2.3966, 11.1555,  6.5373,  5.4814,  4.0304]],
       grad_fn=<BadFFTFunction>)
tensor([[-0.0705,  0.3470, -0.6537,  1.5586,  0.4001,  2.4423, -0.3818,  0.4327],
        [-2.0173,  0.4235,  0.5730, -1.7962, -0.3061, -0.4203,  0.2828,  0.3642],
        [ 2.6415, -0.9624, -0.2076, -1.3889,  0.0127, -1.8734,  1.7997,  0.2824],
        [ 1.6604,  0.2717, -0.8087,  0.1267,  0.5707, -0.1348,  0.3437,  0.3718],
        [ 0.1444,  0.7772, -2.3381, -0.8291,  1.1661,  1.4787,  0.2676,  0.7561],
        [-0.5873, -2.0619,  0.4305,  0.3377, -0.3438, -0.6172,  1.2530, -0.0514],
        [-1.0257,  0.5213, -0.4531, -0.1260,  2.3357, -0.6220, -0.1181, -1.4420],
        [-0.9502, -0.3095,  1.6633,  0.5051,  1.0407,  0.0806,  1.4273, -0.1825]],
       requires_grad=True)
```

## Parametrized example

In deep learning literature, this layer is confusingly referred to as convolution while the actual operation is cross-correlation (the only difference is that filter is flipped for convolution, which is not the case for cross-correlation).

Implementation of a layer with learnable weights, where cross-correlation has a filter (kernel) that represents weights.

The backward pass computes the gradient wrt the input and the gradient wrt the filter.

```
from numpy import flip
import numpy as np
from scipy.signal import convolve2d, correlate2d
from torch.nn.modules.module import Module
from torch.nn.parameter import Parameter


class ScipyConv2dFunction(Function):
    @staticmethod
    def forward(ctx, input, filter, bias):
        # detach so we can cast to NumPy
        input, filter, bias = input.detach(), filter.detach(), bias.detach()
        result = correlate2d(input.numpy(), filter.numpy(), mode='valid')
        result += bias.numpy()
        ctx.save_for_backward(input, filter, bias)
        return torch.as_tensor(result, dtype=input.dtype)

    @staticmethod
    def backward(ctx, grad_output):
        grad_output = grad_output.detach()
        input, filter, bias = ctx.saved_tensors
        grad_output = grad_output.numpy()
        grad_bias = np.sum(grad_output, keepdims=True)
        grad_input = convolve2d(grad_output, filter.numpy(), mode='full')
        # the previous line can be expressed equivalently as:
        # grad_input = correlate2d(grad_output, flip(flip(filter.numpy(), axis=0), axis=1), mode='full')
        grad_filter = correlate2d(input.numpy(), grad_output, mode='valid')
        return torch.from_numpy(grad_input), torch.from_numpy(grad_filter).to(torch.float), torch.from_numpy(grad_bias).to(torch.float)


class ScipyConv2d(Module):
    def __init__(self, filter_width, filter_height):
        super(ScipyConv2d, self).__init__()
        self.filter = Parameter(torch.randn(filter_width, filter_height))
        self.bias = Parameter(torch.randn(1, 1))

    def forward(self, input):
        return ScipyConv2dFunction.apply(input, self.filter, self.bias)
```

**Example usage:**

```
module = ScipyConv2d(3, 3)
print("Filter and bias: ", list(module.parameters()))
input = torch.randn(10, 10, requires_grad=True)
output = module(input)
print("Output from the convolution: ", output)
output.backward(torch.randn(8, 8))
print("Gradient for the input map: ", input.grad)
```

Out:

```
Filter and bias:  [Parameter containing:
tensor([[ 0.6684,  0.0100, -0.1243],
        [ 1.4697, -0.3951, -0.5101],
        [ 1.1163, -0.5926,  0.9089]], requires_grad=True), Parameter containing:
tensor([[-1.0792]], requires_grad=True)]
Output from the convolution:  tensor([[-2.3381e+00,  5.5492e-01, -5.0902e-01, -1.6106e+00, -8.3783e-01,
         -5.0754e-01, -2.9442e+00, -1.5587e+00],
        [-6.5555e-01, -1.4067e-01, -9.2076e-02, -3.7658e+00,  1.6718e+00,
         -3.4454e-01, -7.4749e+00,  5.2639e+00],
        [-3.6720e+00, -3.8789e+00,  1.0448e+00, -7.0677e+00,  2.9028e+00,
         -4.7983e-01, -1.0002e+00,  2.4696e+00],
        [-1.4890e+00, -1.5472e+00,  2.9120e-01, -4.4226e+00,  1.7955e+00,
         -1.8484e+00, -2.7094e+00,  3.4991e-01],
        [-2.0764e-01, -2.1445e+00,  3.6540e-03, -1.0906e+00,  8.5499e-01,
         -4.1489e+00, -4.7638e+00,  7.8557e-03],
        [-9.5946e-01, -1.8508e+00,  6.6503e-01,  5.8294e+00, -6.2242e-01,
         -3.1615e+00, -2.9003e+00, -6.7114e-01],
        [-2.5497e+00, -9.5294e-01, -1.8950e+00,  1.0024e+00, -1.2085e+00,
          1.2271e-01, -8.8431e-01, -2.3676e+00],
        [-4.8732e+00, -2.4313e+00,  1.0556e+00, -6.2132e-01, -1.1233e+00,
          1.2332e+00,  2.6107e+00, -1.3932e+00]],
       grad_fn=<ScipyConv2dFunctionBackward>)
Gradient for the input map:  tensor([[-0.5364,  0.1949, -1.2547, -0.7975,  0.3670,  0.2160,  0.7734,  0.3453,
         -0.1428, -0.0647],
        [-2.9300,  2.2440, -3.0885, -2.5313,  1.8106,  1.8809,  2.2640, -0.0219,
         -0.9311, -0.2499],
        [-4.1725,  5.1687, -3.9375, -3.4594,  1.0597,  2.4467,  2.9267, -1.4078,
          0.1878,  0.5457],
        [-2.5468,  4.8140, -3.1855, -0.9473,  2.4066,  0.2868,  1.1938,  0.9435,
          0.2971, -0.2741],
        [-0.5056,  2.7053,  3.6190, -3.2161,  4.7045, -0.2623,  2.3486,  0.6477,
          0.3025, -0.7271],
        [ 0.0622,  2.5652, -1.6121,  0.1239,  5.5655, -0.6287,  0.8288, -0.9299,
         -0.1968,  2.1928],
        [ 1.8391, -0.9487, -1.5444,  0.8200,  3.6750,  0.7193,  0.2008, -1.9123,
          0.9186, -1.0263],
        [ 0.8215, -0.4266, -2.5043, -0.9937,  0.7501,  0.1493, -0.0498,  2.4537,
          1.1479, -1.0694],
        [ 0.1044,  0.9116, -3.2716, -0.6076, -1.4100, -0.5589,  0.0876,  2.7136,
         -2.4977,  0.4421],
        [ 0.1106,  0.4925, -1.3191,  0.2790, -1.4345, -0.5084,  0.2468,  0.1644,
          0.1747,  0.7332]])
```

**Check the gradients:**

```
from torch.autograd.gradcheck import gradcheck

moduleConv = ScipyConv2d(3, 3)

input = [torch.randn(20, 20, dtype=torch.double, requires_grad=True)]
test = gradcheck(moduleConv, input, eps=1e-6, atol=1e-4)
print("Are the gradients correct: ", test)
```

Out:

```
Are the gradients correct:  True
```


# 相关

- [Creating Extensions Using numpy and scipy](https://pytorch.org/tutorials/advanced/numpy_extensions_tutorial.html)
