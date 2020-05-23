---
title: Java并发编程之对象共享
categories:
  - Java
  - Concurrent
tags: Java
abbrlink: 959bfd05
date: 2020-04-06 19:08:54
---

# 对象数据共享

要实现多个线程之间的数据共享，需要考虑两个问题：

- **通信**：通信是指消息在两条线程之间传递
- **同步**：既然要传递消息，那 `接收线程` 和 `发送线程` 之间必须要有个先后关系。此时就需要用到同步，即控制多条线程之间的执行次序。

<!-- more -->

## 如何通信

一般有两种通信的方式：

1. **共享内存**：共享内存指的是多条线程共享同一片内存，发送者将消息写入内存，接收者从内存中读取消息，从而实现了消息的传递。
2. **消息传递**：顾名思义，消息传递指的是发送线程直接将消息传递给接收线程。

## Java选择哪种通信方式？（Java内存模型）

**Java使用共享内存的方式实现多线程之间的通信**。使用这种方式，程序员需要编写额外的代码用于线程之间的同步。在 Java 中，所有线程都共享一片主内存（Main Memory），用于存储共享变量。此外，每条线程都有各自的存储空间，存储各自的局部变量、方法参数、异常对象。

![buffers-modes](../../../../images/Java/buffers-modes2.png)

---

# 可见性和失效数据

可能你不会相信，在 Java 中，一个线程修改了共享对象的状态后，其他线程可能不能及时看到发生的状态变化。为什么其他线程有可能会看不到变化呢？可以从两个角度理解：

1. 从 Java内存模型（JMM）的角度看，正因为每条线程都有各自的存储空间，在多线程中，假设没有加入同步，如果一个线程修改了一个值，储存在自己的线程内存里，另一个线程就会看不到。又或者，你看到的是一个已经失效的值。
2. 从计算机的角度看，现代多核计算机中，每个 CPU 都有自己的寄存器，为了和主存读写速度匹配，CPU寄存器和主存中间往往有一层 Cache 缓冲，当一个 CPU 修改了一个共享变量，放在自己的寄存器或 Cache 缓冲中，还未写回主存，另一个 CPU 就可能读不到最新修改的数据。
3. 从 JVM 的角度来看，虚拟机会对源代码做一些重排序或优化。

考虑下面的例子，

```java
boolean asleep = false;

// 线程A
while(!asleep)
  countSomeSheep();

// 线程B
asleep = true;
```

线程A不断检查 asleep 的值，直到它变成 true 就停止数羊，线程B将 asleep 设置成true，如果不解决可见性问题，线程B的改动，线程A可能永远都看不到。**像这样，一个线程不能及时看到另一个线程对共享变量的修改这种情况，叫做可见性问题**。

在这里例子中，JVM可能会把 while 循环优化成：

```java
if (!asleep)
  while(true)
    countSomeSheep();
```

---

# 解决可见性问题

## 使用 synchronized

解决可见性问题，可以用 synchronized ，因为 synchronized 的语义规定，对一个变量执行 unlock 操作前，必须先把此变量同步回主内存中。但是 synchronized 每次加锁解锁都需要额外的开销，显得太“重”了，会影响性能。

## 使用 final

我们也可以用 final 解决可见性问题，被 final 修饰的字段在构造器中一旦初始化完毕（且 this 引用没有逃逸），其他线程立即可以看到 final 字段的值。可惜 final 字段不可再次被修改，有时不满足我们的需求。

## 使用 volatile

第三种方法是将变量声明为 volatile 类型。声明为 volatile 的变量，在写操作时，底层的汇编指令会多出一行 Lock 前缀指令。<font color="red">这个指令在多核处理器中引发了两件事情：第一，将当前处理器缓存行的数据写回到系统内存。第二，该操作使在其他CPU里缓存了该内存地址的数据无效。</font>

volatile 保证了变量每次读写时都是最新的值，但不要太过于依赖 volatile，满足以下条件时，才用 volatile：
1. 对变量的写入操作不依赖变量的当前值（因为会产生竞争条件引发[安全性问题](../post/b4ed848b.html)，i++就不满足），或者你的程序只有一个线程更新该变量的值(其他线程可访问但不可修改)；
2. 访问变量时不需要加锁；
3. 该变量不会与其他状态变量一起纳入不变性条件中。

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

假设有两个线程，A线程先执行writer方法，之后B线程执行reader方法。按理说，B会将 i 设置成 4，然而事实却不一定，i还可能是0。原因是，<font color="red">编译器和处理器会对没有依赖关系的语句进行一定程度的重排序</font>。在线程A中，可能 flag 先被设置成 true，然后线程B执行reader，发现flag 为 true，执行赋值语句，i = 0 * 0，最后，A线程的 a 才被赋值为 2。

**解决重排序问题，也可以使用 volatile， volatile 本身包含禁止指令重排序的语义。**

## 深入：为什么 volatile 能解决重排序问题？

声明为 volatile 的变量，实际上相当于程序员显式地告诉编译器和处理器不要使用重排序。汇编指令中多出来的 Lock，实际上也就是一道内存屏障。处理器遇到内存屏障时，就会知道不要对此处乱序执行。事实上，Linux 或 Windows 作为操作系统，也只是调用 CPU 所实现的内存屏障指令而已，归根结底这个不是操作系统或者编译器去实现，而是硬件实现了然后供软件调用。

---

# 线程封闭

**不共享数据是避免使用同步最好的办法，这称为线程封闭（Thread Confinement）**。线程封闭包括 Ad-hoc 、栈封闭、ThreadLocal类，这里只探讨ThreadLocal。

## ThreadLocal 类

在单线程 JDBC 程序中，我们通常在程序启动时初始化一个 Connection 连接，从而避免在调用每个方法时都传递一个 Connection 对象。在多线程 JDBC 程序中，我们希望每个线程建立自己的 Connection 对象连接，不互相干扰。这种场景就可以通过 ThreadLocal 来解决。

ThreadLocal提供了一些比如set、get等来访问接口和方法，每个使用该变量的线程都有一份独立的副本，线程之间互不影响。

## ThreadLocal 类简单例子

声明一个 ThreadLocal 对象
```java
private ThreadLocal myThreadLocal = new ThreadLocal();
// or
private ThreadLocal<String> myThreadLocal = new ThreadLocal<String>();
//or
// 在声明ThreadLocal对象时，即给初值，而不是第一次调用
private ThreadLocal myThreadLocal = new ThreadLocal<String>() {
    @Override
    protected String initialValue() {
        return "This is the initial value";
    }
};    
```

使用`set()`放置线程封闭变量，使用`get()`将其取出。

```java
// 往对象里放置变量
myThreadLocal.set("aStringValue");

// 将 ThreadLocal 里存放的变量取出来
String threadLocalValue = (String) myThreadLocal.get();
```


除了 ThreadLocal 类之外，还有一个 InheritableThreadLocal 是可继承的 ThreadLocal ，只有声明的线程及其子线程可以使用 InheritableThreadLocal 里面存放的变量。

在 JDK 1.7 之后，还有一个 java.util.concurrent.ThreadLocalRandom 类。

```java
// 返回特定于当前线程的 Random 类实例
static ThreadLocalRandom current()
```

---

# 发布与逸出

发布（publish）的意思是，在当前作用域之外的代码中使用对象，例如将对象的引用传递到其他类的方法中。在多线程环境下，如果一个对象在构造完成之前就被发布，会破坏线程安全性。**而当某个不应该发布的对象被不小心发布出去，就叫逸出（escape）**。考虑下面的例子：

```java
class status{
  private String[] s = new String[] {"AK","AL","AJ"};

  public String[] get(){
    return this.s;
  }
}
```

外部可以通过 `get()` 方法获取数组 s 的引用，而 s 是一个 private 数组，外部现在就有权力修改这个数组里面的所有元素，这就是逸出。逸出使得我们的封装变得没有意义。

还有一种逸出的情况就是发布一个类的内部类实例，因为内部类是隐式持有外部类引用的。

## 安全地构造

在构造方法中启动一个线程，this引用会被新创建的线程共享，此时还没构造完毕，会导致线程安全问题。好的做法是，等构造方法结束时，this引用才逸出。在构造方法中创建一个线程，然后通过一个 `start()` 方法来启动线程。**永远不要在构造过程中使 this 引用逸出**。如果想在构造函数中注册一个事件监听或者启动线程，好的办法是使用静态工厂方法（私有构造函数+公共工厂方法）。

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
```

假设线程1对Holder类进行了发布

```java
public Holder holder;

public void initialize(){
    holder = new Holder(42);
}
```

然后线程2调用`assertSanity()`方法，很有可能出现 n != n，抛出 AssertionError 。

原因：存在可见性问题，线程1的 new 指令使 holder 对象开始构造，构造到一半时线程2即调用`assertSanity()`方法了,线程2看到的 holder 对象可能是一个空引用，或者是初始化了一半的值。

## 安全地发布

要安全地发布一个对象，对象的引用和对象的状态必须同时对其他线程可见。

一般可以通过以下几种方式：
- 在静态初始化函数中初始化一个对象的引用
- 将对象的引用保存到 volatile 类型的域或者 AtomicReferance 对象中
- 将对象的引用保存到某个正确构造的 final 类型域中
- 将对象的引用保存到一个由锁保护的域中(如线程安全容器)

---

参考：

- [操作系统漫游（二）进程](../post/be1528d7.html)
- [volatile是怎么实现防止指令重排序的？](https://segmentfault.com/q/1010000006767915)
- [剖析Disruptor:为什么会这么快？(三)揭秘内存屏障](http://ifeve.com/disruptor-memory-barrier/)
- 《Java并发编程实战》
- 《Java并发编程的艺术》
