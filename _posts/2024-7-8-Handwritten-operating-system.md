---
layout: post
title: "30天自制操作系统"
date: 2024-7-8
tags: [books]
comments: true
author: zxy
---

> 编程之美阅读笔记
>
> 书本笔记链接：[read_books: 一些书本的阅读 (gitee.com)](https://gitee.com/zou-xinyu6/read_books)

## 第0天：着手开发之前

> 什么是开发操作系统

所谓开发操作系统，就是制作一张“含有操作系统的，能够自动启动的硬盘。具体流程如下：

1. 在Windows（或其他）系统上编写源代码
2. 用C语言编译器编译源代码，生成机器语言文件
3. 对生成的机器语言文件进行加工，生成软盘映像文件
4. 将映像文件写入磁盘，变成含有操作系统的启动盘

>开发操作系统使用什么语言

使用C语言和汇编语言

C语言是依赖操作系统功能最少的语言了，基本上只要不用函数就不会有问题

C语言要执行需要编译器，但是开发应用程序和开发操作系统用到的编译器是不同的（功能少了）。

我们最终的目的其实是搞出机器代码来，所以在没有开发操作系统用到的编译器的情况下，我们采用直接书写汇编语言来实现。

---

第1天：从计算机结构到汇编程序入门

计算机运行的核心是CPU，CPU是机械执行的，也就是识别0,1代表的电信号，然后执行。

所以程序底层的本质就是0,1代码，所有顶层的东西必须转化成0,1代码才能执行

所以无论是十进制，八进制程序，图像，音乐，最终都会处理为0,1二进制程序

---



第2天：汇编语言学习与Makefile入门





---



第3天：进入32位模式并导入C语言





---



第4天：C语言与画面显示的练习





---



第5天：结构体、文字显示与GDT/IDT初始化





---



第6天：分割编译与中断处理





---



第7天：FIFO与鼠标控制





---



第8天：鼠标控制与32位模式切换





---



第9天：内存管理





---



第10天：叠加处理





---



第11天：制作窗口





---



第12天：定时器（1）





---



第13天：定时器（2）





---



第14天：高分辨率及键盘输入





---



第15天：多任务（1）





---



第16天：多任务（2）





---



第17天：命令行窗口





---



第18天：dir命令





---



第19天：应用程序





---



第20天：API





---



第21天：保护操作系统





---



第22天：用C语言编写应用程序





---



第23天：图形处理相关





---



第24天：窗口操作





---



第25天：增加命令行窗口





---



第26天：为窗口移动提速





---



第27天：LDT与库





---



第28天：文件操作与文件显示





---



第29天：压缩与简单的应用程序





---



第30天：高级的应用程序





---



第31天：写在开发完成之后



