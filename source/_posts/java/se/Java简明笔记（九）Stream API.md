---
title: Java简明笔记（九）Stream API
comments: false
abbrlink: 372345f
date: 2018-02-26 00:07:29
categories:
- Java
- Java SE
tags: Java
---

# Stream

Java 中的 Stream 提供了数据源，让你可以在比集合类更高的概念层上指定操作。**使用 Stream，只需要指定做什么，而不是怎么做**。你只需要将操作的调度执行留给实现。

简单地说，流就是一组数据，经过某种操作，产生我们所需的新流，或者输出成非流数据。

流的来源，可以是集合，数组，I/O channel， 生成器（generator）等。流的聚合操作类似 SQL 语句，比如filter, map, reduce, find, match, sorted等。

例如，从文件从获取流：

```java
try (Stream<String> lines = Files.lines(Paths.get("/path/to/file.txt"))) {
    ...
}
```

<!-- more -->

## 从迭代到 Stream 操作

假设现在有一本电子书`alice.txt`在我们的硬盘里，我们想统计这本书中所有的长单词（超过12个字母），我们可以用迭代的方法。

1. 第一步，先将 alice.txt 所有内容读到字符串里
2. 第二步，创建一个List列表，以非字母为分隔符存放每一个单词字符串
3. 第三步，foreach循环开始迭代

```java
// 读文件，放到 String 里
String contents = new String(readAllBytes((Paths.get("alice"))), StandardCharsets.UTF_8);
// 以非字母为分隔符
List<String> words = Arrays.asList(contents.split("\\PL+"));

int count = 0;
// 在 List 里面迭代，如果找到长度＞12的，计数器+1
for (String w : words) {
    if (w.length() > 12) count++;
    }
```

在 java 8 中，可以用 stream 来实现相同的功能：

```java
// 读文件，放到 String 里
String contents = new String(readAllBytes((Paths.get("alice.txt"))), StandardCharsets.UTF_8);
// 以非字母为分隔符
List<String> words = Arrays.asList(contents.split("\\PL+"));

// 把 List 转换成 流，用 flilter 方法对流的每一个元素进行判断，筛选出＞12的，并计数
long count1 = words.stream().filter(w -> w.length() > 12).count();
```

* `words.stream()`创建的是串行流，`words.parallelStream()`创建的是并行流。

只需要一行，就把过滤字母长度大于12的单词和统计实现出来了。

Stream就是这样遵循 **做什么，而不是怎么去做** 的原则。

---

# 聚合操作（Aggregation）

简单介绍filter, map, reduce, find, match, sorted

1. **filter**: 过滤符合的条件,如在集合里面过滤长度大于5的元素`.filter(w -> w.length() > 5)`
2. **map**：用于映射每个元素到对应的结果，如将每个元素乘方`.map( i -> i*i)`
3. **reduce**：把结果继续和序列的下一个元素做累积计算（第一个参数是起始值）
4. **find**：查找
5. **anyMatch**：匹配，判断的条件里，任意一个元素成功，返回true
6. **allMatch**：判断条件里的元素，所有的都是，返回true
7. **noneMatch**：跟 allMatch 相反
8. **sorted**：排序
9. **limit**：取集合的前 n 个元素


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

比如上面例子的`.count()`，就把流转换成了数字，`.collect(Collectors.toList()`转换成 List 集合， `.max()`和`.min()`获取成流中最大或最小的值。`findFirst()`返回非空集合的第一个值，`findAny()`返回任何符合的值。`anyMatch()`、`noneMatch()`和`allMatch()`返回匹配。

例子：流中是否有以Q开头的元素？有返回True，没有返回False

```java
boolean aWordStartWithQ = words.parallel().anyMatch( s -> s.startWith("Q"));
```

## Collectors

Collectors实现了很多规约操作，例如

1. `.collect(Collectors.toList()`把流转换成 List
2. `.collect(Collectors.joining(",")`把流转换成以逗号分割的 String

---

# parallel stream

parallelStream是并行执行的流，是以多线程的方式运行的。其原理是ForkJoinPool（实现了Executor和ExecutorService接口），主要用分治法(Divide-and-Conquer Algorithm)来解决需要使用相对少的线程处理大量的任务的问题（比如使用4个线程来完成超过200万个任务，任务之间有父子关系），这一点是 ThreadPoolExecutor 做不到的。

> 提示：当需要处理递归分治算法时，考虑使用ForkJoinPool。

---

# stream or parallelStream

使用串行流还是并行流，主要考虑：

## 考虑1：是否需要并行？  

在回答这个问题之前，你需要弄清楚你要解决的问题是什么，数据量有多大，计算的特点是什么？并不是所有的问题都适合使用并发程序来求解，**比如当数据量不大时，顺序执行往往比并行执行更快**。毕竟，准备线程池和其它相关资源也是需要时间的。但是，当任务**涉及到I/O操作并且任务之间不互相依赖时**，那么并行化就是一个不错的选择。通常而言，将这类程序并行化之后，执行速度会提升好几个等级。

## 考虑2：任务之间是否是独立的？是否会引起任何竞态条件？  

对于问题2，如果任务之间是独立的，并且代码中不涉及到对同一个对象的某个状态或者某个变量的更新操作，那么就表明代码是可以被并行化的。

## 考虑3：结果是否取决于任务的调用顺序？  

对于问题3，由于在并行环境中任务的执行顺序是不确定的，因此对于依赖于顺序的任务而言，并行化也许不能给出正确的结果。

---

参考：

- 《写给大忙人看的Java核心技术》
- [Java Functional Programming](http://tutorials.jenkov.com/java-functional-programming/streams.html)
- [深入浅出parallelStream](https://www.jianshu.com/p/bd825cb89e00)
