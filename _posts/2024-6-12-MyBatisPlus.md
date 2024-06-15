---
layout: post
title: "MyBatisPlus"
date: 2024-6-12
tags: [SSM]
comments: true
author: zxy
---

## MyBatisPlus 入门案例与简介

### 入门案例

1.**创建数据库及表**

```sql
create database if not exists mybatisplus_db character set utf8;
use mybatisplus_db;
CREATE TABLE user (
id bigint(20) primary key auto_increment,
name varchar(32) not null,
password varchar(32) not null,
age int(3) not null ,
tel varchar(32) not null
);
insert into user values(1,'Tom','tom',3,'18866668888');
insert into user values(2,'Jerry','jerry',4,'16688886666');
insert into user values(3,'Jock','123456',41,'18812345678');
insert into user values(4,'传智播客','itcast',15,'4006184000');
```

2.**创建 SpringBoot 工程**

3.**勾选配置使用技术**：只有 MySQL，由于 MP 并未被收录到 idea 的系统内置配置，无法直接选择加入，需要手动在 pom.xml 中配置添加

4.**pom.xml 补全依赖**

```xml
		<!-- mybatis-plus -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.4.1</version>
        </dependency>
        <!-- Druid连接池 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.16</version>
        </dependency>
```

- druid 数据源可以加也可以不加，SpringBoot 有内置的数据源，可以配置成使用 Druid 数据源。
- 从 MP 的依赖关系可以看出，通过依赖传递已经将**MyBatis**与**MyBatis 整合 Spring**的 jar 包导入，我们不需要额外在添加 MyBatis 的相关 jar 包。

  5.**添加 MP 的相关配置信息**

resources 默认生成的是 properties 配置文件，可以将其替换成 yml 文件，并在文件中配置数据库连接的相关信息: application.yml

```yaml
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/mybatisplus_db?serverTimezone=UTC
    username: root
    password: hsp
```

6.**根据数据库表创建实体类**

```java
com.itheima.pojo.User
public class User {
    private Long id;
    private String name;
    private String password;
    private Integer age;
    private String tel;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public String getTel() {
        return tel;
    }

    public void setTel(String tel) {
        this.tel = tel;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", password='" + password + '\'' +
                ", age=" + age +
                ", tel='" + tel + '\'' +
                '}';
    }
}

```

7.**创建 Dao 接口(核心)**

```java
com.itheima.dao.UserDao这里要继承一个类BaseMapper
@Mapper
public interface UserDao extends BaseMapper<User> {
}

```

8.**编写引导类**

com.itheima.Mybatisplus01QuickstartApplication

```java
@SpringBootApplication
public class Mybatisplus01QuickstartApplication {

    public static void main(String[] args) {
        SpringApplication.run(Mybatisplus01QuickstartApplication.class, args);
    }

}
```

9.**编写测试类**

```java
@SpringBootTest
class Mybatisplus01QuickstartApplicationTests {
    @Autowired
    private UserDao userDao;

    @Test
    void testGetAll() {
        List<User> list = userDao.selectList(null);
        System.out.println(list);
    }

}
```

### MybatisPlus 简介

MP 旨在成为 MyBatis 的最好搭档，而不是替换 MyBatis,所以可以理解为 MP 是 MyBatis 的一套增强工具，它是在 MyBatis 的基础上进行开发的，我们虽然使用 MP 但是底层依然是 MyBatis 的东西，也就是说我们也可以在 MP 中写 MyBatis 的内容。

MP 的特性:

- 无侵入：只做增强不做改变，不会对现有工程产生影响
- 强大的 CRUD 操作：内置通用 Mapper，少量配置即可实现单表 CRUD 操作
- 支持 Lambda：编写查询条件无需担心字段写错
- 支持主键自动生成
- 内置分页插件

## 标准数据层开发

### 标准 CRUD 使用

| 功能           | 自定义接口                              | MP 接口                                         |
| -------------- | --------------------------------------- | ----------------------------------------------- |
| 新增           | boolean save(T t)                       | int insert(T t)                                 |
| 删除           | boolean delete(int id)                  | int deleteById(Serializable id)                 |
| 修改           | boolean update(T t)                     | int updateById(T t)                             |
| 根据 id 查询   | T getById(int id)                       | T selectById(Serializable id)                   |
| 查询全部       | List< T > getAll()                      | List< T > selectList(Wrapper< T > queryWrapper) |
| 分页查询       | PageInfo< T > getAll(int page,int size) | IPage< T >page,Wrapper< T > queryWrapper        |
| 按条件分页查询 | List< T > getAll(Condition condition)   | IPage< T >page,Wrapper< T > queryWrapper        |

### 新增

```java
@SpringBootTest
class Mybatisplus01QuickstartApplicationTests {
    @Autowired
    private UserDao userDao;
	@Test
    public void testSave() {
        User user = new User();
        user.setName("小王");
        user.setPassword("abc");
        user.setAge(17);
        user.setTel("13565781121");
        int affectedRow = userDao.insert(user);
        System.out.println("affectedRow:" + affectedRow);
    }
 }
```

### 删除

```java
@SpringBootTest
class Mybatisplus01QuickstartApplicationTests {
    @Autowired
    private UserDao userDao;
	@Test
    public void testDelete() {
        int i = userDao.deleteById(1621908110012145665L);
        System.out.println("i:" + i);
    }
 }
int:返回值类型，数据删除成功返回1，未删除数据返回0
MP使用Serializable作为参数类型，就好比我们可以用Object接收任何数据类型一样
```

### 修改

```java
@SpringBootTest
class Mybatisplus01QuickstartApplicationTests {
    @Autowired
    private UserDao userDao;
	 @Test
    public void testUpdate() {
        User user = new User();
        user.setPassword("ccc");
        user.setName("Tommy");
        user.setAge(21);
        user.setId(1L);
        int i = userDao.updateById(user);
        System.out.println("i:" + i);
    }
 }
int:返回值，修改成功后返回1，未修改数据返回0。
```

### 根据 ID 查询

```java
@SpringBootTest
class Mybatisplus01QuickstartApplicationTests {
    @Autowired
    private UserDao userDao;
	@Test
    public void testGetById() {
        User user = userDao.selectById(2L);
        System.out.println(user);
    }
 }
```

### 查询所有

```java
List<T> selectList(Wrapper<T> queryWrapper)
Wrapper：用来构建条件查询的条件，目前我们没有可直接传为Null
List:因为查询的是所有，所以返回的数据是一个集合

@SpringBootTest
class Mybatisplus01QuickstartApplicationTests {
    @Autowired
    private UserDao userDao;
	@Test
    void testGetAll() {
        List<User> list = userDao.selectList(null);
        System.out.println(list);
    }
 }
```

### Lombok

一个 Java 类库，提供了一组注解，简化 POJO 实体类开发

```
步骤1:添加lombok依赖
<!-- lombok对实体类快速标注 -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <!--<version>1.18.12</version>-->
        </dependency>

步骤2:安装Lombok的插件
	新版本已经内置此插件

步骤3:模型类上添加注解
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Long id;
    private String name;
    private String password;
    private Integer age;
    private String tel;
    //只想要某几个参数的构造函数
    public User(String name, String password) {
        this.name = name;
        this.password = password;
    }
}

```

- @Setter:为模型类的属性提供 setter 方法
- @Getter:为模型类的属性提供 getter 方法
- @ToString:为模型类的属性提供 toString 方法
- @EqualsAndHashCode:为模型类的属性提供 equals 和 hashcode 方法
- @Data:是个组合注解，包含上面的注解的功能
- @NoArgsConstructor:提供一个无参构造函数
- @AllArgsConstructor:提供一个包含所有参数的构造函数

### 分页功能

**步骤 1:调用方法传入参数获取返回值**

```java
//分页查询
 @Test
    void testSelectPage() {
        // 1 创建IPage分页对象,设置分页参数,1为当前页码，3为每页显示的记录数
        Page<User> page = new Page<>(1, 3);
        //2 执行分页查询
        userDao.selectPage(page, null);
        //3 获取分页结果
        System.out.println("当前页码值：" + page.getCurrent());
        System.out.println("每页显示数：" + page.getSize());
        System.out.println("一共多少页：" + page.getPages());
        System.out.println("一共多少条数据：" + page.getTotal());
        System.out.println("数据：" + page.getRecords());

    }
```

**步骤 2:设置分页拦截器**

```java
@Configuration
    public class MybatisPlusConfig {
        @Bean
        public MybatisPlusInterceptor mybatisPlusInterceptor(){
        //1 创建MybatisPlusInterceptor拦截器对象
            MybatisPlusInterceptor mpInterceptor=new MybatisPlusInterceptor();
        //2 添加分页拦截器
            mpInterceptor.addInnerInterceptor(new PaginationInnerInterceptor());
            return mpInterceptor;
        }
    }
```

**步骤 3:运行测试程序**

如果想查看 MP 执行的 SQL 语句，可以修改 application.yml 配置文件，

```java
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

## DQL 编程控制

### 条件查询

##### LambdaQueryWrapper 使用 lambda

```java
@SpringBootTest
class Mybatisplus02DqlApplicationTests {
    @Autowired
    private UserDao userDao;

    @Test
    public void testGetUserByAgeLambdaQuery() {
        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
        wrapper.lt(User::getAge, 41);
        List<User> list = userDao.selectList(wrapper);
        list.forEach(System.out::println);
    }

}

lt (小于) less than
le（小于等于）less equal
eq（等于）equal
gt（大于）greater than
ge（大于等于）greater equal
ne (不等于)not equal
```

#### 多条件构建

```java
条件用and连接
@SpringBootTest
class Mybatisplus02DqlApplicationTests {
    @Autowired
    private UserDao userDao;

    @Test
    public void testGetUserAgeManWrappersChain() {
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        //10 < age < 30
        wrapper.lambda().gt(User::getAge, 10).lt(User::getAge, 30);
        List<User> list = userDao.selectList(wrapper);
        list.forEach(System.out::println);
    }

}


条件用or连接
@SpringBootTest
class Mybatisplus02DqlApplicationTests {
    @Autowired
    private UserDao userDao;

   @Test
    public void testGetUserAgeManWrappersChainOr() {
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        //age < 10 || age > 30
        wrapper.lambda().lt(User::getAge, 10).or().gt(User::getAge, 30);
        List<User> list = userDao.selectList(wrapper);
        list.forEach(System.out::println);
    }

}
```

#### null 判定

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Long id;
    private String name;
    private String password;
    private Integer age;
    private String tel;
}


新建一个模型类,让其继承User类，并在其中添加age2属性，UserQuery在拥有User属性后同时添加了age2属性。
@Data
@AllArgsConstructor
@NoArgsConstructor
public class UserQuery extends User {
    private Integer age2;
}

@SpringBootTest
class Mybatisplus02DqlApplicationTests {
    @Autowired
    private UserDao userDao;

    @Test
    public void testGetUserAgeWrappers() {
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        // 模拟页面传过来的2条数据
        UserQuery userQuery = new UserQuery();
        userQuery.setAge(10);
        userQuery.setAge2(30);
		// condition为boolean类型，返回true，则添加条件，返回false则不添加条件
        wrapper.lambda().gt(null != userQuery.getAge(), User::getAge, userQuery.getAge());
        wrapper.lambda().lt(null != userQuery.getAge2(), User::getAge, userQuery.getAge2());

        List<User> list = userDao.selectList(wrapper);
        list.forEach(System.out::println);
    }

}


```

### 查询投影

#### 查询指定字段

```java
@SpringBootTest
class Mybatisplus02DqlApplicationTests {
    @Autowired
    private UserDao userDao;

    @Test
    public void testProjection() {
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        wrapper.lambda().select(User::getName, User::getAge, User::getTel);
        List<User> list = userDao.selectList(wrapper);
        list.forEach(System.out::println);
    }
}


```

### 聚合查询

> 需求:聚合函数查询，完成 count、max、min、avg、sum 的使用
> count:总记录数
> max:最大值
> min:最小值
> avg:平均值
> sum:求和

```java
@SpringBootTest
class Mybatisplus02DqlApplicationTests {
    @Autowired
    private UserDao userDao;

   @Test
    public void testPolymerizationMax() {
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        // 起了个别名
        //wrapper.select("max(age) as maxAge");
        //wrapper.select("avg(age) as avgAge");
        //wrapper.select("min(age) as minAge");
        wrapper.select("sum(age) as sumAge");
        List<Map<String, Object>> maps = userDao.selectMaps(wrapper);
        System.out.println(maps);
    }
}
```

### 分组，排序查询

```java
@SpringBootTest
class Mybatisplus02DqlApplicationTests {
    @Autowired
    private UserDao userDao;

   @Test
    public void testPolymerizationGroup() {
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        // 起了个别名
        wrapper.select("count(age) as countAge,tel");
        wrapper.groupBy("tel");
        wrapper.orderByDesc("tel");
        List<Map<String, Object>> maps = userDao.selectMaps(wrapper);
        System.out.println(maps);
    }
}

```

### 模糊查询

```java
@SpringBootTest
class Mybatisplus02DqlApplicationTests {
    @Autowired
    private UserDao userDao;

    @Test
    public void testSelectWrapperLike() {
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        wrapper.lambda().likeRight(User::getName, "J");
        List<User> list = userDao.selectList(wrapper);
        list.forEach(System.out::println);
    }
}
```

- like():前后加百分号,如 %J%
- likeLeft():前面加百分号,如 %J
- likeRight():后面加百分号,如 J%

### 映射匹配兼容性

前面我们已经能从表中查询出数据，并将数据封装到模型类中，这整个过程涉及到一张表和一个模型类:
![在这里插入图片描述](https://img-blog.csdnimg.cn/9366ee142f9b4e889b27a3be22813f3e.png#pic_center)
之所以数据能够成功的从表中获取并封装到模型对象中，原因是表的字段列名和模型类的属性名一样。

**问题 1:表字段与编码属性设计不同步**

> MP 给我们提供了一个注解@TableField ,使用该注解可以实现模型类属性名和表的列名之间的映射关系。

![在这里插入图片描述](https://img-blog.csdnimg.cn/5c6e8a1341ad4c86ba54e6e03ec8936e.png#pic_center)

**问题 2:编码中添加了数据库中未定义的属性**

![在这里插入图片描述](https://img-blog.csdnimg.cn/74c3047bde6f4707823afb35548e75c0.png#pic_center)

**问题 3：采用默认查询开放了更多的字段查看权限**

![在这里插入图片描述](https://img-blog.csdnimg.cn/e089f7605dab43b8a8fc291adf0745f8.png#pic_center)

**问题 4:表名与编码开发设计不同步**

![在这里插入图片描述](https://img-blog.csdnimg.cn/b2530c2493114b14af96249bdd70da35.png#pic_center)

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@TableName("tbl_user")
public class User {
    private Long id;
    private String name;
    @TableField(value = "pwd", select = false)
    private String password;
    private Integer age;
    private String tel;
    @TableField(exist = false)
    private Double height;
}
```

## DML 编程控制

### id 生成策略控制

- AUTO 的作用是使用数据库 ID 自增，在使用该策略的时候一定要确保对应的数据库表设置了 ID 主键自增，否则无效。
- NONE: 不设置 id 生成策略
- INPUT:用户手工输入 id
- ASSIGN_ID:雪花算法生成 id(可兼容数值型与字符串型)
- ASSIGN_UUID:以 UUID 生成算法作为 id 生成策略,使用 uuid 需要注意的是，主键的类型不能是 Long，而应该改成 String 类型

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@TableName("tbl_user")
public class User {
    @TableId(type = IdType.AUTO)
    private Long id;
    private String name;
    @TableField(value = "pwd", select = false)
    private String password;
    private Integer age;
    private String tel;
    @TableField(exist = false)
    private Double height;
}

```

> 雪花算法

雪花算法(SnowFlake),是 Twitter 官方给出的算法实现 是用 Scala 写的。其生成的结果是一个 64bit 大小整数，它的结构如下图:
![在这里插入图片描述](https://img-blog.csdnimg.cn/5f692d04a8fd423391a06bbd8e03dc0b.png#pic_center)

1. 1bit,不用,因为二进制中最高位是符号位，1 表示负数，0 表示正数。生成的 id 一般都是用整数，所以最高位固定为 0。
2. 41bit-时间戳，用来记录时间戳，毫秒级。
3. 10bit-工作机器 id，用来记录工作机器 id,其中高位 5bit 是数据中心 ID 其取值范围 0-31，低位 5bit 是工作节点 ID 其取值范围 0-31，两个组合起来最多可以容纳 1024 个节点。
4. 序列号占用 12bit，每个节点每毫秒 0 开始不断累加，最多可以累加到 4095，一共可以产生 4096 个 ID。

#### ID 生成策略对比

- NONE: 不设置 id 生成策略，MP 不自动生成，约等于 INPUT,所以这两种方式都需要用户手动设置，但是手动设置第一个问题是容易出现相同的 ID 造成主键冲突，为了保证主键不冲突就需要做很多判定，实现起来比较复杂。
- AUTO:数据库 ID 自增,这种策略适合在数据库服务器只有 1 台的情况下使用,**不可作为分布式 ID 使用**。
- ASSIGN_UUID:可以在分布式的情况下使用，而且能够保证唯一，但是生成的主键是 32 位的字符串，长度过长占用空间而且还不能排序，查询性能也慢。
- ASSIGN_ID:可以在分布式的情况下使用，生成的是 Long 类型的数字，可以排序性能也高，但是生成的策略和服务器时间有关，如果修改了系统时间就有可能导致出现重复主键。
- 综上所述，每一种主键策略都有自己的优缺点，根据自己项目业务的实际情况来选择使用才是最明智的选择。

#### 简化配置

```yaml
#mybatis-plus日志控制台输出,可以在控制台打印出对应的SQL语句，
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  global-config:
    banner: off #关闭mybatisplus启动图标
    db-config:
      id-type: assign_id # 让所有的模型类都可以使用该主键ID策略
      table-prefix: tbl_ # 如果所有的数据库表都_tbl开头，那么增加此设置后，可以对应实体类
```

### 多记录操作

> 需求:根据传入的 id 集合将数据库表中的数据删除掉。

```java
@SpringBootTest
class Mybatisplus03DmlApplicationTests {
    @Autowired
    private UserDao userDao;

   @Test
    public void testBatchDelete() {
        ArrayList<Long> list = new ArrayList<>();
        list.add(1622190878117908482L);
        list.add(1622205120405667841L);
        list.add(1622206703335985154L);
        userDao.deleteBatchIds(list);
    }

}
```

> 需求：根据传入的 ID 集合查询用户信息

```java
@SpringBootTest
class Mybatisplus03DmlApplicationTests {
    @Autowired
    private UserDao userDao;

  @Test
    public void testBatchSelect() {
        ArrayList<Long> list = new ArrayList<>();
        list.add(1L);
        list.add(2L);
        list.add(3L);
        list.add(4L);
        List<User> users = userDao.selectBatchIds(list);
        users.forEach(System.out::println);
    }
}

```

### 逻辑删除

定义：为数据设置是否可用状态字段，删除时设置状态字段为不可用状态，数据保留在数据库中，执行的是 update 操作。

通俗一点来说就是，数据不删，但是不可用。

**步骤 1:修改数据库表添加 deleted 列**

```sql
字段名可以任意，内容也可以自定义，比如0代表正常，1代表删除，可以在添加列的同时设置其默认值为0正常。

ALTER TABLE mybatisplus_db.`tbl_user` ADD deleted INT DEFAULT 0 NOT NULL;

```

**步骤 2:实体类添加属性**

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
//@TableName("tbl_user")
public class User {
    //@TableId(type = IdType.ASSIGN_UUID)
    private Long id;
    private String name;
    @TableField(value = "pwd", select = false)
    private String password;
    private Integer age;
    private String tel;
    @TableField(exist = false)
    private Double height;
    /**
     * 1表示为无效(逻辑删除，离职员工)
     * 0表示为有效(在职员工)
     */
    @TableLogic(value = "0", delval = "1")
    private Integer deleted;

}
```

> 如果每个表都要有逻辑删除，那么就需要在每个模型类的属性上添加@TableLogic 注解，如何优化?

```
#mybatis-plus日志控制台输出,可以在控制台打印出对应的SQL语句，
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  global-config:
    banner: off #关闭mybatisplus启动图标
    db-config:
      id-type: assign_id # 让所有的模型类都可以使用该主键ID策略
      table-prefix: tbl_ # 如果所有的数据库表都_tbl开头，那么增加此设置后，可以对应实体类
      # 逻辑删除字段名
      logic-delete-field: deleted
      # 逻辑删除字面值：删除为1
      logic-delete-value: 1
      # 逻辑删除字面值：未删除为0
      logic-not-delete-value: 0

```

### 乐观锁

在讲解乐观锁之前，我们还是先来分析下问题:
业务并发现象带来的问题:**秒杀**。

- 假如有 100 个商品或者票在出售，为了能保证每个商品或者票只能被一个人购买，如何保证不会出现超买或者重复卖。
- 对于这一类问题，其实有很多的解决方案可以使用。
- 第一个最先想到的就是锁，锁在一台服务器中是可以解决的，但是如果在多台服务器下锁就没有办。
  法控制，比如 12306 有两台服务器在进行卖票，在两台服务器上都添加锁的话，那也有可能会导致。
  在同一时刻有两个线程在进行卖票，还是会出现并发问题。
- 我们接下来介绍的这种方式是针对于小型企业的解决方案，因为数据库本身的性能就是个瓶颈，如果对其并发量超过 2000 以上的就需要考虑其他的解决方案了。

简单来说，乐观锁主要解决的问题是当要更新一条记录的时候，希望这条记录没有被别人更新。

#### 实现思路

乐观锁的实现方式:

- 数据库表中添加 version 列，比如默认值给 1。
- 第一个线程要修改数据之前，取出记录时，获取当前数据库中的 version=1
- 第二个线程要修改数据之前，取出记录时，获取当前数据库中的 version=1
- 第一个线程执行更新时，set version = newVersion where version = oldVersion
  newVersion = version+1 [2]
  oldVersion = version [1]
- 第二个线程执行更新时，set version = newVersion where version = oldVersion
  newVersion = version+1 [2]
  oldVersion = version [1]
- 假如这两个线程都来更新数据，第一个和第二个线程都可能先执行
  假如第一个线程先执行更新，会把 version 改为 2，
  第二个线程再更新的时候，set version = 2 where version = 1,此时数据库表的数据 version 已经为 2，所以第二个线程会修改失败。
  假如第二个线程先执行更新，会把 version 改为 2，
  第一个线程再更新的时候，set version = 2 where version = 1,此时数据库表的数据 version 已经为 2，所以第一个线程会修改失败。
  不管谁先执行都会确保只能有一个线程更新数据，这就是 MP 提供的乐观锁的实现原理分析。

#### 实现步骤

**步骤 1:数据库表添加列**

```sql
列名可以任意，比如使用version ,给列设置默认值为1
ALTER TABLE mybatisplus_db.`tbl_user` ADD VERSION INT DEFAULT 1 NOT NULL;
```

**步骤 2:在模型类中添加对应的属性**

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
//@TableName("tbl_user")
public class User {
    //@TableId(type = IdType.ASSIGN_UUID)
    private Long id;
    private String name;
    @TableField(value = "pwd", select = false)
    private String password;
    private Integer age;
    private String tel;
    @TableField(exist = false)
    private Double height;
    /**
     * 1表示为无效(逻辑删除，离职员工)
     * 0表示为有效(在职员工)
     */
    //@TableLogic(value = "0", delval = "1")
    private Integer deleted;
    @Version
    private Integer version;

}
```

**步骤 3:添加乐观锁的拦截器**

```java
com.itheima.interceptor.MpConfig
@Configuration
public class MpConfig {
    @Bean
    public MybatisPlusInterceptor mpInterceptor() {
        //1.定义Mp拦截器
        MybatisPlusInterceptor mpInterceptor = new MybatisPlusInterceptor();
        //2.添加乐观锁拦截器
        mpInterceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        return mpInterceptor;
    }
}

```

## 快速开发：利用代码生成器

**步骤 1:创建一个 Maven 项目**

```
SpringBoot类型的
插件为MySQL
```

**步骤 2:导入对应的依赖**

```xml
<!-- mybatis-plus -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.4.1</version>
        </dependency>
        <!-- Druid连接池 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.16</version>
        </dependency>
        <!-- lombok对实体类快速标注 -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <!--<version>1.18.12</version>-->
        </dependency>

        <!--代码生成器-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-generator</artifactId>
            <version>3.4.1</version>
        </dependency>
        <!--velocity模板引擎-->
        <dependency>
            <groupId>org.apache.velocity</groupId>
            <artifactId>velocity-engine-core</artifactId>
            <version>2.3</version>
        </dependency>

```

**步骤 3:创建代码生成类**

```java
@SuppressWarnings({"all"})
public class CodeGenerator {
    public static void main(String[] args) {
        //1.获取代码生成器的对象
        AutoGenerator autoGenerator = new AutoGenerator();
        //设置数据库相关配置
        DataSourceConfig dataSource = new DataSourceConfig();
        dataSource.setDriverName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/mybatisplus_db?serverTimezone=UTC");
        dataSource.setUsername("root");
        dataSource.setPassword("123456");
        autoGenerator.setDataSource(dataSource);
        //设置全局配置
        GlobalConfig globalConfig = new GlobalConfig();
        globalConfig.setOutputDir(System.getProperty("user.dir") + "/mybatisplus_04_generator/src/main/java"); //设置代码生成位置
        globalConfig.setOpen(false); //设置生成完毕后是否打开生成代码所在的目录
        globalConfig.setAuthor("wty"); //设置作者
        globalConfig.setFileOverride(true); //设置是否覆盖原始生成的文件
        globalConfig.setMapperName("%sDao"); //设置数据层接口名，%s为占位符，指代模块名称
        globalConfig.setIdType(IdType.ASSIGN_ID); //设置Id生成策略
        autoGenerator.setGlobalConfig(globalConfig);
        //设置包名相关配置
        PackageConfig packageInfo = new PackageConfig();
        packageInfo.setParent("com.aaa"); //设置生成的包名，与代码所在位置不冲突，二者叠加组成完整路径
        packageInfo.setEntity("domain"); //设置实体类包名
        packageInfo.setMapper("dao"); //设置数据层包名
        autoGenerator.setPackageInfo(packageInfo);
        //策略设置
        StrategyConfig strategyConfig = new StrategyConfig();
        strategyConfig.setInclude("tbl_user", "tbl_book"); //设置当前参与生成的表名，参数为可变参数

        strategyConfig.setTablePrefix("tbl_"); //设置数据库表的前缀名称，模块名 = 数据库表名 - 前缀名 例如： User = tbl_user - tbl_
        strategyConfig.setRestControllerStyle(true); //设置是否启用Rest风格
        strategyConfig.setVersionFieldName("version"); //设置乐观锁字段名
        strategyConfig.setLogicDeleteFieldName("deleted"); //设置逻辑删除字段名
        strategyConfig.setEntityLombokModel(true); //设置是否启用lombok
        autoGenerator.setStrategy(strategyConfig);
        //2.执行生成操作
        autoGenerator.execute();
    }
}

```
