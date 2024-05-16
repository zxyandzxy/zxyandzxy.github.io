---
layout: post
title: "过程间分析"
date: 2024-5-16
tags: [static-analysis]
comments: true
author: zxy
---

### Motivation（过程间分析动机）

![image-20240119175155112](https://zxyandzxy.github.io/images/image-20240119175155112.png)

### Call Graph Construction

#### 调用图（call graph）

本质上调用图就是一组从 call site 到调用目标方法的边

![image-20240119175450259](https://zxyandzxy.github.io/images/image-20240119175450259.png)

#### class hierarchy analysis（CHA）类层次调用分析

##### Java 中的方法调用类型

![image-20240119175624114](https://zxyandzxy.github.io/images/image-20240119175624114.png)

##### 虚拟调用中的 dispatch

虚拟调用关注两个东西：调用方法的对象，在 call site 的方法签名

![image-20240119175853991](https://zxyandzxy.github.io/images/image-20240119175853991.png)

dispatch：寻找要调用的方法

![image-20240119180021169](https://zxyandzxy.github.io/images/image-20240119180021169.png)

##### CHA 实现（class hierarchy analysis）

A a = ... a.foo();

假设变量 a 可能指向类 A 的对象或类 A 的所有子类的对象

通过查找类 A 的类层次结构来解析目标方法

解决方式：cs 是 call site，就是调用方法的那行代码
![image-20240119180513358](https://zxyandzxy.github.io/images/image-20240119180513358.png)

情况(1)：call site 是静态调用，目标方法就是 m

情况(2)：![image-20240119180741203](https://zxyandzxy.github.io/images/image-20240119180741203.png)

special call 包括：私有方法，构造函数，父类方法

情况(3)：虚拟调用，那么就找 c 及 c 的所有子类进行**dispatch**（不要代入你的个人分析代码的思路，严格执行 dispatch 逻辑）

![image-20240418110212962](https://zxyandzxy.github.io/images/image-20240418110212962.png)

##### 优缺点

优点：快，只考虑 call site 声明的变量的类型及其继承层次结构，忽略数据流和控制流信息

缺点：不准确，容易引入伪目标方法

#### 调用图构造算法

![image-20240119193359494](https://zxyandzxy.github.io/images/image-20240119193359494.png)

整体思路：

- 从 main 方法开始
- 对于每一个可达方法，通过 CHA 解决目标方法
- 重复上述操作，直至没有新方法被发现了
- 例子 slide 57 页

### Interprocedural Control-Flow Graph（ICFG 过程间控制流图）

程序的 ICFG = 程序中每个方法的 CFG + 两种附加边

两种附加边：

- 调用边:从调用点到被调用方的入口节点
- 返回边:从被调用方的出口节点到其调用点(即返回点)后面的语句。

![image-20240119194441553](https://zxyandzxy.github.io/images/image-20240119194441553.png)

### Interprocedural Data-Flow Analysis

#### 对比过程内分析和过程间分析

![image-20240119194604956](https://zxyandzxy.github.io/images/image-20240119194604956.png)

edge transfer

- 调用边转换：传递参数
- 返回边转换：传递返回值

![image-20240119194908205](https://zxyandzxy.github.io/images/image-20240119194908205.png)

注意：黑色虚线称为 call-to-return edge，这条边仍然要保留

因为如果没有这条边，我们就要通过其他方法传播本地数据流（这个例子里是 a 的值 6，如果没有这条边，那么 a=6 就要传递到方法 addone 里再传回来，很麻烦），这非常低效。

这条边另一个操作是杀死左手边的值（就是返回值赋予的对象，这个例子里是 b = 7 要被杀死，因为 return 10 会被赋予 b，先杀死 b = 7 能够提高精度，否则 b = NAC）

### 重点问题：

#### 1.怎么通过类层次分析构造调用图

- 见上文构造图调用算法。
- 主要思想是通过 BFS 进行所有可达方法的寻找
- 其中涉及 dispatch 和 resolve 算法

#### 2.过程间控制流图的概念

见上文过程间控制流图

#### 3.过程间数据流分析的概念

见上文过程间数据流分析

#### 4.过程间常量传播

见上文例子
