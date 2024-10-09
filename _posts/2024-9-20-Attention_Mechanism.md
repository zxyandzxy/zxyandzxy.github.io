---
layout: post
title: "注意力机制"
date: 2024-9-20
tags: [deep-learning]
comments: true
author: zxy
---

## 注意力提示

[10.1. 注意力提示 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_attention-mechanisms/attention-cues.html)

- 人类的注意力是有限的、有价值和稀缺的资源。
- 受试者使用非自主性和自主性提示有选择性地引导注意力。前者基于突出性，后者则依赖于意识。
- 注意力机制与全连接层或者汇聚层的区别源于增加的自主提示。
- 由于包含了自主性提示，注意力机制与全连接的层或汇聚层不同。
- 注意力机制通过注意力汇聚使选择偏向于值（感官输入），其中包含查询（自主性提示）和键（非自主性提示）。键和值是成对的。
- 可视化查询和键之间的注意力权重是可行的。

![../_images/qkv.svg](https://zh-v2.d2l.ai/_images/qkv.svg)

## 注意力汇聚：Nadaraya-Watson核回归

[10.2. 注意力汇聚：Nadaraya-Watson 核回归 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_attention-mechanisms/nadaraya-waston.html)

- Nadaraya-Watson核回归是具有注意力机制的机器学习范例。
- Nadaraya-Watson核回归的注意力汇聚是对训练数据中输出的加权平均。从注意力的角度来看，分配给每个值的注意力权重取决于将值所对应的键和查询作为输入的函数。
- 注意力汇聚可以分为非参数型和带参数型。

$$
f(x) = \sum_{i=1}^n \alpha(x, x_i) y_i,
$$



## 注意力评分函数

[10.3. 注意力评分函数 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_attention-mechanisms/attention-scoring-functions.html)

- 将注意力汇聚的输出计算可以作为值的加权平均，选择不同的注意力评分函数会带来不同的注意力汇聚操作。
- 当查询和键是不同长度的矢量时，可以使用可加性注意力评分函数。当它们的长度相同时，使用缩放的“点－积”注意力评分函数的计算效率更高。

![../_images/attention-output.svg](https://zh-v2.d2l.ai/_images/attention-output.svg)
$$
f(\mathbf{q}, (\mathbf{k}_1, \mathbf{v}_1), \ldots, (\mathbf{k}_m, \mathbf{v}_m)) = \sum_{i=1}^m \alpha(\mathbf{q}, \mathbf{k}_i) \mathbf{v}_i \in \mathbb{R}^v,
$$

## Bahdanau注意力

[10.4. Bahdanau 注意力 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_attention-mechanisms/bahdanau-attention.html)

- 在预测词元时，如果不是所有输入词元都是相关的，那么具有Bahdanau注意力的循环神经网络编码器-解码器会有选择地统计输入序列的不同部分。这是通过将上下文变量视为加性注意力池化的输出来实现的。
- 在循环神经网络编码器-解码器中，Bahdanau注意力将上一时间步的解码器隐状态视为查询，在所有时间步的编码器隐状态同时视为键和值。

![../_images/seq2seq-attention-details.svg](https://zh-v2.d2l.ai/_images/seq2seq-attention-details.svg)

Bahdanau本质是一种 **加性attention机制**，将decoder的隐状态和encoder所有位置输出通过线性组合对齐，得到context向量，用于改善序列到序列的翻译模型。

**本质：两层全连接网络，隐藏层激活函数tanh，输出层维度为1。**

## 多头注意力

[10.5. 多头注意力 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_attention-mechanisms/multihead-attention.html)

- 多头注意力融合了来自于多个注意力汇聚的不同知识，这些知识的不同来源于相同的查询、键和值的不同的子空间表示。
- 基于适当的张量操作，可以实现多头注意力的并行计算。

![../_images/multi-head-attention.svg](https://zh-v2.d2l.ai/_images/multi-head-attention.svg)

## 自注意力和位置编码

[10.6. 自注意力和位置编码 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_attention-mechanisms/self-attention-and-positional-encoding.html)

- 在自注意力中，查询、键和值都来自同一组输入。
- 卷积神经网络和自注意力都拥有并行计算的优势，而且自注意力的最大路径长度最短。但是因为其计算复杂度是关于序列长度的二次方，所以在很长的序列中计算会非常慢。
- 为了使用序列的顺序信息，可以通过在输入表示中添加位置编码，来注入绝对的或相对的位置信息。

$$
\mathbf{y}_i = f(\mathbf{x}_i, (\mathbf{x}_1, \mathbf{x}_1), \ldots, (\mathbf{x}_n, \mathbf{x}_n)) \in \mathbb{R}^d
$$

![../_images/cnn-rnn-self-attention.svg](https://zh-v2.d2l.ai/_images/cnn-rnn-self-attention.svg)

## Transformer

[10.7. Transformer — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_attention-mechanisms/transformer.html)

- Transformer是编码器－解码器架构的一个实践，尽管在实际情况中编码器或解码器可以单独使用。
- 在Transformer中，多头自注意力用于表示输入序列和输出序列，不过解码器必须通过掩蔽机制来保留自回归属性。
- Transformer中的残差连接和层规范化是训练非常深度模型的重要工具。
- Transformer模型中基于位置的前馈网络使用同一个多层感知机，作用是对所有序列位置的表示进行转换。

![../_images/transformer.svg](https://zh-v2.d2l.ai/_images/transformer.svg)