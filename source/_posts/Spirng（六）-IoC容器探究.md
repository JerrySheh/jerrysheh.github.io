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

## BeanFactory

BeanFactory（Bean工厂）
