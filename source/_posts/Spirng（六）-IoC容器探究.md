---
title: Spirng（六） IoC容器探究
comments: true
categories: Java Web
tags:
  - Java
  - Web
abbrlink: fd78ec01
date: 2018-07-09 22:33:32
---

![Spring](../../../../images/Java/Spring.png)

在 [Spring（一）从 传统Java Web到SpirngBoot](../post/6200df85.html) 中对Ioc的概念已经有了初步认识：Spring通过一个配置文件描述 Bean 与 Bean 之间的依赖关系，利用Java的类加载器和反射机制实例化Bean并建立Bean之间的依赖关系。

我们将调用类对某一接口实现类的依赖关系交由Spring容器管理，容器在我们需要的时候，通过注入及时地将对象进行实例化并装配好bean。

除此之外，由于JDK提供的访问资源的类对底层资源并不友好，缺少从类路径或者Web容器的上下文获取资源的操作类，**Spring重新设计了一个 Resource 接口**，用于更强的底层资源访问能力。有了这个资源类，就可以将Spring的配置信息放在任何地方（数据库、LDAP）。而为了访问不同类型的资源，**Spring还提供了一个强大的加载资源的机制**，定义了一套资源加载的接口 ResourceLoader 及其实现类，可以访问包括`classpath:`、`file:`、`http://`、`ftp://`等地址前缀资源。

这一篇具体讲讲关于 Spring Ioc的更多内容。

<!--more-->

---

# BeanFactory 和 ApplicationContext

一般称 BeanFactory 为 IoC 容器，而 ApplicationContext 为应用上下文。

## BeanFactory

BeanFactory（com.springframework.beans.factory.BeanFactory）是 Spring Framework 最核心的接口，提供了高级的 Ioc 配置机制，使管理不同类型的 Java 对象成为可能。

BeanFactory是 Spring Framework 的基础设施，面向 Spring 本身。

在设计模式中有工厂模式，BeanFactory就是一个类工厂，它是一个通用工厂，可以创建并管理各种类的对象。这些对象都是普通的 pojo ，Spring 称这些对象为 bean 。

> Spring中的 bean 跟 javabean 的区别： javabean 需要满足一定的规范，但 Spring 中只要能被 Spring 容器实例化并管理的对象都称为 bean。

## ApplicationContext

ApplicationContext（com.springframework.context.ApplicationContext）在 BeanFactory 的基础上提供更多面向应用的功能。国际化支持和框架事件体系。

ApplicationContext 面向框架的开发者，几乎所有的应用场合都可以直接使用 ApplicationContext 而非底层的 BeanFactory。

如果把 BeanFactory 比喻成“心脏”，那么 ApplicationContext 就是 “身躯”。

### ApplicationContext 类体系结构

待补充。

Spring支持类注解的配置方式，主要功能来自 Spring 的一个子项目 JavaConfig。

### WebApplicationContext 类体系结构

待补充。

## 父子容器

通过 HierarchicalBeanFactory 接口， Spirng IoC 容器可以建立父子层级关联的容器体系。子容器可以访问父容器的 Bean， 但反过来则不行。这种体系增强了 Spring 容器架构的扩展性和灵活性。我们可以通过编程的方式为一个已存在的容器添加一个或多个由特殊用途的子容器。

例如，在 Spring MVC 中，表现层位于一个子容器中， 业务逻辑层 和 数据访问层 位于父容器中。这样，表现层可以引用业务逻辑层和数据访问层的 Bean，而业务逻辑层和数据访问层看不到表现层的 Bean。

---

# Bean 的生命周期

Bean的生命周期可以从两个层面定义：
- Bean 的作用范围
- 实例化 Bean 时所经历的一系列阶段

## BeanFactory 中 Bean 的生命周期

待补充。

## ApplicationContext 中 Bean 的生命周期

待补充。

---

#   在 IoC 容器中装配 Bean

Spring容器内部协作结构图

TODO...
