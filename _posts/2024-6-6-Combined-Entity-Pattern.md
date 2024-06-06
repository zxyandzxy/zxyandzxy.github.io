---
layout: post
title: "组合实体模式"
date: 2024-6-6
tags: [design-pattern]
comments: true
author: zxy
---

## 概要

> 意图

将数据库中的表转换为应用程序中的组合对象，这些对象可以表示表中的单个记录或一组记录。

> 主要解决的问题

- 解决在对象关系映射中，如何高效地表示和管理数据库中的复杂数据结构，特别是当存在一对多或多对多关系时。

> 使用场景

- 当数据库表具有复杂的关系，或者需要将数据库表映射为应用程序中的复合对象时。

> 实现方式

- **组合实体类**：代表数据库中的表，可以包含单个记录或一组记录。
- **叶子实体类**：代表表中的单个记录。
- **容器实体类**：代表表中的一组记录，可以包含其他叶子或容器实体。

> 关键代码

- **组合实体类**：包含方法来添加、删除和访问子实体。
- **叶子实体类**：实现组合实体接口，代表单个记录。
- **容器实体类**：实现组合实体接口，可以包含多个叶子或容器实体。

> 优点

- **简化数据访问**：通过对象的方式来简化数据库记录的访问和管理。
- **提高代码可读性**：使代码更贴近自然语言，易于理解和维护。
- **易于扩展**：可以方便地添加新的实体类型或修改现有实体。

> 缺点

- **性能问题**：在处理大量数据时，可能会遇到性能瓶颈。
- **复杂性**：对于简单的数据模型，可能会增加不必要的复杂性。

> 使用建议

- 当需要表示和管理数据库中的复杂数据结构时，考虑使用组合实体模式。

> 注意事项

- 确保组合实体模式的使用不会引入性能问题，特别是在处理大量数据时。

> 包含的几个主要角色

- **组合实体（Composite Entity）**：
  - 代表数据库中的表，可以包含单个或多个记录。
- **叶子实体（Leaf Entity）**：
  - 代表表中的单个记录，不包含子实体。
- **容器实体（Container Entity）**：
  - 代表表中的一组记录，可以包含其他叶子或容器实体。
- **数据访问对象（Data Access Object）（可选）**：
  - 用于访问和操作数据库，封装了数据访问逻辑。
- **客户端（Client）（可选）**：
  - 使用组合实体模式来访问和操作数据库记录。

组合实体模式通过将数据库表映射为应用程序中的对象，提供了一种直观和灵活的方式来处理复杂的数据关系。

## 实现

我们将创建作为组合实体的 _CompositeEntity_ 对象。_CoarseGrainedObject_ 是一个包含依赖对象的类。

_CompositeEntityPatternDemo_，我们的演示类使用 _Client_ 类来演示组合实体模式的用法。

![组合实体模式的 UML 图](https://www.runoob.com/wp-content/uploads/2014/08/compositeentity_pattern_uml_diagram.jpg)

**步骤 1**

创建依赖对象。

```java
DependentObject1.java
public class DependentObject1 {

   private String data;

   public void setData(String data){
      this.data = data;
   }

   public String getData(){
      return data;
   }
}

DependentObject2.java
public class DependentObject2 {

   private String data;

   public void setData(String data){
      this.data = data;
   }

   public String getData(){
      return data;
   }
}
```

**步骤 2**

创建粗粒度对象。

```java
CoarseGrainedObject.java
public class CoarseGrainedObject {
   DependentObject1 do1 = new DependentObject1();
   DependentObject2 do2 = new DependentObject2();

   public void setData(String data1, String data2){
      do1.setData(data1);
      do2.setData(data2);
   }

   public String[] getData(){
      return new String[] {do1.getData(),do2.getData()};
   }
}
```

**步骤 3**

创建组合实体。

```java
CompositeEntity.java
public class CompositeEntity {
   private CoarseGrainedObject cgo = new CoarseGrainedObject();

   public void setData(String data1, String data2){
      cgo.setData(data1, data2);
   }

   public String[] getData(){
      return cgo.getData();
   }
}
```

**步骤 4**

创建使用组合实体的客户端类。

```java
Client.java
public class Client {
   private CompositeEntity compositeEntity = new CompositeEntity();

   public void printData(){
      for (int i = 0; i < compositeEntity.getData().length; i++) {
         System.out.println("Data: " + compositeEntity.getData()[i]);
      }
   }

   public void setData(String data1, String data2){
      compositeEntity.setData(data1, data2);
   }
}
```

**步骤 5**

使用 _Client_ 来演示组合实体设计模式的用法。

```java
CompositeEntityPatternDemo.java
public class CompositeEntityPatternDemo {
   public static void main(String[] args) {
       Client client = new Client();
       client.setData("Test", "Data");
       client.printData();
       client.setData("Second Test", "Data1");
       client.printData();
   }
}
```
