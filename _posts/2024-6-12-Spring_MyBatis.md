---
layout: post
title: "Spring_MyBatis"
date: 2024-6-12
tags: [SSM]
comments: true
author: zxy
---

> mybatis 篇
> 项目地址：https://gitee.com/zou-xinyu6/ssm-study/tree/master/mybatis_study

## 全面了解 MyBatis

### 什么是 MyBatis

- Myba 是一款优秀的**持久层框架**
- MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集的过程，减少了代码的冗余，减少程序员的操作。
- MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和 Java 的 实体类 【Plain Old Java Objects,普通的 Java 对象】映射成数据库中的记录。

### 持久化

持久化是将程序数据在**持久状态**和**瞬时状态**间转换的机制。（理解一下这个持久状态转化为瞬时状态，持久化是 mybatis 最重要的特性）

- 即把数据（如内存中的对象）保存到可永久保存的存储设备中（如磁盘）。持久化的主要应用是将内存中的对象存储在数据库中，或者存储在磁盘文件中、XML 数据文件中等等。
- JDBC 就是一种持久化机制。文件 IO 也是一种持久化机制。

> 为什么需要持久化服务

- 内存断点后数据会丢失，但是有些业务不允许这种情况的存在
- 比起硬盘，内存过于昂贵，如果有够量的内存，则不需要持久化服务，但是正是因为内存太贵，储存有限，因此需要持久化来缓存

### 持久层

- 持久层，顾名思义是**完成持久化工作的代码块**，也就是 Date Access Object（Dao 层）
- 大多数情况下特别是企业级应用，数据持久化往往也就意味着将内存中的数据保存到磁盘上加以固化，而持久化的实现过程则大多通过各种**关系数据库**来完成。
- 不过这里有一个字需要特别强调，也就是所谓的“层”。对于应用系统而言，数据持久功能大多是必不可少的组成部分。也就是说，我们的系统中，已经天然的具备了“持久层”概念？也许是，但也许实际情况并非如此。之所以要独立出一个“持久层”的概念,而不是“持久模块”，“持久单元”，也就意味着，我们的系统架构中，应该有一个相对独立的逻辑层面，专注于数据持久化逻辑的实现.
- 与系统其他部分相对而言，这个层面应该具有一个较为清晰和严格的逻辑边界。（**也就是该层就是为了操作数据库而设计的**）

### MyBatis 的作用

- Mybatis 就是帮助程序员将**数据存取**到数据库里面。
- 传统的 jdbc 操作 , 有很多**重复代码块** .比如 : 数据取出时的封装 , 数据库的建立连接等等… , 通过框架可以减少重复代码,提高开发效率 .
- MyBatis 是一个半自动化的**ORM 框架 (Object Relationship Mapping) -->对象关系映射**
- 所有的事情，不用 Mybatis 依旧可以做到，只是用了它，会更加方便更加简单，开发更快速。

> 那么 mybatis 有什么优点吗？

- 简单易学：本身就很小且简单。没有任何第三方依赖，最简单安装只要两个 jar 文件+配置几个 sql 映射文件就可以了，易于学习，易于使用，通过文档和源代码，可以比较完全的掌握它的设计思路和实现。
- 灵活：mybatis 不会对应用程序或者数据库的现有设计强加任何影响。sql 写在 xml 里，便于统一管理和优化。通过 sql 语句可以满足操作数据库的所有需求。
- 解除 sql 与程序代码的耦合：通过提供 DAO 层，将业务逻辑和数据访问逻辑分离，使系统的设计更清晰，更易维护，更易单元测试。sql 和代码的分离，提高了可维护性。
- 提供 xml 标签，支持编写动态 sql。
- 现在主流使用方法

## 第一个 MyBatis 程序

**新建 maven 项目**

![在这里插入图片描述](https://img-blog.csdnimg.cn/cab9595610234a8cb6880b0a7557c30a.png#pic_center)

**搭建数据库**

```
CREATE DATABASE `mybatis`;

USE `mybatis`;

DROP TABLE IF EXISTS `user`;

CREATE TABLE `user` (
`id` int(20) NOT NULL,
`name` varchar(30) DEFAULT NULL,
`pwd` varchar(30) DEFAULT NULL,
PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

insert  into `user`(`id`,`name`,`pwd`) values (1,'阿渌','123456'),(2,'张三','abcdef'),(3,'李四','987654');

```

**导入 MyBatis 相关 jar 包，导入到 pom.xml 文件里面**

```xml
    <dependencies>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.26</version>
        </dependency>
        <!--mybatis-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.2</version>
        </dependency>
        <!--junit-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
    </dependencies>
    <build>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
        </resources>
    </build>

```

**编写 Mybatis 核心配置文件，在 resources 下创建 mybatis-config.xml 文件，配置自己的数据库地址、名字、密码以及 mysql 驱动**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!--configuration核心配置文件-->
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSl=trur&amp;sueUnicode=true&amp;characterEncoding=UTF-8&amp;serverTimezone=Asia/Shanghai"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
     <mappers>
        <mapper resource="com/daimalu/dao/UserMapper.xml"/>
    </mappers>
</configuration>

```

**项目目录文件格式**

![在这里插入图片描述](https://img-blog.csdnimg.cn/eaa8cc2d526e41d1afe1375b059ffee3.png#pic_center)

```java
编写MyBatis工具类 在文件夹utils下新建–>MybatisUtils.class文件,编写代码如下
package com.daimalu.utils;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import java.io.IOException;
import java.io.InputStream;

//sqlSessionFactory -->sqlSession
public class MybatisUtils {

    private static SqlSessionFactory sqlSessionFactory;
    //使用mybatis第一步，获取sqlSessionFactory对象
    static {
        try {
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    //既然有了sqlSessionFactory,顾名思义，我们就可以从中获得sqlSession的实例了
    //sqlSession 完全包含了面向数据库执行SQL命令所需的所有方法
    public static SqlSession getSqlSession(){
        return sqlSessionFactory.openSession();
    }

}

创建实体类，在文件夹pojo下创建 User.class文件,编写代码如下
package com.daimalu.pojo;

public class User {

    private int id;  //id
    private String name;   //姓名
    private String pwd;   //密码

    public User(){

    }
    public User(int id, String name, String pwd) {
        this.id = id;
        this.name = name;
        this.pwd = pwd;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPwd() {
        return pwd;
    }

    public void setPwd(String pwd) {
        this.pwd = pwd;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", pwd='" + pwd + '\'' +
                '}';
    }
}

编写Mapper接口类，在文件夹dao下创建 UserDao.interface文件,编写代码如下
package com.daimalu.dao;

import com.daimalu.pojo.User;

import java.util.List;

public interface UserDao {
    List<User> getUserList();
}

编写Mapper.xml配置文件，在文件夹dao下创建 UserMapper.xml文件,编写代码如下
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namespace=绑定一个对应的Dao/Mapper接口-->
<mapper namespace="com.daimalu.dao.UserDao">
    <!--select查询语句-->
    <select id="getUserList" resultType="com.daimalu.pojo.User">
        select * from mybatis.user
    </select>
</mapper>

```

此时记得要将刚才的 UserMapper.xml 注册到 mybatis-config.xml 文件中，**非常重要，此后每编写一个 Mapper.xm 文件都要注册一次！！！**

```java
编写测试类，测试代码。在test–>java–>com–>daimalu–>dao下创建UserDaoTest.class文件，编写代码如下
package com.daimalu.dao;

import com.daimalu.pojo.User;
import com.daimalu.utils.MybatisUtils;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

import java.util.List;

public class UserDaoTest {
    //第一步：获得sqlSession对象
    SqlSession sqlSession;
    @Test
    public void test(){
        try {

            sqlSession = MybatisUtils.getSqlSession();
            //方式一：getMapper
            UserDao mapper = sqlSession.getMapper(UserDao.class);
            List<User> userList = mapper.getUserList();
            for (User user : userList) {
                System.out.println(user);
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {

            //关闭sqlSession
            sqlSession.close();
        }

    }
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/52fd71f0498c4747bdba3f3fb04504c5.png#pic_center)

## 配置解析

> 核心配置文件 mybatis-config.xml

![在这里插入图片描述](https://img-blog.csdnimg.cn/2e8314bf86ea467f945a383aedbcf350.png#pic_center)

> 环境配置（environments）

Mybatis 可以配置成多种环境，但是**每个 sqlSessionFactory 实例只能选择一种环境**如图，有环境 development 和 test，默认是 development 环境
![在这里插入图片描述](https://img-blog.csdnimg.cn/89037bbff0d3497db6b4b16b4c9c45f9.png#pic_center)
mybatis 默认的事务管理器就是 JDBC（事务管理器有多种），连接池：POOLED

##### 属性（properties）

可以通过 properties 属性来实现引用配置文件
这些属性可以在外部进行配置，并可以进行动态替换。你既可以在典型的 Java 属性文件中配置这些属性，也可以在 properties 元素的子元素中设置（即 db.properties）
db.properties

```properties
driver=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis?useSSl=trur&amp;sueUnicode=true&amp;characterEncoding=UTF-8&amp;serverTimezone=Asia/Shanghai
username=root
password=123456
```

在核心配置文件中引入
![在这里插入图片描述](https://img-blog.csdnimg.cn/8265ed5bdb12422fad19c35bd57202d4.png#pic_center)
注意：如果一个属性在不只一个地方进行了配置，那么，MyBatis 将按照下面的顺序来加载：

1.首先读取在 properties 元素体内指定的属性。 2.然后根据 properties 元素中的 resource 属性读取类路径下属性文件，或根据 url 属性指定的路径读取属性文件，并覆盖之前读取过的同名属性。 3.最后读取作为方法参数传递的属性，并覆盖之前读取过的同名属性。
因此，通过方法参数传递的属性具有最高优先级，resource/url 属性中指定的配置文件次之，最低优先级的则是 properties 元素中指定的属性。

## 类型别名

- 类型别名可为 Java 类型设置一个缩写名字。 它仅用于 XML 配置
- 意在降低冗余的全限定类名书写

  1.可以给实体类起别名

```xml
    <!--可以给实体类起别名-->
    <typeAliases>
        <typeAlias type="com.daimalu.pojo.User" alias="User"/>
    </typeAliases>
1234
    <select id="getUserList" resultType="User">
        select * from mybatis.user
    </select>
123
```

2.也可以指定一个包名，MyBatis 会在包名下面搜索需要的 Java Bean,每一个在包 com.daimalu.pojo 中的 Java Bean，**在没有注解的情况下，会使用 Bean 的首字母小写的非限定类名来作为它的别名**

```xml
   <typeAliases>
        <package name="com.daimalu.pojo"/>
    </typeAliases>
123
```

总结：在实体类比较少的时候，使用第一种方式，如果实体类很多，建议使用第二种，第一种可以 diy（自定义）别名；第二种则不行，**第二种如果非要改，则需要在实体上增加注解**

```java
@Alias("othername")
public class User{}
```

## 映射器（Mapper）

方式一：

```xml
    <mappers>
        <mapper resource="com/daimalu/dao/UserMapper.xml"/>
    </mappers>
```

方式二：使用 class 文件注册

```xml
    <mappers>
        <mapper class="com.daimalu.dao.UserMapper"/>
    </mappers>
```

注意： 1.接口和他的 Mapper 配置文件必须同名 2.接口和他的 Mapper 配置文件必须在同一个包下
方式三：使用包扫描注册

```xml
    <mappers>
        <package name="com.daimalu.dao"/>
    </mappers>
```

注意： 1.接口和他的 Mapper 配置文件必须同名 2.接口和他的 Mapper 配置文件必须在同一个包下

## 生命周期和作用域

![在这里插入图片描述](https://img-blog.csdnimg.cn/b2eff2462f9f44329d15a6fdf13057d1.png#pic_center)

**SqlSessionFactoryBuilder** 1.这个类可以被实例化、使用和丢弃，一旦创建了 SqlSessionFactory，就不再需要它了 2.局部变量

**SqlSessionFactory** 1.可以想象成连接池
2.SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，**没有任何理由丢弃它或重新创建另一个实例**
3.SqlSessionFactory 的最佳作用域是应用作用域 4.最简单的就是使用单例模式或者静态单例模式

**SqlSession** 1.连接到的一个请求，可以想象成数据库连接池中的一个请求。
2.SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域 3.用完之后需要赶紧关闭，否则资源被占用！（类似于数据库连接池的回收）

![在这里插入图片描述](https://img-blog.csdnimg.cn/b72a263e640745328132fedf0746b0bf.png#pic_center)

## 解决属性名和数据库字段名不一致问题

ResultMap 元素是 MyBatis 中最重要最强大的元素。它可以让你从 90% 的 JDBC ResultSets 数据提取代码中解放出来，并在一些情形下允许你进行一些 JDBC 不支持的操作。实际上，在为一些比如连接的复杂语句编写映射代码的时候，一份 resultMap 能够代替实现同等功能的数千行代码。ResultMap 的设计思想是，对简单的语句做到零配置，对于复杂一点的语句，只需要描述语句之间的关系就行了。

```xml
<resultMap id="userResultMap" type="User">
  <id property="id" column="user_id" />
  <result property="username" column="user_name"/>
  <result property="password" column="hashed_password"/>
</resultMap>
```

然后在引用它的语句中设置 resultMap 属性就行了（注意我们去掉了 resultType 属性）。比如:

```xml
<select id="selectUsers" resultMap="userResultMap">
  select user_id, user_name, hashed_password
  from some_table
  where id = #{id}
</select>
```

## 分页

使用 Limit 实现分页

- Mapper 接口，参数为 map

```java
//全部用户实现分页
List<User> selectUser(Map<String,Integer> map)
```

修改 Mapper 文件

```xml
<select id="selectUser" parameterType="map" resultType="user">
  select * from user limit #{startIndex},#{pageSize}
</select>
```

在测试类中传入参数测试

推断：起始位置 = （当前页面 - 1 ） \* 页面大小

```java
//分页查询 , 两个参数startIndex , pageSize
@Test
public void testSelectUser() {
   SqlSession session = MybatisUtils.getSession();
   UserMapper mapper = session.getMapper(UserMapper.class);

   int currentPage = 1;  //第几页
   int pageSize = 2;  //每页显示几个
   Map<String,Integer> map = new HashMap<String,Integer>();
   map.put("startIndex",(currentPage-1)*pageSize);
   map.put("pageSize",pageSize);

   List<User> users = mapper.selectUser(map);

   for (User user: users){
       System.out.println(user);
  }

   session.close();
}

```

## 动态 SQL

> 什么是动态 SQL

- 动态 SQL 指的是根据不同的查询条件 , 生成不同的 Sql 语句也就是复杂的 SQL 语句。
- 官网描述：
  MyBatis 的**强大特性之一便是它的动态 SQL**。如果你有使用 JDBC 或其它类似框架的经验，你就能体会到根据不同条件拼接 SQL 语句的痛苦。例如拼接时要确保不能忘记添加必要的空格，还要注意去掉列表最后一个列名的逗号。利用动态 SQL 这一特性可以彻底摆脱这种痛苦。

> 开发常用和基础的几个元素

- **where**
  **解释**：where 元素可以自动识别拼接的 SQL 语句，像是 and、or 等，会智能的选择加上或者去除。
  **例**：如下代码，假设不满足第一个 if，满足第二个 if，则拼接的语句应该`select * from blog where and author = #{author}`。很显然多了一个 and，此时 where 元素会自动去掉 and，使得 sql 语句正确。

```sql
    <select id="queryBlogIf" parameterType="map" resultType="blog">
        select * from blog
        <where>
            <if test="title != null">
               title = #{title}
            </if>
            <if test="author != null">
                and author = #{author}
            </if>
        </where>
        </select>
```

- **if**
  **解释**：if 元素可以用来 SQL 语句的条件

  **例如**：根据作者名字和博客名字来查询博客，如果作者名字为空，那么只根据博客名字查询，反之，则根据作者名来查询。**如果用一般 SQL 则需要 3 条 SQL 语句分别实现**
  select _ from blog
  select _ from blog where title = #{title}
  select \* from blog where author = #{author}

  使用动态 SQL 则一条语句便可以**同时满足三种条件**，代码如下

```sql
    <select id="queryBlogIf" parameterType="map" resultType="blog">
        select * from blog
        <where>
            <if test="title != null">
               title = #{title}
            </if>
            <if test="author != null">
                and author = #{author}
            </if>
        </where>
        </select>
```

- **choose、when、otherwise**
  **解释**：有时候，我们不想用到所有的查询条件，只想选择其中的一个，查询条件有一个满足即可，使用 choose 标签可以解决此类问题，**这里其实类似于 java 的 Switch 语句**，用 Switch 来理解会很容易
  **例**：下面代码只会查询到满足一种条件的 sql，等同于三种情况 sql
  select _ from blog where title = #{title}
  select _ from blog where author = #{author}
  select \* from blog where views = #{views}

```sql
    <select id="queryBlogChoose" parameterType="map" resultType="blog">
        select * from blog
        <where>
            <choose>
                <when test="title != null">
                    title = #{title}
                </when>
                <when test="author != null">
                    and author = #{author}
                </when>
                <otherwise>
                    and views = #{views}
                </otherwise>
            </choose>
        </where>
    </select>
```

- **Set**
  **解释**：如果在进行更新操作的时候，含有 set 关键词，我们怎么处理呢？也类似于 where 元素，可以自动识别逗号，动态的保留或者去掉。
  **例**：下面代码若满足第一个 if，不满足第二个 if，则 set 元素会自动去掉逗号，sql 语句变成`update blog title = #{title}`

```sql
    <!--注意set是用的逗号隔开-->
    <update id="updateBlog" parameterType="map">
        update blog
        <set>
            <if test="title != null">
                title = #{title},
            </if>
            <if test="author != null">
                author = #{author}
            </if>
        </set>
        where id = #{id};
    </update>
```

- **Foreach**
  **解释**：Foreach 元素常见使用场景是对集合进行遍历（**尤其是在构建 IN 条件语句的时候**）。
  **例**：我们需要查询 blog 表中 id 分别为 1,2,3 的博客信息。

```sql
    <select id="queryBlogForeach" parameterType="map" resultType="blog">
        select * from blog
        <where>
            <!--
            collection:指定输入对象中的集合属性
            item:每次遍历生成的对象
            open:开始遍历时的拼接字符串
            close:结束时拼接的字符串
            separator:遍历对象之间需要拼接的字符串
          -->
            <foreach collection="ids"  item="id" open="and (" close=")" separator="or">
                id=#{id}
            </foreach>
        </where>
    </select>
```

上面代码等同于 SQL 语句`select * from blog where 1=1 and (id=1 or id=2 or id=3)`。

> SQL 片段

有时候可能某个 sql 语句我们用的特别多，为了增加代码的重用性，简化代码，我们需要将这些代码**抽取出来**，然后使用时直接调用。

提取 SQL 片段：

```xml
<sql id="if-title-author">
   <if test="title != null">
      title = #{title}
   </if>
   <if test="author != null">
      and author = #{author}
   </if>
</sql>
```

引用 SQL 片段：

```xml
<select id="queryBlogIf" parameterType="map" resultType="blog">
  select * from blog
   <where>
       <!-- 引用 sql 片段，如果refid 指定的不在本文件中，那么需要在前面加上 namespace -->
       <include refid="if-title-author"></include>
       <!-- 在这里还可以引用其他的 sql 片段 -->
   </where>
</select>
```

上面代码等同于：

```xml
<select id="queryBlogIf" parameterType="map" resultType="blog">
  select * from blog
   <where>
	   <if test="title != null">
	      title = #{title}
	   </if>
	   <if test="author != null">
	      and author = #{author}
	   </if>
   </where>
</select>
```

**注意：**

- 最好基于 单表来定义 sql 片段，提高片段的可重用性
- 在 sql 片段中不要包括 where

## Spring 整合 MyBatis

> 项目地址：https://gitee.com/zou-xinyu6/ssm-study/tree/master/Spring-MyBatis

> 思路分析

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-NQWA8hJQ-1667870226587)(assets/1630137189480.png)]](https://img-blog.csdnimg.cn/ffd94f73505d4fafa61ef2940c2a01a5.png)

从图中可以获取到，真正需要交给 Spring 管理的是 SqlSessionFactory

整合 Mybatis，就是将 Mybatis 用到的内容交给 Spring 管理，分析下配置文件

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-M8IhJTkk-1667870226587)(assets/1630137388717.png)]](https://img-blog.csdnimg.cn/51059e4dcd004d7a9ed88866fc76df3e.png)

- 第一行读取外部 properties 配置文件，Spring 有提供具体的解决方案`@PropertySource`,需要交给 Spring
- 第二行起别名包扫描，为 SqlSessionFactory 服务的，需要交给 Spring
- 第三行主要用于做连接池，Spring 之前我们已经整合了 Druid 连接池，这块也需要交给 Spring
- 前面三行一起都是为了创建 SqlSession 对象用的，那么用 Spring 管理 SqlSession 对象吗?回忆下 SqlSession 是由 SqlSessionFactory 创建出来的，所以只需要将 SqlSessionFactory 交给 Spring 管理即可。
- 第四行是 Mapper 接口和映射文件[如果使用注解就没有该映射文件]，这个是在获取到 SqlSession 以后执行具体操作的时候用，所以它和 SqlSessionFactory 创建的时机都不在同一个时间，可能需要单独管理。

所以 Spring 整合 mybatis 只需要考虑两件事：

第一件事是:Spring 要管理 MyBatis 中的 SqlSessionFactory

第二件事是:Spring 要管理 Mapper 接口的扫描

> 具体实现

使用 SqlSessionFactoryBean 封装 SqlSessionFactory 需要的环境信息

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-ygNrets3-1667870226588)(assets/1630138835057.png)]](https://img-blog.csdnimg.cn/6f7ae8c0603342f08803627dd4afa2f1.png)

- SqlSessionFactoryBean 是前面我们讲解 FactoryBean 的一个子类，在该类中将 SqlSessionFactory 的创建进行了封装，简化对象的创建，我们只需要将其需要的内容设置即可。
- 方法中有一个参数为 dataSource,当前 Spring 容器中已经创建了 Druid 数据源，类型刚好是 DataSource 类型，此时在初始化 SqlSessionFactoryBean 这个对象的时候，发现需要使用 DataSource 对象，而容器中刚好有这么一个对象，就自动加载了 DruidDataSource 对象。

使用 MapperScannerConfigurer 加载 Dao 接口，创建代理对象保存到 IOC 容器中

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-1CRBVDa2-1667870226589)(assets/1630138916939.png)]](https://img-blog.csdnimg.cn/91818acd024f458a947d09552fea2851.png)

- 这个 MapperScannerConfigurer 对象也是 MyBatis 提供的专用于整合的 jar 包中的类，用来处理原始配置文件中的 mappers 相关配置，加载数据层的 Mapper 接口类
- MapperScannerConfigurer 有一个核心属性 basePackage，就是用来设置所扫描的包路径

## Spring 整合 JUnit

- 单元测试，如果测试的是注解配置类，则使用`@ContextConfiguration(classes = 配置类.class)`
- 单元测试，如果测试的是配置文件，则使用`@ContextConfiguration(locations={配置文件名,...})`
- Junit 运行后是基于 Spring 环境运行的，所以 Spring 提供了一个专用的类运行器，这个务必要设置，这个类运行器就在 Spring 的测试专用包中提供的，导入的坐标就是这个东西`SpringJUnit4ClassRunner`
- 上面两个配置都是固定格式，当需要测试哪个 bean 时，使用自动装配加载对应的对象，下面的工作就和以前做 Junit 单元测试完全一样了

##### @RunWith

| 名称 | @RunWith                          |
| ---- | --------------------------------- |
| 类型 | 测试类注解                        |
| 位置 | 测试类定义上方                    |
| 作用 | 设置 JUnit 运行器                 |
| 属性 | value（默认）：运行所使用的运行期 |

##### @ContextConfiguration

| 名称 | @ContextConfiguration                                                                                                    |
| ---- | ------------------------------------------------------------------------------------------------------------------------ |
| 类型 | 测试类注解                                                                                                               |
| 位置 | 测试类定义上方                                                                                                           |
| 作用 | 设置 JUnit 加载的 Spring 核心配置                                                                                        |
| 属性 | classes：核心配置类，可以使用数组的格式设定加载多个配置类 locations:配置文件，可以使用数组的格式设定加载多个配置文件名称 |
