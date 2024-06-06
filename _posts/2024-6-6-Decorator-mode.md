---
layout: post
title: "装饰器模式"
date: 2024-6-6
tags: [design-pattern]
comments: true
author: zxy
---

## 概要

> 意图

动态地给一个对象添加额外的职责，同时不改变其结构。装饰器模式提供了一种灵活的替代继承方式来扩展功能。

> 主要解决的问题

- 避免通过继承引入静态特征，特别是在子类数量急剧膨胀的情况下。
- 允许在运行时动态地添加或修改对象的功能。

> 使用场景

- 当需要在不增加大量子类的情况下扩展类的功能。
- 当需要动态地添加或撤销对象的功能。

> 实现方式

- **定义组件接口**：创建一个接口，规定可以动态添加职责的对象的标准。
- **创建具体组件**：实现该接口的具体类，提供基本功能。
- **创建抽象装饰者**：实现同样的接口，持有一个组件接口的引用，可以在任何时候动态地添加功能。
- **创建具体装饰者**：扩展抽象装饰者，添加额外的职责。

> 关键代码

- **Component 接口**：定义了原始对象和装饰器对象的公共接口或抽象类，可以是具体组件类的父类或接口。
- **ConcreteComponent 类**：是被装饰的原始对象，它定义了需要添加新功能的对象。
- **Decorator 抽象类**：继承自抽象组件，它包含了一个抽象组件对象，并定义了与抽象组件相同的接口，同时可以通过组合方式持有其他装饰器对象。
- **ConcreteDecorator 类**：实现了抽象装饰器的接口，负责向抽象组件添加新的功能。具体装饰器通常会在调用原始对象的方法之前或之后执行自己的操作。

> 优点

- **低耦合**：装饰类和被装饰类可以独立变化，互不影响。
- **灵活性**：可以动态地添加或撤销功能。
- **替代继承**：提供了一种继承之外的扩展对象功能的方式。

> 缺点

- **复杂性**：多层装饰可能导致系统复杂性增加。

> 使用建议

- 在需要动态扩展功能时，考虑使用装饰器模式。
- 保持装饰者和具体组件的接口一致，以确保灵活性。

> 注意事项

- 装饰器模式可以替代继承，但应谨慎使用，避免过度装饰导致系统复杂。

## 实现

我们将创建一个 _Shape_ 接口和实现了 _Shape_ 接口的实体类。然后我们创建一个实现了 _Shape_ 接口的抽象装饰类 _ShapeDecorator_，并把 _Shape_ 对象作为它的实例变量。

_RedShapeDecorator_ 是实现了 _ShapeDecorator_ 的实体类。

_DecoratorPatternDemo_ 类使用 _RedShapeDecorator_ 来装饰 _Shape_ 对象。

![装饰器模式的 UML 图](https://www.runoob.com/wp-content/uploads/2014/08/20210420-decorator-1-decorator-decorator.svg)

### 步骤 1

创建一个接口：

```java
Shape.java
public interface Shape {
   void draw();
}
```

### 步骤 2

创建实现接口的实体类。

```java
Rectangle.java
public class Rectangle implements Shape {

   @Override
   public void draw() {
      System.out.println("Shape: Rectangle");
   }
}

Circle.java
public class Circle implements Shape {

   @Override
   public void draw() {
      System.out.println("Shape: Circle");
   }
}
```

### 步骤 3

创建实现了 _Shape_ 接口的抽象装饰类。

```java
ShapeDecorator.java
public abstract class ShapeDecorator implements Shape {
   protected Shape decoratedShape;

   public ShapeDecorator(Shape decoratedShape){
      this.decoratedShape = decoratedShape;
   }

   public void draw(){
      decoratedShape.draw();
   }
}
```

### 步骤 4

创建扩展了 _ShapeDecorator_ 类的实体装饰类。

```java
RedShapeDecorator.java
public class RedShapeDecorator extends ShapeDecorator {

   public RedShapeDecorator(Shape decoratedShape) {
      super(decoratedShape);
   }

   @Override
   public void draw() {
      decoratedShape.draw();
      setRedBorder(decoratedShape);
   }

   private void setRedBorder(Shape decoratedShape){
      System.out.println("Border Color: Red");
   }
}
```

### 步骤 5

使用 _RedShapeDecorator_ 来装饰 _Shape_ 对象。

```java
DecoratorPatternDemo.java
public class DecoratorPatternDemo {
   public static void main(String[] args) {

      Shape circle = new Circle();
      ShapeDecorator redCircle = new RedShapeDecorator(new Circle());
      ShapeDecorator redRectangle = new RedShapeDecorator(new Rectangle());
      //Shape redCircle = new RedShapeDecorator(new Circle());
      //Shape redRectangle = new RedShapeDecorator(new Rectangle());
      System.out.println("Circle with normal border");
      circle.draw();

      System.out.println("\nCircle of red border");
      redCircle.draw();

      System.out.println("\nRectangle of red border");
      redRectangle.draw();
   }
}
```
