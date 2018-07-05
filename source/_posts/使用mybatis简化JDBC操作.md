---
title: 使用mybatis简化JDBC操作
comments: true
abbrlink: f251f1b
date: 2018-04-16 21:58:52
categories: Java Web
tags:
 - Java
 - Web
 - SQL
---

在 [Java简明笔记（十三）JDBC](../post/f07211ef.html) 中，使用 JDBC 来操作数据库，并把查询到的数据库信息进行 java 对象的映射（ORM），但是 JDBC 的使用是十分繁琐的，除了需要自己写SQL之外，还必须操作Connection, Statment, ResultSet，显得繁琐和枯燥。

于是我们对 JDBC 进行封装，以简化数据库操作。mybatis就是这样的一个框架。

以下简介摘自[官方文档](http://www.mybatis.org/mybatis-3/zh/index.html)：

> MyBatis是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和 Java 的 POJOs(Plain Old Java Objects,普通的 Java对象)映射成数据库中的记录。

这一篇，就通过一个实战，了解一下 mybatis 的使用。

<!--more-->

---

# IDEA 实战

## 创建数据库和表

使用 MYSQL ：
- 创建数据库，库名: myball
- 创建表，表名：category_
- 表分为 id 列 和 name 列， name 列填充category1 和 category2

![SQL](../../../../images/Webapp/mybatis1.png)

## 新建工程

使用 IDEA 新建一个 maven 工程，在 pom.xml 中写入依赖

pom.xml
```xml
<dependencies>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.4.6</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.9-rc</version>
    </dependency>
</dependencies>
```

- mybatis依赖从[官方文档](http://www.mybatis.org/mybatis-3/zh/index.html)中找，版本信息从 [github](https://github.com/mybatis/mybatis-3/releases) 找
- mysql依赖从 [mvn 仓库](http://mvnrepository.com/artifact/mysql/mysql-connector-java/8.0.9-rc)找

## 准备实体类 Category

这个类用来映射数据库信息为java对象

（表category_ -> java的category对象）

注意：这个对象要和数据库信息对应上，比如表category_有id和name，这个category就要有 `int id` 和 `String name`。

src/main/java/com.jerrysheh.pojo/Category.java
```java
package com.jerrysheh.pojo;

public class Category {
    private Long id;
    private String name;
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
}

```

## 创建配置文件 mybatis-config.xml

在 src/main/java 目录下 创建mybatis-config.xml,填入以下内容：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <typeAliases>
        <package name="com.jerrysheh.pojo"/>
    </typeAliases>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/myball?characterEncoding=UTF-8&amp;serverTimezone=GMT%2B8&amp;useSSL=false"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="com/jerrysheh/pojo/Category.xml"/>
    </mappers>
</configuration>
```

- `<dataSource>`主要提供连接数据库用的驱动，数据库名称，编码方式，账号密码以及别名
- `<typeAliases>` 写明包后，就会自动扫描这个包下面的类型
- `<mappers>`是映射

## 创建配置文件 Category.xml

在包 com.jerrysheh.pojo 下创建 Category.xml (如果不想配置烦人的 xml，可以直接看下面的注解方式)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.jerrysheh.pojo">
    <select id="listCategory" resultType="Category">
            select * from category_
        </select>
</mapper>
```

- `namespace`指明哪个包
- `resultType`指映射出来的java对象类型，因为在上一个配置文件已经在`<typeAliases>`写明包名，所以这里不用给出全名（com.jerrysheh.pojo.Category）

## 测试类 Test

Test.java
```java
package com.jerrysheh.pojo;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;

public class Test {
    public static void main(String[] args) throws IOException {
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession session = sqlSessionFactory.openSession();

        List<Category> cs = session.selectList("listCategory");
        for (Category c : cs) {
            System.out.println(c.getName());
        }

    }
}
```

事实上，mybatis做了以下几件事:
1. 应用程序找Mybatis要数据
2. Mybatis从数据库中找来数据(通过mybatis-config.xml 定位哪个数据库，通过Category.xml执行对应的select语句)
3. 基于Category.xml把返回的数据库记录封装在Category对象中
4. 把多个Category对象装在一个Category集合中
5. 返回一个Category集合

---

# 使用 mybatis 增删查改

修改 Category.xml 文件，添加增删查改的SQL语句(如果不想配置烦人的 xml，可以直接看下面的注解方式)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.jerrysheh.pojo">
    <insert id="addCategory" parameterType="Category" >
        insert into category_ ( name ) values (#{name})
    </insert>

    <delete id="deleteCategory" parameterType="Category" >
        delete from category_ where id= #{id}
    </delete>

    <select id="getCategory" parameterType="_int" resultType="Category">
        select * from   category_  where id= #{id}
    </select>

    <update id="updateCategory" parameterType="Category" >
        update category_ set name=#{name} where id=#{id}
    </update>
    <select id="listCategory" resultType="Category">
        select * from   category_
    </select>
</mapper>
```

## 增

在测试类Test中通过session将对象映射为数据库信息，插入表

```java
// 这四句是固定写法
String resource = "mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
SqlSession session = sqlSessionFactory.openSession();

// 实例化一个Category对象
Category c = new Category();
c.setName("category3");

//通过session，将该对象映射为数据库信息，插入到表中
//第一个参数是 Category.xml 中的 id
session.insert("addCategory",c);
```

## 删

删除 id 号为 2 的category

```java
// 实例化一个Category对象
Category c = new Category();
c.setId(2);

//通过session，将该对象映射为数据库信息，从表中删除
session.delete("deleteCategory", c);
```

## 改

```java
// 通过session.selectOne取出一个Category对象
// 第二个参数是 Category.xml 的  getCategory 行的 parameterType，这里对应 int ，也就是 id 号
Category c= session.selectOne("getCategory",3);

//修改对象
c.setName("修改了的Category名稱");

//通过session，将该对象映射为数据库信息，从表中更新
session.update("updateCategory",c);
```

## 查

```java
listAll(session);

// 提交和关闭 session
session.commit();
session.close();

private static void listAll(SqlSession session) {
    List<Category> cs = session.selectList("listCategory");
    for (Category c : cs) {
        System.out.println(c.getName());
    }
}
```

---

# 模糊查询

在 Category.xml 中添加模糊查询语句：

```xml
<select id="listCategoryByName"  parameterType="string" resultType="Category">
    select * from   category_  where name like concat('%',#{0},'%')
</select>
```

> concat是一个 SQL 函数，表示字符串连接。在这个例子中，concat('%',#{0},'%') 表示 [零个或多个字符] + 参数{0} + [零个或多个字符]， 比如参数{0}是 cat，可以匹配 hellocatWTF

在 测试类 Test.java 中

```java
listbyName(session,"gory");

private static void listbyName(SqlSession session, String param) {
    List<Category> cs = session.selectList("listCategoryByName", param);
    for (Category c : cs) {
        System.out.println(c.getName());
    }
}
```

- `session.selectList()`提供第二个参数，这个参数就是Category.xml 中的 param `#{0}`

---

# 多条件查询

在 Category.xml 中添加多条件查询语句：

查询 id 号大于某个参数的

```xml
<select id="listCategoryByIdAndName"  parameterType="map" resultType="Category">
    select * from   category_  where id> #{id}  and name like concat('%',#{name},'%')
</select>
```

因为是多个参数，而selectList方法又只接受一个参数对象，所以需要把多个参数放在Map里，然后把这个Map对象作为参数传递进去

```java
Map<String,Object> params = new HashMap<>();
// id号大于 4 的
params.put("id", 4);
// name 包含 cat的
params.put("name", "cat");
listbyIdAndName(session,params);

private static void listbyIdAndName(SqlSession session, Map<String,Object> param) {
    List<Category> cs = session.selectList("listCategoryByIdAndName", param);
    for (Category c : cs) {
        System.out.println(c.getName());
    }
}
```

---

# 动态SQL

## if

我们前面提供了 listCategory 全部查询 和 listCategoryByName 模糊查询 两种方式。要写两个 SQL 语句。可以看到 `session.selectList()` 既能接受一个参数，也能接受两个参数。

那么可不可以，只写一个 SQL 语句， 当`session.selectList()`只有一个参数的时候，进行全部查询，提供第二个参数的时候，提供模糊查询呢？答案是肯定的。

假如我们要查询的表是 Product_ 表，对应的java对象类为 Product 类。

Product.xml 修改前：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.how2java.pojo">
        <select id="listProduct" resultType="Product">
            select * from product_         
        </select>
        <select id="listProductByName" resultType="Product">
            select * from product_  where name like concat('%',#{name},'%')        
        </select>
    </mapper>
```

Product.xml 修改后：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.how2java.pojo">
        <select id="listProduct" resultType="Product">
            select * from product_
            <if test="name!=null">
                where name like concat('%',#{name},'%')
            </if>        
        </select>
    </mapper>
```

- 关键点：`select * from product_
<if test="name!=null">
    where name like concat('%',#{name},'%')`

## 其他动态SQL语句

除了 if 以外，还有 where、choose、foreach、bind等动态SQL语句，具体用法可以到网上查找。

---

# 注解

上面是通过配置 xml 的方式来进行增删查改，其实，还可以用注解的方式。

## 接口

提供一个接口，其实就是把 xml 搬到这个接口上来。

com.jerrysheh.mapper.CategoryMapper.java
```java
package com.jerrysheh.mapper;

import java.util.List;

import org.apache.ibatis.annotations.Delete;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.annotations.Update;
import com.jerrysheh.pojo.Category;

public interface CategoryMapper {

    @Insert(" insert into category_ ( name ) values (#{name}) ")
    public int add(Category category);

    @Delete(" delete from category_ where id= #{id} ")
    public void delete(int id);

    @Select("select * from category_ where id= #{id} ")
    public Category get(int id);

    @Update("update category_ set name=#{name} where id=#{id} ")
    public int update(Category category);  

    @Select(" select * from category_ ")
    public List<Category> list();
}
```

> 在SpringBoot集成Mybatis的时候，一般把这个接口放在 DAO 包， 在接口前面使用 `@Mapper` 注解，之后这个接口在编译时会生成相应的实现类。需要注意的是：这个接口中不可以定义同名的方法，因为会生成相同的id，也就是说这个接口是不支持重载的

## 配置文件 mybatis-config.xml

mybatis-config.xml 的 `<mappers>` 字段，添加注解类

```xml
<mappers>
    <mapper resource="com/how2java/pojo/Category.xml"/>
    <mapper class="com.jerrysheh.mapper.CategoryMapper"/>
</mappers>
```

## 测试类Test.java

```java
//实例化一个 mapper
CategoryMapper categoryMapper = session.getMapper(CategoryMapper.class);

//获取 id 号为8的对象
Category c = categoryMapper.get(8);
System.out.println(c.getName());
```

- mapper对象有add、get、update、delete等方法。

## xml VS 注解

```java
// xml 配置，是用 session.方法 的方式，传入 xml 的 id 和 对象
session.insert("addCategory",c);

// 注解方式，是先通过 session 获取一个 mapeer
// 然后通过 mapeer 来操作
m = session.getMapper(class);
m.get(8);

```
---

# 动态SQL

虽然我们使用了注解，但是还是在注解接口CategoryMapper中使用了原生 SQL 语句


```java
@Insert(" insert into category_ ( name ) values (#{name}) ")
public int add(Category category);
```

其实，我们可以提供一个类，专门用来生成SQL语句

CategoryDynaSqlProvider.java

```java
package com.jerrysheh.mapper;
import org.apache.ibatis.jdbc.SQL;

public class CategoryDynaSqlProvider {
    public String list() {
        return new SQL()
                .SELECT("*")
                .FROM("category_")
                .toString();

    }
    public String get() {
        return new SQL()
                .SELECT("*")
                .FROM("category_")
                .WHERE("id=#{id}")
                .toString();
    }

    public String add(){
        return new SQL()
                .INSERT_INTO("category_")
                .VALUES("name", "#{name}")
                .toString();
    }
    public String update(){
        return new SQL()
                .UPDATE("category_")
                .SET("name=#{name}")
                .WHERE("id=#{id}")
                .toString();
    }
    public String delete(){
        return new SQL()
                .DELETE_FROM("category_")
                .WHERE("id=#{id}")
                .toString();
    }
}
```

然后修改我们的CategoryMapper接口

```java
package com.jerrysheh.mapper;

import com.jerrysheh.pojo.Category;
import org.apache.ibatis.annotations.*;

import java.util.List;

public interface CategoryMapper {

    @InsertProvider(type=CategoryDynaSqlProvider.class,method="add")
    public int add(Category category);

    @DeleteProvider(type=CategoryDynaSqlProvider.class,method="delete")
    public void delete(int id);

    @SelectProvider(type=CategoryDynaSqlProvider.class,method="get")
    public Category get(int id);

    @UpdateProvider(type=CategoryDynaSqlProvider.class,method="update")
    public int update(Category category);

    @SelectProvider(type=CategoryDynaSqlProvider.class,method="list")
    public List<Category> list();
}
```

这样就可以动态生成SQL语句了

- 注解中的 `type=` 填入我们的动态生成SQL类CategoryDynaSqlProvider.class
- `method=`填入CategoryDynaSqlProvider类里的方法

---

未完待续（一对多查询、相关概念）

---

# Spring boot 集成 Mybatis 简明过程

 - 创建一个 Spring Initalizr 工程，依赖选择 web、MySQL、Mybatis
 - 在application.properties填入以下内容

 ```
 spring.datasource.url=jdbc:mysql://127.0.0.1:3306/neu?characterEncoding=UT F-8&useSSL=false
 spring.datasource.username=root
 spring.datasource.password=YourPassword
 spring.datasource.driver-class-name=com.mysql.jdbc.Driver
 ```

- 创建pojo包，创建Student实体类，跟数据库对应

```java
public class Student {
    Integer id;
    String name;
    String major;
    Integer grade;

    // 省略 getter setter
}
```

- 创建mapper包，创建StudentMapper接口

```java
@Mapper
public interface StudentMapper {

    @Select("SELECT * FROM student")
    List<Student> findAll();
}
```

- 创建Controller包，创建StudentController

```java
@RestController
public class StudentController {

    @Autowired
    StudentMapper studentMapper;

    @GetMapping("/listStudent")
    public void listStudent(){
        List<Student> studentList = studentMapper.findAll();
        for (Student student:
             studentList) {
            System.out.println(student.getName());
        }
    }
}
```

这样，运行后访问 `127.0.0.1:8080/listStudent` ，可看到控制台输出数据库查到的所有 student 名字。
