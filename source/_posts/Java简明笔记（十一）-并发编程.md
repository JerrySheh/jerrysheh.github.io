---
title: Java简明笔记（十一） 并发编程
comments: true
date: 2018-03-01 00:29:51
categories: JAVA
tags: Java
---

# 进程和线程

进程（Process）：一个操作系统任务就是一个进程。可以理解成正在进行中的程序。

线程（Thread）：在一个进程内部要同时运行多个“子任务”，这些“子任务”称为线程。

简单地说，一条线程指的是进程中一个单一顺序的控制流，而一个进程中可以并发多个线程，每条线程并行执行不同的任务。

多进程和多线程的本质区别在于，每个进程拥有自己的一套变量，而线程却共享数据。

# 线程的生命周期

JAVA线程的五种状态
1. 创建状态
2. 就绪状态：调用 start 方法，但未设置成当前线程，或者在线程运行后从等待或者睡眠中回来时
3. 运行状态：就绪 -> 当前
4. 阻塞状态：正在运行 -> 暂停。如等待某事件发生
5. 死亡状态： run方法结束后，或调用了stop方法


![pic](http://www.runoob.com/wp-content/uploads/2014/01/java-thread.jpg)

图片出处见水印

# 线程的创建

## 方式一：继承 Thead 类

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

## 方式二：实现 Runnable 方法

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

## 方式三：通过 Callable 和 Future 创建线程

1. 创建 Callable 接口的实现类，并实现 call() 方法，该 call() 方法将作为线程执行体，并且有返回值。
2. 创建 Callable 实现类的实例，使用 FutureTask 类来包装 Callable 对象，该 FutureTask 对象封装了该 Callable 对象的 call() 方法的返回值。
3. 使用 FutureTask 对象作为 Thread 对象的 target 创建并启动新线程。
4. 调用 FutureTask 对象的 get() 方法来获得子线程执行结束后的返回值。


---

# 比较创建线程的三种方式

采用实现 Runnable、Callable 接口的方式创见多线程时，线程类只是实现了 Runnable 接口或 Callable 接口，还可以继承其他类。

使用继承 Thread 类的方式创建多线程时，编写简单，如果需要访问当前线程，则无需使用 `Thread.currentThread() `方法，直接使用 this 即可获得当前线程。

未完待续
