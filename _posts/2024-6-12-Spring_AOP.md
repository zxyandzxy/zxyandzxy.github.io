---
layout: post
title: "Spring_AOP"
date: 2024-6-12
tags: [SSM]
comments: true
author: zxy
---

## AOP 简介

> 什么是 AOP

- AOP(Aspect Oriented Programming)面向切面编程，一种编程范式，指导开发者如何组织程序结构。
  - OOP(Object Oriented Programming)面向对象编程

OOP 是一种编程思想，AOP 也是一种编程思想，编程思想主要的内容就是指导程序员该如何编写程序，所以它们两个是不同的`编程范式`。

> AOP 的作用

在不惊动原始设计的基础上为其进行功能增强，代理模式这种设计模式也可以实现

> Spring 如何实现的 AOP

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-EEFkefxU-1668042845663)(assets/1630144353462.png)]](https://img-blog.csdnimg.cn/1c043337dff841b98b669ee47ad0600e.png)

- 连接点(JoinPoint)：程序执行过程中的任意位置，粒度为执行方法、抛出异常、设置变量等
  - 在 SpringAOP 中，理解为方法的执行
- 切入点(Pointcut):匹配连接点的式子
  - 在 SpringAOP 中，一个切入点可以描述一个具体方法，也可也匹配多个方法
    - 一个具体的方法:如 com.itheima.dao 包下的 BookDao 接口中的无形参无返回值的 save 方法
    - 匹配多个方法:所有的 save 方法，所有的 get 开头的方法，所有以 Dao 结尾的接口中的任意方法，所有带有一个参数的方法
  - 连接点范围要比切入点范围大，是切入点的方法也一定是连接点，但是是连接点的方法就不一定要被增强，所以可能不是切入点。
- 通知(Advice):在切入点处执行的操作，也就是共性功能
  - 在 SpringAOP 中，功能最终以方法的形式呈现
- 通知类：定义通知的类
- 切面(Aspect):描述通知与切入点的对应关系。

## AOP 入门案例

我们首先明白一个点，AOP 一定不会修改原来的业务逻辑代码，所以 AOP 一定是加代码。

AOP 实现步骤如下：

> 1.导入坐标(pom.xml)
>
> 2.制作连接点(原始操作，Dao 接口与实现类)
>
> 3.制作共性功能(通知类与通知)
>
> 4.定义切入点
>
> 5.绑定切入点与通知关系(切面)

```java
步骤1:添加依赖
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.4</version>
</dependency>
<dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.2.10.RELEASE</version>
</dependency>

//因为spring-context中已经导入了spring-aop,所以不需要再单独导入spring-aop
//导入AspectJ的jar包,AspectJ是AOP思想的一个具体实现，Spring有自己的AOP实现，但是相比于AspectJ来说比较麻烦，所以我们直接采用Spring整合ApsectJ的方式进行AOP开发。

步骤2:定义接口与实现类
//添加BookDao和BookDaoImpl类
public interface BookDao {
    public void save();
    public void update();
}

@Repository
public class BookDaoImpl implements BookDao {

    public void save() {
        System.out.println(System.currentTimeMillis());
        System.out.println("book dao save ...");
    }

    public void update(){
        System.out.println("book dao update ...");
    }
}

步骤3:定义通知类和通知(类名和方法名没有要求，可以任意。)
//通知就是将共性功能抽取出来后形成的方法，共性功能指的就是当前系统时间的打印。
public class MyAdvice {
    public void method(){
        System.out.println(System.currentTimeMillis());
    }
}

步骤4:定义切入点,绑定切面
//BookDaoImpl中有两个方法，分别是save和update，我们要增强的是update方法
@Component  //标记为Spring的一个Bean
@Aspect    //设置此类为通知类
public class MyAdvice {
    @Pointcut("execution(void com.itheima.dao.BookDao.update())")
    private void pt(){}
    @Before("pt()") //绑定切入点与通知关系，并指定通知添加到原始连接点的具体执行位置,@Before翻译过来是之前，也就是说通知会在切入点方法执行之前执行
    public void method(){
        System.out.println(System.currentTimeMillis());
    }
}

步骤5:开启注解格式AOP功能
@Configuration
@ComponentScan("com.itheima")
@EnableAspectJAutoProxy  //就这一行，通知Spring要使用AOP
public class SpringConfig {
}

步骤6:运行程序
public class App {
    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
        BookDao bookDao = ctx.getBean(BookDao.class);
        bookDao.update();
    }
}

```

## AOP 工作流程

##### 流程 1:Spring 容器启动

- 容器启动就需要去加载 bean,哪些类需要被加载呢?
- 需要被增强的类，如:BookServiceImpl
- 通知类，如:MyAdvice
- 注意此时 bean 对象还没有创建成功

##### 流程 2:读取所有切面配置中的切入点

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-uUAY4In5-1668042845678)(assets/1630151682428.png)]](https://img-blog.csdnimg.cn/d39d2563331b4b32a4687afcea21c275.png)

- 上面这个例子中有两个切入点的配置，但是第一个`ptx()`并没有被使用，所以不会被读取。

##### 流程 3:初始化 bean，

判定 bean 对应的类中的方法是否匹配到任意切入点

- 注意第 1 步在容器启动的时候，bean 对象还没有被创建成功。

- 要被实例化 bean 对象的类中的方法和切入点进行匹配

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/2825d533d5ba42ff97f88d7ee5b72400.png)

  - 匹配失败，创建原始对象,如 UserDao
    - 匹配失败说明不需要增强，直接调用原始对象的方法即可。
  - 匹配成功，创建原始对象（目标对象）的代理对象,如:BookDao
    - 匹配成功说明需要对其进行增强
    - 对哪个类做增强，这个类对应的对象就叫做目标对象
    - 因为要对目标对象进行功能增强，而采用的技术是动态代理，所以会为其创建一个代理对象
    - 最终运行的是代理对象的方法，在该方法中会对原始方法进行功能增强

##### 流程 4:获取 bean 执行方法

- 获取的 bean 是原始对象时，调用方法并执行，完成操作
- 获取的 bean 是代理对象时，根据代理对象的运行模式运行原始方法与增强的内容，完成操作

## AOP 配置管理

> AOP 切入点表达式

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-CwYqHq6N-1668042845680)(assets/1630155937718.png)]](https://img-blog.csdnimg.cn/8de2786eddf6496f8bb5c88e8f8ff970.png)

##### 语法格式

- 切入点:要进行增强的方法
- 切入点表达式:要进行增强的方法的描述方式

描述方式一：执行 com.itheima.dao 包下的 BookDao 接口中的无参数 update 方法

```java
execution(void com.itheima.dao.BookDao.update())
```

描述方式二：执行 com.itheima.dao.impl 包下的 BookDaoImpl 类中的无参数 update 方法

```
execution(void com.itheima.dao.impl.BookDaoImpl.update())
```

因为调用接口方法的时候最终运行的还是其实现类的方法，所以上面两种描述方式都是可以的。

> 切入点表达式标准格式：动作关键字(访问修饰符 返回值 包名.类/接口名.方法名(参数) 异常名）

```java
execution(public User com.itheima.service.UserService.findById(int))
```

- execution：动作关键字，描述切入点的行为动作，例如 execution 表示执行到指定切入点
- public:访问修饰符,还可以是 public，private 等，可以省略
- User：返回值，写返回值类型
- com.itheima.service：包名，多级包使用点连接
- UserService:类/接口名称
- findById：方法名
- int:参数，直接写参数的类型，多个类型用逗号隔开
- 异常名：方法定义中抛出指定异常，可以省略

##### 通配符

使用通配符描述切入点，主要的目的就是简化之前的配置，防止对于每一个要增强的方法都写一次切入点表达式

- `*`:单个独立的任意符号，可以独立出现，也可以作为前缀或者后缀的匹配符出现

  ```java
  execution（public * com.itheima.*.UserService.find*(*))
  ```

  匹配 com.itheima 包下的任意包中的 UserService 类或接口中所有 find 开头的带有一个参数的方法

- `..`：多个连续的任意符号，可以独立出现，常用于简化包名与参数的书写

  ```java
  execution（public User com..UserService.findById(..))
  ```

  匹配 com 包下的任意包中的 UserService 类或接口中所有名称为 findById 的方法

- `+`：专用于匹配子类类型

  ```java
  execution(* *..*Service+.*(..))
  ```

  这个使用率较低，描述子类的，咱们做 JavaEE 开发，继承机会就一次，使用都很慎重，所以很少用它。\*Service+，表示所有以 Service 结尾的接口的子类。

##### 书写技巧

- 所有代码按照标准规范开发，否则以下技巧全部失效
- 描述切入点通**常描述接口**，而不描述实现类,如果描述到实现类，就出现紧耦合了
- 访问控制修饰符针对接口开发均采用 public 描述（**可省略访问控制修饰符描述**）
- 返回值类型对于增删改类使用精准类型加速匹配，对于查询类使用\*通配快速描述
- **包名\*\*书写\*\*尽量不使用…匹配**，效率过低，常用\*做单个包描述匹配，或精准匹配
- **接口名/类名\*\*书写名称与模块相关的\*\*采用\*匹配**，例如 UserService 书写成\*Service，绑定业务层接口名
- **方法名\*\*书写以\*\*动词\*\*进行\*\*精准匹配**，名词采用*匹配，例如 getById 书写成 getBy*,selectAll 书写成 selectAll
- 参数规则较为复杂，根据业务方法灵活调整
- 通常**不使用异常**作为**匹配**规则

> AOP 通知类型

##### 类型介绍

- 前置通知
- 后置通知
- **环绕通知(重点)**
- 返回后通知(了解)
- 抛出异常后通知(了解)

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-UTpEWXol-1668042845688)(assets/1630170090945.png)]](https://img-blog.csdnimg.cn/76602318f77c4741a36fb402619d7d86.png)

(1)前置通知，追加功能到方法执行前,类似于在代码 1 或者代码 2 添加内容

(2)后置通知,追加功能到方法执行后,不管方法执行的过程中有没有抛出异常都会执行，类似于在代码 5 添加内容

(3)返回后通知,追加功能到方法执行后，只有方法正常执行结束后才进行,类似于在代码 3 添加内容，如果方法执行抛出异常，返回后通知将不会被添加

(4)抛出异常后通知,追加功能到方法抛出异常后，只有方法执行出异常才进行,类似于在代码 4 添加内容，只有方法抛出异常后才会被添加

(5)环绕通知,环绕通知功能比较强大，它可以追加功能到方法执行的前后，这也是比较常用的方式，它可以实现其他四种通知类型的功能。

**环绕通知注意事项**

1. 环绕通知必须依赖形参 ProceedingJoinPoint 才能实现对原始方法的调用，进而实现原始方法调用前后同时添加通知
2. 通知中如果未使用 ProceedingJoinPoint 对原始方法进行调用将跳过原始方法的执行
3. 对原始方法的调用可以不接收返回值，通知方法设置成 void 即可，如果接收返回值，最好设定为 Object 类型
4. 原始方法的返回值如果是 void 类型，通知方法的返回值类型可以设置成 void,也可以设置成 Object
5. 由于无法预知原始方法运行后是否会抛出异常，因此环绕通知方法必须要处理 Throwable 异常

> AOP 环绕方法补充

```java
@Component
@Aspect
public class ProjectAdvice {
    //配置业务层的所有方法
    @Pointcut("execution(* com.itheima.service.*Service.*(..))")
    private void servicePt(){}
    //@Around("ProjectAdvice.servicePt()") 可以简写为下面的方式
    @Around("servicePt()")
    public void runSpeed(ProceedingJoinPoint pjp){
        //获取执行签名信息
        Signature signature = pjp.getSignature();
        //通过签名获取执行操作名称(接口名)
        String className = signature.getDeclaringTypeName();
        //通过签名获取执行操作名称(方法名)
        String methodName = signature.getName();

        long start = System.currentTimeMillis();
        for (int i = 0; i < 10000; i++) {
           pjp.proceed();
        }
        long end = System.currentTimeMillis();
        System.out.println("万次执行："+ className+"."+methodName+"---->" +(end-start) + "ms");
    }
}
```

> AOP 通知获取数据

AOP 用于增强方法，在通知类对应的增强方法中，我想操作原方法就得先获取原方法的一些信息：参数，返回值，异常等

AOP 就提供了获取这些信息的方法，但是对于不同通知类型，可以拿到的信息不同。

- 获取切入点方法的参数，所有的通知类型都可以获取参数
  - JoinPoint：适用于前置、后置、返回后、抛出异常后通知
  - ProceedingJoinPoint：适用于环绕通知
- 获取切入点方法返回值，前置和抛出异常后通知是没有返回值，后置通知可有可无，所以不做研究
  - 返回后通知
  - 环绕通知
- 获取切入点方法运行异常信息，前置和返回后通知是不会有，后置通知可有可无，所以不做研究
  - 抛出异常后通知
  - 环绕通知

##### 获取参数

```java
非环绕通知获取方式
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(* com.itheima.dao.BookDao.findName(..))")
    private void pt(){}

    @Before("pt()")
    public void before(JoinPoint jp)
        Object[] args = jp.getArgs();
        System.out.println(Arrays.toString(args));
        System.out.println("before advice ..." );
    }
	//...其他的略
}


环绕通知获取方式
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(* com.itheima.dao.BookDao.findName(..))")
    private void pt(){}

    @Around("pt()")
    public Object around(ProceedingJoinPoint pjp)throws Throwable {
        Object[] args = pjp.getArgs();
        System.out.println(Arrays.toString(args));
        Object ret = pjp.proceed();
        return ret;
    }
	//其他的略
}
特别的，在环绕通知中，我们是可以将参数进行修改的
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(* com.itheima.dao.BookDao.findName(..))")
    private void pt(){}

    @Around("pt()")
    public Object around(ProceedingJoinPoint pjp) throws Throwable{
        Object[] args = pjp.getArgs();
        System.out.println(Arrays.toString(args));
        args[0] = 666;
        Object ret = pjp.proceed(args);
        return ret;
    }
	//其他的略
}

```

##### 获取返回值

```java
环绕通知获取返回值
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(* com.itheima.dao.BookDao.findName(..))")
    private void pt(){}

    @Around("pt()")
    public Object around(ProceedingJoinPoint pjp) throws Throwable{
        Object[] args = pjp.getArgs();
        System.out.println(Arrays.toString(args));
        args[0] = 666;
        Object ret = pjp.proceed(args);
        return ret;
    }
	//其他的略
}

返回后通知获取返回值
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(* com.itheima.dao.BookDao.findName(..))")
    private void pt(){}

    @AfterReturning(value = "pt()",returning = "ret")
    public void afterReturning(Object ret) {
        System.out.println("afterReturning advice ..."+ret);
    }
	//其他的略
}
returning = "参数名" 这里必须对应上
```

##### 获取异常

```java
环绕通知获取异常
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(* com.itheima.dao.BookDao.findName(..))")
    private void pt(){}

    @Around("pt()")
    public Object around(ProceedingJoinPoint pjp){
        Object[] args = pjp.getArgs();
        System.out.println(Arrays.toString(args));
        args[0] = 666;
        Object ret = null;
        try{
            ret = pjp.proceed(args);
        }catch(Throwable throwable){
            t.printStackTrace();
        }
        return ret;
    }
	//其他的略
}

抛出异常后通知获取异常
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(* com.itheima.dao.BookDao.findName(..))")
    private void pt(){}

    @AfterThrowing(value = "pt()",throwing = "t")
    public void afterThrowing(Throwable t) {
        System.out.println("afterThrowing advice ..."+t);
    }
	//其他的略
}
```

## AOP 事务管理

- Spring 事务作用：在数据层或**业务层**保障一系列的数据库操作同成功同失败

数据层有事务我们可以理解，为什么业务层也需要处理事务呢?举个简单的例子，

- 转账业务会有两次数据层的调用，一次是加钱一次是减钱
- 把事务放在数据层，加钱和减钱就有两个事务
- 没办法保证加钱和减钱同时成功或者同时失败
- 这个时候就需要将事务放在业务层进行处理。

Spring 为了管理事务，提供了一个平台事务管理器`PlatformTransactionManager`

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-osWvCadt-1668042845700)(assets/1630243651541.png)]](https://img-blog.csdnimg.cn/28e2877f36654e94aba88ef78f2f4e77.png)

commit 是用来提交事务，rollback 是用来回滚事务。

PlatformTransactionManager 只是一个接口，Spring 还为其提供了一个具体的实现:

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-edeDtS5y-1668042845701)(assets/1630243993380.png)]](https://img-blog.csdnimg.cn/287ae6bb6cd94f19b8a521a877f6f1a9.png)

从名称上可以看出，我们只需要给它一个 DataSource 对象，它就可以帮你去在业务层管理事务。其内部采用的是 JDBC 的事务。所以说如果你持久层采用的是 JDBC 相关的技术，就可以采用这个事务管理器来管理你的事务。而 Mybatis 内部采用的就是 JDBC 的事务，所以后期我们 Spring 整合 Mybatis 就采用的这个 DataSourceTransactionManager 事务管理器。

##### 转账案例-需求分析

需求: 实现任意两个账户间转账操作

需求微缩: A 账户减钱，B 账户加钱

为了实现上述的业务需求，我们可以按照下面步骤来实现下:
①：数据层提供基础操作，指定账户减钱（outMoney），指定账户加钱（inMoney）

②：业务层提供转账操作（transfer），调用减钱与加钱的操作

③：提供 2 个账号和操作金额执行转账操作

④：基于 Spring 整合 MyBatis 环境搭建上述操作

> 环境配置

```java
步骤1:准备数据库表
create database spring_db character set utf8;
use spring_db;
create table tbl_account(
    id int primary key auto_increment,
    name varchar(35),
    money double
);
insert into tbl_account values(1,'Tom',1000);
insert into tbl_account values(2,'Jerry',1000);


步骤2:创建项目导入jar包
<dependencies>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>5.2.10.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid</artifactId>
      <version>1.1.16</version>
    </dependency>

    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.5.6</version>
    </dependency>

    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.47</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>5.2.10.RELEASE</version>
    </dependency>

    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>1.3.0</version>
    </dependency>

    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
      <scope>test</scope>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>5.2.10.RELEASE</version>
    </dependency>

  </dependencies>

步骤3:根据表创建模型类
public class Account implements Serializable {

    private Integer id;
    private String name;
    private Double money;
	//setter...getter...toString...方法略
}

步骤4:创建Dao接口
public interface AccountDao {

    @Update("update tbl_account set money = money + #{money} where name = #{name}")
    void inMoney(@Param("name") String name, @Param("money") Double money);

    @Update("update tbl_account set money = money - #{money} where name = #{name}")
    void outMoney(@Param("name") String name, @Param("money") Double money);
}

步骤5:创建Service接口和实现类
public interface AccountService {
    /**
     * 转账操作
     * @param out 传出方
     * @param in 转入方
     * @param money 金额
     */
    public void transfer(String out,String in ,Double money) ;
}

@Service
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountDao accountDao;

    public void transfer(String out,String in ,Double money) {
        accountDao.outMoney(out,money);
        accountDao.inMoney(in,money);
    }

}
步骤6:添加jdbc.properties文件
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/spring_db?useSSL=false
jdbc.username=root
jdbc.password=root

步骤7:创建JdbcConfig配置类
public class JdbcConfig {
    @Value("${jdbc.driver}")
    private String driver;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.username}")
    private String userName;
    @Value("${jdbc.password}")
    private String password;

    @Bean
    public DataSource dataSource(){
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName(driver);
        ds.setUrl(url);
        ds.setUsername(userName);
        ds.setPassword(password);
        return ds;
    }
}

步骤8:创建MybatisConfig配置类
public class MybatisConfig {

    @Bean
    public SqlSessionFactoryBean sqlSessionFactory(DataSource dataSource){
        SqlSessionFactoryBean ssfb = new SqlSessionFactoryBean();
        ssfb.setTypeAliasesPackage("com.itheima.domain");
        ssfb.setDataSource(dataSource);
        return ssfb;
    }

    @Bean
    public MapperScannerConfigurer mapperScannerConfigurer(){
        MapperScannerConfigurer msc = new MapperScannerConfigurer();
        msc.setBasePackage("com.itheima.dao");
        return msc;
    }
}

步骤9:创建SpringConfig配置类
@Configuration
@ComponentScan("com.itheima")
@PropertySource("classpath:jdbc.properties")
@Import({JdbcConfig.class,MybatisConfig.class})
public class SpringConfig {
}

步骤10:编写测试类
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpringConfig.class)
public class AccountServiceTest {

    @Autowired
    private AccountService accountService;

    @Test
    public void testTransfer() throws IOException {
        accountService.transfer("Tom","Jerry",100D);
    }

}
```

> 事务管理

```java
如果业务层transfer函数没有加事务逻辑，那么主函数如果是：
@Service
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountDao accountDao;

    public void transfer(String out,String in ,Double money) {
        accountDao.outMoney(out,money);
        int i = 1/0; //钱就会只少不加
        accountDao.inMoney(in,money);
    }

}

事务开启步骤：
步骤1:在需要被事务管理的方法上添加注解
public interface AccountService {
    /**
     * 转账操作
     * @param out 传出方
     * @param in 转入方
     * @param money 金额
     */
    //配置当前接口方法具有事务
    public void transfer(String out,String in ,Double money) ;
}

@Service
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountDao accountDao;
	@Transactional
    public void transfer(String out,String in ,Double money) {
        accountDao.outMoney(out,money);
        int i = 1/0;
        accountDao.inMoney(in,money);
    }

}
@Transactional可以写在接口类上、接口方法上、实现类上和实现类方法上
	写在接口类上，该接口的所有实现类的所有方法都会有事务
	写在接口方法上，该接口的所有实现类的该方法都会有事务
	写在实现类上，该类中的所有方法都会有事务
	写在实现类方法上，该方法上有事务
	建议写在实现类或实现类的方法上

步骤2:在JdbcConfig类中配置事务管理器
public class JdbcConfig {
    @Value("${jdbc.driver}")
    private String driver;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.username}")
    private String userName;
    @Value("${jdbc.password}")
    private String password;

    @Bean
    public DataSource dataSource(){
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName(driver);
        ds.setUrl(url);
        ds.setUsername(userName);
        ds.setPassword(password);
        return ds;
    }

    //配置事务管理器，mybatis使用的是jdbc事务
    @Bean
    public PlatformTransactionManager transactionManager(DataSource dataSource){
        DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();
        transactionManager.setDataSource(dataSource);
        return transactionManager;
    }
}

步骤3：开启事务注解
@Configuration
@ComponentScan("com.itheima")
@PropertySource("classpath:jdbc.properties")
@Import({JdbcConfig.class,MybatisConfig.class})
//开启注解式事务驱动
@EnableTransactionManagement
public class SpringConfig {
}
```

#### Spring 事务角色

> 未开启 Spring 事务之前:

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-zWhCn5mO-1668042845702)(assets/1630248794837.png)]](https://img-blog.csdnimg.cn/9074d12e751740e79151bfe8c625fe5b.png)

- AccountDao 的 outMoney 因为是修改操作，会开启一个事务 T1
- AccountDao 的 inMoney 因为是修改操作，会开启一个事务 T2
- AccountService 的 transfer 没有事务，
  - 运行过程中如果没有抛出异常，则 T1 和 T2 都正常提交，数据正确
  - 如果在两个方法中间抛出异常，T1 因为执行成功提交事务，T2 因为抛异常不会被执行
  - 就会导致数据出现错误

> 开启 Spring 的事务管理后

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-Kxvpxs5u-1668042845703)(assets/1630249111055.png)]](https://img-blog.csdnimg.cn/60d89e1faa724371b30db1c09708890e.png)

- transfer 上添加了@Transactional 注解，在该方法上就会有一个事务 T
- AccountDao 的 outMoney 方法的事务 T1 加入到 transfer 的事务 T 中
- AccountDao 的 inMoney 方法的事务 T2 加入到 transfer 的事务 T 中
- 这样就保证他们在同一个事务中，当业务层中出现异常，整个事务就会回滚，保证数据的准确性。

通过上面例子的分析，我们就可以得到如下概念:

- 事务管理员：发起事务方，在 Spring 中通常指代业务层开启事务的方法
- 事务协调员：加入事务方，在 Spring 中通常指代数据层方法，也可以是业务层方法

注意:

目前的事务管理是基于`DataSourceTransactionManager`和`SqlSessionFactoryBean`使用的是同一个数据源。

#### Spring 事务属性

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-Vx06lVrA-1668042845704)(assets/1630250069844.png)]](https://img-blog.csdnimg.cn/38c86a28424d42c2beb1dd572890e50b.png)

上面这些属性都可以在`@Transactional`注解的参数上进行设置。

- readOnly：true 只读事务，false 读写事务，增删改要设为 false,查询设为 true。
- timeout:设置超时时间单位秒，在多长时间之内事务没有提交成功就自动回滚，-1 表示不设置超时时间。
- rollbackFor:当出现指定异常进行事务回滚
- noRollbackFor:当出现指定异常不进行事务回滚

> 事务传播行为

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-45yGYrfM-1668042845704)(assets/1630253779575.png)]](https://img-blog.csdnimg.cn/5d8381e565584ecba6b08748717e6a32.png)

对于上述案例的分析:

- log 方法、inMoney 方法和 outMoney 方法都属于增删改，分别有事务 T1,T2,T3
- transfer 因为加了@Transactional 注解，也开启了事务 T
- 前面我们讲过 Spring 事务会把 T1,T2,T3 都加入到事务 T 中
- 所以当转账失败后，所有的事务都回滚，导致日志没有记录下来
- 这和我们的需求不符，这个时候我们就想能不能让 log 方法单独是一个事务呢?

要想解决这个问题，就需要用到事务传播行为，所谓的事务传播行为指的是:

事务传播行为：事务协调员对事务管理员所携带事务的处理态度。

具体如何解决，就需要用到之前我们没有说的`propagation属性`。

```java
@Service
public class LogServiceImpl implements LogService {

    @Autowired
    private LogDao logDao;
	//propagation设置事务属性：传播行为设置为当前操作需要新事务
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void log(String out,String in,Double money ) {
        logDao.log("转账操作由"+out+"到"+in+",金额："+money);
    }
}
运行后，就能实现我们想要的结果，不管转账是否成功，都会记录日志。
```
