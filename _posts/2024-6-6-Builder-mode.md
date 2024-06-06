---
layout: post
title: "建造者模式"
date: 2024-6-6
tags: [design-pattern]
comments: true
author: zxy
---

## 概要

> 意图

将一个复杂的构建过程与其表示相分离，使得同样的构建过程可以创建不同的表示。

> 主要解决

在软件系统中，一个复杂对象的创建通常由多个部分组成，这些部分的组合经常变化，但组合的算法相对稳定。

> 何时使用 及 使用场景

当一些基本部件不变，而其组合经常变化时。

- 需要生成的对象具有复杂的内部结构。
- 需要生成的对象内部属性相互依赖。

> 关键代码

- **建造者**：创建并提供实例。
- **导演**：管理建造出来的实例的依赖关系和控制构建过程。

> 如何解决

将变与不变的部分分离开。

> 结构

建造者模式包含以下几个主要角色：

- **产品（Product）**：要构建的复杂对象。产品类通常包含多个部分或属性。
- **抽象建造者（Builder）**：定义了构建产品的抽象接口，包括构建产品的各个部分的方法。
- **具体建造者（Concrete Builder）**：实现抽象建造者接口，具体确定如何构建产品的各个部分，并负责返回最终构建的产品。
- **指导者（Director）**：负责调用建造者的方法来构建产品，指导者并不了解具体的构建过程，只关心产品的构建顺序和方式。

> 优点

- 分离构建过程和表示，使得构建过程更加灵活，可以构建不同的表示。
- 可以更好地控制构建过程，隐藏具体构建细节。
- 代码复用性高，可以在不同的构建过程中重复使用相同的建造者。

> 缺点

- 如果产品的属性较少，建造者模式可能会导致代码冗余。
- 增加了系统的类和对象数量。

## 实现

我们假设一个快餐店的商业案例，其中，一个典型的套餐可以是一个汉堡（Burger）和一杯冷饮（Cold drink）。汉堡（Burger）可以是素食汉堡（Veg Burger）或鸡肉汉堡（Chicken Burger），它们是包在纸盒中。冷饮（Cold drink）可以是可口可乐（coke）或百事可乐（pepsi），它们是装在瓶子中。

我们将创建一个表示食物条目（比如汉堡和冷饮）的 _Item_ 接口和实现 _Item_ 接口的实体类，以及一个表示食物包装的 _Packing_ 接口和实现 _Packing_ 接口的实体类，汉堡是包在纸盒中，冷饮是装在瓶子中。

然后我们创建一个 _Meal_ 类，带有 _Item_ 的 _ArrayList_ 和一个通过结合 _Item_ 来创建不同类型的 _Meal_ 对象的 _MealBuilder_。_BuilderPatternDemo_ 类使用 _MealBuilder_ 来创建一个 _Meal_。

![建造者模式的 UML 图](https://www.runoob.com/wp-content/uploads/2014/08/20210315-builder-pattern.svg)

### 步骤 1

创建一个表示食物条目和食物包装的接口。

```java
Item.java
public interface Item {
   public String name();
   public Packing packing();
   public float price();
}

Packing.java
public interface Packing {
   public String pack();
}
```

### 步骤 2

创建实现 Packing 接口的实体类。

```java
Wrapper.java
public class Wrapper implements Packing {

   @Override
   public String pack() {
      return "Wrapper";
   }
}

Bottle.java
public class Bottle implements Packing {

   @Override
   public String pack() {
      return "Bottle";
   }
}
```

### 步骤 3

创建实现 Item 接口的抽象类，该类提供了默认的功能。

```java
Burger.java
public abstract class Burger implements Item {

   @Override
   public Packing packing() {
      return new Wrapper();
   }

   @Override
   public abstract float price();
}

ColdDrink.java
public abstract class ColdDrink implements Item {

    @Override
    public Packing packing() {
       return new Bottle();
    }

    @Override
    public abstract float price();
}
```

### 步骤 4

创建扩展了 Burger 和 ColdDrink 的实体类。

```java
VegBurger.java
public class VegBurger extends Burger {

   @Override
   public float price() {
      return 25.0f;
   }

   @Override
   public String name() {
      return "Veg Burger";
   }
}


ChickenBurger.java
public class ChickenBurger extends Burger {

   @Override
   public float price() {
      return 50.5f;
   }

   @Override
   public String name() {
      return "Chicken Burger";
   }
}

Coke.java
public class Coke extends ColdDrink {

   @Override
   public float price() {
      return 30.0f;
   }

   @Override
   public String name() {
      return "Coke";
   }
}

Pepsi.java
public class Pepsi extends ColdDrink {

   @Override
   public float price() {
      return 35.0f;
   }

   @Override
   public String name() {
      return "Pepsi";
   }
}
```

### 步骤 5

创建一个 Meal 类，带有上面定义的 Item 对象。

```java
Meal.java
import java.util.ArrayList;
import java.util.List;

public class Meal {
   private List<Item> items = new ArrayList<Item>();

   public void addItem(Item item){
      items.add(item);
   }

   public float getCost(){
      float cost = 0.0f;
      for (Item item : items) {
         cost += item.price();
      }
      return cost;
   }

   public void showItems(){
      for (Item item : items) {
         System.out.print("Item : "+item.name());
         System.out.print(", Packing : "+item.packing().pack());
         System.out.println(", Price : "+item.price());
      }
   }
}
```

### 步骤 6

创建一个 MealBuilder 类，实际的 builder 类负责创建 Meal 对象。

```java
MealBuilder.java
public class MealBuilder {

   public Meal prepareVegMeal (){
      Meal meal = new Meal();
      meal.addItem(new VegBurger());
      meal.addItem(new Coke());
      return meal;
   }

   public Meal prepareNonVegMeal (){
      Meal meal = new Meal();
      meal.addItem(new ChickenBurger());
      meal.addItem(new Pepsi());
      return meal;
   }
}
```

### 步骤 7

BuiderPatternDemo 使用 MealBuilder 来演示建造者模式（Builder Pattern）。

```java
BuilderPatternDemo.java
public class BuilderPatternDemo {
   public static void main(String[] args) {
      MealBuilder mealBuilder = new MealBuilder();

      Meal vegMeal = mealBuilder.prepareVegMeal();
      System.out.println("Veg Meal");
      vegMeal.showItems();
      System.out.println("Total Cost: " +vegMeal.getCost());

      Meal nonVegMeal = mealBuilder.prepareNonVegMeal();
      System.out.println("\n\nNon-Veg Meal");
      nonVegMeal.showItems();
      System.out.println("Total Cost: " +nonVegMeal.getCost());
   }
}
```
