---
title: 使用RESTful风格开发Java Web
comments: true
categories: Java Web
tags: Java Web
abbrlink: 3479b7db
date: 2018-07-05 16:16:42
---

# 什么是RESTful风格？

REST是 REpresentational State Transfer 的缩写（一般中文翻译为表述性状态转移），REST 是一种体系结构，而 HTTP 是一种包含了 REST 架构属性的协议，为了便于理解，我们把它的首字母拆分成不同的几个部分：

- 表述性（REpresentational）： REST 资源实际上可以用各种形式来进行表述，包括 XML、JSON 甚至 HTML——最适合资源使用者的任意形式；
- 状态（State）： 当使用 REST 的时候，我们更关注资源的状态而不是对资源采取的行为；
- 转义（Transfer）： REST 涉及到转移资源数据，它以某种表述性形式从一个应用转移到另一个应用。

简单地说，REST 就是**将资源的状态以适合客户端或服务端的形式从服务端转移到客户端（或者反过来）** 。

在 REST 中，资源通过 URL 进行识别和定位，然后通过行为(即 HTTP 方法)来定义 REST 来完成怎样的功能。

## 实例说明

在HTTP协议中，有GET、POST、HEAD、PUT、DELETE、OPTIONS、TRACE、CONNECT 众多方法，我们可以通过不同的HTTP方法来对应CRUD的操作。

例如：

HTTP方法|CRUD操作
---|---
GET| 查询(Retrieve)
POST| 更新(Update)
PUT| 增加(Create)
DELETE | 删除(Delete)

尽管通常来讲，HTTP 方法会映射为 CRUD 动作，但这并不是严格的限制，有时候 PUT 也可以用来创建新的资源，POST 也可以用来更新资源。实际上，**POST 请求非幂等的特性(即同一个 URL 可以得到不同的结果)** 使其成一个非常灵活地方法，对于无法适应其他 HTTP 方法语义的操作，它都能够胜任。

在使用 RESTful 风格之前，我们如果想要增加一条商品数据通常是这样的:

```
/addCategory?name=xxx
```

但是使用了 RESTful 风格之后就会变成:

```
/category
```

这就变成了使用同一个 URL ，通过约定不同的 HTTP 方法来实施不同的业务，这就是 RESTful 风格所做的事情了。

![RESTful](../../../../images/Webapp/RESTful.png)

---

# Springboot 实战

未完待续

---

参考
- http://www.cnblogs.com/wmyskxz/p/9104368.html
- http://blog.didispace.com/springbootrestfulapi/
