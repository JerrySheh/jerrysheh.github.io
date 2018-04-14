---
title: 从 MVC 到SpirngBoot
comments: true
categories: Java Web
tags:
  - Java
  - Web
abbrlink: 6200df85
date: 2018-04-15 00:19:08
---

在谈[Spring](https://spring.io/) 和 [SpringBoot](https://projects.spring.io/spring-boot/) 之前，先来看看典型的Java Web应用架构以及框架的来源：

![MVC](../../../../images/Webapp/MVC.jpg)

1. Web浏览器发送HTTP请求到服务端，被Controller(Servlet)获取并进行处理（例如参数解析、请求转发）
2. Controller(Servlet)调用核心业务逻辑——Model部分
3. Model进行数据库存取操作，并将操作结果返回给Model
4. Controller(Servlet)将业务逻辑处理结果交给View（JSP），动态输出HTML内容
5. 动态生成的HTML内容返回到浏览器显示

我们可以把 Servlet 中经常要实现的功能封装起来并提供一层公共抽象，这样我们只要编写简单的POJO代码或者实现一些接口，就能完成复杂的Web请求后端逻辑。

<!--more-->

> POJO是Plain Old Java Object的缩写，是软件开发大师Martin Fowler提出的一个概念，指的是一个普通Java类。也就说，你随便编写一个Java类，就可以称之为POJO。之所以要提出这样一个专门的术语，是为了与基于重量级开发框架的代码相区分，比如EJB，我们编写的类一般都要求符合特定编码规范，实现特定接口、继承特定基类，而POJO则可以说是百无禁忌，灵活方便。

`Spring MVC`就是这样的一个框架。它提供了一个DispacherServlet（Spring MVC是以Servlet技术为基础的），我们只需实现Spring MVC提供的接口就可以完成复杂的操作。

同理，我们可以对数据库操作也做一个封装，在 [Java JDBC 编程](https://jerrysheh.github.io/post/f07211ef.html) 中提到可以将对象和关系数据库进行映射，从而把操作数据库变成操作Java对象。这就是大名鼎鼎的 ORM 技术了。

![MVC](../../../../images/Webapp/MVC2.jpg)

而 `Hibernate` 和 `MyBatis` 就是这样的 ORM 框架。

可见，框架是为了方便我们站在更高级的角度去开发而产生的。

---

# Spring

前面提到`Spring MVC`是Java Web开发中对Servlet进行封装的框架。实际上，Spring是一个大家族，它是一个基于IOC和AOP的结构J2EE系统的框架。

其中最主要的包括Spring Framework（包括了IoC, AOP, MVC以及Testing）, Spring Data, Spring Security, Spring Batch等等，以及快速框架Spring Boot。他们都是为了解决特定的事情而产生的。

但是在学习这些框架前，有必要先弄清楚 Spring 最核心的两个概念：`IoC` 和 `AOP`。

## IoC （Inversion Of Control，反转控制）

简单地说，以前我们通过 new 构造方法来创建对象，现在变成交由 Spring 帮我们创建对象。那 Spring 是如何帮我们创建对象的呢？ 事实上，它是通过一种叫 `DI （依赖注入，Dependency Inject）`的技术实现的。简单地说DI可以拿到的对象的属性。

Spring 通过配置 xml 文档或者 @ 注解 的方式帮我们注入对象，我们得到对象后直接就可以使用，而无需手动构造然后调用各种 setter 方法。

IoC 背后的原理，其实就是 [Java 的反射机制](../post/e753fbbb.html)

## AOP（Aspect Oriented Program，面向切面编程）

在面向切面编程的思想里面，把功能分为核心业务功能，和周边功能。

所谓的核心业务，比如登陆，增加数据，删除数据都叫核心业务，所谓的周边功能，比如性能统计，日志，事务管理等等。

周边功能在Spring的面向切面编程AOP思想里，即被定义为切面。

我们可以对核心业务功能和切面功能分别独立进行开发，然后把切面功能和核心业务功能 "编织" 在一起，这就叫AOP。

AOP 的好处是降低程序的耦合性。

---

# SpringBoot

未完待续
