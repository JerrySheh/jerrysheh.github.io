---
title: Java简明笔记（九）Stream
comments: true
abbrlink: 372345f
date: 2018-02-26 00:07:29
categories: JAVA
tags: Java
---

# 函数式编程

在聊 Java 的 Stream（流）之前，先来谈谈什么是`函数式编程`。

我们平时所采用的`命令式编程`关心解决问题的步骤。你要做什么事情，你得把达到目的的步骤详细的描述出来，然后交给机器去运行。

而函数式编程关心数据的映射，或者说，函数式编程关心类型（代数结构）之间的关系。这里的映射就是数学上“函数”的概念——一种东西和另一种东西之间的对应关系。函数式编程的思维就是如何将这个关系组合起来，用数学的构造主义将其构造出你设计的程序。

简单的说，就是用计算来表示程序, 用计算的组合来表达程序的组合。

---

# Stream

Java 中的 Stream 提供了数据源，让你可以在比集合类更高的概念层上指定操作。使用 Stream，只需要指定做什么，而不是怎么做。你只需要将操作的调度执行留给实现。

简单地说，流就是一组数据，经过某种操作，产生我们所需的新流。

<!-- more -->

流的来源，可以是集合，数组，I/O channel， 生成器（generator）等。
聚合操作类似SQL语句一样的操作， 比如filter, map, reduce, find, match, sorted等。

## 从迭代到 Stream 操作

假设现在有一本电子书`alice.txt`在我们的硬盘里，我们想统计这本书中所有的长单词（超过12个字母），我们可以用迭代的方法。

* 第一步，先将书的内容读到String里
* 第二步，创建一个List列表 words，以非字母为分隔符
* 第三步，foreach循环开始迭代

```java
try {
    String contents = new String(readAllBytes((Paths.get("alice"))), StandardCharsets.UTF_8);
    List<String> words = Arrays.asList(contents.split("\\PL+"));
    int count = 0;
    for (String w :
            words) {
        if (w.length() > 12) count++;
    }
} catch (IOException e){
    System.out.println("IO problem");
}
```

上面的迭代显得有些繁琐，我们可以用流的方法实现

```java
try {
    String contents = new String(readAllBytes((Paths.get("alice.txt"))), StandardCharsets.UTF_8);
    List<String> words = Arrays.asList(contents.split("\\PL+"));
    long count1 = words.stream().filter(w -> w.length() > 12).count();
} catch (IOException e){
    System.out.println("IO problem");
}
```

* `words.stream()`创建的是串行流，`words.parallelStream()`创建的是并行流。

只需要一行，就把过滤字母长度大于12的单词和统计实现出来了。

Stream就是这样遵循“做什么，而不是怎么去做”的原则。

---

# 聚合操作

简单介绍filter, map, reduce, find, match, sorted

* **filter**: 过滤符合的条件,如`.filter(w -> w.length() > 5)`
* **map**：用于映射每个元素到对应的结果，如`.map( i -> i*i)`
* **reduce**：把结果继续和序列的下一个元素做累积计算
* **find**：查找
* **match**：匹配
* **sorted**：排序


关于聚合操作，可参考： [runoob.com](http://www.runoob.com/java/java8-streams.html)

一个例子: 将`alice.txt`的内容读入 String， 以非字母为分隔符存入 List， 通过流取前20个值，过滤出这20个值长度大于5的，并排序，最后存到新的 List 里

```java
public static void streamTest() {
    try {
        String contents = new String(readAllBytes((Paths.get("alice.txt"))), StandardCharsets.UTF_8);
        List<String> words = Arrays.asList(contents.split("\\PL+"));
        List<String> newwords = words.stream().limit(20).filter(w -> w.length() > 5).sorted().collect(Collectors.toList());
        System.out.println(newwords);
    } catch (IOException e) {
        System.out.println("IO problem");
    };
}
```

另一个例子：为每个订单加上12%的税

```java
// 不使用lambda表达式
List costBeforeTax = Arrays.asList(100, 200, 300, 400, 500);
for (Integer cost : costBeforeTax) {
    double price = cost + .12*cost;
    System.out.println(price);
}

// 使用lambda表达式
List costBeforeTax = Arrays.asList(100, 200, 300, 400, 500);
costBeforeTax.stream().map((cost) -> cost + .12*cost).forEach(System.out::println);
```

可见 Lambda 表达式非常地优雅。

---

# 规约方法（reduction）

有时候我们使用聚合操作，操作完成后还是一个流。但有时会转换成非流值，我们把转换完毕后是非流值的方法称为规约方法。

比如上面例子的`.count()`，就把流转换成了数字，`.collect(Collectors.toList()`转换成 List 集合， `.max()`和`.min()`转换成流中最大或最小的值。`findFirst()`返回非空集合的第一个值，`findAny()`返回任何符合的值。`anyMatch()`、`noneMatch()`和`allMatch()`返回匹配。

例子：流中是否有以Q开头的元素？有返回True，没有返回False

```java
boolean aWordStartWithQ = words.parallel().anyMatch( s -> s.startWith("Q"));
```
