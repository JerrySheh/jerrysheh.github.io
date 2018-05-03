---
title: Java简明笔记（七） 异常和断言
comments: true
categories: JAVA
tags: Java
abbrlink: a3bb075d
date: 2018-02-06 00:03:34
---

《Core Java for the Impatient》简明笔记。

本章要点：
* 抛出异常时，控制权转移到最近的异常处理器
* Java中，编译器负责跟踪checked exception
* 在正常执行完之后或者当发生异常时，try-with-resources语句会自动关闭资源

---

<!-- more -->

# 异常处理

在Java异常处理中，一个方法可以通过`抛出(throw)`异常来发出一个严重问题的信号。调回链中的某个方法，负责`捕获（catch）并处理`异常。

异常处理的根本优点是，将错误检测和错误处理的过程解耦（拆分）。

Java 的异常都派生自 Throwable 类，Throwable 又分为 Error 和 Exception。Error不是我们开发的应用程序所能够处理的，比如系统内存耗尽。

我们能预知并处理的错误属于 Exception。 Exception又分为 unchecked exception 和 checked exception。 unchecked exception 属于 RuntimeException 的子类。

> 当然，所有的异常都发生在运行时（Runtime），但是 RuntimeException 派生的子类异常在编译时不会被检查。

![](../../../../images/Java/Throwable.png)

我们所要关注的是一般情况下错误可被提前预知的checked exception，什么是可被提前预知？比如IO操作文件可能损坏或不存在，网络操作的时候网络可能会断开...

而那些unchecked exception，我们完全可以在程序中避免。比如遇到空指针异常，我们完全可以在代码中确保没有引用null值，通过修改代码来避免抛出这个异常。但是文件不存在或网络断开这种就不是我们逻辑代码的问题了，应该抛出异常。

许多异常类派生自 IOException，我们应该尽可能用准确合适的异常类来报告错误。比如在某个路径查找指定文件时，却无法找到，此时应该抛出 FileNotFoundException。

---

# checked exception的声明

假如有一个方法，我们能够预料到它可能会抛出 IOException 和 ReflectiveOperationException 这两种异常，那么我们可以在方法中这样声明：

```java
public void write (Object obj, String filename) throws IOException, ReflectiveOperationException {
  ...
}
```

* Override覆盖的方法不能抛出超出父类异常范围的异常，如果父类没有throws异常，则子类不可以抛出checked exception
* 当一个方法抛出异常时，可以用javadoc的 `@throws` 标签来文档化
* 不可能指定lambda表达式的异常类型

---

# 异常捕获

## 示例1：可以捕获多个异常

```java
try {
  //statements
} catch (ExceptionClass1 ex) {
  //handler1
} catch (ExceptionClass2 ex) {
  //handler2
} catch (ExceptionClass3 ex) {
  //handler3
} finally {
  //statements
}
```

## 示例2：多个捕获共享一个handler

```java
try {
  //statements
} catch (ExceptionClass1 | ExceptionClass2 | ExceptionClass3 ex) {
  //handler3
} finally {
  //statements
}
```

## 示例3：带资源的异常捕获（try-with-resource）

try后面接资源，在正常执行完之后或者当发生异常时，try-with-resources语句会自动关闭资源。

这样我们不用写`out.close()`，但却能够保证每个资源的`out.close()`都会被触发。

```
ArrayList<String> lines ...;
try (PrintWriter out = new PrintWriter("output.txt")) {
  for (String line:lines ) {
    out.println(line.toLowerCase());
  }
} catch (IOException ex) {
  //handler
} finally {
  //statements
}
```

如果没有 try-catch 的话，如果其中一个 line 抛出异常，那么所有的 line 的 `out.close()` 不能被正常执行，导致out结果丢失。

如果用常规的 try-catch 语句，如果要打开两个资源，那么就要嵌套 try-catch 了。try-with-resources的一个好处在于只需要写一个 try-catch 语句。

更多关于异常的内容 异常重抛和链接、堆栈踪迹、Objects.requireNonNull方法见书p186

---

# try里有return，finally还执行么？

答案：执行，并且finally的执行早于try里面的return

结论：
* 不管有没有出现异常，finally块中代码`都会执行`；
* 当try和catch中有return时，finally仍然会执行；
* finally是在return后面的表达式运算后执行的（此时并没有返回运算后的值，而是先把要返回的值保存起来，不管finally中的代码怎么样，返回的值都不会改变，任然是之前保存的值），所以函数返回值是在finally执行前确定的；
* finally中最好不要包含return，否则程序会提前退出，返回值不是try或catch中保存的返回值。

> 一句话总结: 先执行return后面的表达式，把结果保存起来，但不返回，然后执行finally，最后才返回。不要在finally中包含return。

---

# 断言（assert）

断言机制允许我们在测试时加入检测条件，并且在生产代码中自动移除它们。在Java中，断言用于调试目的以验证内部假设。

为了断言 x 是一个非负数
```java
assert x >= 0;
```

或者将 x 的实际值传进 AssertionError 对象，这样后面就可以现实它：
```java
assert x >=0 : x;
```

默认情况下断言是被禁用的，在运行程序时加上 -ebableassertion 或者 -es可以启用断言

```
$ java -ea MainClass
```

不必重新编译程序，当断言被禁用时，类加载器会清除断言代码，所以断言不会降低运行速度。
