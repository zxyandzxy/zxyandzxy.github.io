---
layout: post
title: "注意力机制的前世今生"
date: 2025-06-15
tags: [LLM]
comments: true
author: zxy
---

## Attention? Attention!

> 参考资料：[注意？注意！| Lil'Log --- Attention? Attention! | Lil'Log](https://lilianweng.github.io/posts/2018-06-24-attention/)

注意力在一定程度上是由我们如何将视觉注意力集中在图像的不同区域或关联句子中的单词所驱动的。以图中柴犬的照片为例。

<img src="https://lilianweng.github.io/posts/2018-06-24-attention/shiba-example-attention.png" alt="img" style="zoom: 25%;" />

人类的视觉注意力使我们能够聚焦于某个特定区域以“高分辨率”（即看向黄色框中的尖耳朵）观察，同时感知周围图像以“低分辨率”（即看看雪地背景和服装如何？），然后相应地调整焦点或进行推理。

给定图像的一小块区域，其余像素会提供线索说明那里应该显示什么。我们期望在黄色框中看到尖耳朵，因为我们已经看到了狗鼻子、右侧的另一个尖耳朵，以及柴犬的神秘眼睛（红色框中的内容）。然而，底部的毛衣和毯子可能不如那些狗狗特征那么有帮助。

类似地，我们也可以解释一个句子或邻近语境中词语之间的关系。当我们看到“吃”时，我们期望很快就会遇到一个食物相关的词语。颜色术语描述了食物，但可能并不直接与“吃”相关。

<img src="https://lilianweng.github.io/posts/2018-06-24-attention/sentence-example-attention.png" alt="img" style="zoom: 33%;" />

简而言之，深度学习中的注意力机制可以广泛地解释为一组重要性权重：**为了预测或推断一个元素**，例如图像中的一个像素或句子中的一个词，我们**使用注意力向量来估计它与**（或“关注”，正如你在许多论文中可能读到的）**其他元素的相关性强弱**，并将这些元素值乘以注意力向量的权重之和作为目标的近似值。

## Seq2Seq 模型有什么问题？

Seq2seq 模型诞生于语言建模领域（Sutskever 等人，2014 年）。广义而言，它的目标是**将一个输入序列（源）转换成一个新的序列（目标）**，且这两个序列可以是任意长度。转换任务的例子包括多语言文本或音频的机器翻译、问答对话生成，甚至将句子解析成语法树。

seq2seq 模型通常采用编码器-解码器架构，由以下部分组成：

- 编码器处理输入序列，并将信息压缩成一个固定长度的上下文向量（也称为句子嵌入或“思想”向量）。这种表示形式被期望能很好地概括整个源序列的含义。
- 解码器使用上下文向量来生成转换后的输出。早期的研究只使用编码器网络的最后一个状态作为解码器的初始状态。
- 编码器和解码器都是循环神经网络，即使用 LSTM 或 GRU 单元。

<img src="https://lilianweng.github.io/posts/2018-06-24-attention/encoder-decoder-example.png" alt="img" style="zoom:33%;" />

这种固定长度上下文向量设计的重大且明显的缺点是无法记住长句子。一旦处理完整个输入，它常常会忘记开头部分。注意力机制应运而生 ([Bahdanau et al., 2015](https://arxiv.org/pdf/1409.0473.pdf)) 以解决这个问题。

## Born for Translation

注意力机制诞生于帮助神经机器翻译（NMT）中记忆长源句。它并非通过编码器最后一个隐藏状态构建单个上下文向量。注意力机制发明的秘诀在于**创建上下文向量和整个源输入之间的捷径**。这些捷径连接的权重可以针对每个输出元素进行自定义。

由于上下文向量可以访问整个输入序列，我们无需担心遗忘问题。源序列和目标序列之间的对齐关系由上下文向量学习和控制。本质上，上下文向量消耗了三部分信息：

- 编码器隐藏状态；
- 解码器隐藏状态；
- 源序列和目标序列之间的对齐关系。

<img src="https://lilianweng.github.io/posts/2018-06-24-attention/encoder-decoder-attention.png" alt="img" style="zoom:33%;" />

## Definition

现在我们以科学的方式定义 NMT 中引入的注意力机制。假设我们有一个长度为 n 的源序列 x ，并尝试输出一个长度为 m 的目标序列 y ：
$$
\begin{aligned}
\mathbf{x} &= [x_1, x_2, \dots, x_n] \\
\mathbf{y} &= [y_1, y_2, \dots, y_m]
\end{aligned}
$$
编码器是一个双向 RNN（或其他你选择的循环网络设置），具有一个正向隐藏状态 h→i 和一个反向隐藏状态 h←i 。这两个状态的简单拼接代表了编码器状态。其动机是**在一个词的标注中包含其前后的词**。
$$
\boldsymbol{h}_i = [\overrightarrow{\boldsymbol{h}}_i^\top; \overleftarrow{\boldsymbol{h}}_i^\top]^\top, i=1,\dots,n
$$
解码器网络为输出位置 t 的输出词具有隐藏状态 st = f(st−1, yt−1, ct) ，其中上下文向量 ct 是输入序列的隐藏状态的加权和，权重由对齐分数决定：
$$
\begin{aligned}
\mathbf{c}_t &= \sum_{i=1}^n \alpha_{t,i} \boldsymbol{h}_i & \small{\text{; Context vector for output }y_t}\\
\alpha_{t,i} &= \text{align}(y_t, x_i) & \small{\text{; How well two words }y_t\text{ and }x_i\text{ are aligned.}}\\
&= \frac{\exp(\text{score}(\boldsymbol{s}_{t-1}, \boldsymbol{h}_i))}{\sum_{i'=1}^n \exp(\text{score}(\boldsymbol{s}_{t-1}, \boldsymbol{h}_{i'}))} & \small{\text{; Softmax of some predefined alignment score.}}.
\end{aligned}
$$
对齐模型根据输入位置 i 和输出位置 t 的输入对和输出对 (yt, xi) 的匹配程度，为其分配一个分数 αt,i 。 {αt,i} 是一组权重，定义了每个输出应考虑多少源隐藏状态。在 Bahdanau 的论文中，对齐分数 α 由一个单隐藏层的前馈网络参数化，该网络与其他模型部分一起训练。考虑到 tanh 作为非线性激活函数的使用，分数函数因此具有以下形式：
$$
\text{score}(\boldsymbol{s}_t, \boldsymbol{h}_i) = \mathbf{v}_a^\top \tanh(\mathbf{W}_a[\boldsymbol{s}_t; \boldsymbol{h}_i])
$$
其中 va 和 Wa 都是对齐模型中需要学习的权重矩阵。对齐分数矩阵是一个很好的副产品，可以明确展示源词和目标词之间的相关性。

<img src="https://lilianweng.github.io/posts/2018-06-24-attention/bahdanau-fig3.png" alt="img" style="zoom:25%;" />

## A Family of Attention Mechanisms

借助注意力机制，源序列和目标序列之间的依赖关系不再受中间距离的限制！鉴于注意力机制在机器翻译中的显著改进，它很快就被扩展到计算机视觉领域 ([Xu et al. 2015](http://proceedings.mlr.press/v37/xuc15.pdf))，人们开始探索各种其他形式的注意力机制 ([Luong, et al., 2015](https://arxiv.org/pdf/1508.04025.pdf); [Britz et al., 2017](https://arxiv.org/abs/1703.03906); [Vaswani, et al., 2017](http://papers.nips.cc/paper/7181-attention-is-all-you-need.pdf))

以下是几种流行的注意力机制及其对应的对齐分数函数的总结表：

| Name                   | **Alignment score function**                                 | **Citation**                                                 |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Content-base attention | $\text{score}(\boldsymbol{s}_t, \boldsymbol{h}_i) = \text{cosine}[\boldsymbol{s}_t, \boldsymbol{h}_i]$ | [Graves2014](https://arxiv.org/abs/1410.5401)                |
| Additive               | $\text{score}(\boldsymbol{s}_t, \boldsymbol{h}_i) = \mathbf{v}_a^\top \tanh(\mathbf{W}_a[\boldsymbol{s}_{t-1}; \boldsymbol{h}_i])$ | [ Bahdanau2015](https://arxiv.org/pdf/1409.0473.pdf)         |
| Location-Base          | $\alpha_{t,i} = \text{softmax}(\mathbf{W}_a \boldsymbol{s}_t)$ 这简化了 softmax 对齐，使其仅依赖于目标位置。 | [Luong2015](https://arxiv.org/pdf/1508.04025.pdf)            |
| General                | $\text{score}(\boldsymbol{s}_t, \boldsymbol{h}_i) = \boldsymbol{s}_t^\top\mathbf{W}_a\boldsymbol{h}_i$ 其中 Wa 是注意力层中的一个可训练权重矩阵。 | [Luong2015](https://arxiv.org/pdf/1508.04025.pdf)            |
| Dot-Product            | $\text{score}(\boldsymbol{s}_t, \boldsymbol{h}_i) = \boldsymbol{s}_t^\top\boldsymbol{h}_i$ | [ Luong2015](https://arxiv.org/pdf/1508.4025.pdf)            |
| Scaled Dot-Product     | $\text{score}(\boldsymbol{s}_t, \boldsymbol{h}_i) = \frac{\boldsymbol{s}_t^\top\boldsymbol{h}_i}{\sqrt{n}}$ | [Vaswani2017](http://papers.nips.cc/paper/7181-attention-is-all-you-need.pdf) |

以下是注意力机制更广泛的分类总结：

| **Name**       | **Definition**                                               | **Citation**                                                 |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Self-Attention | 关联同一输入序列的不同位置。理论上，自注意力可以采用上述任何分数函数，只需将目标序列替换为相同的输入序列。 | [Cheng2016](https://arxiv.org/pdf/1601.06733.pdf)            |
| Global/Soft    | 关注整个输入状态空间。                                       | [Xu2015](http://proceedings.mlr.press/v37/xuc15.pdf)         |
| Local/Hard     | 关注输入状态空间的一部分；即输入图像的一个区域。             | [Xu2015](http://proceedings.mlr.press/v37/xuc15.pdf); [Luong2015](https://arxiv.org/pdf/1508.04025.pdf) |

### Self-Attention 自注意力

自注意力机制，也称为序列内注意力，是一种关联单个序列不同位置的注意力机制，用于计算同一序列的表示。研究表明，它在机器阅读、抽象摘要或图像描述生成方面非常有用。

长短期记忆网络论文使用自注意力机制进行机器阅读。在下面的例子中，自注意力机制使我们能够学习当前词语与句子前半部分之间的相关性。

<img src="https://lilianweng.github.io/posts/2018-06-24-attention/cheng2016-fig1.png" alt="img" style="zoom:33%;" />

​                                                              当前词语以红色显示，蓝色阴影的大小表示激活程度。

### Soft vs Hard Attention

在论文“[show, attend and tell](http://proceedings.mlr.press/v37/xuc15.pdf) ”论文中，注意力机制被应用于图像以生成描述。首先，图像通过 CNN 编码以提取特征。然后，LSTM 解码器消耗卷积特征逐个生成描述性词语，其中权重通过注意力机制学习。注意力权重的可视化清晰地展示了模型关注图像的哪些区域，以便输出特定词语。

<img src="https://lilianweng.github.io/posts/2018-06-24-attention/xu2015-fig6b.png" alt="img" style="zoom:33%;" />

这篇文章首次提出了“软”与“硬”注意力的区别，基于注意力是否能够访问整个图像或仅访问图像块：

软注意力：对齐权重被学习并“柔和地”放置在源图像的所有块上；本质上与 Bahdanau 等人，2015 年提出的那种注意力类型相同。

- 优点：模型平滑且可微。
- 缺点：当源输入较大时，计算成本高。

硬注意力：每次仅选择图像中的一个块进行关注。

- 优点：推理时计算量较少。
- 缺点：模型不可微分，需要更复杂的技术如方差减少或强化学习来训练。

### Global vs Local Attention

[Luong, et al., 2015](https://arxiv.org/pdf/1508.04025.pdf) 提出了“全局”和“局部”注意力。全局注意力类似于软注意力，而局部注意力是硬注意力和软注意力的有趣结合，是对硬注意力的改进，使其可微分：模型首先为当前目标词预测一个单一的对应位置，然后使用以源位置为中心的窗口来计算上下文向量。

<img src="https://lilianweng.github.io/posts/2018-06-24-attention/luong2015-fig2-3.png" alt="img" style="zoom:50%;" />

## Transformer

[“Attention is All you Need”](http://papers.nips.cc/paper/7181-attention-is-all-you-need.pdf)  是 2017 年最具影响力和最有趣的一篇论文之一。它对软注意力机制提出了许多改进，并使得无需循环网络单元即可进行 seq2seq 建模成为可能。所提出的“transformer”模型完全基于自注意力机制构建，而没有使用序列对齐的循环架构。

### Key, Value and Query

Transformer 的主要组成部分是多头自注意力机制的单位。Transformer 将输入的编码表示视为一组键值对， (K,V) ，维度为 n （输入序列长度）；在 NMT 的上下文中，键和值都是编码器的隐藏状态。在解码器中，前一个输出被压缩成一个查询（维度为 m 的 Q ），下一个输出通过将这个查询与键值集进行映射产生。

Transformer 采用缩放点积注意力：输出是值的加权求和，其中每个值的权重由查询与所有键的点积决定：
$$
\text{Attention}(\mathbf{Q}, \mathbf{K}, \mathbf{V}) = \text{softmax}(\frac{\mathbf{Q}\mathbf{K}^\top}{\sqrt{n}})\mathbf{V}
$$

#### Multi-Head Self-Attention

<img src="https://lilianweng.github.io/posts/2018-06-24-attention/multi-head-attention.png" alt="img" style="zoom: 50%;" />

与其只计算一次注意力，多头机制会并行多次运行缩放点积注意力。独立的注意力输出简单地连接起来，并线性变换为期望的维度。根据论文，"**多头注意力允许模型在不同位置联合关注来自不同表示子空间的信息。使用单个注意力头，平均会抑制这一点**。"
$$
\begin{aligned}
\text{MultiHead}(\mathbf{Q}, \mathbf{K}, \mathbf{V}) &= [\text{head}_1; \dots; \text{head}_h]\mathbf{W}^O \\
\text{where head}_i &= \text{Attention}(\mathbf{Q}\mathbf{W}^Q_i, \mathbf{K}\mathbf{W}^K_i, \mathbf{V}\mathbf{W}^V_i)
\end{aligned}
$$
其中 WiQ 、 WiK 、 WiV 和 WO 是待学习的参数矩阵。

### Encoder

<img src="https://lilianweng.github.io/posts/2018-06-24-attention/transformer-encoder.png" alt="img" style="zoom:50%;" />

编码器生成基于注意力的表示，能够从潜在无限大的上下文中定位特定信息。

- 一堆 N=6 的相同层。
- 每一层都有一个多头自注意力层和一个简单的逐位置全连接前馈网络。
- 每个子层采用残差连接和层归一化。所有子层输出相同维度的数据 dmodel=512 。

### Decoder

<img src="https://lilianweng.github.io/posts/2018-06-24-attention/transformer-decoder.png" alt="img" style="zoom: 50%;" />

解码器能够从编码表示中检索信息

- 堆叠了 N = 6 个相同的层
- 每一层有两个多头注意力机制子层和一个全连接前馈网络子层
- 与编码器类似，每个子层都采用残差连接和层归一化
- 第一个多头注意力子层被修改，以防止位置关注后续位置，因为我们不想在预测当前位置时查看目标序列的未来

### Full Architecture

<img src="https://lilianweng.github.io/posts/2018-06-24-attention/transformer.png" alt="img" style="zoom: 67%;" />

- 源序列和目标序列首先通过嵌入层处理，以生成维度相同的 dmodel=512 数据。
- 为了保留位置信息，应用了基于正弦波的定位编码，并将其与嵌入输出相加
- 在最终解码器输出中添加了 softmax 层和线性层。

## SNAIL

Transformer 没有循环或卷积结构，即使将位置编码添加到嵌入向量中，序列顺序也只被弱地整合。对于像强化学习这样对位置依赖敏感的问题，这可能是一个大问题。

简单神经注意力元学习器（SNAIL）[Mishra et al., 2017](http://metalearning.ml/papers/metalearn17_mishra.pdf) 的开发部分是为了通过结合 Transformer 中的自注意力机制和时序卷积来解决 Transformer 模型中的位置问题。它已被证明在监督学习和强化学习任务中都很有效。

![img](https://lilianweng.github.io/posts/2018-06-24-attention/snail.png)













