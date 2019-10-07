---
title: Effective Java（一）创建和销毁对象
comments: true
categories: Java
tags: Java
abbrlink: 39fc1edf
date: 2019-09-09 23:35:58
---

# Item 1 使用静态工厂方法替代构造器

获取一个对象，除了构造方法，还可以用静态工厂方法。例如 jdk 里，获取 Boolean 对象：

```java
public static Boolean valueOf(boolean b){
  return b? Boolean.TRUE : Boolean.FALSE;
}
```

这么做的好处有五个：

1. 相比起构造器，静态工厂方法有名字，当有多个构造方法时容易搞混，静态工厂方法就不会；
2. 静态工厂方法不要求每次都返回一个新对象，可以用来做单例和不可实例化保证；
3. 静态工厂方法可以返回一个对象的子类作为返回类型，而构造器不行，如 java.util.Collections；
4. 静态工厂方法返回对象的类可以根据输入参数的不同而不同；
5. 在编写包含该方法的类时，返回的对象的类不需要存在；

使用静态工厂方法，主要的不足是，没有 public 或 protected 的构造器，因此也无法被子类化。但从另一个角度来说，这也是优点，因为这样做鼓励程序员多用组合，而不是继承，这是好的习惯。第二个不足是程序员可能比较难找到他们，以下是静态工厂方法常用的名字：

- from
- of
- valueOf
- instance / getInstance
- create / newInstance
- getType
- newType
- type

---

<!-- more -->


# Item 2 当构造器参数过多，考虑使用 Builder 模式

这里的 Builder 模式不是指设计模式。假设你要组装一台电脑，有品牌、价格、CPU、是否防水、屏幕尺寸等参数，有些是必选的，有些是可选的，如果用构造器，看起来会像是这样：

```java
public class Computer {
  private String brand;        // required
  private double price;        // required
  private int cpuGeneration;   // optional
  private boolean isWaterproof;// optional
  private int screenSize;      // optional

  Computer(String brand, double price){
    // something here
  }

  Computer(String brand, double price, int cpuGeneration){
    // something here
  }

  Computer(String brand, double price, int cpuGeneration, boolean isWaterproof){
    // something here
  }

  // ...

}
```

你会发现，你要写好多好多不同的构造方法，而且还容易搞混。

最好使用内部 builder 类，在调用方根据需要进行组合，如下：

```java
public class Computer {
    private String brand;        // required
    private double price;        // required
    private int cpuGeneration;   // optional
    private boolean isWaterproof;// optional
    private int screenSize;      // optional

    public static class Builder{
        private String brand;
        private double price;

        // Optional parameters - initialized to default values
        private int cpuGeneration = 3;
        private boolean isWaterproof = false;
        private int screenSize = 12;

        // constructor
        public Builder(String brand, double price) {
            this.brand = brand;
            this.price = price;
        }

        public Builder cpuGeneration(int value){
            cpuGeneration = value;
            return this;
        }

        public Builder isWaterproof(boolean value){
            isWaterproof = value;
            return this;
        }

        public Builder screenSize(int value){
            screenSize = value;
            return this;
        }

        public Computer build(){
            return new Computer(this);
        }
    }

    private Computer(Builder builder) {
        this.brand = builder.brand;
        this.price = builder.price;
        this.cpuGeneration = builder.cpuGeneration;
        this.isWaterproof = builder.isWaterproof;
        this.screenSize = builder.screenSize;
    }
}
```

调用方：

```java
public static void main(String[] args) {

    Computer computer = new Computer.Builder("MicroSoft", 6500.00)
            .cpuGeneration(7)
            .isWaterproof(false)
            .screenSize(24)
            .build();

    System.out.println(computer.toString());

}
```

---

# Item 3 使用私有构造器或枚举实现单例（Singleton）

> 注意: 不适用于多线程情况。

单例用于一个类只允许一个实例对象的情况，通常有两种方法实现单例：**公有域** 和 **公有静态方法**。两种方式都是通过 **私有构造器 + 公开静态成员** 来实现的。

第一种方法：公有域如下，客户端通过 `Elvis.INSTANCE` 来获取唯一对象。

```java
// Singleton with public final field
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
}
```

第二种方法：公有静态工厂如下：

```java
// Singleton with static factory
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTANCE; }
}
```

需要注意的是，有特权的客户端可以通过反射`AccessibleObject.setAccessible`的方式来调用私有构造方法。如果需要避免这个潜在的问题，可以修改构造函数，使其在请求创建第二个实例时抛出异常。

当需要序列化单例类对象时，仅仅用 `implements Serializable` 是不够的，因为每一次反序列化都会创建一个新的实例，解决办法是声明所有成员为 `transient`，然后用以下方法来返回实例。

```java
// readResolve method to preserve singleton property
private Object readResolve() {
     // Return the one true Elvis and let the garbage collector
     // take care of the Elvis impersonator.
    return INSTANCE;
}
```

最后还有一种用枚举实现单例的方式：

```java
// Enum singleton - the preferred approach
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() { ... }
}
```

用这种方式无需担心序列化问题和反射攻击，但是如果单例类需要继承除 enum 外的其他父类，就不能使用这种方法。

---

# Item 4 使用私有构造器实现不可实例化

有些类（如工具类）只包含静态域和静态方法，为了避免被误用，可以将其构造器设置为私有，从而不可实例化。

```java
// Noninstantiable utility class
public class UtilityClass {
    // Suppress default constructor for noninstantiability
    private UtilityClass() {
        throw new AssertionError();
    }
    ... // Remainder omitted
}
```

为什么不用抽象类来实现不可实例化呢？因为抽象类可以被继承，其子类可以被实例化，并且会误导用户认为该类是为继承而设计的。
