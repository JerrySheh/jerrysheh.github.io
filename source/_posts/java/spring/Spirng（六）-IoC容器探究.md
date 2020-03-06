---
title: Spirng（六） IoC容器探究
comments: true
categories:
- Java Web
- Spring
tags:
  - Java
  - Web
abbrlink: fd78ec01
date: 2018-07-09 22:33:32
---

![Spring](../../../../images/Java/Spring.png)

在 [Spring（一）从 传统Java Web到SpirngBoot](../post/6200df85.html) 中对 Ioc 的概念已经有了初步认识：Spring 通过一个配置文件描述 Bean 与 Bean 之间的依赖关系，利用 Java 的类加载器和反射机制实例化 Bean 并建立 Bean 之间的依赖关系。

我们将调用类对某一接口实现类的依赖关系交由 Spring 容器管理，容器在我们需要的时候，通过注入及时地将对象进行实例化并装配好 bean，无需我们自己 new 。

除此之外，由于JDK提供的访问资源的类对底层资源并不友好，缺少从类路径或者Web容器的上下文获取资源的操作类，**Spring重新设计了一个 Resource 接口**，用于更强的底层资源访问能力。有了这个资源类，就可以将Spring的配置信息放在任何地方（数据库、LDAP）。而为了访问不同类型的资源，**Spring还提供了一个强大的加载资源的机制**，定义了一套资源加载的接口 ResourceLoader 及其实现类，可以访问包括`classpath:`、`file:`、`http://`、`ftp://`等地址前缀资源。

这一篇具体讲讲关于 Spring Ioc的更多内容。

<!--more-->

---

# BeanFactory 和 ApplicationContext

一般称 BeanFactory 为 IoC 容器，而 ApplicationContext 为应用上下文。

## BeanFactory

BeanFactory（com.springframework.beans.factory.BeanFactory）是 Spring Framework 最核心的接口，提供了高级的 Ioc 配置机制，使管理不同类型的 Java 对象成为可能。

在设计模式中有工厂模式，BeanFactory就是一个类工厂，它是一个通用工厂，可以创建并管理各种类的对象。这些对象都是普通的 pojo ，Spring 称这些对象为 bean 。BeanFactory 在启动的时候不会实例化Bean，getBean()的时候才会实例化。

> Spring中的 bean 跟 javabean 的区别： javabean 需要满足一定的规范，但 Spring 中只要能被 Spring 容器实例化并管理的对象都称为 bean。

BeanFactory 是 Spring Framework 的基础设施，它是解析、管理、实例化所有容器的 Bean 的入口，<font color="red">BeanFactory 面向 Spring 本身</font>。

## ApplicationContext

ApplicationContext（com.springframework.context.ApplicationContext）在 BeanFactory 的基础上提供更多面向应用的功能：国际化支持、统一的资源文件读取方式、框架事件体系等。

<font color="red">ApplicationContext 面向框架的开发者</font>，几乎所有的应用场合都可以直接使用 ApplicationContext 而非底层的 BeanFactory。ApplicationContext在解析配置文件时会对配置文件所有对象都初始化。

如果把 BeanFactory 比喻成“心脏”，那么 ApplicationContext 就是 “身躯”。

### ApplicationContext 类体系结构

待补充。

Spring支持类注解的配置方式，主要功能来自 Spring 的一个子项目 JavaConfig。

### WebApplicationContext 类体系结构

待补充。

---

# 父子容器

通过 HierarchicalBeanFactory 接口， Spirng IoC 容器可以建立父子层级关联的容器体系。子容器可以访问父容器的 Bean， 但反过来则不行。这种体系增强了 Spring 容器架构的扩展性和灵活性。我们可以通过编程的方式为一个已存在的容器添加一个或多个由特殊用途的子容器。

例如，在 Spring MVC 中，表现层位于一个子容器中， 业务逻辑层 和 数据访问层 位于父容器中。这样，表现层可以引用业务逻辑层和数据访问层的 Bean，而业务逻辑层和数据访问层看不到表现层的 Bean。

---

# IoC容器的初始化

IOC容器的初始化的入口是`refresh()`

1. 先进行 Resource 的定位与载入，由 ResourceLoader 完成
2. 载入 BeanDefinition，这个载入过程就是把用户定义的 Bean 转换成 Bean 容器中的数据结构 BeanDefinition。BeanDefinition 的载入第一步是调用SAX进行解析得到 Document 对象，然后用一个 DocumentReader 进行解析，解析完的结果由个BeanDefinitionHolder对象持有
3. 向IOC容器中注入载入后的 BeanDefinition，这个过程是通过BeanDefinitionRegistry实现的，实际上是注册到一个ConcurrentHashMap中


---

# Bean 的生命周期

1. 实例化：Spring通过一定的策略实例化 Bean。
2. 填入属性：spring 将 值 和 bean 引用注入到 bean 的属性中。
3. 如果 Bean 实现了 BeanNameAware 接口，工厂调用 Bean 的 setBeanName() 方法传递 Bean 的 ID。
4. 如果 Bean 实现了 BeanFactoryAware 接口，工厂调用 setBeanFactory() 方法传入工厂自身。
5. 如果实现了 ApplicationContextAware 接口, spring 将调用 setApplicationContext() 方法，将 bean 所在的上下文的引用进来。
6. 如果 BeanPostProcessor 和 Bean 关联，那么它们的 postProcessBeforeInitialization() 方法将被调用。
7. 如果 Bean 指定了 init-method 方法，该方法将被调用。
8. 如果有 BeanPostProcessor 和 Bean 关联，那么它们的 postProcessAfterInitialization() 方法将被调用
9. 如果配置了 destroy-method 方法则注册 DisposableBean
10. 使用：到这个时候，Bean已经可以被应用系统使用了，并且将被保留在Bean Factory中直到它不再需要。
11. 销毁。如果 Bean 实现了 DisposableBean 接口，销毁时就调用其 destroy 方法。

销毁方法：

1. 如果 Bean 实现了 DisposableBean 接口，destory()方法被调用。
2. 如果指定了订制的销毁方法，就调用这个方法。destory-method() 配置时指定。

---

# Spring Bean 的作用域

1. **singleton**: 是 Spring Bean 的默认配置，这个 Bean 在 Spring 容器是 **单例** 的
2. **prototype**: 和 singleton 相反，为每个 Bean 请求提供一个 Bean 实例
3. **request**：在请求 Bean 范围内会给每个客户端的网络请求创建一个实例，请求结束之后会回收
4. **session**: 在每个 session 中有一个 Bean 的实例，session 结束后回收
5. **global-session**: 所有 Portlet 共享的 Bean

## Spring Boot 如何修改 bean 的作用域 ？

加 `@Scope()` 注解即可   

```java
@Bean
@Scope("singleton")
public Person personSingleton() {
    return new Person();
}
```

---

## Spring 自动装配模式

Spring容器可以自动配置相互协作beans之间的关联关系。这意味着Spring可以自动配置一个bean和其他协作bean之间的关系，通过检查 BeanFactory 的内容里没有使用和< property>元素。

1. **no**：Spring 框架的默认设置，开发者要在 Bean 中明确定义依赖
2. **byName**：在配置文件中查找相同名字的 Bean 进行装配
3. **byType**：在配置文件中查找相同类型的 Bean 进行装配
4. **constructor**：寻找有相同构造参数的 Bean 进行装配
5. **autodetect**：先尝试以 constructor 的方法进行装配，失败后 byType 进行装配

---

# 在Spring中注入Java集合类

Spring 提供如下几种类型的集合配置元素：

- list元素用来注入一系列的值，允许有相同的值。
- set元素用来注入一些列的值，不允许有相同的值。
- map用来注入一组”键-值”对，键、值可以是任何类型的。
- props也可以用来注入一组”键-值”对，这里的键、值都字符串类型。

```java
@Autowired
private List<UserService> userServices;

@Autowired
private Map<String,DemoService> demoServiceMap;
```
