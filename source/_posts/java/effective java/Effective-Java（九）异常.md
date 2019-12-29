---
title: Effective Java（九）异常
comments: true
abbrlink: 4e34dae4
date: 2019-11-29 19:50:38
categories: Effective Java
tags: Java
---

# Item 69 异常只用于异常

异常只用于异常的情况，不要用 `try-catch` 捕获 `ArrayIndexOutOfBoundsException ` 并且不做任何处理这种方式来跳出数组遍历。为什么不用 `for-each` 循环呢？

设计良好的 API 不应该强迫它的客户端为了正常的控制流程而使用异常。

<!-- more -->

例如：

```java
for ( Iterator<Foo> i = collection.iterator(); i.hasNext(); ){
    Foo foo = i.next();
    ...
}
```

`hasNext`方法是一个状态测试机。如果 Iterator API 没有设计 hasNext 方法，那么客户端代码将会变得很丑：

```java
/* Do not use this hideous code for iteration over a collection! */
try {
    Iterator<Foo> i = collection.iterator();
    while ( true ) {
        Foo foo = i.next();
        ...
    }
} catch ( NoSuchElementException e ) {
}
```

---

# Item 70 对可恢复的情况使用checked exception，对编程错误使用unchecked exception（runtime exception）

使用 checked exception 还是 unchecked exception 的一个基本原则是，你是不是想让程序恢复。如果是，使用 checked exception。

像 `ArrayIndexOutOfBoundsException `、`NullPointerException` 这类 unchecked exception 异常，可以通过优化代码实现来避免，不应该捕获。而像 `IOException` 是不可避免的，应当捕获并处理。

---

# Item 71 避免对 checked exception 不必要的使用

正如前面所说的，异常只用于异常情况。有些 `try-catch` 实际上可以分解重构成 `if-else`，对于真正有异常的部分才进入 `try-catch` 块。不要过度使用 `try-catch`。

---

# Item 72 优先使用标准的异常

Java 平台类库提供了一组基本的 unchecked exception，它们满足了绝大多数 API 的异常抛出需求。应该尽可能使用。

异常 |	使用场合
---|---
IllegalArgumentException|	非 null 的参数值不正确（如接收非负数的方法传入了-1）
IllegalStateException	|不适合方法调用的对象状态（如对象未被正确初始化前就被调用）
NullPointerException	|在禁止使用 null 的情况下参数值为 null
IndexOutOfBoundsExecption	| 下标参数值越界
ConcurrentModificationException	|在禁止并发修改的情况下，检测到对象的并发修改
UnsupportedOperationException	|对象不支持用户请求的方法

---

# Item 73 异常转译和异常链

高层的实现应该捕获底层的异常，同时抛出可以按照高层抽象进行解释的异常。这种做法称为 **异常转译 （exception translation）**。

例如下面的例子，高层的将 `get(i)` 将底层 `i.next()` 的 `NoSuchElementException` 转译成 `IndexOutOfBoundsException` 并抛出。

```java
/**
 * Returns the element at the specified position in this list.
 * @throws IndexOutOfBoundsException if the index is out of range
 * ({@code index < 0 || index >= size()}).
 */
public E get(int index) {
    ListIterator<E> i = listIterator(index);
    try {
        return(i.next() );
    } catch (NoSuchElementException e) {
        throw new IndexOutOfBoundsException("Index: " + index);
    }
}
```

在转译时，可以形成 **异常链（exception chaining）**，将底层异常带出去。如果低层的异常对于调试导致高层异常的问题非常有帮助，使用异常链就很合适。

```java
// Exception Chaining
try {
... // Use lower-level abstraction to do our bidding
} catch (LowerLevelException cause) {
    throw new HigherLevelException(cause);
}
```

但也不能滥用异常链，使用原则是：优先处理异常，无法处理时才向上抛出或转译抛出，如果底层异常对高层调试有帮助，才用异常链。

---

# Item 74 每个方法抛出的异常都需要创建文档

使用 Javadoc 的 `＠throws` 标签记录一个方法可能抛出的每个unchecked exception，但是不要使用 throws 关键字将 unchecked exception 包含在方法的声明中。

```java
// do this
/**
 * @throws may cause NullPointerException
 **/
public String foo(){

}

// don't do this
public String foo thorws NullPointerException(){

}
```

---

# Item 75 异常输出

输出异常的细节信息应该包含有用的所有参数和字段的值。例如 `IndexOutOfBoundsException` 应该包含下界、上界以及实际使用的下标值。

```
Exception in thread "main" java.lang.IndexOutOfBoundsException: Index: 5, Size: 2
	at java.util.ArrayList.rangeCheck(ArrayList.java:657)
	at java.util.ArrayList.get(ArrayList.java:433)
	at test.Test.main(Test.java:18)
```

> Tips：千万不要在细节消息中包含密码、密钥以及类似的信息！

---

# Item 76 failure atomic

一般而言，失败的方法调用应该使对象保持在被调用之前的状态。具有这种属性的方法被称为具有 **失败原子性（failure atomic）**。也就是说，出现异常并捕获，之后程序恢复，不会因此有其他任何状态的改变。

最简单的方法是用不可变对象。不可变对象的状态永远是一致的。对于在可变对象，获得失败原子性最常见的办法是，在执行操作之前检查参数的有效性 （Item 49）。这可以使得在对象的状态被修改之前，先抛出适当的异常。

```java
public Object pop() {
    if ( size == 0 )
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; /* Eliminate obsolete reference */
    return(result);
}
```

如果取消对初始大小（size）的检查，当这个方法试图从一个空栈中弹出元素时，它仍然会抛出异常。然而，这将会导致 size 字段保持在不一致的状态（负数）之中，从而导致将来对该对象的任何方法调用都会失败。

总而言之，**想办法让在异常抛出前后，对象的状态保持一致**。你可以提前检查（像上面那样），可以调整计算处理的顺序，可以在对象的一份临时拷贝上执行操作，操作完再去代替对象本身。或者是，编写恢复对象状态的代码（不常用）。

虽然一般情况下都希望实现失败原子性，但并非总是可以做到。如果两个线程企图在没有适当的同步机制的情况下，并发地修改同一个对象，这个对象就有可能被留在不一致的状态之中。因此，在捕获了 `ConcurrentModificationException` 异常之后再假设对象仍然是可用的，这就是不正确的。

---

# Item 77 不要忽略异常

忽略一个异常非常容易，只需将方法调用通过 try 语句包围起来，并包含一个空的 catch 块。但是，最好不要这么做。空的 catch 块会使异常达不到应有的目的。

```java
// Empty catch block ignores exception - Highly suspect!
try {
    ...
} catch ( SomeException e ) {
}
```

即使确实不需要做任何处理，把异常记录下来还是明智的做法，因为如果这些异常经常发生，你就可以调查异常的原因。

如果异常确实也不需要记录下来，才选择忽略异常，catch 块中应该包含一条注释，说明为什么可以这么做，并且变量应该命名为 ignored。

```java
try {
    numColors = f.get( 1L, TimeUnit.SECONDS );
} catch ( TimeoutException | ExecutionException ignored ) {
    // Use default: minimal coloring is desirable, not required
}
```
