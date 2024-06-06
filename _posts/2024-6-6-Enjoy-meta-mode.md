---
layout: post
title: "享元模式"
date: 2024-6-6
tags: [design-pattern]
comments: true
author: zxy
---

## 概要

> 意图

通过共享对象来减少创建大量相似对象时的内存消耗。。

> 主要解决的问题

- 避免因创建大量对象而导致的内存溢出问题。
- 通过共享对象，提高内存使用效率。

> 使用场景

- 当系统中存在大量相似或相同的对象。
- 对象的创建和销毁成本较高。
- 对象的状态可以外部化，即对象的部分状态可以独立于对象本身存在。

> 实现方式

- **定义享元接口**：创建一个享元接口，规定可以共享的状态。
- **创建具体享元类**：实现该接口的具体类，包含内部状态。
- **使用享元工厂**：创建一个工厂类，用于管理享元对象的创建和复用。

> 关键代码

- **HashMap**：使用哈希表存储已经创建的享元对象，以便快速检索。

> 应用实例

1. **Java 中的 String 对象**：字符串常量池中已经存在的字符串会被复用。
2. **数据库连接池**：数据库连接被复用，避免频繁创建和销毁连接。

> 优点

- **减少内存消耗**：通过共享对象，减少了内存中对象的数量。
- **提高效率**：减少了对象创建的时间，提高了系统效率。

> 缺点

- **增加系统复杂度**：需要分离内部状态和外部状态，增加了设计和实现的复杂性。
- **线程安全问题**：如果外部状态处理不当，可能会引起线程安全问题。

> 使用建议

- 在创建大量相似对象时考虑使用享元模式。
- 确保享元对象的内部状态是共享的，而外部状态是独立于对象的。

> 注意事项

- **状态分离**：明确区分内部状态和外部状态，避免混淆。
- **享元工厂**：使用享元工厂来控制对象的创建和复用，确保对象的一致性和完整性。

> 结构

**享元模式包含以下几个核心角色：**

- **享元工厂（Flyweight Factory）:**
  - 负责创建和管理享元对象，通常包含一个池（缓存）用于存储和复用已经创建的享元对象。
- **具体享元（Concrete Flyweight）:**
  - 实现了抽象享元接口，包含了内部状态和外部状态。内部状态是可以被共享的，而外部状态则由客户端传递。
- **抽象享元（Flyweight）:**
  - 定义了具体享元和非共享享元的接口，通常包含了设置外部状态的方法。
- **客户端（Client）:**
  - 使用享元工厂获取享元对象，并通过设置外部状态来操作享元对象。客户端通常不需要关心享元对象的具体实现。

## 实现

我们将创建一个 _Shape_ 接口和实现了 _Shape_ 接口的实体类 _Circle_。下一步是定义工厂类 _ShapeFactory_。

_ShapeFactory_ 有一个 _Circle_ 的 _HashMap_，其中键名为 _Circle_ 对象的颜色。无论何时接收到请求，都会创建一个特定颜色的圆。_ShapeFactory_ 检查它的 _HashMap_ 中的 circle 对象，如果找到 _Circle_ 对象，则返回该对象，否则将创建一个存储在 hashmap 中以备后续使用的新对象，并把该对象返回到客户端。

_FlyWeightPatternDemo_ 类使用 _ShapeFactory_ 来获取 _Shape_ 对象。它将向 _ShapeFactory_ 传递信息（_red / green / blue/ black / white_），以便获取它所需对象的颜色。

![享元模式的 UML 图](https://www.runoob.com/wp-content/uploads/2014/08/20201015-fiyweight.svg)

### 步骤 1

创建一个接口。

```java
Shape.java
public interface Shape {
   void draw();
}
```

### 步骤 2

创建实现接口的实体类。

```java
Circle.java
public class Circle implements Shape {
   private String color;
   private int x;
   private int y;
   private int radius;

   public Circle(String color){
      this.color = color;
   }

   public void setX(int x) {
      this.x = x;
   }

   public void setY(int y) {
      this.y = y;
   }

   public void setRadius(int radius) {
      this.radius = radius;
   }

   @Override
   public void draw() {
      System.out.println("Circle: Draw() [Color : " + color
         +", x : " + x +", y :" + y +", radius :" + radius);
   }
}
```

### 步骤 3

创建一个工厂，生成基于给定信息的实体类的对象。

```java
ShapeFactory.java
import java.util.HashMap;

public class ShapeFactory {
   private static final HashMap<String, Shape> circleMap = new HashMap<>();

   public static Shape getCircle(String color) {
      Circle circle = (Circle)circleMap.get(color);

      if(circle == null) {
         circle = new Circle(color);
         circleMap.put(color, circle);
         System.out.println("Creating circle of color : " + color);
      }
      return circle;
   }
}
```

### 步骤 4

使用该工厂，通过传递颜色信息来获取实体类的对象。

```java
FlyweightPatternDemo.java
public class FlyweightPatternDemo {
   private static final String colors[] =
      { "Red", "Green", "Blue", "White", "Black" };
   public static void main(String[] args) {

      for(int i=0; i < 20; ++i) {
         Circle circle =
            (Circle)ShapeFactory.getCircle(getRandomColor());
         circle.setX(getRandomX());
         circle.setY(getRandomY());
         circle.setRadius(100);
         circle.draw();
      }
   }
   private static String getRandomColor() {
      return colors[(int)(Math.random()*colors.length)];
   }
   private static int getRandomX() {
      return (int)(Math.random()*100 );
   }
   private static int getRandomY() {
      return (int)(Math.random()*100);
   }
}
```
