---
layout: post
title: "备忘录模式"
date: 2024-6-6
tags: [design-pattern]
comments: true
author: zxy
---

## 概要

> 意图

在不破坏封装性的前提下，捕获一个对象的内部状态，并允许在对象之外保存和恢复这些状态。

> 主要解决的问题

- 允许捕获并保存一个对象的内部状态，以便在将来可以恢复到该状态，实现撤销和回滚操作。

> 使用场景

- 当需要提供一种撤销机制，允许用户回退到之前的状态时。

> 实现方式

- **创建备忘录类**：用于存储和封装对象的状态。
- **创建发起人角色**：负责创建备忘录，并根据需要恢复状态。
- **创建备忘录管理类**（可选）：负责管理所有备忘录对象。

> 关键代码

- **备忘录**：存储发起人的状态信息。
- **发起人**：创建备忘录，并根据备忘录恢复状态。

> 优点

- **提供状态恢复机制**：允许用户方便地回到历史状态。
- **封装状态信息**：用户不需要关心状态的保存细节。

> 缺点

- **资源消耗**：如果对象的状态复杂，保存状态可能会占用较多资源。

> 使用建议

- 在需要保存和恢复数据状态的场景中使用备忘录模式。
- 考虑使用原型模式结合备忘录模式，以节约内存。

> 注意事项

- 为了降低耦合度，应通过备忘录管理类间接管理备忘录对象。
- 备忘录模式应谨慎使用，避免过度消耗系统资源。

> 结构

备忘录模式包含以下几个主要角色：

- **备忘录（Memento）**：负责存储原发器对象的内部状态。备忘录可以保持原发器的状态的一部分或全部信息。
- **原发器（Originator）**：创建一个备忘录对象，并且可以使用备忘录对象恢复自身的内部状态。原发器通常会在需要保存状态的时候创建备忘录对象，并在需要恢复状态的时候使用备忘录对象。
- **负责人（Caretaker）**：负责保存备忘录对象，但是不对备忘录对象进行操作或检查。负责人只能将备忘录传递给其他对象。

## 实现

备忘录模式使用三个类 _Memento_、_Originator_ 和 _CareTaker_。Memento 包含了要被恢复的对象的状态。Originator 创建并在 Memento 对象中存储状态。Caretaker 对象负责从 Memento 中恢复对象的状态。

_MementoPatternDemo_，我们的演示类使用 _CareTaker_ 和 _Originator_ 对象来显示对象的状态恢复。

![备忘录模式的 UML 图](https://www.runoob.com/wp-content/uploads/2014/08/memento-20220601-memento.svg)

### 步骤 1

创建 Memento 类。

```java
Memento.java
public class Memento {
   private String state;

   public Memento(String state){
      this.state = state;
   }

   public String getState(){
      return state;
   }
}
```

### 步骤 2

创建 Originator 类。

```java
Originator.java
public class Originator {
   private String state;

   public void setState(String state){
      this.state = state;
   }

   public String getState(){
      return state;
   }

   public Memento saveStateToMemento(){
      return new Memento(state);
   }

   public void getStateFromMemento(Memento Memento){
      state = Memento.getState();
   }
}
```

### 步骤 3

创建 CareTaker 类。

```java
CareTaker.java
import java.util.ArrayList;
import java.util.List;

public class CareTaker {
   private List<Memento> mementoList = new ArrayList<Memento>();

   public void add(Memento state){
      mementoList.add(state);
   }

   public Memento get(int index){
      return mementoList.get(index);
   }
}
```

### 步骤 4

使用 _CareTaker_ 和 _Originator_ 对象。

```java
MementoPatternDemo.java
public class MementoPatternDemo {
   public static void main(String[] args) {
      Originator originator = new Originator();
      CareTaker careTaker = new CareTaker();
      originator.setState("State #1");
      originator.setState("State #2");
      careTaker.add(originator.saveStateToMemento());
      originator.setState("State #3");
      careTaker.add(originator.saveStateToMemento());
      originator.setState("State #4");

      System.out.println("Current State: " + originator.getState());
      originator.getStateFromMemento(careTaker.get(0));
      System.out.println("First saved State: " + originator.getState());
      originator.getStateFromMemento(careTaker.get(1));
      System.out.println("Second saved State: " + originator.getState());
   }
}
```
