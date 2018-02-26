---
title: Java简明笔记（八）Lambda表达式
comments: true
date: 2018-02-25 16:55:10
categories: JAVA
tags: Java
---

Lambda 表达式，也可称为闭包，或者匿名函数，它是推动 Java 8 发布的最重要新特性。

Lambda 允许把函数作为一个方法的参数（函数作为参数传递进方法中）。

使用 Lambda 表达式可以使代码变的更加简洁紧凑。

# 语法

我们用 `->` 来表达 Lambda 表达式，`->`之前是输入的参数，`->`之后是输出的结果。

```
(parameters) -> expression
或
(parameters) ->{ statements; }
```

注意：lambda 表达式只能引用 final 或 final 局部变量，这就是说不能在 lambda 内部修改定义在域外的局部变量，否则会编译错误。

# 简单例子

比较字符串长度

```java
(String first, String second) -> first.length() - second.length()
```

给出两个数字，求和

```java
(int a, int b) -> a + b;
```

如果结果无法用一个表达式写出，则用大括号，并明确 return

```java
(int a, int b) -> {
  if (a > b) return a * b;
  else return a / b;
}
```

如果没有参数，`->`前面的参数给出空括号

```java
Runnable task = () -> { for (int i = 0; i < 100; i++) do(); };
```

---

# 操作符 「::」

操作符`::`将 方法名称 与 类或对象名称分隔开。

可用于
* 类::实例方法

 比如，`String::comparaToIgnoreCase` 等同于 `(x,y) -> x.comparaToIgnoreCase(y)`

* 类::静态方法

 比如，`Objects::isNull` 等同于 `x -> Objects.isNull(x)`

* 对象::实例方法

 比如，`System.out::println` 等同于 `x -> System.out::println(x)`

## 例子

打印列表的所有元素，可以这么做
```java
list.forEach(x -> System.out.println(x));
```

但也可以直接这么做
```java
list.forEach(System.out::println);
```

* this也是可以使用的，比如，`this::equals` 等同于 `x -> this.equals(x)`

---

# 高阶函数

处理或返回函数的函数称为`高阶函数`。

往后学习到的时候再补充。
