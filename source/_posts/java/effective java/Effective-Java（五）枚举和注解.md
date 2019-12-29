---
title: Effective Java（五）枚举和注解
comments: true
abbrlink: acf36022
date: 2019-10-22 22:24:26
categories: Effective Java
tags: Java
---

# Item 34 使用枚举类型替代整型常量

如果你需要一组常量，比如球的红绿蓝三种颜色，四则运算的加减乘除操作，用枚举类会比用 `final static int` 或 `String` 好。枚举更具可读性，更安全，更强大。

<!-- more -->

```java
// Enum type with constant-specific method implementations
public enum Operation {
  PLUS  {public double apply(double x, double y){return x + y;}},
  MINUS {public double apply(double x, double y){return x - y;}},
  TIMES {public double apply(double x, double y){return x * y;}},
  DIVIDE{public double apply(double x, double y){return x / y;}};

  public abstract double apply(double x, double y);
}

public static void main(String[] args) {
    double d = Operation.PLUS.apply(1.2,3.4);
}
```

---

# Item 35 使用实例属性替代序数

永远不要从枚举的序号中得出与它相关的值; 请将其保存在实例属性中。

```java
// Abuse of ordinal to derive an associated value - DON'T DO THIS
public enum Ensemble {
    SOLO,   DUET,   TRIO, QUARTET, QUINTET,
    SEXTET, SEPTET, OCTET, NONET,  DECTET;

    public int numberOfMusicians() { return ordinal() + 1; }
}

// good
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians;

    Ensemble(int size) { this.numberOfMusicians = size; }
    public int numberOfMusicians() { return numberOfMusicians; }
}
```

枚举类的 `ordinal` 方法被设计用于基于枚举的通用数据结构，如 `EnumSet` 和 `EnumMap`。除此之外避免使用 ordinal 方法。

---

# Item 36 使用 EnumSet 替代位属性

`java.util.EnumSet` 类可以有效地表示从单个枚举类型中提取的值集合。

```java
// EnumSet - a modern replacement for bit fields
public class Text {
    public enum Style { BOLD,          // 1 << 0   1
                        ITALIC,        // 1 << 1   2
                        UNDERLINE,     // 1 << 2   4
                        STRIKETHROUGH  // 1 << 3   8
                       }

    // Any Set could be passed in, but EnumSet is clearly best
    public void applyStyles(Set<Style> styles) { ... }
}


text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

---

# Item 37 使用 EnumMap 替代序数索引

`java.util.EnumMap` 类可以用作从枚举到值的映射。

```java
class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }
    final String name;
    final LifeCycle lifeCycle;
}

// Using an EnumMap to associate data with an enum
Map<Plant.LifeCycle, Set<Plant>>  plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);

for (Plant.LifeCycle lc : Plant.LifeCycle.values()){
    plantsByLifeCycle.put(lc, new HashSet<>());
}


for (Plant p : garden){
    plantsByLifeCycle.get(p.lifeCycle).add(p);
}

System.out.println(plantsByLifeCycle);
```

---

# Item 38 使用接口模拟可扩展的枚举

枚举类型的扩展性不好，因为它可能要修改类定义。例如加减乘除操作的枚举类 `Operation` ，有时需要让 API 的用户提供他们自己的操作，从而有效地扩展 API 提供的操作集。这个时候，让你的枚举类 `BasicOperation` 实现 `Operation` 接口。这样用户可以自己去实现扩展的运算。

```java
// Emulated extensible enum using an interface
public interface Operation {
    double apply(double x, double y);
}

public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };
    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override public String toString() {
        return symbol;
    }
}
```

用户的实现：

```java
// Emulated extension enum
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };

    private final String symbol;

    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override public String toString() {
        return symbol;
    }
}
```

---

# Item 39 注解优于命名模式

命名模式指的是以某些格式规范命名的方法，如 JUnit4 之前，测试框架要求其用户通过以 `test[Beck04]` 开始名称来指定测试方法。这很容易犯错。JUnit4 之后提供了 `@Test` 注解，更不容易犯错。

有时候，我们一个注解有多个值，通常是：

```java
@ExceptionTest({ IndexOutOfBoundsException.class,
                 NullPointerException.class })
public static void doublyBad() { ... }
```

在 Java 8 之后，如果注解有元注解`@Repeatable`，那么我们可以多次使用同一个注解。

```java
// Code containing a repeated annotation
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad() { ... }
```

除了特定的开发者（toolsmith）之外，大多数程序员都不需要定义注解类型。 但所有程序员都应该使用 Java 提供的预定义注解类型。

---

# Item 40 始终使用 Override 注解

`@Override` 注解可以减少很多低级错误。如重写一个方法，不小心把参数类型写错了，如果有 `@Override` ，编译器会提示你。但如果不写注解，也可以编译通过，然而这是一个重载而不是重写。

---

# Item 41 使用标记接口定义类型

标记接口（marker interface），是没有任何方法和属性的接口，只起到标记作用，表示一个类实现了具有某些属性的接口。 例如 `Serializable` 接口。实现这个接口，表示实现这个接口的类的实例可以被序列化（即写入 `ObjectOutputStream`）。

虽然我们也可以用标记注解，但标记接口类型的存在允许在编译时捕获错误，如果使用标记注解，则直到运行时才能捕获错误。编译时错误检测是标记接口的意图。然好笑的是，JDK 里面 `ObjectOutputStream.writeObject` 却没有利用 `Serializable` ，它的参数被声明为 Object 类型，所以尝试序列化一个不可序列化的对象直到运行时才会失败。

## 所以什么时候应该使用标记注解，什么时候应该使用标记接口？

如果标记是应用于 **除类或接口以外** 的任何元素（如方法、域），则必须使用注解。如果标记是大量使用注解的框架的一部分，则标记注解是明确的选择。

如果只需要标记类和接口，那么问自己问题：「**我是不是想编写一些接收被标记的对象作为参数的方法？**」如果是这样，则应该优先使用标记接口而不是注解。因为这样你可以将接口作为参数，而不是具体的实现类。这将带来编译时类型检查的好处。

总之，标记接口和标记注释都有其用处。 如果你想定义一个没有任何关联的新方法的类型，一个标记接口是一种可行的方法。 如果要标记除类和接口以外的程序元素，或者将标记符合到已经大量使用注解类型的框架中，那么标记注解是正确的选择。

---

参考
- [41. 使用标记接口定义类型](https://sjsdfg.github.io/effective-java-3rd-chinese/#/notes/41.%20%E4%BD%BF%E7%94%A8%E6%A0%87%E8%AE%B0%E6%8E%A5%E5%8F%A3%E5%AE%9A%E4%B9%89%E7%B1%BB%E5%9E%8B)
