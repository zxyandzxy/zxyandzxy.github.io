---
layout: post
title: "线性神经网络"
date: 2024-9-17
tags: [deep-learning]
comments: true
author: zxy
---

## 线性回归

[3.1. 线性回归 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_linear-networks/linear-regression.html#)

线性回归的基本元素（线性模型，损失函数，解析解，随机梯度下降求解最佳参数，用模型进行预测）

pytorch矢量化加速样本运算，替代掉for循环（就是利用多CPU,GPU性能）

正态分布与平方损失（噪声的正态分布带来的结果就是 对线性模型的最大似然估计  = 最小化均方误差损失函数）

线性回归与深度网络（线性模型就是只有一层的神经网络）

## 线性回归的从零开始实现

[3.2. 线性回归的从零开始实现 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_linear-networks/linear-regression-scratch.html)

1. 生成数据集：先定义好最终结果的线性模型（已知权重和偏置），正态分布随机生成一些样本点X
2. 读取数据集：随机小批量读取原数据集
3. 初始化模型参数：随机初始化权重和偏置（和最终结果不同）
4. 定义模型，定义损失函数，定义小批量梯度下降优化方法
5. 训练模型，比对最终训练参数和结果参数的距离

## 线性回归的简洁实现

[3.3. 线性回归的简洁实现 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_linear-networks/linear-regression-concise.html)

利用pytorch提供的高级API，快速实现生成数据集，读取数据集，定义参数，损失函数，训练模型的过程

## softmax回归

[3.4. softmax回归 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_linear-networks/softmax-regression.html)

**分类问题**：如果类别间有一些自然顺序， 那么将这个问题转变为回归问题，并且保留这种格式{1， 2， 3}是有意义的。但是一般的分类问题并不与类别之间的自然顺序有关。 幸运的是，统计学家很早以前就发明了一种表示分类数据的简单方法：*独热编码*（one-hot encoding）。 独热编码是一个向量，它的分量和类别一样多。 类别对应的分量设置为1，其他所有分量设置为0。 

**网络架构**：为了解决线性模型的分类问题，我们需要和输出一样多的*仿射函数*（affine function）。 每个输出对应于它自己的仿射函数。然后构建单层神经网络。

![../_images/softmaxreg.svg](https://zh-v2.d2l.ai/_images/softmaxreg.svg)

**全连接层的参数开销**：比如上面的图，就有3 * 4 + 3= 15 个参数

**softmax运算**：softmax函数能够将未规范化的预测变换为非负数并且总和为1，同时让模型保持 可导的性质。 为了完成这一目标，我们首先对每个未规范化的预测求幂，这样可以确保输出非负。 为了确保最终输出的概率值总和为1，我们再让每个求幂后的结果除以它们的总和。如下式：
$$
\hat{\mathbf{y}} = \mathrm{softmax}(\mathbf{o})\quad \text{其中}\quad \hat{y}_j = \frac{\exp(o_j)}{\sum_k \exp(o_k)}
$$
尽管softmax是一个非线性函数，但softmax回归的输出仍然由输入特征的仿射变换决定。 因此，softmax回归是一个*线性模型*（linear model）。

**小批量样本矢量化**：为了提高计算效率并且充分利用GPU，我们通常会对小批量样本的数据执行矢量计算。 

**损失函数**：*交叉熵损失*（cross-entropy loss），由对数似然进行推导，当预测正确时，这个损失函数不会再减少
$$
l(\mathbf{y}, \hat{\mathbf{y}}) = - \sum_{j=1}^q y_j \log \hat{y}_j.
$$
**softmax及其导数**：导数是我们softmax模型分配的概率与实际发生的情况（由独热标签向量表示）之间的差异。 从这个意义上讲，这与我们在回归中看到的非常相似， 其中梯度是观测值y和估计值y^之间的差异。
$$
\begin{split}\begin{aligned}
l(\mathbf{y}, \hat{\mathbf{y}}) &=  - \sum_{j=1}^q y_j \log \frac{\exp(o_j)}{\sum_{k=1}^q \exp(o_k)} \\
&= \sum_{j=1}^q y_j \log \sum_{k=1}^q \exp(o_k) - \sum_{j=1}^q y_j o_j\\
&= \log \sum_{k=1}^q \exp(o_k) - \sum_{j=1}^q y_j o_j.
\end{aligned}\end{split}
$$

$$
\partial_{o_j} l(\mathbf{y}, \hat{\mathbf{y}}) = \frac{\exp(o_j)}{\sum_{k=1}^q \exp(o_k)} - y_j = \mathrm{softmax}(\mathbf{o})_j - y_j.
$$
**信息论基础**：信息论的核心思想是量化数据中的信息内容，在信息论中，该数值被称为分布P的*熵*（entropy）。可以通过以下方程得到：
$$
H[P] = \sum_j - P(j) \log P(j).
$$
压缩与预测有什么关系呢？ 想象一下，我们有一个要压缩的数据流。 如果我们很容易预测下一个数据，那么这个数据就很容易压缩。 为什么呢？ 举一个极端的例子，假如数据流中的每个数据完全相同，这会是一个非常无聊的数据流。 由于它们总是相同的，我们总是知道下一个数据是什么。 所以，为了传递数据流的内容，我们不必传输任何信息。也就是说，“下一个数据是xx”这个事件毫无信息量。

熵， 是当分配的概率真正匹配数据生成过程时的*信息量的期望*。可以把交叉熵想象为“主观概率为Q的观察者在看到根据概率P生成的数据时的预期惊异”。 当P=Q时，交叉熵达到最低。 在这种情况下，从P到Q的交叉熵是H(P,P)=H(P)。

**模型预测和评估**：在训练softmax回归模型后，给出任何样本特征，我们可以预测每个输出类别的概率。 通常我们使用预测概率最高的类别作为输出类别。 如果预测与实际类别（标签）一致，则预测是正确的。 在接下来的实验中，我们将使用精度（accuracy）来评估模型的性能。 精度等于正确预测数与预测总数之间的比率

## 图像分类数据集

[3.5. 图像分类数据集 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_linear-networks/image-classification-dataset.html)

Fashion-MNIST图像分类数据集，可以利用数据迭代器小批量读取数据（依靠数据迭代器可以加速读取过程，torch自带的）

## softmax回归的从零开始实现

[3.6. softmax回归的从零开始实现 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_linear-networks/softmax-regression-scratch.html)

初始化模型参数 + 定义softmax操作 + 定义模型 + 定义损失函数 + 分类精度 + 训练与预测

## softmax回归的简洁实现

利用pytorch提供的高级API，可以简化代码的实现。

初始化模型参数，通过数学上的等价推导，解决未规范化的Oi经过指数运算后可能产生的数值上溢和下溢的问题
