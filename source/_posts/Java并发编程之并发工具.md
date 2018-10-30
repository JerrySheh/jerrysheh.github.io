---
title: Java并发编程之并发工具
comments: true
date: 2018-10-30 15:08:26
categories: JAVA
tags: Java
---

Java自带的平台类库里面包含了很多有用的工具，来帮助我们更好地处理并发问题。例如，线程安全的容器类、用于协调线程控制流的同步工具类。这一篇主要介绍一下这些工具。

<!-- more -->

# 同步容器类

同步容器类包括 Vector 和 Hashtable。它们实现线程安全的方式十分简单粗暴：对每个公有方法进行同步，使得每次只有一个线程能够访问容器的状态。这种线程安全方式对于容器自身来说是安全的，但在调用方可能会出现问题，因此使用时要注意调用方可能需要做一些额外的协调。例如：

```java
// 获取 Vector 最后一个元素
public static Object getLast(Vector list){
  int lastIndex = list.size() - 1;
  return list.get(lastIndex);
}

// 删除 Vector 最后一个元素
public static void deleteLast(Vector list){
  int lastIndex = list.size() - 1;
  list.remove(lastIndex);
}
```

从 Vector 的角度，无论你用多少个线程调用多少次`deleteLast()`方法，都不会让 Vector 内部出问题。然而，从调用者的角度，线程A调用getLast，线程A先观察到size为10，然后时间片切换到线程B调用deleteLast，B也观察到size为10，然后B删除了最后一个元素，然后A获取最后一个元素，发现这个元素不存在，于是抛出`ArrayIndexOutOfBoundsException`。

另一个例子是，用一个 for 循环迭代 Vector，循环到一半时，另一个线程删除了后面某些元素，迭代到后面时就会找不到元素抛出`ArrayIndexOutOfBoundsException`。

```java
// 如果迭代到一半，另一个线程删除了后面的元素，导致 get(i) 取不到
for (int i=0; i < vector.size() ;i++ ) {
  doSomething(vector.get(i));
}
```

我们通常会用 Iterator 可以对集合进行遍历，但是却 **不能在遍历过程对原集合做增、删、改**，会抛出 `ConcurrentModificationException`。这是 Java 的一种并发预警警示器，叫 fail-fast。告知我们集合在遍历过程中被修改了。

有时候，我们看起来好像没由迭代，但仍然抛出了`ConcurrentModificationException`。是因为有些方法隐式地进行了迭代，如打印一个Hashset，事实上会调用 toString 方法，这个方法不断调用 StringBuilder.append 把各个元素转为字符串，这其实就是迭代了。同理，hashCode方法、equals方法、containAll、removeAll、retainAll等方法都是如此。

因此，使用同步容器类时，需要在调用方加 synchronized 同步。

---

# 并发容器

同步容器简单粗暴地对公有方法加同步，实际上是强行将对容器状态的访问串行化了，这对并发性能带来了很大影响。在 Java 5 之后，增加了如 ConcurrentHashMap、ConcurrentLinkedQueue 这样的并发容器，天生为并发程序设计。在多线程中应该尽可能用并发容器，而不是同步容器。

## ConcurrentHashMap

未完待续
