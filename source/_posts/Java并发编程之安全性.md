---
title: Java并发编程之安全性
comments: true
categories: JAVA
tags: Java
abbrlink: b4ed848b
date: 2018-03-30 16:08:54
---

并发编程显然有很多优势，然而，多线程也带来了一定的风险。例如安全性问题、活跃性问题、性能问题等。

* **安全性问题** 的含义是“永远不发生糟糕的事情”，例如多个线程同时修改一个共享变量，导致结果跟预期不符（售票例子）。

* **活跃性问题** 关注“某件正确的事情最终会发生”，假若不能，就会产生活跃性问题。例如死锁，A、B进程互相等待对方释放某资源，结果谁也执行不下去。

* **性能问题** 则是在解决安全性问题和活跃性问题的时候带来的额外开销，我们必须想办法减少开销。

并发编程的问题，在[Java简明笔记（十一） 并发编程](../post/727d207c.html)中就有提及，这一篇，主要就安全性问题，详细谈谈Java并发编程的问题。

---

# 线程安全性

一个对象是否需要是线程安全的，取决于它是否被多个线程访问。那什么是安全性呢？说白了，就是要保证结果正确！

无状态对象，比如一个 Servlet , 从 Request 从提取数值，执行计算，然后封装到 Response 中。 每个收到要计算的 Servlet 线程实例都是自己算自己的，没有跟其他线程的 Servlet 实例共享状态。 因此，它是线程安全的。

但是假设多个 Servlet 之间，要处理共享的一个变量，这时候多个 Servlet 实例就会产生竞争条件。竞争条件并不总会发生错误，但在某种不恰当的执行时序下，可能会出错。因此是线程不安全的。

```java
private long count = 0; // count 是共享变量

public long getCount(){ return count; }

public void service (ServletRequest req, ServletResponse resp) {
  //do something
  ++count; // 线程不安全
  //do something
}
```

那么要怎么改呢？

在 java.util.concurrent.atomic 包中包含了一些原子变量类，可以提供原子操作。即把 count 的类型从 `long` 改为 `Atomiclong`。

```java
private Atomiclong count = new AtomicLong(0); // count 是共享变量

public long getCount(){ return count.get(); }

public void service (ServletRequest req, ServletResponse resp) {
  //do something
  count.incrementAndGet(); // 线程安全
  //do something
}
```

> 像 `Atomiclong` 这样的是线程安全对象，应该尽可能地在并发程序中使用。但并不是添加多个这样的安全对象，程序就线程安全了。还要考虑当更新一个变量时，在同一个原子操作中对其他变量同时进行更新。

## 加锁机制

Java 提供了一种内置的锁机制来支持原子性，即`同步代码块（Synchronized Block）`。

```java
synchronized (lock) {
  // 访问或修改共享变量和状态
}
```

或者在方法中加入 `synchronized` 修饰符，例如

```java
public synchronized void service(ServletRequest req,
                                    ServletResponse resp) {
  // do something
}
```

然而，这种做法有点极端。因为就Servlet的例子来说，多个客户端无法同时使用计算，服务的响应性能非常低，这就变成一个性能问题了。

## 重入

一个线程请求其他线程持有的锁时，发出请求的线程会被阻塞。但是，如果一个线程试图获得一个**自己**持有的锁，则会请求成功。因为内置锁是可重入的。

重入的一种实现方式是，为每个锁关联一个获取计数值和一个所有者线程。当计数值为0时，锁没有被任何线程持有。当一个线程获取该锁，JVM将记下锁的持有者，并把计数值+1，这个线程第二次请求该锁，计数值再+1。第二次请求的操作执行完毕后，计数值-1，第一次请求的操作执行完毕后，计数值再-1，便恢复到0，锁被释放。

重入锁的作用体现在下面的代码中, 如果不可重入，那么会发生死锁。

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

`synchronized`是可重入的，有了重入，我们就可以在上述 Servlet 的例子中，把原子操作用`synchronized`方法封装起来，缩小锁的范围，从而提高并发性了。至于同步代码块的范围要缩小多少，就需要在设计需求之间进行权衡了，包括安全性、简单性和性能等方面。

---

# 对象的共享

我们不仅希望一个线程正在访问某对象状态时，另一个线程参与了修改，还希望确保一个线程修改了对象状态后，其他线程能够看到其中的变化。

## 可见性和失效数据

在多线程中，假设没有加入同步，如果一个线程修改了一个值，另一个线程可能会看不到。（至于为什么，是因为编译器和运行时的重排序，这个问题值得再深入研究，这里先接受这个奇怪的事实）

又或者，你看到的是一个已经失效的值。即使不考虑失效数据的问题，在并发程序中使用共享的 long 或者 double 等类型也是不安全的，因为JVM允许将64位的操作分解为两个32位操作。

## volatile

Java 中的 volatile 变量可以被看作是一种 “程度较轻的 synchronized”，与 synchronized 块相比，volatile 变量所需的编码较少，并且运行时开销也较少，但是它所能实现的功能也仅是synchronized 的一部分。

当变量声明为 volatile 类型后，编译器和运行时会注意到这是个共享变量，就不会将该变量的操作与其他内存操作一起重排序。因此在读取 volatile 变量时总会返回最新的值。

但是不要太过于依赖 volatile ， 满足以下条件时，才用 volatile
* 对变量的写入操作不依赖变量的当前值（count++就不满足），或者你的程序只有一个线程更新该变量的值(其他线程可访问但不可修改)。
* 访问变量时不需要加锁
* 该变量不会与其他状态变量一起纳入不变性条件中

也就是说， volatile 是解决可见性问题的，并不能解决所有安全问题。

volatile例子

```java
volatile boolean asleep;
...
  while(!asleep)
    countSomeSheep();
```

## 安全地构造对象

在构造方法中启动一个线程，this引用会被新创建的线程共享，此时还没构造完毕，因此会导致线程安全问题。

好的做法是，等构造方法返回时，this引用才逸出。在构造方法中创建一个线程，然后通过一个start()方法来启动线程。

## 线程封闭

不共享数据是避免使用同步最好的办法。这称为线程封闭（Thread Confinement）。线程封闭的例子有 JDBC 中的 Connection 对象。

## ThreadLocal 类

在单线程 JDBC 程序中，我们通常在程序启动时初始化一个 Connection 连接，从而避免在调用每个方法时都传递一个 Connection 对象。

在多线程 JDBC 程序中，我们希望每个线程建立自己的Connection对象连接，不互相干扰。可以通过 ThreadLocal来解决。

```java
private static ThreadLocal<Connection> connectionHolder =
                               new ThreadLocal<Connection>(){
    public Connection initValue(){
      return DriverManager.getConnection(DB_URL);
    }
};

public static Connection getConnection(){
  return connectionHolder.get();
}
```

---

# 对象组合

未完待续

（参考：[操作系统漫游（二）进程](../post/be1528d7.html)）
