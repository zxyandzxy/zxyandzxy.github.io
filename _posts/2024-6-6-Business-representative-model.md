---
layout: post
title: "业务代表模式"
date: 2024-6-6
tags: [design-pattern]
comments: true
author: zxy
---

## 概要

> 意图

抽象和封装应用程序的访问逻辑，从而为表示层提供对业务逻辑层的访问。

> 主要解决的问题

- 解决表示层与业务逻辑层之间的耦合问题，允许表示层通过业务代表间接访问业务逻辑层。

> 使用场景

- 当需要在多层应用程序中清晰地分离表示层和业务逻辑层时。

> 实现方式

- **业务代表接口**：定义访问业务逻辑的方法。
- **业务代表实现**：实现业务代表接口，封装调用业务逻辑层的逻辑。
- **业务服务**：业务逻辑层的接口或类，包含业务操作。

> 关键代码

- **业务代表接口**：声明访问业务逻辑的方法。
- **业务代表实现**：实现接口，包含调用业务服务的代码。
- **业务服务**：业务逻辑的具体实现。

> 应用实例

- **Web 应用程序**：Web 层作为表示层，通过业务代表访问后端服务层。

> 优点

- **表示层与业务逻辑层解耦**：业务代表作为中间层，降低耦合度。
- **集中访问逻辑**：简化表示层的代码，将访问逻辑集中在业务代表中。
- **易于维护和扩展**：添加新的业务逻辑访问时，只需修改业务代表。

> 缺点

- **可能增加复杂性**：对于简单应用程序，可能增加不必要的抽象层次。

> 使用建议

- 当开发多层应用程序，需要在表示层和业务逻辑层之间提供清晰分离时，考虑使用业务代表模式。

> 注意事项

- 业务代表应该尽量简洁，避免包含复杂的逻辑。

> 包含的几个主要角色

- **业务代表接口（Business Delegate Interface）**：
  - 声明访问业务逻辑的方法。
- **业务代表实现（Business Delegate Implementation）**：
  - 实现业务代表接口，封装对业务服务的调用。
- **业务服务（Business Service）**：
  - 业务逻辑层的接口或类，包含实际的业务操作。
- **表示层（Presentation Layer）**：
  - 应用程序的前端，通过业务代表访问业务逻辑。
- **业务逻辑层（Business Logic Layer）**：
  - 包含应用程序的核心业务逻辑。

业务代表模式通过提供一个抽象层来访问业务逻辑，有助于保持应用程序的灵活性和可维护性。

## 实现

我们将创建 _Client_、_BusinessDelegate_、_BusinessService_、_BusinessLookUp_、_JMSService_ 和 _EJBService_ 来表示业务代表模式中的各种实体。

_BusinessDelegatePatternDemo_ 类使用 _BusinessDelegate_ 和 _Client_ 来演示业务代表模式的用法。

![业务代表模式的 UML 图](https://www.runoob.com/wp-content/uploads/2014/08/business.svg)

**步骤 1**

创建 BusinessService 接口。

```java
BusinessService.java
public interface BusinessService {
   public void doProcessing();
}
```

**步骤 2**

创建实体服务类。

```java
EJBService.java
public class EJBService implements BusinessService {

   @Override
   public void doProcessing() {
      System.out.println("Processing task by invoking EJB Service");
   }
}

JMSService.java
public class JMSService implements BusinessService {

   @Override
   public void doProcessing() {
      System.out.println("Processing task by invoking JMS Service");
   }
}
```

**步骤 3**

创建业务查询服务。

```java
BusinessLookUp.java
public class BusinessLookUp {
   public BusinessService getBusinessService(String serviceType){
      if(serviceType.equalsIgnoreCase("EJB")){
         return new EJBService();
      }else {
         return new JMSService();
      }
   }
}
```

**步骤 4**

创建业务代表。

```java
BusinessDelegate.java
public class BusinessDelegate {
   private BusinessLookUp lookupService = new BusinessLookUp();
   private BusinessService businessService;
   private String serviceType;

   public void setServiceType(String serviceType){
      this.serviceType = serviceType;
   }

   public void doTask(){
      businessService = lookupService.getBusinessService(serviceType);
      businessService.doProcessing();
   }
}
```

**步骤 5**

创建客户端。

```java
Client.java
public class Client {

   BusinessDelegate businessService;

   public Client(BusinessDelegate businessService){
      this.businessService  = businessService;
   }

   public void doTask(){
      businessService.doTask();
   }
}
```

**步骤 6**

使用 BusinessDelegate 和 Client 类来演示业务代表模式。

```java
BusinessDelegatePatternDemo.java
public class BusinessDelegatePatternDemo {

   public static void main(String[] args) {

      BusinessDelegate businessDelegate = new BusinessDelegate();
      businessDelegate.setServiceType("EJB");

      Client client = new Client(businessDelegate);
      client.doTask();

      businessDelegate.setServiceType("JMS");
      client.doTask();
   }
}
```
