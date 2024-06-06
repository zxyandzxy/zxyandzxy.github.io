---
layout: post
title: "服务定位器模式"
date: 2024-6-6
tags: [design-pattern]
comments: true
author: zxy
---

## 概要

> 意图

用于在应用程序中提供一个中心化的服务访问点，用以获取各种服务或资源。

> 主要解决的问题

- 解决在应用程序中需要访问各种服务或资源时，避免硬编码服务访问逻辑的问题。

> 使用场景

- 当应用程序需要访问多个外部服务或资源，并且希望将这些访问逻辑集中管理时。

> 实现方式

- **服务定位器接口**：定义获取服务的方法。
- **服务定位器实现**：实现服务定位器接口，封装服务的查找和访问逻辑。
- **服务接口**：定义服务的接口或契约。
- **具体服务**：实现服务接口，提供具体的服务功能。

> 关键代码

- **服务定位器**：包含逻辑以查找和返回请求的服务实例。
- **服务接口**：定义服务的抽象表示。

> 应用实例

- **企业应用程序**：应用程序使用服务定位器来访问数据库服务、消息队列服务等。

> 优点

1. **解耦服务访问**：将服务访问逻辑与使用服务的业务逻辑分离。
2. **集中管理**：服务的访问点集中管理，便于维护和扩展。
3. **灵活性**：易于添加、修改或替换服务。

> 缺点

- **性能问题**：服务定位器可能引入性能开销，特别是在每次服务请求都进行查找时。
- **过度使用**：可能导致设计模式的滥用，从而隐藏系统结构。

> 使用建议

- 当应用程序需要以一种灵活且可维护的方式访问多个服务时，考虑使用服务定位器模式。

> 注意事项

- 避免过度依赖服务定位器，因为它可能掩盖系统的依赖关系，使得调试和优化变得困难。

> 包含的几个主要角色

1. **服务定位器接口（Service Locator Interface）**：
   - 定义获取服务的方法。
2. **服务定位器实现（Service Locator Implementation）**：
   - 实现服务定位器接口，封装服务的查找和访问逻辑。
3. **服务接口（Service Interface）**：
   - 定义服务的抽象表示。
4. **具体服务（Concrete Service）**：
   - 实现服务接口，提供具体的服务功能。
5. **客户端（Client）**：
   - 使用服务定位器来访问所需的服务。

服务定位器模式通过提供一个集中的服务访问点，有助于简化服务访问逻辑，并提高应用程序的灵活性和可维护性。

## 实现

我们将创建 _ServiceLocator_、_InitialContext_、_Cache_、_Service_ 作为表示实体的各种对象。_Service1_ 和 _Service2_ 表示实体服务。

_ServiceLocatorPatternDemo_ 类在这里是作为一个客户端，将使用 _ServiceLocator_ 来演示服务定位器设计模式。

![服务定位器模式的 UML 图](https://www.runoob.com/wp-content/uploads/2014/08/20201015-service-locator.svg)

**步骤 1**

创建服务接口 Service。

```java
Service.java
public interface Service {
   public String getName();
   public void execute();
}
```

**步骤 2**

创建实体服务。

```java
Service1.java
public class Service1 implements Service {
   public void execute(){
      System.out.println("Executing Service1");
   }

   @Override
   public String getName() {
      return "Service1";
   }
}

Service2.java
public class Service2 implements Service {
   public void execute(){
      System.out.println("Executing Service2");
   }

   @Override
   public String getName() {
      return "Service2";
   }
}
```

**步骤 3**

为 JNDI 查询创建 InitialContext。

```java
InitialContext.java
public class InitialContext {
   public Object lookup(String jndiName){
      if(jndiName.equalsIgnoreCase("SERVICE1")){
         System.out.println("Looking up and creating a new Service1 object");
         return new Service1();
      }else if (jndiName.equalsIgnoreCase("SERVICE2")){
         System.out.println("Looking up and creating a new Service2 object");
         return new Service2();
      }
      return null;
   }
}
```

**步骤 4**

创建缓存 Cache。

```java
Cache.java
import java.util.ArrayList;
import java.util.List;

public class Cache {

   private List<Service> services;

   public Cache(){
      services = new ArrayList<Service>();
   }

   public Service getService(String serviceName){
      for (Service service : services) {
         if(service.getName().equalsIgnoreCase(serviceName)){
            System.out.println("Returning cached  "+serviceName+" object");
            return service;
         }
      }
      return null;
   }

   public void addService(Service newService){
      boolean exists = false;
      for (Service service : services) {
         if(service.getName().equalsIgnoreCase(newService.getName())){
            exists = true;
         }
      }
      if(!exists){
         services.add(newService);
      }
   }
}
```

**步骤 5**

创建服务定位器。

```java
ServiceLocator.java
public class ServiceLocator {
   private static Cache cache;

   static {
      cache = new Cache();
   }

   public static Service getService(String jndiName){

      Service service = cache.getService(jndiName);

      if(service != null){
         return service;
      }

      InitialContext context = new InitialContext();
      Service service1 = (Service)context.lookup(jndiName);
      cache.addService(service1);
      return service1;
   }
}
```

**步骤 6**

使用 _ServiceLocator_ 来演示服务定位器设计模式。

```java
ServiceLocatorPatternDemo.java
public class ServiceLocatorPatternDemo {
   public static void main(String[] args) {
      Service service = ServiceLocator.getService("Service1");
      service.execute();
      service = ServiceLocator.getService("Service2");
      service.execute();
      service = ServiceLocator.getService("Service1");
      service.execute();
      service = ServiceLocator.getService("Service2");
      service.execute();
   }
}
```
