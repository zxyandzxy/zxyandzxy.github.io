---
layout: post
title: "多层感知机"
date: 2024-9-18
tags: [deep-learning]
comments: true
author: zxy
---

## 多层感知机

[4.1. 多层感知机 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_multilayer-perceptrons/mlp.html)

隐藏层：线性模型只有一层；现实中的很多问题需要建模出的数学函数不一定是线性的，所有单纯使用一层神经网络拟合效果很差。所以可以加入很多隐藏层。通过在每个隐藏层中加入激活函数，提高神经网络对于非线性函数的拟合能力

激活函数：ReLU，sigmoid，tanh

## 多层感知机的从零开始实现

[4.2. 多层感知机的从零开始实现 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_multilayer-perceptrons/mlp-scratch.html)

介绍用pytorch的高级API如何实现一个只有1个隐藏层的多层感知机（无激活函数）

## 多层感知机的简洁实现

[4.3. 多层感知机的简洁实现 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_multilayer-perceptrons/mlp-concise.html#)

介绍用pytorch的高级API如何实现一个只有1个隐藏层的多层感知机（带激活函数）

## 模型选择、欠拟合和过拟合

[4.4. 模型选择、欠拟合和过拟合 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_multilayer-perceptrons/underfit-overfit.html#)

### 训练误差与泛化误差

我们的目标是发现某些模式， 这些模式捕捉到了我们训练集潜在总体的规律。 如果成功做到了这点，即使是对以前从未遇到过的个体， 模型也可以成功地评估风险。 如何发现可以泛化的模式是机器学习的根本问题。

将模型在训练数据上拟合的比在潜在分布中更接近的现象称为*过拟合*（overfitting）， 用于对抗过拟合的技术称为*正则化*（regularization）。 

泛化误差（generalization error）是指， 模型应用在同样从原始样本的分布中抽取的无限多数据样本时，模型误差的期望。

> 统计学习理论

假设训练数据和测试数据都是从相同的分布中独立提取的。 这通常被称为独立同分布假设（i.i.d. assumption）， 这意味着对数据进行采样的过程没有进行“记忆”。 换句话说，抽取的第2个样本和第3个样本的相关性， 并不比抽取的第2个样本和第200万个样本的相关性更强。

有时候我们即使轻微违背独立同分布假设，模型仍将继续运行得非常好。

有些违背独立同分布假设的行为肯定会带来麻烦。

> 模型复杂性

一个模型是否能很好地泛化取决于很多因素。

1. 可调整参数的数量。当可调整参数的数量（有时称为自由度）很大时，模型往往更容易过拟合。

2. 参数采用的值。当权重的取值范围较大时，模型可能更容易过拟合。

3. 训练样本的数量。即使模型很简单，也很容易过拟合只包含一两个样本的数据集。而过拟合一个有数百万个样本的数据集则需要一个极其灵活的模型。

### 模型选择

> 验证集

原则上，在我们确定所有的超参数之前，我们不希望用到测试集。 如果我们在模型选择过程中使用测试数据，可能会有过拟合测试数据的风险，那就麻烦大了。 如果我们过拟合了训练数据，还可以在测试数据上的评估来判断过拟合。 但是如果我们过拟合了测试数据，我们又该怎么知道呢？

解决此问题的常见做法是将我们的数据分成三份， 除了训练和测试数据集之外，还增加一个*验证数据集*（validation dataset）， 也叫*验证集*（validation set）。

> K折交叉验证

当训练数据稀缺时，我们甚至可能无法提供足够的数据来构成一个合适的验证集。 这个问题的一个流行的解决方案是采用K*折交叉验证*。 这里，原始训练数据被分成K个不重叠的子集。 然后执行K次模型训练和验证，每次在K−1个子集上进行训练， 并在剩余的一个子集（在该轮中没有用于训练的子集）上进行验证。 最后，通过对K次实验的结果取平均来估计训练和验证误差。

### 欠拟合与过拟合

> 模型复杂性

高阶多项式的参数较多，模型函数的选择范围较广。 因此在固定训练数据集的情况下， 高阶多项式函数相对于低阶多项式的训练误差应该始终更低（最坏也是相等）。 事实上，当数据样本包含了x的不同值时， 函数阶数等于数据样本数量的多项式函数可以完美拟合训练集。 在 [图4.4.1](https://zh-v2.d2l.ai/chapter_multilayer-perceptrons/underfit-overfit.html#fig-capacity-vs-error)中， 我们直观地描述了多项式的阶数和欠拟合与过拟合之间的关系。

![../_images/capacity-vs-error.svg](https://zh-v2.d2l.ai/_images/capacity-vs-error.svg)

>数据集大小

另一个重要因素是数据集的大小。 训练数据集中的样本越少，我们就越有可能（且更严重地）过拟合。 随着训练数据量的增加，泛化误差通常会减小。 此外，一般来说，更多的数据不会有什么坏处。 对于固定的任务和数据分布，模型复杂性和数据集大小之间通常存在关系。 给出更多的数据，我们可能会尝试拟合一个更复杂的模型。 能够拟合更复杂的模型可能是有益的。 如果没有足够的数据，简单的模型可能更有用。 对于许多任务，深度学习只有在有数千个训练样本时才优于线性模型。

## 权重衰减

[4.5. 权重衰减 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_multilayer-perceptrons/weight-decay.html)

我们总是可以通过去收集更多的训练数据来缓解过拟合。 但这可能成本很高，耗时颇多，或者完全超出我们的控制，因而在短期内不可能做到。 假设我们已经拥有尽可能多的高质量数据，我们便可以将重点放在正则化技术上。

在训练参数化机器学习模型时， *权重衰减*（weight decay）是最广泛使用的正则化的技术之一， 它通常也被称为L2*正则化*。 这项技术通过函数与零的距离来衡量函数的复杂度， 因为在所有函数f中，函数f=0（所有输入都得到值0） 在某种意义上是最简单的。

最常用方法是将其范数作为惩罚项加到最小化损失的问题中。 将原来的训练目标*最小化训练标签上的预测损失*， 调整为*最小化预测损失和惩罚项之和*。 现在，如果我们的权重向量增长的太大， 我们的学习算法可能会更集中于最小化权重范数‖w‖2。 这正是我们想要的。
$$
L(\mathbf{w}, b) = \frac{1}{n}\sum_{i=1}^n \frac{1}{2}\left(\mathbf{w}^\top \mathbf{x}^{(i)} + b - y^{(i)}\right)^2.
$$

$$
L(\mathbf{w}, b) + \frac{\lambda}{2} \|\mathbf{w}\|^2,
$$

为什么在这里我们使用平方范数而不是标准范数（即欧几里得距离）？ 我们这样做是为了便于计算。 通过平方L2范数，我们去掉平方根，留下权重向量每个分量的平方和。 这使得惩罚的导数很容易计算：导数的和等于和的导数。

我们仅考虑惩罚项，优化算法在训练的每一步衰减权重。 与特征选择相比，权重衰减为我们提供了一种连续的机制来调整函数的复杂度。

## 暂退法

[4.6. 暂退法（Dropout） — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_multilayer-perceptrons/dropout.html)

当面对更多的特征而样本不足时，线性模型往往会过拟合。 相反，当给出更多样本而不是特征，通常线性模型不会过拟合。 不幸的是，线性模型泛化的可靠性是有代价的。 简单地说，线性模型没有考虑到特征之间的交互作用。 对于每个特征，线性模型必须指定正的或负的权重，而忽略其他特征。

泛化性和灵活性之间的这种基本权衡被描述为偏差-方差权衡（bias-variance tradeoff）。 线性模型有很高的偏差：它们只能表示一小类函数。 然而，这些模型的方差很低：它们在不同的随机数据样本上可以得出相似的结果。

深度神经网络位于偏差-方差谱的另一端。 与线性模型不同，神经网络并不局限于单独查看每个特征，而是学习特征之间的交互。

> 扰动的稳健性

经典泛化理论认为，为了缩小训练和测试性能之间的差距，应该以简单的模型为目标。 简单性以较小维度的形式展现，参数的范数也代表了一种有用的简单性度量。

简单性的另一个角度是平滑性，即函数不应该对其输入的微小变化敏感。在训练过程中，他们建议在计算后续层之前向网络的每一层注入噪声。 因为当训练一个有多层的深层网络时，注入噪声只会在输入-输出映射上增强平滑性。

这个想法被称为*暂退法*（dropout）。 暂退法在前向传播过程中，计算每一内部层的同时注入噪声，这已经成为训练神经网络的常用技术。 这种方法之所以被称为*暂退法*，因为我们从表面上看是在训练过程中丢弃（drop out）一些神经元。 在整个训练过程的每一次迭代中，标准暂退法包括在计算下一层之前将当前层中的一些节点置零。

> 神经网络中如何使用暂退法

通常，我们在测试时不用暂退法。 给定一个训练好的模型和一个新的样本，我们不会丢弃任何节点，因此不需要标准化。 然而也有一些例外：一些研究人员在测试时使用暂退法， 用于估计神经网络预测的“不确定性”： 如果通过许多不同的暂退法遮盖后得到的预测结果都是一致的，那么我们可以说网络发挥更稳定。

我们可以将暂退法应用于每个隐藏层的输出（在激活函数之后）， 并且可以为每一层分别设置暂退概率： 常见的技巧是在靠近输入层的地方设置较低的暂退概率。 

## 前向传播、反向传播和计算图

[4.7. 前向传播、反向传播和计算图 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_multilayer-perceptrons/backprop.html)

前向传播（forward propagation或forward pass） 指的是：按顺序（从输入层到输出层）计算和存储神经网络中每层的结果。

![../_images/forward.svg](https://zh-v2.d2l.ai/_images/forward.svg)

反向传播（backward propagation或backpropagation）指的是计算神经网络参数梯度的方法。 简言之，该方法根据微积分中的链式规则，按相反的顺序从输出层到输入层遍历网络。 该算法存储了计算某些参数梯度时所需的任何中间变量（偏导数）。

我们使用prod运算符在执行必要的操作（如换位和交换输入位置）后将其参数相乘。 对于向量，这很简单，它只是矩阵-矩阵乘法。 对于高维张量，我们使用适当的对应项。 运算符prod指代了所有的这些符号。
$$
\frac{\partial J}{\partial \mathbf{W}^{(2)}}= \text{prod}\left(\frac{\partial J}{\partial \mathbf{o}}, \frac{\partial \mathbf{o}}{\partial \mathbf{W}^{(2)}}\right) + \text{prod}\left(\frac{\partial J}{\partial s}, \frac{\partial s}{\partial \mathbf{W}^{(2)}}\right)= \frac{\partial J}{\partial \mathbf{o}} \mathbf{h}^\top + \lambda \mathbf{W}^{(2)}.
$$

$$
\frac{\partial J}{\partial \mathbf{W}^{(1)}}
= \text{prod}\left(\frac{\partial J}{\partial \mathbf{z}}, \frac{\partial \mathbf{z}}{\partial \mathbf{W}^{(1)}}\right) + \text{prod}\left(\frac{\partial J}{\partial s}, \frac{\partial s}{\partial \mathbf{W}^{(1)}}\right)
= \frac{\partial J}{\partial \mathbf{z}} \mathbf{x}^\top + \lambda \mathbf{W}^{(1)}.
$$

在训练神经网络时，前向传播和反向传播相互依赖。 对于前向传播，我们沿着依赖的方向遍历计算图并计算其路径上的所有变量。 然后将这些用于反向传播，其中计算顺序与计算图的相反。

因此，在训练神经网络时，在初始化模型参数后， 我们交替使用前向传播和反向传播，利用反向传播给出的梯度来更新模型参数。 注意，反向传播重复利用前向传播中存储的中间值，以避免重复计算。 带来的影响之一是我们需要保留中间值，直到反向传播完成。 这也是训练比单纯的预测需要更多的内存（显存）的原因之一。 此外，这些中间值的大小与网络层的数量和批量的大小大致成正比。 因此，使用更大的批量来训练更深层次的网络更容易导致*内存不足*（out of memory）错误。

## 数值稳定性和模型初始化

[4.8. 数值稳定性和模型初始化 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_multilayer-perceptrons/numerical-stability-and-init.html)

在神经网络中，选择哪个非线性函数以及如何初始化参数可以决定优化算法收敛的速度有多快。

梯度爆炸和梯度消失：不稳定梯度带来的风险不止在于数值表示； 不稳定梯度也威胁到我们优化算法的稳定性。 我们可能面临一些问题。 要么是梯度爆炸（gradient exploding）问题： 参数更新过大，破坏了模型的稳定收敛； 要么是梯度消失（gradient vanishing）问题： 参数更新过小，在每次更新时几乎不会移动，导致模型无法学习。

打破对称性：在隐藏层的神经元中，如果参数初始化相同，那么由于是全连接的，输入相同，转换函数相同，那么多个隐藏层神经元就是等价的。这样的迭代永远不会打破对称性，我们可能永远也无法实现网络的表达能力。 隐藏层的行为就好像只有一个单元。 请注意，虽然小批量随机梯度下降不会打破这种对称性，但暂退法正则化可以。

参数初始化：

- 不指定初始化方法，框架会有默认的初始化方法（比如高斯分布参数）。
- 不考虑非线性层时，如果想要全连接层输出的方差不大，保证梯度的稳定，那么可以通过Xavier初始化参数

$$
\begin{aligned}
\frac{1}{2} (n_\mathrm{in} + n_\mathrm{out}) \sigma^2 = 1 \text{ 或等价于 }
\sigma = \sqrt{\frac{2}{n_\mathrm{in} + n_\mathrm{out}}}.
\end{aligned}
$$



## 环境和分布偏移

[4.9. 环境和分布偏移 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_multilayer-perceptrons/environment.html)

- 在许多情况下，训练集和测试集并不来自同一个分布。这就是所谓的分布偏移。
- 真实风险是从真实分布中抽取的所有数据的总体损失的预期。然而，这个数据总体通常是无法获得的。经验风险是训练数据的平均损失，用于近似真实风险。在实践中，我们进行经验风险最小化。
- 在相应的假设条件下，可以在测试时检测并纠正协变量偏移和标签偏移。在测试时，不考虑这种偏移可能会成为问题。
- 在某些情况下，环境可能会记住自动操作并以令人惊讶的方式做出响应。在构建模型时，我们必须考虑到这种可能性，并继续监控实时系统，并对我们的模型和环境以意想不到的方式纠缠在一起的可能性持开放态度。

## 实战Kaggle：预测放假

[4.10. 实战Kaggle比赛：预测房价 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_multilayer-perceptrons/kaggle-house-price.html)

- 真实数据通常混合了不同的数据类型，需要进行预处理。
- 常用的预处理方法：将实值数据重新缩放为零均值和单位方法；用均值替换缺失值。
- 将类别特征转化为指标特征，可以使我们把这个特征当作一个独热向量来对待。
- 我们可以使用K折交叉验证来选择模型并调整超参数。
- 对数对于相对误差很有用。