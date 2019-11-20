---
title: Effective Java（四）泛型
abbrlink: 53a4cf82
date: 2019-11-20 21:38:49
categories: Effective Java
tags: Java
---

Java 5 加入了泛型。在有泛型之前，你必须转换从集合中读取的每个对象。如果有人不小心插入了错误类型的对象，则在运行时可能会失败。使用泛型，你告诉编译器在集合中允许存放哪些类型的对象。编译器会自动插入强制转换，并在编译时告诉你是否尝试插入错误类型的对象。

---

# Item 26 使用泛型，不要使用原始类型

如果使用原始类型的集合，在一个字符串集合里插入一个数字是合法的，可能到运行时才出现问题。

```java
List names = ...;
names.add("jerry");
names.add(1.0);

for (Iterator i = names.iterator(); i.hasNext(); )
    String s = (String) i.next(); // Throws ClassCastException

```

但是如果使用泛型，当你向字符串集合插入数字或其他类型时，编译器会报错。使得问题在编译器被发现。

```java
List<String> names = ...;
names.add("jerry");
names.add(1.0); // 编译出错
```

---

# Item 27 消除 unchecked 警告

使用泛型编程时，会看到许多编译器警告：未经检查的强制转换警告，未经检查的方法调用警告，未经检查的参数化可变长度类型警告以及未经检查的转换警告。

例如，如果你写：

```java
Set<Lark> exaltation = new HashSet();
```

编译器会发出 unchecked conversion 警告，修改如下即可消除（JDK 1.7）：

```java
Set<Lark> exaltation = new HashSet<>();
```

尽可能的消灭这些警告，以保证安全。如果你无法消除，但可以确定代码是安全的，可以用 `@SuppressWarnings("unchecked")` 来抑制警告。使用时，请添加注释，说明为什么是安全的。

---

# Item 28 List 优于 数组

未完待续
