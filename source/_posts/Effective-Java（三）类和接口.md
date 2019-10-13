---
title: Effective Java（三）类和接口
comments: true
categories: Effective Java
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
