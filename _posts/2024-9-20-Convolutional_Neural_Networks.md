---
layout: post
title: "卷积神经网络"
date: 2024-9-20
tags: [deep-learning]
comments: true
author: zxy
---

## 从全连接层到卷积

[6.1. 从全连接层到卷积 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_convolutional-neural-networks/why-conv.html)

- 图像的平移不变性使我们以相同的方式处理局部图像，而不在乎它的位置。
- 局部性意味着计算相应的隐藏表示只需一小部分局部图像像素。
- 在图像处理中，卷积层通常比全连接层需要更少的参数，但依旧获得高效用的模型。
- 卷积神经网络（CNN）是一类特殊的神经网络，它可以包含多个卷积层。
- 多个输入和输出通道使模型在每个空间位置可以获取图像的多方面特征。

## 图像卷积

[6.2. 图像卷积 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_convolutional-neural-networks/conv-layer.html)

- 二维卷积层的核心计算是二维互相关运算。最简单的形式是，对二维输入数据和卷积核执行互相关操作，然后添加一个偏置。
- 我们可以设计一个卷积核来检测图像的边缘。
- 我们可以从数据中学习卷积核的参数。
- 学习卷积核时，无论用严格卷积运算或互相关运算，卷积层的输出不会受太大影响。
- 当需要检测输入特征中更广区域时，我们可以构建一个更深的卷积网络。

## 填充和步幅

[6.3. 填充和步幅 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_convolutional-neural-networks/padding-and-strides.html)

- 填充可以增加输出的高度和宽度。这常用来使输出与输入具有相同的高和宽。
- 步幅可以减小输出的高和宽，例如输出的高和宽仅为输入的高和宽的1/n（n是一个大于1的整数）。
- 填充和步幅可用于有效地调整数据的维度。

## 多输入多输出通道

[6.4. 多输入多输出通道 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_convolutional-neural-networks/channels.html)

- 多输入多输出通道可以用来扩展卷积层的模型。
- 当以每像素为基础应用时，1×1卷积层相当于全连接层。
- 1×1卷积层通常用于调整网络层的通道数量和控制模型复杂性。
- [深度学习笔记（六）：1x1卷积核的作用归纳和实例分析_1x1卷积降维-CSDN博客](https://blog.csdn.net/qq_41554005/article/details/107392995)

## 汇聚层

[6.5. 汇聚层 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_convolutional-neural-networks/pooling.html)

- 对于给定输入元素，最大汇聚层会输出该窗口内的最大值，平均汇聚层会输出该窗口内的平均值。

- 汇聚层的主要优点之一是减轻卷积层对位置的过度敏感。

- 我们可以指定汇聚层的填充和步幅。

- 使用最大汇聚层以及大于1的步幅，可减少空间维度（如高度和宽度）。

- 汇聚层的输出通道数与输入通道数相同。

## 卷积神经网络（LeNet）

[6.6. 卷积神经网络（LeNet） — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_convolutional-neural-networks/lenet.html)

- 卷积神经网络（CNN）是一类使用卷积层的网络。
- 在卷积神经网络中，我们组合使用卷积层、非线性激活函数和汇聚层。
- 为了构造高性能的卷积神经网络，我们通常对卷积层进行排列，逐渐降低其表示的空间分辨率，同时增加通道数。
- 在传统的卷积神经网络中，卷积块编码得到的表征在输出之前需由一个或多个全连接层进行处理。
- LeNet是最早发布的卷积神经网络之一。
