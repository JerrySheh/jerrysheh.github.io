---
title: Java并发编程之AQS
categories:
  - Java
  - Concurrent
tags: Java
abbrlink: eecbf398
date: 2020-04-08 22:54:55
---

# 什么是 AQS

[同步工具类](../post/a23f9c20.html) 也叫同步器（Synchronizer）。在使用同步器时，我们发现不同的同步器存在许多共同点，例如 ReentrantLock 和 Semaphore 都支持每次允许一定数量线程通过/等待/取消，也都支持让等待线程执行公平或非公平的队列操作等。

事实上，很多同步工具类在实现时都使用了共同的基类，这就是 `AbstractQueuedSynchronizer（AQS），抽象队列同步器`。

<!-- more -->

从字面上理解 AQS:

- Abstract：抽象类，说明只实现一些通用逻辑，有些方法由子类实现；
- Queue：使用先进先出（FIFO）队列存储数据；
- Synchronizer：实现了同步的功能。

说白了，**AQS就是一个用来构建锁和同步器的框架，我们可以使用 AQS 简单且高效地构造出应用广泛的同步器**。在《Java并发编程实战》一书中，AQS是放在“构建自定义的同步工具”这一章讲解的。当然，JDK里有很多并发工具类也都是基于AQS，包括：

- ReentrantLock
- ReentrantReadWriteLock
- Semaphore
- CountDownLatch
- FutureTask

---

# AQS 的 state

AQS 是一个同步器，同步器的本质作用是协调资源。我们把资源是否可用称为资源的状态。在 AQS 中用一个 `int` 变量表示资源状态。

不同的实现对 `state` 的意义不尽相同，例如：

- ReentrantLock 的 `state` 表示锁持有者重入的次数，每重入一次就加一，当 `state` 降为 0 才表示资源可用；
- Semaphore 的 `state` 则表示剩余的可用资源数量；
- FutureTask 的 `state` 用来表示任务的状态（尚未开始、正在运行、已完成、已取消等）。

```java
// volatile 确保了它的修改对任何线程都是及时可见的
private volatile int state;

// protected 意味着可由子类重写具体实现
protected final int getState() {
    return state;
}

protected final void setState(int newState) {
    state = newState;
}

protected final boolean compareAndSetState(int expect, int update) {
    // See below for intrinsics setup to support this
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

资源有两种模式：

- **独占模式（Exclusive）**：一次只能一个线程获取，如 ReentrantLock。
- **共享模式（Share）**：同时可以被多个线程获取，具体的资源个数可以通过参数指定，如 Semaphore。

一般一种同步器只会用到一种模式，但也有两种模式同时使用的，如 `readWriteLock`。

---

# acquire 获取资源

有了对资源状态的表示，便可通过 `acquire(int arg)` 方法来获取资源。参数 `arg` 表示要获取的资源的个数。

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg)){
      selfInterrupt();
    }
}
```

获取资源时，由同步器判断是否成功，如果是，则线程获得资源，此时会将该线程当作一个节点放入队列头表示占用资源，然后更新同步器的状态；如果否，在把该线程节点放入队列等待的同时，线程将阻塞。该队列是一个先进先出（FIFO）的双端队列，使用两个指针head和tail用于标识队列的头部和尾部。

![AQS数据结构（图片来源 concurrent.redspider.group）](../../../../images/Java/AQS数据结构.png)

队列并不是直接储存线程，而是储存拥有线程的Node节点。

```java
static final class Node {
    // 标记一个结点（对应的线程）在共享模式下等待
    static final Node SHARED = new Node();
    // 标记一个结点（对应的线程）在独占模式下等待
    static final Node EXCLUSIVE = null;

    // waitStatus的值，表示该结点（对应的线程）已被取消
    static final int CANCELLED = 1;
    // waitStatus的值，表示后继结点（对应的线程）需要被唤醒
    static final int SIGNAL = -1;
    // waitStatus的值，表示该结点（对应的线程）在等待某一条件
    static final int CONDITION = -2;
    /*waitStatus的值，表示有资源可用，新head结点需要继续唤醒后继结点（共享模式下，多线程并发释放资源，而head唤醒其后继结点后，需要把多出来的资源留给后面的结点；设置新的head结点时，会继续唤醒其后继结点）*/
    static final int PROPAGATE = -3;

    // 等待状态，取值范围，-3，-2，-1，0，1
    volatile int waitStatus;
    volatile Node prev; // 前驱结点
    volatile Node next; // 后继结点
    volatile Thread thread; // 结点对应的线程
    Node nextWaiter; // 等待队列里下一个等待条件的结点


    // 判断共享模式的方法
    final boolean isShared() {
        return nextWaiter == SHARED;
    }

    Node(Thread thread, Node mode) {     // Used by addWaiter
        this.nextWaiter = mode;
        this.thread = thread;
    }

    // 其它方法忽略，可以参考具体的源码
}
```

`acquire` 方法里的 `addWaiter(Node.EXCLUSIVE)` 也在 Node 里面实现。

```java
// AQS里面的addWaiter私有方法
private Node addWaiter(Node mode) {
    // 生成该线程对应的Node节点
    Node node = new Node(Thread.currentThread(), mode);
    // 将Node插入队列中
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        // 使用CAS尝试，如果成功就返回
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    // 如果等待队列为空或者上述CAS失败，再自旋CAS插入
    enq(node);
    return node;
}
```

线程节点以CAS的方式加入队列后，除头结点外，其余节点都处于阻塞状态。而头结点的下一个结点会不断通过 `acquireQueued` 尝试获取资源。

获取资源的方法除了acquire外，还有：

- acquireInterruptibly：申请可中断的资源（独占模式）
- acquireShared：申请共享模式的资源
- acquireSharedInterruptibly：申请可中断的资源（共享模式）

值得注意的是，这些 `acquire` 前缀的方法，内部都会再调用 `tryAcquire` 方法来判断是否能执行，以防止并发操作带来的安全性问题。

---

# release 释放资源

释放资源时，首先用 `tryRelease` 判断能否释放，如果能，则进入 `unparkSuccessor` 进行释放。

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

释放时先更新同步器的状态，如果新的状态允许队列中某个被阻塞的线程结点获取资源，则解除它的阻塞。

```java
private void unparkSuccessor(Node node) {
    // 如果状态是负数，尝试把它设置为0
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);
    // 得到头结点的后继结点head.next
    Node s = node.next;
    // 如果这个后继结点为空或者状态大于0
    // 通过前面的定义我们知道，大于0只有一种可能，就是这个结点已被取消
    if (s == null || s.waitStatus > 0) {
        s = null;
        // 等待队列中所有还有用的结点，都向前移动
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    // 如果后继结点不为空，
    if (s != null)
        LockSupport.unpark(s.thread);
}
```
