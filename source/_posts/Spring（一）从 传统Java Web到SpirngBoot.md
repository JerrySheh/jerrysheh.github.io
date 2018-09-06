---
title: Spring（一）从 传统Java Web到SpirngBoot
comments: true
categories: Java Web
tags:
  - Java
  - Web
abbrlink: 6200df85
date: 2018-04-15 00:19:08
---

# 从 MVC 结构到框架

## 传统 Model-View-Controller 架构

在谈[Spring](https://spring.io/) 和 [SpringBoot](https://projects.spring.io/spring-boot/) 之前，先来看看典型的Java Web应用架构以及框架的来源：

![MVC](../../../../images/Webapp/MVC.jpg)

1. Web浏览器发送HTTP请求到服务端，被Controller(Servlet)获取并进行处理（例如参数解析、请求转发）
2. Controller 调用核心业务逻辑——Model部分
3. Model进行数据库存取操作，并将操作结果返回
4. Controller 将业务逻辑处理结果交给View（JSP），动态输出HTML内容
5. 动态生成的HTML内容返回到浏览器显示

我们可以把 Servlet 中经常要实现的功能封装起来并提供一层公共抽象，这样我们只要编写简单的POJO代码或者实现一些接口，就能完成复杂的Web请求后端逻辑。

<!--more-->

> POJO是Plain Old Java Object的缩写，是软件开发大师Martin Fowler提出的一个概念，指的是一个普通Java类。也就说，你随便编写一个Java类，就可以称之为POJO。之所以要提出这样一个专门的术语，是为了与基于重量级开发框架的代码相区分，比如EJB，我们编写的类一般都要求符合特定编码规范，实现特定接口、继承特定基类，而POJO则可以说是百无禁忌，灵活方便。

`Spring MVC`就是这样的一个框架。它提供了一个DispacherServlet（Spring MVC是以Servlet技术为基础的），我们只需实现Spring MVC提供的接口就可以完成复杂的操作。

同理，我们可以对数据库操作也做一个封装，在 [Java JDBC 编程](https://jerrysheh.github.io/post/f07211ef.html) 中提到可以将对象和关系数据库进行映射，从而把操作数据库变成操作Java对象。这就是大名鼎鼎的 ORM 技术了。

![MVC](../../../../images/Webapp/MVC2.jpg)

`Hibernate` 和 `MyBatis` 就是这样的 ORM 框架。

---

# Spring Framework

![Spring](https://spring.io/img/homepage/icon-spring-framework.svg)

## 简介

前面提到`Spring MVC`是Java Web开发中对Servlet进行封装的框架。实际上，Spring是一个大家族，它是一个轻量级的 DI / IoC 和 AOP 容器的开源框架，来源于 Rod Johnson 在其著作《Expert one on one J2EE design and development》中阐述的部分理念和原型。

Spring Framework包括以下几大部分：

![Framework](../../../../images/Webapp/SpringFramework.png)

- **Data Access/Integration** : 包含有JDBC、ORM、OXM、JMS和Transaction模块。
- **Web**：包含了Web、Web-Servlet、WebSocket、Web-Porlet模块。
- **AOP模块**：提供了一个符合AOP联盟标准的面向切面编程的实现。
- **Core Container(核心容器)**：包含有Beans、Core、Context和SpEL模块。
- **Test模块**：支持使用JUnit和TestNG对Spring组件进行测试。

但是在学习这些前，有必要先弄清楚 Spring 最核心的两个概念：`IoC` 和 `AOP`。

## IoC （Inversion of Control，反转控制）

在设计一辆车的时候，如果我们先设计轮子、再根据轮子设计底盘、再根据底盘设计车身，这是一种自底向上的设计思想。但是，假若未来某一天需要改造一下轮子（比如由直径30cm改成40cm），那么底盘、车身不得不相应地进行改动。在大型的软件工程中，这种做法几乎是不可维护的，因为一个类可能作为其他上百个类的底层，我们不可能一一去修改。

对于软件来说，某一接口的具体实现类的选择控制权从调用类中移除，转交由第三方决定。

### 依赖注入

`反转控制`其实是一种依赖倒置原则的设计思想。也就是反过来，让底层依赖上层。具体的做法就是使用 `DI （依赖注入，Dependency Inject）`，DI把底层类作为参数传给上层，实现上层对下层的控制。

使用DI的一个好处是，让互相协作的软件组件保持松耦合。

### 反转控制容器

采用依赖倒置原则的设计之后，会产生一个问题，假设我们要调用一个上层，由于上层需要接受下层作为参数，我们必须在构造上层前构造下层，这样我们的代码中就会写很多 new 。

这时候`反转控制容器（IoC Container）`出现了，这个容器可以自动对我们的代码进行初始化，而我们要做的，只是维护一个 Configuration， 具体到 Spring 中，我们可以通过 xml 配置、 @ 注解 或 Java配置（推荐）的方式让 Spring 帮我们注入对象(Spring 容器通过 `bean 工厂` 和 `应用上下文` 两种类型来实现)，我们得到对象后直接就可以使用，而不需要了解注入过程层层依赖的细节。

这样，调用类对接口实现类的依赖关系变成了由第三方（容器或协作类）注入。注入包括构造函数注入、Setter注入、接口注入。

简单地说，就是当我们要使用某个对象，只需要从 Spring 容器中获取需要使用的对象，不关心对象的创建过程，把创建对象的控制权反转给了Spring框架，而 Spring 容器是通过 DI，在创建对象的过程中将对象依赖属性（简单值，集合，对象）通过配置设值给该对象。

![IoC](../../../../images/Webapp/SpringIOC.png)

### IoC 是如何实现的

如果我们自己来实现这个依赖注入的功能，我们怎么来做？无外乎：

1. 读取标注或者配置文件，看看 bean 依赖的是哪个Source，拿到类名
2. 使用反射的API，基于类名实例化对应的对象实例
3. 将对象实例，通过构造函数或者 setter，传递给 bean

我们发现其实自己来实现也不是很难，Spring实际也就是这么做的。这么看的话其实IoC就是一个工厂模式的升级版！当然要做一个成熟的IoC框架，还是非常多细致的工作要做，Spring不仅提供了一个已经成为业界标准的Java IoC框架，还提供了更多强大的功能。

参考：
- [知乎:Spring IoC有什么好处呢？
](https://www.zhihu.com/question/23277575/answer/169698662)
- [Spring学习(1)——快速入门](https://www.cnblogs.com/wmyskxz/p/8820371.html)
- IoC 背后的Java原理，其实就是 [Java 的反射机制](../post/e753fbbb.html)

## AOP（Aspect Oriented Program，面向切面编程）

在面向切面编程的思想里面，把功能分为`核心业务功能`和`周边功能`。

所谓核心业务，包括登录，增加数据，删除数据等。所谓的周边功能，包括性能统计，日志，事务管理等等。

周边功能在Spring的面向切面编程AOP思想里，即被定义为`切面`。

我们可以对核心业务功能和切面功能分别独立进行开发，然后把切面功能和核心业务功能 "编织" 在一起，这就叫AOP。同样，“编织”的方式可以是 xml 或者 注解。

AOP 的好处是允许我们把遍布应用各处的功能分离出来形成可重用的组件。

- 参考书籍：《Spring实战》第4版

---

# Spring Boot

![boot](../../../../images/Webapp/SpringBootLogo.png)

在 Spring MVC 框架中，我们不得不进行大量的配置，而在 Spring Boot 快速框架中，很多配置框架都帮你做好，拿来即用。

注意：
- Spring Boot使用 “习惯优于配置” （项目中存在大量的配置，此外还内置一个习惯性的配置）的理念让你的项目快速运行起来。
- Spring Boot并不是什么新的框架，而是默认配置了很多框架的使用方式，就像 Maven 整合了所有的 jar 包一样，Spring Boot 整合了所有框架。

## IDEA Spring Boot 实战

### 创建工程

在 IDEA 中，创建一个新的Spring Initalizr工程, Type 选择 Maven， 组件选择 Web ， IDEA 会自动帮我们新建一个基于 Maven 的 Spring Boot 工程。

> 或者通过 https://start.spring.io/ 初始化工程

看一下 pom.xml 大概长这样

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>jerrysheh</groupId>
    <artifactId>springboot-demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>springboot-demo</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

### java代码

在src/main/java/Example.java里面，应该已经有类似下面这样的代码了。

```java
@SpringBootApplication
public class ToywebApplication {

    public static void main(String[] args) {
        SpringApplication.run(ToywebApplication.class, args);
    }
}
```

写一个类：HelloController.java

```java
@RestController
public class HelloController {

    @GetMapping("/")
    public String hello(){
        return "Hello World";
    }
}
```

直接运行， 访问`127.0.0.1:8080`， 竟然已经能看到 Hello World 了，我们还没有进行 project structure 以及 Tomcat 配置呢 ？ 事实上， Spring Boot 已经内置了这些配置，拿来即用。

- `@RestController` 注解是 `@Controller` 和 `@ResponseBody` 的合体
- `@SpringBootApplication` 是 Spring Boot 的核心注解，它是一个组合注解，该注解组合了：`@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan`

### 配置文件

 Spring Boot 的配置文件为 application.properties 或 application.yml，放置在【src/main/resources】目录或者类路径的 /config 下。

 ![prop](../../../../images/Webapp/Springbootprop.png)

### 排除自动配置

在 Spring Initalizr 的时候，如果我们点多了组件，有可能会导致启动失败，这时候在`@SpringBootApplication`注解后添加排除项即可。或者在 pom.xml 中去除多余组件。

```java
@SpringBootApplication (exclude= {DataSourceAutoConfiguration.class})
```

### 打包 jar

确保 pom.xml 里面有

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

执行以下语句进行打包。

```
mvn package
```

### 运行

将打包的 jar 拷贝到 linux 服务器上， 执行以下命令即可启动

```
java -jar spring-boot01-1.0-SNAPSHOT.jar
```

但是这样命令行一退出程序也跟着退出了，可以使用以下命令，将 log 输入到文件，保持程序在后台运行。

```
java -jar spring-boot01-1.0-SNAPSHOT.jar > log.file 2>&1 &
```

这种方式看似脱离终端了，但是实际上还是受终端影响，当 SSH 退出时终端关闭，项目也会跟着关闭。

因此好的办法是将其写入到 shell 脚本中


```
vim run.sh
```

在 run.sh 里面输入
```shell
#!/bin/bash
java -jar spring-boot01-1.0-SNAPSHOT.jar > log.file 2>&1 &

```

run.sh添加执行权限，再执行

```
chmod +x run.sh
./run.sh
```

### 热部署

在 pom.xml 里面添加以下语句即可热部署，也就是我们修改了代码之后无需重启工程，即可看到效果。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional> <!-- 这个需要为 true 热部署才有效 -->
</dependency>

<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <!-- 没有该配置，devtools 不生效 -->
        <fork>true</fork>
    </configuration>
</plugin>
```

---

参考：
- [Spring Boot【快速入门】](https://www.cnblogs.com/wmyskxz/p/9010832.html)
- [Spring官方教程：Building a RESTful Web Service](https://spring.io/guides/gs/rest-service/)
- [how2j：SPRINGBOOT入门](http://how2j.cn/k/springboot/springboot-eclipse/1640.html#nowhere)
