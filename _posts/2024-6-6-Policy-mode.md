---
layout: post
title: "策略模式"
date: 2024-6-6
tags: [design-pattern]
comments: true
author: zxy
---

## 概要

> 意图

将每个算法封装起来，使它们可以互换使用。

> 主要解决的问题

- 解决在多种相似算法存在时，使用条件语句（如 if...else）导致的复杂性和难以维护的问题。

> 使用场景

- 当一个系统中有许多类，它们之间的区别仅在于它们的行为时。

> 实现方式

- **定义策略接口**：所有策略类都将实现这个统一的接口。
- **创建具体策略类**：每个策略类封装一个具体的算法或行为。
- **上下文类**：包含一个策略对象的引用，并通过该引用调用策略。

> 关键代码

- **策略接口**：规定了所有策略类必须实现的方法。
- **具体策略类**：实现了策略接口，包含具体的算法实现。

> 优点

1. **算法切换自由**：可以在运行时根据需要切换算法。
2. **避免多重条件判断**：消除了复杂的条件语句。
3. **扩展性好**：新增算法只需新增一个策略类，无需修改现有代码。

> 缺点

1. **策略类数量增多**：每增加一个算法，就需要增加一个策略类。
2. **所有策略类都需要暴露**：策略类需要对外公开，以便可以被选择和使用。

> 使用建议

- 当系统中有多种算法或行为，且它们之间可以相互替换时，使用策略模式。
- 当系统需要动态选择算法时，策略模式是一个合适的选择。

> 注意事项

- 如果系统中策略类数量过多，考虑使用其他模式或设计技巧来解决类膨胀问题。

> 结构

**策略模式包含以下几个核心角色：**

- 环境（Context）：维护一个对策略对象的引用，负责将客户端请求委派给具体的策略对象执行。环境类可以通过依赖注入、简单工厂等方式来获取具体策略对象。
- 抽象策略（Abstract Strategy）：定义了策略对象的公共接口或抽象类，规定了具体策略类必须实现的方法。
- 具体策略（Concrete Strategy）：实现了抽象策略定义的接口或抽象类，包含了具体的算法实现。

策略模式通过将算法与使用算法的代码解耦，提供了一种动态选择不同算法的方法。客户端代码不需要知道具体的算法细节，而是通过调用环境类来使用所选择的策略。

## 实现

我们将创建一个定义活动的 _Strategy_ 接口和实现了 _Strategy_ 接口的实体策略类。_Context_ 是一个使用了某种策略的类。

_StrategyPatternDemo_，我们的演示类使用 _Context_ 和策略对象来演示 Context 在它所配置或使用的策略改变时的行为变化。

![策略模式的 UML 图](https://www.runoob.com/wp-content/uploads/2014/08/strategy_pattern_uml_diagram.jpg)

### 步骤 1

创建一个接口。

```java
Strategy.java
public interface Strategy {
   public int doOperation(int num1, int num2);
}
```

### 步骤 2

创建实现接口的实体类。

```java
OperationAdd.java
public class OperationAdd implements Strategy{
   @Override
   public int doOperation(int num1, int num2) {
      return num1 + num2;
   }
}

OperationSubtract.java
public class OperationSubtract implements Strategy{
   @Override
   public int doOperation(int num1, int num2) {
      return num1 - num2;
   }
}

OperationMultiply.java
public class OperationMultiply implements Strategy{
   @Override
   public int doOperation(int num1, int num2) {
      return num1 * num2;
   }
}
```

### 步骤 3

创建 _Context_ 类。

```java
Context.java
public class Context {
   private Strategy strategy;

   public Context(Strategy strategy){
      this.strategy = strategy;
   }

   public int executeStrategy(int num1, int num2){
      return strategy.doOperation(num1, num2);
   }
}
```

### 步骤 4

使用 _Context_ 来查看当它改变策略 _Strategy_ 时的行为变化。

```java
StrategyPatternDemo.java
public class StrategyPatternDemo {
   public static void main(String[] args) {
      Context context = new Context(new OperationAdd());
      System.out.println("10 + 5 = " + context.executeStrategy(10, 5));

      context = new Context(new OperationSubtract());
      System.out.println("10 - 5 = " + context.executeStrategy(10, 5));

      context = new Context(new OperationMultiply());
      System.out.println("10 * 5 = " + context.executeStrategy(10, 5));
   }
}
```
