---
title: 'Java Web（二）JavaServer Pages （JSP） '
comments: true
categories: Java Web
tags:
 - Java
 - Web
abbrlink: dfdfe2eb
date: 2018-03-22 17:52:26
---

我们知道，Servlet 中可以对客户端发来的信息进行处理（doGet、doPost等），可是，在 Servlet 里面输出 HTML 代码是一件很酸爽的事情。

如果我们直接写 HTML 代码，然后在需要动态获取的地方用 Java 代码来实现，不是很方便？

JSP 就是干这个事的！

维基百科定义: JSP（全称JavaServer Pages）是由Sun Microsystems公司主导建立的一种动态网页技术标准。 JSP部署于网络服务器上，可以响应客户端发送的请求，并根据请求内容动态地生成HTML、XML或其他格式文档的Web网页，然后返回给请求者。

<!-- more -->

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

---

# 作用域

JSP 有 4 个作用域
- **pageContext**： 只能在当前页面访问，在其他页面就不能访问了。
- **requestContext**： 一次请求。随着本次请求结束，其中的数据也就被回收。
- **sessionContext**： 当前会话。从一个用户打开你的网站的那一刻起，无论访问了多少个子网页，链接都属于同一个会话，直到浏览器关闭。
- **applicationContext**： 全局，所有用户共享

作用域示例

setContext.jsp
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>

<%
    pageContext.setAttribute("name","gareen");
%>

<%=pageContext.getAttribute("name")%>
```

getContext.jsp
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>

<%=pageContext.getAttribute("name")%>
```

getContext.jsp从setContext.jsp获取了数据，由于是`pageContext`，所以只能在当前页面，跳转到其他页面就不能获取了。

---

# 隐式对象

JSP的隐式对象指的是不需要显示定义，直接就可以使用的对象。

JSP一共有9个隐式对象，分别是
- **request**：请求
- **response**：响应
- **out**：输出
- **pageContext**：当前页面作用域
- **session**：会话作用域
- **application**：全局作用域
- **page**：表示当前对象。JSP 会被编译为一个Servlet类 ，运行的时候是一个Servlet实例。 page即代表this
- **config**：config可以获取一些在web.xml中初始化的参数。
- **exception**：异常。只有当前页面的<%@page 指令设置为isErrorPage="true"的时候才可以使用。

参考：[HOW2J](http://how2j.cn/k/jsp/jsp-object/580.html#nowhere)

---

# JSTL

JSP Standard Tag Library 标准标签库

JSTL允许开人员可以像使用HTML标签 那样在JSP中开发Java功能。

JSTL库用得比较多的有 core 和 fmt

---

# EL表达式

首先在 jsp 头标注isELIgnored="false"，因为不同版本的 Tomcat 对 EL 表达式默认开关不一样。

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8" isELIgnored="false"%>

<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>

<c:set var="name" value="${'gareen'}" scope="request" />

通过标签获取name: <c:out value="${name}" /> <br>

通过 EL 获取name: ${name}
```

可见，JSTL输出要写成`<c:out value="${name}" /> `的代码，用 EL表达式只需要写`${name}`，非常方便。

## JavaBean

JavaBean是一种标准
1. 提供无参public的构造方法(默认提供)
2. 每个属性，都有public的getter和setter
3. 如果属性是boolean,那么就对应is和setter方法

我们可以用 EL 表达式来获取 JavaBean 的属性

如`${student.name}`，就会自动调用getName方法

完整例子来自 How2j

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8" isELIgnored="false" import="bean.*"%>

<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>

<%
    Hero hero =new Hero();
    hero.setName("盖伦");
    hero.setHp(616);

    request.setAttribute("hero", hero);
%>

英雄名字 ： ${hero.name} <br>
英雄血量 ： ${hero.hp}
```

例子2

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8" import="java.util.*"%>

<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>

<%
    List<String> heros = new ArrayList<String>();
    heros.add("塔姆");
    heros.add("艾克");
    heros.add("巴德");
    heros.add("雷克赛");
    heros.add("卡莉丝塔");
    request.setAttribute("heros",heros);
%>

<table width="200px" align="center" border="1" cellspacing="0">
<tr>
    <td>编号</td>
    <td>英雄</td>
</tr>

<c:forEach items="${heros}" var="hero" varStatus="st"  >
    <tr>
        <td>${st.count}</td>
        <td>${hero}</td>
    </tr>
</c:forEach>
</table>
```
