---
title: Java Web（一）Servlet和web容器
comments: true
categories: JAVA
tags: Java
abbrlink: d697e4e7
date: 2018-03-04 23:43:38
---


`Servlet`是任何Java EE Web应用程序的一个关键组件，用于接受和响应HTTP请求的Java类，可以简单地理解为是服务器端处理数据的java小程序。而`web容器`，就是帮助我们管理着servlet等，使我们只需要将重心专注于业务逻辑。

<!-- more -->

# Web容器 - Tomcat

不管什么web资源，想被远程计算机访问，都必须有一个与之对应的网络通信程序，当用户来访问时，这个网络通信程序读取web资源数据，并把数据发送给来访者。WEB服务器就是这样一个程序，它用于完成底层网络通迅。使用这些服务器，We应用的开发者只需要关注web资源怎么编写，而不需要关心资源如何发送到客户端手中，从而极大的减轻了开发者的开发工作量。

Tomcat就是这样的一个 Web 服务器。

# Servlet

如果想开发一个Java程序向浏览器输出数据，需要完成以下2个步骤：
1. 编写一个Java类，实现servlet接口。
2. 把开发好的Java类部署到web服务器中。

按照一种约定俗成的称呼习惯，通常我们也把实现了servlet接口的java程序，称之为Servlet。

## Servlet的运行过程

Servlet程序由Web服务器调用，web服务器收到客户端的Servlet访问请求后：
1. Web服务器首先检查是否已经装载并创建了该Servlet的实例对象。如果是，则直接执行第4步，否则，执行第2步
2. 装载并创建该Servlet的一个实例对象
3. 调用Servlet实例对象的init()方法
4. 创建一个用于封装HTTP请求消息的`HttpServletRequest对象`和一个代表HTTP响应消息的`HttpServletResponse对象`，然后调用Servlet的service()方法并将请求和响应对象作为参数传递进去。
5. Web应用程序被停止或重新启动之前，Servlet引擎将卸载Servlet，并在卸载之前调用Servlet的`destroy()`方法。

也就是说，web容器只有在首次访问时才创建Servlet，然后调用Servlet的`init()`方法。之后web容器创建请求（request）对象和响应（response）对象（响应对象此时为空），并将这两个对象作为参数，传入Servlet的`service()`方法。经`service()`方法处理后，将结果写入响应信息（此时响应对象已有内容）。最后，由web容器取出响应信息回送给浏览器。

![servlet](../../../../images/Webapp/Servlet.png)

helloServlet.java
```java
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

@WebServlet("/hello")
public class helloServlet extends HttpServlet {
    private String message;

    public void init() throws ServletException{
        message = "hello world!!";
    }

    @Override
    public void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {

// 设置响应内容类型
        response.setContentType("text/html");

// 实际的逻辑是在这里
        PrintWriter out = response.getWriter();
        out.println("<h1>" + message + "</h1>");
    }
}
```

访问 `localhost:8080/hello`，能看到 hello world!!

## Servlet与普通Java类的区别　

Servlet是一个供其他Java程序（Servlet引擎）调用的Java类，它不能独立运行，它的运行完全由Servlet引擎来控制和调度。

针对客户端的多次Servlet请求，通常情况下，**服务器只会创建一个Servlet实例对象**，也就是说Servlet实例对象一旦创建，它就会驻留在内存中，为后续的其它请求服务，直至web容器退出，servlet实例对象才会销毁。

在Servlet的整个生命周期内，Servlet的**init方法只被调用一次**。而对一个Servlet的每次访问请求都导致Servlet引擎调用一次servlet的service方法。对于每次访问请求，Servlet引擎都会创建一个新的HttpServletRequest请求对象和一个新的HttpServletResponse响应对象，然后将这两个对象作为参数传递给它调用的Servlet的service()方法，service方法再根据请求方式分别调用doXXX方法。

如果在<servlet>元素中配置了一个<load-on-startup>元素，那么WEB应用程序在启动时，就会装载并创建Servlet的实例对象、以及调用Servlet实例对象的init()方法。

## Servlet的线程安全问题

当多个客户端并发访问同一个Servlet时，web服务器会为每一个客户端的访问请求创建一个线程，并在这个线程上调用Servlet的service方法，因此service方法内如果访问了同一个资源的话，就有可能引发线程安全问题。

线程安全问题只存在多个线程并发操作同一个资源的情况下，所以在编写Servlet的时候，如果并发访问某一个资源(变量，集合等)，就会存在线程安全问题。

解决方案：让Servlet去实现一个`SingleThreadModel`接口，如果某个Servlet实现了`SingleThreadModel`标记接口，那么Servlet引擎将以单线程模式来调用其service方法。

对于实现了`SingleThreadModel`接口的Servlet，Servlet引擎仍然支持对该Servlet的多线程并发访问，其采用的方式是产生多个Servlet实例对象，并发的每个线程分别调用一个独立的Servlet实例对象。

实现SingleThreadModel接口并不能真正解决Servlet的线程安全问题，因为Servlet引擎会创建多个Servlet实例对象，而真正意义上解决多线程安全问题是指一个Servlet实例对象被多个线程同时调用的问题。

事实上，在Servlet API 2.4中，已经将`SingleThreadModel`标记为Deprecated（过时的）。  

### 标记接口

在Java中，把没有定义任何方法和常量的接口称之为标记接口，经常看到的一个最典型的标记接口就是"Serializable"，这个接口也是没有定义任何方法和常量的，标记接口在Java中有什么用呢？主要作用就是给某个对象打上一个标志，告诉JVM，这个对象可以做什么，比如实现了"Serializable"接口的类的对象就可以被序列化，还有一个"Cloneable"接口，这个也是一个标记接口，在默认情况下，Java中的对象是不允许被克隆的，就像现实生活中的人一样，不允许克隆，但是只要实现了"Cloneable"接口，那么对象就可以被克隆了。




---

参考链接

* [IntelliJ idea 2017创建Web项目后web文件夹下没有WEB-INF的解决方法](http://blog.csdn.net/xwx617/article/details/79269939)
