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

# Callable

Runnable 用于一个异步执行的任务，没有参数和返回值。Callable 与 Runnable 类似，区别是，Callable有返回值，且可以抛出异常。

<!-- more -->

```java
package java.util.concurrent;
@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```

无论是 Runnable 还是 Callable，想要放到独立的线程中去运行，都是需要借助 Thread 类的。

```java
new Thread(callable).start();
```

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

RunnableFuture<V> 是一个继承了 Runnable 和 Future<V> 的接口，再添加一个 `run()` 方法。而 FutureTask<V> 是 RunnableFuture<V> 的实现，包含以下四个field：

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

Callable 由构造器传入，所以，当我们有一个异步任务 Callable ，可通过 FutureTask<V> 转换成 Future，如：

```java
FutureTask<Long> future = new FutureTask<Long>( () -> {
    Thread.sleep(1000);
    return 5L;
});

new Thread(future).start();

Long result = future.get();
System.out.println(result);
```

---

# Executor

## Executor 接口

Executor 只是一个顶级接口，用来执行一个 Runnable。看起来并没有什么卵用。

```java
public interface Executor {
    void execute(Runnable command);
}
```

## ExecutorService 接口

Doug Lea（JUC的设计者）又写了一个 ExecutorService 接口，继承 Executor 再添加几个方法，主要是以下两个：

```java
public interface ExecutorService extends Executor {
    <T> Future<T> submit(Callable<T> task);
    <T> Future<T> submit(Runnable task, T result);
    Future<?> submit(Runnable task);
    // ... 其他方法略
}
```

这两个方法用于向线程提交异步任务，然后返回一个 Future，我们再用 Future 来判断异步任务结束没有，或者获取结果。这个接口的默认实现是 `ThreadPoolExecutor`，用于启动一个线程池，另一个实现是 Fork/Join 框架（JDK1.7）。

### submit 和 execute 的区别

`execute()` 是 Executor 接口的方法，表示执行一个 Runnable， `submit()` 是 ExecutorService 接口的方法，内部调用了 `execute()` ，但还会返回一个异步计算结果 Future 对象（也意味着可以做异常处理）。

## ScheduledExecutorService 接口

看起来已经够用了，Doug Lea大佬觉得哪里还不太够，于是劈里啪啦又写了一个 ScheduledExecutorService 接口，继承 ExecutorService 接口，再添加几个 schedule 方法：

```java
public ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit);
public <V> ScheduledFuture<V> schedule(Callable<V> callable, long delay, TimeUnit unit);
```

ScheduledExecutorService 的功能和 Timer/TimerTask 类似，解决那些需要任务重复执行的问题。这包括延迟时间一次性执行、延迟时间周期性执行以及固定延迟时间周期性执行等。

后来 Doug Lea 又写了几个例如 CompletionService 等接口，这里就不展开讲了。线程执行Executor接口大家族完善后，接下来就要开始实现了。

---

# Fork/Join 框架（JDK1.7）

todo

---

# Stream并行计算（JDK1.8）

todo
