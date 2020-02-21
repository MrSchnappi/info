
# Fast R-CNN



目的：

- 针对 SPP-Net 解决的问题融合进来

地址：

- <http://www.cv-foundation.org/openaccess/content_iccv_2015/papers/Girshick_Fast_R-CNN_ICCV_2015_paper.pdf>

介绍:

- 借鉴 SPP-Net 算法结构，设计一种 ROI pooling 的池化层结构，有效解决 R-CNN 算法必须将图像区域剪裁、缩放到相同尺寸大小的操作。
- 提出多任务损失函数思想，将分类损失和边框回归损失结合统一训练学习，并输出对应分类和边框坐标，不再需要额外的硬盘空间来存储中间层的特征，梯度能够通过 RoI Pooling 层直接传播。

缺点：

- 仍然没有摆脱选择性搜索算法生成正负样本候选框的问题。

