---
title: Effective Java（三）类和接口
comments: true
categories:
- Java
- Effective Java
tags: Java
abbrlink: 20ef17da
date: 2019-10-13 16:58:42
---

# Item 15 最小化类和成员的可访问性

一个设计良好的组件，应该隐藏其内部细节，将 API 与它的实现分离开来，外界只与 API 通信。这也是软件设计的基本原则。

Java中的四种访问级别：
- **private** —— 该成员只能在声明它的顶级类内访问。
- **package-private** —— 成员可以从被声明的包中的任何类中访问。从技术上讲，如果没有指定访问修饰符（接口除外，它默认是公共的），这是默认访问级别。
- **protected** —— 成员可以从被声明的类的子类中访问，以及它声明的包中的任何类。
- **public** —— 该成员可以从任何地方被访问。

能用 private 的，绝不用 protected，能用 protected 的，绝不用 public。

<!-- more -->

如果子类方法重写父类方法，子类方法不能有比父类方法更大的访问级别，但可以更小。但如果是实现接口，则实现类方法必须只能是 public。

带有 public 的可变字段的类通常不是线程安全的。

请注意下面的写法，数组 Thing[] 本身是 final 的，但是 Thing[] 里面的元素却是可以变的

```java
// Potential security hole!
public static final Thing[] VALUES = { ... };
```

解决办法如下：

```java
private static final Thing[] PRIVATE_VALUES = { ... };
public static final List<Thing> VALUES = Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
```

---

# Item 16 使用 public 方法来访问属性，而不是 public 属性

对于一个 public类 来说，应该用 private域 + getter setter 方法来访问属性。这体现了封装和面向对象。

但是，如果一个类是 package-private 的，或者是一个 private 内部类，那么暴露它的数据属性就没有什么本质上的错误——假设它们提供足够描述该类提供的抽象。

---

# Item 17 最小化可变性

immutable class 简单来说就是实例不能被修改的类。包含在每个实例中的所有信息在对象的生命周期中是固定的，因此不会观察到任何变化。比如 String，BigDecimal 等。

设计一个 immutable class 的原则如下：

1. 没有修改对象的方法（mutators）
2. 不能继承
3. 所有属性 private final
4. 如果有引用的可变对象，确保它无法被客户端访问到

immutable class 的优点是简单，本质上是线程安全的，不需要同步。缺点是是对于每个不同的值都需要一个单独的对象。

总原则：

1. 坚决不要为每个属性编写一个 get 方法后再编写一个对应的 set 方法，除非有充分的理由使类成为可变类，否则类应该是不可变的。
2. 除非有足够的理由把属性设置为非 final ，否则每个属性都应该设置为 final 的。
3. 构造方法应该创建完全初始化的对象，并建立所有的不变性。

Jdk 中一个好的示范是 CountDownLatch 类，它本身是可变的，但它的状态空间有意保持最小范围。创建一个实例，使用它一次直到完成，一旦 countdown 锁的计数器已经达到零，就不能再重用它。

---

# Item 18 多用组合少用继承

如果软件设计之初，专门为了继承而设计的类，并且有文档说明，使用继承是安全的。但是跨越到 package 之外去继承，就很危险。它打破了封装。

不要继承一个现有的类，而应该给你的新类增加一个私有属性，该属性是现有类的实例引用，这种设计称为 **组合（composition）**。  

什么时候用继承？《Effective Java》作者 Joshua Bloch 的建议：

> 只有在子类真的是父类的子类型的情况下，继承才是合适的。 换句话说，只有在两个类之间存在「is-a」关系的情况下，B 类才能继承 A 类。 如果你试图让 B 类继承 A 类时，问自己这个问题：每个 B 都是 A 吗？ 如果你不能如实回答这个问题，那么 B 就不应该继承 A。如果答案是否定的，那么 B 通常包含一个 A 的私有实例，并且暴露一个不同的 API ：A 不是 B 的重要部分 ，只是其实现细节。

---

# Item 19 为继承编写文档，否则就不要用继承

如果一个类没有设计和文档说明，那么「外来」类去继承是危险的。

测试为继承而设计的类的唯一方法是编写子类。

构造方法绝不能直接或间接调用可重写的方法，因为父类构造方法先于子类构造方法运行，导致在子类构造方法运行之前，子类中的重写方法就已被调用。

如果不想你的类被继承，将类设计为 final，或者让构造器私有，用静态工厂来实例化你的类。

---

# Item 20 接口优于抽象类

Java 8 对接口引入了默认实现，抽象类和接口都允许有实现。

因为 Java 只允许单一继承，所以继承抽象类会有一些限制，而一个类实现多个接口。接口允许构建非层级类型的框架。


---

# Item 21 接口的默认实现

Java 8 提供了接口的默认实现，本质上是为了不改变接口设计的条件下，向接口加入更多方法。其用途主要在接口演化。

考虑一个旧接口有一些旧的实现，后来新需求需要对该接口添加新方法，但是又不想改旧实现，此时就就可以用默认实现。但 **应该尽量避免使用默认实现，因为默认实现可能会破坏接口的功能**。例如 Collection 接口的 removeIf 方法，在很多集合类上都工作正常，但对于 `org.apache.commons.collections4.collection.SynchronizedCollection` ，如果这个类与 Java 8 一起使用，它将继承 removeIf 的默认实现，但实际上不能保持类的基本承诺：自动同步每个方法调用。

默认实现对同步一无所知，并且不能访问包含锁定对象的属性。 如果客户端在另一个线程同时修改集合的情况下调用 SynchronizedCollection 实例上的 removeIf 方法，则可能会导致 `ConcurrentModificationException` 异常或其他未指定的行为。

---

# Item 22 接口仅用来定义类型，不用于导出常量

客户端可以直接用接口引用具体实现类，这是接口的正确使用方式。

糟糕的使用方式是常量接口（constant interface），即只为接口定义常量属性。类在内部使用一些常量，完全属于该类的实现细节，实现一个常量接口会导致这个实现细节在外部可见（因为接口是public的），不符合隐藏实现的原则。

如果你确实想暴露一些常量给外部，用不可实例化的工具类：

```java
// Constant utility class
package com.effectivejava.science;

public class PhysicalConstants {
  private PhysicalConstants() { }  // Prevents instantiation

  public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
  public static final double BOLTZMANN_CONST  = 1.380_648_52e-23;
  public static final double ELECTRON_MASS    = 9.109_383_56e-31;
}
```

客户端调用：

```java
// Use of static import to avoid qualifying constants
import static com.effectivejava.science.PhysicalConstants.*;

public class Test {
    double  atoms(double mols) {
        return AVOGADROS_NUMBER * mols;
    }
    ...
    // Many more uses of PhysicalConstants justify static import
}
```

---

# Item 23 为类设计层次结构，不要混杂类

如果有一个图形类，即可以表示圆又可以表示矩形，那么应该把这个类抽成两个类，共同继承于抽象图形类，而不是在类里用一个标签判断类型，再用 Switch 去走分支。

不好的设计：

```java
class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    double area() {
        switch(shape) {
          case RECTANGLE:
            return length * width;
          case CIRCLE:
            return Math.PI * (radius * radius);
          default:
            throw new AssertionError(shape);
        }
    }

}
```

好的设计：

```java
abstract class Figure {
    abstract double area();
}

class Circle extends Figure {
    final double radius;

    @Override
    double area() { return Math.PI * (radius * radius); }
}

class Rectangle extends Figure {
    final double length;
    final double width;

    @Override
    double area() { return length * width; }
}
```

---

# Item 24 尽量使用静态成员内部类

建议：如果你声明了一个不需要访问宿主实例的成员类，最好把 static 修饰符放在它的声明中，使它成为一个静态成员类，而不是普通成员类。

如果忽略了这个修饰符，每个实例都会有一个隐藏的外部引用给它的宿主实例。存储这个引用需要占用时间和空间。更严重的是，会导致即使宿主类在满足垃圾回收的条件时却仍然驻留在内存中。由此产生的内存泄漏可能是灾难性的。由于引用是不可见的，所以通常难以检测到。

---

# Item 25 一个.java源文件只定义一个顶级类

永远不要将多个顶级类或接口放在一个源文件中。 遵循这个规则保证在编译时不能有多个定义。 这又保证了编译生成的类文件以及生成的程序的行为与源文件传递给编译器的顺序无关。
