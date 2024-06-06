---
layout: post
title: "拦截过滤器模式"
date: 2024-6-6
tags: [design-pattern]
comments: true
author: zxy
---

## 概要

> 意图

用于在请求到达最终目的地之前，通过一系列过滤器对请求进行预处理和后处理。

> 主要解决的问题

- 解决 Web 应用程序中需要在请求处理前后执行通用操作（如日志记录、安全检查、事务管理等）的问题。

> 使用场景

- 当需要在请求处理流程中插入额外的逻辑，而这些逻辑与业务逻辑无关时。

> 实现方式

- **过滤器链**：一系列过滤器按照特定顺序执行。
- **过滤器**：对请求和响应进行预处理和后处理的组件。
- **目标对象**：请求处理的最终目的地。

> 关键代码

- **过滤器接口**：定义了过滤器必须实现的方法，如`before()`和`after()`。
- **具体过滤器**：实现过滤器接口，包含具体的处理逻辑。
- **过滤器管理器**：负责过滤器链的维护和过滤器的调用顺序。

> 应用实例

- **Web 应用服务器**：如 Tomcat 使用过滤器进行请求的拦截处理，例如权限检查、日志记录等。

> 优点

1. **职责分离**：将通用操作与业务逻辑分离，提高代码的可维护性。
2. **可扩展性**：可以灵活地添加、删除或修改过滤器。
3. **复用性**：过滤器可以在不同的请求处理中重用。

> 缺点

- **性能影响**：如果过滤器链过长或过滤器执行的操作复杂，可能会影响性能。

> 使用建议

- 当需要在请求处理流程中插入与业务逻辑无关的通用操作时，考虑使用拦截过滤器模式。

> 注意事项

- 确保过滤器的执行顺序符合预期，以避免逻辑错误。

> 包含的几个主要角色

1. **过滤器接口（Filter Interface）**：
   - 定义了过滤器必须实现的方法。
2. **具体过滤器（Concrete Filter）**：
   - 实现过滤器接口，包含具体的处理逻辑。
3. **过滤器管理器（Filter Manager）**：
   - 负责维护过滤器链和调用过滤器。
4. **目标对象（Target）**：
   - 请求处理的最终目的地，业务逻辑的实现。
5. **客户端（Client）（可选）**：
   - 发出请求的 Web 浏览器或 API 客户端。

拦截过滤器模式通过在请求处理流程中插入通用操作，有助于保持业务逻辑的清晰和专注。

## 实现

我们将创建 _FilterChain_、_FilterManager_、_Target_、_Client_ 作为表示实体的各种对象。_AuthenticationFilter_ 和 _DebugFilter_ 表示实体过滤器。

_InterceptingFilterDemo_ 类使用 _Client_ 来演示拦截过滤器设计模式。

![拦截过滤器模式的 UML 图](https://www.runoob.com/wp-content/uploads/2014/08/20201015-intercepting.svg)

**步骤 1**

创建过滤器接口 Filter。

```java
Filter.java
public interface Filter {
   public void execute(String request);
}
```

**步骤 2**

创建实体过滤器。

```java
AuthenticationFilter.java
public class AuthenticationFilter implements Filter {
   public void execute(String request){
      System.out.println("Authenticating request: " + request);
   }
}

DebugFilter.java
public class DebugFilter implements Filter {
   public void execute(String request){
      System.out.println("request log: " + request);
   }
}
```

**步骤 3**

创建 Target。

```
Target.java
public class Target {
   public void execute(String request){
      System.out.println("Executing request: " + request);
   }
}
```

**步骤 4**

创建过滤器链。

```java
FilterChain.java
import java.util.ArrayList;
import java.util.List;

public class FilterChain {
   private List<Filter> filters = new ArrayList<Filter>();
   private Target target;

   public void addFilter(Filter filter){
      filters.add(filter);
   }

   public void execute(String request){
      for (Filter filter : filters) {
         filter.execute(request);
      }
      target.execute(request);
   }

   public void setTarget(Target target){
      this.target = target;
   }
}
```

**步骤 5**

创建过滤管理器。

```java
FilterManager.java
public class FilterManager {
   FilterChain filterChain;

   public FilterManager(Target target){
      filterChain = new FilterChain();
      filterChain.setTarget(target);
   }
   public void setFilter(Filter filter){
      filterChain.addFilter(filter);
   }

   public void filterRequest(String request){
      filterChain.execute(request);
   }
}
```

**步骤 6**

创建客户端 Client。

```java
Client.java
public class Client {
   FilterManager filterManager;

   public void setFilterManager(FilterManager filterManager){
      this.filterManager = filterManager;
   }

   public void sendRequest(String request){
      filterManager.filterRequest(request);
   }
}
```

**步骤 7**

使用 _Client_ 来演示拦截过滤器设计模式。

```java
InterceptingFilterDemo.java
public class InterceptingFilterDemo {
   public static void main(String[] args) {
      FilterManager filterManager = new FilterManager(new Target());
      filterManager.setFilter(new AuthenticationFilter());
      filterManager.setFilter(new DebugFilter());

      Client client = new Client();
      client.setFilterManager(filterManager);
      client.sendRequest("HOME");
   }
}
```
