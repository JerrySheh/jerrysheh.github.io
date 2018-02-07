---
title: Java简明笔记（六） 集合
comments: true
categories: JAVA
tags: Java
abbrlink: f85bb872
date: 2018-02-04 22:43:40
---

《Core Java for the Impatient》简明笔记。

Java API 提供了常用数据结构和算法的实现，以及组织数据结构和算法的框架。这一篇，主要介绍如何使用列表、集合、映射和其他集合。

本章要点：
* Collection接口为所有集合类提供了共同方法（映射除外，映射是通过Map接口）
* 列表是一个有序集合，其中的每个元素都有一个整数索引
* 集合（set）针对高效包含测试进行过优化。Java提供了HashSet和TreeSet实现
* Collection接口和Collections类提供了很多有用的算法：设置操作、查询、排序、打乱原先元素等

---

# 集合和集合框架

## 集合是什么？

通俗的说，集合就是一个放数据的容器，准确的说是放数据对象引用的容器。

Java集合主要可以划分为四个部分：List、Set、Map、工具类（Iterator迭代器、Enumeration枚举类、Arrays和VCollections）。

java的集合类主要由两个接口派生而来，Collection和Map，这两个是集合框架的根接口，这两个接口又包含了一些子接口或实现类。如下图所示是Collectio集合体系和Map集合体系的框架图。

![collection](../../../../images/Java/collection.jpg)

![map](../../../../images/Java/map.jpg)

<!-- more -->

所有的集合框架都包含如下内容：

* 接口：是代表集合的抽象数据类型。接口允许集合独立操纵其代表的细节。在面向对象的语言，接口通常形成一个层次。

* 实现（类）：是集合接口的具体实现。从本质上讲，它们是可重复使用的数据结构。

* 算法：是实现集合接口的对象里的方法执行的一些有用的计算，例如：搜索和排序。这些算法被称为多态，那是因为相同的方法可以在相似的接口上有着不同的实现。

![集合框架](http://www.runoob.com/wp-content/uploads/2014/01/java-coll.png)

除了集合，该框架也定义了几个Map接口和类。Map里存储的是键/值对。尽管Map不是collections，但是它们完全整合在集合中。

(参考：[菜鸟教程](http://www.runoob.com/java/java-collections.html), [极乐科技](https://zhuanlan.zhihu.com/p/24234059))

## 为什么要用集合？

使用集合框架的优点之一是：当遇到一些基本算法时，不必重新实现。一些例如addALL、removeIf等基本方法是Collection接口已经定义好的。

Collection工具类包含很多额外的可在集合上操作的算法，你可以对列表进行排序、打乱、旋转、翻转操作；在集合中查询最大或最小元素，或者任意一个元素的位置；生成不含元素或者含一个元素或同一个元素n份拷贝的集合。

---

# List、Set、Queue

## List

List 是一个有序的集合， ArrayList 和 LinkedList 实现了 List 接口。 链表插入操作很快，但遍历很慢。因此当应用需要有序集合时，用ArrayList可能会更好。但是ArrayList 和 LinkedList 都是线程不安全的。

## Set

Set 中元素不会被插入到特定的位置，并且不允许重复的元素，Set 取出元素的方法只有迭代器。HashSet和TreeSet实现了Set接口，都是线程不安全的。如果想要按顺序遍历集合，可以使用TreeSet。

TreeSet的排序是通过compareTo或者compare方法中的来保证元素的唯一性。元素是以二叉树的形式存放的。

## Queue

Queue会记住插入顺序，但只能在尾端插入，头端删除。Deque有两个尾端，头尾都可以插入和删除。

使用方法举例

创建一个数组列表集合
```java
List<String> words = new ArrayList<>();
```

HashSet包含敏感词检测
```java
Set<String> badWords = new HashSet<>();
badWords.add("fuck");
badWords.add("drugs");
badWords.add("shit");
if (badWords.contain(username.toLowerCase())) {
  System.out.println("please choose a different username")
}
```

---

# Map

map存储键值对。用`put()`方法添加新的键值对或者改变原有的值。

map的一个实现是`Hashtable`，线程安全，速度快。底层是哈希表数据结构。是同步的。不允许null作为键和值。

map的另一个实现是`HashMap`，线程不安全，速度慢。底层也是哈希表数据结构。是不同步的。允许null作为键，null作为值。替代了Hashtable.


还有一种`LinkedHashMap`，可以保证HashMap集合有序。存入的顺序和取出的顺序一致。

如果需要按顺序访问键，用`TreeMap`。


map例子

```java
Map<String, Integer> counts = new HashMap();
counts.put("Alice",1);
counts.put("Jerry",2);
counts.put("Alice",3); //改变原有值

int count = counts.get("Alice"); //获取Alice对应的值，这里是3

//如果Alice对应的值不存在，用get方法会得到空指针异常
//下面这句避免了空指针异常

//获取Alice对应的值，如果值不存在，返回0
int count = counts.getOrDefault("Alice", 0);

//如果word不存在，将word与1形成键值对，否则将word+1
counts.merge(word, 1, Integer::sum);
```

---

# Iterator

每个集合都提供了某种顺序迭代元素的方式。

Collection的父接口Iterable<T>定义了一个方法：

```java
Iterator<T> Iterator()
```

这个方法生成了一个迭代器。迭代器用来访问元素。


在下面这个例子中，把ArrayList变成了一个迭代器，用来访问元素（当然就这个例子来说用 forEach 更简单）
```java
Collection<String> co = new ArrayList<>();
co.add("hello");
co.add("world");
Iterator<String> iter = co.iterator();
while(iter.hasNext()){
    String element = iter.next();
    System.out.println(element);
}
```

---

# Collection 和 Collections的区别

Collection是集合类的上级接口，子接口主要有Set 和List、Map。

Collections是针对集合类的一个帮助类，提供了操作集合的工具方法：一系列静态方法实现对各种集合的搜索、排序、线程安全化等操作。


---

# 简要介绍其他集合

## Properties

Properties类实现了可以很容易地使用纯文本格式保存和加载的映射。是 map 的一种实现。 用于配置文件的定义和操作，使用频率非常高，同时键和值都是字符串。是集合中可以和IO技术相结合的对象。(到了IO在学习它的特有和io相关的功能。)

## BitSet

BitSet（位组）类用来存储一系列比特。

## Enumset 和 Enummap

收集一些枚举值的集合，用EnumSet而不是BitSet

## stack、queue、deque、PriorityQueue

不支持从中间添加元素。如果需要用stack、queue、deque且不关心线程安全问题，建议用 ArrayDeque

## WeakHashMap

WeakHashMap类用来与垃圾回收器配合，当键的唯一引用来自Hash表条目时，就删除键值对。
