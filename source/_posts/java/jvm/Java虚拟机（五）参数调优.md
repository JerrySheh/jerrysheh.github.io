---
title: Java虚拟机（五）JVM参数和调优
comments: true
categories:
- Java
- JVM
tags:
  - Java
  - JVM
abbrlink: a3255dff
date: 2018-10-28 16:22:48
---

# 本地线程分配缓冲（TLAB）

Java虚拟机遇到 new 指令时，需要在堆内存上为新对象分配内存空间。如果堆是规整的，一边是分配过的内存，一边是空闲内存，那只要在中间用一个指针隔开，为新对象分配内存时，指针往后移动相应的空间距离即可。

![pointer_move](../../../../images/Java/pointer_move.png)

<!-- more -->

然而，在多线程环境下，线程A和线程B同时为新对象分配内存，线程A还没来得及改指针位置，线程B也使用了这个位置来分配内存，就会出现问题。有两种方法解决这个问题，第一是采用同步，事实上虚拟机采用的 CAS 失败重试的方式来保证更新内存的原子性。第二种是本地线程分配缓冲（Thread Local Allocation Buffer）。

本地线程分配缓冲会在 Java 堆内存里预先分配一小块内存专门给某个线程用来分配空间，所以不同的线程分配内存是在不同的位置。这样就不会导致冲突。只有当 TLAB 用完并分配新的缓冲区时，才需要同步锁定。

![pointer_move2](../../../../images/Java/pointer_move.png)

在 Java 中，用以下参数来设定是否要开启TLAB

```java
// 启用TLAB（默认）
-XX:+UseTLAB

// 禁用TLAB
-XX:-UseTLAB

// 设置TLAB空间所占用Eden空间的百分比大小(默认1%)
-XX:TLABWasteTargetPercent
```

参考：https://blog.csdn.net/zyc88888/article/details/80361635

---

持续更新
