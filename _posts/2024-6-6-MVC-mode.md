---
layout: post
title: "MVC模式"
date: 2024-6-6
tags: [design-pattern]
comments: true
author: zxy
---

## 概要

> 意图

将应用程序分为三个核心组件：模型（Model）、视图（View）和控制器（Controller），以实现关注点分离。

> 主要解决的问题

- 解决了应用程序中业务逻辑、数据和界面显示的耦合问题，使得开发和维护更加清晰和简单。

> 使用场景

- 当需要将数据、业务逻辑和界面显示分离，以便于独立开发和维护时。

> 实现方式

- **模型（Model）**：负责数据和业务逻辑，通常包含数据存储、检索和业务规则。
- **视图（View）**：负责显示数据（模型）的用户界面，不包含业务逻辑。
- **控制器（Controller）**：接收用户的输入，调用模型和视图去完成用户的请求。

> 关键代码

- **模型**：包含业务逻辑和数据状态。
- **视图**：包含展示逻辑，将模型的数据渲染为用户界面。
- **控制器**：包含逻辑，用于响应用户输入并更新模型和视图。

> 应用实例

- **Web 应用程序**：用户通过浏览器（视图）发送请求，服务器端的控制器处理请求，模型进行数据处理。

> 优点

- **关注点分离**：将数据、业务逻辑和界面显示分离，降低耦合度。
- **易于维护**：每个组件负责特定的任务，便于单独开发和维护。
- **可扩展性**：可以独立地替换或更新模型、视图或控制器。

> 缺点

- **可能增加复杂性**：对于简单项目，引入 MVC 可能会增加不必要的复杂性。
- **性能问题**：如果不正确使用，可能会导致性能问题。

> 使用建议

- 当开发大型应用程序，需要清晰分离数据、业务逻辑和用户界面时，考虑使用 MVC 模式。

> 注意事项

- 确保模型、视图和控制器之间的交互清晰，避免相互依赖。

> 包含的几个主要角色

- **模型（Model）**：
  - 管理数据和业务逻辑。
- **视图（View）**：
  - 显示模型中的数据，提供用户界面。
- **控制器（Controller）**：
  - 接收用户输入，调用模型和视图进行处理。
- **用户（User）（可选）**：
  - 与视图交互，触发应用程序的流程。

## 实现

我们将创建一个作为模型的 _Student_ 对象。_StudentView_ 是一个把学生详细信息输出到控制台的视图类，_StudentController_ 是负责存储数据到 _Student_ 对象中的控制器类，并相应地更新视图 _StudentView_。

_MVCPatternDemo_，我们的演示类使用 _StudentController_ 来演示 MVC 模式的用法。

![MVC 模式的 UML 图](https://www.runoob.com/wp-content/uploads/2014/08/mvc-3.svg)

**步骤 1**

创建模型。

```java
Student.java
public class Student {
   private String rollNo;
   private String name;
   public String getRollNo() {
      return rollNo;
   }
   public void setRollNo(String rollNo) {
      this.rollNo = rollNo;
   }
   public String getName() {
      return name;
   }
   public void setName(String name) {
      this.name = name;
   }
}
```

**步骤 2**

创建视图。

```java
StudentView.java
public class StudentView {
   public void printStudentDetails(String studentName, String studentRollNo){
      System.out.println("Student: ");
      System.out.println("Name: " + studentName);
      System.out.println("Roll No: " + studentRollNo);
   }
}
```

**步骤 3**

创建控制器。

```java
StudentController.java
public class StudentController {
   private Student model;
   private StudentView view;

   public StudentController(Student model, StudentView view){
      this.model = model;
      this.view = view;
   }

   public void setStudentName(String name){
      model.setName(name);
   }

   public String getStudentName(){
      return model.getName();
   }

   public void setStudentRollNo(String rollNo){
      model.setRollNo(rollNo);
   }

   public String getStudentRollNo(){
      return model.getRollNo();
   }

   public void updateView(){
      view.printStudentDetails(model.getName(), model.getRollNo());
   }
}
```

**步骤 4**

使用 _StudentController_ 方法来演示 MVC 设计模式的用法。

```
MVCPatternDemo.java
public class MVCPatternDemo {
   public static void main(String[] args) {

      //从数据库获取学生记录
      Student model  = retrieveStudentFromDatabase();

      //创建一个视图：把学生详细信息输出到控制台
      StudentView view = new StudentView();

      StudentController controller = new StudentController(model, view);

      controller.updateView();

      //更新模型数据
      controller.setStudentName("John");

      controller.updateView();
   }

   private static Student retrieveStudentFromDatabase(){
      Student student = new Student();
      student.setName("Robert");
      student.setRollNo("10");
      return student;
   }
}
```
