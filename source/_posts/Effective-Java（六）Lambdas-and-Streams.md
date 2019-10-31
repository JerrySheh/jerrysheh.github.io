---
title: Effective Java（六）Lambdas and Streams
comments: false
categories: Effective Java
tags: Java
abbrlink: cc85a16e
date: 2019-10-22 22:39:14
---

不要问为什么从三直接就到六了，时间很贵，优先挑感兴趣的看呗。

# Item 42 lambda 表达式优于匿名类

Java 的 Lambda 表达式本质上就是一个匿名类。而什么是匿名类？就是在使用的时候现场 new 并实现的类。

只有一个方法的接口称为 **函数式接口（functioning interface）**，Lambda 表达式本质上就是对这样子的接口做现场实现。可以参考我之前写的：[Java简明笔记（八）Lambda和函数式编程](../post/68278ec8.html)

然而 lambda 也不是万能的，它只对函数是接口有用，如果一个接口有多个方法需要重写，那只能用匿名类。this 关键字在 lambda 中引用封闭实例，在匿名类中引用匿名类实例。如果你需要从其内部访问函数对象，则必须使用匿名类。

Lambdas 与匿名类都无法可靠地序列化和反序列化。因此，尽量少去 (如果有的话) 序列化一个 lambda (或一个匿名类实例)。如果有一个想要进行序列化的函数对象，比如一个 Comparator，那么使用一个私有静态嵌套类的实例（见 Item 24 ）。

作者建议：一行代码对于 lambda 说是理想的，三行代码是合理的最大值。 如果违反这一规定，可能会严重损害程序的可读性。

<!-- more -->

---

# Item 43 方法引用优于 lambda 表达式

lambda 比 匿名类 简洁，方法引用比 lambda 简洁。

考虑一个例子：

```java
map.merge(key, 1, (count, incr) -> count + incr);
```

第三个参数是一个 lambda，就只是求两数之和，而求和这个方法在 `Integer` 类中是存在的。所以可以直接用方法引用：

```java
map.merge(key, 1, Integer::sum);
```

Method Ref Type |	Example | Lambda Equivalent
---|---|---
Static  |	Integer::parseInt	              | str -> Integer.parseInt(str)
Bound   |	Instant.now()::isAfter         	| Instant then = Instant.now();t -> then.isAfter(t)
Unbound	| String::toLowerCase             |	str -> str.toLowerCase()
Class   | Constructor	TreeMap<K, V>::new	| () -> new TreeMap<K, V>
Array   | Constructor	int[]::new          |	len -> new int[len]

原则：如果方法引用看起来更简短更清晰，请使用它们；否则，还是坚持 lambda。

---

# Item 44 优先使用标准的函数式接口

java 8 提供了很多标准函数式接口（`java.util.Function` 有 43 个接口），其中有 6 个基本接口。当我们编写函数对象时，应该优先考虑标准接口，而不是自己定义函数式接口。

接口|	方法|	示例
---|---|---
UnaryOperator<T> |	T apply(T t)|	String::toLowerCase
BinaryOperator<T> |	T apply(T t1, T t2)|	BigInteger::add
Predicate<T>|	boolean test(T t)|	Collection::isEmpty
Function<T,R>	|R apply(T t)	|Arrays::asList
Supplier<T>|	T get()	|Instant::now
Consumer<T>|	void accept(T t)|	System.out::println

这 6 个标准接口接收相应不同的参数，返回相应不同的对象。参考：[Java简明笔记（八）Lambda和函数式编程](../post/68278ec8.html)

---

# Item 45 使用 Stream

Java 8 提供了 Stream API，其中有两个关键抽象：流(Stream)表示有限或无限的数据元素序列，流管道(stream pipeline)表示对这些元素的多级计算。常见的流的来源包括集合，数组，文件，正则表达式模式匹配器，伪随机数生成器和其他流。流中的数据可以是引用对象，或 int，long 和 double 这三种基本数据类型。

流包括转换和规约，转换把一个流转换成另一个流，规约把流转换成非流（集合，数组，数字）。流是惰性计算的，遇到规约操作才会开始计算。

流虽然简化了代码，但过度使用流也可能使程序难于阅读和维护。最好是迭代跟流结合着使用。如果不确定一个任务是通过流还是迭代更好地完成，那么尝试这两种方法，看看哪一种效果更好。

关于流的用法，参考：[Java简明笔记（九）Stream API](../post/372345f.html)

---

# Item 46 优先考虑流中无副作用的函数

流不仅仅是一个 API，它是函数式编程的范式（paradigm）。函数式编程应该尽可能使用纯函数（pure function）。纯函数的结果仅取决于其输入，不依赖于任何可变状态，也不更新任何状态。为此，传递给流操作的任何函数对象（中间操作和终结操作）都应该没有副作用。

一个建议是 forEach 操作应仅用于报告流计算的结果，而不是用于执行计算。考虑下面的代码，它只是伪装成流代码的迭代代码，并没有享受到流带来的好处。

```java
// Uses the streams API but not the paradigm--Don't do this!
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
    words.forEach(word -> {
        freq.merge(word.toLowerCase(), 1L, Long::sum);
    });
}
```

好的做法：

```java
// Proper use of streams to initialize a frequency table
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).tokens()) {
    freq = words
        .collect(groupingBy(String::toLowerCase, counting()));
}
```


---

# Item 47 优先使用 Collection 而不是 Stream 来作为方法的返回类型

如果在返回一些序列元素的方法里返回了一个流，而你想迭代，（或相反），可以用适配器将流和 iterator 互相转换。但这样会降低效率。

```java
// Adapter from  Stream<E> to Iterable<E>
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
    return stream::iterator;
}

// Adapter from Iterable<E> to Stream<E>
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
    return StreamSupport.stream(iterable.spliterator(), false);
}
```

在实践中，最好优先考虑返回集合，而不是返回一个流。如果返回集合是不可行的，则返回流或可迭代对象。

---

# Item 48 谨慎使用流并行

让我们回顾一下java的并发历史： 1996 年 java 发布 1.0 时就内置了对线程的支持，包括同步和 wait / notify 机制，java 5 加入了 `java.util.concurrent` 类库，提供了并发集合和执行器框架。Java 7 引入了 fork-join 包，这是一个用于并行分解的高性能框架。 Java 8 引入了流，可以通过对 parallel 方法的单个调用来并行化。用 Java 编写并发程序变得越来越容易，但编写正确快速的并发程序还像以前一样困难。

通常，并行在 ArrayList、HashMap、HashSet 和 ConcurrentHashMap 实例、数组、int 类型和 long 类型的流上性能提升是最好的。因为它们都可以精确而廉价地分割成任意大小的子程序。

Java 8 的 parallel 本质上是 fork-join 的封装，适合用少量线程执行大量任务的情况。本质上，是通过分治归并实现并行的。但这并不适合所有情况。只有在充分测试确实没有安全隐患和性能问题时，才考虑使用 parallel 。
