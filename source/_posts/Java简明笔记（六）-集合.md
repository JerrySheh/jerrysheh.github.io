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

List 是一个有序的集合。List类是一个抽象类。实现类 `ArrayList` 和 `LinkedList` 实现了 List 接口。所以我们平时用的时候，要指定是`ArrayList` 还是 `LinkedList`。

> 我应该用哪种 List？

> **链表插入操作很快，但遍历很慢**。因此当应用需要有序集合时，用ArrayList可能会更好。但是注意， ArrayList 和 LinkedList 都是线程不安全的。

ArrayList使用示例：

```java
public static void main(String[] args) {
    List<String> groupName = new ArrayList<>();
    groupName.add("jerry");
    groupName.add("calm");
    groupName.add("Superman");
    System.out.println("groupName的大小：" + groupName.size());
    System.out.println("groupName原始的内容" + groupName);
    System.out.println("jerry在容器的位置：" + groupName.indexOf("jerry"));

    //将下标1的内容替换为 Paul
    groupName.set(1,"Paul");
    System.out.println("\ngroupName替换后的内容" + groupName);
}
```

输出：
```
groupName的大小：3
groupName原始的内容[jerry, calm, Superman]
jerry在容器的位置：0

groupName替换后的内容[jerry, Paul, Superman]
```

下面是一些常用方法

| 常用方法 | 简介 |
| :------------- | :------------- |
| add     | 增加，支持直接加在末尾，或者指定位置 |
| contain | 判断容器中是否存在某个对象（而不是对象值相等） |
| get     | 获取指定位置的对象（如果越界会报错）	|
| indexOf | 获取对象所处的位置(从0开始)	|
| remove | 删除，支持按下标或者按对象 |
| set (index, object)  | 替换 |
|  size  | 获取大小 |
| toArray  | 转换为数组 |
|  addAll | 把另一个容器所有对象都加进来 |
| clear | 清空 |



## Set

Set 中元素是无序的，且不允许重复的元素，从 Set 容器中取出元素的方法只有迭代器。

HashSet和TreeSet实现了Set接口，但都是线程不安全的。

如果想要按顺序遍历集合，可以使用TreeSet。TreeSet的排序是通过compareTo或者compare方法中的来保证元素的唯一性。元素是以二叉树的形式存放的。

HashSet包含敏感词检测的例子

```java
Set<String> badWords = new HashSet<>();
badWords.add("fuck");
badWords.add("drugs");
badWords.add("shit");
if (badWords.contain(username.toLowerCase())) {
  System.out.println("please choose a different username")
}
```

## Queue

Queue会记住插入顺序，但只能在尾端插入，头端删除。Deque有两个尾端，头尾都可以插入和删除。

---

# Map

map存储键值对。用`put()`方法添加新的键值对或者改变原有的值。

map的一个实现是`Hashtable`，线程安全，速度快。底层是哈希表数据结构。是同步的。不允许null作为键和值。

map的另一个实现是`HashMap`，线程不安全，速度慢。底层也是哈希表数据结构。是不同步的。允许null作为键，null作为值。替代了Hashtable.

| Hashtable | HashMap     |
| :------------- | :------------- |
| 线程安全       | 线程不安全       |
哈希表|哈希表
同步|不同步
不允许null|允许null


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

# 迭代器 Iterator

每个集合都提供了某种顺序迭代元素的方式。

Collection的父接口Iterable<T>定义了一个方法：

```java
Iterator<T> Iterator()
```

这个方法生成了一个迭代器。迭代器用来访问元素。


在下面这个例子中，iter是一个迭代器，迭代的对象是 groupName ， while循环用来访问元素。

```java
public static void main(String[] args) {
    List<String> groupName = new ArrayList<>();
    groupName.add("jerry");
    groupName.add("calm");
    groupName.add("Superman");
    groupName.set(1,"Paul");

    // 用迭代器
    Iterator<String> iter = groupName.iterator();
    while (iter.hasNext()){
        String name = iter.next();
        System.out.println(name);
    }
}
```

当然这个例子用 foreach 更简单
```java
//用 foreach
for (String name :
        groupName) {
    System.out.println(name);
}
```
当然 foreach 也有缺点：
* 无法用来进行ArrayList的初始化
* 无法得知当前是第几个元素了，当需要只打印单数元素的时候，就做不到了。必须再自定下标变量。

## ArrayList和Iterator的实际例子

ArrayList和Iterator的例子，要求初始化50个 hero，名字为 hero0,hero1,hero2...hero50， 然后删除 hero8,hero16,hero24...

- 可以用 foreach ， 也可以用 Iterator

```java
public static void main(String[] args) {
    List<String> groupName = new ArrayList<>();

    // 初始化 50 个 hero
    for (int i = 0; i <= 50; i++) {
        groupName.add("hero" + i);
    }

    System.out.println(groupName);

    Iterator<String> iter = groupName.iterator(); //迭代groupName的迭代器
    List<String> waitToRemove = new ArrayList<>();

    //开始迭代，把8的倍数记录下来
    while (iter.hasNext()){
        String name = iter.next();
        int i = groupName.indexOf(name);
        if (i % 8 ==0 ){
            waitToRemove.add(name);
        }
    }

    groupName.removeAll(waitToRemove);
    System.out.println(groupName);
}
```

这里有两个坑:
* 不能在iterator迭代的过程中，对容器进行增删操作。否则会抛出ConcurrentModificationException
* 不要在一次迭代中进行多次 `iter.next()` 操作

解决办法是：用一个waitToRemove容器，来存放待删除的数据，迭代完成再一并删除。

---

# Collection 和 Collections的区别

Collection是集合类的上级接口，子接口主要有Set 和List、Map。

Collections是针对集合类的一个帮助类，提供了操作集合的工具方法：一系列静态方法实现对各种集合的搜索、排序、线程安全化等操作。

---

# ArrayList、LinkedList、Vector的区别

## ArrayList 和 Vector

相同点：
* Arraylist和Vector是采用数组方式存储数据。

> 一般数组元素数大于实际存储的数据以便增加插入元素，Arraylist和Vector都允许直接序号索引元素。插入数据要涉及到数组元素移动等内存操作，所以插入数据慢，查找有下标，所以查询数据快。

不同点:
* Vector由于使用了synchronized方法-线程安全，所以性能上比ArrayList要差。但是ArrayList是线程不安全的。

## ArrayList 和 LinkedList

相同点：
* ArrayList 和 LinkedList 在末尾插入都很快。

不同点：
* LinkedList使用**双向链表**实现存储，按序号索引数据需要进行向前或向后遍历，插入数据时只需要记录本项前后项即可，因此在中间插入数据较快。
* ArrayList遍历十分快，LinkedList中间插入特别快。

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
