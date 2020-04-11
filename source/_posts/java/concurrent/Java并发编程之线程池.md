---
title: Java并发编程之线程池
categories:
- Java
- Concurrent
tags:
  - Java
  - 并发
abbrlink: b4abef1f
date: 2020-04-11 23:47:54
---

> 线程池是线程的管理工具，跟线程本身一样，线程池也有不同的状态。

在并发编程中，我们可能需要创建许多个线程来执行任务，这些线程频繁地被创建、执行，然后又销毁，这个过程的开销是比较大的。能不能弄一个像池子一样的线程池，线程统一创建，需要执行任务时从线程池里取一个线程来执行，执行完再把线程放回去呢？

<!-- more -->

# ThreadPoolExecutor

在 [Java并发编程之异步任务](../post/377edbcb.html) 提到，Java.util.concurrent 包的作者 Doug Lea 设计了 `ExecutorService` 接口，用于提交异步任务。而 ThreadPoolExecutor 是 `ExecutorService` 的默认实现，用来启动一个线程池。但是，线程池是一个配置繁杂，状态较多的工具，我们通常不手动启动，而是借助 Executors 类的静态工厂方法来生成。

---

# 使用 Executors 创建线程池

Executors 类跟 Executor 接口关系不大，这个类主要提供了一些静态工厂，用来生成一些常用的线程池执行器。

```java
// 单线程执行器
ExecutorService executorService1 = Executors.newSingleThreadExecutor();

// 固定线程数量
ExecutorService executorService2 = Executors.newFixedThreadPool(10);

// 多线程调度执行器
ExecutorService executorService3 = Executors.newScheduledThreadPool(10);
```

简单例子

```java
ExecutorService executorService = Executors.newFixedThreadPool(10);

executorService.execute(new Runnable() {
    public void run() {
        System.out.println("Asynchronous task");
    }
});

executorService.shutdown();
```

---

# 线程池生命周期

既然 ThreadPoolExecutor 是 ExecutorService 的实现，而 ExecutorService 又继承 Executor 接口。所以，线程池的生命周期即是 Executor 的生命周期啦。

## CREATE & RUNNING

当我们 new 一个 Executor 或者通过 Executors 静态工厂构造一个 Executor，线程池即进入了 RUNNING 状态，在此之前是 CREATE 状态。严格意义上讲线程池构造完成后并没有线程被立即启动，只有进行“预启动”或者接收到任务的时候才会启动线程。但是无论如何，构造完后，线程池已经在运行，随时准备接受任务来执行。

## SHUTDOWN & STOP

通过`shutdown()`和`shutdownNow()`来将线程关闭。

- `shutdown()`平缓关闭，线程池停止接受新的任务，同时等待已经提交的任务执行完毕，包括那些进入队列还没有开始的任务，这时候线程池处于SHUTDOWN状态。
- `shutdownNow()`是一个立即关闭过程，线程池停止接受新的任务，同时线程池取消所有执行的任务和已经进入队列但是还没有执行的任务，这时候线程池处于STOP状态。

## TERMINATED

一旦`shutdown()`或者`shutdownNow()`执行完毕，线程池就进入TERMINATED状态，此时线程池就结束了。

- `isTerminating()`描述的是SHUTDOWN和STOP两种状态。
- `isShutdown()`描述的是非RUNNING状态，也就是SHUTDOWN/STOP/TERMINATED三种状态。

---
