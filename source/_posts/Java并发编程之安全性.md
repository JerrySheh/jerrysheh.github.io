---
title: Java并发编程之安全性
comments: true
categories: JAVA
tags: Java
abbrlink: b4ed848b
date: 2018-03-30 16:08:54
---

并发编程显然有很多优势，然而，多线程也带来了一定的风险。例如安全性问题、活跃性问题、性能问题等。

* **安全性问题**： 含义是“永远不发生糟糕的事情”，例如多个线程同时修改一个共享变量，导致结果跟预期不符。
* **活跃性问题**： 关注“某件正确的事情最终会发生”，假若不能，就会产生活跃性问题。例如死锁，A、B进程互相等待对方释放某资源，结果谁也执行不下去。
* **性能问题**： 在解决安全性问题和活跃性问题的时候会带来额外开销，我们必须想办法减少开销。

并发编程的问题，在[Java简明笔记（十一） 并发编程](../post/727d207c.html)中就有提及，这一篇，主要就安全性问题，详细谈谈Java并发编程的问题。

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

假如线程A和线程B同时执行getInstance()方法，线程A观察到 instance 为 null，于是 new 一个实例，由于 new 不是一个原子操作，在 new 还没完成时，instance仍然为null，此时时间片切换到线程B，B也观察到 instance 为 null，又 new 了一个实例。

竞争条件并不总会发生错误，但在某种不恰当的执行时序下，可能会出错。因此是线程不安全的。


## 使用 atomic 原子类解决原子性问题

在 java.util.concurrent.atomic 包中包含了一些原子变量类，可以提供原子操作。只需把 count 的类型从 `long` 改为 `Atomiclong`。在程序员的角度，可以认为 Atomiclong 把上述 读取 count 的值、计算加一、计算结果写回 count 这三个操作，合并成一个原子操作。这跟数据库的事务有点类似。

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

atomic原子类底层是用非阻塞并发算法实现的。具体是用了 CAS 算法。**CAS指的是比较并交换(compare and swap)**。它包含三个数：需要读写的内存位置V、进行比较的值A、拟写入的新值B。当 V 和 A 相等时，才将 V 的值更新为 B。无论是否更新成功，都返回当前内存位置 V 值。

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

# 对象数据共享

要实现多个线程之间的数据共享，需要考虑两个问题：

- **通信**：通信是指消息在两条线程之间传递
- **同步**：既然要传递消息，那接收线程 和 发送线程 之间必须要有个先后关系。此时就需要用到同步，即控制多条线程之间的执行次序。

## 如何通信

一般有两种通信的方式：

1. **共享内存**：共享内存指的是多条线程共享同一片内存，发送者将消息写入内存，接收者从内存中读取消息，从而实现了消息的传递。
2. **消息传递**：顾名思义，消息传递指的是发送线程直接将消息传递给接收线程。

### Java选择哪种通信方式？（Java内存模型）

**Java使用共享内存的方式实现多线程之间的消息传递**。使用这种方式，程序员需要编写额外的代码用于线程之间的同步。在 Java 中，所有线程都共享一片主内存（Main Memory），用于存储共享变量。此外，每条线程都有各自的存储空间，存储各自的局部变量、方法参数、异常对象。

![buffers-modes](../../../../images/Java/buffers-modes2.png)

---

# 可见性和失效数据

可能你不会相信，在 Java 中，一个线程修改了共享对象的状态后，其他线程可能不能及时看到发生的状态变化。为什么其他线程有可能会看不到变化呢？可以从两个角度理解：

1. 从 Java内存模型（JMM）的角度看，正因为每条线程都有各自的存储空间，在多线程中，假设没有加入同步，如果一个线程修改了一个值，储存在自己的线程内存里，另一个线程就会看不到。又或者，你看到的是一个已经失效的值。
2. 从计算机的角度看，现代多核计算机中，每个 CPU 都有自己的寄存器，为了和主存读写速度匹配，CPU寄存器和主存中间往往有一层 Cache 缓冲，当一个 CPU 修改了一个共享变量，放在自己的寄存器或 Cache 缓冲中，还未写回主存，另一个 CPU 就可能读不到最新修改的数据。

考虑下面的例子，

```java
boolean asleep = false;

// 线程A
while(!asleep)
  countSomeSheep();

// 线程B
asleep = true;
```

线程A不断检查 asleep 的值，直到它变成 true 就停止数羊，线程B将 asleep 设置成true，如果不解决可见性问题，线程B的改动，线程A可能永远都看不到。像这样，一个线程不能及时看到另一个线程对共享变量的修改这种情况，叫做可见性问题。

## 使用 volatile 解决可见性问题

解决可见性问题，可以用 synchronized ，因为 synchronized 的语义规定，对一个变量执行 unlock 操作前，必须先把此变量同步回主内存中。但是 synchronized 每次加锁解锁都需要额外的开销，显得太“重”了，会影响性能。

我们也可以用 final 解决可见性问题，被 final 修饰的字段在构造器中一旦初始化完毕（且 this 引用没有逃逸），其他线程立即可以看到 final 字段的值。可惜 final 字段不可再次被修改，有时不满足我们的需求。

第三种方法是将变量声明为 volatile 类型。声明为 volatile 的变量，在写操作时，底层的汇编指令会多出一行 Lock 前缀指令。<font color="red">这个指令在多核处理器中引发了两件事情：第一，将当前处理器缓存行的数据写回到系统内存。第二，该操作使在其他CPU里缓存了该内存地址的数据无效。</font>

volatile 保证了变量每次读写时都是最新的值，但不要太过于依赖 volatile，满足以下条件时，才用 volatile：
1. 对变量的写入操作不依赖变量的当前值（count++就不满足），或者你的程序只有一个线程更新该变量的值(其他线程可访问但不可修改)。
2. 访问变量时不需要加锁
3. 该变量不会与其他状态变量一起纳入不变性条件中

也就是说， volatile 是解决 **可见性** 问题的，并不能解决所有原子性问题。另外，当想禁止编译器的重排序功能时，也可以用 volatile。

---

# 重排序问题

当我们写一个单线程程序时，总以为计算机会一行行地运行代码，然而事实并非如此。编译器、处理器会在不改变程序执行结果的前提下，**重新排列指令的执行顺序**，以达到最佳的运行效率，这就是重排序。多线程环境下，重排序可能带来一些问题。考虑下面的例子：

```java
class Reorder{
    int a = 0;
    boolean flag = false;

    // 线程A
    public void writer(){
        a = 2;
        flag = true;
    }

    // 线程B
    public void reader(){
        if (flag) {
            int i = a*a;
        }
    }
}
```

假设有两个线程，A线程先执行writer方法，之后B线程执行reader方法。按理说，B会将 i 设置成 4，然而事实却不一定。原因是，<font color="red">编译器和处理器会对没有依赖关系的语句进行一定程度的重排序</font>。在线程A中，可能 flag 先被设置成 true，然后线程B执行reader，发现flag 为 true，执行赋值语句，最后A线程的 a 才被赋值为 2。

解决重排序问题，也可以使用 volatile， volatile 本身包含禁止指令重排序的语义。

## 深入：为什么 volatile 能解决重排序问题？

声明为 volatile 的变量，实际上相当于程序员显式地告诉编译器和处理器不要使用重排序。汇编指令中多出来的 Lock，实际上也就是一道内存屏障。处理器遇到内存屏障时，就会知道不要对此处乱序执行。事实上，Linux 或 Windows 作为操作系统，也只是调用 CPU 所实现的内存屏障指令而已，归根结底这个不是操作系统或者编译器去实现，而是硬件实现了然后供软件调用。

---

# 线程封闭

不共享数据是避免使用同步最好的办法。这称为线程封闭（Thread Confinement）。线程封闭的例子有 JDBC 中的 Connection 对象。

线程封闭包括 Ad-hoc 、栈封闭、ThreadLocal类。这里只探讨ThreadLocal。

## ThreadLocal 类

在单线程 JDBC 程序中，我们通常在程序启动时初始化一个 Connection 连接，从而避免在调用每个方法时都传递一个 Connection 对象。

在多线程 JDBC 程序中，我们希望每个线程建立自己的 Connection 对象连接，不互相干扰。可以通过 ThreadLocal 来解决。

```java
// 使用ThreadLocal封闭
private static ThreadLocal<Connection> connectionHolder =
                               new ThreadLocal<Connection>(){
    public Connection initValue(){
      return DriverManager.getConnection(DB_URL);
    }
};

// 获取一个连接
public static Connection getConnection(){
  return connectionHolder.get();
}
```

ThreadLocal提供了一些比如set、get等来访问接口和方法，每个使用该变量的线程都有一份独立的副本，线程之间互不影响。

ThreadLocal对象常用于防止对可变的单实例变量（singeton）或全局变量进行共享。

---

# 发布与逸出

发布（publish）的意思是，在当前作用域之外的代码中使用对象，例如将对象的引用传递到其他类的方法中。在多线程环境下，如果一个对象在构造完成之前就被发布，会破坏线程安全性。而当某个不应该发布的对象被不小心发布出去，就叫逸出（escape）。考虑下面的例子：

```java
class status{
  private String[] s = new String[] {"AK","AL","AJ"};

  public String[] get(){
    return this.s;
  }
}
```

外部可以通过 get() 方法获取数组 s 的引用，而 s 是一个 private 数组，外部现在就有权力修改这个数组里面的所有元素，这就是逸出。逸出使得我们的封装变得没有意义。

还有一种逸出的情况就是发布一个类的内部类实例，因为内部类是隐式持有外部类引用的。

## 安全地构造

在构造方法中启动一个线程，this引用会被新创建的线程共享，此时还没构造完毕，会导致线程安全问题。好的做法是，等构造方法结束时，this引用才逸出。在构造方法中创建一个线程，然后通过一个 `start()` 方法来启动线程。永远不要在构造过程中使 this 引用逸出。如果想在构造函数中注册一个事件监听或者启动线程，好的办法是使用静态工厂方法（私有构造函数+公共工厂方法）。

---

# 安全发布

有 Holder 这么一个类

```java
public class Holder {
    private int n;

    public Holder(int n){
        this.n = n;
    }

    public void assertSanity(){
        if (n != n) {
            thorw new AssertionError("statement false")
        }
    }
}

假设线程1对Holder类进行了发布

```java
public Holder holder;

public void initialize(){
    holder = new Holder(42);
}
```

然后线程2调用assertSanity()方法，很有可能出现 n != n，抛出 AssertionError 。因为线程1的发布，没有使用同步对其他线程可见。

## 安全地发布

要安全地发布一个对象，对象的引用和对象的状态必须同时对其他线程可见。

一般可以通过以下几种方式：
- 在静态初始化函数中初始化一个对象的引用
- 将对象的引用保存到 volatile 类型的域或者 AtomicReferance 对象中
- 将对象的引用保存到某个正确构造的 final 类型域中
- 将对象的引用保存到一个由锁保护的域中

---


参考：

- [操作系统漫游（二）进程](../post/be1528d7.html)
- [volatile是怎么实现防止指令重排序的？](https://segmentfault.com/q/1010000006767915)
- [剖析Disruptor:为什么会这么快？(三)揭秘内存屏障](http://ifeve.com/disruptor-memory-barrier/)
- 《Java并发编程实战》
- 《Java并发编程的艺术》
