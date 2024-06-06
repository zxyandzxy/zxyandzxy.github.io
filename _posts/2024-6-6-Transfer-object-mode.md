---
layout: post
title: "传输对象模式"
date: 2024-6-6
tags: [design-pattern]
comments: true
author: zxy
---

## 概要

> 意图

用于简化网络或应用程序层之间的数据传输。它通过创建一个包含多个属性的类来封装数据，这些属性代表需要传输的数据。

> 主要解决的问题

- 解决在分布式系统中，尤其是多层应用程序中，数据在层与层之间传输时的效率和封装性问题。

> 使用场景

- 当需要在客户端和服务器之间传输大量数据时，特别是 Web 服务或远程过程调用（RPC）。

> 实现方式

- **传输对象类**：定义一个包含多个属性的类，这些属性代表需要传输的数据。
- **数据访问层**：负责从数据库或其他数据源提取数据，并填充到传输对象中。
- **业务逻辑层**：可选，处理业务逻辑，可能需要使用传输对象。
- **表示层或客户端**：使用传输对象来显示数据或作为用户输入的数据载体。

> 关键代码

- **传输对象类**：包含属性和可能的业务逻辑，用于数据的封装和传输。

> 应用实例

- **Web 应用程序**：表示层通过传输对象从服务器获取或发送数据。

> 优点

1. **减少网络通信开销**：通过一次性传输多个数据，减少网络请求次数。
2. **封装性**：传输对象作为数据的封装载体，隐藏了内部数据结构。
3. **易于维护**：集中管理传输数据，便于修改和扩展。

> 缺点

- **过度使用**：可能会因为创建过多的传输对象而导致系统复杂性增加。

> 使用建议

- 当需要在多层应用程序的不同层之间传输大量数据时，考虑使用传输对象模式。

> 注意事项

- 传输对象应该保持轻量级，避免包含过多的业务逻辑。

> 包含的几个主要角色

1. **传输对象类（Transfer Object Class）**：
   - 封装了需要传输的数据。
2. **数据访问层（Data Access Layer）**：
   - 提供数据，并可能负责将数据填充到传输对象中。
3. **业务逻辑层（Business Logic Layer）（可选）**：
   - 可选，处理业务逻辑，可能使用传输对象。
4. **表示层或客户端（Presentation Layer or Client）**：
   - 使用传输对象来显示数据或发送请求。

传输对象模式通过封装传输数据为一个对象，提供了一种有效的方式来减少网络通信次数，并简化数据传输过程。

## 实现

我们将创建一个作为业务对象的 _StudentBO_ 和作为传输对象的 _StudentVO_，它们都代表了我们的实体。

_TransferObjectPatternDemo_ 类在这里是作为一个客户端，将使用 _StudentBO_ 和 _Student_ 来演示传输对象设计模式。

![传输对象模式的 UML 图](https://www.runoob.com/wp-content/uploads/2014/08/20201015-transfer.svg)

**步骤 1**

创建传输对象。

```java
StudentVO.java
public class StudentVO {
   private String name;
   private int rollNo;

   StudentVO(String name, int rollNo){
      this.name = name;
      this.rollNo = rollNo;
   }

   public String getName() {
      return name;
   }

   public void setName(String name) {
      this.name = name;
   }

   public int getRollNo() {
      return rollNo;
   }

   public void setRollNo(int rollNo) {
      this.rollNo = rollNo;
   }
}
```

**步骤 2**

创建业务对象。

```java
StudentBO.java
import java.util.ArrayList;
import java.util.List;

public class StudentBO {

   //列表是当作一个数据库
   List<StudentVO> students;

   public StudentBO(){
      students = new ArrayList<StudentVO>();
      StudentVO student1 = new StudentVO("Robert",0);
      StudentVO student2 = new StudentVO("John",1);
      students.add(student1);
      students.add(student2);
   }
   public void deleteStudent(StudentVO student) {
      students.remove(student.getRollNo());
      System.out.println("Student: Roll No "
      + student.getRollNo() +", deleted from database");
   }

   //从数据库中检索学生名单
   public List<StudentVO> getAllStudents() {
      return students;
   }

   public StudentVO getStudent(int rollNo) {
      return students.get(rollNo);
   }

   public void updateStudent(StudentVO student) {
      students.get(student.getRollNo()).setName(student.getName());
      System.out.println("Student: Roll No "
      + student.getRollNo() +", updated in the database");
   }
}
```

**步骤 3**

使用 _StudentBO_ 来演示传输对象设计模式。

```java
TransferObjectPatternDemo.java
public class TransferObjectPatternDemo {
   public static void main(String[] args) {
      StudentBO studentBusinessObject = new StudentBO();

      //输出所有的学生
      for (StudentVO student : studentBusinessObject.getAllStudents()) {
         System.out.println("Student: [RollNo : "
         +student.getRollNo()+", Name : "+student.getName()+" ]");
      }

      //更新学生
      StudentVO student =studentBusinessObject.getAllStudents().get(0);
      student.setName("Michael");
      studentBusinessObject.updateStudent(student);

      //获取学生
      studentBusinessObject.getStudent(0);
      System.out.println("Student: [RollNo : "
      +student.getRollNo()+", Name : "+student.getName()+" ]");
   }
}
```
