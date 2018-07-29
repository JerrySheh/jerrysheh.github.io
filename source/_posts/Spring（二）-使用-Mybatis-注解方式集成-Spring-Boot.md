---
title: Spring（二） 使用 Mybatis 注解方式集成 Spring Boot
comments: true
abbrlink: 72ef7508
date: 2018-04-17 23:48:59
categories: Java Web
tags:
 - Java
 - Web
 - SQL
---

![mybatis](http://www.mybatis.org/images/mybatis-logo.png)

# 使用注解替代xml

在 [使用 Mybatis 简化 JDBC 操作](../post/f251f1b.html) 中，通过配置 xml 的方式来进行增删查改，其实，在Spring Boot中推荐使用注解的方式使用 Mybatis，更加方便。

---

# Spring 集成简明过程

## 提供接口

在 Spring 项目中，写一个接口，其实就是把 xml 搬到这个接口上来。

com.jerrysheh.mapper.CategoryMapper.java
```java
package com.jerrysheh.mapper;

import java.util.List;

import org.apache.ibatis.annotations.Delete;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.annotations.Update;
import com.jerrysheh.pojo.Category;

@Repository
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

> 在 Spring 集成 Mybatis 的时候，一般把这个接口放在 DAO 包（或者是 Mapper包），在接口前面使用 `@Mapper` 注解，之后这个接口在编译时会生成相应的实现类。需要注意的是：这个接口中不可以定义同名的方法，因为会生成相同的id，也就是说这个接口是不支持重载的。

如果传入多个参数，需要使用 `@Param` 注解。

```java
// 修改密码
@Update("update user set password=#{password} WHERE id=#{id}")
void updatePassword(@Param("password") String password, @Param("id") Integer id);

```

## 提供配置文件 mybatis-config.xml

> Spring Boot无需此步骤

mybatis-config.xml 的 `<mappers>` 字段，添加注解类

```xml
<mappers>
    <mapper resource="com/how2java/pojo/Category.xml"/>
    <mapper class="com.jerrysheh.mapper.CategoryMapper"/>
</mappers>
```

## 提供测试类Test.java

```java
//实例化一个 mapper
CategoryMapper categoryMapper = session.getMapper(CategoryMapper.class);

//获取 id 号为8的对象
Category c = categoryMapper.get(8);
System.out.println(c.getName());
```

- mapper对象有add、get、update、delete等方法。

### xml VS 注解 测试类写法的不同

```java
// xml 配置，是用 session.方法 的方式，传入 xml 的 id 和 对象
session.insert("addCategory",c);

// 注解方式，是先通过 session 获取一个 mapeer
// 然后通过 mapeer 来操作
m = session.getMapper(class);
m.get(8);

```
---

# Spring Boot 集成 Mybatis 简明过程

 - 创建一个 Spring Initalizr 工程，依赖选择 web、MySQL、Mybatis
 - 在application.properties填入以下内容

 ```
 spring.datasource.url=jdbc:mysql://127.0.0.1:3306/neu?characterEncoding=UTF-8&useSSL=false
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

 > 在SpringBootApplication类中，添加`@MapperScan("io.jerrysheh.student.mapper")`注解，即可不用在 mapper 包下面的每一个接口都注解Mapper了。

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

---

以下为Mybatis知识点

---

# 动态SQL

虽然我们使用了注解，但是还是在注解接口CategoryMapper中使用了原生 SQL 语句。

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

# @Results结果映射

如果 javabean 的属性字段 跟 数据库字段一一对应，名字保持一致，则直接可以：

```java
@Select("select *from Demo where id=#{id}")  
public Demo selectById(int id);  
```

但如果不对应，就要用`@Result`修饰返回的结果集，而`@Results`注解将指定的数据库列与指定JavaBean属性映射起来。

```java
@Select("SELECT * FROM `wx_message_config` WHERE `content_key_words` IS NOT NULL AND LENGTH(content_key_words) > 0")
@Results({
        @Result(property = "msgType", column = "msg_type"),
        @Result(property = "eventType", column = "event_type"),
        @Result(property = "eventKey",column = "event_key"),
        @Result(property = "contentKeyWords",column = "content_key_words")
})
List<WxMessageConfig> queryAllKeyWords();

@Select("SELECT * FROM `wx_message_config` WHERE `id` = #{id}")
@Results({
        @Result(property = "msgType", column = "msg_type"),
        @Result(property = "eventType", column = "event_type"),
        @Result(property = "eventKey",column = "event_key"),
        @Result(property = "contentKeyWords",column = "content_key_words")
})
WxMessageConfig queryKwById(int id);
```

这样会导致写很多重复内容，可以用 `@ResultMap(“id”) `

```java
@Select("SELECT id, name, password FROM user WHERE id = #{id}")
@Results(id = "userMap", value = { @Result(column = "id", property = "id", javaType = Integer.class),
        @Result(column = "name", property = "name", javaType = String.class),
        @Result(column = "password", property = "password", javaType = String.class) })
User findById(Integer id);

@Select("SELECT * FROM user")
@ResultMap("userMap")
List<User> fingAll();
```

---

# 一对多查询

假设有一张 商品表 和 一张 图片表， 一个商品对应多张图片

![](../../../../images/Webapp/mybatis2.png)

![](../../../../images/Webapp/mybatis3.png)

那么如何取出一个商品，包含商品的所有属性，以及对应的所有图片呢？

## 实体类

商品的所有字段，同时，要添加一个 `List<String>` 表示多张图片的集合

```java
public class Product {
    private Integer id;
    private Integer user_id;
    private String name;
    private String price;
    private Date gmt_create;
    private String description;
    private Integer cate_id;
    private Integer number;

    // 关键！
    private List<String> link;

    // 省略 getter setter
}
```

图片的所有属性，用 String 表示图片地址

```java
public class Image {
    private Integer id;
    private Integer product_id;
    private String link;
}
```

## Mapper接口

在图片的Mapper接口中，根据商品id找到对应的所有图片

```java
// 根据商品id找图片
@Select("SELECT link from image WHERE product_id = #{product_id}")
List<String> getImageLinksByProductId(Integer product_id);
```

在商品的Mapper接口中，通过 `@Results` 和 `@Many` 来进行关联

```java
// 取出在售的所有商品，最新的排前面
@Select("select * from product WHERE number > 0 ORDER BY id DESC")
@Results({
        // 这里要对id进行限定，否则 id 会为 null
        @Result(property = "id", column = "id"),

        // 将 image 的 link 和 product 的 id 绑定，通过 @Many 查询 返回 List
        @Result(property = "link", column = "id", javaType = List.class, many = @Many(select = "com.zsc.tradeplatform.mapper.ImageMapper.getImageLinksByProductId")),
})
List<Product> getAll();
```

## 控制器

```java
@ResponseBody
@GetMapping("/api/product")
    public List<Product> getAllProduct(){
        return productService.getAllProduct();
    }
```

访问 `127.0.0.1:8080/api/product` 查看结果

![](../../../../images/Webapp/mybatisOneToManyResult.png)
