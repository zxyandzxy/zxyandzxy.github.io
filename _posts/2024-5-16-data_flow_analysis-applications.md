---
layout: post
title: "数据流分析-应用"
date: 2024-5-16
tags: [static-analysis]
comments: true
author: zxy
---

### Overview of Data Flow Analysis（总览数据流分析）

一句话：不同的数据流分析技术有不同的数据抽象和不同的数据流安全近似策略（即不同的转换函数和不同的控制流处理）

![image-20240119141617095](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119141617095.png)

数据流分析要做的事：

- 定义数据抽象
- 定义转换函数
- 定义数据流控制交汇处汇合方法

### Preliminaries of Data Flow Analysis（数据流分析基础）

#### Input and Output States（输入输出状态）

- 每一个中间语句将输入状态转换成输出状态
- 输入和输出状态关联到一个程序点
- 在每个数据流分析应用程序中，我们将一个数据流值与每个程序点关联起来，该数据流值表示该点可以观察到的所有可能的程序状态集合的抽象。
- 可能的数据流值的集合是此应用程序的域
- 数据流分析是为所有语句在 IN[s]和 OUT[s]上找到一个满足一组 safe-approximation directed constraints 的解决方案。（即满足转换函数和控制流约束）

![image-20240119141810822](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119141810822.png)

#### 控制流约束

![image-20240119142356441](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119142356441.png)

两个方向数据流分析：前向数据流分析+后向数据流分析

^：meet operator，表示在程序交汇处总结不同路径带来的约束

保命声明：下面的数据流分析不包含方法调用和别名

### Reaching Definitions Analysis（到达定值）

#### 基本信息

定义：在程序点 p 处的定值 d 到达程序点 q：如果有一条从 p 到 q 的路径使得 d 没有在这条路径上被“杀死”

![image-20240119142911147](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119142911147.png)

作用：到达定值可以用来检测可能的未定义变量。例如，在控制流图入口（entry 节点）为每个变量 v 引入一个虚拟定义，如果 v 的虚拟定义达到使用 v 的程序点 p，则 v 可以在定义之前使用(即未定义的 v 到达 p)。

类别：前向流分析（要想知道第一个定值在哪才能确定会不会被杀死）+ may analysis（要考虑所有路径的到达定值）

#### 数据流 abstraction

- 在每一个程序点都有这么一串二进制数
- 为程序的所有**定值**设置一个二进制位，若第 i 个定值成立，则第 i 为置 1

![image-20240119143252007](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119143252007.png)

#### 转换函数

对于语句 D: v = x _op_ y 来说，它产生了对 v 的定值 D，杀死了其他对于 v 的定值。

我们以一个基本块 B 为单位进行分析，gen(B)是这个基本块生成的所有定值，kill(B)是这个基本块杀死的所有定值

![image-20240119143441702](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119143441702.png)

栗子：![image-20240119143915606](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119143915606.png)

#### 控制流

为了保证 safe-approximation，只要有一条程序路径可以传递定值，那就认为在程序汇合点处可以定值。

![image-20240119144230307](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119144230307.png)

这里的 U 表示 union，用二进制就是**或**操作（只能从 0->0，0->1，1->1，不能从 1->0）

#### 算法

![image-20240119144417309](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119144417309.png)

算法中：

OUT[entry] = ∅ //在程序开始处，每个变量都没定值

OUT[B] = ∅ //程序还没运行时，每个变量都没定值

算法一定会停：

- 第一：数据流有上限（二进制全 1）
- 第二：gen[B]和 kill[B]都不变
- 第三：开始二进制全为 0
- 第四：根据转换函数，OUT[B]不会有 1->0 的情况

超长的栗子：slide 61 页

### Live Variables Analysis（活跃变量）

#### 基本信息

定义：在一个程序点 p，如果变量 v 活跃，那么 v 就可以在沿着 p 到 exit 的一条程序路径上使用

![image-20240119145810923](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119145810923.png)

作用：活跃变量的信息可用于寄存器分配。例如，在某些情况下，所有寄存器都满了，我们需要使用一个寄存器，那么我们应该倾向于使用存储不活跃变量的寄存器

类别：后向数据流分析（需要知道在后面某个变量有没有被重定义）+ may analysis（后向有任何一条路径使用了变量，那就说这个变量是活跃的）

#### 数据流 abstraction

- 在所有程序点都有一个二进制串
- 为程序所有的变量设置一个二进制位，如果在某个程序点这个变量活跃，那么这个位置为 1

![image-20240119150335417](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119150335417.png)

#### 转换函数与控制流

![image-20240119150736274](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119150736274.png)

对于语句 x = y op z 来说，y，z 被 use 了，x 被 redefine 了

use[B]：在基本块 B 中被使用的变量

def[B]：在基本块 B 中被重定义的变量

U：表示**或**操作

注意：要从基本块 B 的最后一条语句开始，一步一步往上分析，最终得出 IN[B]，所以其实基本单位是每一条语句。

#### 算法

![image-20240119151136256](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119151136256.png)

IN[exit] = ∅：出口处不会再有变量被使用，全部不活跃

IN[B] = ∅：开始未扫描任何语句，全部变量不活跃

算法一定会停：

- 第一：数据流有上限（二进制全 1）
- 第二：use[B]和 def[B]都不变
- 第三：开始二进制全为 0
- 第四：根据转换函数，IN[B]不会有 1->0 的情况

超长栗子：slide 169 页

### Available Expressions Analysis（可用表达式）

#### 基本信息

定义：在程序点 p 的一个表达式 x op y 可用必须满足两个条件：第一从 entry 到 p 点，必须计算过 x op y。第二最后一次计算 x op y 后，x 和 y 都没被修改过。

作用：检测公共子表达式，用于编译代码优化

类别：前向数据流（先得知道 x op y 在哪定值）+ must analysis（多条程序路径必须都对 x op y 进行计算，在汇合点才能使用 x op y）

#### 数据流 abstraction

在每一个程序点都有一个二进制串

对于所有表达式，如果在这个程序点可用，就置为 1

![image-20240119152658605](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119152658605.png)

#### 转换函数和控制流

gen[B]：基本块 B 产生的可用表达式

kill[B]：基本块 B 杀死的可用表达式

![image-20240119152935917](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119152935917.png)

倒 U 表示：**与**操作，即所有路径都有才认为可用

#### 算法

![image-20240119153050290](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119153050290.png)

OUT[entry] = ∅：程序入口必然没有可用表达式

OUT[B] = U：刚开始认为所有表达式可用，根据转换函数和控制流一步步删减

超长栗子：slide 252 页

### 总结三种数据流分析

![image-20240119153731540](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119153731540.png)

### 重点问题：

#### 1.理解三种数据流分析

见上文（基本信息 + 数据流抽象 + 转换函数 + 控制流 + 迭代算法）

#### 2.阐述三种数据流分析的相同点和不同点

见总结**三种数据流分析**

#### 3.理解迭代算法，并说明为什么迭代算法可以停止

**见上文，算法部分**
