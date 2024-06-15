---
layout: post
title: "SpringMVC"
date: 2024-6-12
tags: [SSM]
comments: true
author: zxy
---

## SpringMVC 简介

SpringMVC 是用来替换 Servlet 的，SpringMVC 开发更加高效，简单

SpringMVC 和 Servlet 都是用于表现层开发的

#### SpringMVC 概述

![在这里插入图片描述](https://img-blog.csdnimg.cn/3014e49fc77945ad9f347d703e3e66fe.png)

当前 WEB 程序的工作流程:

三层架构

- web 程序通过浏览器访问前端页面，发送异步请求到后端服务器
- 后台服务器采用三层架构进行功能开发
  - 表现层负责接收请求和数据然后将数据转交给业务层
  - 业务层负责调用数据层完成数据库表的增删改查，并将结果返给表现层
  - 表现层将数据转换成 json 格式返回给前端
- 前端页面将数据进行解析最终展示给用户。

表现层与数据层的技术选型:

- 数据层采用 Mybatis 框架
- 变现层采用 SpringMVC 框架，SpringMVC 主要负责的内容有:

  - controller 如何接收请求和数据
  - 如何将请求和数据转发给业务层
  - 如何将响应数据转换成 json 发回到前端

- SpringMVC 是一种基于 Java 实现 MVC 模型的轻量级 Web 框架

### SpringMVC 入门案例

> SpringMVC 程序的工作流程

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-bVv9BVgV-1673230130572)(assets/1651596361174.png)]](https://img-blog.csdnimg.cn/554e20b54bd54cca81c1b05f54d4d4bd.png)

1. 浏览器发送请求到 Tomcat 服务器
2. Tomcat 服务器接收到请求后，会将请求交给 SpringMVC 中的 DispatcherServlet[前端控制器] 来处理请求
3. DispatcherServlet 不真正处理请求，只是按照对应的规则将请求分发到对应的 Bean 对象
4. Bean 对象是有我们自己编写来处理不同的请求，每个 Bean 中可以处理一个或多个不同的请求 url
5. DispatcherServlet 和 Bean 对象都需要交给 Spring 容器来进行管理

综上所述，需要我们编写的内容包含:

- Bean 对象的编写
- 请求 url 和 Bean 对象对应关系的配置
- 构建 Spring 容器，将 DispatcherServlet 和 Bean 对象交给容器管理
- 配置 Tomcat 服务器，使其能够识别 Spring 容器，并将请求交给容器中的 DispatcherServlet 来分发请求

具体的实现步骤如下:

1. 创建 web 工程(Maven 结构)并在工程的 pom.xml 添加 SpringMVC 和 Servlet 坐标
2. 创建 SpringMVC 控制器类(等同于 Servlet 功能)
3. 初始化 SpringMVC 环境(同 Spring 环境)，设定 SpringMVC 加载对应的 bean
4. 初始化 Servlet 容器，加载 SpringMVC 环境，并设置 SpringMVC 技术处理的请求

```java
步骤1：新建项目，加载依赖pom.xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.itheima</groupId>
  <artifactId>springmvc_01_quickstart</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>
  <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
   </properties>
   <!--1. 导入SpringMVC与servlet的坐标-->
  <dependencies>
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.2.10.RELEASE</version>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.tomcat.maven</groupId>
        <artifactId>tomcat7-maven-plugin</artifactId>
        <version>2.1</version>
        <configuration>
          <port>80</port>
          <path>/</path>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>

步骤2:创建控制器类
//2.制作控制器类，等同于Servlet
//2.1必须是一个spring管理的bean
//2.2定义具体处理请求的方法
//2.3设置当前方法的访问路径
//2.4设置响应结果为json数据
@Controller
public class UserController {

    @RequestMapping("/save")
    @ResponseBody
    public String save(){
        System.out.println("user save ...");
        return "{'module':'springmvc'}";
    }
}

步骤3:创建配置类
//3.定义配置类加载Controller对应的bean
@Configuration
@ComponentScan("com.itheima.controller")
public class SpringMvcConfig {
}

步骤4:创建Tomcat的Servlet容器配置类
//4.定义servlet容器的配置类
public class ServletContainersInitConfig extends AbstractDispatcherServletInitializer {
    //加载springMVC配置
    protected WebApplicationContext createServletApplicationContext() {
        //初始化WebApplicationContext对象
        AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
        //加载指定配置类
        ctx.register(SpringMvcConfig.class);
        return ctx;
    }

    //设置Tomcat接收的请求哪些归SpringMVC处理
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }

    //设置spring相关配置
    protected WebApplicationContext createRootApplicationContext() {
        return null;
    }
}
```

> 启动服务器初始化过程

1. 服务器启动，执行 ServletContainersInitConfig 类，初始化 web 容器

2. 执行 createServletApplicationContext 方法，创建了 WebApplicationContext 对象

   - 该方法加载 SpringMVC 的配置类 SpringMvcConfig 来初始化 SpringMVC 的容器

3. 加载 SpringMvcConfig 配置类

   ![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-0eaujtTh-1673230130576)(assets/1630433335744.png)]](https://img-blog.csdnimg.cn/d16dc13df99341f99cb135669e33e40c.png)

4. 执行@ComponentScan 加载对应的 bean

   - 扫描指定包下所有类上的注解，如 Controller 类上的@Controller 注解

5. 加载 UserController，每个@RequestMapping 的名称对应一个具体的方法

   ![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-C6A4kM5h-1673230130577)(assets/1630433398932.png)]](https://img-blog.csdnimg.cn/5188e87f3fe5463aa765a7fa4014e1cf.png)

   - 此时就建立了 `/save` 和 save 方法的对应关系

6. 执行 getServletMappings 方法，定义所有的请求都通过 SpringMVC

   ![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-ioWax4lt-1673230130578)(assets/1630433510528.png)]](https://img-blog.csdnimg.cn/13a49e2ccb2645d7ae8fff113fe6106f.png)

   - `/`代表所拦截请求的路径规则，只有被拦截后才能交给 SpringMVC 来处理请求

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-v5Sdc9EK-1673230130576)(assets/1630432494752.png)]](https://img-blog.csdnimg.cn/94e813b3fbdd48d8a35b68507aafa54f.png)

> 单次请求过程

1. 发送请求 localhost/save
2. web 容器发现所有请求都经过 SpringMVC，将请求交给 SpringMVC 处理
   - 因为符合上面第六步设置的请求路径，所以该请求会交给 SpringMVC 来处理
3. 解析请求路径/save
4. 由/save 匹配执行对应的方法 save(）
   - 上面的第五步已经将请求路径和方法建立了对应关系，通过/save 就能找到对应的 save 方法
5. 执行 save()
6. 检测到有@ResponseBody 直接将 save()方法的返回值作为响应求体返回给请求方

### bean 加载控制

在入门案例中我们创建过一个`SpringMvcConfig`的配置类，再回想前面咱们学习 Spring 的时候也创建过一个配置类`SpringConfig`。这两个配置类都需要加载资源。

controller、service 和 dao 这些类都需要被容器管理成 bean 对象，那么到底是该让 SpringMVC 加载还是让 Spring 加载呢?

- SpringMVC 加载其相关 bean(表现层 bean),也就是 controller 包下的类
- Spring 控制的 bean
  - 业务 bean(Service)
  - 功能 bean(DataSource,SqlSessionFactoryBean,MapperScannerConfigurer 等)

分析清楚谁该管哪些 bean 以后，接下来要解决的问题是如何让 Spring 和 SpringMVC 分开加载各自的内容。

> 因为功能不同，如何避免 Spring 错误加载到 SpringMVC 的 bean

- 方式一:Spring 加载的 bean 设定扫描范围为 com.itheima,排除掉 controller 包中的 bean

```java
@Configuration
@ComponentScan(value="com.itheima",
    excludeFilters=@ComponentScan.Filter(
    	type = FilterType.ANNOTATION,
        classes = Controller.class
    )
)
public class SpringConfig {
}

excludeFilters属性：设置扫描加载bean时，排除的过滤规则

type属性：设置排除规则，当前使用按照bean定义时的注解类型进行排除
	ANNOTATION：按照注解排除
	ASSIGNABLE_TYPE:按照指定的类型过滤
	ASPECTJ:按照Aspectj表达式排除，基本上不会用
	REGEX:按照正则表达式排除
	CUSTOM:按照自定义规则排除

classes属性：设置排除的具体注解类，当前设置排除@Controller定义的bean
```

- 方式二:Spring 加载的 bean 设定扫描范围为精准范围，例如 service 包、dao 包等

```java
@Configuration
@ComponentScan({"com.itheima.service","comitheima.dao"})
public class SpringConfig {
}
```

- 方式三:不区分 Spring 与 SpringMVC 的环境，加载到同一个环境中[了解即可]

> 有了 Spring 的配置类，要想在 tomcat 服务器启动将其加载，我们需要修改 ServletContainersInitConfig

```java
public class ServletContainersInitConfig extends AbstractDispatcherServletInitializer {
    protected WebApplicationContext createServletApplicationContext() {
        AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
        ctx.register(SpringMvcConfig.class);
        return ctx;
    }
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }
    protected WebApplicationContext createRootApplicationContext() {
      AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
        ctx.register(SpringConfig.class);
        return ctx;
    }
}
```

## 请求

> 设置请求映射路径

**问题分析：**

- UserController 有一个 save 方法，访问路径为`http://localhost/save`
- BookController 也有一个 save 方法，访问路径为`http://localhost/save`
- 当访问`http://localhost/saved`的时候，到底是访问 UserController 还是 BookController?

```java
解决方案：
@Controller
@RequestMapping("/user")
public class UserController {

    @RequestMapping("/save")
    @ResponseBody
    public String save(){
        System.out.println("user save ...");
        return "{'module':'user save'}";
    }

    @RequestMapping("/delete")
    @ResponseBody
    public String save(){
        System.out.println("user delete ...");
        return "{'module':'user delete'}";
    }
}

@Controller
@RequestMapping("/book")
public class BookController {

    @RequestMapping("/save")
    @ResponseBody
    public String save(){
        System.out.println("book save ...");
        return "{'module':'book save'}";
    }
}

```

> 请求参数

#### GET 类型参数

```java
http://localhost/commonParam?name=itcast&age=15

@Controller
public class UserController {

    @RequestMapping("/commonParam")
    @ResponseBody
    public String commonParam(String name,int age){
        System.out.println("普通参数传递 name ==> "+name);
        System.out.println("普通参数传递 age ==> "+age);
        return "{'module':'commonParam'}";
    }
}
直接用函数的参数接收参数即可
```

> GET 请求中文乱码

```java
http://localhost/commonParam?name=张三&age=18 这种请求发到后端，表现层拿到的参数打印到控制台会出现中文乱码
解决方法：配置tomcat UTF-8解码
<build>
    <plugins>
      <plugin>
        <groupId>org.apache.tomcat.maven</groupId>
        <artifactId>tomcat7-maven-plugin</artifactId>
        <version>2.1</version>
        <configuration>
          <port>80</port><!--tomcat端口号-->
          <path>/</path> <!--虚拟目录-->
          <uriEncoding>UTF-8</uriEncoding><!--访问路径编解码字符集-->
        </configuration>
      </plugin>
    </plugins>
  </build>

```

#### POST 发送参数

```
在请求体body里面写参数
后端接收还是一样，直接用函数的参数即可
```

> POST 请求中文乱码

```java
public class ServletContainersInitConfig extends AbstractAnnotationConfigDispatcherServletInitializer {
    protected Class<?>[] getRootConfigClasses() {
        return new Class[0];
    }

    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{SpringMvcConfig.class};
    }

    protected String[] getServletMappings() {
        return new String[]{"/"};
    }

    //乱码处理
    @Override
    protected Filter[] getServletFilters() {
        CharacterEncodingFilter filter = new CharacterEncodingFilter();
        filter.setEncoding("UTF-8");
        return new Filter[]{filter};
    }
}

```

> 返回给客户端数据中文乱码问题

```java
    @RequestMapping(value = "/AddUser",produces = "application/json;charset=utf-8")
    @ResponseBody
    public List<User> getUser(@RequestBody List<User> users){
        System.out.println(users);
        return users;
    }
```

#### 各种类型的参数传递

##### 普通参数

url 地址传参，地址参数名与形参变量名相同，定义形参即可接收参数。

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-p7ExDdqf-1673230130591)(assets/1630481585729.png)]](https://img-blog.csdnimg.cn/1d3a493a69c94e248422866b8a91f42d.png)

> 如果形参与地址参数名不一致该如何解决?

发送请求与参数:

```
http://localhost/commonParamDifferentName?name=张三&age=18
```

后台接收参数:

```java
@RequestMapping("/commonParamDifferentName")
    @ResponseBody
    public String commonParamDifferentName(@RequestPaam("name") String userName , int age){
        System.out.println("普通参数传递 userName ==> "+userName);
        System.out.println("普通参数传递 age ==> "+age);
        return "{'module':'common param different name'}";
    }
```

##### POJO 数据类型

简单数据类型一般处理的是参数个数比较少的请求，如果参数比较多，那么后台接收参数的时候就比较复杂，这个时候我们可以考虑使用 POJO 数据类型。

- POJO 参数：请求参数名与形参对象属性名相同，定义 POJO 类型形参即可接收参数

```java
POJO类
public class User {
    private String name;
    private int age;
    //setter...getter...略
}
```

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-rUgGATCr-1673230130592)(assets/1630482186745.png)]](https://img-blog.csdnimg.cn/71a0703baf214d5992cbba76d262bcc2.png)

后台接收参数:

```java
//POJO参数：请求参数与形参对象中的属性对应即可完成参数传递
@RequestMapping("/pojoParam")
@ResponseBody
public String pojoParam(User user){
    System.out.println("pojo参数传递 user ==> "+user);
    return "{'module':'pojo param'}";
}
POJO参数接收，前端GET和POST发送请求数据的方式不变。
请求参数key的名称要和POJO中属性的名称一致，否则无法封装。
```

> 嵌套 POJO 类型参数怎么处理

```java
POJO类
public class Address {
    private String province;
    private String city;
    //setter...getter...略
}
public class User {
    private String name;
    private int age;
    private Address address;
    //setter...getter...略
}
```

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-yAeZ1Y4S-1673230130592)(assets/1630482363291.png)]](https://img-blog.csdnimg.cn/6651bb5014f94425bd4022ee1ce0be2d.png)

```java
//POJO参数：请求参数与形参对象中的属性对应即可完成参数传递
@RequestMapping("/pojoParam")
@ResponseBody
public String pojoParam(User user){
    System.out.println("pojo参数传递 user ==> "+user);
    return "{'module':'pojo param'}";
}
```

##### 数组类型参数

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-hHGko8n5-1673230130592)(assets/1630482999626.png)]](https://img-blog.csdnimg.cn/738cb33c8567412ea6b99db9234be520.png)

后台接收参数:

```java
  //数组参数：同名请求参数可以直接映射到对应名称的形参数组对象中
    @RequestMapping("/arrayParam")
    @ResponseBody
    public String arrayParam(String[] likes){
        System.out.println("数组参数传递 likes ==> "+ Arrays.toString(likes));
        return "{'module':'array param'}";
    }
```

##### 集合类型参数

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-H7sCMls3-1673230130593)(assets/1630484283773.png)]](https://img-blog.csdnimg.cn/2d6f1cbb702640b382b752a789850613.png)

```java
//集合参数：同名请求参数可以使用@RequestParam注解映射到对应名称的集合对象中作为数据
@RequestMapping("/listParam")
@ResponseBody
public String listParam(@RequestParam List<String> likes){
    System.out.println("集合参数传递 likes ==> "+ likes);
    return "{'module':'list param'}";
}
```

#### JSON 数据传输参数

```java
步骤1:pom.xml添加依赖
SpringMVC默认使用的是jackson来处理json的转换，所以需要在pom.xml添加jackson依赖
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.0</version>
</dependency>

步骤2:前端发送JSON数据

步骤3:开启SpringMVC注解支持
@Configuration
@ComponentScan("com.itheima.controller")
//开启json数据类型自动转换
@EnableWebMvc
public class SpringMvcConfig {
}

步骤4:参数前添加@RequestBody
//使用@RequestBody注解将外部传递的json数组数据映射到形参的集合对象中作为数据
@RequestMapping("/listParamForJson")
@ResponseBody
public String listParamForJson(@RequestBody 参数类型与名称){
    System.out.println("list common(json)参数传递 list ==> "+likes);
    return "{'module':'list common for json param'}";
}

如果是json数组：List<String> likes
如果是json对象：User user
如果是 对象数组：List<User> list
```

> @RequestBody 与@RequestParam 区别

- @RequestParam 用于接收 url 地址传参，表单传参【application/x-www-form-urlencoded】
- @RequestBody 用于接收 json 数据【application/json】

#### 日期类型参数传递

请求 URL：`http://localhost/dataParam?date=2088/08/08&date1=2088-08-08&date2=2088/08/08 8:08:08`

```java
@RequestMapping("/dataParam")
@ResponseBody
public String dataParam(Date date,
                        @DateTimeFormat(pattern="yyyy-MM-dd") Date date1,
                        @DateTimeFormat(pattern="yyyy/MM/dd HH:mm:ss") Date date2)
    System.out.println("参数传递 date ==> "+date);
	System.out.println("参数传递 date1(yyyy-MM-dd) ==> "+date1);
	System.out.println("参数传递 date2(yyyy/MM/dd HH:mm:ss) ==> "+date2);
    return "{'module':'data param'}";
}
利用 @DateTimeFormat指定接收日期类型数据的格式
```

## 响应

后端处理完数据后，会向前端返回内容，具体包括：

- 响应页面
- 响应数据
  - 文本数据
  - json 数据（前后端分离项目都是这个）

> 响应页面

```java
@Controller
public class UserController {

    @RequestMapping("/toJumpPage")
    //注意
    //1.此处不能添加@ResponseBody,如果加了该注入，会直接将page.jsp当字符串返回前端
    //2.方法需要返回String
    public String toJumpPage(){
        System.out.println("跳转页面");
        return "page.jsp";  //page.jsp是一个jsp文件
    }

}
```

> 返回文本数据

```java
@Controller
public class UserController {

   	@RequestMapping("/toText")
	//注意此处该注解就不能省略，如果省略了,会把response text当前页面名称去查找，如果没有回报404错误
    @ResponseBody
    public String toText(){
        System.out.println("返回纯文本数据");
        return "response text";
    }

}
```

> 响应 JSON 数据

```java
响应POJO对象
@Controller
public class UserController {

    @RequestMapping("/toJsonPOJO")
    @ResponseBody
    public User toJsonPOJO(){   //返回值类型是对象
        System.out.println("返回json对象数据");
        User user = new User();
        user.setName("itcast");
        user.setAge(15);
        return user;
    }

}

响应POJO集合对象
@Controller
public class UserController {

    @RequestMapping("/toJsonList")
    @ResponseBody
    public List<User> toJsonList(){  //返回值类型是集合
        System.out.println("返回json集合数据");
        User user1 = new User();
        user1.setName("传智播客");
        user1.setAge(15);

        User user2 = new User();
        user2.setName("黑马程序员");
        user2.setAge(12);

        List<User> userList = new ArrayList<User>();
        userList.add(user1);
        userList.add(user2);

        return userList;
    }

}
```

## Rest 风格

REST（Representational State Transfer），表现形式状态转换,它是一种[软件架构](https://so.csdn.net/so/search?q=软件架构&spm=1001.2101.3001.7020)风格

REST 的优点有:

- 隐藏资源的访问行为，无法通过地址得知对资源是何种操作
- 书写简化

按照 REST 风格访问资源时使用行为动作区分对资源进行了何种操作

- `http://localhost/users` 查询全部用户信息 GET（查询）
- `http://localhost/users/1` 查询指定用户信息 GET（查询）
- `http://localhost/users` 添加用户信息 POST（新增/保存）
- `http://localhost/users` 修改用户信息 PUT（修改/更新）
- `http://localhost/users/1` 删除用户信息 DELETE（删除）

请求的方式比较多，但是比较常用的就 4 种，分别是`GET`,`POST`,`PUT`,`DELETE`。

根据 REST 风格对资源进行访问称为 RESTful。

### RESTful 入门案例

```java
@Controller
public class UserController {
	//设置当前请求方法为POST，表示REST风格中的添加操作
    @RequestMapping(value = "/users",method = RequestMethod.POST)
    @ResponseBody
    public String save() {
        System.out.println("user save...");
        return "{'module':'user save'}";
    }

    //设置当前请求方法为DELETE，表示REST风格中的删除操作
	@RequestMapping(value = "/users/{id}/{name}",method = RequestMethod.DELETE)
    @ResponseBody
    public String delete(@PathVariable Integer id,@PathVariable String name) {
        System.out.println("user delete..." + id+","+name);
        return "{'module':'user delete'}";
    }

    //设置当前请求方法为PUT，表示REST风格中的修改操作
    @RequestMapping(value = "/users",method = RequestMethod.PUT)
    @ResponseBody
    public String update(@RequestBody User user) {
        System.out.println("user update..." + user);
        return "{'module':'user update'}";
    }

    //设置当前请求方法为GET，表示REST风格中的查询操作
    @RequestMapping(value = "/users/{id}" ,method = RequestMethod.GET)
    @ResponseBody
    public String getById(@PathVariable Integer id){
        System.out.println("user getById..."+id);
        return "{'module':'user getById'}";
    }

    //设置当前请求方法为GET，表示REST风格中的查询操作
    @RequestMapping(value = "/users" ,method = RequestMethod.GET)
    @ResponseBody
    public String getAll() {
        System.out.println("user getAll...");
        return "{'module':'user getAll'}";
    }

}
```

> 三个注解`@RequestBody`、`@RequestParam`、`@PathVariable`,这三个注解之间的区别和应用分别是什么?

- 区别
  - @RequestParam 用于接收 url 地址传参或表单传参
  - @RequestBody 用于接收 json 数据
  - @PathVariable 用于接收路径参数，使用{参数名称}描述路径参数
- 应用
  - 后期开发中，发送请求参数超过 1 个时，以 json 格式为主，@RequestBody 应用较广
  - 如果发送非 json 格式数据，选用@RequestParam 接收请求参数
  - 采用 RESTful 进行开发，当参数数量较少时，例如 1 个，可以采用@PathVariable 接收请求路径变量，通常用于传递 id 值

#### RESTful 快速开发

```java
@RestController //@Controller + ReponseBody
@RequestMapping("/books")
public class BookController {

	//@RequestMapping(method = RequestMethod.POST)
    @PostMapping
    public String save(@RequestBody Book book){
        System.out.println("book save..." + book);
        return "{'module':'book save'}";
    }

    //@RequestMapping(value = "/{id}",method = RequestMethod.DELETE)
    @DeleteMapping("/{id}")
    public String delete(@PathVariable Integer id){
        System.out.println("book delete..." + id);
        return "{'module':'book delete'}";
    }

    //@RequestMapping(method = RequestMethod.PUT)
    @PutMapping
    public String update(@RequestBody Book book){
        System.out.println("book update..." + book);
        return "{'module':'book update'}";
    }

    //@RequestMapping(value = "/{id}",method = RequestMethod.GET)
    @GetMapping("/{id}")
    public String getById(@PathVariable Integer id){
        System.out.println("book getById..." + id);
        return "{'module':'book getById'}";
    }

    //@RequestMapping(method = RequestMethod.GET)
    @GetMapping
    public String getAll(){
        System.out.println("book getAll...");
        return "{'module':'book getAll'}";
    }

}
```

##### 知识点 1：@RestController

| 名称 | @RestController                                                                      |
| ---- | ------------------------------------------------------------------------------------ |
| 类型 | 类注解                                                                               |
| 位置 | 基于 SpringMVC 的 RESTful 开发控制器类定义上方                                       |
| 作用 | 设置当前控制器类为 RESTful 风格， 等同于@Controller 与@ResponseBody 两个注解组合功能 |

##### 知识点 2：@GetMapping @PostMapping @PutMapping @DeleteMapping

| 名称     | @GetMapping @PostMapping @PutMapping @DeleteMapping                                            |
| -------- | ---------------------------------------------------------------------------------------------- |
| 类型     | 方法注解                                                                                       |
| 位置     | 基于 SpringMVC 的 RESTful 开发控制器方法定义上方                                               |
| 作用     | 设置当前控制器方法请求访问路径与请求动作，每种对应一个请求动作， 例如@GetMapping 对应 GET 请求 |
| 相关属性 | value（默认）：请求访问路径                                                                    |

## SSM 整合（Spring + SpringMVC + MyBatis）

> 项目位置：https://gitee.com/zou-xinyu6/ssm-study/tree/master/SSM

### 统一结果封装

> 后端需要将返回的数据封装成统一的格式，保证前端都能进行处理

```java
public class Result{
	private Object data;   //数据
	private Integer code;  //提示码
	private String msg;    //提示消息
}
```

### 统一异常处理

项目的任何位置都有可能出现异常：

- 框架内部抛出的异常：因使用不合规导致
- 数据层抛出的异常：因外部服务器故障导致（例如：服务器访问超时）
- 业务层抛出的异常：因业务逻辑书写错误导致（例如：遍历业务书写操作，导致索引异常等）
- 表现层抛出的异常：因数据收集、校验等规则导致（例如：不匹配的数据类型间导致异常）
- 工具类抛出的异常：因工具类书写不严谨不够健壮导致（例如：必要释放的连接长期未释放等）

这么多的异常如果分布在不同层次进行处理，那么异常处理的代码会非常乱，所以我们统一将异常处理放在表现层（反正最后都由表现层与前端通信）

> 将异常进行分类，定义异常处理器，分别处理不同类型的异常

```java
@RestControllerAdvice
public class ProjectExceptionAdvice {    //异常处理器
    @ExceptionHandler(SystemException.class)
    public Result doSystemException(SystemException e) {
        //记录日志
        //发送消息给运维
        //发送邮件给开发人员,将异常对象发给开发人员
        //返回消息给前端用户，保证响应
        return new Result(e.getCode(), null, e.getMessage());
    }

    @ExceptionHandler(BusinessException.class)
    public Result doBusinessException(BusinessException e) {
        //返回消息给前端用户，保证响应
        return new Result(e.getCode(), null, e.getMessage());
    }

    @ExceptionHandler(Exception.class)
    public Result doException(Exception e) {
        //记录日志
        //发送消息给运维
        //发送邮件给开发人员,将异常对象发给开发人员
        //返回消息给前端用户，保证响应
        return new Result(Code.SYSTEM_UNKNOWN_ERROR, null, "系统繁忙，请稍后再试");  //这里的消息就只是为了安抚用户了
    }
}
```

## 拦截器

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-TBhV3bDq-1673233932232)(assets/1630676280170.png)]](https://img-blog.csdnimg.cn/caf6f8f0ccd14929844d6a4ac5842df1.png)

- 拦截器（Interceptor）是一种动态拦截方法调用的机制，在 SpringMVC 中动态拦截控制器方法的执行
- 作用:
  - 在指定的方法调用前后执行预先设定的代码
  - 阻止原始方法的执行

> 拦截器和过滤器之间的区别是什么?

- 归属不同：Filter 属于 Servlet 技术，Interceptor 属于 SpringMVC 技术
- 拦截内容不同：Filter 对所有访问进行增强，Interceptor 仅针对 SpringMVC 的访问进行增强

![img](https://img-blog.csdnimg.cn/ac6b3d20b57a438ea28bfa70db768ac2.png)

> 拦截器链

![img](https://img-blog.csdnimg.cn/fd1251523f5a460faef8a2e494b214af.png)
