---
layout: post
title: "现代循环神经网络"
date: 2024-9-20
tags: [deep-learning]
comments: true
author: zxy
---

## 门控循环单元(GRU)

[9.1. 门控循环单元（GRU） — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_recurrent-modern/gru.html)

- 门控循环神经网络可以更好地捕获时间步距离很长的序列上的依赖关系。
- 重置门有助于捕获序列中的短期依赖关系。
- 更新门有助于捕获序列中的长期依赖关系。
- 重置门打开时，门控循环单元包含基本循环神经网络；更新门打开时，门控循环单元可以跳过子序列。

![../_images/gru-3.svg](https://zh-v2.d2l.ai/_images/gru-3.svg)

## 长短期记忆网络(LSTM)

[9.2. 长短期记忆网络（LSTM） — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_recurrent-modern/lstm.html)

- 长短期记忆网络有三种类型的门：输入门、遗忘门和输出门。
- 长短期记忆网络的隐藏层输出包括“隐状态”和“记忆元”。只有隐状态会传递到输出层，而记忆元完全属于内部信息。
- 长短期记忆网络可以缓解梯度消失和梯度爆炸。

![../_images/lstm-3.svg](https://zh-v2.d2l.ai/_images/lstm-3.svg)

## 深度循环神经网络

[9.3. 深度循环神经网络 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_recurrent-modern/deep-rnn.html)

- 在深度循环神经网络中，隐状态的信息被传递到当前层的下一时间步和下一层的当前时间步。
- 有许多不同风格的深度循环神经网络， 如长短期记忆网络、门控循环单元、或经典循环神经网络。 这些模型在深度学习框架的高级API中都有涵盖。
- 总体而言，深度循环神经网络需要大量的调参（如学习率和修剪） 来确保合适的收敛，模型的初始化也需要谨慎。

![../_images/deep-rnn.svg](https://zh-v2.d2l.ai/_images/deep-rnn.svg)

## 双向循环神经网络

[9.4. 双向循环神经网络 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_recurrent-modern/bi-rnn.html)

- 在双向循环神经网络中，每个时间步的隐状态由当前时间步的前后数据同时决定。
- 双向循环神经网络与概率图模型中的“前向-后向”算法具有相似性。
- 双向循环神经网络主要用于序列编码和给定双向上下文的观测估计。
- 由于梯度链更长，因此双向循环神经网络的训练代价非常高。

![../_images/birnn.svg](https://zh-v2.d2l.ai/_images/birnn.svg)

## 机器翻译与数据集

[9.5. 机器翻译与数据集 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_recurrent-modern/machine-translation-and-dataset.html)

- 机器翻译指的是将文本序列从一种语言自动翻译成另一种语言。
- 使用单词级词元化时的词表大小，将明显大于使用字符级词元化时的词表大小。为了缓解这一问题，我们可以将低频词元视为相同的未知词元。
- 通过截断和填充文本序列，可以保证所有的文本序列都具有相同的长度，以便以小批量的方式加载。

## 编码器-解码器架构

[9.6. 编码器-解码器架构 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_recurrent-modern/encoder-decoder.html)

- “编码器－解码器”架构可以将长度可变的序列作为输入和输出，因此适用于机器翻译等序列转换问题。
- 编码器将长度可变的序列作为输入，并将其转换为具有固定形状的编码状态。
- 解码器将具有固定形状的编码状态映射为长度可变的序列。

![../_images/encoder-decoder.svg](https://zh-v2.d2l.ai/_images/encoder-decoder.svg)

## 序列到序列学习(seq2seq)

[9.7. 序列到序列学习（seq2seq） — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_recurrent-modern/seq2seq.html)

- 根据“编码器-解码器”架构的设计， 我们可以使用两个循环神经网络来设计一个序列到序列学习的模型。
- 在实现编码器和解码器时，我们可以使用多层循环神经网络。
- 我们可以使用遮蔽来过滤不相关的计算，例如在计算损失时。
- 在“编码器－解码器”训练中，强制教学方法将原始输出序列（而非预测结果）输入解码器。
- BLEU是一种常用的评估方法，它通过测量预测序列和标签序列之间的n元语法的匹配度来评估预测。

![../_images/seq2seq.svg](https://zh-v2.d2l.ai/_images/seq2seq.svg)

## 束搜索

[9.8. 束搜索 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_recurrent-modern/beam-search.html)

- 序列搜索策略包括贪心搜索、穷举搜索和束搜索。
- 贪心搜索所选取序列的计算量最小，但精度相对较低。
- 穷举搜索所选取序列的精度最高，但计算量最大。
- 束搜索通过灵活选择束宽，在正确率和计算代价之间进行权衡。

![../_images/beam-search.svg](https://zh-v2.d2l.ai/_images/beam-search.svg)