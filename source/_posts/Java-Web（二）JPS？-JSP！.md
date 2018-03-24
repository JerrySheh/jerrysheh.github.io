---
title: 'Java Web（二）JPS？ JSP！ '
comments: true
categories: JAVA
tags: Java
abbrlink: dfdfe2eb
date: 2018-03-22 17:52:26
---

在装了 JRE 的机器中，我们在终端输入 `JPS` 可以查看正在运行的java进程。可是`JSP`又是什么鬼？

我们知道，Servlet 中可以对客户端发来的信息进行处理（doGet、doPost等），可是，在 Servlet 里面输出 HTML 代码是一件很酸爽的事情。

如果我们直接写 HTML 代码，然后在需要动态获取的地方用 Java 代码来实现，不是很方便？

JSP 就是干这个事的！

---

# JSP 如何转成 HTML

1. 把 hello.jsp 转译为hello_jsp.java
2. hello_jsp.java继承了`HttpServlet`类，因此它是一个servlet
4. hello_jsp.java 被编译为hello_jsp.class
5. 执行 hello_jsp，生成 html
6. 通过 http 协议把html 响应返回给浏览器

---

# JSP 的页面元素

## 静态内容

包括 HTML、CSS、JavaScript 等内容

## 指令

类似于下面 以 `<%@` 开头，以` %>` 结尾的
```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
```

## Scriptlet

类似于下面 以 `<%` 开头，以` %>` 结尾的
```
<%
    response.sendRedirect("hello.jsp");
%>
```

如果是`<%=`，后面加了个 `=` ，比如`<%="hello jsp"%> ` ，其实相当于 `<%out.println("hello jsp");%>`，这是一种隐式对象。

```
<%=new Date().toString()%>
```

## 动作

在jsp页面中包含另一个页面
```
<jsp:include page="Filename" >
```

跳转到另一个页面（服务端跳转）
```
<jsp:forward page="hello.jsp"/>
```

---

# Cookie 和 Session

关于 Cookie 和 Session 的概念，可参考 [HTTP之旅](../post/1707ee78.html)

## setCookie

我们可以在web目录下创建一个文件 setCookie.jsp，然后用 Scriptlet`<%...%>` new一个 Cookie 对象。

* 用`c.setMaxAge()`来设置保留时间（以秒为单位）。
* 用`c.setPath("127.0.0.1")`来设置主机名
* 用`response.addCookie(c);`把这个cookie保存在浏览器端

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8" import="javax.servlet.http.Cookie"%>

<%
    Cookie c = new Cookie("name", "Gareen");
    c.setMaxAge(60 * 24 * 60);
    c.setPath("127.0.0.1");
    response.addCookie(c);
%>

<a href="getCookie.jsp">跳转到获取cookie的页面</a>
```

访问：http://127.0.0.1/setCookie.jsp ，用Chrome F12工具可看到 cookie

## getCookie

在web目录下创建文件getCookie.jsp，填入

```
<%
    Cookie[] cookies = request.getCookies();
    if (null != cookies)
        for (int d = 0; d <= cookies.length - 1; d++) {
            out.print(cookies[d].getName() + ":" + cookies[d].getValue() + "<br>");
        }
%>
```


然后访问 http://127.0.0.1/getCookie.jsp ，可以看到name:Gareen，这就是setCookie.jsp中设置的Cookie
