---
layout: post
title: "SpringBoot"
date: 2024-6-12
tags: [SSM]
comments: true
author: zxy
---

> 项目地址：(前后端分离项目)
> 后端：https://gitee.com/zou-xinyu6/ssm-study/tree/master/big-event-back
> 前端：https://gitee.com/zou-xinyu6/ssm-study/tree/master/big-event-front

### SpringBoot 简介

`SpringBoot` 是由 `Pivotal` 团队提供的全新框架，其设计目的是用来简化 `Spring` 应用的初始搭建以及开发过程。

> 对比 Spring 和 SpringBoot

![在这里插入图片描述](https://img-blog.csdnimg.cn/006147d25f53456685e51e3cdd4dccb8.png)

注意：基于 Idea 的 `Spring Initializr` 快速构建 `SpringBoot` 工程时需要联网。

> 基于 SpringBoot 如何进行前后端分离开发

后端可以将 `SpringBoot` 工程打成 `jar` 包，该 `jar` 包运行不依赖于 `Tomcat` 和 `Idea` 这些工具也可以正常运行，只是这个 `jar` 包在运行过程中连接和我们自己程序相同的 `Mysql` 数据库即可。这样就可以解决这个问题，如下图

![在这里插入图片描述](https://img-blog.csdnimg.cn/0cc88a28025e49208223598dc3a93433.png)

```
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
基于此插件即可生成对应jar包
```

只需要使用 `Maven` 的 `package` 指令打包就会在 `target` 目录下生成对应的 `Jar` 包。

找到 jar 包位置，使用 cmd 打开，然后运行此命令`jar -jar springboot_01_quickstart-0.0.1-SNAPSHOT.jar`即可运行 SpringBoot 后端程序

#### SpringBoot 概述

原始 `Spring` 环境搭建和开发存在以下问题：

- 配置繁琐
- 依赖设置繁琐

`SpringBoot` 程序优点恰巧就是针对 `Spring` 的缺点

- 自动配置。这个是用来解决 `Spring` 程序配置繁琐的问题
- 起步依赖。这个是用来解决 `Spring` 程序依赖设置繁琐的问题
- 辅助功能（内置服务器,…）。我们在启动 `SpringBoot` 程序时既没有使用本地的 `tomcat` 也没有使用 `tomcat` 插件，而是使用 `SpringBoot` 内置的服务器。

##### 起步依赖

我们使用 `Spring Initializr` 方式创建的 `Maven` 工程的的 `pom.xml` 配置文件中自动生成了很多包含 `starter` 的依赖，如下图

![在这里插入图片描述](https://img-blog.csdnimg.cn/7c0173ab8fe043d5840ad9f7ce09bdd8.png)

这些依赖就是启动依赖。

**starter**

- `SpringBoot` 中常见项目名称，定义了当前项目使用的所有项目坐标，以达到减少依赖配置的目的

**parent**

- 所有 `SpringBoot` 项目要继承的项目，定义了若干个坐标版本号（依赖管理，而非依赖），以达到减少依赖冲突的目的
- `spring-boot-starter-parent`（2.5.0）与 `spring-boot-starter-parent`（2.4.6）共计 57 处坐标版本不同

**实际开发**

- 使用任意坐标时，仅书写 GAV 中的 G 和 A，V 由 SpringBoot 提供

  > G：groupid
  >
  > A：artifactId
  >
  > V：version

- 如发生坐标错误，再指定 version（要小心版本冲突）

##### 程序启动

创建的每一个 `SpringBoot` 程序时都包含一个类似于下面的类，我们将这个类称作引导类

```java
@SpringBootApplication
public class Springboot01QuickstartApplication {

    public static void main(String[] args) {
        SpringApplication.run(Springboot01QuickstartApplication.class, args);
    }
}
```

注意：

- `SpringBoot` 在创建项目时，采用 jar 的打包方式

- `SpringBoot` 的引导类是项目的入口，运行 `main` 方法就可以启动项目

  因为我们在 `pom.xml` 中配置了 `spring-boot-starter-web` 依赖，而该依赖通过前面的学习知道它依赖 `tomcat` ，所以运行 `main` 方法就可以使用 `tomcat` 启动咱们的工程。

##### 切换 web 服务器

现在我们启动工程使用的是 `tomcat` 服务器，那能不能不使用 `tomcat` 而使用 `jetty` 服务器，`jetty` 在我们 `maven` 高级时讲 `maven` 私服使用的服务器。而要切换 `web` 服务器就需要将默认的 `tomcat` 服务器给排除掉，怎么排除呢？使用 `exclusion` 标签

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <groupId>org.springframework.boot</groupId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

### 配置文件

#### 配置文件格式

我们现在启动服务器默认的端口号是 `8080`，访问路径可以书写为

```
http://localhost:8080/books/1
```

在线上环境我们还是希望将端口号改为 `80`，这样在访问的时候就可以不写端口号了，如下

```
http://localhost/books/1
```

而 `SpringBoot` 程序如何修改呢？`SpringBoot` 提供了多种属性配置方式

- `application.properties`

  ```
  server.port=80
  ```

- `application.yml`

  ```yaml
  server:
  	port: 81
  ```

- `application.yaml`

  ```yaml
  server:
  	port: 82
  ```

> 注意：`SpringBoot` 程序的配置文件名必须是 `application` ，只是后缀名不同而已。
>
> 三种配置文件的优先级是：application.properties`>`application.yml`>`application.yaml

#### yaml 格式

- 大小写敏感

- 属性层级关系使用多行描述，每行结尾使用冒号结束

- 使用缩进表示层级关系，同层级左侧对齐，只允许使用空格（不允许使用 Tab 键）

  空格的个数并不重要，只要保证同层级的左侧对齐即可。

- 属性值前面添加空格（属性名与属性值之间使用冒号+空格作为分隔）

- \# 表示注释

核心规则：数据前面要加空格与冒号隔开

数组数据在数据书写位置的下方使用减号作为数据开始符号，每行书写一个数据，减号与数据间空格分隔，例如

```yaml
enterprise:
  name: itcast
  age: 16
  tel: 4006184000
  subject:
    - Java
    - 前端
    - 大数据
```

> 程序中如何读取 yml 配置文件中的数据

`SpringBoot` 提供了将配置文件中的数据封装到我们自定义的实体类对象中的方式。具体操作如下：

- 将实体类 `bean` 的创建交给 `Spring` 管理。在类上添加 `@Component` 注解
- 使用 `@ConfigurationProperties` 注解表示加载配置文件。在该注解中也可以使用 `prefix` 属性指定只加载指定前缀的数据
- 在 `BookController` 中进行注入
- 加上依赖

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

**具体代码如下：**

`Enterprise` 实体类内容如下：

```java
@Component
@ConfigurationProperties(prefix = "enterprise")
public class Enterprise {
    private String name;
    private int age;
    private String tel;
    private String[] subject;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getTel() {
        return tel;
    }

    public void setTel(String tel) {
        this.tel = tel;
    }

    public String[] getSubject() {
        return subject;
    }

    public void setSubject(String[] subject) {
        this.subject = subject;
    }

    @Override
    public String toString() {
        return "Enterprise{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", tel='" + tel + '\'' +
                ", subject=" + Arrays.toString(subject) +
                '}';
    }
}
```

`BookController` 内容如下：

```java
@RestController
@RequestMapping("/books")
public class BookController {

    @Autowired
    private Enterprise enterprise;

    @GetMapping("/{id}")
    public String getById(@PathVariable Integer id){
        System.out.println(enterprise.getName());
        System.out.println(enterprise.getAge());
        System.out.println(enterprise.getSubject());
        System.out.println(enterprise.getTel());
        System.out.println(enterprise.getSubject()[0]);
        return "hello , spring boot!";
    }
}
```

#### 多环境配置

以后在工作中，对于开发环境、测试环境、生产环境的配置肯定都不相同，比如我们开发阶段会在自己的电脑上安装 `mysql` ，连接自己电脑上的 `mysql` 即可，但是项目开发完毕后要上线就需要该配置，将环境的配置改为线上环境的。

![在这里插入图片描述](https://img-blog.csdnimg.cn/b54817e1479d442ba6acc19517177b0b.png)

来回的修改配置会很麻烦，而 `SpringBoot` 给开发者提供了多环境的快捷配置，需要切换环境时只需要改一个配置即可。不同类型的配置文件多环境开发的配置都不相同，接下来对不同类型的配置文件进行说明

##### yaml 文件

在 `application.yml` 中使用 `---` 来分割不同的配置，内容如下

```java
#设置启用的环境
spring:
  profiles:
    active: dev

---
#开发
spring:
  profiles: dev
server:
  port: 80
---
#生产
spring:
  profiles: pro
server:
  port: 81
---
#测试
spring:
  profiles: test
server:
  port: 82
---
```

##### properties 文件

`properties` 类型的配置文件配置多环境需要定义不同的配置文件

- `application-dev.properties` 是开发环境的配置文件。我们在该文件中配置端口号为 `80`

  ```properties
  server.port=80
  ```

- `application-test.properties` 是测试环境的配置文件。我们在该文件中配置端口号为 `81`

  ```properties
  server.port=81
  ```

- `application-pro.properties` 是生产环境的配置文件。我们在该文件中配置端口号为 `82`

  ```properties
  server.port=82
  ```

`SpringBoot` 只会默认加载名为 `application.properties` 的配置文件，所以需要在 `application.properties` 配置文件中设置启用哪个配置文件，配置如下:

```properties
spring.profiles.active=pro
```

##### 命令行启动参数设置

使用 `SpringBoot` 开发的程序以后都是打成 `jar` 包，通过 `java -jar xxx.jar` 的方式启动服务的。那么就存在一个问题，如何切换环境呢？因为配置文件打到的 jar 包中了。

我们知道 `jar` 包其实就是一个压缩包，可以解压缩，然后修改配置，最后再打成 jar 包就可以了。这种方式显然有点麻烦，而 `SpringBoot` 提供了在运行 `jar` 时设置开启指定的环境的方式，如下

```shell
java –jar xxx.jar –-spring.profiles.active=test
```

那么这种方式能不能临时修改端口号呢？也是可以的，可以通过如下方式

```shell
java –jar xxx.jar –-server.port=88
```

当然也可以同时设置多个配置，比如即指定启用哪个环境配置，又临时指定端口，如下

```shell
java –jar springboot.jar –-server.port=88 –-spring.profiles.active=test
```

大家进行测试后就会发现命令行设置的端口号优先级高（也就是使用的是命令行设置的端口号）

> maven 和 springboot 多环境兼容问题

如果我在 maven 中设置了多环境

```xml
<profiles>
    <profile>
    	<id>dev_env</id>
    	<properties>
    		<profile.active>dev</profile.active>
    	</properties>
    	<acvtivation>
    		<activaByDefault>true</activaByDefault>
    	</activation>
    </profile>
    <profile>
    	<id>pro_env</id>
    	<properties>
    		<profile.active>pro</profile.active>
    	</properties>
    </profile>
    <profile>
    	<id>test_env</id>
    	<properties>
    		<profile.active>test</profile.active>
    	</properties>
    </profile>
</profiles>
```

springboot 的配置文件中也配置多环境

```yaml
spring:
  profile:
    active: ${profile.active}
---
spring:
  profiles: pro
server:
  port: 80
---
spring:
  profiles: dev
server:
  port: 81
---
spring:
  profiles: dev
server:
  port: 81
```

这样子配置${profile.active}就会解析 maven 文件中的设置，不过需要多个插件

```
<build>
	<plugins>
		<plugin>
			<artifactId>maven-resources-plugin</artifactId>
			<configuration>
				<encoding>utf-8</encoding>
				<useDefaultDelimiters>true</useDefaultDelimiters>
			</configuration>
		</plugin>
	</plugins>
</build>
```

#### 配置文件分类

有这样的场景，我们开发完毕后需要测试人员进行测试，由于测试环境和开发环境的很多配置都不相同，所以测试人员在运行我们的工程时需要临时修改很多配置，如下

```shell
java –jar springboot.jar –-spring.profiles.active=test --server.port=85 --server.servlet.context-path=/heima --server.tomcat.connection-timeout=-1 …… …… …… …… ……
```

针对这种情况，`SpringBoot` 定义了配置文件不同的放置的位置；而放在不同位置的优先级时不同的。

`SpringBoot` 中 4 级配置文件放置位置：

- 1 级：classpath：application.yml
- 2 级：classpath：config/application.yml
- 3 级：file ：application.yml
- 4 级：file ：config/application.yml

> 说明： 级别越高优先级越高 (4 > 3 > 2 > 1)

注意：

SpringBoot 2.5.0 版本存在一个 bug，我们在使用这个版本时，需要在 `jar` 所在位置的 `config` 目录下创建一个任意名称的文件夹（4 级）

### SpringBoot 整合 junit

在 `com.itheima.service` 下创建 `BookService` 接口，内容如下

```java
public interface BookService {
    public void save();
}
```

在 `com.itheima.service.impl` 包写创建一个 `BookServiceImpl` 类，使其实现 `BookService` 接口，内容如下

```java
@Service
public class BookServiceImpl implements BookService {
    @Override
    public void save() {
        System.out.println("book service is running ...");
    }
}
```

#### 3.2 编写测试类

在 `test/java` 下创建 `com.itheima` 包，在该包下创建测试类，将 `BookService` 注入到该测试类中

```java
@SpringBootTest
class Springboot07TestApplicationTests {

    @Autowired
    private BookService bookService;

    @Test
    public void save() {
        bookService.save();
    }
}
```

> ==注意：==这里的引导类所在包必须是测试类所在包及其子包。
>
> 例如：
>
> - 引导类所在包是 `com.itheima`
> - 测试类所在包是 `com.itheima`
>
> 如果不满足这个要求的话，就需要在使用 `@SpringBootTest` 注解时，使用 `classes` 属性指定引导类的字节码对象。如 `@SpringBootTest(classes = Springboot07TestApplication.class)`

#### SpringBoot 整合 mybatis

- 创建新模块，选择 `Spring Initializr`，并配置模块相关基础信息
- 选择当前模块需要使用的技术集（MyBatis、MySQL）

```java
在 com.itheima.domain 包下定义实体类 Book，内容如下
public class Book {
    private Integer id;
    private String name;
    private String type;
    private String description;

    //setter and  getter

    //toString
}

在 com.itheima.dao 包下定义 BookDao 接口，内容如下
@Mapper
public interface BookDao {
    @Select("select * from tbl_book where id = #{id}")
    public Book getById(Integer id);
}

在 test/java 下定义包 com.itheima ，在该包下测试类，内容如下
@SpringBootTest
class Springboot08MybatisApplicationTests {

	@Autowired
	private BookDao bookDao;

	@Test
	void testGetById() {
		Book book = bookDao.getById(1);
		System.out.println(book);
	}
}

编写配置，在 application.yml 配置文件中配置如下内容
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/ssm_db
    username: root
    password: root
```

> 注意：`SpringBoot` 版本低于 2.4.3(不含)，Mysql 驱动版本大于 8.0 时，需要在 url 连接串中配置时区 `jdbc:mysql://localhost:3306/ssm_db?serverTimezone=UTC`，或在 MySQL 数据库端配置时区解决此问题

现在我们并没有指定数据源，`SpringBoot` 有默认的数据源，我们也可以指定使用 `Druid` 数据源，按照以下步骤实现

- 导入 `Druid` 依赖

  ```xml
  <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid</artifactId>
      <version>1.1.16</version>
  </dependency>
  ```

- 在 `application.yml` 配置文件配置

  可以通过 `spring.datasource.type` 来配置使用什么数据源。配置文件内容可以改进为

  ```yaml
  spring:
    datasource:
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://localhost:3306/ssm_db?serverTimezone=UTC
      username: root
      password: root
      type: com.alibaba.druid.pool.DruidDataSource
  ```
