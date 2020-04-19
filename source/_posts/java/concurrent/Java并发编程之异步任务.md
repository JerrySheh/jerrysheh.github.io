---
title: Java并发编程之异步任务
comments: true
categories:
- Java
- Concurrent
tags:
  - Java
  - 并发
abbrlink: 377edbcb
date: 2019-06-05 00:25:54
---

有时候，我们想在主线程之外执行一些异步任务，不难想到，可以开一个新线程专门去处理某个任务。在 Java 中处理异步任务都有哪些需要注意的呢？

<!-- more -->

# 最简单的异步任务执行

## Runnable

Runnable 用于一个异步执行的任务，没有参数和返回值。实现 Runnable 接口，把我们想要执行的任务写在 run 方法里，即可表示任务内容。

```java
package java.lang;
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```


## Callable

Callable 与 Runnable 类似，区别是，Callable有返回值，且可以抛出异常。

```java
package java.util.concurrent;
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

## 一个简易Web服务器例子

无论是 Runnable 还是 Callable，想要放到独立的线程中去运行，都是需要借助 Thread 类的。

```java
new Thread(runnable).start();
new Thread(callable).start();
```

我们可以编写一个简易的Web服务器，当一个请求进来时，新建一个线程去服务这个请求。（不要在生产环境使用）

```java
public static void main(String[] args) throws IOException {
    ServerSocket socket = new ServerSocket(80);
    for (;;){
        final Socket conn = socket.accept();
        Runnable task = () -> handleRequest(conn);
        new Thread(task).start();
    }
}

private static void handleRequest(Socket conn){
    // do something
}
```

---

# Executor 接口

由我们手动去 new 线程执行任务，有一些缺点：

- 随着新任务一个个的到来，旧任务一个个结束，线程不断创建销毁，开销很大；
- 线程数量不可控，如果一下子有大量任务到来，将无限制创建线程，即使没有OOM，也会因线程数量太多而导致性能降低（CPU在线程之间切换也是有开销）

所以通常我们不会手动去 new 线程执行任务，而是借助 Executor 接口帮助我们执行。

Executor 接口用来执行一个 Runnable。

```java
public interface Executor {
    void execute(Runnable command);
}
```

为什么要把线程放到 Executor 里面执行呢？

- Executor 提供了一种标准的方法将任务的提交过程和执行过程解耦。这是一种生产者-消费者模式。
- 我们可以在 Executor 的实现类里，添加一些方法，用于管理生命周期和做其他监控，便于我们调度异步任务。

## 借助 Executor 的简易Web服务器

这一次，我们不再 new 线程，而是把 task 放到 Executor 里面去执行。

```java
public class ExecutorExample {
    private static final int THREAD_NUMBER = 100;
    private static final Executor exec = Executors.newFixedThreadPool(THREAD_NUMBER);

    public static void main(String[] args) throws IOException {
        ServerSocket socket = new ServerSocket(80);
        for (;;){
            final Socket conn = socket.accept();
            Runnable task = () -> handleRequest(conn);
            exec.execute(task);
        }
    }

    private static void handleRequest(Socket conn){
        // do something
    }

}
```

## Executors 工具类

Executor 的实现类各种各样，我们可以把 Executor 实现为单线程，或者实现为跟每次都自己 new 线程一样，也可以实现为限制最大线程数量等。通常情况下不同的实现类参数复杂，状态较多，所以我们通常不自己 new Executor，而是借助工具类 Executors 帮助我们创建 Executor。

```java
// 单线程执行器，如果该线程异常，将创建另一个来替代
Executor exec1 = Executors.newSingleThreadExecutor();

// 固定线程数量，提交一个任务就创建一个，直至最大值
Executor exec2 = Executors.newFixedThreadPool(10);

// 多线程调度执行器
Executor exec3 = Executors.newScheduledThreadPool(10);
```

## ExecutorService 接口

事实上，我们用 Executors 创建出来的执行器是 Executor 的子接口 ExecutorService。既然已经有了 Executor 了，为什么还要 ExecutorService 呢？前面说到，我们不只是把任务丢到线程里让他去执行就完了，有时候，我们还想获取异步任务的状态，以及管理 Executor 执行器本身。

因此，ExecutorService 继承了 Executor ，再添加几个方法，主要是`submit`方法，用于向线程提交异步任务，然后返回一个 Future，我们再用 Future 来判断异步任务结束没有，或者获取结果。

```java
public interface ExecutorService extends Executor {
    <T> Future<T> submit(Callable<T> task);
    <T> Future<T> submit(Runnable task, T result);
    Future<?> submit(Runnable task);
    // ... 其他方法见下面生命周期部分
}
```

这个接口的默认实现是 `ThreadPoolExecutor`，用于启动一个线程池，另一个实现是 Fork/Join 框架（JDK1.7）。

### submit 和 execute 的区别

`execute()` 是 Executor 接口的方法，表示执行一个 Runnable， `submit()` 是 ExecutorService 接口的方法，内部调用了 `execute()` ，但还会返回一个异步计算结果 Future 对象（也意味着可以做异常处理）。


## ScheduledExecutorService 接口

看起来已经够用了，JUC的设计者Doug Lea大佬觉得哪里还不太够，于是劈里啪啦又写了一个 ScheduledExecutorService 接口，继承 ExecutorService 接口，再添加几个 schedule 方法：

```java
public ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit);
public <V> ScheduledFuture<V> schedule(Callable<V> callable, long delay, TimeUnit unit);
```

ScheduledExecutorService 的功能和 Timer/TimerTask 类似，解决那些需要任务重复执行的问题。这包括延迟时间一次性执行、延迟时间周期性执行以及固定延迟时间周期性执行等。

---

# Future

Callable 的 `call()` 方法可以获取一个返回值，但假设异步任务要执行很久，调用方就会阻塞。**Future是一个接口，用来判断异步计算是否已完成以及帮助我们获取异步计算的结果**。在没有Future之前我们检测一个线程是否执行完毕通常使用`Thread.join()`或者用一个死循环加状态位来描述线程执行完毕。Future是一种更好的方法，能够阻塞线程，检测任务执行完毕，甚至取消执行中或者未开始执行的任务。

```java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();

    // 调用 get 时，如果还没计算完，将阻塞
    V get() throws InterruptedException, ExecutionException;

    // 如果过了设定的时间还没计算完，抛出超时异常
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

`RunnableFuture<V>` 是一个继承了 `Runnable` 和 `Future<V>` 的接口，再添加一个 `run()` 方法。而 `FutureTask<V>` 是它的实现，包含以下四个field：

```java
/** The underlying callable; nulled out after running */
private Callable<V> callable;
/** The result to return or exception to throw from get() */
private Object outcome; // non-volatile, protected by state reads/writes
/** The thread running the callable; CASed during run() */
private volatile Thread runner;
/** Treiber stack of waiting threads */
private volatile WaitNode waiters;
```

Callable 由构造器传入，所以，当我们有一个异步任务 Callable ，可通过 FutureTask<V> 来获取返回值，如：

```java
FutureTask<Long> future = new FutureTask<Long>( () -> {
    Thread.sleep(1000);
    return 5L;
});

new Thread(future).start();

// 执行get时，若异步任务还未结束，主线程将阻塞
Long result = future.get();
System.out.println(result);
```

---

# Executor 生命周期

既然 ThreadPoolExecutor 是 ExecutorService 的实现，而 ExecutorService 又继承 Executor 接口。所以，线程池的生命周期即是 Executor 的生命周期啦。

```java
public interface ExecutorService extends Executor {
    // 用于提交异步任务和获取结果
    <T> Future<T> submit(Callable<T> task);
    <T> Future<T> submit(Runnable task, T result);
    Future<?> submit(Runnable task);

    // 用于管理执行器本身
    void shutdown();
    List<Runnable> shutdownNow();
    boolean isShutdown();
    boolean isTerminated();
    boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;
}
```

## CREATE & RUNNING

当我们 new 一个 Executor 或者通过 Executors 静态工厂构造一个 Executor，线程池即进入了 RUNNING 状态，在此之前是 CREATE 状态。严格意义上讲线程池构造完成后并没有线程被立即启动，只有进行“预启动”或者接收到任务的时候才会启动线程。但是无论如何，构造完后，线程池已经在运行，随时准备接受任务来执行。

## SHUTDOWN & STOP

通过`shutdown()`和`shutdownNow()`来将线程关闭。

- `shutdown()`平缓关闭，线程池停止接受新的任务，同时等待已经提交的任务执行完毕，包括那些进入队列还没有开始的任务，这时候线程池处于SHUTDOWN状态。
- `shutdownNow()`是一个立即关闭过程，线程池停止接受新的任务，同时尝试取消所有正在执行的任务和已经进入队列但是还没有执行的任务，这时候线程池处于STOP状态。

## TERMINATED

一旦`shutdown()`或者`shutdownNow()`执行完毕，线程池就进入TERMINATED状态，此时线程池就结束了。

- `isTerminating()`描述的是SHUTDOWN和STOP两种状态。
- `isShutdown()`描述的是非RUNNING状态，也就是SHUTDOWN/STOP/TERMINATED三种状态。
- 可以调用 `awaitTermination()` 来等待 `ExecutorService` 到达 TERMINATED 状态。

---

参考：
- 《Java并发编程实战》
