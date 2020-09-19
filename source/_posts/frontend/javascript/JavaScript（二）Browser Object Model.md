---
title: JavaScript（二）Browser Object Model
categories: 前端
tags:
  - JavaScript
  - Web
abbrlink: 8a9941f1
date: 2018-04-15 11:07:25
---

# 浏览器对象模型(Brower Object Model)

浏览器对象模型(Brower Object Model)  就是所谓的 BOM

浏览器对象包括：
- Window(窗口)
- Navigator(浏览器)
- Screen (客户端屏幕)
- History(访问历史)
- Location(浏览器地址)

<!-- more -->

## windows

- 用`window.innerWidth`和`window.innerHeight`获取浏览器的**文档显示区域**的宽和高
- 用`window.outerWidth`和`window.outerHeight`获取浏览器**外部窗体**的宽和高
- 用`window.open("/")`打开一个新页面，这里的`/`指本站的根目录。只能打开本站网页。（不建议在用户不知情的情况下随意打开新页面，影响用户体验）

体验一下：

<script>
function openNewWindow(){
  myWindow=window.open("/");
}
</script>

<button onclick="openNewWindow()">回到主页</button>

## Navigator

Navigator提供浏览器相关的信息

属性|简介
---|---
navigator.appName|浏览器产品名称
navigator.appVersion|浏览器版本号
navigator.appCodeName|浏览器内部代码
navigator.platform|操作系统
navigator.cookieEnabled|是否启用Cookies
navigator.userAgent|浏览器的用户代理报头

## 弹出框

- 用`alert`显示一个警告窗
- 用`confirm`显示一个确认框，根据用户选择，返回true or false
- 用`prompt`显示一个输入框

确认框例子

```html
<script>
function del(){
var d = confirm("是否要删除"); // d 是 boolean 类型
}
</script>

<br>
<button onclick="del()">删除</button>
```

输入框例子

```html
<script>
function p(){
var name = prompt("请输入用户名:"); // name 是 string 类型
alert("您输入的用户名是:" + name);
}
</script>

<br>
<button onclick="p()">请输入用户名</button>
```

## Location

Location 对象包含有关当前 URL 的信息。


```html
<script>
// url的值为当前URL
var url = location.href

// 跳转到谷歌
location.href = "https://www.google.com"
</script>
```

参考：

- [Location 对象](http://www.w3school.com.cn/jsref/dom_obj_location.asp)

---

[下一篇](../post/f75e92e9.html)介绍Javascript中的DOM（文档对象模型）
