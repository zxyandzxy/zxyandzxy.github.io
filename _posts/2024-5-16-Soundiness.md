---
layout: post
title: "静态分析：soundiness"
date: 2024-5-16
tags: [static-analysis]
comments: true
author: zxy
---

### Soundness and Soundiness

- 到现在为止：学术界+工业界的静态分析都做不到完全的 sound

- 因为存在难以分析的特征：对这些特征的激进保守处理可能会使分析过于不精确而无法扩展，从而使分析无用

- 难以分析的特征：Java 反射机制，Java 本地代码，Java 动态类加载，C/C++的函数指针等

- 导致一个问题：

  - 静态分析工具：作者声称是 sound，但是只是大部分语言特征是 sound，对于 hard feature 是不 sound

  - 大部分论文对于 hard language feature 问题提都不提，或者随意的在小角落提一下

  - 这些 hard language feature 如果没有正确处理，可能会对分析结果有巨大影响
  - 对于非专家来说，他们可能会错误地认为分析是合理的，并自信地依赖分析结果
  - 对于专家来说，如果没有一个关于他们如何处理这些 hard language feature 的清晰解释，他们仍然很难解释分析结果(分析有多健全、多快速、多精确)

- 所以提出了 soundiness 这个词，用以强调那些 hard language feature 是怎么处理的，不要误导别人

### Hard Language Feature: Java Reflection

定义：![image-20240120153341230](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240120153341230.png)

为什么需要分析反射：

- Bug detection Security Analysis
- Verification + Optimization

怎么分析 Java 反射：

第一种：String Constant analysis + Pointer Analysis

- 思路是：知道字符串的值，我们就能解析反射目标，从而进行静态分析
- 但是字符串值的来源在静态分析中难以获取

第二种：Type Inference + String analysis + Pointer Analysis

- 在使用字符串的时候推导出来字符串的值
- 在使用这个字符串时，根据 Java 强大的类型信息及其他线索
- 推断出这个字符串必须具有的特性，然后推断出这个字符串的值

第三种：Assisted by Dynamic Analysis

- 曲线救国：先用一些测试用例跑一下程序，得出一些 string 值，然后将这些 string 的值用于静态分析（很直接的，但是不 soundness,因为测试用例无法保证覆盖所有程序执行路径），但是很准

### Hard Language Feature: Native Code

#### JNI

JNI 就是 JVM 提供的一个函数模块，允许 Java 和用 C++/C 写的本地代码交互

![image-20240120153756229](C:\Users\zxy\AppData\Roaming\Typora\typora-user-images\image-20240120153756229.png)

JNI 用于帮助使用内存管理，文件系统等和操作系统相关的机制，也可以使用 JNI 提供的一些功能，实现代码重用
