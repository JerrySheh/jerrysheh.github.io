---
title: Java简明笔记（五） 泛型编程
comments: true
categories:
- Java
- Java SE
tags: Java
abbrlink: 76bad10f
date: 2018-01-24 16:59:55
---

# 什么是泛型类

假设我们现在有一个存储字符串字典键值对的类，就像这样

```Java
public class Entry {
    private int key;
    private String value;

    // 构造函数：int 类型的 key， String 类型的 value
    public Entry(int key, String value) {
      this.key = key;
      this.value = value;
    }

    public int getKey() { return key; }
    public String getValue() { return value; }
}
```

在这个类中，我们用 int 类型来存储 key 值， 用 String 类型来存储 value 值。

现在，老板要求，除了 int 类型的 key 和 String 类型的 value之外，还得提供其他类型的 key 和 value 。 比如 double 类型的 key， boolean 类型的value。

我们不可能写很多个相似的类，只是换一下类型。8种基本数据类型或许可以这么干，但是存储的是抽象数据类型呢？我们不可能所有类型都写一个对应的类。

为了解决这个问题，我们可以用 Java 泛型： 只写一个类，实例化的时候再写明是什么类型就好了。这就是泛型类。

> 泛型仅仅是java的语法糖，它不会影响java虚拟机生成的汇编代码，在编译阶段，虚拟机就会把泛型的类型擦除，还原成没有泛型的代码。

<!-- more -->


```Java
public class Entry<K, V> {
    private K key;
    private V value;

    public Entry(K key, V value) {
      this.key = key;
      this.value = value;
    }

    public K getKey() { return key; }
    public V getValue() { return value; }
}
```

实例化泛型类

```Java
// new 后面尖括号的类型参数可以省略
Entry<String, Integer> entry = new Entry<>("Fred", 42);
```


---

# 泛型方法

泛型类是带类型参数的类，同理，泛型方法是带类型参数的方法。

一个普通类的泛型方法的例子，swap方法用于交换任何数组中的元素。声明一个泛型方法时，类型参数放在返回类型之前，在这个例子中是 void 前面的 <T> ，说明 T 是一个泛型类型。

```Java
public class Array {
  public static <T> void swap (T[] array, int i, int j)
    T temp = array[i];
    array[i] = array[j];
    array[j] = temp;
}

String[] friends = ...;
Array.swap(friends, 0, 1);
```

调用泛型方法时，不需要指定类型参数。编译器会自动推断。当然，指定也不会错。如

```java
Array.<String>swap(friends, 0, 1);
```

---

# 类型限定（extends）

假设有一个类对象的ArrayList，该类实现了AutoCloseable接口。里面有一个关闭所有的方法。

```Java
public static <T extends AutoCloseable> void closeAll(ArrayList<T> elems) throws Exception {
  for (T elem: elems) elem.close();
}
```

观察`<T extends AutoCloseable>`， 我们用 `extends` 限制了 T 类型是AutoCloseable的子类型。以防传入了一些奇奇怪怪不可接受的类型。

* 多个限制可以用 &， 如`<T extends Runnable & AutoCloseable>`

---

# 类型变异和通配符

// TODO 遇到相关问题时再补充
