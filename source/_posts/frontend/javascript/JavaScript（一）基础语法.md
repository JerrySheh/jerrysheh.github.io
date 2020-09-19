---
title: JavaScript（一）基础语法
comments: true
categories: 前端
tags:
  - JavaScript
  - Web
abbrlink: 21eaf7cd
date: 2018-04-02 11:07:25
---

JavaScript用于网页和用户之间的交互，比如提交的时候，进行用户名是否为空的判断。

完整的 Javascript 由以下三个部分组成：
- 语言基础
- BOM（Browser Object Model 浏览器对象模型）
- DOM（Document Object Model 文档对象模型）

<!-- more -->

---

# 基本语法

## 标签

JavaScript 必须写在 `<script>` 里

```html
<html>
  <head>
   <script>
      document.write("这是 javascript");
   </script>
  </head>
</html>
```
> 在文档加载结束后使用 document.write，会覆盖整个HTML的内容

也可以在HTML中引用 .js 文件

```html
<html>
  <script src="http://jerrysheh.com/study/hello.js"></script>
</html>
```

## 变量

用 var 声明变量，关键字`var` 也可以省略。但省略就是全局变量，所以不建议省略。

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
- JavaScript 中，即使是基本数据类型，也有属性和方法

当不确定某个变量是什么类型的时候，可以用使用`typeof`来进行判断数据类型

```js
var name = 'jerry'
typeof name // string
```

JavaScript中的变量类型是动态的

```JavaScript
var x                // x 为 undefined
var x = 6;           // x 为数字
var x = "Bill";      // x 为字符串
```

### 类型转换

使用内置函数`toString()`转换为字符串。

比如说，我们可以对 Number 或者 Boolean 执行 `toString()`方法。

将a转换为16进制字符串

```js
var a = 18
a.toString(16)
```

对于有Null值的情况，用`String()`会返回null，用`toString()`报错。

- 使用内置函数 `parseInt()`、`parseFloat()`或 `Number()`转换为数字
- `parseInt`会一直定位数字，直到出现非字符。 所以`10abc`会被转换为`10`。但是`abc10`返回NaN，如果没有数字，也返回NaN。
- `Number()`必须为纯数字，如果参杂其他字符，直接返回NaN。

使用内置函数`Boolean()`转换为Boolean值
- 当转换字符串时：非空即为true
- 当转换数字时： 非0即为true
- 当转换对象时：非null即为true

### 绝对等运算符

JavaScript的运算符跟 Java 区别不大，但是有一点要注意：

- `==`是等于运算符，表示：值相等（类型不一定相等）
- `===`是绝对等运算符，表示：类型相等，值也相等

例如，数字1和字符串1，值都是1，但是类型是不一样的

---

# 函数和语句

## 函数

定义了函数，必须调用才能执行。return、作用域等跟 Java 类似。不再赘述。

```html
<script>
function print(message){
  document.write(message);
}
print("hello");
print("<br>");
print("world");
</script>
```

输出
```
hello
world
```

> 在写前端文件的时候，通常把 JavaScript 脚本写在最后面，以提高网页的加载速度。当需要在加载时立即执行 JavaScript 代码，可以用 `onload="jfun()"` 属性。其中 `jfun()` 是 JS 函数名称。

## 语句

跟Java类似， JavaScript也支持 **条件语句**、**循环语句**、**try-catch语句**

其中循环语句还支持 forEach 循环。 但是跟 Java 的 forEach 有点不一样

```javascript
var strs = ["America","Greece","Britain","Canada","China","Egypt"];
var count = 0;
for (var str in strs) {
    if (strs[str].match("a") != null) {
        count++;
    }
}
```

这里的str，是strs的下标，因此要表达数组的每个元素，用`strs[str]`

continue、break 什么的当然也是存在的啦

---

# 事件

用户任何对网页的操作，都会产生一个事件。

事件有很多种，比如鼠标移动，鼠标点击，键盘点击等等。

## 鼠标点击事件

```html
<script>
function showHello(){
   alert("Hello JavaScript");
}
</script>

<button onclick="showHello()">点击一下</button>
```

效果如下

<script>
function showHello(){
   alert("Hello JavaScript");
}
</script>

<button onclick="showHello()">点击有惊喜</button>

---

# 对象

对象是有属性和方法的一种特殊数据类型。

常见的对象有数字Number，字符串String，日期Date，数组Array等。

跟 Java 一样，可以通过 new 来创建一个对象

```html
<script>
    var x = "5";
    var y = new String(x);
</script>
```

## 常用的数字方法

关键字	|简介
---|---
new Number|	创建一个数字对象
属性MIN_VALUE | 最小值
属性MAX_VALUE	| 最大值
属性NaN|	表示不是数字
方法toFixed|	返回一个数字的小数表达
方法toExponential|	返回一个数字的科学计数法表达
方法valueOf|	返回一个数字对象的基本数字类型

## 常用的字符串方法

关键字	|简介
---|---
new String()|	创建字符串对象
属性 length|	字符串长度
方法 charAt charCodeAt|	返回指定位置的字符
方法 concat|	字符串拼接
方法 indexOf lastIndexOf	|子字符串出现的位置
方法 localeCompare|	比较两段字符串是否相同
方法 substring	|截取一段子字符串
方法 split	|根据分隔符，把字符串转换为数组
方法 replace	|替换子字符串

## 常用的数组方法

关键字|简介
---|---
new Array|	创建数组对象
属性 length|	数组长度
for / for in|	遍历一个数组
方法 concat|	连接数组
方法 join	|通过指定分隔符，返回一个数组的字符串表达
方法 push pop|	分别在最后的位置插入数据和获取数据(获取后删除)
方法 unshift shift|	分别在最开始的位置插入数据和获取数据(获取后删除)
方法 sort|	对数组的内容进行排序
方法 sort(comparator)|	自定义排序算法
方法 reverse|	对数组的内容进行反转
方法 slice	|获取子数组
方法 splice|	删除和插入元素

## 常用的数学方法

关键字	|简介
---|---
属性E PI	|自然对数和圆周率
方法 abs|	绝对值
方法 min max|	最小最大
方法 pow	|求幂
方法 round	|四舍五入
方法 random|	随机数

## 自定义对象

### 直接定义对象

```javascript
var person={
firstname : "Bill",
lastname  : "Gates",
id        :  5566
};

var n = person.lastname;
```

### 函数封装对象

```JavaScript
function Hero(name){
  this.name = name;
  this.go = function(){
     document.write(this.name + "正在前进<br>");
  }
}

var gareen = new Hero("盖伦");
gareen.go();

var teemo = new Hero("提莫");
teemo.go();
```

用 prototype 实现为已存在的对象增添新的方法，例如为 `Hero` 增添 `back` 方法

```javascript
Hero.prototype.back = function(){
  document.write(this.name + "正在后退<br>");
}

gareen.back();
```

---

# 日期格式化函数

```JavaScript
getDate:function (strDate) {
    var d = new Date(strDate);
    return d.getFullYear() + '-' + (d.getMonth() + 1) + '-' + d.getDate() + ' ' + d.getHours() + ':' + d.getMinutes() + ':' + d.getSeconds();
}
```
