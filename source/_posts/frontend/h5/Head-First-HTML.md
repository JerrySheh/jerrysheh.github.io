---
title: Head First HTML
categories: 前端
tags:
  - HTML
  - Web
abbrlink: 65acdb59
date: 2018-04-02 10:05:16
---

HTML是`HyperText Markup Language 超文本标记语言`的缩写。

HTML是由一套标记标签 （markup tag）组成，通常就叫标签。

<!-- more -->

# 基本元素

## 普通元素
元素|简介
---|---
&lt;h1&gt;|标题
&lt;p&gt;|段落（paragraph），自带换行效果
&lt;b> 或 &lt;strong> | 粗体，推荐用 &lt;strong&gt;
&lt;br /&gt;| 空行
&lt;i> 或 &lt;em&gt;| 斜体
&lt;pre&gt;|预格式，用来显示代码
&lt;del&gt;|删除线
&lt;ins&gt;|下划线(尽量不用，因为会跟超链接混淆)
&lt;a&gt;|定义超链接，用于从一张页面链接到另一张页面。
例子

```HTML
<html>
  <head>
  <meta http-equiv="Content-Type" content="text/html; charset=GB2312">
  </head>
  <body>
    <h1>大标题</h1>
    <br/>
    <h2>二级标题</h2>
    <p>Hello World</p>
  </body>
</html>
```

> &lt;meta&gt;解决了中文编码问题，如果 GB2312 不行，就用UTF-8

在标签中可以加入属性，比如居中

```html
<h1 >未设置居中的标题</h1>

<h1 align="center">居中标题</h1>
```

## 图片元素

元素|简介
---|---
&lt;img src="http://example.com/hello.png"/>|图片（链接可以是本地也可以是联网）
&lt;img width="200" height="200" src="http://example.com/hello.png"/>|设置图片宽高
&lt;img src="http://example.cn/not_exist.gif" alt="加载失败" />| alt用来显示图片加载失败时的提示语

让图片显示在页面左边

```html
<div align="left">
  <img src="http://example.com"/>
</div>
```

## 超链接

文字超链接

```html
<a href="http://www.12306.com" title="跳转到http://www.12306.com">www.12306.com</a>
```

图片超链接

```html
<a href="http://www.12306.com">
<img src="http://how2j.cn/example.gif"/>
</a>
```


## 表格

三行二列的表格

-  &lt;tr> 表示 行
-  &lt;td> 表示 列
- border属性表示边框
- width表示宽度（屏幕是1920*1080，那横向就有1920像素）

```html
<table border="1" width="800px">
  <tr>
      <td>1</td>
      <td>2</td>
  </tr>

  <tr>
      <td>3</td>
      <td>4</td>
  </tr>

  <tr>
      <td>a</td>
      <td>b</td>
  </tr>

</table>
```

- `align`水平对齐，`valign`垂直对齐，`bgcolor`背景颜色
- `colspan`属性表示，这一行横跨了 第一列和第二列
- `rowspan`属性表示，这一列纵跨了 第一行和第二行

## 列表

### 无序列表

```html
<ul>
<li>DOTA</li>
<li>LOL</li>
</ul>
```

效果：
- DOTA
- LOL

### 有序列表

```html
<ol>
<li>地卜师</li>
<li>卡尔</li>
</ol>
```

效果：
1. 地卜师
2. 卡尔

## 字体和颜色

`<font>`标签表示字体

可用 `color` 属性 和 `size` 属性

```html
<font color="green">绿色默认大小字体</font>
<br>
<font color="blue" size="+2">蓝色大2号字体</font>
<br>
<font color="red" size="-2">红色小2号字体</font>
```

## &lt;div> 和 &lt;span>

这两种标签都是布局用的，本身没有任何显示效果，通常是用来结合css进行页面布局

- div是块元素，即自动换行，常见的块元素还有h1,table,p
- span是内联元素，即不会换行，常见的内联元素还有img,a,b,strong

例子

```html
<img style="margin-left:50px" src="http://how2j.cn/example.gif"/>
  <br/>
 <img style="margin-left:50px" src="http://how2j.cn/example.gif"/>

<div style="margin-left:100px" >
 <img src="http://how2j.cn/example.gif"/>
  <br/>
 <img src="http://how2j.cn/example.gif"/>
</div>
```




---

# 转义

有时候我们确实想输入 &lt;h&gt; 这几个符号，但是HTML渲染的时候会自动把他们渲染成元素，而使我们看不到，这时候就要用转义。

 显示 | 转义
---|---
 空格 | `&nbsp;`
&lt;| `&lt;`
&gt;| `&gt;`
&amp;| `&amp;`
&quot; （引号）| `&quot;`
&apos; （撇号）(IE不支持)| `&apos;`

<br />

---

# 文本框

`<input>`标签中的 type 属性，设置为 text 是文本框， password 是密码框

- 属性`disabled="disabled"`可以让文本框只输出，不能输入
- 属性`id='id'`可以给这个标签添加唯一id号

## 有初始值的文本框

```html
<input type="text" value="666">
```
<div align="center">
效果： <input type="text" value="666">
</div>

### 获取文本框的值

```JavaScript
document.getElementById(id).value;
```

> 如果需要类型转换，对整体做 parseInt(); 方法

### 给文本框赋值

```JavaScript
var d = 66;
document.getElementById(id).value = d;
```

## 有提示语的文本框（HTML5 only）

```html
<input type="text" placeholder="请输入账号">
```
<div align="center">
效果： <input type="text" placeholder="请输入账号">
</div>

## 密码框

```html
<input type="password">
```
<div align="center">
效果：<input type="password" value="666666">
</div>

---

# 选择框

`<input>`标签中的 type 属性，设置为 `radio` 是单选， `checkbox`是复选。

## 单选框

- `checked`属性表示默认选中
- `name`属性表示同一组

```html
是<input type="radio" name="yesOrNo" value="是" >
&nbsp;
否<input type="radio" checked="checked" name="yesOrNo" value="否" >  
```

是<input type="radio" name="yesOrNo" value="是" > &nbsp;&nbsp; 否 <input type="radio" checked="checked" name="yesOrNo" value="否" >

## 复选框

```html
<input type="checkbox" name="Ojbk" value="jerry" >
&nbsp;
<input type="checkbox" checked="checked" name="Ojbk" value="calm" >  
&nbsp;
<input type="checkbox" name="Ojbk" value="superman" >
```

jerry<input type="checkbox" name="Ojbk" value="jerry" > &nbsp; calm<input type="checkbox" checked="checked" name="Ojbk" value="calm" >  &nbsp; superman<input type="checkbox" name="Ojbk" value="superman" >

---

# 下拉列表

`<select>`标签表示一个下拉列表， `<option>`标签是下拉列表的每个项

- `multiple="multiple"` 属性表示可以多选，按ctrl或shift进行多选
- `selected="selected"` 属性表示默认选中

```html
<select multiple="multiple" selected="selected" >
 <option >jerry</option>
 <option >calm</option>
 <option >superman</option>
</select>
```

<div align="center">

单选 <select>
 <option >jerry</option>
 <option >calm</option>
 <option >superman</option>
</select>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;多选（按住ctrl） <select multiple="multiple">
 <option >jerry</option>
 <option >calm</option>
 <option >superman</option>
</select>

</div>

---

# 按钮

## 使用&lt;input>标签

`<input>`标签中的 type 属性，设置为 `button`是 按钮

```html
<input type="button" value="我是按钮">
```

* **注意**：普通按钮不具备在表单中提交的功能，如要提交，用`type="submit"`，见下文。

`<input>`标签中的 type 属性，设置为 `reset`，可以把输入框的改动复原

```html
<input type="reset" value="重置">
```

## 使用&lt;button>标签

`<button></button>`即按钮标签，与`<input type="button">`不同的是，`<button>`标签功能更为丰富。按钮标签里的内容可以是文字也可以是图像

- 如果`<button>`的`type="submit"` ，那么它就具备提交form的功能

```html
<button><img src="http://how2j.cn/example.gif"/></button>
```

效果：    <button><img src="http://how2j.cn/example.gif"/></button>

---

# 表单(form)

`<form>` 用于向服务器提交数据，比如账号密码

- `action`属性表示要提交的服务器页面地址
- `method`属性表示HTTP方法（get or post）

```html
<form method="get" action="http:/127.0.0.1/login.jsp">
账号：<input type="text" name="name"> <br/>
密码：<input type="password" name="password" > <br/>
<input type="submit" value="登录">
<input type="reset" value="重置">
</form>
```

<form action="http:/127.0.0.1/login.jsp">
账号：<input type="text" name="name"> &nbsp; 密码：<input type="password" name="password" > &nbsp; <input type="submit" value="登录"> &nbsp; <input type="reset" value="重置">
</form>

<br/>
> 通常，我们提交一些文本数据，可以用 get 方法，而且数据会在浏览器地址栏显示出来。但是假若我们想提交二进制数据，比如上传文件，就必须用post。此外，为了安全起见，登录功能也要用post

* 如果要提交图片，把 input 标签的type属性改为 image

## 表单中 id 跟 name 的区别

name是传到服务器被服务器识别的

id是HTML中该元素的唯一标识

---

# 文本域

`<textarea>`标签表示文本域

- `cols` 和 `rows `表示宽高

```html
<textarea cols="15" rows="5">abc
def
</textarea>
```
<textarea cols="15" rows="4">hello world, and hello again</textarea>

---

# 如何去除两个紧邻元素的空格？

当我们想让两张图片紧挨着，如：

```html
<img id="1" src="../static/icon/star_hollow_hover.png">
<img id="2" src="../static/icon/star_hollow_hover.png">
```

却发现中间多了一个空格。去除方法是：在父div上添加`font-size: 0`

```html
<div id="img_group" style="font-size: 0;">
    <img id="1" src="../static/icon/star_hollow_hover.png">
    <img id="2" src="../static/icon/star_hollow_hover.png">
</div>
```

---


- 参考：[Basic HTML Codes for Beginners](https://www.websiteplanet.com/blog/html-guide-beginners/)
