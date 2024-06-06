---
layout: post
title: "单例模式"
date: 2024-6-6
tags: [design-pattern]
comments: true
author: zxy
---

## 概要

> 意图

确保一个类只有一个实例，并提供一个全局访问点来访问该实例。

> 主要解决

频繁创建和销毁全局使用的类实例的问题。

> 何时使用

当需要控制实例数目，节省系统资源时。

> 如何解决

检查系统是否已经存在该单例，如果存在则返回该实例；如果不存在则创建一个新实例。

> 关键代码

构造函数是私有的。

> 应用实例

- Windows 在多进程多线程环境下操作文件时，避免多个进程或线程同时操作一个文件，需要通过唯一实例进行处理。
- 设备管理器设计为单例模式，例如电脑有两台打印机，避免同时打印同一个文件。

> 优点

- 内存中只有一个实例，减少内存开销，尤其是频繁创建和销毁实例时（如管理学院首页页面缓存）。
- 避免资源的多重占用（如写文件操作）。

> 缺点

- 没有接口，不能继承。
- 与单一职责原则冲突，一个类应该只关心内部逻辑，而不关心实例化方式。

> 使用场景

- 生成唯一序列号。
- WEB 中的计数器，避免每次刷新都在数据库中增加计数，先缓存起来。
- 创建消耗资源过多的对象，如 I/O 与数据库连接等。

> 实现中的注意事项

- **线程安全**：`getInstance()` 方法中需要使用同步锁 `synchronized (Singleton.class)` 防止多线程同时进入造成实例被多次创建。
- **延迟初始化**：实例在第一次调用 `getInstance()` 方法时创建。
- **序列化和反序列化**：重写 `readResolve` 方法以确保反序列化时不会创建新的实例。
- **反射攻击**：在构造函数中添加防护代码，防止通过反射创建新实例。
- **类加载器问题**：注意复杂类加载环境可能导致的多个实例问题。

## 实现

我们将创建一个 _SingleObject_ 类。_SingleObject_ 类有它的私有构造函数和本身的一个静态实例。_SingleObject_ 类提供了一个静态方法，供外界获取它的静态实例。

_SingletonPatternDemo_ 类使用 _SingleObject_ 类来获取 _SingleObject_ 对象。

![单例模式的 UML 图](https://www.runoob.com/wp-content/uploads/2014/08/62576915-36E0-4B67-B078-704699CA980A.jpg)

> 代码实现

```java
SingleObject.java
public class SingleObject {
    //创建 SingleObject 对象
    private static SingleObject instance = new SingleObject();

    //让构造函数为 private, 防止该类被实例初始化
    private SingleObject() {}

    //获取唯一可用的对象
    public static SingleObject getInstance() {
        return instance;
    }

    public void showMessage() {
        System.out.println("hello world");
    }
}

SingletonPatternDemo.java
public class SinglePatternDemo {
    public static void main(String[] args) {
        // 获取唯一可用对象
        SingleObject object = SingleObject.getInstance();
        object.showMessage();
    }
}
```

## 单例模式的几种实现方式

> 什么叫做单例模式中的线程安全

单例模式中，重点就是怎么保证这个单例类只有一个实例，保证类只有 1 个实例就叫线程安全。饿汉模式调用静态方法只能拿到同一个实例，所以饿汉模式使用类加载机制天生实现了线程安全。

### 1、懒汉式，线程不安全

- **是否 Lazy 初始化：**是
- **是否多线程安全：**否
- **实现难度：**易

**描述：**这种方式是最基本的实现方式，这种实现最大的问题就是不支持多线程。因为没有加锁 synchronized，所以严格意义上它并不算单例模式。这种方式 lazy loading 很明显，不要求线程安全，在多线程不能正常工作。

```java
public class Singleton {
    private static Singleton instance;
    private Singleton (){}

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

### 2、懒汉式，线程安全

- **是否 Lazy 初始化：**是
- **是否多线程安全：**是
- **实现难度：**易

**描述：**这种方式具备很好的 lazy loading，能够在多线程中很好的工作，但是，效率很低，99% 情况下不需要同步。
优点：第一次调用才初始化，避免内存浪费。
缺点：必须加锁 synchronized 才能保证单例，但加锁会影响效率。
getInstance() 的性能对应用程序不是很关键（该方法使用不太频繁）。

```java
public class Singleton {
    private static Singleton instance;
    private Singleton (){}
    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

### 3、饿汉式

- **是否 Lazy 初始化：**否
- **是否多线程安全：**是
- **实现难度：**易

**描述：**这种方式比较常用，但容易产生垃圾对象。
优点：没有加锁，执行效率会提高。
缺点：类加载时就初始化，浪费内存。
它基于 classloader 机制避免了多线程的同步问题，不过，instance 在类装载时就实例化，虽然导致类装载的原因有很多种，在单例模式中大多数都是调用 getInstance 方法， 但是也不能确定有其他的方式（或者其他的静态方法）导致类装载，这时候初始化 instance 显然没有达到 lazy loading 的效果。

```java
public class Singleton {
    private static Singleton instance = new Singleton();
    private Singleton (){}
    public static Singleton getInstance() {
    	return instance;
    }
}
```

### 4、双检锁/双重校验锁（DCL，即 double-checked locking）

- **JDK 版本：**JDK1.5 起
- **是否 Lazy 初始化：**是
- **是否多线程安全：**是
- **实现难度：**较复杂

**描述：**这种方式采用双锁机制，安全且在多线程情况下能保持高性能。（没有将整个 getSingleton 方法进行加锁，只是对某些语句进行加锁）
getInstance() 的性能对应用程序很关键。

```java
public class Singleton {
    private volatile static Singleton singleton;
    private Singleton (){}
    public static Singleton getSingleton() {             // 1
        if (singleton == null) {                         // 2
            synchronized (Singleton.class) {             // 3
                if (singleton == null) {                 // 4
                    singleton = new Singleton();         // 5
                }
            }
        }
        return singleton;                                // 6
    }
}
```

> volatile 是神马东东

**volatile**是 Java 提供的一种轻量级的同步机制。Java 语言包含两种内在的同步机制：同步块（或方法）和 volatile 变量，相比于 synchronized（synchronized 通常称为重量级锁），volatile 更轻量级，因为**它不会引起线程上下文的切换和调度**。但是 volatile 变量的同步性较差（有时它更简单并且开销更低），而且其使用也更容易出错。

> 双检锁/双重校验锁加 volatile 的原因

在并发情况下，如果没有 volatile 关键字，在第 5 行会出现问题。instance = new TestInstance();可以分解为 3 行伪代码：

```java
a. memory = allocate() //分配内存
b. ctorInstanc(memory) //初始化对象
c. instance = memory   //设置instance指向刚分配的地址
```

上面的代码在编译运行时，可能会出现重排序从 a-b-c 排序为 a-c-b。在多线程的情况下会出现以下问题。当线程 A 在执行第 5 行代码时，B 线程进来执行到第 2 行代码。假设此时 A 执行的过程中发生了指令重排序，即先执行了 a 和 c，没有执行 b。那么由于 A 线程执行了 c 导致 instance 指向了一段地址，所以 B 线程判断 instance 不为 null，会直接跳到第 6 行并返回一个未初始化的对象。
