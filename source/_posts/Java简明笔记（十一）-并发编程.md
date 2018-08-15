---
title: Java简明笔记（十一） 并发编程
comments: true
categories: JAVA
tags: Java
abbrlink: 727d207c
date: 2018-03-01 00:29:51
---

// TODO 线程的方法 join 等

# 进程和线程

进程（Process）：一个操作系统任务就是一个进程。可以理解成正在进行中的程序。

线程（Thread）：在一个进程内部要同时运行多个“子任务”，这些“子任务”称为线程。

简单地说，一条线程指的是进程中一个单一顺序的控制流，而一个进程中可以并发多个线程，每条线程并行执行不同的任务。

多进程和多线程的本质区别在于，每个进程拥有自己的一套变量，而线程却共享数据。

关于进程的概念，可参考[操作系统漫游（二）进程](../post/be1528d7.html)

# 线程的生命周期

JAVA线程的五种状态
1. 创建状态
2. 就绪状态：调用 start 方法，但未设置成当前线程，或者在线程运行后从等待或者睡眠中回来时
3. 运行状态：就绪 -> 当前
4. 阻塞状态：正在运行 -> 暂停。如等待某事件发生
5. 死亡状态： run方法结束后，或调用了stop方法


![pic](http://www.runoob.com/wp-content/uploads/2014/01/java-thread.jpg)

图片出处见水印

<!-- more -->

# 线程的创建

## 方式一：实现 Runnable 方法 （推荐）

1. 重写接口 run 方法
2. 创建重写后的Runnable实例 r
3. 创建线程实例，传入 r
4. 调用线程对象的 start 方法

```java
public class myRun implements Runnable {
    public void run(){
      //doing something
    }
}

public class helloworld {
    public static void main(String[] args){
      Runnable r = new myRun();
      Thread t1 = new Thread(r);
      Thread t2 = new Thread(r);
      t1.start();
      t2.start();
    }
}
```

**注意**：
 1. 如果需要共享数据，只需要一个 r 实例，不要创建多个 r 实例。
 2. 调用线程的`run`方法并不会启动线程，只是当前线程去执行`run`方法而已！

## 方式二：继承 Thead 类 （不推荐）

不推荐的原因是：如果一个类继承Thread，则不适合资源共享。但是如果实现了Runable接口的话，则很容易的实现资源共享。

步骤:
1. 定义一个类继承 Thread 类
2. 重写 Thread 类的 run 方法
3. 创建 Thead 类的子类对象创建线程
4. 调用 start 方法开启线程 并调用线程的任务 run 方法执行

```java
//步骤一：定义一个类继承 Thread 类
class ThreadDemo extends Thread {
   private Thread t;

     //步骤二：重写 Thread 类的 run 方法
   public void run(){
     //doing something
   }
}

public class TestThread {

  public static void main(String args[]) {
     //步骤三：创建 Thead 类的子类对象创建线程
     ThreadDemo T1 = new ThreadDemo();
     //步骤四：调用 start 方法开启线程
     T1.start();


     ThreadDemo T2 = new ThreadDemo();
     T2.start();
  }   
}
```

## 方式三：通过 Callable 和 Future 创建线程 （不推荐）

这种方式使用得比较少。

1. 创建 Callable 接口的实现类，并实现 call() 方法，该 call() 方法将作为线程执行体，并且有返回值。
2. 创建 Callable 实现类的实例，使用 FutureTask 类来包装 Callable 对象，该 FutureTask 对象封装了该 Callable 对象的 call() 方法的返回值。
3. 使用 FutureTask 对象作为 Thread 对象的 target 创建并启动新线程。
4. 调用 FutureTask 对象的 get() 方法来获得子线程执行结束后的返回值。


---

# 比较创建线程的三种方式

采用实现 Runnable、Callable 接口的方式创见多线程时，线程类只是实现了 Runnable 接口或 Callable 接口，还可以继承其他类。

使用继承 Thread 类的方式创建多线程时，编写简单，如果需要访问当前线程，则无需使用 `Thread.currentThread() `方法，直接使用 this 即可获得当前线程。

---

# 线程的优先级

Java 线程的优先级是一个整数，取值范围是 1 （Thread.MIN_PRIORITY ） - 10 （Thread.MAX_PRIORITY ）。

默认情况下，每一个线程都会分配一个优先级 NORM_PRIORITY（5）。

不要太过于依赖线程优先级，因为线程优先级并不能保证线程执行的顺序，而且非常依赖于平台。

---

# 线程同步和安全问题

多个线程共享对一个数据的存取，就会产生`竞争条件（race condition）`

如果一个线程在操作共享数据的时候，其他线程参与了运算，就会产生线程安全问题。

## 解决办法一：Syncchronized

将操作共享数据的代码封装在Syncchronized里，一个线程在操作它的时候，其他线程禁止参与。

```java
Object obj = new Object(); //不要放在run()里面

Syncchronized(obj){
	Code
}
```

* obj是任意对象

找到安全隐患所在位置，加入同步代码块

必须有多个线程并使用同一个锁

或者可以在方法修饰符加入同步关键字 `synchronized`

```java
Public synchronized void add(int num){
  //...
}
```

## 解决办法二：ReentrantLock

在类中声明锁
```java
private Lock lock = new ReentrantLock();
```

然后在安全隐患位置加入 lock 和 unlock

```java
private void ticketSelling() {
    lock.lock();
    try {
      //do something
    } catch (InterruptedException e) {
        System.out.println("InterruptedException problem");
    } finally {
        lock.unlock();
    }
}
```

* **注意**： `lock.lock();` 后面接 try 语句， `lock.unlock();`写在 finally里

## 什么时候使用sync，什么时候使用Lock ？

《java核心技术 卷I 第九版》给我们的建议是：

* 最好都不使用，在许多情况下用 java.util.concurrent 包中的一种机制，它会为你处理所有的加锁。如`阻塞队列`（见下文）。
* 如果不是很复杂的竞争情况，尽量用Syncchronized
* 如果比较特殊（比如两个线程可同时读一个文件，但不可一读一写或者同时写等情况），用Lock/condition

延伸阅读：[Syncchronized 和 Lock的区别](https://www.cnblogs.com/nsw2018/p/5821738.html)



---

# 售票例子

假设有 3 个售票厅共同售卖 100 张票，售票厅售卖票的步骤如下：

1. 判断是否有余票
2. 若有余票，售出，总票数-1
3. 若无余票，提示无票

我们用 3 个线程来表示 3 个售票厅，由于线程是并发执行的，想象一个场景：

当剩下最后一张票时，线程1判断有余票，然后进入第二个步骤，此时，线程3恰好 在执行判断余票步骤，由于线程1还没执行 “总票数-1” 这个步骤，于是线程3也判断有余票。之后，线程1执行 “总票数-1” ，总票数为0，但是此时线程3紧接其后也执行了总票数-1，于是导致余票为 -1 的情况。

> 还有一种可能是卖了100次，余票却还有1的情况。从操作系统的角度理解，语句 `ticket--`不是一条语句，而是三条。分别为：从内存中读到CPU，在CPU进行减1，写入内存。当线程一对变量ticket减1的时候，线程二执行了从内存中读到CPU，即把CPU寄存器的值又改回未修改前，之后，线程一执行写到内存，此时ticket一点没变，线程二再执行写到内存，ticket减1。可见，两个线程操作，ticket只减了一次。

myRun.java
```java
public class myRun implements Runnable {
    private int ticketNumber = 100;
    private static int soldTicketNumber = 0;

    public void run(){
        try {
            while ( ticketSelling()){
                Thread.sleep(0);
            }
        } catch (InterruptedException e) {
            System.out.println("problem");
        }

        }

    private boolean ticketSelling() {
        if (ticketNumber > 0){

            try {
                Thread.sleep(30);
            } catch (InterruptedException e) {
                System.out.println("problem");
            }

            System.out.printf("%s号售票厅售出票号为%d的票，余票%d\n",Thread.currentThread().getName(),++soldTicketNumber,--ticketNumber);
            return true;
        } else {
            return false;
        }
    }
}
```

main.java
```java
public class helloworld {
    public static void main(String[] args){
        myRun sellStation = new myRun();
        Thread t = new Thread(sellStation);
        Thread t2 = new Thread(sellStation);
        Thread t3 = new Thread(sellStation);
        t.start();
        t2.start();
        t3.start();
    }
 }
```

输出

```
Thread-1号售票厅售出票号为1的票，余票99
Thread-2号售票厅售出票号为3的票，余票98
Thread-0号售票厅售出票号为2的票，余票99
Thread-1号售票厅售出票号为4的票，余票97
Thread-2号售票厅售出票号为5的票，余票96
Thread-0号售票厅售出票号为6的票，余票95
Thread-2号售票厅售出票号为7的票，余票94
Thread-1号售票厅售出票号为8的票，余票93
Thread-0号售票厅售出票号为9的票，余票92
Thread-0号售票厅售出票号为10的票，余票91
Thread-1号售票厅售出票号为11的票，余票90
Thread-2号售票厅售出票号为12的票，余票89
...
Thread-2号售票厅售出票号为89的票，余票12
Thread-1号售票厅售出票号为91的票，余票10
Thread-0号售票厅售出票号为90的票，余票11
Thread-1号售票厅售出票号为92的票，余票9
Thread-2号售票厅售出票号为93的票，余票8
Thread-0号售票厅售出票号为92的票，余票9
Thread-1号售票厅售出票号为94的票，余票7
Thread-0号售票厅售出票号为95的票，余票6
Thread-2号售票厅售出票号为96的票，余票5
Thread-0号售票厅售出票号为97的票，余票4
Thread-1号售票厅售出票号为98的票，余票3
Thread-2号售票厅售出票号为99的票，余票2
Thread-0号售票厅售出票号为100的票，余票1
Thread-1号售票厅售出票号为101的票，余票0
Thread-2号售票厅售出票号为102的票，余票-1
```

可以看到，余票出现了-1的现象。

稍作修改，把会影响线程资源的代码放到 `synchronized`里，或者给方法加上`synchronized`修饰符。

```java
private synchronized boolean ticketSelling() {
    if (ticketNumber > 0){

        try {
            Thread.sleep(30);
        } catch (InterruptedException e) {
            System.out.println("problem");
        }

        System.out.printf("%s号售票厅售出票号为%d的票，余票%d\n",Thread.currentThread().getName(),++soldTicketNumber,--ticketNumber);
        return true;
    } else {
        return false;
    }
}
```

输出

```
Thread-0号售票厅售出票号为1的票，余票99
Thread-2号售票厅售出票号为2的票，余票98
Thread-1号售票厅售出票号为3的票，余票97
Thread-1号售票厅售出票号为4的票，余票96
Thread-2号售票厅售出票号为5的票，余票95
Thread-0号售票厅售出票号为6的票，余票94
Thread-2号售票厅售出票号为7的票，余票93
Thread-1号售票厅售出票号为8的票，余票92
Thread-2号售票厅售出票号为9的票，余票91
Thread-0号售票厅售出票号为10的票，余票90
Thread-2号售票厅售出票号为11的票，余票89
...
Thread-2号售票厅售出票号为89的票，余票11
Thread-2号售票厅售出票号为90的票，余票10
Thread-2号售票厅售出票号为91的票，余票9
Thread-0号售票厅售出票号为92的票，余票8
Thread-2号售票厅售出票号为93的票，余票7
Thread-1号售票厅售出票号为94的票，余票6
Thread-2号售票厅售出票号为95的票，余票5
Thread-2号售票厅售出票号为96的票，余票4
Thread-0号售票厅售出票号为97的票，余票3
Thread-0号售票厅售出票号为98的票，余票2
Thread-2号售票厅售出票号为99的票，余票1
Thread-2号售票厅售出票号为100的票，余票0
```

---

# 避免同步: 阻塞队列

比如在售票例子中，每个售票厅售出票的时候，不是直接操作「总票数 -1」，而是把 -1 这个指令插入到一个队列中，交由专门的一个线程去做，只有特定的线程才能访问并修改总票数，这样就不需要同步了。

这里只给出一个思路，不深入探讨。
