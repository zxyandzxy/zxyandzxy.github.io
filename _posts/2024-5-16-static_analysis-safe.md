---
layout: post
title: "静态分析：安全"
date: 2024-5-16
tags: [static-analysis]
comments: true
author: zxy
---

### Information Flow Security

#### 访问控制（access control）

- 检查程序是否具有访问某些信息的权利/权限
- 关注如何访问信息

#### 信息流安全（information flow security）

- 跟踪信息如何流经程序以确保程序安全处理信息
- 关注信息如何传播
- 端到端
- 将程序变量划分为不同的安全级别
- 指定这些级别之间允许的流，即信息流策略

#### 安全级别与信息流策略

- 最简单的安全分级方法：高级+低级
- H, meaning _high_ security, secret information
- L, meaning _low_ security, public observable information
- 安全级别可以按照格进行建模 L <= H
- Noninterference policy（不干扰策略）
  - 要求高等级的变量信息对低等级的变量信息没有影响(即不应该干扰)
  - 直观地说，你不应该能够通过观察低密变量得出关于高密信息的任何结论
  - 一句话：高密信息不允许以任何形式传递给低密信息
  - ![image-20240120124710172](https://zxyandzxy.github.io/images/image-20240120124710172.png)

### Confidentiality and Integrity（对称的，可以用一种手段解决）

机密性（Confidentiality）

- 防止机密信息泄露（有用信息不要传给外界）
- 读保护：保护高等级信息不会流向低等级信息

完整性（Integrity）

- 防止不可信的信息破坏(可信的)关键信息（外界信息不要污染可靠信息）
- 写保护：不允许低等级数据流向高等级数据（可能会污染高级信息）
- 广义上指的是保证数据的正确性、完整性和一致性
  - 正确性:为了信息流的完整性，(可信的)关键数据不应该被不可信的数据破坏
  - 完整性：数据库系统应该完整地存储所有数据
  - 一致性：文件传输系统应确保发送端和接收端文件内容相同

### Explicit Flows and Covert Channels

#### 显式流（explicit flow）

- 直接通过赋值传递数据
- xH = yH xL = yH xL = yL + zH

#### 隐式流（implicit flow）

- 简单理解就是通过控制流去泄露高级信息
- 高等级信息影响控制流，我们就可以根据控制流信息的副作用反推高等级信息
- ![image-20240120125258195](https://zxyandzxy.github.io/images/image-20240120125258195.png)
- 这个例子里我们可以通过观察 publik 的值来确定 secret 的正负（泄露了 secret 的信息）

#### Covert/Hidden Channels（隐藏信道）

- 通过计算系统发送信息信号的机制称为信道。
- 利用不以信息传递为主要目的的机制的通道称为隐蔽通道
- ![image-20240120125515909](https://zxyandzxy.github.io/images/image-20240120125515909.png)
- 上述四种均隐藏信道泄露了信息，分别通过布尔控制流，程序是否终止，程序执行时间，程序异常状态泄露了高等级信息。
- 一般来说，显式流会比隐藏信道泄露更多信息（见下例）
- ![image-20240120125735366](https://zxyandzxy.github.io/images/image-20240120125735366.png)

### Taint Analysis

- 污点分析是最常见的信息流分析。它将程序数据分为两类:
- 感兴趣的数据，某些类型的标签与数据相关联，称为污点数据
- 其他数据，称为非污点数据
- 污点数据的来源称为数据源。
- 在实践中，污点数据通常来自某些方法的返回值(被视为源)。
- 污点分析跟踪污点数据如何流经程序，并观察它们是否可以流向感兴趣的位置(称为 sink)。
- 在实践中，sink 通常是一些敏感的方法
- 污点分析回答：污点数据是否会流向 sink / sink 的指针会指向污点数据

#### 污点分析结合指针分析（上下文不敏感指针分析）

##### 域与标号

![image-20240120130316624](https://zxyandzxy.github.io/images/image-20240120130316624.png)

##### 输入输出

- Sources：一组源方法(对这些方法的调用返回污点数据)
- Sinks：一组带有敏感参数的接收方法(流向这些方法参数的污点数据违反了安全策略)
- TaintFlows：<污点数据源，sink 方法>的集合

##### 规则

![image-20240120130537173](https://zxyandzxy.github.io/images/image-20240120130537173.png)

栗子：slide 77 页
