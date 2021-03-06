
## GPU和并行技术——深度学习和计算视觉发展的加速器

<p align="center">
    <img width="70%" height="70%" src="http://images.iterate.site/blog/image/180830/e26k6ACkK4.png?imageslim">
</p>


前面提到过，深度学习的概念早就有了，早期制约其发展的因素是方方面面的，其中一个很重要的方面就是计算能力的限制。相对其他许多传统的机器学习方法，深度神经网络本身就是一个消耗计算量的大户。另一方面，由于多层神经网络本身极强的表达能力， 对数据量也提出了很高的要求。如图 1-8所示，一个普遍被接受的观点是，深度学习在数据量较少时，和传统算法差别不大，甚至有时候传统算法更胜一筹。而当数据量持续增加的情况下，传统的算法往 往会出现性能上的“饱和”，而深度学习则会随着 数据的增加持续提高性能。所以大数据和深度神经 网络的碰撞才擦出了今天深度学习的火花，而大数据之大更加大了对计算能力的需求。在 GPU 被广泛应用到深度学习训练之前，计算能力的低下限制了对算法的探索和实验，以及在海量数据上进行训练的可行性。

从 20 世纪 80 年代开始人们就开始使用专门的运算单元负责对三维模型形成的图像进行渲染。不过直到 1999 年 NVIDIA 发布 GeForce 256时，才正式提出了 GPU的概念。早期的 GPU 中，显卡的作用主要是渲染，但因为天然就很强的并行处理能力和少逻辑重计算 的属性，从 2000 年开始就有不少科研人员开始尝试用 GPU 来加速通用高密度、大吞吐量 的计算任务。2001 年，通用图形处理器(General-Purpose computing on GPU，GPGPU)的 概念被正式提出。2002年，多伦多大学的 James Fung发布了 OpenVIDIA，利用 GPU 实现 了一些计算机视觉库的加速，这是第一次正式将 GPU 用到了渲染以外的用途上。到了 2006 年，NVIDIA推出了利用 GPU 进行通用计算的平台 CUDA，让开发者不用再和着色器/ OpenGL打交道，而更专注于计算逻辑的实现。这时，GPU无论是在带宽还是浮点运算能 力上都已经接近同时期 CPU 能力的 10 倍，而 CUDA 的推出一下降低了 GPGPU编程的门槛，于是 CUDA 很快就流行开并成为了 GPU 通用计算的主流框架。后来深度学习诞生了，鉴于科研界对 GPU 计算的一贯偏爱，自然开始有人利用 GPU 进行深度网络的训练。之后 的事情前面也讲到了，GPU 助 Alex一战成名，同时也成为了训练深度神经网络的标配。

除了 NVIDIA, ATI (后被 AMD 收购)也是另一大 GPU 厂商。事实上 ATI 在 GPU 通用计算领域的探索比 NVIDIA 还早，但也许是因为投入程度不够，或是其他原因，被 NVIDIA 占尽先机，尤其是后来在深度学习领域。

说完了 GPU 领域的风云变幻，接下来看一些实际的问题：如何选购一块做深度学习的 GPU？一提到用于深度学习的 GPU，很多人立刻会想到 NVIDIA 的 Tesla 系列。实际上根 据使用场景和预算的不同，选择是可以很多样化的。NVIDIA主要有 3 个系列的显卡： GeForce、Quadro 和 Tesla。GeForce 面向游戏，Quadro 面向 3D 设计、专业图像和 CAD 等， 而 Tesla 则是面向科学计算。所以这里主要讨论一下 GeForce 和 Tesla 的区别。

GeForce 系列显卡面向游戏，所以性能要求是最高的，而精度上的限制就低很多，另外稳定性比 Tesla 也差很多。毕竟玩游戏的时候如果程序崩了也就丢个存档，但服务器要是崩了没准能“挂掉” 一个公司。当然从实际角度看最大的两个优点是：既可以进行深度学习又可以玩游戏；便宜。

Tesla 从诞生之初就是瞄准高精度科学运算。所以 Tesla 严格意义上来说不是块显卡， 而是计算加速卡。对于不带视频输出的 Tesla 卡而言，玩游戏是指望不上的。由于 Tesla 开始面向的主要是高性能计算，尤其是科学计算，在许多科学计算领域，如大气等物理过程 的模拟中对精度的要求非常高，所以 Tesla 的设计上双精度浮点数的能力比起 GeForce 系列强很多。如 GTX Titan 和 K40 两块卡，GTX Titan的单精度浮点数运算能力是 K40 的 1.5 倍，但是双精度浮点数运算能力却只有 K40 的不到 15%。不过从深度学习的角度来看，双精度显得不是那么必要，如经典的 AlexNet 就是两块 GTX 580 训练出来的。所以，2016 年开始，NVIDIA 也在 Tesla 系列里推出了 M 系列加速卡，专门针对深度学习进行了优化， 并且牺牲双精度运算能力而大幅提升了单精度运算的性能。前面也提到了，除了精度，Tesla 主要面向工作站和服务器，所以稳定性特别好，同时也会有很多针对服务器的优化，如高 端的 Tesla 卡上的 GPUDirect 技术可以支持远程直接内存访问(Remote Direct Memory Access, RDMA)用来提升节点间数据交互的效率。当然，Tesla系列有一个最大的特点就是贵。

综上所述，如果是在大规模集群上进行深度学习研发和部署，Tesla系列是首选，尤其是 M 和 P 子系列是不二之选。单机上开发的话，“土豪”或者追求稳定性高的人就选择 Tesla。而最有性价比且能兼顾日常使用的选择则是 GeForce。<span style="color:red;">嗯。</span>
