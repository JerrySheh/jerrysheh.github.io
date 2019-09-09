---
title: Effective Java（一）创建和销毁对象
comments: true
categories: JAVA
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

# Item 3 使用私有构造器或枚举实现单例

未完待续
