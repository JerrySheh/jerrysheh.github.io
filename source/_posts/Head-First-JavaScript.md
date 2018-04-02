---
title: Head First JavaScript
comments: true
date: 2018-04-02 11:07:25
categories: 前端
tags:
 - JavaScript
---

JavaScript用于网页和用户之间的交互，比如提交的时候，进行用户名是否为空的判断。

完整的javascript由语言基础，BOM和DOM组成。


---

# 基本语法

## 标签

JavaScript 的必须写在 `<script> </script>` 里

```html
<html>
  <head>
   <script>
      document.write("这是 javascript");
   </script>
  </head>
</html>
```

可以在HTML中引用 .js 文件

```html
<html>
  <script src="http://jerrysheh.com/study/hello.js"></script>
</html>
```

## 变量

用 var 声明变量， var 也可以省略

```html
<script>
  var x = 10;
  document.write("变量x的值:"+x);
</script>
```

## 基本数据类型

关键字 |	简介
--|--
undefined	 |声明了但未赋值
Boolean	|布尔
Number	|数字
String	|字符串
var	|动态类型
typeof	|变量类型判断
null	| 空对象/对象不存在

- JavaScript 中单引号和双引号都表示字符串
- 当不确定某个变量是什么类型的时候，可以用使用typeof来进行判断数据类型

JavaScript中，即使是基本数据类型，也有属性和方法

> 比如说，我们可以对 Number 或者 Boolean 执行 `toString()`方法。 用法：a.toString(16)，将a转换为16进制字符串

javascript分别提供内置函数 `parseInt()`和`parseFloat()`，转换为数字

> parseInt会一直定位数字，直到出现非字符。 所以"10abc" 会被转换为 10

使用内置函数Boolean() 转换为Boolean值
- 当转换字符串时：非空即为true
- 当转换数字时： 非0即为true
- 当转换对象时：非null即为true
