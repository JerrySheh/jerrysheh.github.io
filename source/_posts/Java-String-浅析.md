---
title: Java String 浅析
comments: true
categories: JAVA
tags: Java
abbrlink: 689b9445
date: 2018-02-05 00:27:20
---

本文探讨以下几个问题
* String str=new String("abc")创建了几个对象？
* String 格式化输出
* String 的一些常见用法
* StringBuffer 和 StringBuilder


---

# String str=new String("abc")创建了几个对象？

答案是：2 个

首先先来看一下常规的`String str = "abc";`

在Java虚拟机（JVM）中存在着一个字符串池，其中保存着很多String对象，并且可以被共享使用，因此它提高了效率。由于String类是final的，它的值一经创建就不可改变，因此我们不用担心String对象共享而带来程序的混乱。 在`String str = "abc"`语句中，String类会先去字符串池寻找`"abc"`这个对象，如果`abc`存在，则把它的引用给str，如果`"abc"`不存在，则创建`"abc"`对象。


再看String类的构造方法中，其中一个是

```java
/*
 * 初始化一个新创建的 String 对象，使其表示一个与参数相同的字符序列；
 * 换句话说，新创建的字符串是该参数字符串的副本。
 */
String(String original) {
  ...
}
```

可以发现，被调用的构造器方法接受的参数也是一个String对象。也就是说，`String str=new String("abc")`语句中，String类会先去字符串池寻找`"abc"`，发现`"abc"`不存在，于是创建`"abc"`这个对象，然后把`"abc"`作为构造方法的参数，传给String构造器`new String("abc")`相当于新创建了参数字符串的副本，于是又创建了一个对象。

只是，第一个`"abc"`对象是存在于字符串池当中的，第二个跟其他对象一样存在于内存当中。

<!-- more -->

---

# String 格式化输出

两个例子

```java
String fs;
fs = String.format("浮点型变量为%f, 整型变量为%d, 字符串变量为%s", floatVar, intVar, stringVar);

String hello
hello = String.format("Hello, %s. Next year you will be %d.", name, age);
```

---

# String 的几个用法

这些用法在 [Java简明笔记（一）](https://jerrysheh.github.io/post/b3088ac5.html)中提及过，转抄如下

* `String name = String.join("-","hello","and","again"); `，输出 hello-and-again 。 第一个参数是连接符，第二到n个参数是需要连接的字符串

* `str.substring(7,12)`提取子字符串，如`Hello, World!` 第7（包括）到第12（不包括）位，即`World`这个单词。

* `str.split(" ")`，以空格为分隔符，将子字符串提取出来。最终结果为一个字符串数组。

* `str.equals("World")`，判断相等。

* 不要用 `==` 符号来判断字符串相等！！在Java虚拟机中，每个文字串只有一个实例，`"World" == "World"` 确实会返回真，但是如果前后比较的字符串是用分割提取等方法获取到的，它将会被存入一个新的对象当中，这时用==判断会出现假，因为不是同一个对象。

* `String middlename = null;` ，测试一个字符串对象是否为null，可以用`==`。

* null说明该变量没有引用任何对象。空字符串 `""`是长度为零的字符串， null不是字符串

* `if ("World".equals(location))` 是个好习惯， 把文字串放在前面

* `equalsIgnoreCase`方法，不考虑大小写比较字符串，`myStr.equalsIgnoreCase("world");`

* 数字转字符串，`str = Integer.toString(n,2)`，如果n是42，则把42转为二进制字符串 “101010”，第二个参数默认为10进制，范围在[2,36]

* 字符串转数字，`n = Integer.parseInt(str，2)`

---

# StringBuffer 和 StringBuilder

当对字符串进行修改的时候，需要使用 StringBuffer 和 StringBuilder 类。

和 String 类不同的是，StringBuffer 和 StringBuilder 类的对象能够被多次的修改，并且不产生新的未使用对象。

StringBuilder 类在 Java 5 中被提出，它和 StringBuffer 之间的最大不同在于 StringBuilder 的方法不是线程安全的（不能同步访问）。

由于 StringBuilder 相较于 StringBuffer 有速度优势，所以多数情况下建议使用 StringBuilder 类。然而在应用程序要求线程安全的情况下，则必须使用 StringBuffer 类。 （[以上文字转载出处](http://www.runoob.com/java/java-stringbuffer.html)）

总结起来就是： `StringBuilder比StringBuffer快，但涉及线程安全必须用StringBuffer。它们两者与 String 的不同点在于对象能被多次修改。`

## StringBuffer 方法

StringBuffer有跟String类似的方法，列举几个

* `public StringBuffer append(String s)` 追加字符串

```java
StringBuffer s = new StringBuffer("hello world，");
s.append("I am");
s.append("Jerry.");
System.out.println(s);
```

输出结果： hello world，I am Jerry.

反转
```java
public StringBuffer reverse()
```

删除start和end之间的字符
```java
public delete(int start, int end)
```

将 int 参数的字符串表示形式插入此序列中。
```java
`public insert(int offset, int i)`
```

取代
```java
replace(int start, int end, String str)
```
