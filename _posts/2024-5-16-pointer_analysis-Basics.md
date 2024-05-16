---
layout: post
title: "指针分析-基础"
date: 2024-5-16
tags: [static-analysis]
comments: true
author: zxy
---

### Pointer Analysis: Rules

指针分析涉及的域和符号

![image-20240119202646091](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119202646091.png)

pt：可以看成是一个 map，描述一个键值对（指针：指针所指向的对象集合）

指针指向规则

![image-20240119202817174](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119202817174.png)

New 语句：oi 无条件添加到 x 所指向的对象集合中

Assign：将 y 指向的对象全部添加到 x 所指向的对象集合中

Store：将 y 所指向的对象全部添加到 x 所指向的对象 oi 的域 f 所指向的对象集合中

Load：将 x 所指向的对象 oi 的域 f 所指向的对象全部添加到 y 指向的对象集合中

![image-20240119203227269](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119203227269.png)

premise：流向

### How to Implement Pointer Analysis

#### 总览

- 本质上，指针分析是在指针(变量和字段)之间传播指向信息。
- 实现关键：当 pt(x)发生变化时，将变化的部分传播到跟 x 相关的指针上
- 我们用图来连接相关指针，当 pt(x)变化，传播改变的部分到 x 的后继节点

#### pointer flow graph

定义：程序的指针流图是表示对象如何在程序中的指针之间流动的有向图

节点： Pointer = V ⋃ (O × F)

边： Pointer × Pointer （边的 x**→**y 意味着被指针 x 指向的对象可能流向指针 y）

![image-20240119203837777](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119203837777.png)

栗子：

![image-20240119204016478](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119204016478.png)

实现指针分析：

两步：建立 PFG + 在 PFG 上传播指针指向信息

这两步互相依赖，没有 PFG 就没法传播指向信息，指向信息又是构建 PFG 的要素

### Pointer Analysis: Algorithms（流不敏感）

![image-20240119204243128](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119204243128.png)

WL：存储要处理的指针指向信息 WL = [一系列<pointer，pts>]

![image-20240119204528821](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119204528821.png)

表示 pts 这个集合里的对象要加入 pt(pointer)中，即 pts 里面的对象有可能被 pointer 指向

刚开始，所有对象和变量的 pts()都是空

△ = pts - pt(n) 这一行是去重代码，△ 中是这一轮真正要加入 pt(n)的对象集合

- 采用差分传播，避免了冗余信息的传播和处理
- 为什么可以减少冗余信息传播：因为之前有一部分信息已经传播过了
- 在实践中，Δ 通常比原始集合小，因此只传播新的信息(Δ)可以提高效率
- 此外，Δ 对于处理存储、加载和方法调用时的效率也很重要

算法讲解 slide 47 页

栗子 slide 70 页

### Pointer Analysis with Method Calls

#### 规则

![image-20240119210343535](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119210343535.png)

四步走：

- dispatch：根据 x 指向的对象去寻找调用的方法（这就是指针分析比 CHA 更准确的原因）
- 传递 this 指针：this 指针是调用的方法的指针
- 传递参数：实参指向的对象都加入到对应形参指向的对象集合中
- 传递返回值：方法 return 的对象加入到方法赋予的对象指针指向集合中
- PFG 中不需要边 x -> m.this，因为 x 可能指向多个方法，有这条边就没办法决定真正的方法

#### 实现算法（流不敏感）

指针分析 + 调用图构造 要一起实现，这两步会互相影响

实现时：从 main 方法出发，每次调用其他方法就加入“可达世界”中，直到所有可达方法分析完成

![image-20240119211141212](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119211141212.png)

算法理解： slide 116 页

栗子： slide 126 页

### 重点问题

#### 理解指针分析的规则

new 语句， 赋值语句，store 语句，load 语句是 四种会改变指针指向对象的语句

理解上文的那张图即可。

#### 理解指针流图

指针流图是记录对象如何在程序指针之间流动的有向图

指针分析就是依靠指针流图进行指针传递的分析方法

#### 理解指针分析算法

理解上图算法即可

#### 理解带有方法调用的指针分析的规则

dispatch 找调用方法 + 传递参数 + 传递返回值

#### 理解过程间指针分析算法

以 BFS 思路进行，逐步寻找可达方法，构造 pfs 图

#### 理解如何以指针分析思路构造调用图

其实就是在过程间指针分析过程中实现的
