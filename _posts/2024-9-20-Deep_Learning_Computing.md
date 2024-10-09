---
layout: post
title: "深度学习计算"
date: 2024-9-20
tags: [deep-learning]
comments: true
author: zxy
---

## 层与块

[5.1. 层和块 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_deep-learning-computation/model-construction.html)

*块*（block）可以描述单个层、由多个层组成的组件或整个模型本身。 使用块进行抽象的一个好处是可以将一些块组合成更大的组件， 这一过程通常是递归的，如图所示。 通过定义代码来按需生成任意复杂度的块， 我们可以通过简洁的代码实现复杂的神经网络。

![../_images/blocks.svg](https://zh-v2.d2l.ai/_images/blocks.svg)

神经网络可以在构成函数中定义块，也可以在前向传播函数中定义块，这取决于是不是神经网络需不需要处理复杂的前向传播逻辑。

- 一个块可以由许多层组成；一个块可以由许多块组成。
- 块可以包含代码。
- 块负责大量的内部处理，包括参数初始化和反向传播。
- 层和块的顺序连接由`Sequential`块处理。

## 参数管理

[5.2. 参数管理 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_deep-learning-computation/parameters.html)

**参数访问：**

- 直接通过索引拿目标参数：我们从已有模型中访问参数。 当通过Sequential类定义模型时， 我们可以通过索引来访问模型的任意层。 这就像模型是一个列表一样，每层的参数都在其属性中。 参数是复合的对象，包含值、梯度和额外信息。 这就是我们需要显式参数值的原因。 除了值之外，我们还可以访问每个参数的梯度。 在上面这个网络中，由于我们还没有调用反向传播，所以参数的梯度处于初始状态。
- 一次访问所有参数：通过Python自带的遍历`print(*[(name, param.shape) for name, param in net.named_parameters()])`访问所有层的参数
- 从嵌套层参数：上一节讲过，可以通过层与块嵌套构建网络。我们也可以通过多级索引（像三维数组访问一样）访问每一层的参数

**参数初始化：**

- pytorch提供了多种参数初始化API：高斯分布初始化，常量初始化，自定义初始化方法，对不同层利用不同初始化方法
- 通过指定某个参数的值，直接对它进行初始化

**参数绑定：**

希望在多个层间共享参数： 我们可以定义一个稠密层，然后使用它的参数来设置另一个层的参数。

```python
# 我们需要给共享层一个名称，以便可以引用它的参数
shared = nn.Linear(8, 8)
net = nn.Sequential(nn.Linear(4, 8), nn.ReLU(),
                    shared, nn.ReLU(),
                    shared, nn.ReLU(),
                    nn.Linear(8, 1))
net(X)
# 检查参数是否相同
print(net[2].weight.data[0] == net[4].weight.data[0]) # tensor([True, True, True, True, True, True, True, True])
net[2].weight.data[0, 0] = 100
# 确保它们实际上是同一个对象，而不只是有相同的值
print(net[2].weight.data[0] == net[4].weight.data[0]) # tensor([True, True, True, True, True, True, True, True])  
```

这个例子表明第三个和第五个神经网络层的参数是绑定的。 它们不仅值相等，而且由相同的张量表示。 因此，如果我们改变其中一个参数，另一个参数也会改变。 这里有一个问题：当参数绑定时，梯度会发生什么情况？ 答案是由于模型参数包含梯度，因此在反向传播期间第二个隐藏层 （即第三个神经网络层）和第三个隐藏层（即第五个神经网络层）的梯度会加在一起。

## 延后初始化

[5.3. 延后初始化 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_deep-learning-computation/deferred-init.html)

如果我们在定义网络时，我们需要出现以下问题：

- 我们定义了网络架构，但没有指定输入维度。
- 我们添加层时没有指定前一层的输出维度。
- 我们在初始化参数时，甚至没有足够的信息来确定模型应该包含多少参数。

正常来说，此时代码是无法运行的。但是框架会自动帮我们解决。

这里的诀窍是框架的*延后初始化*（defers initialization）， 即直到数据第一次通过模型传递时，框架才会动态地推断出每个层的大小。

在我们输入第一层数据后，框架可以识别出第一层的形状后，框架处理第二层，依此类推，直到所有形状都已知为止。 注意，在这种情况下，只有第一层需要延迟初始化，但是框架仍是按顺序初始化的。 等到知道了所有的参数形状，框架就可以初始化参数。

## 自定义层

[5.4. 自定义层 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_deep-learning-computation/custom-layer.html)

- 我们可以通过基本层类设计自定义层。这允许我们定义灵活的新层，其行为与深度学习框架中的任何现有层不同。
- 在自定义层定义完成后，我们就可以在任意环境和网络架构中调用该自定义层。
- 层可以有局部参数，这些参数可以通过内置函数创建。

## 读写文件

[5.5. 读写文件 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_deep-learning-computation/read-write.html)

对于单个张量，我们可以直接调用load和save函数分别读写它们。 这两个函数都要求我们提供一个名称，save要求将要保存的变量作为输入。

深度学习框架提供了内置函数来保存和加载整个网络。 需要注意的一个重要细节是，这将保存模型的参数而不是保存整个模型。 例如，如果我们有一个3层多层感知机，我们需要单独指定架构。 因为模型本身可以包含任意代码，所以模型本身难以序列化。 因此，为了恢复模型，我们需要用代码生成架构， 然后从磁盘加载参数。 

## GPU

[5.6. GPU — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_deep-learning-computation/use-gpu.html)

- 我们可以指定用于存储和计算的设备，例如CPU或GPU。默认情况下，数据在主内存中创建，然后使用CPU进行计算。
- 深度学习框架要求计算的所有输入数据都在同一设备上，无论是CPU还是GPU。
- 不经意地移动数据可能会显著降低性能。一个典型的错误如下：计算GPU上每个小批量的损失，并在命令行中将其报告给用户（或将其记录在NumPy `ndarray`中）时，将触发全局解释器锁，从而使所有GPU阻塞。最好是为GPU内部的日志分配内存，并且只移动较大的日志。
