---
title: Java中的引用类型
comments: false
abbrlink: ae9388fa
date: 2018-04-14 10:35:48
categories:
- Java
- Java SE
tags: Java
---

# 什么是引用类型


`引用类型（reference type）`是一种基于类的数据类型。Java中，除去基本数据类型外，其它类型都是引用类型。包括Java提供的或者自己定义的class类。

当我们对某个对象声明一个变量的时候，例如

```
Ball b1 = new Ball();
```

变量 b1 事实上指向了这个对象的引用，而不是对象本身。

```
Ball b2 = b1;
```

b2 和 b1 都指向了 ball 类的同一个实例。

Java中有四种引用：
- 强引用（Strong Reference）
- 软引用（Soft Reference）
- 弱引用（Weak Reference）
- 虚引用（Phantom Reference）

<!--more-->

---

# 强引用（Strong Reference）

如果一个对象具有强引用，那垃圾回收器(GC)绝不会回收它。

比如上面 Ball 的例子，就是一个强引用。

```
Ball b1 = new Ball();
```

当内存空间不足，Java虚拟机宁愿抛出OutOfMemoryError错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足的问题。如果不使用时，可以通过 `b1=null;`的方式来弱化引用，帮助GC回收对象。

## ArrayList 中的强引用

```java
private transient Object[] elementData;
public void clear() {
        modCount++;
        // Let gc do its work
        for (int i = 0; i < size; i++)
            elementData[i] = null;
        size = 0;
}
```

可以看到，`clear()`方法不是直接对 elementData 置空，而是对elementData[i] 置空，这样elementData就不会被 GC 回收，避免在后续调用 add()等方法添加元素时进行重新的内存分配。

---

# 软引用（Soft Reference）

如果一个对象只具有软引用，则内存空间足够，垃圾回收器就不会回收它；如果内存空间不足了，就会回收这些对象的内存。

只要垃圾回收器没有回收它，该对象就可以被程序使用。软引用可用来实现内存敏感的高速缓存。

> 软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收器回收，Java虚拟机就会把这个软引用加入到与之关联的引用队列中。

```java
// 强引用
String str=new String("abc");       

// 软引用
SoftReference<String> softRef=new SoftReference<String>(str);   
```

软引用在网页中的应用:
- 如果一个网页在浏览结束时就进行内容的回收，则按后退查看前面浏览过的页面时，需要重新构建
- 如果将浏览过的网页存储到内存中会造成内存的大量浪费，甚至会造成内存溢出
这时候就可以使用软引用

```java
// 获取页面进行浏览
Browser prev = new Browser();               

 // 浏览完毕后置为软引用
SoftReference sr = new SoftReference(prev);   

// 还没有被回收器回收，直接获取
if(sr.get()!=null){
    rev = (Browser) sr.get();           
}else{ // 由于内存吃紧，所以对软引用的对象回收了
    prev = new Browser();               
    sr = new SoftReference(prev);  // 那么就重新构建
}

```

---

# 弱引用（Weak Reference）

弱引用与软引用的区别在于：

- 只具有弱引用的对象拥有更短暂的生命周期。
- 在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，**不管当前内存空间足够与否，都会回收它的内存**。

> 弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回收，Java虚拟机就会把这个弱引用加入到与之关联的引用队列中。

```java
StringBuilder sb = new StringBuilder();

SoftReference<StringBuilder> sbSoftRef = new SoftReference<>(sb);
```

当你想引用一个对象，但是这个对象有自己的生命周期，你不想介入这个对象的生命周期，这时候你就是用弱引用。这个引用不会在对象的垃圾回收判断中产生任何附加的影响。

比如说Thread中保存的ThreadLocal的全局映射，因为我们的Thread不想在 ThreadLocal 生命周期结束后还对其造成影响，所以应该使用弱引用，这个和缓存没有关系，只是为了防止内存泄漏所做的特殊操作。

---

# 虚引用（Phantom Reference）

虚引用主要用来跟踪对象被垃圾回收器回收的活动。

虚引用与软引用和弱引用的一个区别在于：虚引用必须和引用队列 （ReferenceQueue）联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存后，把这个虚引用加入到与之关联的引用队列中。也就是说，**为一个对象设置虚引用的唯一目的是能在这个对象被收集器回收时受到一个系统通知**。

```java
StringBuilder sb = new StringBuilder();

ReferenceQueue<StringBuilder> refQ = new ReferenceQueue<>();

PhantomReference<StringBuilder> sbPhantomRef = new PhantomReference<>(sb, refQ);
```

程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否被垃圾回收。**如果程序发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存回收后采取必要的行动。**

如果引用队列中出现了你的虚引用，说明它已经被回收，那么你可以在其中做一些相关操作。
