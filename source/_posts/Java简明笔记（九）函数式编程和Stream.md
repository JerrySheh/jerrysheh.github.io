---
title: Java简明笔记（九）函数式编程和Stream
comments: false
abbrlink: 372345f
date: 2018-02-26 00:07:29
categories: JAVA
tags: Java
---

# 函数式编程

在聊 Java 的 Stream（流）之前，先来谈谈什么是函数式编程。

我们平时所采用的 **命令式编程**（OO也是命令式编程的一种）关心解决问题的步骤。你要做什么事情，你得把达到目的的步骤详细的描述出来，然后交给机器去运行。

而函数式编程关心数据的映射，或者说，函数式编程关心类型（代数结构）之间的关系。这里的映射就是数学上“函数”的概念——一种东西和另一种东西之间的对应关系。函数式编程的思维就是如何将这个关系组合起来，用数学的构造主义将其构造出你设计的程序。

用计算来表示程序, 用计算的组合来表达程序的组合。

## 函数式编程思想

函数式编程有三个关键点：
1. **函数第一（Functions as first class objects）**：函数跟其他对象一样，一个引用变量可以指向一个函数，就像我们声明一个引用 s 指向一个字符串 String 一样。可惜的是，在 Java 中，函数不是第一的，但 Scala 里面是。
2. **纯函数（Pure functions）**：函数内部不依赖于外部变量。
3. **高阶函数（Higher order functions）**：函数可以作为参数传递进来，也可以作为返回值返回。在 Java 中，一个方法可以接受一个 lambda 表达式，也可以返回一个 lambda 表达式。

纯函数的四个关键点：
1. **无状态(No state)**：函数内部不能使用外部变量。
2. **无副作用（No side effects）**：函数内部不能修改外部变量。
3. **对象不可变（Immutable variables）**：使用不可变对象来避免副作用。如果要修改一个传进来的参数对象，那修改完毕后返回一个新的对象，而不是修改后的该对象本身。
4. **递归（Favour recursion over looping）**：使用递归，而非循环。

<!-- more -->

---

# Java中的函数式接口

## 自定义函数式接口

**只有一个未实现的抽象方法的接口称为函数式接口，static 和 default 不影响**。之所以规定不能有多个抽象方法，是因为 lambda 表达式只能接受一个方法。用`@FunctionalInterface`注解检查是否符合函数式接口规范。

```java
// 函数式接口
@FunctionalInterface
public interface Cal {
    int cal(int n1, int n2);
}
```

Cal是一个函数式接口，只有一个 cal 方法。使用的时候可以用 lambda 表达式定义方法做什么。

```java
public static void main(String[] args) {

    // 做加法
    Cal sum = (n1, n2) -> n1 + n2;
    int r1 = sum.cal(10, 20);
    System.out.println(r1); // 30

    // 做减法
    Cal sub = (n1, n2) -> n1 - n2;
    int r2 = sub.cal(10, 20);
    System.out.println(r2); // -10

}
```

当然，我们可以再做一层封装，提供 calculator 方法，接收两个数字和一个表示如何计算的 lambda，返回计算结果。

```java
public static int calculator(int num1, int num2, Cal c){
    return c.cal(num1, num2);
}

public static void main(String[] args) {
    int n = calculator(10, 20, (n1, n2)-> n1 + n2); // 30
}
```

可以看到，calculator 方法传入了一个 lambda 表达式，事实上，这个 lambda 就是 Cal 接口的一个匿名实现，在传参的时候“现场”声明罢了。

## Java预置的函数式接口

jdk 1.8 预置了一些函数式接口，在 java.util.function 包里。

### Consumer<T>

接收一个对象 T， 无返回。

```java
Consumer<Double> cal = (d) -> System.out.println(d*2);
cal.accept(3.5);
```

### Supplier<T>

不接收参数，返回一个对象 T

### Predicate<T>

接收一个对象T，返回 boolean

### Function<T,R>

接收一个对象T，返回一个对象R

---

# Stream

Java 中的 Stream 提供了数据源，让你可以在比集合类更高的概念层上指定操作。**使用 Stream，只需要指定做什么，而不是怎么做**。你只需要将操作的调度执行留给实现。

简单地说，流就是一组数据，经过某种操作，产生我们所需的新流，或者输出成非流数据。

流的来源，可以是集合，数组，I/O channel， 生成器（generator）等。流的聚合操作类似 SQL 语句，比如filter, map, reduce, find, match, sorted等。

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
3. **reduce**：把结果继续和序列的下一个元素做累积计算
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

参考：

- 《写给大忙人看的Java核心技术》
- [函数式编程](https://coolshell.cn/articles/10822.html)
- [Java Functional Programming](http://tutorials.jenkov.com/java-functional-programming/index.html)
