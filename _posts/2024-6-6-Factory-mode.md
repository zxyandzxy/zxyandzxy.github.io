---
layout: post
title: "工厂模式"
date: 2024-6-6
tags: [design-pattern]
comments: true
author: zxy
---

### 定义

- 属于创建型模式，只要给出要创建对象的名称，就可以创建具体的类对象。相当于把多个类封装到一个工厂里面，由这个工厂统一创建所有类的对象
- 工厂方法一定是包装复杂类的对象，简单类直接用 new 新建对象就好
- 特点是将产品类的对象的构建放到工厂类中

### 优点

- 提供了创建对象的统一接口，不需要知道类怎么实现，只需要知道类的名称就可以了
- 可拓展性高，想要添加对象，就在工厂中添加一个类

### 缺点

- 每次拓展一个类，就要修改工厂类的源代码
- 在现实使用中，产品类是特别多的，所以会造成系统中类个数成倍增加，工厂类巨大

### 核心结构

抽象产品 + 具体产品 + 工厂

![工厂模式的 UML 图](https://www.runoob.com/wp-content/uploads/2014/08/AB6B814A-0B09-4863-93D6-1E22D6B07FF8.jpg)

### 代码实现

```java
Shape.java
public interface Shape {
   void draw();
}

Rectangle.java
public class Rectangle implements Shape {
   @Override
   public void draw() {
      System.out.println("Inside Rectangle::draw() method.");
   }
}

Square.java
public class Square implements Shape {
   @Override
   public void draw() {
      System.out.println("Inside Square::draw() method.");
   }
}

Circle.java
public class Circle implements Shape {
   @Override
   public void draw() {
      System.out.println("Inside Circle::draw() method.");
   }
}

ShapeFactory.java
public class ShapeFactory {
   //使用 getShape 方法获取形状类型的对象
   public Shape getShape(String shapeType){
      if(shapeType == null){
         return null;
      }
      if(shapeType.equalsIgnoreCase("CIRCLE")){
         return new Circle();
      } else if(shapeType.equalsIgnoreCase("RECTANGLE")){
         return new Rectangle();
      } else if(shapeType.equalsIgnoreCase("SQUARE")){
         return new Square();
      }
      return null;
   }
}
```
