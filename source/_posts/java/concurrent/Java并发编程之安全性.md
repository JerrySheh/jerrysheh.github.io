---
title: Java并发编程之安全性
comments: true
categories:
- Java
- Concurrent
tags: Java
abbrlink: b4ed848b
date: 2018-03-30 16:08:54
---

并发编程显然有很多优势，然而，多线程也带来了一定的风险。例如安全性问题、活跃性问题、性能问题等。

* **安全性问题**： 含义是“永远不发生糟糕的事情”，例如多个线程同时修改一个共享变量，导致结果跟预期不符。
* **活跃性问题**： 关注“某件正确的事情最终会发生”，假若不能，就会产生活跃性问题。例如死锁，A、B进程互相等待对方释放某资源，结果谁也执行不下去。
* **性能问题**： 在解决安全性问题和活跃性问题的时候会带来额外开销，我们必须想办法减少开销。

并发编程的问题，在[Java简明笔记（十一） 并发编程](../post/727d207c.html)中就有提及，这一篇，主要就安全性问题，详细谈谈Java并发编程的问题。

<!-- more -->

---

# 线程安全性

一个对象是否需要是线程安全的，取决于它是否被多个线程访问。那什么是安全性呢？说白了，就是要保证结果正确！《Java并发编程实战》的作者Brain Goetz对线程安全的定义是：**当多个线程访问某个对象时，不管运行时环境采用何种调度方式或者如何交替执行，并且调用方不需要任何额外的同步操作，调用这个对象的行为都能获得正确的结果，那么就称这个对象是线程安全的**。

## 无状态对象 - 安全

**无状态对象一定是线程安全的**。比如一个 Servlet , 从 Request 从提取数值，执行计算，然后封装到 Response 中。每个收到要计算的 Servlet 线程实例都是自己算自己的，没有跟其他线程的 Servlet 实例共享状态。因此，它是线程安全的。

## 有状态对象 - 原子性问题（竞争条件）

但假设多个 Servlet 之间，要处理共享的一个变量，由于这个变量值是会变的，我们称之为“状态”，又由于这个“状态”可以被多个线程同时读写，我们称之为“共享状态”。这时候多个 Servlet 实例就会产生竞争条件。<font color="red">所谓竞争条件，是指由不恰当的执行时序导致的出现不正确的结果</font>，最常见的竞态条件就是“先观察后执行”，问题就出在观察的值可能是错的。

### 竞争条件例子1：计数器

例如要统计网站访问总人数，用一个 count 共享变量来表示。如下所示：

```java
// count 是共享变量
private static long count = 0;

public long getCount(){ return count; }

public void service (ServletRequest req, ServletResponse resp) {
  //do something
  ++count; // 线程不安全
  //do something
}
```

使用 Intellij IDEA 的 jclasslib 插件，可以看到上述 service 方法翻译成JVM字节码后如下：

```java
0 getstatic #2 <Test/AtomicTest.count> // 获取常量
3 lconst_1                             //将 long 类型的常量添加进操作栈
4 ladd                                 // +1
5 putstatic #2 <Test/AtomicTest.count> // 常量写回
8 return
```

可见 ++count 并非原子操作，它实际上包含 读取 count 的值、计算加一、计算结果写回 count 三个操作（即复合操作）。因此，有可能出现当线程A观察 count 的值，发现为 5，并对其加一，此时线程B观察 count 的值，也发现为 5，并对其加一，之后线程A计算结果将6写回count，线程B也将6写回count。我们预期结果是7，但实际上却是6。这就是竞争条件带来的原子性问题。

### 竞争条件例子2：延迟初始化

延迟初始化是指将对象的初始化操作推迟到实际被使用时才进行。

```java
public class lazyInitRace{
    // 当类被加载，先不初始化
    private ExpensiveObject instance = null;

    //当第一次被使用时，才初始化
    public ExpensiveObject getInstance(){
        // 如果第一次被使用发现为 null，先初始化
        if (instance == null){
            this.instance = new ExpensiveObject();
        }
        // 第二次之后直接返回该对象
        return instance;
    }
}
```

假如线程A和线程B同时执行`getInstance()`方法，线程A观察到 instance 为 null，于是 new 一个实例，由于 new 不是一个原子操作，在 new 还没完成时，instance仍然为null，此时时间片切换到线程B，B也观察到 instance 为 null，又 new 了一个实例。

竞争条件并不总会发生错误，但在某种不恰当的执行时序下，可能会出错。因此是线程不安全的。


## 使用 atomic 原子类解决原子性问题

在 `java.util.concurrent.atomic` 包中包含了一些原子变量类，可以提供原子操作。只需把 count 的类型从 `long` 改为 `Atomiclong`。在程序员的角度，可以认为 Atomiclong 把上述 读取 count 的值、计算加一、计算结果写回 count 这三个操作，合并成一个原子操作。这跟数据库的事务有点类似。

```java
 // count 是共享变量
private Atomiclong count = new AtomicLong(0);

public long getCount(){ return count.get(); }

public void service (ServletRequest req, ServletResponse resp) {
  //do something
  count.incrementAndGet(); // 线程安全
  //do something
}
```

像这样，<font color="red">当一个对象只有一个“状态”时，将该状态交给如 Atomiclong 这类线程安全对象来管理，那么这个类仍然是线程安全的</font>。但并不是添加多个这样的安全对象，程序就线程安全了。当对象不止有一个“状态”时，或者说，涉及多个状态变量时，各个变量之间有时候并不是彼此独立，而是某个变量的值会对其他变量产生影响。这时候修改一个原子变量的同时，也要更新另一个原子变量。这种情况 Atomic 原子类是无法解决的，只能用下文将提到的加锁机制了。

### 深入：atomic原子类为什么可以保证原子性？

atomic原子类底层是用非阻塞并发算法实现的。具体是用了 CAS 算法，**CAS指的是比较并交换(compare and swap)** 。它包含三个数：需要读写的内存位置V、进行比较的值A、拟写入的新值B。当 V 和 A 相等时，才将 V 的值更新为 B。无论是否更新成功，都返回当前内存位置 V 值。

可以这样理解CAS：我认为 V 的值应该为 A，如果是，那么将 V 的值更新为 B，否则不修改并告诉 V 的值实际为多少。

因此，当多个线程并发修改atomic原子变量时，可能的情况为：

```
线程A：观察 V 的值，发现为 5
线程A：进行加一操作（得到6）
线程B：观察 V，也发现为 5
线程B：进行加一操作（得到6）
线程A：再次观察 V 的值，看看是不是预期值5，发现是，就把加一后的值 6 写回 V
线程B：再次观察 V 的值，看看是不是预期值5，发现不是，不写回，重试
线程B：观察 V 的值，发现为 6
线程B：进行加一操作（得到7）
线程B：再次观察 V 的值，看看是不是预期值6，发现是，就把加一后的值 7 写回 V
```

关于atomic原子类可参考另一篇：[Java并发编程之并发工具](../post/a23f9c20.html)

## 使用 加锁机制 解决原子性问题

前面提到，Atomic 原子类可以保证一个“状态”是安全的。但是当对象中存在多个“状态”，并且互相影响。那 Atomic 原子类就不再适用。Java 提供了一种内置的锁机制来支持原子性，即 **同步代码块（Synchronized Block）**。

```java
// synchronized 可用在独立的代码片段，也可以修饰方法
synchronized (obj) {
  // do something .. (访问或修改共享变量和状态)
}
```

synchronized 修饰符表示一个锁。括号里是要锁住的对象（称为对象锁，对象锁是Java的内置锁或者叫监视锁，隐式存在于每个对象中）。当线程A进入了 synchronized 块，它就持有了某个对象的锁，当CPU时间片从线程A切换到其他线程，也执行到这里，就会阻塞在 synchronized 块之外，直到线程A退出synchronized块，释放该锁。需要注意，当对一个父类加了对象锁，子类是不会受到影响的，相反也是如此。

对于有多个状态并且相互影响的对象才使用锁。否则 Atomic 原子类已经足够。此外，synchronized 最好仅仅包含需要互斥同步的临界区代码片段，包含在整个方法的做法有点极端。因为就 Servlet 的例子来说，锁住整个 service 方法，每次只有一个客户端能够响应，多个客户端无法同时使用和计算，服务的响应性能非常低，这就变成一个性能问题了。

### 重入

一个线程请求其他线程持有的锁时，发出请求的线程会被阻塞。但是，如果一个线程试图获得一个 **自己** 持有的锁，则会请求成功。因为 synchronized 内置锁是可重入的。

#### 重入如何实现？

重入的一种实现方式是，为每个锁关联一个获取计数值和一个所有者线程。当计数值为0时，锁没有被任何线程持有。当一个线程获取该锁，JVM将记下锁的持有者，并把计数值+1，这个线程第二次请求该锁，计数值再+1。第二次请求的操作执行完毕后，计数值-1，第一次请求的操作执行完毕后，计数值再-1，便恢复到0，锁被释放。

#### 为什么要重入？

考虑下面的例子, 如果不可重入，那么会发生死锁。在 LoggingWidget 的 doSomething 方法中，跳出去执行父类 Widget 的 doSomething 方法，之后，调用栈返回，又回到子类 LoggingWidget 的 doSomething 方法，会发现锁已经被占用，然而占用锁的人正是它自己。

```java
public class Widget{
  public synchronized void doSomething(){
    //...
  }
}

public class LoggingWidget extends Widget{
  public synchronized void doSomething(){
    super.doSomething();
  }
}
```

好在 synchronized 是可重入的，有了重入，我们就可以在上述 Servlet 的例子中，把需要原子操作的代码片段用 synchronized 封装起来，缩小锁的范围，从而提高并发性了。至于同步代码块的范围要缩小多少，就需要在设计需求之间进行权衡了，包括安全性、简单性和性能等方面。

### 深入：synchronized 原理

同步代码块基于 monitorenter 和 monitorexit 字节码指令来实现。编译后的代码，monitorenter 指令会被插入到同步代码块的开始位置，而 monitorexit 会被插入到代码块结束处和异常处。线程执行到 monitorenter 指令时，将会尝试获取对象所对应的 monitor 所有权。

```java
static int count = 0;

public static void service(){
    synchronized (go.class){
        count++;
    }
}
```

```java
0 ldc #2 <Test/go>
2 dup
3 astore_0
4 monitorenter  // 获得锁
5 getstatic #3 <Test/go.count>
8 iconst_1
9 iadd
10 putstatic #3 <Test/go.count>
13 aload_0
14 monitorexit  // 正常结束 释放锁
15 goto 23 (+8)
18 astore_1
19 aload_0
20 monitorexit  // 发生异常 释放锁
21 aload_1
22 athrow
23 return
```

---


参考：

- [操作系统漫游（二）进程](../post/be1528d7.html)
- [volatile是怎么实现防止指令重排序的？](https://segmentfault.com/q/1010000006767915)
- [剖析Disruptor:为什么会这么快？(三)揭秘内存屏障](http://ifeve.com/disruptor-memory-barrier/)
- 《Java并发编程实战》
- 《Java并发编程的艺术》
