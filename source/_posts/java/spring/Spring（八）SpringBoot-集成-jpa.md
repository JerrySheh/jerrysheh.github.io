---
title: Spring（八）SpringBoot 集成 JPA
comments: true
categories:
- Java Web
- Spring
tags:
  - Java
  - Web
  - SQL
abbrlink: cf93e9b3
date: 2018-11-11 13:10:53
---

# 什么是 JPA ？

之前在 Spring Boot 工程中，一直用 Mybatis 注解方式作为持久层框架。但是 Mybatis 需要手写 SQL 语句，对于简单的项目稍显麻烦。最近发现了 JPA ，使用 JPA 我们几乎可以不用写一句 SQL 语句，非常适合 CURD 场景。JPA 是 Java Persistence API（Java持久化接口） 的缩写。JPA 让我们的应用程序以统一的方式访问持久层。JPA 是 Hibernate 的一个抽象，是一种 ORM 规范，是可以理解为是 Hibernate 功能的一个子集。


<!-- more -->

---

# Spring Boot 集成 JPA 简要过程

## 1. 添加依赖

新建工程在 Spirng initializr 中勾选 JPA 即可，如果没有勾选，可以在 pom.xml 里面添加 JPA 依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-JPA</artifactId>
</dependency>
```

## 2.新建实体类

这里推荐一个插件 lombok，使用 lombok 我们可以直接用 `@Data` 注解替代 getter、setter 和 toString 方法。之后，在实体类的主键字段添加`@Id`注解和`@GeneratedValue`注解。在 gmt_create 和 gmt_modified 字段添加时间注解，并用`@EntityListeners`监听，这样，我们可以不用在后面的代码中每次都添加修改时间。

在 IDEA 插件中搜索并安装 lombok，之后重启 IDEA，在 maven 中添加：

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

pojo类

```java
@Entity
@Data
@EntityListeners(AuditingEntityListener.class)
public class Message {

    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    long id;

    String message;

    @CreatedDate
    Date gmtCreate;

    @LastModifiedDate
    Date gmtModified;

    // 无需编写 getter setter toString

}
```

几种注解的区别：

1. `@GeneratedValue(strategy = GenerationType.AUTO)`主键增长方式由数据库自动选择。
2. `@GeneratedValue(strategy = GenerationType.IDENTITY)` 要求数据库选择自增方式，oracle不支持此种方式。
3. `@GeneratedValue(strategy = GenerationType.SEQUENCE)` 采用数据库提供的sequence机制生成主键。mysql不支持此种方式。

## 3. 编写 JPA 接口

接口里面什么都不用写，就已经有了 CURD 方法了。<font color="red">注意泛型参数应该为 **实体类** 和 **主键** 的类型</font>。

```java
import io.jerrysheh.message_me.pojo.Message;
import org.springframework.data.repository.CrudRepository;

public interface MessageRepository extends CrudRepository<Message, Long>  { // 注意泛型参数

}

```

## 4. save方法

使用 save 方法来新增或更新一条记录，JPA会判断是否已经有该 id，如果没有则新增，如果有则更新。这里有一个坑，有时候一个对象我们只需要修改其中某一个字段，其他字段不变，但是修改时 JPA 会把不需要变的字段也修改为 null，解决办法是先取出该对象的所有字段，让他们不为null，再修改需要改动的字段。

```java
@Test
public void testAdd(){
    Message msg = new Message();
    msg.setMessage("第一条message");
    messageRepository.save(msg);
}

@Test
public void testUpdate(){
    Optional<Message> opt = messageRepository.findById(4L);
    if (opt.isPresent()){
        Message msg = opt.get();
        msg.setMessage("修改后的心语");
        messageRepository.save(msg);
    }
}
```

## 5.

---

# 问题锦集

## 1. No identifier specified for entity

在 id 字段加 `@Id` 注解 和 `@GeneratedValue(strategy=GenerationType.IDENTITY)` 注解。几种 GeneratedValue 的区别上面已经提及。

## 2. CreatedDate 的时间不对

数据源加上`serverTimezone=GMT%2B8`

---

# 参考

- [Spring官方文档 - Accessing Data with JPA](https://spring.io/guides/gs/accessing-data-JPA/)
