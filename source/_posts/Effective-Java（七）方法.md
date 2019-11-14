---
title: Effective Java（七）方法
comments: true
abbrlink: 387fb533
date: 2019-10-28 21:39:53
categories: Effective Java
tags: Java
---


# Item 49 检查参数有效性

在 Java 7 之后，可以用 `requireNonNull` 来判空，如果为空，自动抛出空指针异常。

```java
this.strategy = Objects.requireNonNull(strategy, "strategy");
```

其内部实现

```java
public static <T> T requireNonNull(T obj) {
    if (obj == null)
        throw new NullPointerException();
    return obj;
}
```

---

# Item 50 防御性复制

假设你编写一个 final 的类 Period  ，其中包含两个 final 的 Date 字段。乍一看，这个类似乎是不可变的，然而 Date 里面的成员却是可以变的。这就像声明了一个 final 数组，数组本身不能变，然而数组里面的元素却可以改变一样。

```java
// Attack the internals of a Period instance
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78);  // Modifies internals of p!
```

解决这一问题的办法是使用 Instant（或 LocalDateTime 或 ZonedDateTime）代替Date，因为 Instant 和其他 java.time 包下的类是不可变的（Item 17）。Date 已过时，不应再在新代码中使用。

如果实在要用像 Date 这样的可变类，在构造器和访问器做拷贝：

```java
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end   = new Date(end.getTime());
    //...

    // 防御性拷贝
    public Date start() {
        return new Date(start.getTime());
    }

    public Date end() {
        return new Date(end.getTime());
    }
}
```

---

# Item 51 设计API

Tips 1：设计API时避免参数过长，三种方式解决参数过长：

1. 将方法分解为多个方法，每个方法只需要参数的一个子集。
2. 创建辅助类来保存参数组（通常是静态成员类）
3. 使用 Builder 模式

Tips 2：设计API时，参数类型优先选择接口而不是类。

Tips 3：与布尔型参数相比，优先使用两个元素枚举类型，例如：

```java
public enum TemperatureScale { FAHRENHEIT, CELSIUS }
```

---

# Item 52 重载

重载方法的选择是在 **编译期**，如下所示会打印两次 `Unknown Collection`，即使运行时第一次传入的是 List，但传入 Collection 的重载方法仍然会被使用。

```java
public class CollectionClassifier {
    public static String classify(List<?> lst) {
        return "List";
    }

    public static String classify(Collection<?> c) {
        return "Unknown Collection";
    }

    public static void main(String[] args) {
        Collection<?>[] collections = {
            new ArrayList<BigInteger>(),
            new HashMap<String, String>().values()
        };

        for (Collection<?> c : collections)
            System.out.println(classify(c));
    }
}
```

记住，重载（overloaded）方法之间的选择是静态的，而重写（overridden）方法之间的选择是动态的。

修复这一问题的办法如下：

```java
public static String classify(Collection<?> c) {
  return c instanceof Set  ? "Set" :
         c instanceof List ? "List" : "Unknown Collection";
}
```

JDK 里面为了避免这一问题，例如 `ObjectOutputStream` 类，会用 `writeBoolean(boolean)`、`writeInt(int)`和`writeLong(long)`，而不是重载write方法。 当然，也有一些违反常规的，需要特别注意。例如，`TreeSet` 的 `remove` 选择的是重载 remove(E) 方法，而  `ArrayList` 的 `remove` 却是 remove(int i)，一个是元素值，一个是元素下标。可能会带来混乱。

---

# Item 53 可变参数

可变参数机制：首先创建一个数组（数组大小为传入的可变参数数量），然后将参数值放入数组中，最后将数组传递给方法。

尽量不要使用可变参数。

---

# Item 54 返回空的数组或集合，不要返回 null

如果你有一个数组或集合，它可能为空，请直接返回一个空的容器，不要返回null

```java
//The right way to return a possibly empty collection
public List<Cheese> getCheeses() {
    return new ArrayList<>(cheesesInStock);
}

//The right way to return a possibly empty array
public Cheese[] getCheeses() {
    return cheesesInStock.toArray(new Cheese[0]);
}
```

如果担心每次都 new 会影响性能，考虑优化如下，跟上面的区别在于 `Collections.emptyList()` 永远是同一个，不同每次都 new 一个新的空集合。

```java
// Optimization - avoids allocating empty collections
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? Collections.emptyList()
        : new ArrayList<>(cheesesInStock);
}

// Optimization - avoids allocating empty arrays
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
    return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```


---

# Item 55 Optional

Java 8 加入了 `Optional<T>`，本质上是一个集合（但没有实现`Collection<T>`接口）。例如，本来应该返回 Student 的，但如果 Student 可能为空，那么就有返回 Student 或返回 null 两种情况。

Optional把两种情况变成一种，统一返回 `Optional<Student>`。

```java
public static Optional<Student> getFirstStu(Collection<Student> c) {
    // 如果为空，返回 empty
    if (c.isEmpty())
        return Optional.empty();
    // 非空返回
    else
        return Optional.of(c.get(0));
}
```

并不是所有的返回类型都适合用 Optional。容器类型，包括集合、映射、Stream、数组和 Optional，不应该封装在 Optional 中。与其返回一个空的`Optional<List<T>>`，还不如返回一个空的 `List<T>`

---

# Item 56 编写文档注释

这一条还是直接翻原书吧。
