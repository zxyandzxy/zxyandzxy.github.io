---
layout: post
title: "指针分析"
date: 2024-5-16
tags: [static-analysis]
comments: true
author: zxy
---

### Motivation（动机）

![image-20240119195824538](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119195824538.png)

因为 CHA 不准确，所以要进行指针分析

### Introduction to Pointer Analysis

#### 基本信息

- 研究一个指针可以指向那些内存位置
- 在面向对象（OO）语言中，研究一个指针可以指向那些对象
- 是一个 may analysis，只要有可能指向就要添加进入指向的集合中
- 区别于别名分析（Alias Analysis）：研究两个指针可不可以指向同一个对象
- 是最最最基础的静态分析。

### Key Factors of Pointer Analysis

很多因素会影响指针分析的精度和效率

| Factor                              | Problem（研究什么）  | Choice（本课程会学习）                |
| ----------------------------------- | -------------------- | ------------------------------------- |
| heap abstraction（堆抽象）          | 怎么对堆内存进行建模 | allocation-site                       |
| context sensitivity（上下文敏感性） | 怎么建模调用上下文   | context-sensitive context-insensitive |
| flow sensitivity（流敏感性）        | 怎么建模控制流信息   | flow-insensitive                      |
| analysis scope（分析域）            | 分析程序的哪些部分   | whole-program                         |

#### heap abstraction

- 在动态执行中，由于循环和递归，堆对象的数量可以不受限制
- 为了确保终止，堆抽象将动态分配的无界具体对象建模为静态分析的有限抽象对象

![image-20240119200814763](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119200814763.png)

##### allocation-site

根据具体对象的 allocation-site 对其建模，即在每一个 allocation-site 建立一个抽象对象来表示在这个 allocation-site 分配的所有具体对象（allocation-site 可以理解为程序的某一行）

![image-20240119201220199](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119201220199.png)

由于程序的 allocation-site 有限，所以抽象对象肯定有限

#### context sensitivity

| context-sensitive                  | context-insensitive                      |
| ---------------------------------- | ---------------------------------------- |
| 区分方法的不同调用上下文           | 合并一个方法的所有调用上下文             |
| 对每个方法的多个上下文一一进行分析 | 每个方法分析一次（合并控制流会丢失精度） |

#### flow sensitivity

| flow-sensitive                   | flow-insensitive                 |
| -------------------------------- | -------------------------------- |
| 尊重语句的执行顺序               | 忽略控制流的顺序                 |
| 在每个程序位置维护点到关系的映射 | 为整个程序维护一个点到关系的映射 |

![image-20240119201813663](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119201813663.png)

#### analysis scope

| whole-program                | demand-program                         |
| ---------------------------- | -------------------------------------- |
| 计算程序中所有指针的指向信息 | 仅计算特定感兴趣的 site 的指针指向信息 |
| 为所有可能的需求提供信息     | 只提供特定信息                         |

### Concerned Statements

![image-20240119202213277](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240119202213277.png)

只关心可能引起指针指向对象变化的语句，就上面 5 种

### 重点问题

#### 什么是指针分析

指针分析是计算机科学领域的一个概念，主要用于程序静态分析和优化。在程序中，指针是一种数据类型，用于存储内存地址，可以指向其他变量或数据结构。指针分析旨在确定程序中的指针可以指向哪些内存位置，以及在程序执行期间可能出现的情况。

指针分析的主要目标包括：

1. **指针关系分析**：确定指针变量可能指向的内存位置。这可以帮助理解程序中数据的流动和依赖关系。

2. **指针别名分析**：确定程序中哪些指针可能引用同一内存位置，即别名。这对于优化程序以及检测潜在的错误非常重要。

3. **空指针分析**：检测程序中可能存在的空指针引用，以避免潜在的崩溃或错误。

4. **指针的生命周期分析**：确定指针何时创建、使用和销毁，以便进行资源管理和优化。

指针分析在编译器优化、程序验证和安全性分析等领域都有广泛的应用。通过有效的指针分析，可以帮助开发人员理解和改进程序的性能、可靠性和安全性。

#### 理解指针分析的关键影响因素

堆抽象，上下文敏感性与否，流敏感与否，分析域大小

#### 理解我们在指针分析中分析什么

new 语句，store 语句，load 语句，赋值语句，调用语句
