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

前面提到`Spring MVC`是Java Web开发中对Servlet进行封装的框架。实际上，Spring是一个大家族，它是一个基于IoC和AOP的结构J2EE系统的框架。

其中最主要的包括Spring Framework（包括了IoC, AOP, MVC以及Testing）, Spring Data, Spring Security, Spring Batch等等，以及快速框架Spring Boot。他们都是为了解决特定的事情而产生的。

但是在学习这些框架前，有必要先弄清楚 Spring 最核心的两个概念：`IoC` 和 `AOP`。

## IoC （Inversion of Control，反转控制）

在设计一辆车的时候，如果我们先设计轮子、再根据轮子设计底盘、再根据底盘设计车身，这是一种自底向上的设计思想。但是，假若未来某一天需要改造一下轮子（比如由直径30cm改成40cm），那么底盘、车身不得不相应地进行改动。在大型的软件工程中，这种做法几乎是不可维护的，因为一个类可能作为其他上百个类的底层，我们不可能一一去修改。

`反转控制`其实是一种依赖倒置原则的设计思想。也就是反过来，让底层依赖上层。具体的做法就是使用 `DI （依赖注入，Dependency Inject）`，DI把底层类作为参数传给上层，实现上层对下层的控制。（包括可以使用 构造函数传递、Setter传递、接口传递）使用DI的一个好处是，让互相协作的软件组件保持松耦合。

采用依赖倒置原则的设计之后，会产生一个问题，假设我们要调用一个上层，由于上层需要接受下层作为参数，我们必须在构造上层前构造下层，这样我们的代码中就会写很多 new 。

这时候`反转控制容器（IoC Container）`出现了，这个容器可以自动对我们的代码进行初始化，而我们要做的，只是维护一个 Configuration， 具体到 Spring 中，我们可以通过 xml 配置、 @ 注解 或 Java配置（推荐）的方式让 Spring 帮我们注入对象(Spring 容器通过 `bean 工厂` 和 `应用上下文` 两种类型来实现)，我们得到对象后直接就可以使用，而不需要了解注入过程层层依赖的细节。

事实上，DI远没有这么简单，上面的例子只是为了帮助理解的一个通俗解释，并不严谨，譬如说《敏捷软件开发》第11章提到：

> 依赖倒置原则

> a.高层模块不应该依赖于底层模块，二者都应该依赖于抽象。
> b.抽象不应该依赖于细节，细节应该依赖于抽象。

而上面的例子并没有依赖于抽象。但我们这里先不讨论依赖关系该如何组织。现阶段所要理解的是，DI是组装应用对象的一种方式，我们无需知道依赖来自何处或者依赖的实现方式。

- 以上例子，来自 [知乎:Spring IoC有什么好处呢？
](https://www.zhihu.com/question/23277575/answer/169698662)
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

在 Spring MVC 框架中，我们不得不进行大量的配置， 而在 Spring Boot 快速框架中，很多配置框架都帮你做好。拿来即用。

---

# IDEA Spring Boot 实战

## 创建工程

在 IDEA 中，创建一个新的Spring Initalizr工程, Type 选择 Maven， 组件选择 Web ， IDEA 会自动帮我们新建一个基于 Maven 的 Spring Boot 工程。

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

## java代码

在src/main/java/Example.java里面，应该已经有类似下面这样的代码了，如果没有，需要手动添加。

```java
import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.web.bind.annotation.*;

@RestController
@EnableAutoConfiguration
public class Example {

	@RequestMapping("/")
	String home() {
		return "Hello World!";
	}

	public static void main(String[] args) throws Exception {
		SpringApplication.run(Example.class, args);
	}

}
```

然后运行一下， 登录 127.0.0.1:8080， 竟然已经能看到 Hello World 了，我们还没有进行 project structure 以及 Tomcat 配置呢 ？ 这是什么操作？

事实上， Spring Boot 已经内置了这些配置，拿来即用。

### 代码解析

以`@`开头的是注解。注解既方便我们阅读，也让框架识别某些代码的角色。

- `@RestController`：是`@ResponseBody`和`@Controller`的缩写，它表明我们的 Example 类是一个 Web Controller（控制器），当有 Web Request 进来的时候，Spring 会进行相应。
- `@RequestMapping`：表示路由路径映射，比如`@RequestMapping("/hello")`，就映射到 127.0.0.1:8080/hello 。
- `@EnableAutoConfiguration`：让 Spring Boot 根据你的依赖信息自动进行配置，例如我们在 pom.xml 中添加了`spring-boot-starter-web`，Spring Boot会认为你正在开发的是 Web 应用，因此进行 Web 的配置。

> 注意：如果不用`@RestController`而仅用`@Controller`的话，需要在每一个映射路径方法下添加`@ResponseBody`注解。

## 打包 jar

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

然后执行以下语句进行打包。

```
mvn package
```

## 热部署

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

关于 Spring Boot 的更多内容，放到下一篇说。
