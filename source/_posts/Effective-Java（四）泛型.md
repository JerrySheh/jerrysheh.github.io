---
title: Effective Java（四）泛型
abbrlink: 53a4cf82
date: 2019-11-20 21:38:49
categories: Effective Java
tags: Java
---

Java 5 加入了泛型。在有泛型之前，你必须转换从集合中读取的每个对象。如果有人不小心插入了错误类型的对象，则在运行时可能会失败。使用泛型，你告诉编译器在集合中允许存放哪些类型的对象。编译器会自动插入强制转换，并在编译时告诉你是否尝试插入错误类型的对象。

---

# Item 26 使用泛型，不要使用原始类型

如果使用原始类型的集合，在一个字符串集合里插入一个数字是合法的，可能到运行时才出现问题。

```java
List names = ...;
names.add("jerry");
names.add(1.0);

for (Iterator i = names.iterator(); i.hasNext(); )
    String s = (String) i.next(); // Throws ClassCastException

```

但是如果使用泛型，当你向字符串集合插入数字或其他类型时，编译器会报错。使得问题在编译器被发现。

```java
List<String> names = ...;
names.add("jerry");
names.add(1.0); // 编译出错
```

---

# Item 27 消除 unchecked 警告

使用泛型编程时，会看到许多编译器警告：未经检查的强制转换警告，未经检查的方法调用警告，未经检查的参数化可变长度类型警告以及未经检查的转换警告。

例如，如果你写：

```java
Set<Lark> exaltation = new HashSet();
```

编译器会发出 unchecked conversion 警告，修改如下即可消除（JDK 1.7）：

```java
Set<Lark> exaltation = new HashSet<>();
```

尽可能的消灭这些警告，以保证安全。如果你无法消除，但可以确定代码是安全的，可以用 `@SuppressWarnings("unchecked")` 来抑制警告。使用时，请添加注释，说明为什么是安全的。

---

# Item 28 List 优于 数组

你不能把一个 `String` 放到一个 `Long` 类型的容器中，会编译出错。但是同样的情况在数组里却可以编译通过，要等到运行时才会抛出`ArrayStoreException`。

数组和泛型不能混用。`new List<E>[]`，`new List<String>[]`，`new E[]`，这些将在编译时导致泛型数组创建错误。因为泛型数组不是类型安全的。泛型会在编译时被擦除。而数组是具体化的。

---

# Item 29 优先编写泛型元素

如果你要写一个栈 `Stack` 类，如下：

```java
// Object-based collection - a prime candidate for generics
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }

}
```

请考虑把它变成泛型的

```java
// Initial attempt to generify Stack - won't compile!
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new E[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        E result = (E) elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }
    ... // no changes in isEmpty or ensureCapacity
}
```

这样一来，客户端可以创建一个 `Stack<Object>`，`Stack<int[]>`，`Stack<List<String>>` 或者其他任何对象的 Stack 引用类型。

---

# Item 30 优先使用泛型方法

使用泛型方法的好处是类型安全，把运行时可能的错误提前到编译期。

before：

```java
// Uses raw types - unacceptable! [Item 26]
public static Set union(Set s1, Set s2) {
    Set result = new HashSet(s1);
    result.addAll(s2);
    return result;
}
```

after：

```java
// Generic method
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;

}
```
