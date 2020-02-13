---
title: 02 OverFeat
toc: true
date: 2018-09-22
---
# OverFeat

2013年 Yann Lecun在纽约大学的团队提出了著名的 OverFeat 算法，其利用滑动窗口和规则块生成候选框，再利用多尺度滑动窗口增加检测结果，解决图像目标形状复杂、尺寸不一问题，最后利用卷积神经网络和回归模型分类、定位目标。该算法首次将分类、定位以及检测三个计算机视觉任务放在一起解决，获得同年 ILSVRC 2013任务 3（分类+定位）的冠军，但其很快就被同期的 R-CNN算法取代。

![](http://images.iterate.site/blog/image/180922/I16hbC6dj1.png?imageslim){ width=55% }
