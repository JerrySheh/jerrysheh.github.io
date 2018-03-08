---
title: HTTP之旅
comments: true
categories: 计算机网络
tags: 计算机网络
abbrlink: 1707ee78
date: 2018-03-08 11:54:26
---

# 简介

一个 Web 应用程序，首先接触的是应用层协议是`超文本传输协议（HyperText Transfer Protocol，HTTP）`，HTTP由两个程序实现：一个客户端、一个服务器。HTTP的连接模型大概为：客户端向服务器发起`请求（request）`，服务器收到请求后，进行`响应（response）`，并返回响应的内容。

## web 对象

一般来说，一个 Web page（页面）是由很多对象组成的。对象可以是 html 页面，可以是图片，可以是嵌入的视频，还可以是java小程序。比如我们访问 Google.com ，这个 page 是由 Google 提供的一个基本 html 页面，以及搜索框上面大大的 logo 图片组成。

![Google](../../../../images/networking/google.png)


 这个 html 页面是一个对象（通常为index.html）， 这个 logo 图片也是一个对象。这些对象都存储在服务器上面。

 实际上对象一般都可以通过 URL 寻址，比如这张图的URL地址就是https://www.google.com/images/branding/googlelogo/1x/googlelogo_color_272x92dp.png(复制这个网址到浏览器，可以看到Google的图片，但是你之所以打不开是因为Google被￥@#￥墙了)

而 HTTP ， 就定义了客户端如何向服务器请求 Web page 的方式。HTTP使用TCP作为支撑运输协议，所以不用担心请求的过程数据在中途丢失或出错的问题。

<!-- more -->

## 无状态

我们可能注意到，我们使用浏览器打开Google.com，然后新建一个标签页又打开一次，Google 的 page 还是会又一次地显示出来，不存在服务器之前已经给你发过了所以不再发这种事。因此我们说HTTP是`无状态协议(stateless protocol)`。

---

# 非持续连接和持续连接

如果一个客户与服务器的 每一个 `请求/响应对`，分别由单独的 TCP 连接发送，这样的Web应用程序称为`非持续连接（non-persistent connection）`。假设在一个由 1 个 html.index 和 10 张 jpg 图片组成的 Web page中传输，则会建立11个 TCP 连接。

反之，如果同一个客户端的所有请求及服务器对它的响应经过相同的 TCP 连接发送，称为`持续连接（persistent connection）`。这样在上述例子中只需要建立1个 TCP 连接。

HTTP/1.0 协议使用非持久连接，即在非持久连接下,一个tcp连接只传输一个Web对象；

HTTP/1.1 默认使用持久连接(然而，HTTP/1.1协议的客户机和服务器也可以配置成使用非持久连接)。


---

# HTTP报文



## request报文

HTTP request报文结构如下：

![http](../../../../images/networking/http.jpg)

一个简单的 HTTP 报文如下：

```
GET /www.zsc.edu.cn/index.html HTTP /1.1
Accept-Language:zh-CN,zh;q=0.9,en;q=0.8,zh-TW;q=0.7
Host:www.zsc.edu.cn
Connection: close
User-Agent:Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36
```

第一行称为`请求行（request line）` GET 说明采用了 GET 方法， HTTP协议中，有GET、POST、HEAD、PUT、DELETE、OPTIONS、TRACE、CONNECT 等方法，其中最常用的是 GET 和 POST， GET用于获取内容， POST常用于表单提交。其后接着的是 GET 的页面以及采用的 HTTP协议版本，这里是 1.1。

> 不是向服务器提交表单就一定要用 POST，比如我们在某网站的表单上填 hello 和 banana，然后点确定，可能会构造一个类似于 www.somesite.com/select?hello&banana 这样的URL，服务器解析这种URL就知道你填的是什么了。当然，这种情况不适合输入账号和密码。

后面几行称为`首部行（header line）`。其中，`Accept-Language`指明了客户端期望接受的语言，这里是汉语。`Host`表面请求的主机域名，`Connection`表面客户端期望在本次连接后就断掉 TCP 连接，`User-Agent`则指明了客户端的浏览器类型，以方便服务器根据不同的浏览器类型发送不同的版本。

> 一个简单的例子是，当我们访问 https://developer.android.com/studio/index.html 想要下载 Android Studio软件时，如果我们用的是Windows 系统的浏览器，页面则会默认显示Windows版本的Android Studio软件下载，反正如果我们用的是 Mac 系统，则会默认显示 Mac 版本。这就是因为服务器根据我们的 User-Agent判断我们是当前什么系统。

如果是 POST 方法，在 首部行 后面会有一个空行，紧接着是`请求包体`。包括了表单中提交的内容。

## response报文

![http](../../../../images/networking/http2.jpg)

```
HTTP/1.1 200 OK
Connection:close
Content-Type:text/html; charset=UTF-8
Date:Thu, 08 Mar 2018 08:37:21 GMT
Expires:Thu, 08 Mar 2018 09:07:21 GMT
Pragma:no-cache
Server:Apache/2.2.15 (CentOS)
Transfer-Encoding:chunked
X-Powered-By:PHP/5.3.3

（然后是数据）
```

第一行称为`状态行（status line）`，这一行包括了协议和状态码。常见的状态码有：

* **200 OK**: 表示请求成功
* **301 Moved Permanently**: 重定向转移
* **400 Bad request**： 请求不能被服务器理解
* **404 Not Found**： 请求的对象在服务器上找不到
* **500 Internal Server Error**：服务器已收到请求，但服务器内部出错导致无法响应

下面几行称为`首部行（header line）`，内容与 request 的首部行大同小异。

最后是数据，也就是请求的对象。如果请求的是 html 页面则在浏览器显示网页，如果请求的是图片则显示图片等。

---

# cookie

HTTP是无状态协议，那么网站是怎么识别我们的呢？ cookie 用于站点对同一用户进行识别。

cookie技术有 4 个组件：
* request报文首部中有一个 cookie 的首部行
* response报文首部中有一个 cookie 的首部行
* 客户端系统中有一个 cookie 文件，由浏览器进行管理
* 服务器后端数据库中有 cookie 相关数据

比如说，当我们第一次登录 JD.com 进行购物的时候，JD服务器的 response 报文会对我的浏览器设置一个 `set-cookie: 1678`的首部行，这个`set-cookie`于是存入了我们的电脑。然后我们点击几样商品，然后转入购物车结算页面，此时对 HTTP 来说是一次全新的连接，但是结算页面却能准确显示我们刚才选的商品，这是因为我们在进入结算页面时，request首部中包含了刚刚 JD.com 给我们的 `cookie: 1678`， JD.com 就知道你是刚刚 1678 那个人了。于是把刚才页面你勾选的商品显示出来进行结算。

## Cookie 和 Session 的区别和联系

在某些地方你可能会听说过 Session 这个名词，简单地说，Session是在服务端保存的一个数据结构，用来跟踪用户的状态，这个数据可以保存在集群、数据库、文件中；而Cookie是客户端保存用户信息的一种机制，用来记录用户的一些信息，也是实现 Session 的一种方式。

---

# 代理服务器

`Web 缓存器（Web cache）`，或者叫`代理服务器（proxy server）`。用于存放真实服务器上面的一些对象。提高用户访问速度。

比如说，我们访问 www.somesite.com/campus.gif， 如果对方服务器有配置代理服务器，则我们的请求会首先发给代理服务器，代理服务器检查自己有没有这个文件，如果有，直接response 给客户，如果没有，代理服务器向真实服务器请求这个文件，获取到以后先自己复制一份，然后发送给客户。

这样做的好处就是降低了服务器的压力（因为服务器除了存储对象还有其他事要干，资源获取这种简单的活就交给代理服务器去干了）。同时，也有利于提高用户访问速度。

## 条件 GET 方法

使用代理服务器有一个问题，那就是代理服务器缓存的对象，也可能是旧的。比如今天缓存了一张图片，明天真实服务器修改了这张图片，后天有一个浏览器请求这张图片。如果代理服务器直接发给客户，不就发了旧版本吗？

解决方案是条件 GET 方法， 这个 GET 方法 由代理服务器向真实服务器 发出，其中包括 `If-Modified-Since`首部行，如果与服务器上的一致，则证明是最新的，否则，从服务器请求最新的文件过来。
