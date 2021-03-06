
#### TensorFlow实现经典卷 积神经网络

本章将介绍 4 种经典的卷积神经网络，分别是 AlexNet45、VGGNet46、Google Inception Net47和 ResNet48，这 4 种网络依照出现的先后顺序排列，深度和复杂度也依次递进。它 们分别获得了 ILSVRC （ImageNet Large Scale Visual Recognition Challenge） 49 比赛分 类项目的 2012 年冠军（top-5错误率 16.4%，使用额外数据可达到 15.3%, 8层神经网络）、 2014年亚军（top-5错误率 7.3%, 19层神经网络），2014年冠军（top-5错误率 6.7%, 22 层神经网络）和 2015 年的冠军（top-5错误率 3.57%, 152层神经网络）。这 4 个经典的网 络都在各自的年代率先使用了很多先进的卷积神经网络结构，对卷积网络乃至深度学习有 非常大的推动作用，也象征了卷积神经网络在 2012—2015这四年间的快速发展。如图 6-1 所示，ILSVRC的 top-5错误率在最近几年取得重大突破，而主要的突破点都是在深度学 习和卷积神经网络，成绩的大幅提升几乎都伴随着卷积神经网络的层数加深。而传统机器 学习算法目前在 ILSVRC 上已经难以追上深度学习的步伐了，以至于逐渐被称为浅层学 习（Shallow Learning）。目前在 ImageNet50 数据集上人眼能达到的错误率大概在 5.1%, 这还是经过了大量训练的专家能达到的成绩，一般人要区分 1000 种类型的图片是比较困 难的。而 ILSVRC 2015年冠军——152层 ResNet 的成绩达到错误率 3.57%，已经超过了

—「TensorFlow 实故

人眼，这说明卷积神经网络已经基本解决了 ImageNet数据集上的图片分类问题。

152 layers



22 layers 19 layers

IL5VRC15

RcsNet

IISVRC'14

GoogleNet

ILSVRC14

VGG



28.2

8 layers 8 layers    shallow

、IB**"* H—BL—層

ILSVRC13

ILSVRC12

AtexNet

ILSVRCll ILSVRC10



图 6-1历届 ILSVRC 比赛代表性模型的成绩及其神经网络深度

前面提到的计算机视觉比赛 ILSVRC 使用的数据都来自 ImageNet，如图 6-2所示。 ImageNet项目于 2007 年由斯坦福大学华人教授李飞飞创办，目标是收集大量带有标注信 息的图片数据供计算机视觉模型训练。ImageNet拥有 1500 万张标注过的高清图片，总共 拥有 22000 类，其中约有 100 万张标注了图片中主要物体的定位边框。ImageNet项目最 早的灵感来自于人类通过视觉学习世界的方式，如果假定儿童的眼睛是生物照相机，他们 平均每 200ms 就拍照一次（眼球转动一次的平均时间），那么 3 岁大时孩子就已经看过了 上亿张真实世界的照片，可以算得上是一个非常大的数据集。ImageNet项目下载了互联 网上近 10 亿张图片，使用亚马逊的土耳其机器人平台实现众包的标注过程，有来自世界 上 167 个国家的近 5 万名工作者帮忙一起筛选、标注。

图 6-2 ImageNet数据集图片示例



每年度的 ILSVRC 比赛数据集中大概拥有 120 万张图片，以及 1000 类的标注，是 ImageNet全部数据的一个子集。比赛一般采用 top-5和 top-1分类错误率作为模型性能的 评测指标，图 6-3所示为 AlexNet 识别 ILSVRC 数据集中图片的情况，每张图片下面是分 类预测得分最高的 5 个分类及其分值。

图 6-3 AlexNet识别 ILSVRC 数据集的 top-5分类



| mite    container ship motor scooter    leopard     |                                   |                                           |         |
| --------------------------------------------------- | --------------------------------- | ----------------------------------------- | ------- |
| mite black widow cockroachJ    tickstarfish         | container s^ip                    | motor scooter                             | leopard |
| _J    lifeboatamphibian fire boat drilling platform | go-kart moped bumper car golfcart | jaguar cheetah snovf leopard Egyptian cat |         |



##### 6.1 TensorFlow 实现 AlexNet

2012年，Hinton的学生 Alex Krizhevsky提出了深度卷积神经网络模型 AlexNet，它 可以算是 LeNet 的一种更深更宽的版本。AlexNet中包含了几个比较新的技术点，也首次 在 CNN 中成功应用了 ReLU、Dropout和 LRN 等 Trick。同时 AlexNet 也使用了 GPU进 行运算加速，作者开源了他们在 GPU 上训练卷积神经网络的 CUDA 代码。AlexNet包含 了 6亿 3000 万个连接，6000万个参数和 65 万个神经元，拥有 5 个卷积层，其中 3 个卷 积层后面连接了最大池化层，最后还有 3 个全连接层。AlexNet以显著的优势赢得了竞争 激烈的 ILSVRC 2012比赛，top-5的错误率降低至了 16.4%，相比第二名的成绩 26.2%错 误率有了巨大的提升。AlexNet可以说是神轻网络在低谷期后的第一次发声，确立了深度 学习(深度卷积网络)在计算机视觉的统治地位，同时也推动了深度学习在语音识别、自 然语言处理、强化学习等领域的拓展。

AlexNet将 LeNet 的思想发扬光大，把 CNN 的基本原理应用到了很深很宽的网络中。 AlexNet主要使用到的新技术点如下。

(1 )成功使用 ReLU 作为 CNN 的激活函数，并验证其效果在较深的网络超过了 Sigmoid，成功解决了 Sigmoid在网络较深时的梯度弥散问题。虽然 ReLU 激活函数在很

久之前就被提出了，但是直到 AlexNet 的出现才将其发扬光大。

(2)训练时使用 Dropout 随机忽略一部分神经元，以避免模型过拟合。Dropout虽有 单独的论文论述，但是 AlexNet 将其实用化，通过实践证实了它的效果。在 AlexNet 中主 要是最后几个全连接层使用了 Dropouto

(3 )在 CNN 中使用重叠的最大池化。此前 CNN 中普遍使用平均池化，AlexNet全部 使用最大池化，避免平均池化的模糊化效果。并且 AlexNet 中提出让步长比池化核的尺寸 小，这样池化层的输出之间会有重叠和覆盖，提升了特征的丰富性。

(4)    提出了 LRN层，对局部神经元的活动创建竞争机制，使得其中响应比较大的值 变得相对更大，并抑制其他反馈较小的神经元，增强了模型的泛化能力。

(5)    使用 CUDA 加速深度卷积网络的训练，利用 GPU 强大的并行计算能力，处理神 经网络训练时大量的矩阵运算。AlexNet使用了两块 GTX 580 GPU进行训练，单个 GTX 580 只有 3GB 显存，这限制了可训练的网络的最大规模。因此作者将 AlexNet 分布在两个 GPU 上，在每个 GPU 的显存中储存一半的神经元的参数。因为 GPU 之间通信方便’可以互相访 问显存，而不需要通过主机内存，所以同时使用多块 GPU 也是非常高效的。同时，AlexNet 的设计让 GPU 之间的通信只在网络的某些层进行，控制了通信的性能损耗。

(6)    数据增强，随机地从 256x256 的原始图像中截取 224x224 大小的区域(以及水 平翻转的镜像)，相当于增加了(256-224)\2=2048倍的数据量。如果没有数据增强，仅靠 原始的数据量，参数众多的 CNN 会陷入过拟合中，使用了数据增强后可以大大减轻过拟 合，提升泛化能力。进行预测时，则是取图片的四个角加中间共 5 个位置，并进行左右翻 转，一共获得 10 张图片，对他们进行预测并对 10 次结果求均值。同时，AlexNet论文中 提到了会对图像的 RGB 数据进行 PCA 处理，并对主成分做一个标准差为 0.1的高斯扰动, 增加一些噪声，这个 Trick 可以让错误率再下降 1%。

整个 AlexNet 有 8 个需要训练参数的厘不包括池化层和 LRN 层)，前 5 层为卷积层， 后 3 层为全连接层，如图 6-4所示。AlexNet最后一层是有 1000 类输出的 Softmax 层用作 分类。LRN层出现在第 1 个及第 2 个卷积层后，而最大池化层出现在两个 LRN 层及最 后一个卷积层后。ReLU激活函数则应用在这 8 层每一层的后面。因为 AlexNet 训练时使 用了两块 GPU，因此这个结构图中不少组件都被拆为了两部分。现在我们 GPU 的显存可 以放下全部模型参数，因此只考虑一块 GPU 的情况即可。

3    48

图 6-4 AlexNet的网络结构

AlexNet每层的超参数如图 6-5所示。其中输入的图片尺寸为 224x224，第一个卷积 层使用了较大的卷积核尺寸 11x11，步长为 4，有 96 个卷积核；紧接着一个 LRN 层；然 后是一个 3x3 的最大池化层，步长为 2。这之后的卷积核尺寸都比较小，都是 5x5 或者 3x3的大小，并且步长都为 1，即会扫描全图所有像素；而最大池化层依然保持为 3x3, 并且步长为 2。我们可以发现一个比较有意思的现象，在前几个卷积层，虽然计算量很大， 但参数量很小，都在 1M 左右甚至更小，只占 AlexNet 总参数量的很小一部分。这就是卷 积层有用的地方，可以通过较小的参数量提取有效的特征。而如果前几层直接使用全连接 层，那么参数量和计算量将成为天文数字。虽然每一个卷积层占整个网络的参数量的 1% 都不到，但是如果去掉任何一个卷积层，都会使网络的分类性能大幅地下降。

params    AlexNet    FLOPs

图 6-5 AlexNet每层的超参数及参数数量

因为使用 ImageNet 数据集训练一个完整的 AlexNet 耗时非常长，因此本节中 AlexNet

的实现将不涉及实际数据的训练。我们会建立一个完整的 AlexNet 卷积神经网络，然后对 它每个 batch 的前馈计算(forward )和反馈计算(backward)的速度进行测试。这里使用 随机图片数据来计算每轮前馈、反馈的平均耗时。有兴趣的读者，可以自行下载 ImageNet 数据并使用本书构建的 AlexNet 完成训练，并在测试集上进行测试。

首先导入几个接下来会用到的几个系统库，包括 datetime、math和 time，并载入 TensorFlow。本节代码主要来自 TensorFlow 的开源实现 51。

from datetime import datetime

import math

import time

import tensorflow as tf

这里设置 batch_size为 32，num_batches为 100，即总共测试 100 个 batch 的数据。

batch一 size=32

num_batches=100

定义一个用来显示网络每一层结构的函数 print_actications，展示每一个卷积层或池化 层输出 tensor 的尺寸。这个函数接受一个 tensor 作为输入，并显示其名称(t.op.name )和 tensor 尺寸(t.get_shape.as_list())。

def print_activations(t):

print(t.op.name) ' 、 t.get_shape().as_list())

接下来设计 AlexNet 的网络结构。我们先定义函数 inference，它接受 images 作为输 入，返回最后一层 pool5 (第 5 个池化层)及 parameters (AlexNet中所有需要训练的模型 参数)。这个 inference 函数将会很大，包括多个卷积和池化层，因此下面将拆为几个小段 分别讲解。

首先是第一个卷积层 convl，这里使用 TensorFlow 中的 name_scope，通过 with tf.name_scope('conv 1 *) as scope 可以将 scope 内生成的 Variable 自动命名为 convl/xxx，便 于区分不同卷积层之间的组件。然后定义第一个卷积层，和之前一样使用 tf.truncated_normal截断的正态分布函数(标准差为 0.1 )初始化卷积核的参数 kernel。卷 积核尺寸为 11x11，颜色通道为 3，卷积核数量为 64。准备好了 kernel，再使用 tf.nn.conv2d 对输入 images 完成卷积操作，我们将 strides 步长设置为 4x4 (即在图片上每 4x4 区域只 取样一次，横向间隔是 4，纵向间隔也为 4，每次取样的卷积核大小都为 11x11)，padding

k 1004

模式设为 SAME。将卷积层的 biases 全部初始化为 0，再使用 tf.nn.bias_add将 conv 和 biases 加起来，并使用激活函数 tf.nn.relu对结果进行非线性处理。最后使用 print_activations将 这一层最后输出的 tensor convl的结构打印出来，并将这一层可训练的参数 kernel、biases 添加到 parameters 中。

def inference(images): parameters =[]

with tf.name_scope('convl') as scope:

kernel = tf.Variable(tf.truncated_normal([11^ 11^ 33 64]

dtype=tf .float32j stddev=le-l) name='weights ') conv = tf.nn.conv2d(imageskernel, [1, 4_, 4, 1]padding= 'SAME') biases = tf.Variable(tf.constant(0.0^ shape=[64]dtype=tf.float32)j

trainable=True_, name= * biases') bias = tf.nn.bias_add(conVj biases) convl = tf.nn.relu(biaSj name=scope) print一 activations(convl) parameters += [kernel^ biases]

在第 1 个卷职层后再添加 LRN 层和最大池化层。先使用 tf.nn.lm对前面输出的 tensor convl 进行 LRN 处理，这里使用的 depth_radius 设为 4, bias 设为 1，alpha 为 0.001/9, beta 为 0.75，基本都是 AlexNet 的论文中的推荐值。不过目前除了 AlexNet，其他经典的卷积 神经网络模型基本都放弃了 LRN (主要是效果不明显)，而我们使用 LRN 也会让前馈、 反馈的速度大大下降(整体速度降到 1/3)，读者可以自主选择是否使用 LRN。下面使用 tf.nn.maxjpool对前面的输出 1ml 进行最大池化处理，这里的池化尺寸为 3x3，即将 3x3 大小的像素块降为 1x1 的像素，取样的步长为 2x2, padding模式设为 VALID，即•取样时 不能超过边框，不像 SAME 模式那样可以填充边界外的点。最后将输出结果 pooll 的结 构打印出来。

lrnl = tf.nn.lrn(convl, 4, bias=1.0, alpha=0.001/9) beta=0.75j name=,lrnl,) pooll = tf.nn.max_pool(lrnlJ ksize=[l, 3, 3, 1], strides=[l, 2, 1]_,

padding='VALID•, name='pooll1) print_activations(pooll)

接下来设计第 2 个卷积层，大部分步骤和第 1 个卷积层相同，只有几个参数不同。主

要区别在于我们的卷积核尺寸是 5x5，输入通道数（即上一层输出通道数，也就是上一层 卷积核数量）为 64，卷积核数量为 192。同时，卷积的步长也全部设为 1，即扫描全图像 素 O

with tf.name_scope('conv2') as scope:

kernel = tf.VariableCtf.tPuncated-normalQS, 5) 64, 192]j

dtype=tf.float32j stddev=le-l), name='weights') conv = tf .nn.conv2d(pooll, kernel) [1_, 1』1, 1], padding='SAME') biases = tf.Variable(tf.constant(0.0^ shape=[192],

dtype=tf.float32) trainable=True_, name='biases') bias = tf.nn.bias_add(conVj biases) conv2 = tf.nn.relu(biaSj name=scope) parameters += [kernel, biases]

print_activations(conv2)

接下来对第 2 个卷积层的输出 conv2 进行处理，同样是先做 LRN 处理，再进行最大 池化处理，参数和之前完全一样，这里就不再赘述了。

lrn2 = tf.nn.lrn(conv2, 4, bias=1.0, alpha=0.001/9, beta=0.75, name='lrn2') pool2 = tf.nn.max_pool(lrn2, ksize=[l_, 3, 1]strides=[l., 2, 23 1],

padding='VALID*} name=•pool2') print_activations(pool2)

下面创建第 3 个卷积层，基本结构和前面两个类似，也只是参数不同。这一层的卷积 核尺寸为 3x3，输入的通道数为 192，卷积核数量继续扩大为 384，同时卷积的步长全部 为 1，其他地方和前面保持一致。

with tf.name_scope('conv3') as scope:

kernel = tf.Variable(tf.truncated_normal([3j 3^ 192, 384],

dtype=tf.float32, stddev=le-l), name=*weights1) conv = tf.nn.conv2d(pool2, kernel^ [1, 1} 1, 1], padding='SAME') biases = tf.Variable(tf.constant(0.0^ shape=[384]

dtype=tf.float32)trainable=True, name='biases') bias = tf.nn.bias_add(conv, biases) conv3 = tf.nn.relu(bias, name=scope) parameters += [kernel， biases]

print_activations(conv3)

第 4 个卷积层和之前也类似，这一层的卷积核尺寸为 3x3，输入通道数为 384，但是 卷积核数量降为 256。

with tf.name_scope('conv4*) as scope:

kernel = tf.Variable(tf.truncated_normal([33} 384』 256]j

dtype=tf,float32j stddev=le-l)name='weights') conv = tf.nn.conv2d(conv3j kernel, [1, 1) 1) 1]) padding='SAME') biases = tf.Variable(tf.constant(0.0j shape=[256]j

dtype=tf.float32), trainable=TrueJ name='biases') bias = tf,nn.bias_add(conVj biases) conv4 = tf.nn.relu(biaSj name=scope) parameters += [kernel^ biases] print_activations(conv4)

最后的第 5 个卷积层同样是 3x3 大小的卷积核，输入通道数为 256，卷积核数量也为

256O

with tf.name_scope('conv5') as scope:

kernel = tf.Variable(tf.truncated_normal([3, 3, 256, 256],

dtype=tf.float32) stddev=le-l), name=■weights•) conv = tf.nn.conv2d(conv4j kernel, [1, 1, 1, 1]) padding='SAME') biases = tf.Variable(tf.constant(0.0j shape=[256]

dtype=tf.float32)trainable=True_, name=* biases') bias = tf.nn.bias_add(conv, biases) conv5 = tf.nn.relu(biasj name=scope) parameters += [kernel, biases] print_activations(conv5)

在第 5 个卷积层之后，还有一个最大池化层，这个池化层和前两个卷积层后的池化层 一致，最后我们返回这个池化层的输出 pool5。至此，inference函数就完成了，它可以创 建 AlexNet 的卷积部分。在正式使用 AlexNet 来训练或预测时，还需要添加 3 个全连接层， 隐含节点数分别为 4096、4096和 1000。由于最后 3 个全连接层的计算量很小，就没放到 计算速度评测中，他们对计算耗时的影响非常小。读者在正式使用 AlexNet 时需要自行添

加这 3 个全连接层，全连接层在 TensorFlow 中的实现方法在第 4 章已经讲解过，这里不 再赘述。

pool5 = tf.nn.max_pool(conv5ksize=[l_, 3^ 3, 1], strides=[l, 2, 2，1]^ padding='VALID'name=1pool5’)

print_activations(pool5)

return pool5_, parameters

接下来实现一个评估 AlexNet 每轮计算时间的函数 time_tensorflow_run。这个函数的 第一个输入是 TensorFlow 的 Session，第二个变量是需要评测的运算算子，第三个变量是 测试的名称。先定义预热轮数 num_StepS_bUrn_in=10，它的作用是给程序热身，头几轮迭 代有显存加载、cache命中等问题因此可以跳过，我们只考量 10 轮迭代之后的计算时间。 同时，也记录总时间 total_duration和平方和 total_duration_squared用以计算方差。

def time_tensorflow_run(session> target, info_string): num_steps_burn_in = 10 total_duration = 0.0 total_duration_squared = 0.0

我们进行 num_batches+num_steps_bum_in次迭代计算，使用 time.time()记录时间，每 次迭代通过 session.run(target)执行。在初始热身的 num_steps_bum_in次迭代后，每 10 轮 迭代显示当前迭代所需要的时间。同时每轮将 total_duration和 total_duration_squared累加， 以便后面计算每轮耗时的均值和标准差。

for i in range(num_batches + num_steps_burn_in): start_time = time.time()

_ = session.run(target)

duration = time.time() - start_time

if i >= num_steps_burn_in:

if not i % 10:

print('%s: step %d, duration = %.3f' %

(datetime.now()j i - num_steps_burn_in_, duration))

total_duration += duration

total_duration_squared += duration * duration

在循环结束后，计算每轮迭代的平均耗时 mn 和标准差 sd，最后将结果显示出来。这 样就完成了计算每轮迭代耗时的评测函数 time_tensorflow_runo

mn = total_duration / num_batches

vr = total_duration_squared / num_batches - mn * mn

sd = math.sqrt(vr)

print('%s: %s across %d steps, %.3f +/- %.3f sec / batch1 %

(datetime.now()info_string3 num_batches_, mn, sd))

接下来是主函数 run_benchmark。首先使用 with tf.Graph().as_default()定义默认的 Graph方便后面使用。如前面所说，我们并不使用 ImageNet 数据集来训练，只使用随机 图片数据测试前馈和反馈计算的耗时。我们使用 tf.random_normal函数构造正态分布(标 准差为 0.1)的随机 tensor，第一个维度是 batch_size，即每轮迭代的样本数，第二个和第 三个维度是图片的尺寸 image_SiZe=224，第四个维度是图片的颜色通道数。接下来，使用 前面定义的 inference 函数构建整个 AlexNet 网络，得到最后一个池化层的输出 pool5 和网 络中需要训练的参数的集合 parameterso 接下来，我们使用 tf.Session()创建新的 Session 并通过 tf.global_variables_initializer()初始化所有参数。

def run_benchmark():

with tf.Graph().as_default():

image_size = 224

images = tf .Variable(tf. random_normal( [batch_size_, image_size_, image一 size_, 3] dtype=tf.float32^ stddev=le-l))

pool5, parameters = inference(images)

init = tf.global_variables_initializer() sess = tf.Session() sess.run(init)

下面进行 AlexNet 的 forward 计算的评测，这里直接使用 time_tensorflow_run统计运 算时间，传入的 target 就是 pool5，即卷积网络最后一个池化层的输出。然后进行 backward 即训练过程的评测，这里和 forward 计算有些不同，我们需要给最后的输出 pool5 设置一 个优化目标 loss。我们使用 tf.nn.l2_loss计算 pool5 的 loss，再使用 tf.gradients求相对于 loss 的所有模型参数的梯度，这样就模拟了一个训练的过程。当然，训练时还有一个根据梯度 更新参数的过程，不过这个计算量很小，就不统计在评测程序里了。最后我们使用 time_tensorflow_run统计 backward 的运算时间，这里的 target 就是求整个网络梯度 gard 的操作。

time_tensorflow_run(sessj pools, "Forward")

objective = tf.nn.12_loss(pool5)

grad = tf.gradients(objective^ parameters)

time_tensorflow_run(sessj grad^ "Forward-backward")

最后执行主函数。

run_benchmark()

程序显示的结果有三段，首先是 AlexNet 的网络结构，可以看到我们定义的 5 个卷积 层中第 1 个、第 2 个和第 5 个卷积层后面还连接着池化层，另外每一层输出 tensor 的尺寸 也显示出来了。

| convlpooll   | [32, 56, 56, 64] |      |
| ------------ | ---------------- | ---- |
| [32, 27, 27, | 64]              |      |
| conv2        | [32, 27, 27,     | 192] |
| pool2        | [32, 13, 13,     | 192] |
| conv3        | [32、 13, 13,    | 384] |
| conv4        | [32、 13, 13,    | 256] |
| conv5        | [32, 13, 13,     | 256] |
| pool5        | [32j 6, 6, 256]  |      |

然后显示的是 forward 计算的时间。我们使用的 GPU 是 GTX 1080，软件环境包括 CUDA 8.0和 cuDNN 5.1。在有 LRN 层时每轮迭代时间大约为 0.026s；去除 LRN 层时每 轮迭代时间大约为 0.007s，运算时间有了大幅缩减，大约快了 3倍多。因为 LRN 层对最 终准确率的影响不是很大’所以读者可以自行考虑是否使用 LRN。

2016-12-10 21:08:31.851750: step 0, duration = 0.026

2016-12-10 21:08:32.109889: step 10、 duration = 0.026

| 2016-12-10 | 21:08:32.367558： | step    | 20,        | duration = | :0.026  |
| ---------- | ----------------- | ------- | ---------- | ---------- | ------- |
| 2016-12-10 | 21:08:32.625277： | step    | 30)        | duration = | :0.026  |
| 2016-12-10 | 21:08:32.884085： | step    | 40、       | duration = | :0.026  |
| 2016-12-10 | 21:08:33.141951： | step    | 50,        | duration = | :0.026  |
| 2016-12-10 | 21:08:33.400239： | step    | 60,        | duration = | :0.026  |
| 2016-12-10 | 21:08:33.658713： | step    | 70)        | duration = | ，0.026 |
| 2016-12-10 | 21:08:33.916780： | step    | 80,        | duration = | :0.026  |
| 2016-12-10 | 21:08:34.174840： | step    | 90,        | duration = | :0.026  |
| 2016-12-10 | 21:08:34.407022： | Forward | across 100 | steps,     |         |
| batch      |                   |         |            |            |         |

然后是显示的 backward 运算的时间。在使用 LRN 层时，每轮的迭代时间为 0.078s; 在去除 LRN 层后，每轮迭代时间约为 0.025s，速度也快了 3倍多。另外可以发现不论是 否有 LRN 层，我们 backward 运算的耗时大约是 forward 耗时的三倍。

| 2016-12-10 | 21:08:35.447963： | step 0j duration : | =0.078                     |
| ---------- | ----------------- | ------------------ | -------------------------- |
| 2016-12-10 | 21:08:36.225855： | step 10j duration  | =0.078                     |
| 2016-12-10 | 21:08:37.002277： | step 20j duration  | =0.077                     |
| 2016-12-10 | 21:08:37.777730： | step 30^ duration  | =0.078                     |
| 2016-12-10 | 21:08:38.555231： | step 40j duration  | =0.078                     |
| 2016-12-10 | 21:08:39.333113:  | step 5Qj duration  | =0.077                     |
| 2016-12-10 | 21:08:40.110716:  | step 60j duration  | =0.078                     |
| 2016-12-10 | 21:08:40.887492： | step 70j duration  | =0.078                     |
| 2016-12-10 | 21:08:41.664907： | step 80j duration  | =0.078                     |
| 2016-12-10 | 21:08:42.441733： | step 90? duration  | =0.078                     |
| 2016-12-10 | 21:08:43.139532:  | Forward-backward   | across 100 steps, 0.078 +/ |

00 sec / batch

CNN的训练过程（即 backward 计算熥常都比较耗时，而且不像预测过程（即 forward 计算），训练通常需要过很多遍数据，进行大量的迭代。因此应用 CNN 的主要瓶颈还是 在训练，用 CNN 做预测问题不大。目前 TensorFlow 已经支持在 iOS、Android系统中运 行，所以在手机上使用 CPU 进行人脸识别或图片分类已经非常方便了，并且响应速度也 很快。

至此，AlexNet的 TensorFlow 实现和运算时间评测就完成了。AlexNet为卷积神经网 络和深度学习正名，以绝对优势拿下 ILSVRC 2012冠军，弓 I 起了学术界的极大关注，为 复兴神经网络做出了很大贡献。AlexNet在 ILSVRC 数据集上可达到 16.4%的错误率（读 者可自行下载数据集测试，但注意 batchjize 可能要设为 1 才能复现论文中的结果），其 中用到的许多网络结构和 Trick 给深度学习的发展带来了深刻的影响。当然，我们也不能 忽视 ImageNet 数据集给深度学习带来的贡献。训练深度卷积神经网络，必须拥有一个像 ImageNet这样超大的数据集才能避免过拟合，发挥深度学习的优势。可以说，传统机器 学习模型适合学习一个小型数据集，但是对于大型数据集，我们需要有更大学习容量 （Learning Capacity ）的模型，即深度学习模型。

在 AlexNet 发表在 NIPS 时，Hinton曾说“如果你没有参加过之前十几年的 NIPS, 那没关系，因为直到今年神经网络才真正开始生效。”当时有很多与会的教授表示不能接 受神经网络，他们认为深度学习是一个黑箱模型，不可解释，有超过 6000 万的参数，在 ILSVRC上的成绩可能只是某种过拟合，对计算机视觉贡献不大。然而之前使用传统方法 获得过 ILSVRC 比赛冠军的 NEC Labs的模型参数量也不少。他们当时使用 sparse SIFT 加 Pyramid Pooling提取特征，然后使用 SVM 进行分类，但是他们的 SVM 模型参数也有 超过 1 亿 6000 万，远比 AlexNet 的参数多。因此说 CNN 参数多所以没有价值的观点是站 不住脚的。深度学习的参数并不一定比传统机器学习模型多，尤其卷积层使用的参数量其 实很少，但是其抽取特征的能力是非常强的，这也是 CNN 之所以有效的原因。

K 108

##### 6.2 TensorFlow 实现 VGGNet

VGGNet 是牛津大学计算机视觉组（Visual Geometry Group）和 Google DeepMind 公司的研究员一起研发的的深度卷积神经网络。VGGNet探索了卷积神经网络的深度与其 性能之间的关系，通过反复堆叠 3x3 的小型卷积核和 2x2 的最大池化层，VGGNet成功地 构筑了 16~19层深的卷积神经网络。VGGNet相比之前 state-of-the-art的网络结南，错误 率大幅下降，并取得了 ILSVRC 2014比赛分类项目的第 2 名和定位项目的第 1 名。同时 VGGNet的拓展性很强，迁移到其他图片数据上的泛化性非常好。VGGNet的结构非常简 洁，整个网络都使用了同样大小的卷积核尺寸（3x3）和最大池化尺寸（2x2）。到目前为 止，VGGNet依然经常被用来提取图像特征。VGGNet训练后的模型参数在其官方网站上 开源了，可用来在 domain specific的图像分类任务上进行再训练（相当于提供了非常好 的初始化权重），因此被用在了很多地方。

VGGNet论文中全部使用了 3x3的卷积核和 2x2 的池化核，通过不断加深网络结构来 提升性能。图 6-6所示为 VGGNet 各级别的网络结构图，图 6-7所示为每一级别的参数量， 从 11 层的网络一直到 19 层的网络都有详尽的性能测试。虽然从 A 到 E 每一级网络逐渐 变深，但是网络的参数量并没有増长很多，这是因为参数量主要都消耗在最后 3 个全连接 层。前面的卷职部分虽然很深，但是消耗的参数量不大，不过训练比较耗时的部分依然是 卷职，因其计算量比较大。这其中的 D、E也就是我们常说的 VGGNet-16和 VGGNet-19。

C很有意思，相比 B 多了几个 1x1 的卷积层，1x1卷积的意义主要在于线性变换，而输入 通道数和输出通道数不变，没有发生降维。

| ConvNet Configuration       |                    |                    |                             |                             |                                      |
| --------------------------- | ------------------ | ------------------ | --------------------------- | --------------------------- | ------------------------------------ |
| A                           | A-LRN              | B                  | C                           | D                           | E                                    |
| ll weightlayers             | 11 weightlayers    | 13 weightlayers    | 16 weightlayers             | 16 weight layers            | 19 weight layers                     |
| input (224 x 224 RGB image) |                    |                    |                             |                             |                                      |
| conv3-64                    | conv3-64LRN        | conv3-64conv3-64   | conv3-64conv3-64            | conv3-64conv3-64            | conv3-64conv3-64                     |
| maxpool                     |                    |                    |                             |                             |                                      |
| conv3-128                   | conv3-128          | conv3-128conv3-128 | conv3-128conv3-128          | conv3-128conv3-128          | conv3-128conv3-128                   |
| maxpool                     |                    |                    |                             |                             |                                      |
| conv3-256conv3-256          | conv3-256conv3-256 | conv3-256conv3-256 | conv3-256conv3-256convl-256 | conv3-256conv3-256conv3-256 | conv3-256conv3-256conv3-256conv3-256 |
| maxpool                     |                    |                    |                             |                             |                                      |
| conv3-5l2conv3-512          | conv3-512conv3-512 | conv3-512conv3-512 | conv3-512conv3-512convl-512 | conv3-512conv3-512conv3-512 | conv3-512conv3-512conv3-512conv3-512 |
| maxpool                     |                    |                    |                             |                             |                                      |
| conv3-512conv3-512          | conv3-512conv3-512 | conv3-512conv3-512 | conv3-512conv3-512convl-512 | conv3-512conv3-512conv3-512 | conv3-512conv3-512conv3-512conv3-512 |
| maxpool                     |                    |                    |                             |                             |                                      |
| FC-4096                     |                    |                    |                             |                             |                                      |
| FC-4096                     |                    |                    |                             |                             |                                      |
| FC-1000                     |                    |                    |                             |                             |                                      |
| soft-max                    |                    |                    |                             |                             |                                      |

图 6-6 VGGNet各级别网络结构图

| Network              | A,A-LRN | B    | C    | D    | E    |
| -------------------- | ------- | ---- | ---- | ---- | ---- |
| Number of parameters | 133     | 133  | 134  | 138  | 144  |

图 6-7 VGGNet各级别网络参数量（单位为百万）

VGGNet拥有 5 段卷积，毎一段内有 2~3个卷积层，同时每段尾部会连接一个最大池

化层用来缩小图片尺寸。毎段内的卷积核数量一样，越靠后的段的卷积核数量越多：64-128 - 256 - 512 - 512。其中经常出现多个完全一样的 3x3 的卷积层堆叠在一起的情况， 这其实是非常有用的设计。如图 6-8所示，两个 3x3 的卷积层串联相当于 1 个 5x5 的卷积 层，即一个像素会跟周围 5x5 的像素产生关联，可以说感受野大小为 5x5。而 3 个 3x3 的卷积层串联的效果则相当于 1 个 7x7 的卷积层。除此之外，3个串联的 3x3 的卷积层, 拥有比 1 个 7x7 的卷积层更少的参数量，只有后者的^ = 55%。最重要的是，3个 3x3 的卷积层拥有比 1 个 7x7 的卷积层更多的非线性变拠前者可以使用三次 ReLU 激活函数， 而后者只有一次），使得 CNN 对特征的学习能力更强。

图 6-8两个串联 3x3 的卷积层功能类似于一个 5x5 的卷积层

VGGNet在训练时有一个小技巧，先训练级别 A 的简单网络，再复用 A 网络的权重 来初始化后面的几个复杂模型，这样训练收敛的速度更快。在预测时，VGG采用 Multi-Scale的方法，将图像 scale 到一个尺寸 Q，并将图片输入卷积网络计算。然后在最 后一个卷积层使用滑窗的方式进行分类预测，将不同窗口的分类结果平均，再将不同尺寸 Q的结果平均得到最后结果，这样可提高图片数据的利用率并提升预测准确率。同时在训 练中，VGGNet还使用了 Multi-Scale的方法做数据增强，将原始图像缩放到不同尺寸 S， 然后再随机裁切 224x224 的图片，这样能增加很多数据量，对于防止模型过拟合有很不 错的效果。实践中，作者令 S 在［256,512］这个区间内取值，使用 Multi-Scale获得多个版 本的数据，并将多个版本的数据合在一起进行训练。图 6-9所示为 VGGNet 使用 Multi-Scale 训练时得到的结果，可以看到 D 和 E 都可以达到 7.5%的错误率。最终提交到 ILSVRC 2014 的版本是仅使用 Single-Scale的 6 个不同等级的网络与 Multi-Scale的 D 网络的融合，达到 了 7.3%的错误率。不过比赛结束后作者发现只融合 Multi-Scale的 D 和 E 可以达到更好的 效果，错误率达到 7.0%，再使用其他优化策略最终错误率可达到 6.8%左右，非常接近同 年的冠军 Google Inceptin Net。同时，作者在对比各级网络时总结出了以下几个观点。

(1 ) LRN层作用不大。

(2)    越深的网络效果越好。

(3)    1x1的卷积也是很有效的，但是没有 3x3 的卷积好，大一些的卷积核可以学习更 大的空间特征。

| ConvNet config. (Tabic 1) | smallest image side | top-1 val. error (%) | top-5 val. error (%) |      |
| ------------------------- | ------------------- | -------------------- | -------------------- | ---- |
| train (5)                 | test (Q)            |                      |                      |      |
| B                         | 256                 | 224,256,288          | 28.2                 | 9.6  |
| C                         | 256                 | 224,256,288          | 27.7                 | 9.2  |
| 384                       | 352,384,416         | 27.8                 | 9.2                  |      |
| [256; 512J                | 256,384,512         | -2^3-                | 8.2                  |      |
| D                         | 256                 | 224,256,288          | 26.6                 | 8.6  |
| 384                       | 352,384,416         | K5-―                 | 8.6                  |      |
| [256;512j                 | 256,384,512         |                      | 7.5                  |      |
| E                         | 256                 | 224,256,288          | 26.9                 | 8.7  |
| 384                       | 352,384,416         | 26.7                 | 8.6                  |      |
| [256; 512]                | 256,384,512         | 24.8                 | 7.5                  |      |

图 6-9各级别 VGGNet 在使用 Multi-Scale训练时的 top-5错误率

VGGNet训练时使用了 4块 Geforce GTX Titan GPU并行计算，速度比单块 GPU 快 3.75倍，几乎没有太多性能损耗。但是，每个网络耗时 2〜3周才可以训练完。因此我们 这里不直接使用 ImageNet 数据训练一个 VGGNet，而是采用跟 AlexNet—样的方式：构 造出 VGGNet 网络结构，并评测其 forward (inference )耗时和 backward (training )耗时。

下面就开始实现 VGGNet-16，也就是上面的版本 D，其他版本读者可以仿照本节的 代码自行修改并实现，难度不大。首先，我们载入几个系统库和 TensorFlow。本节代码 主要来自 tensorflow-vgg的开源实现 52。

from datetime import datetime

import math    .

import time

import tensorflow as tf

VGGNet-16包含很多层的卷职，因此我们先写一个函数 conv_Op，用来创建卷积层并 把本层的参数存入参数列表。先来看 conv_op函数的输入，input_op是输入的 tensor, name 是这一层的名称，kh是 kernel height即卷积核的高，kw是 kernel width即卷职核的宽， n_out是卷积核数量即输出通道数，dh是步长的高，dw是步长的宽，p是参数列表。下面 使用 get_shape()[-l].value获取输入 input_op的通道数，比如输入图片的尺寸 224x224x3 中最后的那个 3。然后使用 tf.name_scope(name)设置 scope。我们的 kemel(即卷积核参数) 使用 tf.get_variable创建，其中 shape 就是[kh，kw, n_in，n_out]即[卷积核的高，卷积核的 宽，输入通道数，输出通道数]，同时使用 tf.contrib.layers.xavier_initializer_conv2d()做参 数初始化。Xavier初始化方法我们在第 4 章实现过，其原理也讲解过，在此不做赘述。

def conv_op(input_opname, kh, kw_, n_outj dhj dw_, p): n_in = input_op.get_shape()[-1].value

with tf.name一 scope(name) as scope:

kernel = tf.get_variable(scope+"w"4

shape=[khJ kw, n_in_, n_out]dtype=tf.float32j initializer=tf.contrib.layers.xavier_initializer_conv2d())

接着使用 tf.nn.conv2d对 input_op进行卷职处理，卷积核即为 kernel，步长是 dhxdw, padding模式设为 SAME。biases使用 tf.constant赋值为 0，再使用 tf.Variable将其转成可 训练的参数。我们使用 tf.nn.bias_add将卷积结果 conv 与 bias 相加，再使用 tf.nn.relu对其 进行非线性处理得到 activation。最后将创建卷积层时用到的参数 kernel 和 biases 添加进 参数列表 p，并将卷积层的输出 activation 作为函数结果返回。

conv = tf.nn.conv2d(input_opj kernel, (1^ dh, dw_, 1), padding:’SAME')

bias_init_val = tf.constant(0.04 shape=[n_out], dtype=tf.float32) biases = tf.Variable(bias_init_valj trainable=True, name='b') z = tf.nn.bias_add(conVj biases)

activation = tf.nn.relu(z^ name=scope) p += [kernel, biases] return activation

下面定义全连接层的创建函数 fc_opD —样是先获取输入 input_op的通道数，然后使 用 tf.get_Variable创建全连接层的参数，只不过参数的维度只有两个，第一个维度为输入 的通道数 n_in，第二个维度为输出的通道数 n_out。同样，参数初始化方法也使用 xavier_initializero这里 biases 不再初始化为 0，而是赋予一个较小的值 0.1以避免 dead neuron。然后使用 tf.nn.relu_layer对输入变量 input_op与 kernel 做矩阵乘法并加上 biases, 再做 ReLU 非线性变换得到 activation。最后将这个全连接层用到参数 kernel、biases添加

到参数列表 p，并将 activation 作为函数结果返回。

def fc_op(input_opj name^ n_out, p):

n_in = input_op.get_shape()[-1].value

with tf.name_scope(name) as scope:

kernel = tf.get_variable(scope+"w",

shape=[n_in_, n_out]dtype=tf.float32j initializer=tf.contrib.layers.xavier_initializer())

biases = tf.Variable(tf.constant(0.1^ shape=[n_out],

dtype= tf.float32)j name='b')

activation = tf.nn.relu_layer(input_opj kernel, biases, name= scope) p += [kernel^ biases] return activation

再定义最大池化层的创建函数 mpool_op。这里直接使用 tf.nn.max_pool，输入即为 input_op，池化尺寸为 khxkw，步长是 dhxdw, padding模式设为 SAME。

def mpool_op(input_op_, name_, kh_, kWj dhj dw): return tf.nn.max_pool(input_opj

ksize=[lj khj kw_, l]j strides=[lj dhj dw3 1]) padding='SAME*} name=name)

完成了卷积层、全连接层和最大池化层的创建函数，接下来就开始创建 VGGNet-16 的网络结构。VGGNet-16主要分为 6 个部分，前 5 段为卷积网络，最后一段是全连接网 络。我们定义创建 VGGNet-16网络结构的函数 inference_op，输入有 input_op和 keep_prob, 这里的 keep_prob是控制 dropout 比率的一个 placeholder。第一步先初始化参数列表 p。然 后创建第一段卷积网络，这一段正如图 6-6中的网络结构，由两个卷积层和一个最大池化 层构成。我们使用前面写好的函数 conv_op、mpool_op来创建他们。这两个卷积层的卷积 核的大小都是 3x3，同时卷积核数量（输出通道数）均为 64，步长为 1x1，全像素扫描。 第一个卷积层的输入 input_op的尺寸为 224x224x3，输出尺寸为 224x224x64；而第二个 卷积层的输入输出尺寸均为 224x224x64。卷积层后的最大池化层则是一个标准的 2x2 的 最大池化，将输出结果尺寸变为了 112x112x64。

def inference_op(input_opj keep_prob):

P =[]

convl_l = conv_op(input_op, name="convl_l", kh=3) kw=3_, n_out=64, dh=l, dw=l_, p=p)

convl_2 = conv_op(convl_l, name=,,convl_2"J kh=3, kw=3, n_out=64, dh=l) dw=lj p=p)

pooll = mpool_op(convl_2j name="pooll,\ kh=2, kw=2, dw=2) dh=2)

第二段卷积网络和第一段非常类似，同样是两个卷积层加一个最大池化层，两个卷积 层的卷积核尺寸也是 3x3，但是输出通道数变为 128，是以前的两倍。最大池化层则和前 面保持一致，因此这一段卷积网络的输出尺寸变为 56x56x128。

conv2_l = conv_op(poollj name="conv2_l"j kh=3) kw=3, n_out=128, dh=l_, dw=lj p=p)

conv2_2 = conv_op(conv2_l_, name="conv2_2", kh=3j kw=3_, n_out=128, dh=l, dw=lj p=p)

pool2 = mpool_op(conv2_2name="pool2"kh=2_, kw=2_, dh=2_, dw=2)

接下来是第三段卷积网络，这里有 3 个卷积层和 1 个最大池化层。3个卷积层的卷积 核大小依然是 3x3，但是输出通道数增长为 256，而最大池化层保持不变，因此这一段卷 积网络的输出尺寸是 28x28x256。

conv3_l = conv_op(pool2_, name="conv3_l,,J kh=3j kw=3^ n_out=256j dh=l, dw=l_, p=p)

conv3_2 = conv_op(conv3_lj name="conv3_2"J kh=3_, kw=3_, n_out=256^ dh=lj dw=l, p=p)

conv3_3 = conv_op(conv3_2_, name="conv3_3"j kh=3, kw=3j n_out=256_, dh=l_, dw=lj p=p)

pool3 = mpool_op(conv3_3? name="pool3"kh=2_, kw=2_, dh=2j dw=2)

第四段卷积网络也是 3 个卷积层加 1 个最大池化层。读者可能已经发现规律了，到目 前为止，VGGNet-16的每一段卷积网络都会将图像的边长缩小一半，但是将卷积输出通 道数翻倍。这样图像面积缩小到 1/4，输出通道数变为 2 倍，因此输出 tensor 的总尺寸每 次缩小一半。这一层就是将卷积输出通道数増加到 512，但是通过最大池化将图片缩小为

14x14。

conv4_l = conv_op(pool3? name="conv4_l"kh=3j kw=33 n_out=512, dh=lj dw=lj p=p)

conv4_2 = conv_op(conv4_lj name="conv4_2"j kh=3』 kw=3, n_out=512, dh=l) dw=l4 p=p)

conv4_3 = conv_op(conv4_2, name="conv4_3"kh=3) kw=3, n_out=512, dh=l_, dw=lj p=p)

pool4 = mpool_op(conv4_3j name="pool4", kh=2) kw=2j dh=2_, dw=2)

最后一段卷积网络有所变化’这里卷积输出的通道数不再增加，继续维持在 512。最 后一段卷积网络同样是 3 个卷积层加一个最大池化层，卷积核尺寸为 3x3，步长为 1x1， 池化层尺寸为 2x2，步长为 2x2。因此到这里输出的尺寸变为 7x7x512。

conv5_l = conv_op(pool4, name="conv5_l\ kh=3_, kw=3, n_out= 512, dh=l_, dw=l? p=p)

conv5_2 = conv_op(conv5_lname="conv5_2", kh=3j kw=3, n_out= 512_, dh=lj dw=l, p=p)

conv5_3 = conv_op(conv5_2, name="conv5_3"kh=3, kw=3_, n_out=512^ dh=l^ dw=l_, p=p)

pool5 = mpool_op(conv5_3, name="pool5"j kh=2, kw=2, dw=2_, dh=2)

我们将第 5 段卷积网络的输出结果进行扁平化，使用 tf.reshape函数将每个样本化为 长度为 7x7x512=25088的一维向量。

shp = pool5.get_shape()

flattened—shape = shp[l].value * shp[2].value * shp[3].value reshl =    . reshape (pools [-1、f lattened_shape] name="reshl")

然后连接一个隐含节点数为 4096 的全连接层，激活函数为 ReLU。然后连接一个 Dropout层，在训练时节点保留率为 0.5，预测时为 1.0。

fc6 = fc_op(reshlj name=',fc6"J n_out=4096j p=p) fc6_drop = tf. nn.dropout(fc6j keep—probj name=,,fc6_drop")

接下来是一个和前面一样的全连接层，之后同样连接一个 Dropout 层。 fc7 = fc_op(fc6_dropj name="fc7' n_out=4096j p=p)

fc7_drop = tf. nn. dropout(fc7? keep_prob? name=,,fc7_drop")

最后连接一个有 1000 个输出节点的全连接层，并使用 Softmax 进行处理得到分类输 出概率。这里使用 tf.argmax求输出概率最大的类另 U。最后将 fc8、softmax、predictions和 参数列表 p—起返回。到此为止，VGGNet-16的网络结构就全部构建完成了。

fc8 = fc_op(fc7_dropJ name="fc8", n_out=1000_, p=p) softmax = tf.nn.softmax(fc8) predictions = tf.argmax(softmaxj 1) return predictions, softmax^ fc8^ p

我们的评测函数 time_tensorflow_run()和前面 AlexNet 中的非常相似，只有一点区别： 我们在 session.run()方法中引入了 feed_dict，方便后面传入 keep_prob来控制 Dropout 层的 保留比率。

def time_tensorflow_run(sessiontarget』 feedj info_string): num_steps_burn_in = 10 total_duration = 0.0 total_duration_squared = 0.0

for i in range(num_batches + num_steps_burn_in): start_time = time.time()

_ = session.run(targetfeed_dict=feed) duration = time.time() - start_time if i >= num_steps_burn_in:

if not i % 10:

print (*%s: step %dj duration = %.3f' %

(datetime.now(), i - num_steps_burn_inj duration))

total_duration += duration

total_duration_squared += duration * duration mn = total_duration / num_batches vr = total_duration_squared / num_batches - mn * mn sd = math.sqrt(vr)

print ('%s: %s across %d steps) %.3f +/- %.3f sec / batch' %

(datetime.nowOj info_string_, num_batches_, mn, sd))

下面定义评测的主函数 run_benchmark，我们的目标依然是仅评测 forward 和 backward 的运算性能，并不进行实质的训练和预测。首先是生成尺寸为 224x224 的随机图片，方 法与 AlexNet 中一样，通过 tf.random_normal函数生成标准差为 0.1的正态分布的随机数 0

def run_benchmark():

with tf.Graph().as_default():

image_size = 224

images = tf .Variable(tf. random_normal( [batch_size_, image_sizej image_size_, 3]』 dtype=tf.float32j stddev=le-l))

接下来，创建 keep_prob的 placeholder，并调用 inference_op函数构建 VGGNet-16白勺 网络结构，获得 predictions、softmax、fc8和参数列表 po

keep_prob = tf.placeholder(tf.float32)

predictions^ softmax^ fc8_, p = inference_op(images_, keep_prob)

然后创建 Session 并初始化全局参数。

init = tf.global_variables_initializer() sess = tf.Session() sess.run(init)

我们通过将 keep_prob设为 1.0来执行预测，并使用 time_tensorflow_run评测 forward 运算时间。再计算 VGGNet-16最后的全连接层的输出 fc8 的 12 loss，并使用 tf.gradients 求相对于这个 loss 的所有模型参数的梯度。最后使用 time_tensorflow_run评测 backward 运算时间，这里 target 为求解梯度的操作 grad, keep_prob为 0.5。

time^ensorflow^unCsesSj predictions, {keep_prob:1.0}j "Forward") objective = tf.nn.l2_loss(fc8) grad = tf.gradients(objectivep)

time_tensopflow_run(sessj grad3 {keep_prob:0.5}_, "Forward-backwardn)

我们设置 batch_size为 32，因为 VGGNet-16的模型体积比较大，如果使用较大的 batch_size，GPU显存会不够用。最后执行评测的主函数 mn_benchmark()，测试 VGGNet-16 在 TensorFlow 上的 forward 和 backward 耗时。

batch_size=32

num_batches=100

run_benchmark()

forward计算时平均每个 batch 的耗时为 0.152s，相比于同样 batch size的 AlexNet 的 0.026s (如果无 LRN 则是 0.007s )慢 6 倍。这说明 VGGNet-16的计算复杂度相比 AlexNet 确实高了很多，不过同样也带来了很大的准确率提升。

2016-12-01 17:13:31.761549: step 0, duration = 0.151

2016-12-01 17:13:46.795223: Forward across 100 steps) 0.152 +/- 0.002 sec / batch

而 backward 求解梯度时，每个 batch 的平均耗时达到 0.617s，相比于 AlexNet 的 0.078s 也局了很多。

| 2016-12-01 17:13:57.078991： | step     | 03       | duration = | =0.613 |
| ---------------------------- | -------- | -------- | ---------- | ------ |
| 2016-12-01 17:14:03.241287： | step     | 10)      | duration   | =0.621 |
| 2016-12-01 17:14:09.398178： | step     | 20,      | duration   | =0.616 |
| 2016-12-01 17:14:15.555161： | step     | 30,      | duration   | =0.617 |
| 2016-12-01 17:14:21.713196： | step     | 40,      | duration   | =0.614 |
| 2016-12-01 17:14:27.879734： | step     | 50)      | duration   | =0.614 |
| 2016-12-01 17:14:34.044447： | step     | 60、     | duration   | =0.614 |
| 2016-12-01 17:14:40.204176： | step     | 70)      | duration   | =0.619 |
| 2016-12-01 17:14:46.373392： | step     | 80,      | duration   | =0.615 |
| 2016-12-01 17:14:52.550798： | step     | 90,      | duration   | =0.620 |
| 2016-12-01 17:14:58.123548： | Forward- | backward | across 100 |        |

steps, 0.617 +/- 0.0

02 sec / batch

至此 VGGNet-16的实现和评狈蹴完成了。VGG系列的卷积神经网络在 ILSVRC 2014 比赛中最终达到了 7.3%的错误率，相比 AlexNet 进步非常大，读者可以使用 ImageNet 数 据集复现其结果。VGGNet的模型参数虽然比 AlexNet 多，但反而只需要较少的迭代次数 就可以收敛，主要原因是更深的网络和更小的卷积核带来的隐式的正则化效果。VGGNet 凭借其相对不算很高的复杂度和优秀的分类性能/成为了一代经典的卷积神经网络，直到 现在依然被应用在很多地方。

##### 6.3 TensorFlow 实现 Google Inception Net

Google Inception Net首次出现在 ILSVRC 2014的比赛中（和 VGGNet 同年），就以 较大优势取得了第一名。那届比赛中的 Inception Net通常被称为 Inception VI，它最大的 特点是控制了计算量和参数量的同时，获得了非常好的分类性能一 top-5错误率 6.67%， 只有 AlexNet 的一半不到。Inception VI有 22 层深，比 AlexNet 的 8 层或者 VGGNet 的 19层还要更深。但其计算量只有 15 亿次浮点运算，同时只有 500 万的参数量，仅为 AlexNet 参数量（6000万）的 1/12，却可以达到远胜于 AlexNet 的准确率，可以说是非常优秀并 且非常实用的模型。Inception VI降低参数量的目的有两点，第一，参数越多模型越庞大， 需要供模型学习的数据量就越大，而目前高质量的数据非常昂贵；第二，参数越多，耗费 的计算资源也会更大。Inception VI参数少但效果好的原因除了模型层数更深、表达能力 更强外，还有两点：一是去除了最后的全连接层，用全局平均池化层（即将图片尺寸变为 1x1 ）来取代它。全连接层几乎占据了 AlexNet或 VGGNet 中 90%的参数量，而且会引起 过拟合，去除全连接层后模型训练更快并且减轻了过拟合。用全局平均池化层取代全连接 层的做法借鉴了 Network In Network （以下简称 NIN）论文。二是 Inception VI中精心设 计的 Inception Module提高了参数的利用效率，其结构如图 6-10所示。这一部分也借鉴 了 NIN的思想，形象的解释就是 Inception Module本身如同大网络中的一个小网络，其 结构可以反复堆叠在一起形成大网络。不过 Inception VI比 NIN 更进一步的是增加了分 支网络，NIN则主要是级联的卷积层和 MLPConv 层。一般来说卷积层要提升表达能力， 主要依靠增加输出通道数，但副作用是计算量增大和过拟合。每一个输出通道对应一个滤 波器，同一个滤波器共享参数，只能提取一类特征，因此一个输出通道只能做一种特征处 理。而 NIN 中的 MLPConv 则拥有更强大的能力，允许在输出通道之间组合信息，因此 效果明显。可以说，MLPConv基本等效于普通卷积层后再连接 1x1 的卷积和 ReLU 激活

函数。

我们再来看 Inception Module的基本结构，其中有 4 个分支：第一个分支对输入进行 1x1的卷积，这其实也是 NIN 中提出的一个重要结构。1x1的卷积是一个非常优秀的结构， 它可以跨通道组织信息，提高网络的表达能力，同时可以对输出通道升维和降维。可以看 到 Inception Module的 4 个分支都用到了 1x1卷积，来进行低成本（计算量比 3x3 小很多） 的跨通道的特征变换。第二个分支先使用了 1x1卷积，然后连接 3x3 卷积，相当于进行 了两次特征变换。第三个分支类似，先是 1x1 的卷积，然后连接 5x5 卷积。最后一个分 支则是 3x3 最大池化后直接使用 1x1 卷积。我们可以发现，有的分支只使用 1x1 卷积， 有的分支使用了其他尺寸的卷积时也会再使用 1x1 卷积，这是因为 1x1 卷积的性价比很 高，用很小的计算量就能增加一层特征变换和非线性化。Inception Module的 4 个分支在 最后通过一个聚合操作合并（在输出通道数这个维度上聚合）。Inception Module中包含 了 3种不同尺寸的卷积和 1 个最大池化，增加了网络对不同尺度的适应性，这一部分和 Multi-Scale的思想类似。早期计算机视觉的研究中，受灵长类神经视觉系统的启发，Serre 使用不同尺寸的 Gabor 滤波器处理不同尺寸的图片，Inception VI借鉴了这种思想。 Inception VI的论文中指出，Inception Module可以让网络的深度和宽度高效率地扩充， 提升准确率且不致于过拟合。

图 6-10 Inception Module 结构图

人脑神经元的连接是稀疏的，因此研究者认为大型神经网络的合理的连接方式应该也 是稀疏的。稀疏结构是非常适合神经网络的一种结构，尤其是对非常大型、非常深的神经 网络，可以减轻过拟合并降低计算量，例如卷积神经网络就是稀疏的连接。Inception Net 的主要目标就是找到最优的稀疏结构单元（即 Inception Module ），论文中提到其稀疏结 构基于 Hebbian 原理，这里简单解释一下 Hebbian 原理：神经反射活动的持续与重复会导

致神经元连接稳定性的持久提升，当两个神经元细胞 A 和 B 距离很近，并且 A 参与了对 B重复、持续的兴奋，那么某些代谢变化会导致 A 将作为能使 B 兴奋的细胞。总结一下 即“一起发射的神经元会连在一起”(Cells that fire together, wire together )，学习过程中 的刺激会使神经元间的突触强度増加。受 Hebbian 原理启发，另一篇文章 Provable Bounds for Learning Some Deep    似提出，如果数据集的概率分布可以被一个很大很

稀疏的神经网络所表达，那么构筑这个网络的最佳方法是逐层构筑网络：将上一层高度相 关(correlated )的节点聚类，并将聚类出来的每一个小簇(cluster )连接到一起，如图 6-11 所示。这个相关性高的节点应该被连接在一起的结论，即是从神经网络的角度对 Hebbian 原理有效性的证明。

图 6-11将高度相关的节点连接在一起，形成稀疏网络

因此一个“好”的稀疏结构，应该是符合 Hebbian 原理的，我们应该把相关性高的一 簇神经元节点连接在一起。在普通的数据集中，这可能需要对神经元节点聚类，但是在图 片数据中，天然的就是临近区域的数据相关性高，因此相邻的像素点被卷积操作连接在一 起。而我们可能有多个卷积核，在同一空间位置但在不同通道的卷积核的输出结果相关性 极高。因此，一个 1x1 的卷积就可以很自然地把这些相关性很高的、在同一个空间位置 但是不同通道的特征连接在一起，这就是为什么 1x1 卷积这么频繁地被应用到 Inception Net中的原因。1x1卷积所连接的节点的相关性是最高的，而稍微大一点尺寸的卷积，比 如 3x3、5x5的卷积所连接的节点相关性也很高，因此也可以适当地使用一些大尺寸的卷 积，增加多样性(diversity )。最后 Inception Module通过 4 个分支中不同尺寸的 lxl、3x3、 5x5等小型卷积将相关性很高的节点连接在一起，就完成了其设计初衷，构建出了很高效 的符合 Hebbian 原理的稀疏结构。

在 Inception Module中，通常 1x1 卷积的比例(输出通道数占比)最高，3x3卷积和 5x5卷积稍低。而在整个网络中，会有多个堆叠的 Inception Module，我们希望靠后的 Inception Module可以捕捉更高阶的抽象特征，因此靠后的 Inception Module的卷积的空

间集中度应该逐渐降低，这样可以捕获更大面积的特征。因此，越靠后的 Inception Module 中，3x3和 5x5 这两个大面积的卷积核的占比（输出通道数）应该更多。

Inception Net有 22 层深，除了最后一层的输出，其中间节点的分类效果也很好。因 此在 Inception Net中，还使用到了辅助分类节点（auxiliary classifiers ），即将中间某一层 的输出用作分类，并按一个较小的权重（0.3）加到最终分类结果中。这样相当于做了模 型融合，同时给网络增加了反向传播的梯度信号，也提供了额外的正则化，对于整个 Inception Net的训练很有裨益。

当年的 Inception VI还是跑在 TensorFlow 的前辈 DistBelief 上的，并且只运行在 CPU 上。当时使用了异步的 SGD 训练，学习速率每迭代 8 个 epoch P。低 4%。同时，Inception VI也使用了 Multi-Scale, Multi-Crop等数据增强方法，并在不同的釆样数据上训练了 7 个模型进行融合，得到了最后的 ILSVRC 2014的比赛成绩——top-5错误率 6.67%。

同时，Google Inception Net还是一个大家族，包括：

•    20M 年 9 月的论文 Going Deeper with Convolutions 提出的 Inception VI （top-5 错误率 6.67%） o

•    2015 年 2 月的论文及 zZc/z Normalization: Accelerating Deep Network Training by Reducing Internal CbvanTzZe 提出的 Inception V2 （top-5 错误率 4.8% ） o

•    2015 年 12 月的论文 AeZAznhng fAe Inception Architecture for Computer P'Zsf⑽提 出的 Inception V3 （top-5 错误率 3.5% ）。

•    2016 年 2 月的论文 Inception-v4, Inception-ResNet and the Impact of Residual

Connections on    ■提出的 Inception V4 （top-5 错误率 3.08% ） o

Inception V2学习了 VGGNet，用两个 3x3 的卷积代替 5x5 的大卷积（用％降低参数 量并减轻过拟合），还提出了著名的 Batch Normalization （以下简称 BN ）方法。BN是一 个非常有效的正则化方法，可以让大型卷积网络的训练速度加快很多倍，同时收敛后的分 类准确率也可以得到大幅提高。BN在用于神经网络某层时，会对每一个 mini-batch数据 的内部进行标准化（normalization ）处理，使输出规范化到 N（0，l）的正态分布，减少了 Internal Covariate Shift （内部神经元分布的改变）。BN的论文指出，传统的深度神经网 络在训练时，每一层的输入的分布都在变化，导致训练变得困难，我们只能使用一个很小 的学习速率解决这个问题。而对每一层使用 BN 之后，我们就可以有效地解决这个问题， 学习速率可以增大很多倍，达到之前的准确率所需要的迭代次数只有 1/14，训练时间大 大缩短。而达到之前的准确率后，可以继续训练，并最终取得远超于 Inception VI模型的 性能——top-5错误率 4.8%，已经优于人眼水平。因为 BN 某种意义上还起到了正则化的 作用，所以可以减少或者取消 Dropout，简化网络结构。

当然，只是单纯地使用 BN 获得的增益还不明显，还需要一些相应的调整：增大学习 速率并加快学习衰减速度以适用 BN 规范化后的数据；去除 Dropout 并减轻 L2 正则（因 BN已起到正则化的作用）；去除 LRN；更彻底地对训练样本进行 shuffle；减少数据增强 过程中对数据的光学畸变（因为 BN 训练更快，毎个样本被训练的次数更少，因此更真实 的样本对训练更有帮助）。在使用了这些措施后，Inception V2在训练达到 Inception VI 的准确率时快了 14倍，并且模型在收敛时的准确率上限更高。

而 Inception V3网络则主要有两方面的改造：一是引入了 Factorization into small convolutions的思想，将一个较大的二维卷积拆成两个较小的一维卷积，比如将 7x7 卷积 拆成 1x7 卷积和 7x1 卷积，或者将 3x3 卷积拆成 1x3 卷积和 3x1 卷积，如图 6-12所示。 一方面节约了大量参数，加速运算并减轻了过拟合（比将 7x7 卷积拆成 1x7 卷积和 7x1 卷积，比拆成 3 个 3x3 卷积更节约参数），同时增加了一层非线性扩展模型表达能力。论 文中指出，这种非对称的卷积结构拆分，其结果比对称地拆为几个相同的小卷积核效果更 明显，可以处理更多、更丰富的空间特征，增加特征多样性。

图 6-12将一个 3x3 卷积拆成 1x3 卷积和 3x1 卷积

另一方面，Inception V3 优化了 Inception Module 的结构，现在 Inception Module 有 35x35, 17x17和 8x8 三种不同结构，如图 6-13所示。这些 Inception Module只在网络的 后部出现，前部还是普通的卷积层。并且 Inception V3除了在 Inception Module中使用分 支，还在分支中使用了分支（8x8的结构中），可以说是 Network In Network In Network。

图 6_13 Inception V3 中三种结构的 Inception Module

而 Inception V4相比 V3 主要是结合了微软的 ResNet，而 ResNet 将在 6.4.节单独讲 解，这里不多做赘述。因此本节将实现的是 Inception V3，其整个网络结构如表 6-1所示。 由于 Google Inception Net V3相对比较复杂，所以这里使用 tf.contrib.slim辅助设计这个 网络。contrib.slim中的一些功能和组件可以大大减少设计 Inception Net的代码量，我们 只需要少量代码即可构建好有 42 层深的 Inception V3。

| 表 6-1 Inception V3网络结构 |                           |           |
| -------------------------- | ------------------------- | --------- |
| 类    型                   | kernel尺寸/步长（或注释） | 输入尺寸  |
| 卷积                       | 3x3/2                     | 299x299x3 |

续表

| 类    型        | kernel尺寸/步长（或注释） | 输入尺寸   |
| --------------- | ------------------------- | ---------- |
| 卷积            | 3x3/1                     | 149x149x32 |
| 卷积            | 3x3/1                     | 147x147x32 |
| 池化            | 3x3/2                     | 147x147x64 |
| 卷积            | 3x3/1                     | 73x73x64   |
| 卷积            | 3x3/2                     | 71x71x80   |
| 卷积            | 3x3/1                     | 35x35x192  |
| Inception模块组 | 3 个 Inception Module     | 35x35x288  |
| Inception模块坦 | 5 个 Inception Module     | 17x17x768  |
| Inception模块组 | 3 个 Inception Module     | 8x8x1280   |
| 池化            | 8x8                       | 8x8x2048   |
| 线性            | logits                    | 1x1x2048   |
| Softmax         | 分类输出                  | 1x1x1000   |

首先定义一个简单的函数 trunc_normal，产生截断的正态分布。本节代码主要来自 TensorFlow的开源实现 53。

import tensorflow as tf

slim = tf.contrib.slim

trunc_normal = lambda stddev: tf.truncated_normal_initializep(0.0j stddev)

下面定义函数 inception_v3_arg_scope，用来生成网络中经常用到的函数的默认参数， 比如卷积的激活函数、权重初始化方式、标准化器等。设置 L2 正则的 weight_deCay默认 值为 0.00004，标准差 stddev 默认值为 0.1，参数 batch_norm_var_collection默认值为 moving_vars。接下来，定义 batch normalization的参数字典，定义其衰减系数 decay 为 0.9997, epsilon 为 0.001，updates_collctions 为 tf.GrpahKeys.UPDATE_OPS，然后字典 variables_collections 中 beta 和 gamma 均设置为 None, moving_mean 和 moving_variance 均设置为前面的 batch_ _norm_var_collection。

接下来使用 slim.arg_scope，这是一个非常有用的工具，它可以给函数的参数自动赋 予某些默认值。例如，这句 with slim.arg_scope([slim.conv2d, slim.fully_connected], weig hts_regularizer=slim.l2_regularizer(weight_decay))，会对[slim.conv2d, slim.ftilly一 connected] 这两个函数的参数自动赋值，将参数 weights_regularizer的值默认设为 slim.l2_regularizer(w eight_decay)。使用了 slim.arg_scope后就不需要每次都重复设置参数了，只需要在有修改

Tensor闩 ow 实故

时设置。接下来，嵌套一个 slim.arg_scope，对卷积层生成函数 slim.conv2d的几个参数赋 予默认值，其权重初始化器 weights_initializer设置为 trunc_normal(stddev)，激活函数设置 为 ReLU，标准化器设置为 slim.batch_norm，标准化器的参数设置为前面定义的 batch_no rmjparamso最后返回定义好的 scope。

因为事先定义好了 slim.C0Iw2d中的各种默认参数，包括激活函数和标准化器，因此 后面定义一个卷积层将会变得非常方便。我们可以用一行代码定义一个卷积层，整体代码 会变得非常简洁美观，同时设计网络的工作量也会大大减轻。

def inception_v3_arg_scope(weight_decay=0.00004J stddev=0.1^

batch_norm_var_collection= * moving_vars'):

batch_norm_params = {

'decay*: 0.9997,

'epsilon': 0.001,

1updates_collections,: tf.GraphKeys.UPDATE_OPS?

*variables_collections*: {

'beta*: None,

'gamma *: None,

'movingjnean*: [batch__norm_var_collection]

'moving_variance,: [batch_norm_var_collection]t

}

}

with slim.arg_scope([slim.conv2dj slim.fully_connected], weights_regularizer=slim,12_regularizer(weight_decay)):

with slim.arg_scope(

[slim.conv2d]

weights_initializer=tf.truncated_normal_initializer(stddev=stddev) activation_fn=tf.nn.relu, normalizer_fn=slim.batch_norm^ normalizer_params=batch_norm_params) as sc:

return sc

接下来我们就定义函数 inception_v3_base，它可以生成 Inception V3网络的卷积部分, 参数 inputs 为输入的图片数据的 tensor, scope为包含了函数默认参数的环境。我们定义 一个字典表 endpoints，用来保存某些关键节点供之后使用。接着再使用 slim.arg_scope， 对 slim.conv2d、slim.max_pool2d和 slim avg poo!2d这三个函数的参数设置默认值，将 stride设为 1，padding设为 VALID。下面正式开始定义 Inception V3的网络结构，首先是 前面的非 Inception Module的卷积层。这里直接使用 slim.conv2d创建卷积层，slim.conv2d 的第 1 个参数为输入的 tensor，第 2 个参数为输出的通道数，第 3 个参数为卷积核尺寸， 第 4 个参数为步长 stnde，第 5 个参数为 padding 模式。我们的第一个卷积层的输出通道 数为 32，卷积核尺寸为 3x3，步长为 2, padding模式则是默认的 VALID。后面的几个卷 积层采用相同的形式，按照论文中的定义，逐层定义好网络结构。因为使用了 slim及 Slim.arg_scope，我们一行代码就可以定义好一个卷积层，相比之前 AlexNet 的实现中使用 好几行代码定义一个卷积层，或是 VGGNet 中专门写一个函数来定义卷积层，都更加方 便。

我们可以观察到，在前面几个普通的非 Inception Module的卷积层中，主要使用了 3x3的小卷积核，这是充分借鉴了 VGGNet的结构。同时，Inception V3论文中也提出了 Factorization into small convolutions思想，利用两个 1 维卷积模拟大尺寸的 2 维卷积，减 少参数量同时增加非线性。前面几层卷积中还有一层 1x1 卷积，这也是前面提到的 Inception Module中经常使用的结构之一，可低成本的跨通道的对特征进行组合。另外可 以看到，除了第一个卷积层步长为 2，其余的卷积层步长均为 1，而池化层则是尺寸为 3x3、 步长为 2 的重叠最大池化，这是 AlexNet 中使用过的结构。网络的输入数据尺寸为 299x299x3，在经历 3 个步长为 2 的层之后，尺寸最后缩小为 35x35x192，空间尺寸大大 降低，但是输出通道增加了很多。这部分代码中一共有 5 个卷积层，2个池化层，实现了 对输入图片数据的尺寸压缩，并对图片特征进行了抽象。

def inception_v3_base(inputs^ scope=None):

end_points = {}

with tf.variable_scope(scopej ■InceptionV3\ [inputs]):

with slim.arg_scope([slim.conv2d, slim.max_pool2dJ slim.avg_pool2d]_,

stride=l, padding='VALID'):

net = slim.conv2d(inputs_, 32，[3，3]) stride=2, scope='Conv2d_la_3x3') net = slim.conv2d(netj 32, [3, 3]， scope='Conv2d_2a_3x3')

—厂 TensorFlow 实战

net = slim.conv2d(net, 64) [3, 3]) padding='SAME', scope=*Conv2d_2b_3x3')

net = slim.max_pool2d(net, [3^ 3]stride=2j scope='MaxPool_3a_3x3')

net = slim.conv2d(net, 80) [1』1], scope=’Conv2d_3b_lxl,)

net = slim.conv2d(netj 192j [3_, 3] scope=1 Conv2d_4a_3x3')

net = slim.max_pool2d(netj [3_, 3]_, stride=2^ scope='MaxPool_5a_3x3')

接下来就将是三个连续的 Inception 模块组，这三个 Inception 模块组中各自分别有多 个 Inception Module，这部分的网络结构即是 Inception V3的精华所在。每个 Inception 模块组内部的几个 Inception Module结构非常类似，但存在一些细节不同。

第 1 个 Inception 模块组包含了 3个结构类似的 Inception Module，它们的结构和图 6-13中第一幅图非常相似。其中第 1 个 Inception Module的名称为 Mixed_5b。我们先使 用 slim.arg_scope设置所有 Inception 模块组的默认参数，将所有卷积层、最大池化、平均 池化层的步长设为 1，padding模式设为 SAME。然后设置这个 Inception Module的 variable_scope 名称为 Mixed_5bo 这个 Inception Module 中有 4 个分支，从 Branch_0 到 Branch_3，第一个分支为有 64 输出通道的 1x1 卷职；第 2 个分支为有 48 输出通道的 1x1 卷积，连接有 64 输出通道的 5x5 卷积；第 3 个分支为有 64 输出通道的 1x1 卷积，再连 续连接 2 个有 96 输出通道的 3x3 卷积；第 4 个分支为 3x3 的平均池化，连接有 32 输出 通道的 1x1 卷积。最后，使用 tf.concat将 4 个分支的输出合并在一起(在第 3 个维度合 并，即输出通道上合并)，生成这个 Inception Module的最终输出。因为这里所有的层步 长均为 1，并且 padding 模式为 SAME，所以图片的尺寸并不会缩小，依然维持在 35x35。 不过通道数增加了，4个分支的输出通道数之和 64+64+96+32=256，即最终输出的 tensor 尺寸为 35x35x256。这里需注意，第 1 个 Inception 模块组中所有 Inception Module输出 的图片尺寸均为 35x35，但是后两个 Inception Module的通道数会发生变化。.

with slim.arg_scope([slim.conv2dj slim.max_pool2dJ slim.avg_pool2d], stride=l, padding=*SAME'):

with tf.variable_scope('Mixed_5b1): with tf.variable_scope(1Branch_0*):

branch_0 = slim.conv2d(net, 64, [1， 1], scope='Conv2d_0a_lxl') with tf.variable_scope('Branch_l'):

branch_l = slim.conv2d(netj 48j [1^ 1], scope=1Conv2d_0a_lxl') branch_l = slim.conv2d(branch_lj 64^ [5^ 5],

scope='Conv2d_0b_5x5 *) with tf. variable_scope(' Branch」1):

branch_2 = slim.conv2d(netj 64，[1_, 1]_, scope='Conv2d_0a_lxl') branch_2 = slim.conv2d(branch_2j 96j [3? 3]?

scope='Conv2d_0b_3x3 *)

branch」=slim.conv2d(branch_2j 96j [3_, 3]j scope='Conv2d_0c_3x3')

with tf.variable_scope('Branch_3'):

branch_3 = slim.avg_pool2d(netJ [3j 3]_, scope='AvgPool_0a_3x3') branch」=slim.conv2d(branch_3? 32j [lj 1]

scope:1Conv2d_0b_lxl')

net = tf.concat([branch_0j branch_lj branch_2j branch_3]_, 3)

接下来是第 1 个 Inception 模块组的第 2 个 Inception Module-Mixed_5c，这里依然

使用前面设置的默认参数：步长为 1，padding模式为 SAME。这个 Inception Module同 样有 4 个分支，唯一不同的是第 4 个分支最后接的是 64 输出通道的 1x1 卷积，而此前是 32输出通道。因此，我们输出 tensor 的最终尺寸为 35x35x288，输出通道数相比之前增 加了 32O

with tf.variable_scope('Mixed_5c'): with tf.variable_scope('Branch_0'):

branch_0 = slim.conv2d(net, 64, [1, 1], scope= *Conv2d_0a_lxl*) with tf.variable_scope(* Branch_l1):

branch_l = slim.conv2d(net, 48， [1) 1], scope='Conv2d_0b_lxl') branch_l = slim.conv2d(branch_lj 64， [5， 5]，

scope:’ Conv_l_0c_5x5') with tf.variable一 scope('Branch_2'):

branch_2 = slim.conv2d(netj 64j [1, 1], scope='Conv2d_0a_lxl') branch_2 = slim.conv2d(branch_2j 96】[3_, 3]，

scope:■Conv2d_0b_3x3')

branch_2 = slim.conv2d(branch_2, 96) [3, 3], scope:1Conv2d_0c_3x3')

with tf.variable_scope('Branch_31):

branch_3 = slim.avg_pool2d(net, [3， 3]， scope='AvgPool_0a_3x3')

branch_3 = slim.conv2d(branch_3, 64， [1， 1]， scope:'Conv2d_0b_lxl')

net = tf.concat([branch_0j branch_l， branch一 2, branch_3]j 3)

而第 1 个 Inception 模块组的第 3 个 Inception Module-Mixed_5d和上一个

Inception Module完全相同，4个分支的结构、参数一模一样，输出 tensor 的尺寸也为 35x35x288。

with tf.variable_scope('Mixed_5d'): with tf.variable_scope('Branch_0'):

branch_0 = slim.conv2d(net，64, [1，1]，scope= *Conv2d_0a_lxl1) with tf.variable_scope('Branch一 11):

branch_l = slim.conv2d(net, 48) [1) 1], scope=1Conv2d_0a_lxl') branch_l = slim.conv2d(branch_l, 64， [5， 5])

scope:•Conv2d_0b_5x51) with tf.variable_scope('Branch_2'):

branch_2 = slim.conv2d(netj 64，[1_, 1]，scope='Conv2d_0a_lxl') branch_2 = slim.conv2d(branch_2j 96, [3^

scope:1Conv2d_0b_3x3')

branch_2 = slim.conv2d(branch_2j 96^ [3^ 3], scope='Conv2d_0c_3x3')

with tf .variable_scope(' Branch」')：

branch一 3 = slim.avg_pool2d(netJ [34    scope= *AvgPool_0a_3x3')

branch_3 = slim.conv2d(branch_3> 64， [1， 1]

scope:’ Conv2d_0b_lxl')

net = tf.concat([branch_0j branch_l_, branch_2_, branch」],3)

第 2 个 Inception 模块组是一个非常大的模块组，包含了 5个 Inception Module，其中 第 2 个到第 5 个 Inception Module的结构非常类似，它们的结构如图 6-13中第二幅图所 示。其中第 1 个 Inception Module名称为 Mixed_6a，它包含 3 个分支。第 1 个分支是一 个 384 输出通道的 3x3 卷积，这个分支的通道数一下就超过了之前的通道数之和。不过 步长为 2，因此图片尺寸将会被压缩，且 padding 模式为 VALID，所以图片尺寸缩小为 17x17; 第 2 个分支有三层，分别是一个 64 输出通道的 1x1 卷积和两个 96 输出通道的 3x3 卷积。 这里需要注意，最后一层的步长为 2, padding模式为 VALID，因此图片尺寸也被压缩，

本分支最终输出的 tensor 尺寸为 17x17x96；第 3 个分支是一个 3x3 最大池化层，步长同 样为 2, padding模式为 VALID，因此输出的 tensor 尺寸为 17x17x256。最后依然是使用 tf.concat将三个分支在输出通道上合并，最后的输出尺寸为 17x17x(384+96+256)=17x17x768。在第 2 个 Inception 模块组中，5 个 Inception Module 输出 tensor 的尺寸将全部定格为 17x17x768，即图片尺寸和输出通道数都没有发生变化。

with tf.variable_scope('Mixed_6a'): with tf.variable_scope(*Branch_0*):

branch_0 = slim.conv2d(net384_, [3_, 3], stride=2^

padding='VALID'scope='Conv2d_la_lxl')

with tf.variable_scope(*Branch_l*):

branch_l = slim. conv2d(net4 64, [1，1]_, scope='Conv2d_0a_lxl') branch_l = slim.conv2d(branch_lJ 96, [3? 3],

scope:1Conv2d_0b_3x3')

branch_l = slim.conv2d(branch_l, 96,    3]_, stride=2>

padding= ’VALID■, scope='Conv2d_la_lxl')

with tf.variable_scope('Branch_2*):

branch_2 = slim.max_pool2d(net^ [3^ 3]_, stride=2j padding= 'VALID',

scope=•MaxPool_la_3x3')

net = tf.concat([branch_0j branch_l, branch一 2], 3)

接下来是第 2 个 Inception 模块组的第 2 个 Inception Module-Mixed_6b，它有 4

个分支。第 1 个分支是一个简单的 192 输出通道的 1x1 卷积；第 2 个分支由 3 个卷积层 组成，第 1 层是 128 输出通道的 1x1 卷积，第 2 层是 128 通道数的 1x7 卷积，第 3 层是 192输出通道数的 7x1 卷积。这里即是前面提到的 Factorization into small convolutions 思想，串联的 1x7 卷积和 7x1 卷积相当于合成了一个 7x7 卷积，不过参数量大大减少了 (只有后者的 2/7 )并减轻了过拟合，同时多了一个激活函数增强了非线性特征变换；第 3 个分支一下子拥有了 5个卷积层，分别是 128 输出通道的 1x1 卷积，128输出通道的 7x1 卷积，128输出通道的 1x7 卷积，128输出通道的 7x1 卷积和 192 输出通道的 1x7 卷积。 这个分支可以算是利用 Factorization into small convolutions的典范，反复地将 7x7 卷积 进行拆分；最后，第 4 个分支是一个 3x3 的平均池化层，再连接 192 输出通道的 1x1 卷 积。最后将 4 个分支合并，这一层输出 tensor 的尺寸即是 17x17x(192+192+192+192)=17x17x768。

with tf.variable_scope(,Mixed_6b,): with tf.variable_scope(1Branch_0'):

branch_0 = slim.conv2d(net, 192， [1, 1]， scope='Conv2d_0a_lxl') with tf.variable_scope(1Branch_l'):

branch_l = slim.conv2d(netj 128, [1_, 1]scope='Conv2d_0a_lxl') branch_l = slim.conv2d(branch_l, 128, [1_, 7],

scope:1Conv2d_0b_lx7 ’) branch_l = slim.conv2d(branch_lJ 192, [7, 1], scope:'Conv2d_0c_7xl')

with tf.variable_scope('Branch_2'):

branch_2 = slim.conv2d(net128? [1? 1]} scope='Conv2d_0a_lxl') branch_2 = slim.conv2d(branch_2j 128, [7, 1],

scope='Conv2d_0b_7xl*) branch_2 = slim.conv2d(branch_2, 128, [1) 7], scope:1Conv2d_0c_lx7,) branch一 2 = slim.conv2d(branch_2> 12% [7} 1], scope:1Conv2d_0d_7xl') branch_2 = slim.conv2d(branch_2> 1923 [1,7], scope:1Conv2d_0e_lx7 ’)

with tf. variable_scope(' Branch」1):

branch_3 = slim.avg_pool2d(net[3, 3]j scope='AvgPool_0a_3x3') branch_3 = slim.conv2d(branch_3J 192) [1, 1],

scope:1Conv2d_0b_lxl')

net = tf.concat([branch_03 branch_l7 branch_2> branch」],3)

然后是我们第 2 个 Inception 模块组的第 3 个 Inception Module--Mixed_6co

Mixed_6c和前面一个 Inception Module非常相似，只有一个地方不同，即第 2 个分支和 第 3 个分支中前几个卷积层的输出通道数不同，从 128 变为了 160，但是这两个分支的最 终输出通道数不变，都是 192。其他地方则完全一致。需要注意的是，我们的网络每经过 —个 Inception Module，即使输出 tensor 尺寸不变，但是特征都相当于被重新精炼了一遍， 其中丰富的卷积和非线性化对提升网络性能帮助很大。

with tf.variable_scope(*Mixed_6c'): with tf.variable_scope(1Branch_0*):

branch_0 = slim.conv2d(net, 192， [1， 1]， scope='Conv2d_0a_lxl') with tf.variable_scope('Branch_l'):

branch_l = slim.conv2d(net, 160， [1, 1]j scope='Conv2d_0a_lxl*) branch_l = slim.conv2d(branch_l, 160j [1} 7],

scope='Conv2d_0b_lx71) branch_l = slim.conv2d(branch_lj 192) [7) 1], scope:•Conv2d_0c_7xl')

with tf. variable_scope(' Branch」1):

branch_2 = slim.conv2d(net, 160，[1，1]scope= *Conv2d_0a_lxl') branch_2 = slim.conv2d(branch_2J 160, [7〕 1],

scope:'Conv2d_0b_7xl') branch_2 = slim.conv2d(branch_2j 160, [1_, 7]3 scope='Conv2d_0c_lx7') branch_2 = slim.conv2d(branch_2j 160, [1} 1], scope:1Conv2d_0d_7xl•) branch_2 = slim.conv2d(branch_2j 192j [1, 7], scope:■Conv2d_0e_lx7')

with tf.variable_scope('Branch_3'):

branch_3 = slim.avg_pool2d(net[3_, 3], scope='AvgPool_0a_3x31) branch_3 = slim.conv2d(branch_3j 192) [1) 1]}

scope:* Conv2d_0b_lxl1)

net = tf.concat([branch一 0, branch_lj branch_2j branch_3]j 3)

Mixed_6d和前面的 Mixed_6c完全一致，目的同样是通过 Inception Module精心设计 的结构增加卷积和非线性，提炼特征。

with tf.variable_scope('Mixed_6d'): with tf.variable_scope(1Branch_0,):

branch_0 = slim.conv2d(net_, 192_, [1_, 1]_, scope='Conv2d_0a_lxl1) with tf.variable_scope(*Branch_l'):

branch」=slim.conv2d(netj 160_, [1_, 1]) scope:lConv2d_0a_lxl,) branch_l = slim.conv2d(branch_lj 1603 [13 7]j

scope:'Conv2d_0b_lx7•) branch_l = slim.conv2d(branch_lj 192j [7} 1],

scope:1Conv2d_0c_7xl') with tf.variable_scope('Branch_2'):

branch_2 = slim.conv2d(net160[1，1], scope='Conv2d_0a_lxl') branch_2 = slim.conv2d(branch_2j 160) [7, 1]?

scope:1Conv2d_0b_7xl') branch_2 = slim.conv2d(branch一 2, 160, [1, 7], scope:1Conv2d_0c_lx7*) branch_2 = slim.conv2d(branch_2， 160， [7， 1], scope:1Conv2d_0d_7xl') branch_2 = slim.conv2d(branch_2J192_, [1_, 7], scope:1Conv2d_0e_lx71)

with tf.variable_scope('Branch_3'):

branch_3 = slim.avg_pool2d(netJ [3_, 3] j scope= * AvgPool_0a_3x3') branch_3 = slim.conv2d(branch_3, 192) [1, 1],

scope:1Conv2d_0b_lxl')

net = tf.concat([branch_0j branch_lj branch_2, branch_3], 3)

Mixed_6e也和前面两个 Inception Module完全一致。这是第 2 个 Inception 模块组的 最后一个 Inception Module。我们将 Mixed_6e 存储于 end_points 中，作为 Auxiliary Classifier辅助模型的分类 D

with tf.variable_scope('Mixed_6e*): with tf.variable_scope('Branch_0'):

branch_0 = slim.conv2d(net, 192) [1, 1], scope='Conv2d_0a_lxl') with tf.variable_scope('Branch_l'):

branch_l = slim.convZdCnetj 192) [1) 1]scope='Conv2d_0a_lxl') branch_l = slim.conv2d(branch__l, 192, [1, 7],

scope:’ Conv2d_0b_lx71) branch_l = slim.conv2d(branch_l, 192, [7, 1]， scope:•Conv2d_0c_7xl')

with tf.variable_scope(* Branch一 2*):

branch_2 = slim.conv2d(net, 192) [1, 1]j scope= *Conv2d_0a_lxl') branch_2 = slim.conv2d(branch_2, 192， [7， 1]

scope:1Conv2d_0b_7xl')

branch_2 = slim.conv2d(branch_2, 192) [1_, 7]』 scope:1Conv2d_0c_lx7') branch一 2 = slim.conv2d(branch_2j 192j [7, 1]_, scope:1Conv2d_0d_7xl') branch_2 = slim.conv2d(branch_2j 192j [1_, 7]j scope:•Conv2d_0e_lx7')

with tf.variable_scope('Branch_3'):

branch_3 = slim.avg_pool2d(net[3j 3]j scope='AvgPool_0a_3x3') branch_3 = slim.conv2d(branch_3j 192_, [lj 1]

scope:1Conv2d_0b_lxl')

net = tf.concat([branch_0j branch_lj branch_2j branch_3]3) end_points['Mixed_6e'] = net

第 3 个 Inception 模块组包含了 3 个 Inception Module，其中后两个 Inception Module 的结构非常类似，它们的结构如图 6-13中第三幅图所示。其中第 1 个 Inception Module 的名称为 Mixed_7a，包含了 3个分支。第 1 个分支是 192 输出通道的 1x1 卷积，再接 320 输出通道数的 3x3 卷积，不过步长为 2,padding模式为 VALID，因此图片尺寸缩小为 8x8; 第 2 个分支有 4 个卷积层，分别是 192 输出通道的 1x1 卷积、192输出通道的 1x7 卷积、 192输出通道的 7x1 卷积，以及 192 输出通道的 3x3 卷积。注意最后一个卷积层同样步长 为 2, padding为 VALID，因此最后输出的 tensor 尺寸为 8x8x192；第 3 个分支则是一个 3x3的最大池化层，步长为 2, padding为 VALID，而池化层不会对输出通道产生改变， 因此这个分支的输出尺寸为 8x8x768。最后，我们将 3 个分支在输出通道上合并，输出 tensor 尺寸为 8x8x(320+192+768)=8x8xl280。从这个 Inception Module 开始，输出的图片 尺寸又被缩小了，同时通道数也增加了，tensor的总 size 在持续下降中。

with tf.variable_scope(*Mixed_7a'):    -

with tf.variable_scope('Branch一 0'):

branch_0 = slim.conv2d(net, 192, [1, 1]， scope='Conv2d_0a_lxl1) branch_0 = slim.conv2d(branch_0, 320) [3, 3], stride=2,

padding= ’VALID•, scope=*Conv2d_la_3x3*) with tf.variable_scope('Branch_l'):

i    135 A

[www.aibbt.com](http://www.aibbt.com) 让未来触手可及



branch_l = slim.conv2d(net, 192, [1, 1]4 scope=1Conv2d_0a_lxl') branch_l = slim.conv2d(branch_l, 192, [1, 7],

scope:■Conv2d_0b_lx7')

branch_l = slim.conv2d(branch_l? 192， [7， 1]} scope:1Conv2d_0c_7xl1)

branch_l = slim.conv2d(branch_lj 192j [3} stride=2j

padding='VALID', scope='Conv2d_la_3x3')

with tf.variable_scope(* Branch_2*):

branch_2 = slim.max_pool2d(net[3} 3]_, stride=2_, padding='VALID'

scope=1MaxPool_la_3x3')

net = tf.concat([branch_0, branch_l, branch_2]3)

接下来是第 3 个 Inception 模块组的第 2 个 Inception Module，它有 4 个分支。第 1 个分支是一个简单的 320 输出通道的 1x1 卷积；第 2 个分支先是 1 个 384 输出通道的 1x1 卷积，随后在分支内开了两个分支，这两个分支分别是 384 输出通道的 1x3 卷积和 384 输出通道的 3x1 卷积，然后使用 tf.concat合并两个分支，得到的输出 tensor 尺寸为 8x8x(384+384)=8x8x768；第 3 个分支更复杂，先是 448 输出通道的 1x1 卷积，然后是 384 输出通道的 3x3 卷积，然后同样在分支内拆成两个分支，分别是 384 输出通道的 1x3 卷 积和 384 输出通道的 3x1 卷积，最后合并得到 8x8x768 的输出 tensor；第 4 个分支是在一 个 3x3 的平均池化层后接一个 192 输出通道的 1x1 卷积。最后，将这个非常复杂的 Inception Module的 4 个分支合并在一起，得到的输出 tensor 尺寸为 8x8x(320+768+768+192)=8x8x2048。到这个 Inception Module，输出通道数从 1280 增加 到了 2048。

with tf.variable_scope('Mixed_7b'): with tf.variable_scope(1Bpanch_0*):

branch_0 = slim.conv2d(net, 320? \_13 1}3 scope='Conv2d_0a_lxl') with tf.variable_scope('Branch_l'):

branch_l = slim.conv2d(net/ 384』[1， 1]， scope='Conv2d_0a_lxr,) branch_l = tf.concat([

slim.conv2d(branch_l) 384) [1， 3]) scope='Conv2d_0b_lx3') slim.conv2d(branch_l> 384) [3， 1]， scope='Conv2d_0b_3xl')]^ 3)

with tf.variable_scope('Branch_2'):

branch_2 = slim.conv2d(netj 44% [1_, 1]_, scope='Conv2d_0a_lxl') branch_2 = slim.conv2d(branch_2_, 384, [3_, 3]_,

scope= ’ Conv2d_0b_3x3*)

branch_2 = tf.concat([

slim.conv2d(branch_2j 384_, [1』3]_, scope='Conv2d_0c_lx3 *) slim.conv2d(branch_2j 384』[3， 1]j scope='Conv2d_0d_3xl,)]J 3)

with tf.variable_scope('Branch_3'):

branch_3 = slim.avg_pool2d(netj 3], scope=1AvgPool_0a_3x3') branch_3 = slim.conv2d(branch_3j 192_, [1_, 1])

scope='Conv2d_0b_lxl')

net = tf.concat([branch_0j branch_l_, branch_2? branch_3]j 3)

Mixed_7c是第 3 个 Inception 模块组的最后一个 Inception Module，不过它和前面的 Mixed_7b是完全一致的，输出 tensor 也是 8x8x2()48D最后，我们返回这个 Inception Module 的结果，作为 inceptio_v3_base函数的最终输出。

with tf.variable_scope('Mixed_7c'): with tf.variable_scope('Branch_0'):

branch_0 = slim.conv2d(netj 320_, [1) 1], scope= *Conv2d_0a_lxl') with tf.variable一 scope(1Branch_l'):

branch_l = slim.conv2d(net, 384』[1， 1]， scope='Conv2d_0a_lxl1) branch_l = tf.concat([

slim.conv2d(branch_l, 384， [1， 3]， scope='Conv2d_0b_lx3')? slim.conv2d(branch_lJ 384， [3， 1]， scope='Conv2d_0c_3xl1)]^ 3)

with tf.variable_scope('Branch_2'):

branch_2 = slim.conv2d(netj 44% [1，1]_, scope='Conv2d_0a_lxl1) branch_2 = slim.conv2d(branch_2j 384, [3, 3]

scope:’ Conv2d_0b_3x31)

branch_2 = tf.concat([

slim.conv2d(branch_2? 384，[1，3]，scope= *Conv2d_0c_lx3') slim.conv2d(branch_2, 384_, [3) 1]』scope='Conv2d_0d_3xl')], 3)

with tf.variable_scope(1 Branch一 31):

branch_3 = slim.avg_pool2d(net, [3， 3}, scope='AvgPool_0a_3x3') branch_3 = slim.conv2d(branch_3^ 192^ [1^ 1]

scope:1Conv2d_0b_lxl’)

net = tf.concat([branch_0, branch_l_, branch_2, branch_3]j 3) return net, end_points

至此，Inception V3网络的核心部分，即卷积层部分就完成了。回忆一下 Inception V3

的网络结构：首先是 5 个卷积层和 2 个池化层交替的普通结构，然后是 3 个 Inception 模 块组，每个模块组内包含多个结构类似的 Inception Module□设计 Inception Net的一个重 要原则是，图片尺寸是不断缩小的，从 299x299 通过 5 个步长为 2 的卷积层或池化层后， 缩小为 8x8；同时，输出通道数持续增加，从一开始的 3 （RGB三色）到 2048。从这里 可以看出，每一层卷积、池化或 Inception 模块组的目的都是将空间结构简化，同时将空 间信息转化为高阶抽象的特征信息，即将空间的维度转为通道的维度。这一过程同时也使 毎层输出 tensor 的总 size 持续下降，降低了计算量。读者可能也发现了 Inception Module 的规律，一般情况下有 4 个分支，第 1 个分支一般是 1x1 卷积，第 2 个分支一般是 1x1 卷积再接分解后（factorized ）的 lxn 和 nxl 卷积，第 3 个分支和第 2 个分支类似，但是 一般更深一些，第 4 个分支一般具有最大池化或平均池化。因此，Inception Module是通 过组合比较简单的特征抽象（分支 1 ）、比较复杂的特征抽象（分支 2 和分支 3 ）和一个简 化结构的池化层（分支 4）, 一共 4 种不同程度的特征抽象和变换来有选择地保留不同层 次的高阶特征，这样可以最大程度地丰富网络的表达能力。

接下来，我们来实现 Inception V3网络的最后一部分——全局平均池化、Softmax和

Auxiliary Logits。先看函数 inception_v3的输入参数，num_classes即最后需要分类的数量， 这里默认的 1000 是 ILSVRC 比赛数据集的种类数；is_training标志是否是训练过程，对 Batch Normalization 和 Dropout 有影响，只有在训练时 Batch Normalization 和 Dropout 才 会被启用；dropout_keep_prob即训练时 Dropout 所需保留节点的比例，默认为 0.8; prediction_fo是最后用来进行分类的函数，这里默认是使用 slim.softmax; spatial_squeeze 参数标志是否对输出进行 squeeze 操作（即去除维数为 1 的维度，比如 5x3x1 转为 5x3 ）; reuse标志是否会对网络和 Variable 进行重复使用；最后，scope为包含了函数默认参数的 环境。首先，使用 tf.variable_scope定义网络的 name 和 reuse 等参数的默认值，然后使用 slim.arg_scope 定义 Batch Normalization 和 Dropout 的 is_training 标志的默认值。最后，使 用前面定义好的 inception_v3_base构筑整个网络的卷积部分，拿到最后一层的输出 net 和 重要节点的字典表 end_points0

def inception一 v3（inputs

num_classes=1000j is_training=True? dropout一 keep_prob=0.8 prediction_fn=slim.softmax, spatial_squeeze=True，

reuse=None, scope='InceptionV3'):

with tf.variable_scope(scope_, *InceptionV31 [inputs, num_ classes], reuse=reuse) as scope:

with slim.arg_scope([slim.batch_normj slim.dropout] is_training=is_training):

netj end_points = inception_v3_base(inputsscope=scope)

接下来处理 Auxiliary Logits这部分的逻辑，Auxiliary Logits作为辅助分类的节点， 对分类结果预测有很大帮助。先使用 SHm.arg_SCOpe将卷积、最大池化、平均池化的默认 步长设为 1，默认 padding 模式设为 SAME。然后通过 end_points取到 Mixed_6e，并在 Mixed_6e之后再接一个 5x5 的平均池化，步长为 3, padding设为 VALID，这样输出的尺 寸就从 17x17x768 变为 5x5x768。接着连接一个 128 输出通道的 1x1 卷积和一个 768 输出 通道的 5x5 卷积，这里权重初始化方式重设为标准差为 0.01的正态分布，padding模式设 为 VALID，输出尺寸变为 1x1x768。然后再连接一个输出通道数为 num_classes的 1 x 1卷 积，不设激活函数和规范化函数，权重初始化方式重设为标准差为 0.001的正态分布，这 样输出变为了 1x1x1000。接下来，使用 tf.squeeze函数消除输出 tensor 中前两个为 1 的维 度。最后将辅助分类节点的输出 auxjogits 储存到字典表 end_points中。

with slim.arg_scope([slim.conv2dj slim.max_pool2dslim.avg_ pool2d]_, stride=l, padding='SAME'):

aux_logits = end_points[1Mixed_6e'] with tf.variable_scope('AuxLogits'):

aux一 logits = slim.avg_pool2d(

aux_logits_, [55]_, stride=3_, padding= 'VALID*}    •

scope='AvgPool_la_5x5')

aux_logits = slim. conv2d(aux_logits? 128_, [1^ 1]_, scope:■Conv2d_lb_lxl')

aux_logits = slim.conv2d( aux_logitSj 768_, [5,5]? weights_initializep=trunc_normal(0.01) padding=■VALID'scope= ’Conv2d_2a_5x5')

aux一 logits = slim.conv2d(

aux一 logits, num_classeSj [1_, 1], activation_fn=NoneJ normalizer_fn=Nonej weights_initializer=trunc_normal(0.001)? scope:■Conv2d_2b_lxl,)

if spatial_squeeze:

aux_logits = tf.squeeze(aux_logits[1_, 2]_,

name='SpatialSqueeze')

end_points['AuxLogits'] = aux_logits

下面处理正常的分类预测的逻辑。我们直接对 Mixed_7e即最后一个卷积层的输出进 行一个 8x8 全局平均池化，padding模式为 VALID，这样输出 tensor 的尺寸就变为了 1x1x2048。然后连接一•个 Dropout 层，节点保留率为 dropout_keep_prob□接着连接一个 输出通道数为 1000 的 1x1 卷积，激活函数和规范化函数设为空。下面使用 tf.squeeze去 除输出 tensor 中维数为 1 的维度，再连接一个 Softmax 对结果进行分类预测。最后返回输 出结果 logits 和包含辅助节点的 end_ponits0

with tf.variable_scope(1 Logits'):

net = slim.avg_pool2d(netJ [8, 8], padding='VALID'

scope='AvgPool_la_8x8 ’)

net = slim, dropout (net keep_prob=dropout一 keep_prob_, scope=*Dropout一 lb1)

end_points[* PreLogits'] = net

logits = slim.conv2d(netj num一 classes) [1_, 1]_, activation— fn=Nonej normalizer_f n=None_, scope= ■ Conv2d_lc_lxl')

if spatial_squeeze:

logits = tf.squeeze(logitsj [1, 2], name='SpatialSqueeze1)    .

end_points['Logits'] = logits

end_points [1 Predictions * ] = prediction_fn( logitsscope=' Predictions *) return logitsend_points

至此，整个 Inception V3网络的构建就完成了。Inception V3是一个非常复杂、精妙 的模型，其中用到了非常多之前积累下来的设计大型卷积网络的经验和技巧。不过，虽然 Inception V3论文中给出了设计卷积网络的几个原则，但是其中很多超参数的选择，包括 层数、卷积核的尺寸、池化的位置、步长的大小、factorization使用的时机，以及分支的

设计，都很难一一解释。目前，我们只能认为深度学习，尤其是大型卷积网络的设计，是 一门实验学科，其中需要大量的探索和实践。我们很难证明某种网络结构一定更好，更多 的是通过实验积累下来的经验总结出一些结论。深度学习的研究中，理论证明部分依然是 短板，但通过实验得到的结论通常也具有不错的推广性，在其他数据集上泛化性良好。

下面对 Inception V3进行运算性能测试。这里使用的 time_tensorflow_run函数和 AlexNet那节一样，因此就不再重复定义，读者可以在 6.1节中找到代码并加载。因为 Inception V3网络；结构较大，所以依然令 batch_size为 32，以免 GPU 显存不够。图片尺 寸设置为 299x299，并用 tf.random_uniform生成随机图片数据作为 input。接着，我们使 用 slim.arg_scope 加载前面定义好的 inception_v3_arg_scope()，在这个 scope 中包含了 Batch Normalization的默认参数，以及激活函数和参数初始化方式的默认值。然后在这个 、arg_scope 下，调用 inception_v3 函数，并传入 inputs，获取 logits 和 endjpoints。下面创 建 Session 并初始化全部模型参数。最后我们设置测试的 batch 数量为 100，并使用

time_tensorflow_run 测试 Inception V3 网络的 forward 性能。

batch_size = 32 height, width = 299, 299

inputs = tf.random_uniform((batch_sizej height, width, 3)) with slim.arg_scope(inception_v3_arg_scope()):

logitsj end_points = inception_v3(inputsj is_training=False)

init = tf.global_variables_initializer()

sess = tf.Session()    ,

sess.run(init)

num_batches=100

time_tensorflow_run(sessj logits^ "Forward")

从结果来看,Inception V3网络的 forward 性能不错，在 GTX 1080,CUDA 8、cuDNN 5.1的环境下，每个 batch (包含 32 张图片)预测耗时仅为 0.145s。虽然输入图片的面积 比 VGGNet 的 224x224 大了 78%，但是 forward 速度却比 VGGNet 的 0.152s更快。这主 要归功于其较小的参数量，Inception V3网络仅有 2500 万个参数，虽然比 Inception VI 的 700 万多了很多，不过仍然不到 AlexNet 的 6000 万参数量的一半，相比 VGGNet 的 1.4 亿参数量就更少了，这对一个 42 层深的大型网络来说是极为不易的。同时，整个网络的 浮点计算量仅为 50 亿次，虽也比 Incepion VI的 15 亿次大了不少，但是相比 VGGNet

仍然不算大。较小的计算量让 Inception V3.网络变得非常实用，我们可以轻松地将其移植 到普通服务器上提供快速响应的服务，甚至是移植到手机上进行实时的图像识别。

2016-12-10 21:07:09.535980：

2016-12-10 21:07:10.982748：

2016-12-10 21:07:12.430209：

2016-12-10 21:07:13.877055：

2016-12-10 21:07:15.324095：

2016-12-10 21:07:16.770960：

2016-12-10 21:07:18.218127：

2016-12-10 21:07:19.665192：

2016-12-10 21:07:21.113429:

2016-12-10 21:07:22.563213：

2016-12-10 21:07:23.867730：

batch



| step Q, duration = | :0.145 |
| ------------------ | ------ |
| step 10j duration  | =0.145 |
| step 20j duration  | =0.145 |
| step 30j duration  | =0.145 |
| step 40duration    | =0.145 |
| step 50^ duration  | =0.145 |
| step 60j duration  | =0.145 |
| step 70j duration  | =0.145 |
| step 80^ duration  | =0.145 |
| step 90j duration  | =0.145 |

Forward across 100 steps.



0.145 +/- 0.000 sec /



因为篇幅原因，我们就不对 Inception V3的 backward 性能进行测试了，这部分的代码 比较冗长。感兴趣的读者，可以将整个网络的所有鐵加入参数列表，测试对全部参数求导 所需的时间，或者直接下载 ImageNet 数据集，使用真实样本进行训练并评测所需时间。

Inception V3作为一个极深的卷积神经网络，拥有非常精妙的设计和构造，整个网络 的结构和分支非常复杂。我们平时可能不必设计这么复杂的网络，但 Inception V3中仍有 许多设计 CNN 的思想和 Trick 值得借鉴。

(1 ) Factorization into small convolutions很有效，可以降低参数量、减轻过拟合，增 加网络非线性的表达能力。

(2)卷积网络从输入到输出，应该让图片尺寸逐渐减小，输出通道数逐渐增加，即让 空间结构简化，将空间信息转化为高阶抽象的特征信息。

(3 ) Inception Module用多个分支提取不同抽象程度的高阶特征的思路很有效，可以 丰富网络的表达能力。

##### 6.4 TensorFlow 实现 ResNet

ResNet ( Residual Neural Network )由微软研究院的 Kaiming He 等 4 名华人提出，

通过使用 Residual Unit成功训练 152 层深的神经网络，在 ILSVRC 2015比赛中获得了冠 军，取得 3.57%的 top-5错误率，同时参数量却比 VGGNet 低，效果非常突出。ResNet的 结构可以极快地加速超深神经网络的训练，模型的准确率也有非常大的提升。6.3节我们 讲角军并实现了 Inception V3，而 Inception V4 则是将 Inception Module 和 ResNet 相结合。

可以看到 ResNet 是一个推广性非常好的网络结构，甚至可以直接应用到 Inception Net中。 本节就讲解 ResNet 的基本原理，以及如何用 TensorFlow 来实现它。

在 ResNet 之前，瑞士教授 Schmidhuber 提出了 Highway Network，原理与 ResNet 很 相似。这位 Schmidhuber 教授同时也是 LSTM 网络的发明者，而且是早在 1997 年发明的，

可谓是神经网络领域元老级的学者。通常认为神经网络的深度对其性能非常重要，但是网 络越深其训练难度越大，Highway Network的目标就是解决极深的神经网络难以训练的问 题。Highway Network相当于修改了每一层的激活函数，此前的激活函数只是对输入做一 个非线性变换 y = W(x,WH), Highway NetWork则允许保留一定比例的原始输入 x，即 y - H(x, W„) - T(x, WT) + x • C(x, Wc)，其中 T 为变换系数，C为保留系数，论文中令 C = 1- T。这样前面一层的信息，有一定比例可以不经过矩阵乘法和非线性变换，直接 传输到下一层，仿佛一条信息高速公路，因此得名 Highway Network。Highway Network 主要通过 gating units学习如何控制网络中的信息流，即学习原始信息应保留的比例。这 个可学习的 gating 机制：正是借鉴自 Schmidhuber 教授早年的 LSTM 循环神经网络中的 gating。几百乃至上千层深的 Highway Network可以直接使用梯度下降算法训练，并可以 配合多种非线性激活函数，学习极深的神经网络现在变得可行了。事实上，Highway Network的设计在理论上允许其训练任意深的网络，其优化方法基本上与网络的深度独立, 而传统的神经网络结构则对深度非常敏感/训练复杂度随深度增加而急剧增加。'

ResNet和 Highway Network非常类似，也是允许原始输入信息直接传输到后面的层 中。ResNet最初的灵感出自这个问题：在不断加神经网络的深度时，会出现一个 Degradation的问题，即准确率会先上升然后达到饱和，再持续增加深度则会导致准确率 下降。这并不是过拟合的问题，因为不光在测试集上误差增大，训练集本身误差也会增大。 假设有一个比较浅的网络达到了饱和的准确率，那么后面再加上几个 y = x的全等映射层， 起码误差不会增加，即更深的网络不应该带来训练集上误差上升。而这里提到的使用全等

映射直接将前一层输出传到后面的思想，就是 ResNet 的灵感来源。假定某段神经网络的 输入是 X，期望输出是 H(x)，如果我们直接把输入 x 传到输出作为初始结果，那么此时我 们需要学习的目标就是 F0) = //(x)-xD如图 6-14所示，这就是一个 ResNet 的残差学习 单元(Residual Unit )，ResNet相当于将学习目标改变了，不再是学习一个完整的输出//(x), 只是输出和输入的差别 H(x)-x，即残差。

图 6-14 ResNet的残差学习模块

图 6-15所示为 VGGNet-19，以及一个 34 层深的普通卷积网络，和 34 层深的 ResNet 网络的对比图。可以看到普通直连的卷积神经网络和 ResNet 的最大区别在于，ResNet有 很多旁路的支线将输入直接连到后面的层，使得后面的层可以直接学习残差，这种结构也 被称为 shortcut 或 skip connections o

传统的卷积层或全连接层在信息传递时，或多或少会存在信息丢失、损耗等问题。 ResNet在某种程度上解决了这个问题’通过直接将输入信息绕道传到输出，保护信息的 完整性，整个网络则只需要学习输入、输出差别的那一部分，简化学习目标和难度。

在 ResNet 的论文中，除了提出图 6-16中的两层残差学习单元，还有三层的残差学习 单元。两层的残差学习单元中包含两个相同输出通道数(因为残差等于目标输出减去输入, 即 H(x)-x，因此输入、输出维度需保持一致)的 3x3 卷积；而 3 层的残差网络则使用了 Network In Network和 Inception Net中的 lxl 卷积，并且是在中间 3x3 的卷积前后都使 用了 1x1卷积，有先降维再升维的操作。另外，如果有输入、输出维度不同的情况，我 们可以对 x 做一个线性映射变换维度，再连接到后面的层。

图 6-17所示为 ResNet 在不同层数时的网络配置，其中基础结构很类似，都是前面提 到的两层和三层的残差学习单元的堆叠。

output size: 224



output size: 112



output size: 56



output size: 28



VGG-19

image



3x3 conv, 64



pool, /2

3x3 conv, 128



3x3 conv, 128



34-layer plain

image



34-layer residual



image



图 6-15 VGG-19，直连的 34 层网络，ResNet的 34 层网络的结构对比



pool, /2



3x3 conv, 256



3x3 conv, 256



3x3 conv, 2S6



3x3 copv, 256



pool，/2





3x3 conv. 512



![img](06TensorFlow9e18_c4875a088c74095baibbt-80.jpg)



![img](06TensorFlow9e18_c4875a088c74095baibbt-81.jpg)



图 6-16两层及三层的 ResNet 残差学习模块



| layer name | output size |                                  | 18-layer          |      |          | 34-Iayer          |      |          | 50-layer                      |      |         | 101-layer                   |      |           | 15 2-1 ay er                 |      |
| ---------- | ----------- | -------------------------------- | ----------------- | ---- | -------- | ----------------- | ---- | -------- | ----------------------------- | ---- | ------- | --------------------------- | ---- | --------- | ---------------------------- | ---- |
| convl      | 112x112     | 7x7,64, stride 2                 |                   |      |          |                   |      |          |                               |      |         |                             |      |           |                              |      |
|            |             | 3x3 max pool, stride 2           |                   |      |          |                   |      |          |                               |      |         |                             |      |           |                              |      |
| conv2_x    | 56x56       |                                  | ■ 3x3,64 ' 3x3,64 | x2   |          | ■ 3x3,64 ' 3x3.64 | x3   |          | '1x1,64 "3x3,641x1,256        | x3   |         | '1x1,64 '3x3.641x1.256      | x3   |           | 1x1,64 '3x3,641x1,256        | x3   |
| conv3_x    | 28x28       |                                  | 3x3,128 3x3,128   | x2   |          | 3x3, 128 3x3, 128 | x4   |          | _ lxl, 128 ' 3x3, 128 1x1,512 | x4   |         | 'lxl, 128 ' 3x3.128 1x1,512 | x4   |           | 'lxl, 128 ' 3x3, 128 1x1,512 | x8   |
| conv4_x    | 14x14       |                                  | 3x3,2563x3,256    | x2   |          | 3x3,256 3x3, 256  | x6   |          | 1x1,256 3x3,256 lxl, 1024     | x6   |         | 1x1,256 ' 3x3,256 1x1，1024 | x23  |           | 1x1,256 ' 3x3,256 lx], 1024  | x36  |
| conv5_x    | 7x7         |                                  | 3x3,5123x3,512    | x2   |          | 3x3,5123x3,512    | x3   |          | 1x1,5123x3,5121x1,2048        | x3   |         | '1x1,512 3x3,512 1x1,2048   | x3   |           | 1x1,5123x3,5121x1,2048       | x3   |
|            | lxl         | average pool, 1000-d fc, softmax |                   |      |          |                   |      |          |                               |      |         |                             |      |           |                              |      |
| FLOPs      |             | 1.8x10°                          |                   |      | 3.6xlO,J |                   |      | 3.8xlO5' |                               |      | 7.6xl09 |                             |      | 11.3x10s' |                              |      |

图 6-17 ResNet不同层数时的网络配置

在使用了 ResNet的结构后，可以发现层数不断加深导致的训练集上误差増大的现象 被消除了，ResNet网络的训练误差会随着层数增大而逐渐减小，并且在测试集上的表现 也会变好。在 ResNet 推出后不久，Google就借鉴了 ResNet的精髓，提出了 Inception V4 和 Inception-ResNet-V2，并通过融合这两个觀，在 ILSVRC 数据集上取得了惊人的 3.08% 的错误率。可见，ResNet及其思想对卷积神经网络研究的贡献确实非常显著，具有很强 的推广性。在 ResNet 的作者的第二篇相关论文 Identity Mappings in Deep Residual Networks中，ResNet V2被提出。ResNet V2和 ResNet VI的主要区别在于，作者通过 研究 ResNet 残差学习单元的传播公式，发现前馈和反馈信号可以直接传输，因此 skip connection 的非线性激活函数(如 ReLU )替换为 Identity Mappings( y = x )。同时，ResNet V2在每一层中都使用了 Batch Normalization。这样处理之后，新的残差学习单元将比以 前更容易训练且泛化性更强。

根据 Schmidhuber 教授的观点，ResNet类似于一个没有 gates 的 LSTM 网络，即将输 入 x 传递到后面层的过程是一直发生的，而不是学习出来的。同时，最近也有商篇论文 表示，ResNet基本等价于 RNN 且 ResNet 的效果类似于在多层网络间的集成方法 (ensemble)。ResNet在加深网络层数上做出了重大贡献，而另一篇论文 77ze Power of Depth for Feedforward Neural Networks则从理论上证明了加深网络比加宽网络更有效， 算是给 ResNet 提供了声援，也是给深度学习为什么要深才有效提供了合理解释。

下面我们就用 TensorFlow 实现一个 ResNet V2网络。我们依然使用方便的 contrib.slim 库来辅助创建 ResNet，其余载入的库还有原生的 collections。本节代码主要来自 TensorFlow 的开源实现 54。

import collections

import tensorflow as tf

slim = tf.contrib.slim

我们使用 collections.namedtuple设计 ResNet 基本 Block 模块组(图 6-17中所示的 Block ) 的 named tuple，并用它创建 Block 的类，但只包含数据结构，不包含具体方法。我们要 定义一个典型的 Block，需要输入三个参数，分别是 scope、unit_fh和 args。以 Blockfblockl1， bottleneck, [(256, 64, 1)] x 2 + [(256, 64, 2)])这一行代码为例，它可以定义一个典型的 Block，其中 blockl 就是我们这个 Block 的名称(或 scope )； bottleneck是 ResNet V2中 的残差学习单元；而最后一个参数[(256, 64, 1)] x 2 + [(256, 64, 2)]则是这个 Block 的 args, args是一个列表，其中每个元素都对应一个 bottleneck 残差学习单元，前面两个元 素都是(256, 64，1)，最后一个是(256, 64, 2)。每个元素都是一个三元 tuple，即(depth， depth_bottleneck, stride )o 比如(256，64，3)，代表构建的 bottleneck 残差学习单元(每 个残差学习单元包含三个卷积层)中，第三层输出通道数 depth 为 256，前两层输出通道 数 depth_bottleneck为 64，且中间那层的步长 stride 为 3。这个残差学习单元结构即为 [(lxl/sl, 64), (3x3/s2，64), (lxl/sl, 256)]。而在这个 Block 中，一共有 3 个 bottleneck 残 差学习单元，除了最后一个的步长由 3 变为 2，其余都一致。

class Block(collections.namedtuple('Block*j ['scope、1unit_fn'j 'args'])):

'A named tuple describing a ResNet block.'

下面定义一个降釆样 subsample 的方法，参数包括 inputs (输入)、factor (采样因子)

和 scope。这个函数也非常简单，如果 factor 为 1，则不做修改直接返回 inputs；如果不为 1，则使用 slim.max_pool2d最大池化来实现，通过 1x1 的池化尺寸，stride作步长，即可 实现降采样。

def subsample(inputs_, factor, scope=No’ne): if factor == 1:

return inputs else:

return slim.max_pool2d(inputs[1, 1]stride=factor3 scope=scope)

再定义一个 conv2d_same函数创建卷职层。先判断 stride 是否为 1，如果为 1，则直 接使用 slim.conv2d并令 padding 模式为 SAME。如果 stride 不为 1，则显式地 pad zero, 要 pad zero的总数为 kemel_size-l, pad_beg为 pad//2，pad_end为余下的部分。接下来使

—「TensorFlow 实拔

用 tf.pad对输入变量进行补零操作。最后，因为已经进行了 zero padding，所以只需再使 用一个 padding 模式为 VALID 的 slim.conv2d创建这个卷积层。

def conv2d_same(inputSj num_outputSj kernel_size_, stride, scope=None): if stride == 1:

return slim.conv2d(inputsj num_outputs_, kernel_size, stride=lj padding:1 SAME* scope=scope)

else:

pad_total = kernel_size - 1 pad_beg = pad_total // 2 pad_end = pad_total - pad_beg

inputs = tf.pad(inputs? [[0， 0][pad_beg, pad_end]

[pad_beg, pad_end], [0, 0]])

return slim.conv2d(inputs4 num_outputs_, kernel_size_, stride=stride_, padding='VALID'scope=scope)

接下来定义堆叠 Blocks 的函数，参数中的 net 即为输入，blocks是之前定义的 Block 的 class 的列表，而 outputs_collections则是用来收集各个 endjpoints 的 collectionso 下面 使用两层循环，逐个 Block，逐个 Residual Unit ±也堆叠，先使用两个 tf.variable_scope将 残差学习单元命名为 blockl/unit_l的形式。在第 2 层循环中，我们拿到每个 Block 中每个 Residual Unit 的 args，并展开为 depth、depth_bottleneck 和 stride，其含义在前面定义 Blocks 类时已经讲解过。然后使用 unit_fh函数(即残差学习单元的生成函数)顺序地创建并连 接戶斤有的残差学习单元。最后，我们使用 slim.utils.collect_named_outputs函数将输出 net 添加到 collection 中。最后，当所有 Block 中的所有 Residual Unit都堆叠完之后，我们再 返回最后的 net 作为 stack_blocks_dense函数的结果。

@slim.add_arg_scope

def stack_blocks_dense(netj blocks, outputs_collections=None):

for block in blocks:

with tf.variable_scope(block.scope1 block'j [net]) as sc: for unit in enumerate(block.args):

with tf.variable_scope('unit_%d' % (i + 1), values=[net]): unit_depthj unit_depth_bottleneck, unit_stride = unit

net = block.unit_fn(netj

depth=unit_depthj

depth_bottleneck=unit_depth_bottleneck^

stride=unit_stride)

net = slim.utils.collect_named_outputs(outputs_collectionssc.name, net)

return net

这里创建 ResNet 通用的 arg_scope，关于 arg_scope，我们在前面的章节已经介绍过 其功能——用来定义某些函数的参数默认值。这里定义训练标记 isjraining 默认为 True, 权重衰减速率 weight_decay默认为 0.0001, BN的衰减速率默认为 0.997, BN的 epsilon 默认为 le-5, BN的 scale 默认为 True。和在 Inception V3定义 arg_scope—样，先设置好 BN的各项参数，然后通过 slim.arg_scope将 slim.conv2d的几个默认参数设置好：权重正 则器设置为 L2 正则，权重初始化器设为 slim.variance_scaling_initializer()，激活函数设为. ReLU，标准化器设为 BN。并将最大池化的 padding 模式默认设为 SAME (注意，ResNet 原论文中使用的是 VALID 模式，设为 SAME 可让特征对齐更简单，读者可以尝试改为 VALID)。最后，将几层嵌套的 arg_scope作为结果返回。

def resnet_arg_scope(is_training=Truej

weight一 decay=0•0001, batch_norm_decay=0.997_, batch_norm_epsilon=le-5j batch_norm_scale=True):

batch_norm_params = {    •

'isjraining* : is_training?

'decay': batch_norm_decayJ 'epsilon' : batch_norm_epsilon_,

1 scale': batch_norm_scale,

*updates_collections': tf.GraphKeys.UPDATE_OPSj

}

with slim.arg_scope(

[slim.conv2d],

weights_regularizer=slim.l2_regularizer(weight_decay) weights_initializer=slim.variance_scaling_initializer() activation_fn=tf.nn.relu,

normalizer_fn=slim.batch_normj normalizer_params=batch_norm_params):

with slim.arg_scope([slim.batch一 norm], **batch_norm_params):

with slim.arg_scope([slim.max_pool2d]? padding='SAME1) as arg_sc:

return arg_sc

接下来定义核心的 bottleneck 残差学习单元，它是 ResNet V2的论文中提到的 Full Preactivation Residual Unit的一个变种。它和 ResNet VI中的残差学习单兀的主要区别有 两点，一是在每一层前都用了 Batch Normalization，二是对输入进行 preactivation，而不 是在卷积进行激活函数处理。我们来看一下 bottleneck 函数的参数，inputs是输入，depth、 depth_bottleneck 和 stride 这三个参数前面的 Blocks 类中的 args, outputs_collections 是收 集 end_points 的 collection, scope 是这个 unit 的名称。下面先使用 slim.utils.last_dimension 函数获取输入的最后一个维度，即输出通道数，其中的参数 min_rank=4可以限定最少为 4个维度。接着，使用 slim.batch_norm对输入进行 Batch Normalization，并使用 ReLU 函 数进行预激活 Preactivate。然后定义 shorcut (即直连的 x):如果残差单元的输入通道数 depth_in和输出通道数 depth 一致，那么使用 subsample 按步长为 stride 对 inputs 进行空间 上的降采样(确保空间尺寸和残差一致，因为残差中间那层的卷积步长为 stride)；如果输 入、输出通道数不一样，我们用步长为 stride 的 1x1 卷积改变其通道数，使得与输出通道 数一致。然后定义 residual (残差)，residual这里有 3 层，先是一个 1x1 尺寸、步长为 1、 输出通道数为 depth_bottleneck的卷积，然后是一个 3x3 尺寸、步长为 stride、输出通道数 为 depth_bottleneck的卷积，最后是一个 1x1 卷积、步长为 1、输出通道数为 depth 的卷积， 得到最终的 residual，这里注意最后一层没有正则项也没有激活函数。然后将 residual 和 shorcut相加，得到最后结果 output，再使用 slim.utils.collect_named_outputs将结果添加进 collection并返回 output 作为函数结果。

@slim.add_arg_scope

def bottleneck (inputs depths depth_bottleneck_, stride_, outputs_collections=Nonej scope=None):

with tf.variable_scope(scope'bottleneck一 v2*[inputs]) as sc:

depth_in = slim.utils.last_dimension(inputs.get_shape(), min_rank=4) preact = slim.batch_norm(inputs? activation_fn=tf.nn.reluj

scope='preact *)

if depth == depth_in:

shortcut = subsample(inputs, stride, 'shortcut*) else:

shortcut = slim.conv2d(preactj depth, [1, 1], stride=stridej

normalizer_fn=Nonej activation_fn=None_, scope='shortcut')

residual = slim.conv2d(preactdepth—bottleneck, [1, 1]stride=lj scope=* convl*)

residual = conv2d_same(residual, depth_bottleneckj 3, stride, scope='conv2*)

residual = slim.conv2d(residual_, depth, [1^ 1], stride=lj

normalizer_fn=None, activation_fn=Nonej scope=*conv3*)

output = shortcut + residual

return slim.utils.collect_named_outputs(outputs_collections, sc.name, output)

下面定义生成 ResNet V2的主函数，我们只要预先定义好网络的残差学习模块组 blocks，它就可以生成对应的完整的 ResNet。先来看一下这个函数的参数，inputs即输入， blocks为定义好的 Block 类的列表，num_classes是最后输出的类数，global_pool标志是 否加上最后的一层全局平均池化，include_root_block标志是否加上 ResNet 网络最前面通 常使用的 7x7 卷积和最大池化，reuse标志是否重用，scope是整个网络的名称。在函数体 内，我们先定义好 variable_scope 及 end_points_collection，再通过 slim.arg_scope 将 (slim.con2d、bottleneck、stack_block_dense )这三个函数的参数 outputs_collections 默认设 为 end_points_collectiono 然后根据 include_root_block 标记，创建 ResNet 最前面的 64 输 出通道的步长为 2 的 7x7 卷积，然后再接一个步长为 2 的 3x3 最大池化。经历两个步长 为 2 的层，图片尺寸已经被缩小为 1/4。然后，使用前面定义的 Stack_bl0CkS_denSe将残差 学习模块组生成好，再根据标记添加全局平均池化层，这里用 tf.reduCe_mean实现全局平 均池化，效率比直接用 avg_p00l高。下面根据是否有分类数，添加一个输出通道为 num_classes的 1x1 卷积(该卷积层无激活函数和正则项)，再添加一个 Softmax 层输出网 络结果。同时使用 slim.utils.convert_collection_to_dict 将 collection 转化为 python 的 diet, 最后返回 net 和 end_points0

def resnet_v2(inputs, blocks

num_classes=None? global一 pool=T rue include_root_block=True^ reuse=None, scope=None):

with tf.variable_scope(scopej Tresnet_v2’[inputs]reuse=reuse) as sc: end_points_collection = sc.original_name_scope + *_end_points' with slim.arg_scope([slim.conv2d, bottleneckj

stack_blocks_dense],

outputs_collections=end一 points 一 collection):

net = inputs if include一 root_block:

with slim.arg_scope([slim.conv2d]^ activation_fn=NoneJ normalizer_fn=None):

net = conv2d_same(netj 64, 7, stride=2, scope=,convl,) net = slim.max_pool2d(netJ [3, stride=2_, scope= * pooll')

net = stack_blocks_dense(netJ blocks)

net = slim.batch_norm(netactivation_fn=tf .nn.relu, scope=' postnorm') if global_pool:

net = tf.reduce_mean(netj [1, 2], name='pool5 * keep_dims=True) if num_classes is not None:

net = slim.conv2d(net, num_classes4 [1_» 1]activation_fn=Nonej normalizer_fn=None_, scope=* logits *)

end_points = slim.utils.convert_collection_to_dict( end_points_collection)

if num_classes is not None:

end_points[' predictions' ] = slim. softmax(netscope=1 predictions') return netj end_points

至此，我们就将 ResNet 的生成函数定义好了，下面根据图 6-17中推荐的几个不同深 度的 ResNet 网络配置，来设计层数分别为 50、101、152和 200 的 ResNet。我们先来看 50层的 ResNet，其严格遵守了图 6-17所示的设置，4个残差学习 Blocks 的 units 数量分 别为 3、4、6和 3，总层数即为(3+4+6+3 ) x3+2=50。需要注意的是，残差学习模块之 前的卷积、池化已经将尺寸缩小了 4倍，我们前 3 个 Blocks 又都包含步长为 2 的层，因 此总尺寸缩小了 4x8=32倍，输入图片尺寸最后变为 224/32=7。和 Inception V3很像， ResNet不断使用步长为 2 的层来缩减尺寸，但同时输出通道数也在持续增加，最后达到 了 2048。

def resnet_v2_50(inputs

num_classes=Nonej global 一 pool=T rue reuse=None?

scope=•resnet_v2_50*):

blocks =[

Block(’blockl., bottleneck, [(256, 64、 1)] * 2 + [(256, 64, 2)]), BlockCblockZ1, bottleneck, [(512, 128} 1)] * 3 + [(512, 128, 2)]), BlockCblocks、bottleneck, [(1024, 256) 1)] * 5 + [(1024, 256, 2)]), Block('block4'bottleneck』 [(2048， 512， 1)] * 3)]

return resnet_v2(inputs3 blocks, num_classeSj globaVpoolj

include_root_block=Truej reuse=reusej scope= scope)

101层的 ResNet 和 50 层相比，主要变化就是把 4 个 Blocks 的 units 数量从 3、4、6、

3提升到了 3、4、23、3。即将第三个残差学习 Block 的 units 数增加到接近 4 倍。

def resnet_v2_101(inputs?

num_classes=Nonej global_pool=T rue_, reuse=None_,

scope=* resnet_v2_101'):

blocks =[

BlockCblockl*, bottleneck, [(256, 64, 1)] * 2 + [(256) 64, 2)]), Block( .blocks, bottleneck, [(512, 128, 1)] * 3 + [(512, 128) 2)]), Blocky blocks、bottleneck, [(1024, 256, 1)] * 22 + [(1024, 256, 2)]), Block('block4'bottleneck) [(2048) 512， 1)] * 3)]

return resnet_v2(inputs, blocks, num一 classes, global_pool_,

include_root_block=True> reuse=reuse, scope= scope)

然后 152 层的 ResNet，则是将第二个 Block 的 units 数提高到 8，将第三个 Block 的 units数提高到 36。Units数量提升的主要场所依然是第三个 Block。

def resnet_v2_152(inputs

num_classes=Nonej global_pool=T rue reuse=Nonej

scope='resnet_v2_152*):

blocks =[

BlockCblockl1, bottleneck： [(256, 64, 1)] * 2 + [(256) 64, 2)]), Block('block2', bottleneck, [(512, 128, 1)] * 7 + [(512, 128, 2)]), BlockCblockS1, bottleneck, [(1024, 256, 1)] * 35 + [(1024, 256, 2)]), BlockCblocks, bottleneck) [(2048, 512, 1)] * 3)]

return resnet_v2(inputSj blocks) num_classes_, global—pool,

include_root_block=True_, reuse=reusej scope=scope)

最后,200层的 ResNet 相比 152 层的 ResNet，没有继续提升第三个 Block 的 units 数， 而是将第二个 Block 的 units 数一下子提升到了 23。

def resnet_v2_200(inputs,

num_classes=NoneJ global_pool=T rue reuse=Nonej

scope:* resnet_v2_200'):

blocks =[

BlockCblockl1, bottleneck, [(256, 64) 1)] * 2 + [(256, 64, 2)]),

Block(丨 block2.、 bottleneck, [(512, 128, 1)] * 23 + [(512) 128, 2)]), BlockCblocks', bottleneck, [(1024, 256, 1)] * 35 + [(1024』 256) 2)]), Block('block4'j bottleneck, [(2048, 512^ 1)] * 3)]

return resnet_v2(inputsblocks, num_classesj global_pool_,

include_root_block=Truereuse=reusej scope= scope)

最后我们使用一直以来的评测函数 time_tensorflow_run，来测试 152 层深的 ResNet (即获得 ILSVRC 2015冠军的版本)的 forward 性能。图片尺寸回归到 AlexNet、VGGNet 的 224x224, batch size 为 32。我们将 is_training 这个 FLAG 置为 False，然后使用 resnet_v2_152创建网络，再由 time_tensorflow_run函数评测其 forward 性能。由于篇幅原 因，就不对其训练时的性能进行测试了，感兴趣的读者可以测试求解 ResNet 全部参数的 梯度所需要的时间。

batch_size = 32

height, width = 224, 224

inputs = tf.random_uniform((batch_sizej heightwidth, 3))

with slim.arg_scope(resnet_arg_scope(is_training=False)): netj end一 points = resnet_v2一 152(inputs, 1000)

init = tf.global_variables_initializer()

sess = tf.Session()

sess.run(init)

num_batches=100

time_tensopflow_run(sessj net, "Forward")

这里可以看到，虽然这个 ResNet 有 152 层深，但其 forward 计算耗时并没有特别夸 张，相比 VGGNet 和 Inception V3，大概只增加了 50%，每 batch 为 0.202秒。这说明 ResNet 也是一个实用的卷积神经网络结构，不仅支持超深网络的训练，同时在实际工业应用时也 有不差的 forward 性能。

| 2016-12-10 21:23:43.676945： | step 0j duration = | 0.202  |
| ---------------------------- | ------------------ | ------ |
| 2016-12-10 21:23:45.699069： | step 10j duration  | =0.203 |
| 2016-12-10 21:23:47.722190： | step 20j duration  | =0.203 |
| 2016-12-10 21:23:49.745069： | step 30j duration  | =0.202 |
| 2016-12-10 21:23:51.770527： | step 40duration    | =0.202 |

2016-12-10 21:23:53.797204：

2016-12-10 21:23:55.822281：

2016-12-10 21:23:57.848449：

2016-12-10 21:23:59.877548：

2016-12-10 21:24:01.906566:

2016-12-10 21:24:03.734302：

batch



step 50^ duration = 0.202

step 60^ duration = 0.203

step 70^ duration = 0.203

step 80, duration = 0.203

step 90j duration = 0.203

Forward across 100 steps, 0.203 +/- 0.000 sec /

本节我们完整地讲解了 ResNet的基本原理及其 TensorFlow 实现，也设计了一系列不 同深度的 ResNet。读者若感兴趣，可以自行探索不同深度、乃至不同残差单元结构的 ResNet的分类性能。例如，ResNet原论文中主要增加的是第二个和第三个 Block 的 units 数，读者可以尝试增加其余两个 Block 的 units 数，或者修改 bottleneck 单元中的 depth、 depth_bOttleneCk等参数，可对其参数设置的意义加深理解。ResNet可以算是深度学习中 一个里程碑式的突破，真正意义上支持了极深神经网络的训练。其网络结构值得反复思索， 如 Google 等已将其融合到自家的 Inception Net中，并取得了非常好的效果。相信 ResNet 的成功也会启发其他在深度学习领域研究的灵感。

k 156

[www.aibbt.com](http://www.aibbt.com) 让未来触手可及

##### 6.5卷积神经网络发展趋势

本节，我们简单回顾卷积神经网络的历史，图 6-18所示大致勾勒出最近几十年卷积 神经网络的发展方向 oPerceptron（感知机汗 1957 年由 Frank Resenblatt提出，而 Perceptron 不仅是卷积网络，也是神经网络的始祖。Neocognitron （神经认知机）是一种多层级的神 经网络，由日本科学家 Kunihiko Fukushima于 20 世纪 80 年代提出，具有一定程度的视 觉认知的功能，并直接启发了后来的卷积神经网络。LeNet-5由 CNN 之父 Yann LeCun 于 1997 年提出，首次提出了多层级联的卷积结构，可对手写数字进行有效识别。可以看 到前面这三次关于卷积神经网络的技术突破，间隔时间非常长，需要十余年甚至更久才出 现一次理论创新。而后于 2012 年，Hinton的学生 Alex 依靠 8 层深的卷积神经网络一举获 得了 ILSVRC 2012比赛的冠军，瞬间点燃了卷积神经网络研究的热潮。AlexNet成功应 用了 ReLU激活函数、Dropout、最大覆盖池化、LRN层、GPU加速等新技术，并启发了 后续更多的技术创新，卷积神经网络的研究从此进入快车道。

图 6-18卷积神经网络发展图

在 AlexNet 之后，我们可以将卷积神经网络的发展分为两类，一类是网络结构上的改 进调整（图 6-18中的左侧分支），另一类是网络深度的增加（图 6-18中的右侧分支）。2013 年，颜水成教授的 Network in Network工作首次发表，优化了卷积神经网络的结构，并 推广了 1x1的卷积结构。在改进卷积网络结构的工作中，后继者还有 2014 年的 Google Inception Net VI，提出了 Inception Module这个可以反复堆叠的高效的卷积网络结构， 并获得了当年 ILSVRC 比赛的冠军。2015年初的 Inception V2提出了 Batch Normalization, 大大加速了训练过程，并提升了网络性能。2015年年末的 Inception V3则继续优化了网 络结构，提出了 Factorization in Small Convolutions的思想，分解大尺寸卷积为多个小卷 积乃至一维卷积。而另一条分支上，许多研究工作则致力于加深网络层数，2014年， ILSVRC比赛的亚军 VGGNet 全程使用 3x3 的卷积，成功训练了深达 19 层的网络，当年 的季军 MSRA-Net也使用了非常深的网络。2015年，微软的 ResNet 成功训练了 152层深 的网络，一举拿下了当年 ILSVRC 比赛的冠军，top-5错误率降低至 3.46%。其后又更新 了 ResNet V2，增加了 Batch Normalization，并去除了激活层而使用 Identity Mapping 或 Preactivation，进一步提升了网络性能。此后，Inception ResNet V2 融合了 Inception Net 优良的网络结构，和 ResNet 训练极深网络的残差学习模块，集两个方向之长，取得了更

好的分类效果。

我们可以看到，自 AlexNet 于 2012 年提出后，深度学习领域的研究发展极其迅速， 基本上每年甚至每几个月都会出现新一代的技术。新的技术往往伴随着新的网络结构，更 深的网络的训练方法等，并在图像识别等领域不断创造新的准确率记录。至今，ILSVRC 比赛和卷积神经网络的研究依然处于高速发展期，CNN的技术日新月异。当然其中不可 忽视的推动力是，我们拥有了更快的 GPU 计算资源用以实验，以及非常方便的开源工具 （比如 TensorFlow）可以让研究人员快速地进行探索和尝试。在以前，研究人员如果没有 像 Alex 那样高超的编程实力能自己实现 cuda-convnet，可能都没办法设计 CNN 或者快速 地进行实验。现在有了 TensorFlow，研究人员和开发人员都可以简单而快速地设计神经 网络结构并进行研究、测试、部署乃至实用。
