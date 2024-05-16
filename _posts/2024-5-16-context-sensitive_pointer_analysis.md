---
layout: post
title: "上下文敏感指针分析"
date: 2024-5-16
tags: [static-analysis]
comments: true
author: zxy
---

### Introduction

#### 上下文不敏感指针分析（CI）

- 在动态执行中，一个方法可能在不同的调用上下文中被多次调用
- 在不同的调用上下文下，方法的变量可能指向不同的对象
- 在 CI 指针分析中，不同上下文下的对象被混合并传播到程序的其他部分(通过返回值或副作用)，从而导致虚假的数据流，进而导致分析不准确
- 注意上下文不敏感和流不敏感的区别（CI 更关注对象，是指同一个方法被多个对象调用，但是方法不做区分导致多个对象混在混在一个方法中。流不敏感更关注执行顺序，强调从上到下执行）

#### 上下文敏感指针分析（CS）

- 上下文敏感性模型通过区分不同上下文的不同数据流来调用上下文，以提高精度
- call site sensitivity 策略：每个方法有一个上下文，这个上下文用链表示链上连接的一系列 call site，用来抽象模拟动态运行时的栈
- 一个 call site 可以用一个语句来理解

#### 基于克隆的上下文敏感性

- 在基于克隆的上下文敏感指针分析中，每个方法都由一个或多个上下文限定
- 变量也由上下文限定(从声明它们的方法继承上下文)
- 实际上，每个方法及其变量都是克隆的，每个上下文一个克隆

![image-20240120113040854](https://zxyandzxy.github.io/images/image-20240120113040854.png)

#### 上下文敏感+堆抽象

- 在实践中，为了提高精度，还应该将上下文敏感性应用于堆抽象
- 抽象对象也由上下文限定(称为堆上下文)。
- 上下文敏感的堆抽象在 call site 抽象之上提供了更细粒度的堆模型

![image-20240120113309136](https://zxyandzxy.github.io/images/image-20240120113309136.png)

##### 上下文敏感堆可以提高精度

- 在动态执行中，call site 可以在不同的调用上下文下创建多个对象（用下面的代码看，程序不同位置调用 newX 方法都会创建 x 对象，他们是不同上下文的）
- 不同的对象(由同一 call site 分配，方法里面的 call site 一样)可以使用不同的数据流进行操作，例如，在其字段中存储不同的值
- 在指针分析中，分析没有堆上下文的代码可能会因为将不同上下文的数据流合并到一个抽象对象而失去精度。
- 相反，通过堆上下文区分来自同一分配位置的不同对象可以获得精度（即在对象前加上上下文信息）
- 总结：在精度上 上下文敏感堆 > 上下文敏感无堆 > 上下文不敏感
- 栗子见 slide 34 页

![image-20240120113631406](https://zxyandzxy.github.io/images/image-20240120113631406.png)

### Context Sensitive Pointer Analysis: Rules

#### 域及符号

![image-20240120114253657](https://zxyandzxy.github.io/images/image-20240120114253657.png)

#### 规则

![image-20240120114447800](https://zxyandzxy.github.io/images/image-20240120114447800.png)

![image-20240120114525180](https://zxyandzxy.github.io/images/image-20240120114525180.png)

具有上下文的方法调用：

- dispatch：带着上下文指向的对象寻找目标方法
- select：根据已知信息为目标方法选择上下文
- m.this：传递目标方法的 this 指针
- 传递形参：带着上下文信息传递形参
- 传递返回值：带着上下文信息传递返回值

### Context Sensitive Pointer Analysis: Algorithms

![image-20240120115018260](https://zxyandzxy.github.io/images/image-20240120115018260.png)

- 带着上下文信息建立 PFG 图和传递指针信息
- Nodes: CSPointer = (C × V) ⋃ (C × O × F) 节点 n 表示上下文敏感的变量或上下文敏感的抽象对象
- Edges: CSPointer × CSPointer 一条边 x->y 代表 x 指向的对象可能会流向 y 指向的对象

![image-20240120115224831](https://zxyandzxy.github.io/images/image-20240120115224831.png)

![image-20240120115259118](https://zxyandzxy.github.io/images/image-20240120115259118.png)

#### 算法

![image-20240120115825983](https://zxyandzxy.github.io/images/image-20240120115825983.png)

算法解释 slide 88 页 开始的 main 方法默认没有上下文

### Context Sensitivity Variants（如何设计 select 方法）

![image-20240120120309583](https://zxyandzxy.github.io/images/image-20240120120309583.png)

#### context insensitivity

- CI 可以看做 CS 的一种特例，不管输入什么，上下文都设为空
- Select(_, _, \_)= [ ]

#### call-site sensitivity

- 上下文用一系列 call-site 描述
- 在方法调用时，将 call-site 作为被调用方法的上下文附加到调用方法的上下文
- 本质上是对动态运行时调用栈的抽象
- ![image-20240120120350031](https://zxyandzxy.github.io/images/image-20240120120350031.png)
- 一般来说，这里的 call-site 数量最多是 k 个，即保留后 k 个 call-site 作为上下文
- ![image-20240120120626651](https://zxyandzxy.github.io/images/image-20240120120626651.png)
- 栗子： slide 116 页

#### object sensitivity

- 每个上下文由一个抽象对象列表组成(由它们的分配位置表示)
- 在方法调用时，使用接收对象及其堆上下文作为被调用方法的上下文
- 栗子： slide 145 页

#### 对比 call-site sensitivity 与 object sensitivity

理论上，它们的精确度是不可以比较的（有些情况 call-site 更好，有些 object 更好）

在实践中，object sensitivity 通常优于 call-site sensitivity，对于 OO 语言(如 Java）

#### type sensitivity

- 上下文由一系列类型组成
- 在方法调用时，使用包含接收器对象的分配位置及其堆上下文的类型作为被调用上下文
- 是对对象敏感性的粗略抽象
- ![image-20240120121727652](https://zxyandzxy.github.io/images/image-20240120121727652.png)
- 相同 k 的限制下，type sensitivity 的精度低于 object sensitivity，但是速度更快
- ![image-20240120121918568](https://zxyandzxy.github.io/images/image-20240120121918568.png)

### 重点问题：

#### 理解上下文敏感的概念

见上文 introduction-CS

#### 理解带堆上下文敏感的概念

见上文 introduction-上下文敏感+堆抽象（**关注被调用方法内部-----声明和赋值的对象的区分**）

#### 为什么上下文敏感和带堆上下文敏感可以提高精度

见上文

#### 理解上下文敏感指针分析规则

见上文

#### 理解上下文敏感指针分析算法

就是带着上下文的指针分析算法

#### 常见的上下文敏感变种

以 object 区分，以 type 区分，以 call site 区分

### 不同上下文敏感变种的关系和区别

见上文
