---
title: Spring了然于心
comments: false
tags: JAVA
date: 2018-10-11 21:13:51
---

# 1. MVC 结构流程

1. Web浏览器发送HTTP请求到服务端，被Controller(Servlet)获取并进行处理（例如参数解析、请求转发）
2. Controller 调用核心业务逻辑——Model部分
3. Model进行数据库存取操作，并将操作结果返回
4. Controller 将业务逻辑处理结果交给View（JSP），动态输出HTML内容
5. 动态生成的HTML内容返回到浏览器显示

# 2. 什么是 Ioc ？

Ioc（Inversion of Control）是控制反转，具体做法的依赖注入（DI，Dependency Inject）。即某一接口的实现类的选择控制权从调用类中移除，转交由第三方决定，当需要的时候，由第三方进行注入，而不由调用类 new，这就是反转控制。

具体到Spring中，Spring提供了一个反转控制容器，当我们要使用某个对象，只需要从 Spring 容器中获取需要使用的对象，不关心对象的创建过程，把创建对象的控制权反转给了 Spring 框架，而 Spring 容器是通过 DI，在创建对象的过程中将对象依赖属性（简单值，集合，对象）通过配置设值给该对象。

# 3. Ioc是如何实现的?

1. 读取标注或者配置文件，看看 bean 依赖的是哪个Source，拿到类名
2. 使用反射的API，基于类名实例化对应的对象实例
3. 将对象实例通过构造函数或者 setter，注入给 bean

# 4. 什么是 AOP ？

Aspect Oriented Program，面向切面编程。即把功能分为 核心业务功能 和 周边功能。两者分别独立进行开发，然后把切面功能和核心业务功能 “编织” 在一起。

AOP 的好处是允许我们把遍布应用各处的功能分离出来形成可重用的组件。

# 5. 什么是 Spring Boot ？

Spring Boot并不是什么新的框架，而是默认配置了很多框架的使用方式，就像 Maven 整合了所有的 jar 包一样，Spring Boot 整合了大部分框架，比如Mybatis、Hibernate、Spring MVC。

Spring Boot使用 “习惯优于配置” （项目中存在大量的配置，此外还内置一个习惯性的配置）的理念让你的项目快速运行起来。

# 6. 什么是 Spring ？

Spring Framework是一个大家族，它是一个轻量级的 DI / IoC 和 AOP 容器的开源框架。包含了许多模块。

1. Data Access/Integration : 包含有JDBC、ORM、OXM、JMS和Transaction模块。
2. Web：包含了Web、Web-Servlet、WebSocket、Web-Porlet模块。
3. AOP模块：提供了一个符合AOP联盟标准的面向切面编程的实现。
4. Core Container(核心容器)：包含有Beans、Core、Context和SpEL模块。
5. Test模块：支持使用JUnit和TestNG对Spring组件进行测试。

# 7. 什么是 Spring Cloud ？

Spring Cloud是一套分布式服务治理的框架。可以理解成是一个注册中心，提供如服务发现注册、配置中心、消息总线、负载均衡、断路器、数据监控等。

Spring Cloud + Spring Boot 非常适合做微服务架构，Boot的轻量级适合开发单个微服务，多个服务再统一在 Cloud 中注册。

# 8. BeanFactory和ApplicationContext的区别

一般称 BeanFactory 为 IoC 容器，而 ApplicationContext 为应用上下文。

BeanFactory 是解析、管理、实例化所有容器的 Bean 的入口，**面向 Spring 本身**。且是 Spring Framework 最核心的接口，提供了高级的 Ioc 配置机制，使管理不同类型的 Java 对象成为可能。

ApplicationContext **面向框架的开发者**，提供国际化支持、统一的资源文件读取方式、框架事件体系等。

BeanFactory在启动的时候不会实例化Bean，getBean()的时候才会实例化。ApplicationContext在解析配置文件时会对配置文件所有对象都初始化。

# 9. Spring Bean的5种作用域

1. **singleton**: 是 Spring Bean 的默认配置，这个 Bean 在 Spring 容器是 单例 的。
2. **prototype**: 和 singleton 相反，为每个 Bean 请求提供一个 Bean 实例
3. **request**：在请求 Bean 范围内会给每个客户端的网络请求创建一个实例，请求结束之后会回收
4. **session**: 在每个 session 中有一个 Bean 的实例，session 结束后回收
5. **global-session**: 所有 Portlet 共享的 Bean

注意，Singleton Bean不是线程安全的，需要自行保证线程安全。

# 10. Spring 的5种自动装配模式

1. **no**：Spring 框架的默认设置，开发者要在 Bean 中明确定义依赖
2. **byName**：在配置文件中查找相同名字的 Bean 进行装配
3. **byType**：在配置文件中查找相同类型的 Bean 进行装配
4. **constructor**：寻找有相同构造参数的 Bean 进行装配
5. **autodetect**：先尝试以 constructor 的方法进行装配，失败后 byType 进行装配

# 11 SpringMVC处理请求的流程
1. 用户发送请求，被DispatcherServlet拦截，DispatcherServlet收到请求之后自己不处理，而是交给其他的Handler进行处理
2. DispatcherServlet初始化HandlerMapping，HandlerMapping会把请求映射成一个HandlerExecutionChain对象，这个对象包括一个Handler和多个Interceptor，然后把这个Handler适配成HandlerAdapter
3. DispatcherServlet传过来的请求会和HandlerAdapter进行适配，先要进行一些数据转换，然后调用HandlerAdapter的handle()，返回一个ModelAndView对象
4. mv.render()，通过ViewResolver进行渲染，把刚才HandlerAdapter返回的Model渲染到View上
5. 最后进行请求的转发
