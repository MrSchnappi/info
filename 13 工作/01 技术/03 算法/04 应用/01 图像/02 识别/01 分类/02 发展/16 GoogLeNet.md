# GoogLeNet


在分类的准确率上面相比过去两届冠军 ZFNet 和 AlexNet都有很大的提升。


最核心的部分是其内部子网络结构 Inception，该结构灵感来源于 NIN，至今已经经历了四次版本迭代（Inception$_{v1-4}$）。

Inception 性能比较图：

<p align="center">
    <img width="70%" height="70%" src="http://images.iterate.site/blog/image/20190722/jVuSjy0EFjRQ.png?imageslim">
</p>

<span style="color:red;">横坐标的 Operations 是什么意思？</span>

## 结构

GoogLeNet 网络结构图：

<p align="center">
    <img width="70%" height="70%" src="http://images.iterate.site/blog/image/20190722/I9pMUsIOeJI3.jpeg?imageslim">
</p>

如图 4.9中所示，GoogLeNet 相比于以前的卷积神经网络结构，除了在深度上进行了延伸，还对网络的宽度进行了扩展，整个网络由许多块状子网络的堆叠而成，这个子网络构成了 Inception 结构。

图 4.9 为 Inception 的四个版本：<span style="color:red;">想深入了解下。</span>

- $Inception_{v1}​$ 在同一层中采用不同的卷积核，并对卷积结果进行合并;
- $Inception_{v2}​$ 组合不同卷积核的堆叠形式，并对卷积结果进行合并;
- $Inception_{v3}​$ 则在 $v_2​$ 基础上进行深度组合的尝试;
- $Inception_{v4}​$ 结构相比于前面的版本更加复杂，子网络中嵌套着子网络。


$Inception_{v1}$

<p align="center">
    <img width="70%" height="70%" src="http://images.iterate.site/blog/image/20190722/6BLFHpihc0GY.png?imageslim">
</p>


<p align="center">
    <img width="70%" height="70%" src="http://images.iterate.site/blog/image/20190722/W6MFXPSfXQxH.png?imageslim">
</p>

<span style="color:red;">怎么合并不同的卷积结果的？怎么还有 max pooling ？也合并进来了吗？</span>

$Inception_{v2}$

<p align="center">
    <img width="70%" height="70%" src="http://images.iterate.site/blog/image/20190722/Qdxvn9AEePhw.png?imageslim">
</p>


<p align="center">
    <img width="70%" height="70%" src="http://images.iterate.site/blog/image/20190722/Xsp3lQ54osLM.png?imageslim">
</p>


<p align="center">
    <img width="70%" height="70%" src="http://images.iterate.site/blog/image/20190722/weVIuRRX44qV.png?imageslim">
</p>

<span style="color:red;">$n$ 是怎么定的？为什么上面的结构是合理的？</span>

$Inception_{v3}$

<p align="center">
    <img width="70%" height="70%" src="http://images.iterate.site/blog/image/20190722/HCaOfpofR4d1.png?imageslim">
</p>


<span style="color:red;">这个是什么？</span>


$Inception_{v4}$

<p align="center">
    <img width="70%" height="70%" src="http://images.iterate.site/blog/image/20190722/BBwO8qMwCayc.png?imageslim">
</p>


<p align="center">
    <img width="70%" height="70%" src="http://images.iterate.site/blog/image/20190722/SJ2WkJ7SeQ0j.jpg?imageslim">
</p>

<span style="color:red;">上面这个为什么是合理的？</span>


图 4.10 Inception$_{v1-4}$ 结构图：

<p align="center">
    <img width="70%" height="70%" src="http://images.iterate.site/blog/image/20190722/O2yqknELcK4y.png?imageslim">
</p>



表 4.6 GoogLeNet中 Inception$_{v1}$ 网络参数配置

|      网络层       |                      输入尺寸                      |         核尺寸          |                      输出尺寸                      |               参数个数               |
|:-----------------:|:--------------------------------------------------:|:-----------------------:|:--------------------------------------------------:|:------------------------------------:|
|  卷积层 $C_{11}$  |              $H\times{W}\times{C_1}$               | $1\times1\times{C_2}/2$ |     $\frac{H}{2}\times\frac{W}{2}\times{C_2}$      | $(1\times1\times{C_1}+1)\times{C_2}$ |
|  卷积层 $C_{21}$  |              $H\times{W}\times{C_2}$               | $1\times1\times{C_2}/2$ |     $\frac{H}{2}\times\frac{W}{2}\times{C_2}$      | $(1\times1\times{C_2}+1)\times{C_2}$ |
|  卷积层 $C_{22}$  |              $H\times{W}\times{C_2}$               | $3\times3\times{C_2}/1$ |             $H\times{W}\times{C_2}/1$              | $(3\times3\times{C_2}+1)\times{C_2}$ |
|  卷积层 $C_{31}$  |              $H\times{W}\times{C_1}$               | $1\times1\times{C_2}/2$ |     $\frac{H}{2}\times\frac{W}{2}\times{C_2}$      | $(1\times1\times{C_1}+1)\times{C_2}$ |
|  卷积层 $C_{32}$  |              $H\times{W}\times{C_2}$               | $5\times5\times{C_2}/1$ |             $H\times{W}\times{C_2}/1$              | $(5\times5\times{C_2}+1)\times{C_2}$ |
| 下采样层 $S_{41}$ |              $H\times{W}\times{C_1}$               |      $3\times3/2$       |     $\frac{H}{2}\times\frac{W}{2}\times{C_2}$      |                 $0$                  |
|  卷积层 $C_{42}$  |     $\frac{H}{2}\times\frac{W}{2}\times{C_2}$      | $1\times1\times{C_2}/1$ |     $\frac{H}{2}\times\frac{W}{2}\times{C_2}$      | $(3\times3\times{C_2}+1)\times{C_2}$ |
|    合并层 $M$     | $\frac{H}{2}\times\frac{W}{2}\times{C_2}(\times4)$ |          拼接           | $\frac{H}{2}\times\frac{W}{2}\times({C_2}\times4)$ |                 $0$                  |

## 特性

- 采用不同大小的卷积核意味着不同大小的感受野，最后拼接意味着不同尺度特征的融合；
- 之所以卷积核大小采用 1、3 和 5，主要是为了方便对齐。设定卷积步长 stride=1 之后，只要分别设定 pad=0、1、2，那么卷积之后便可以得到相同维度的特征，然后这些特征就可以直接拼接在一起了；<span style="color:red;">这么机智吗。。但是为什么这种拼接是合理的？而且，是以什么形式拼接的？</span>
- 网络越到后面，特征越抽象，而且每个特征所涉及的感受野也更大了，因此随着层数的增加，3x3和 5x5 卷积的比例也要增加。但是，使用 5x5 的卷积核仍然会带来巨大的计算量。 为此，文章借鉴 NIN2，采用 1x1 卷积核来进行降维。<span style="color:red;">嗯，1x1卷积的使用不会破坏掉什么吗？对这个卷积核的理解还不够。 </span>


