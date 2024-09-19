---
layout: post
title: "预备知识"
date: 2024-9-16
tags: [deep-learning]
comments: true
author: zxy
---

## 数据操作

[2.1. 数据操作 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_preliminaries/ndarray.html#)

主要介绍pytorch中的张量的基本信息和基本操作API。

1. 运算符（加减乘除，平方，指数）
2. 广播机制：形状不同的张量，按照元素为单位进行运算
3. 索引与切片：通过Python中类似数组的切片形式拆解张量
4. 节省内存：原地更新变量，使得内存占用减小（id函数查看内存地址）
5. 与其他Python对象的相互转换：ndarray  tensor

## 数据预处理

[2.2. 数据预处理 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_preliminaries/pandas.html)

介绍利用pandas读取数据，然后处理缺失值，最后转化成tensor格式

## 线性代数

[2.3. 线性代数 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_preliminaries/linear-algebra.html)

讲解深度学习需要的线性代数基础知识

1. 标量，向量，矩阵，张量
2. 张量算法的基本性质
3. 降维与非降维求和
4. 点积，矩阵-向量积，矩阵-矩阵乘法，范数

## 微积分

[2.4. 微积分 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_preliminaries/calculus.html)

讲解微积分基础知识

1. 导数与微分，偏导数
2. 梯度与链式法则

## 自动微分

[2.5. 自动微分 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_preliminaries/autograd.html#)

讲解pytorch框架下，反向传播过程中，框架会帮助我们自动计算微分，也就是梯度

1. 非标量变量的反向传播
2. 分离计算：希望将某些计算移动到记录的计算图之外。 例如，假设`y`是作为`x`的函数计算的，而`z`则是作为`y`和`x`的函数计算的。 想象一下，我们想计算`z`关于`x`的梯度，但由于某种原因，希望将`y`视为一个常数， 并且只考虑到`x`在`y`被计算后发挥的作用。这里可以分离`y`来返回一个新变量`u`，该变量与`y`具有相同的值， 但丢弃计算图中如何计算`y`的任何信息。 换句话说，梯度不会向后流经`u`到`x`。 因此，下面的反向传播函数计算`z=u*x`关于`x`的偏导数，同时将`u`作为常数处理， 而不是`z=x*x*x`关于`x`的偏导数。
3. Python控制流的梯度计算：自动微分的一个好处是： 即使构建函数的计算图需要通过Python控制流（例如，条件、循环或任意函数调用）

## 概率

[2.6. 概率 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_preliminaries/probability.html)

讲解概率的基本知识

1. 概率论公理和随机变量
2. 处理多个随机变量（联合概率 + 条件概率 + 贝叶斯定理 + 边际化 + 独立性）
3. 期望与方差
