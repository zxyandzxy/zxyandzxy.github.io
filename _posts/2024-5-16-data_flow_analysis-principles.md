---
layout: post
title: "数据流分析-原理"
date: 2024-5-16
tags: [static-analysis]
comments: true
author: zxy
---

### Iterative Algorithm, Another View（换个视角看迭代）

前向数据流分析过程

- 给定一个有 k 个节点的 CFG（程序），迭代算法会在每次迭代中为每个节点 n 更新 OUT[n]
- 假设数据流分析中的值域为 V，那么我们可以定义一个 k 元组（因为有 k 个程序点），作为集合（V1 × V2 ... × Vk）的元素，表示为 V^k，用于保存每次迭代后的分析值。
- 每次迭代都可以看作是通过传递函数和控制流处理，将 V^k 中的一个元素映射到 V^k 中的一个新元素，抽象为函数 F: V^k→V^k
- 然后，该算法迭代输出一系列 k 元组，直到连续两次迭代的 k 元组与最后一个相同

![image-20240119161550420](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119161550420.png)

即最终求出了一个不动点 X = F(X)

### Partial Order（偏序）

定义：P 是一个集合。偏序满足自反性，对称性，传递性

![image-20240119161810629](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119161810629.png)

理解偏序：偏序意味着集合 P 中，不需要保证每两个元素都要满足偏序定义的二元关系

### Upper and Lower Bounds（上界与下界）

![image-20240119162157030](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119162157030.png)

![image-20240119162203058](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119162203058.png)

上界和下界都是 P 中的元素。

#### 最小上界（lub / join）和最大下界（glb / meet）

![image-20240119162442542](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119162442542.png)

![image-20240119162456455](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119162456455.png)

即上界里面找最小和下界里面找最大

##### 特别的

![image-20240119162556364](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119162556364.png)

#### 性质

- 不是所有偏序集都有最小上界和最大下界
- 一旦偏序集有最小上界和最大下界，那 lub，glb 必唯一

### Lattice, Semilattice, Complete and Product Lattice（格，半格，完全格，积格）

格：如果一个偏序集的每一对元素都有一个最小上界和一个最大下界，那么它就是一个格。这个最小上界和最大下界在整个偏序集里面找。

![image-20240119163124784](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119163124784.png)

半格：最小上界和最大下界只有一个存在

![image-20240119163132486](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119163132486.png)

全格：给定一个格，对于这个格的任意一个子集，如果这个子集的最小上界和最大下界都存在，那这个格就是全格（还可以这么判断：每一个有限格就是完全格）

![image-20240119163236765](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119163236765.png)

![image-20240119163405540](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119163405540.png)

积格：

![image-20240119165312198](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119165312198.png)

性质：

- 积格也是一个格
- 如果一个积格 L 是一个完全格的积，那么 L 也是全格

### Data Flow Analysis Framework via Lattice（基于格的数据流分析框架）

![image-20240119165530333](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119165530333.png)

数据流分析可以看作是对格的值迭代地应用传递函数和 meet / join 操作

### Monotonicity and Fixed Point Theorem（单调性与不动点定理）

单独性定义：

![image-20240119165928156](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119165928156.png)

不动点定理：

![image-20240119165942448](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119165942448.png)

证明 slide 123 页

说明在基于格的理论上，算法是能求出最小 / 大不动点的

### Relate Iterative Algorithm to Fixed Point Theorem（联系迭代算法与不动点定理）

要关联迭代算法到不动点理论，两步：

**第一步**：证明迭代算法表示的格是有限的

![image-20240119171005632](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119171005632.png)

有 k 个程序点，每个程序点都可以表示为一个格。那么整个程序就可以表示为一个积格。由于每个格都是有限（0,1 二进制串的个数有限）和全格，那么积格也是有限和全格。

**第二步**：证明迭代算法的转换函数，汇合函数是单调的

![image-20240119171456894](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119171456894.png)

现在可以讲出三个关于迭代算法的三个结论：（用格的理论已经证明）

- 算法必然到达一个不动点
- 数据流分析有多个不动点，迭代算法总能找到一个最好的不动点
- 最多的迭代轮数 = 格高度 \* CFG 的节点数（每轮迭代只更新一个节点的某个位置）

格高：从格的 top 到 bottom 的最长路径长

### May/Must Analysis, A Lattice View（以格视角观察 may/must analysis）

![image-20240119172244470](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119172244470.png)

我们在数据流分析中设计的转换函数和控制流汇合模式决定了我们每一次向 safe 的步长都是最小的，也就决定了一定能找到最优不动点

### MOP and Distributivity（MOP 与可分配性）

![image-20240119172642787](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119172642787.png)

- MOP 是对每条路径先应用 Fp，然后在进行 join / meet 操作
- 由于这些路径不一定每一条都执行，所以 MOP 的结果是 sound。

#### 对比迭代算法和 MOP

![image-20240119172859302](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119172859302.png)

![image-20240119173023823](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119173023823.png)

到现在我们研究的到达定值，活跃变量，可用表达式的转换函数和控制流汇合模式都是可分配的

### Constant Propagation（常量传播，不是可分配的）

定义：给定程序点 p 处的变量 x，确定 x 是否保证在 p 处保持恒定值。

以格视角解读常量传播：

CFG 中每个节点的 OUT 包含一组 (x, v)，其中 x 是一个变量，v 是 x 在该节点之后持有的值

前向数据流问题：先定义常量，常量再传播

域及汇合操作符

![image-20240119173437015](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119173437015.png)

转换函数

![image-20240119173526970](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119173526970.png)

### Worklist Algorithm（工作流算法）

![image-20240119173800132](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119173800132.png)

- 如果前序 OUT[]不变，那么它的后继 IN[]不变，IN[]不变，OUT[]必然不变
- 只要考虑 OUT[]值变了的节点的后继，这样就减少了每轮迭代要访问的节点，减小复杂度

### 重点问题：

#### 1.从函数视角理解迭代算法

见上文：换个视角看迭代

#### 2.格和全格的定义

见上文：格，半格，完全格，积格

#### 3.理解不动点定理

见上文：单调性与不动点定理

#### 4.怎么从格的视角总结 may/must analysis

见上文：以格的视角观察 may/must analysis

#### 5.MOP 和迭代算法之间的关联与区别

见上文：对比迭代算法和 MOP

#### 6.理解常量传播

见上文：常量传播

#### 7.工作流算法

见上文：工作流算法
