---
layout: post
title: "组合模式"
date: 2024-6-6
tags: [design-pattern]
comments: true
author: zxy
---

## 概要

> 意图

将对象组合成树形结构以表示"部分-整体"的层次结构。组合模式使得用户对单个对象和组合对象的使用具有一致性。

> 主要解决的问题

- 简化树形结构中对象的处理，无论它们是单个对象还是组合对象。
- 解耦客户端代码与复杂元素的内部结构，使得客户端可以统一处理所有类型的节点。

> 使用场景

- 当需要表示对象的层次结构时，如文件系统或组织结构。
- 当希望客户端代码能够以一致的方式处理树形结构中的所有对象时。

> 实现方式

- **统一接口**：定义一个接口，所有对象（树枝和叶子）都实现这个接口。
- **组合结构**：树枝对象包含一个接口的引用列表，这些引用可以是叶子或树枝。

> 关键代码

- **Component 接口**：定义了所有对象必须实现的操作。
- **Leaf 类**：实现 Component 接口，代表树中的叶子节点。
- **Composite 类**：也实现 Component 接口，并包含其他 Component 对象的集合。

> 应用实例

1. **算术表达式**：构建一个由操作数、操作符和子表达式组成的树形结构。
2. **GUI 组件**：在 Java 的 AWT 和 Swing 库中，容器（如 Panel）可以包含其他组件（如按钮和复选框）。

> 优点

1. **简化客户端代码**：客户端可以统一处理所有类型的节点。
2. **易于扩展**：可以轻松添加新的叶子类型或树枝类型。

> 缺点

- **违反依赖倒置原则**：组件的声明是基于具体类而不是接口，这可能导致代码的灵活性降低。

> 使用建议

- 在设计时，优先使用接口而非具体类，以提高系统的灵活性和可维护性。
- 适用于需要处理复杂树形结构的场景，如文件系统、组织结构等。

> 注意事项

- 在实现时，确保所有组件都遵循统一的接口，以保持一致性。
- 考虑使用工厂模式来创建不同类型的组件，以进一步解耦组件的创建逻辑。

> 结构

**组合模式的核心角色包括：**

- **组件（Component）:**
  - 定义了组合中所有对象的通用接口，可以是抽象类或接口。它声明了用于访问和管理子组件的方法，包括添加、删除、获取子组件等。
- **叶子节点（Leaf）:**
  - 表示组合中的叶子节点对象，叶子节点没有子节点。它实现了组件接口的方法，但通常不包含子组件。
- **复合节点（Composite）:**
  - 表示组合中的复合对象，复合节点可以包含子节点，可以是叶子节点，也可以是其他复合节点。它实现了组件接口的方法，包括管理子组件的方法。
- **客户端（Client）:**
  - 通过组件接口与组合结构进行交互，客户端不需要区分叶子节点和复合节点，可以一致地对待整体和部分。

## 实现

我们有一个类 _Employee_，该类被当作组合模型类。_CompositePatternDemo_ 类使用 _Employee_ 类来添加部门层次结构，并打印所有员工。

![组合模式的 UML 图](https://www.runoob.com/wp-content/uploads/2014/08/20210817-composite-composite.svg)

### 步骤 1

创建 _Employee_ 类，该类带有 _Employee_ 对象的列表。

```java
Employee.java
import java.util.ArrayList;
import java.util.List;

public class Employee {
   private String name;
   private String dept;
   private int salary;
   private List<Employee> subordinates;

   //构造函数
   public Employee(String name,String dept, int sal) {
      this.name = name;
      this.dept = dept;
      this.salary = sal;
      subordinates = new ArrayList<Employee>();
   }

   public void add(Employee e) {
      subordinates.add(e);
   }

   public void remove(Employee e) {
      subordinates.remove(e);
   }

   public List<Employee> getSubordinates(){
     return subordinates;
   }

   public String toString(){
      return ("Employee :[ Name : "+ name
      +", dept : "+ dept + ", salary :"
      + salary+" ]");
   }
}
```

### 步骤 2

使用 _Employee_ 类来创建和打印员工的层次结构。

```java
CompositePatternDemo.java
public class CompositePatternDemo {
   public static void main(String[] args) {
      Employee CEO = new Employee("John","CEO", 30000);

      Employee headSales = new Employee("Robert","Head Sales", 20000);

      Employee headMarketing = new Employee("Michel","Head Marketing", 20000);

      Employee clerk1 = new Employee("Laura","Marketing", 10000);
      Employee clerk2 = new Employee("Bob","Marketing", 10000);

      Employee salesExecutive1 = new Employee("Richard","Sales", 10000);
      Employee salesExecutive2 = new Employee("Rob","Sales", 10000);

      CEO.add(headSales);
      CEO.add(headMarketing);

      headSales.add(salesExecutive1);
      headSales.add(salesExecutive2);

      headMarketing.add(clerk1);
      headMarketing.add(clerk2);

      //打印该组织的所有员工 （这里一般使用递归，因为树高未知）
      System.out.println(CEO);
      for (Employee headEmployee : CEO.getSubordinates()) {
         System.out.println(headEmployee);
         for (Employee employee : headEmployee.getSubordinates()) {
            System.out.println(employee);
         }
      }
   }
}
```
