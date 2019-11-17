---
title: Effective Java（八）General Programming
comments: true
abbrlink: 7d5810ff
date: 2019-11-14 21:55:19
categories: Effective Java
tags: Java
---

# Item 57 最小化局部变量的作用域

> 好的编程习惯：在首次使用的地方声明它。

1. 如果循环终止后不需要循环变量的内容，那么优先选择 for 循环而不是 while 循环。
2. 如果变量需要在 `try-catch` 之外使用，那就必须在外面提前声明，这是一个例外。其他情况都应该遵循在首次使用的地方声明。
3. 每个行为对应一个方法。保持方法小而集中。

---

# Item 58 for-each 循环优于 for-i 循环

如果你只是需要容器里的元素，而不需要下标，for-i循环显然增加出错的可能性。最好用 for-each。for-each还可以用来遍历实现 Iterable 接口的任何对象。

但也有不能用for-each的情况：

- **过滤删除**：如果需要遍历集合，并删除指定选元素，则需要使用显式iterator，以便可以调用其 remove 方法。 通常可以使用在 Java 8 中添加的 Collection 类中的 removeIf 方法，来避免显式遍历。

```java
List<String> li = new ArrayList<>(Arrays.asList("aa","bb","cc"));
li.removeIf( a -> "aa".equals(a));
```

- **转换**：如果需要遍历一个列表或数组并替换其元素的部分或全部值，那么需要 iterator 或数组索引来替换元素的值。
- **并行迭代**

---

# Item 59 了解并使用库

> 使用标准库，等于站在巨人肩膀上。

例如，生成随机数，自己写有很大的不确定性，但是直接使用 `Random.nextInt(int)` 可以直接得到期望的结果。Java 7 更应该用 `ThreadLocalRandom`，它能产生更高质量的随机数，而且速度比`Random`快。对于 fork 连接池和并行流，使用 `SplittableRandom`。

每个程序员都应该熟悉 java.lang、java.util 和 java.io 的基础知识及其子包。其他库的知识可以根据需要学习。此外，Collections 框架和 Streams 库应该是每个程序员的基本工具包的一部分，`java.util.concurrent` 中的并发实用程序也应该是其中的一部分。

如果你在 Java 平台库中找不到你需要的东西，你的下一个选择应该是寻找高质量的第三方库，比如谷歌的优秀的开源 Guava 库 [Guava]。

不要重复造轮子！

---

# Item 60 精确数字就应避免 float 和 double ，使用 BigDecimal

《阿里巴巴Java开发手册》提到：

> 【强制】小数类型为 decimal ，禁止使用 float 和 double 。
> 说明： float 和 double 在存储的时候，存在精度损失的问题，很可能在值的比较时，得到不正确的结果。如果存储的数据范围超过 decimal 的范围，建议将数据拆成整数和小数分开存储。

float 和 double 类型特别不适合进行货币计算。

```java
// 输出：0.6100000000000001
System.out.println(1.03 - 0.42);
```

使用 BigDecimal 能解决这个问题，注意：**使用 BigDecimal 的 String 构造函数而不是它的 double 构造函数**。

```java
final BigDecimal TEN_CENTS = new BigDecimal(".10");
BigDecimal funds = new BigDecimal("1.00");
```

---

# Item 61 基本数据类型优于包装类

基本数据类型和其包装类两者之间有真正的区别！！自动装箱和自动拆箱模糊了基本类型和包装类型之间的区别，但不会消除它们的区别。

1. 基本类型只有它们的值，而包装类型有方法，引用，对象。
2. 基本类型只有值，而包装类型还能是 null。
3. 基本类型比包装类型更节省时间和空间。

如果你不小心的话，这三种差异都会给你带来真正的麻烦。例如，将 `==` 操作符应用于包装类型，这几乎都会带来错误。因为包装类同值可不同对象。

混合使用基本类型和包装类型，包装类型就会自动拆箱。如果一个空对象引用自动拆箱，将导致 `NullPointerException`。

还有一个问题，在 for 循环中声明包装类，可能会产生很多对象，或者反复装箱和拆箱，从而导致性能下降。

那什么时候应该用包装类呢？

1. **作为集合中的元素、键和值**：不能将基本类型放在集合中，因此必须使用包装类型。
2. **泛型**：不能将变量声明为 `ThreadLocal<int>` 类型，只能用 `ThreadLocal<Integer>`。
3. **反射**：在进行反射方法调用时，必须使用包装类型

---

# Item 62 当使用其他类型更合适时应避免使用字符串

略

---

# Item 63 当心字符串连接引起的性能问题

不要使用 + 连接大量字符串，除非性能无关紧要。因为用 + 连接两个字符串时，本质上会复制这两个字符串的内容。一般这种需求最好使用 StringBuilder 的 append 方法。

---

# Item 64 通过接口引用对象

如果存在合适的接口类型，那么应该使用接口类型声明参数、返回值、变量和字段。除非具体类要使用的方法是接口没有的。

```java
// Good - uses interface as type
Set<Son> sonSet = new LinkedHashSet<>();

// Bad - uses class as type!
LinkedHashSet<Son> sonSet = new LinkedHashSet<>();
```

---

# Item 65 接口优于反射

用反射调用方法比普通调用要慢得多，可能会造成性能损失。而且不能在编译时做类型检查。

通常在代码分析工具或依赖注入框架里会看到反射。仅仅在需要使用编译时不存在的类时才会用到反射。除此之外最好都用接口来声明类。

---

# Item 66 谨慎使用 Native 方法

JNI 允许 Java 程序调用本地方法，这些方法是用 C 或 C++ 等本地编程语言编写的。由于本地语言比 Java 更依赖于平台，因此使用本地方法的程序的可移植性较差，也更难调试。

---

# Item 67 谨慎优化

> 编写好的程序，而不是快速的程序

很多计算上的过失都被归昝于效率。不要去计较效率上的一些小小的得失，在 97% 的情况下，不成熟的优化才是一切问题的根源。​ —William A. Wulf [Wulf72] —Donald E. Knuth [Knuth74]

在优化方面，我们应该遵守两条规则：
- 规则 1：不要进行优化。
- 规则 2 （仅针对专家）：还是不要进行优化，也就是说，在你还没有绝对清晰的未优化方案之前，请不要进行优化。​ —M. A. Jackson [Jackson75]

但是在设计系统时一定要考虑性能，特别是在设计API、线路层协议和持久数据格式时。

---

# Item 68 遵守被广泛认可的命名约定

参考《阿里巴巴Java开发手册》

Identifier Type	| Example
---|---
Package or module |	org.junit.jupiter.api, com.google.common.collect
Class or Interface |	Stream, FutureTask, LinkedHashMap,HttpClient
Method or Field	 |remove, groupingBy, getCrc
Constant Field、	|MIN_VALUE, NEGATIVE_INFINITY
Local Variable|	i, denom, houseNum
Type Parameter|	T, E, K, V, X, R, U, V, T1, T2

特别提一下容易被忽略的参数类型：T 表示任意类型，E 表示集合的元素类型，K 和 V 表示 Map 的键和值类型，X 表示异常。函数的返回类型通常为 R。任意类型的序列可以是 T、U、V 或 T1、T2、T3。
