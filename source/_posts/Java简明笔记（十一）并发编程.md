---
title: Java简明笔记（十一）并发编程
comments: true
categories: JAVA
tags: Java
abbrlink: 727d207c
date: 2018-03-01 00:29:51
---

# 问题的来源

以前的计算机都只有一个 CPU， 并且一次只能执行一个程序。后来出现了 **多任务（multitasking）** 使得计算机可以同时执行多个程序，但这并不是真正的“同时”，只是把 CPU 分成多个时间片，由操作系统去调度切换。再之后出现了 **多线程（multithreading）** 使得在一个程序里面可以同时执行线程，就像你有多个 CPU 在执行同一个程序一样。在单 CPU 的计算机中，多线程的“同时”并不是“同时”，但现代计算机一般都是多核 CPU，不同的线程可以被不同的 CPU 核心同时执行，是真正的同时。

![multithreading](http://tutorials.jenkov.com/images/java-concurrency/java-concurrency-tutorial-introduction-1.png)

如果一个线程在读一块内存区域的同时，另一个线程在往里面写，那么这块区域的值是什么？或者两个线程同时写一块内存区域，它的值又是什么？假如我们没有对这些可能出现的结果进行防范，那么结果将是不可预测的。什么情况都可能发生。因此，我们需要在一些共享资源上做一些措施，例如内存、文件、数据库等。

<!-- more -->

---

# 为什么要多线程？

## 利

1. **更好的资源利用**：多线程程序在一个线程加载 IO 的同时，另一个线程可以处理已经加载完毕的 IO，以节省时间。
2. **简化程序设计**：单线程程序，既要负责加载IO，又要负责处理。多线程程序，可以让一个线程专门加载，另一个线程专门处理。程序逻辑更加清晰。
3. **更加高效的程序**：当一个请求进来时，处理请求可能需要耗费一些时间。单线程程序这时候就无法接收新的请求了，而多线程程序一个线程负责接收请求，每次收到请求都开一个专门的线程去处理，实现了多请求。

## 弊

1. **更加复杂的设计**：多线程有时候会让程序变得更加复杂（complex）。
2. **上下文切换消耗**：CPU从一个线程切换到另一个线程时，要先保存上一个线程的 local data，程序指针等。这会带来一些消耗。
3. **提高资源消耗**：多线程本身需要一些内存用于存储其 local stack，这可能消耗一些内存资源。不要以为它很小，实际上可能比你想象的多。

---

# 并发模型（Concurrency Models）

并发模型指的是线程如何在系统中协同完成一项工作。不同的并发系统可以用不同的并发模型来实现。

## 并发模型和分布式系统的相似性

本文涉及的并发系统模型跟分布式系统（distributed systems）十分相似。例如，并发系统中是不同的线程之间互相通讯（communicate），而分布式系统是不同的进程之间互相通讯（这些进程可能在不同的计算机上）。

进程和线程在某些时候十分相似，这也是为什么不同的并发模型往往看起来都很像分布式系统架构。分布式系统需要面临网络请求可能失败、远程计算机或进程可能挂掉等问题，在并发系统中也会遇到类似 CPU 故障、网卡故障、硬盘故障等问题，虽然这些故障发生的概率很低，但理论上确实存在。

正因为并发模型和分布式系统十分相似，因此他们之间有些设计思想是相通的。例如，并发里面用于分配 workers（threads）的模型，就类似于分布式系统的负载均衡（load balancing in distributed systems）。

## 模型1：并行 Workers 模型

![parallel workers](http://tutorials.jenkov.com/images/java-concurrency/concurrency-models-1.png)

在这个模型中，每一个 Worker（Thread）都由 delegator（委派者） 来委派。每一个 Worker 都完成一个完整的工作。例如，在一个车间工厂，一辆车从零到一都只由一个 Worker 来完成。

这种模型在 Java 中用得非常多。在 java.util.concurrent 包中，许多的并发工具都是用这个模型来设计的。在 J2EE 应用服务器中也能看到这种模型的影子。

## 模型2：流水线模型

流水线模型，也叫响应式系统或者事件驱动系统。在这个模型中，每一个 Worker 只负责整个工作的一小部分，一旦完成自己的部分，就交给下一个 Worker 接着做。每个 Worker 都运行在单独的线程上，与其他 Worker 不共享状态。因此，流水线模型也叫做无共享（shared nothing）模型。这种模型通常用非阻塞IO来实现（non-blocking IO）。

所谓响应式，或者事件驱动，指的是当一个事件发生，Worker 就响应对应的动作。例如，一个HTTP请求进来，或者一个文件被加载进内存。响应式平台的例子有Vert.x、Akka、Node.JS。

![Assembly Line](http://tutorials.jenkov.com/images/java-concurrency/concurrency-models-3.png)

## 模型3：函数式并行

函数式并行的基本思想是采用函数调用实现程序。函数都是通过拷贝来传递参数的，所以 **除了接收函数外没有实体可以操作数据，这就避免了共享数据带来的竞争**。同样也使得函数的执行类似于原子操作。每个函数调用的执行独立于任何其他函数的调用。

一旦每个函数调用都可以独立的执行，它们就可以分散在不同的CPU上执行了。这也就意味着能够在多处理器上并行的执行使用函数式实现的算法。在 Java8 中，并行 streams 能够用来帮助我们并行的迭代大型集合。

---

# 在 Java 中创建线程

在 Java 中，线程是 java.lang.Thread 或其子类中的实例。通过 new 一个线程类并调用 start 方法来启动线程。

```java
Thread thread = new Thread();
thread.start();
```

但是我们总得指定线程做一些事，可以用两种方式来指定。

## 方式一：继承 Thread 类

```java
public class MyThread extends Thread {
    @Override
    public void run(){
       System.out.println("MyThread running");
    }
}

Thread myThread = new myThread();
myThread.start();
```

## 方式二：实现 Runnable 方法(推荐)

```java
// 可以直接在  Runnable 接口上操作
Runnable myRunnable = new Runnable(){
    @Override
    public void run(){
      System.out.println("Runnable running");
    }
}

// 也可以用一个类实现 Runnable 方法
class MyRunnable implements Runnable {
    @Override
    public void run(){
       System.out.println("MyRunnable running");
    }
}

Thread thread = new Thread(new MyRunnable()， “第二个参数指定线程名”);
thread.start();

// 暂停线程
thread.sleep();

// 停止线程
thrad.stop();
```

## Java8方式：lambda表达式

这种方式本质上是取代了匿名类

```java
Thread t = new Thread( ()-> System.out.println("do something"));
```

<font color="red">特别提醒</font>：
- 调用 .start() 方法才是启动线程，而调用 .run() 方法只是在当前线程上去执行 run 函数，并没有开启新线程。注意不要搞混。
- .stop() 方法已经被标记为 deprecated。因为它无法保证被停止的线程的状态（有潜在的死锁危险）。正确停止线程的方式应该是给定一个 boolean 变量，在方法中将其置 false。可以在 run 方法中加入 while() 循环 ，当主线程将 boolean 置 false， 子线程的 while 不再执行，从而 run 方法结束，从而线程退出。

---

# 竞争条件和临界区

当多个线程同时访问一块共享区域的时候，就会产生竞争条件。考虑下面的例子：

```java
public class Counter {

   protected long count = 0;

   public void add(long value){
       this.count = this.count + value;
   }
}
```

从 Java 虚拟机的角度看，add 方法并不是一个原子操作，而是分为三步：

1. 将 this.count 读进 CPU 寄存器
2. 在寄存器里 + 数值
3. 将寄存器的值写回内存

假如有A、B两个线程同时执行 add 方法，那么可能发生：

```
this.count = 0;

A:  将 this.count 读进 CPU 寄存器 (0)
B:  将 this.count 读进 CPU 寄存器 (0)
B:  在寄存器里 + 2
B:  将寄存器的值 (2) 写回内存， this.count 现在是 2
A:  在寄存器里 + 3
A:  将寄存器的值 (3) 写回内存， this.count 现在是 3
```

我们期望的值是5，而结果却是3。如何解决呢？可以用 Java 提供的 **synchronized 同步代码块** 或者 **锁结构** 或者 java.util.concurrent.atomic包里面的 **原子变量** 来解决。

同步代码块
```java
public void add(int val1, int val2){
    synchronized(this.sum1Lock){
        this.sum1 += val1;   
    }
    synchronized(this.sum2Lock){
        this.sum2 += val2;
    }
}
```

---

# 线程安全和资源共享

## 什么是线程安全？

《Java并发编程实战》的作者 Brian Goetz 对线程安全的定义是：**当多个线程访问某个对象时，不管运行时环境采用何种调度方式或者如何交替执行，并且调用方不需要任何额外的同步操作，调用这个对象的行为都能获得正确的结果，那么就称这个对象是线程安全的**。

如果一段代码是线程安全的，那么就不会发生竞争条件。反过来说，对于非线程安全的代码，多个线程同时修改共享资源的时候，就会发生竞争条件。

因此我们得先知道 Java 中哪些资源是共享的，哪些是不共享的。

## 局部本地变量（完全线程安全）

局部变量存储在每个线程自己的栈内存中，不会共享，是线程安全的。

```java
public void someMethod(){

  long threadSafeInt = 0;

  threadSafeInt++;
}
```

## 引用变量（部分安全）

局部引用本身跟局部变量一样，其本身是不共享。但是引用指向的对象就不一定了，因为所有的对象存储在共享的堆里面的，因此不是线程安全的。但如果对象是局部方法里面声明的，那就是安全的。

## 类成员变量（不安全）

很明显成员变量（类变量）不是线程安全的。考虑下面的例子，当两个线程同时执行了 add 方法。结果很难预测。

```java
public class NotThreadSafe{
    StringBuilder builder = new StringBuilder();

    public add(String text){
        this.builder.append(text);
    }
}
```

## Thread Control Escape Rule

Thread Control Escape Rule（线程控制逃逸规则）：如果一个资源从创建、使用、到释放(dispose)这整个过程都由相同的一个线程负责，其控制权不会交给其他线程，那么整个资源就是线程安全的。

这里的资源，可以是一个对象、数组、文件、数据库连接、socket等等。在 Java 中，我们无法手动释放资源，因此释放可以理解为失去该对象的引用或置空(nulling)的过程。

但是注意，有时候尽管一个对象是线程安全的，但是应用不一定是安全的。比如，线程1和线程2分别创建了数据库链接 connection1 和 connection2， connection 本身是安全的，但是通过 connection 去访问数据库，可能造成不安全的后果。比如：

```
Thread 1 checks if record X exists. Result = no
Thread 2 checks if record X exists. Result = no
Thread 1 inserts record X
Thread 2 inserts record X
```

---

# 线程安全和不可变性

竞争条件只发生在多个线程同时写一个资源的过程，读的过程并不会造成竞争条件。因此，我们可以用不可变这个特性，来确保线程安全。具体的做法是：

1. 通过构造器传递值，没有 setter
2. 修改时返回一个新的对象，而不是在该对象上变更值

```java
public class ImmutableValue{

  private int value = 0;

  // 构造器传值
  public ImmutableValue(int value){
    this.value = value;
  }

  public int getValue(){
    return this.value;
  }

  // 如果要修改，直接返回一个新的ImmutableValue对象，而不是在该对象上变更值
  public ImmutableValue add(int valueToAdd){
    return new ImmutableValue(this.value + valueToAdd);
  }

}
```

此时，ImmutableValue对象本身是安全的，但是注意，使用这个对象时，也可能不是安全的。考虑下面的例子：

Calculator 对象使用了 ImmutableValue，当有多个线程同时调用 Calculator 的 setValue 或 add 方法，那 Calculator 的类变量 ImmutableValue 的值无法确定。

```java
public class Calculator{
  private ImmutableValue currentValue = null;

  public ImmutableValue getValue(){
    return currentValue;
  }

  public void setValue(ImmutableValue newValue){
    this.currentValue = newValue;
  }

  public void add(int newValue){
    this.currentValue = this.currentValue.add(newValue);
  }
}
```

需要对 Calculator 类的 getValue(), setValue(), 和 add() 方法添加 synchronized 修饰符，这时候才是真正安全。

---

# 从内存模型看并发

在冯诺依曼计算机架构中，主存和 CPU 中间，往往有 Cache 缓冲，CPU 内部也有用于临时数据处理的寄存器。在Java虚拟机中，内存模型大致可以分为堆和栈。每个线程都有单独的线程栈，不会共享，而所有的对象都存放在共享的堆里。

![](http://tutorials.jenkov.com/images/java-concurrency/java-memory-model-5.png)

正因为对象和变量可以存储在不同的内存区域，因此会导致一些问题。最主要的两个是：
1. **可见性问题**：一个线程修改了变量，但是还保存在线程栈中，没有写回内存。另一个线程就看不到这个修改。
2. **竞争条件**：两个线程同时修改类变量，结果不可预期。

在 Java 中，竞争条件通过 synchronized 同步代码块解决，可见性问题通过声明 volatile 关键字解决。

---

# synchronized

synchronized用于解决竞争条件问题。

```java
// 用于实例方法
public synchronized void add(int value){
    this.count += value;
}

// 用于静态方法
public static synchronized void add(int value){
      count += value;
  }
```

同步是一种高开销的操作，因此应该尽量减少同步的内容。把 synchronized 放在方法里，可能太过于粗粒度了，这样会损失一些并发性，synchronized 也可以单独放在临界区。缩小同步控制的范围。

```java
// 用于部分临界区代码块
public void add(int value){
    synchronized(this){
         this.count += value;   
    }
}
```

---

# Volatile 关键字

## 可见性问题

由于 CPU 有 Cache 缓存，一个线程修改的数据，保存在某CPU核心的 Cache 缓存里，还未写回主存，运行在另一个CPU核心的线程可能看不到修改的值。这就是可见性问题。

Volatile 用于解决可见性问题，被 Volatile 关键字修饰的变量，永远是储存在主存里面的。线程每次都从主存读取或往主存写入，而不是往 CPU Cache 读写。

之前一直有个疑问，如果其他 CPU Cache 已经缓存了数据，那你及时写回主存又有什么用呢？其他 CPU 从 Cache 读的数据还是旧的值呀？

事实上，对 Volatile 修饰的共享变量进行写操作时，在编译成汇编的代码里会加一个 `lock` 前缀，这个 `lock`引发了两件事情：

1. 将当前 CPU Cache 的数据写回主存
2. 使其他 CPU Cache 缓存了该内存地址的数据无效（CPU会嗅探在总线上传播的数据来检查自己的缓存是不是过期了）

有了第二点保证，就能让其他 CPU 下次要用到该值时，不得不去主存里拿，因为之前缓存的已经失效了。

## 什么时候用 Volatile

很多时候，只使用 Volatile 是不够的，通常都要配合 synchronized 。只有当满足以下条件时，才只使用 Volatile：

* 对变量的写入操作不依赖变量的当前值（count++就不满足），或者你的程序只有一个线程更新该变量的值(其他线程可访问但不可修改)。
* 访问变量时不需要加锁
* 该变量不会与其他状态变量一起纳入不变性条件中

---

# ThreadLocal 类

ThreadLocal 类可以让你创建一些变量，只能够在同一个线程里读写。其他线程看不到。

```java
// 声明一个 ThreadLocal 对象
private ThreadLocal myThreadLocal = new ThreadLocal();

// 也可以用泛型声明
private ThreadLocal<String> myThreadLocal = new ThreadLocal<String>();

// 往对象里放一些变量
myThreadLocal.set("aStringValue");

// 将 ThreadLocal 里存放的变量取出来
String threadLocalValue = (String) myThreadLocal.get();

// 在声明ThreadLocal对象时，即给初值
private ThreadLocal myThreadLocal = new ThreadLocal<String>() {
    @Override
    protected String initialValue() {
        return "This is the initial value";
    }
};    
```

除了 ThreadLocal 类之外，还有一个 InheritableThreadLocal 是可继承的 ThreadLocal ，只有声明的线程及其子线程可以使用 InheritableThreadLocal 里面存放的变量。

---

# 线程通信（Signaling）

线程通信用于线程与线程之间互相发送信号，或者让线程等待其他线程的信号以协调工作。有两种线程通信的方式，其一是消息队列，其二是共享对象。Java采用的是共享对象的方式。

## 通过共享对象发送信号

线程之间通信最简单的方式就是通过设置一个所有线程都可访问的共享对象变量，通过改变它的值来表示一个信号。例如，线程A对一个共享的 boolean 变量设置为 true，线程B检查该变量值的变化并做出处理。注意，这个共享变量必须要做同步操作，例如 synchronized

```java
public class MySignal{

  protected boolean hasDataToProcess = false;

  public synchronized boolean hasDataToProcess(){
    return this.hasDataToProcess;
  }

  public synchronized void setHasDataToProcess(boolean hasData){
    this.hasDataToProcess = hasData;  
  }

}
```

考虑上面的例子，线程A和线程B分别持有 Mysignal 的引用，线程A 将 hasDataToProcess 设置成 true， 线程B不断查询 hasDataToProcess 的状态，直至其被 线程A 设置成 true 即可开始处理一些事情。线程B不断查询的这个过程，称为 **忙等待（busy wait）**。

```java
while(!sharedSignal.hasDataToProcess()){
  //do nothing... busy waiting
}
```

## wait(), notify() 和 notifyAll()

忙等待会损耗大量 CPU 资源，因此 Java 提供了一些内置的机制，能够让信号未到来之前使线程处于非活跃（inactive）状态。等信号来了之后再唤醒它。这样就不用一直忙等。

---

# 死锁

线程A持有 1 资源，同时需要 2 资源， 线程B 持有 2 资源，同时需要 1 资源，但是 1 资源 和 2 资源都是互斥的。此时，A线程 在等待 B线程 释放 2 资源，B线程在等待 A线程 释放 1 资源，结果谁都无法释放，谁也都得不到资源。这就是死锁。

---

未完待续
