---
title: Java并发编程之并发工具
comments: true
categories: JAVA
tags: Java
abbrlink: a23f9c20
date: 2018-10-30 15:08:26
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

有时候，我们看起来好像没有迭代，但仍然抛出了`ConcurrentModificationException`。是因为有些方法隐式地进行了迭代，如打印一个Hashset，事实上会调用 toString 方法，这个方法不断调用 StringBuilder.append 把各个元素转为字符串，这其实就是迭代了。同理，hashCode方法、equals方法、containAll、removeAll、retainAll等方法都是如此。

因此，使用同步容器类时，需要在调用方加 synchronized 同步。

---

# 并发容器

同步容器简单粗暴地对公有方法加同步，实际上是强行将对容器状态的访问串行化了，这对并发性能带来了很大影响。在 Java 5 之后，增加了如 ConcurrentHashMap、ConcurrentLinkedQueue 这样的并发容器，天生为并发程序设计。在多线程中应该尽可能用并发容器，而不是同步容器。

## ConcurrentHashMap

ConcurrentHashMap 采用了细粒度的加锁机制，称为分段锁（Lock Striping）。分段锁的原理是，容器内部有多个锁，每一把锁只锁住容器内一部分数据。在 JDK 1.7 里，一个 ConcurrentHashMap 内部是一个 Segment 数组， Segment 数组每个元素都是一个 Entry 数组，Entry 数组每个元素都是 Entry 链表对象。当需要加锁的适合，不是加锁整个 ConcurrentHashMap，而是加锁 Segment 数组上的每个 Segment 对象。

![concurrentHashMap](../../../../images/Java/concurrentHashMap.png)

但是在JDK 1.8中，取消了基于 Segment 的分段锁思想，改用 CAS + synchronized 控制并发操作。

ConcurrentHashMap 迭代时不会抛出 `ConcurrentModificationException`，是 fail-safe 的。它实现了 ConcurrentMap 接口，如下：

```java
public interface ConcurrentMap<K, V> extends Map<K, V> {

    // 仅当 K 没有相应的映射值时才插入
    V putIfAbsent(K key, V value);

    // 仅当 K 被映射到 value 时才插入
    boolean remove(Object key, Object value);

    // 仅当 K 被映射到 oldValue 时才替换为 newValue
    boolean replace(K key, V oldValue, V newValue);

    // 仅当 K 被映射到某个值时才替换为 newValue
    V replace(K key, V value);
}
```

## CopyOnWriteArrayList

这是一个写入时复制（Copy-On-Write）并发容器，用于替代同步的List。在每次修改时，都会创建并重新发布一个新的容器副本。迭代器不会抛出`ConcurrentModificationException`，是 fail-safe 的。当迭代操作远远多于修改操作时，应该考虑使用Copy-On-Write容器。


## BlockingQueue（生产者消费者模式）

BlockingQueue 是一个阻塞队列。其 put 方法将一个元素放进队列头端，如果队列已满，就一直阻塞，直到队列空出位置。同理，take 方法将从队列尾端取出一个元素，如果队列未空，就一直阻塞，直到队列有元素。BlockingQueue非常适合用来做生产者-消费者模式。其优点是，将生产数据的过程与使用数据的过程解耦。

BlockingQueue 也提供了一个 offer 方法，如果数据不能添加进队列，返回一个失败状态。

BlockingQueue的实现类有 LinkedBlockingQueue 和 ArrayBlockingQueue，以及按优先级排序的 PriorityBlockingQueue。还有一个比较特殊的 SynchronousQueue，它没有存储空间，只是维护一组线程。例如，一个线程 put 会被阻塞，直到另一个线程 take，才算成功交付。

---

# 同步工具类

BlockingQueue 阻塞队列不仅能作为保存对象的容器，而且能根据其自身的状态来协调线程的控制流。所以它既是并发容器，也是一个同步工具。我们把能 **根据其自身的状态来协调线程的控制流的工具称为同步工具**。

同步工具类的特点是：封装了一些状态，这些状态决定执行同步工具类的线程是继续执行还是等待，此外还提供一些方法对状态进行操作。常见的同步工具类有：闭锁（Latch）、栅栏（Barrier）、信号量（Semaphore）。

## 闭锁（Latch）

闭锁相当于一个门，且有一个开门的条件。未开门前，所以线程都不能通过，当门打开后才允许所有线程通过。闭锁打开之后将不能再关上。使用闭锁的场景有：

- 确保某个计算在其需要的所有资源都被初始化后才继续执行。
- 确保某个服务在其依赖的其他服务都被启动之后才启动。
- 等待某个操作的参与者都就绪再继续执行（多人在线游戏）

CountDownLatch 是一种闭锁的实现。包括一个计数器，一开始为正数，表示需要等待的事件数量。以及 countDown 方法，每当一个等待的事件发生了，计数器就减一。直到为零闭锁打开。如果计数器不为零，那 await 会一直阻塞直到计数器为零，或者等待中的线程中断，或者等待超时。

## FutureTask

待补充。

## 信号量（Semaphore）

计数信号量（Counting Semaphore）用来控制同时访问某个特定资源的操作数量，或者同时执行某个指定操作的数量。Semaphore 可以用来实现资源池，如数据库连接池。

## 栅栏（Barrier）

栅栏跟闭锁类似，不同点在于，所有线程必须同时到达栅栏位置，才能继续执行。**闭锁用于等待事件，而栅栏用于等待其他线程**。CyclicBarrier 可以使一定数据的参与方反复地在栅栏位置汇集，通常用于并行迭代算法。
