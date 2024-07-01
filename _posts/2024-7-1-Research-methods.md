---
layout: post
title: "科研方法"
date: 2024-7-1
tags: [Research Chat]
comments: true
author: zxy
---

### 一个科研周期

- 文献查找与阅读
- 问题发现与创新点挖掘
- 实验验证
- 论文写作
- Rebuttal

### 查找文献

熟悉 **CCF 列表**——中国计算机学会

1. 重点：找自己领域的**顶会网址**

2. 谷歌学术检索关键字，综合排名，会议

### 文献阅读

- 精读 abstract、introduction、related work。了解领域发展史以及问题（看看有没有漏掉的），地柜查询法
- 不要用翻译软件大段翻译，不利于自己的阅读水平的提高
- 泛读方法、实验，但是要懂原理
- 下载有代码的文章来看：顶会的 paper github 一般都有代码

> 其他文献阅读

- 其他领域的泛读，发展方向的创新
- 课外知识量（李永乐）
- 机器学习、深度学习视频课程（李宏毅，July的算法教程）

### 问题发现与创新点挖掘

1. 问题 Problem

  怎么从别人的论文中发现问题？

- 局部 VS 全局（局部全局融合） 栗子：他只做了局部考虑，那我就可以全局考虑
- 静态 VS 动态 （卷积核都是静态设计好的，动态的卷积核？）
- 单一 VS 全面（考虑多个任务，方法的转移）
-  针对缺陷：效果好 -> 效率差/高 -> 鲁棒性高

2. **动机 Motivation**

3. 贡献 Contribution

**一般来说，上面的问题，动机，贡献在实际的科研过程中是反过来的，先有创新点，在谈动机，在讲问题**

当然按照上面的顺序来也是没什么问题，看真实情况下的具体情况就会

考虑的不全面 -> 全面

- 比如前面的网络利用的不够好？
- 比如残差网络的变化太剧烈  ->考虑一个缓慢增加的结构：

- 创新（贡献）：提出了一个金字塔的结构
- 动机：特征图数缓慢变化似乎更有利于网络的特征学习
- 问题：特征突变可能不利于网络  -> 特征图的网络是影响性能的

### 实验验证

**自己的方法实验过程**：

- 获取正确的评价指标的代码（就是寻找能验证我的方法对问题有效的那些指标怎么算）
- 小规模小数据集验证**有效与否**（防止一个无效实验跑3,4天的情况出现）
- 大规模大数据集验证**先进性**

**与先进方法对比实验：**

1. 效果高于对比的论文 -》 直接引用其结果。方法比较高效，但结果比较难调，适合会议论文（快）

2. 效果不及对比论文 -》 复现、下载别人的代码。因为别人的论文有 trick，自己的论文比较细心一点。时间长，易高于别人，适合期刊论文（慢）

### 论文写作

**一篇论文来说，排版 + 格式 + 结构 + 图表是至关重要的**

> **论文写作顺序：**

1. Related works
2. Method
3. Experiment and analysis
4. Discussion
5. Conclusion
6. Introduction

- 先写Related works有两点好处：增强对研究问题的理解（因为看了很多别人的论文） + 避免成为井底之蛙（自己的方法已经有人写过了）
- Introduction是整篇论文的统领与概括，和abstract一样要后写或者先打大致草稿
- 会议的Related works部分相对简单，三言两语介绍最近的工作即可
- 期刊的Related works往往就要写成该领域的一个小综述
- Related works的行文方式分为 `时间脉络 +分门别类`，一定不能平铺直叙
- Related works的核心思想是 `夸赞 + 批评`，只有说出他们有什么不足，我们的工作才有意义 ，这里的不足要说跟我们idea高度相关的

> **Abstract**

最先写，骨架

内容是：解决xx问题，提出xx方法，怎么做的，实验效果？

具体可以分为几个步骤：

1. 交代研究领域的大背景
2. 细化研究背景
3. 这个研究背景下的相关研究存在 X 问题
4. 为了解决这个问题，我们提出了 Y 方法
5. 实验表明，我们的方法很nice
6. 我们的工作为小领域，以至整个领域有什么贡献

> **Introduction**

1. 介绍问题
2. 经过哪些发展
3. 现在有什么问题
4. 本文提出了什么去解决
5. 列举贡献点

> **Related work**

  会议论文短的话可以不要写

内容写：

- 文章所用到的关键技术点的发展
- 存在的问题

> **Our method**

  “总”、“分”式写法

- 介绍总体架构
- 介绍子结构

> **Experiment**

必须要写的实验：

1. 介绍数据集和评价方法
2. 实验具体是怎么做的
3. 分模块有效性对比实验（消融实验）
4. 与先进方法的对比实验（现今）

可选实验：

1. 结果可视化
2. 参数量或者效率对比
3. 鲁棒性对比
4. 讨论优缺点

多元化效果展示：

- 表格对比
- 训练曲线对比
- 可视化结果对比
- 矩阵图，柱状图对比

> **Conclusion ——换一种表述的 abstract**

> 一些英文写作的语言细节

| Instead of            | say         | or say      |
| --------------------- | ----------- | ----------- |
| Research work         | Research    | Work        |
| Limit condition       | Limit       | Condition   |
| Knowledge memory      | Knowledge   | Memory      |
| Sketch map            | Sketch      | Map         |
| Layout scheme         | Layout      | scheme      |
| Arrangement plan      | Arrangement | plan        |
| Output performance    | Output      | performance |
| Simulation results    | results     | simulation  |
| Knowledge information | Knowledge   | information |
| Calculation results   | results     | calculation |
| Application           | Results     | Application |

**Academic-phrasebank**

- Many researchers have utilized X to measures ...
- One of the most well-known tools for assessing ...
- Traditionally, X has been assessed by measureing ...
- A number of techniques have been developed to ...
- Different methods have been proposed to classify ...
- X is the main non-invasive method used to determine ...
- Different authors have measured X in a variety of ways.
- Serveral methods currently exist for the measurement of X.
- Previous studies have based their criteria for selection on ...
- X is one of the most common procedures for determining ...
- There are three main types of study design used to identify ...  

> 其他信息

- Latex 【laetex】
- 直接英文写作（不翻译）（现在也可以ChatGPT）
- 句式尽量简单、直接。最多嵌套两层的定语从句
- 绘制整体的结构图
- 最好使用 8 个公式 —— （怎么体现出公式感？即使没有公式也要加公式）

要和问题结合表述当前的应用方向、应用任务（就论文中每句话的表述要始终关心这一点）

### Rebuttal

思维模式：回答审稿人的问题，感谢其有利的建议，委婉反驳其对自己不利的评论

一般审稿人的回复：

- 质疑贡献点：强调贡献点
- 质疑实验补全：补充实验 / 强调为什么没必要这个实验
- 文章书写差：承诺润色文章

负面例子：

考虑不充分？没有用 validation 集？ 

- 不能回答没考虑，没有时间
- 要回答不能实现

### 总结 

- 有论文为驱动力、压力才有动力
- 精益求精、挖掘细节
- 多思考、多分析
- 较强的行动力
- 长时间的精神高度集中



> 原视频链接：

【研究生如何科研-B站最实用的科研方法(计算机专业)】 https://www.bilibili.com/video/BV1aa41167jN/?share_source=copy_web&vd_source=fa43e9d6cd4073e1172b87c3220f143c

