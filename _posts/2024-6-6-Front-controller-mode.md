---
layout: post
title: "前端控制器模式"
date: 2024-6-6
tags: [design-pattern]
comments: true
author: zxy
---

## 概要

> 意图

使用一个中心控制器（或处理器）来转发客户端请求到适当的处理程序。

> 主要解决的问题

- 解决 Web 应用程序中请求处理分散的问题，提供统一的请求处理入口。

> 使用场景

- 当需要对 Web 应用程序中的请求进行统一管理和分发时。

> 实现方式

- **前端控制器**：作为请求的单一入口点，负责请求的接收和转发。
- **视图**：用于呈现处理结果。
- **处理程序**：实际执行请求处理的组件。

> 关键代码

- **前端控制器**：包含逻辑以决定将请求转发到哪个处理程序。
- **处理程序映射**：将请求映射到相应的处理程序。

> 应用实例

- **Web 框架**：如 Spring MVC 中的 DispatcherServlet，作为前端控制器。

> 优点

1. **集中请求处理**：简化请求处理流程，易于管理和维护。
2. **减少代码重复**：通过重用控制器减少视图和处理程序中的重复代码。
3. **易于扩展**：新增请求处理逻辑时，只需添加新的处理程序。

> 缺点

- **可能成为性能瓶颈**：所有请求都通过前端控制器，可能影响性能。

> 使用建议

- 当需要构建一个具有清晰请求处理流程的 Web 应用程序时，考虑使用前端控制器模式。

> 注意事项

- 确保前端控制器不会由于集中处理所有请求而成为性能瓶颈。

> 包含的几个主要角色

1. **前端控制器（Front Controller）**：
   - 作为请求的单一入口点，负责接收请求并决定如何处理。
2. **处理程序（Handler）**：
   - 实际执行请求处理的组件。
3. **视图（View）**：
   - 用于呈现处理程序生成的响应。
4. **处理程序映射（Handler Mapping）（可选）**：
   - 将请求映射到相应的处理程序。
5. **客户端（Client）（可选）**：
   - 发出请求的 Web 浏览器或 API 客户端。

前端控制器模式通过提供一个集中的请求处理机制，有助于构建易于维护和扩展的 Web 应用程序。

## 实现

我们将创建 _FrontController_、_Dispatcher_ 分别当作前端控制器和调度器。_HomeView_ 和 _StudentView_ 表示各种为前端控制器接收到的请求而创建的视图。

_FrontControllerPatternDemo_，我们的演示类使用 _FrontController_ 来演示前端控制器设计模式。

![前端控制器模式的 UML 图](https://www.runoob.com/wp-content/uploads/2014/08/frontcontroller_pattern_uml_diagram.jpg)

**步骤 1**

创建视图。

```java
HomeView.java
public class HomeView {
   public void show(){
      System.out.println("Displaying Home Page");
   }
}

StudentView.java
public class StudentView {
   public void show(){
      System.out.println("Displaying Student Page");
   }
}
```

**步骤 2**

创建调度器 Dispatcher。

```java
Dispatcher.java
public class Dispatcher {
   private StudentView studentView;
   private HomeView homeView;
   public Dispatcher(){
      studentView = new StudentView();
      homeView = new HomeView();
   }

   public void dispatch(String request){
      if(request.equalsIgnoreCase("STUDENT")){
         studentView.show();
      }else{
         homeView.show();
      }
   }
}
```

**步骤 3**

创建前端控制器 FrontController。

```java
FrontController.java
public class FrontController {

   private Dispatcher dispatcher;

   public FrontController(){
      dispatcher = new Dispatcher();
   }

   private boolean isAuthenticUser(){
      System.out.println("User is authenticated successfully.");
      return true;
   }

   private void trackRequest(String request){
      System.out.println("Page requested: " + request);
   }

   public void dispatchRequest(String request){
      //记录每一个请求
      trackRequest(request);
      //对用户进行身份验证
      if(isAuthenticUser()){
         dispatcher.dispatch(request);
      }
   }
}
```

**步骤 4**

使用 _FrontController_ 来演示前端控制器设计模式。

```java
FrontControllerPatternDemo.java
public class FrontControllerPatternDemo {
   public static void main(String[] args) {
      FrontController frontController = new FrontController();
      frontController.dispatchRequest("HOME");
      frontController.dispatchRequest("STUDENT");
   }
}
```
