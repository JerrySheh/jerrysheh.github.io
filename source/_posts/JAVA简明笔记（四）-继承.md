---
title: Java简明笔记（四） 继承
comments: true
categories: Java
tags: Java
abbrlink: 2fJava80
date: 2018-01-23 18:14:26
---

《Core Java for the Impatient》简明笔记。

本章要点：

* 子类可以继承或者覆盖父类的方法。
* 使用关键字`super`来调用父类方法或者构造函数。
* final方法不能被覆盖，final类不能被继承。
* abstract方法没有实现，abstract类不能被实例化。
* 子类方法可以访问子类的受保护成员，但仅适用于同一个子类的对象。
* 所有类都是Object的子类，Object提供了toString、equals、hashCode、clone等方法。

---

# 什么是继承

继承是在现有的类的基础上创建新类的过程。继承一个类，你也就重用了它的方法，而且还可以添加新的方法和域。

举个例子。

```Java
public class Manager extends Employee {
  private double bonus;
  ...

  public void setBonus (double bouns) {
    this.bonus = bonus;
  }
}
```

`Manager`类继承了`Employee`类，除了获得Employee类的变量和方法外，还额外添加了bonus变量和setBonus方法。


<!-- more -->
---

# 方法覆盖（重写）

`Employee`类有个`setSalary`方法，返回员工的总薪水。对于管理层来说，除了工资外，还有奖金，于是`Employee`的`setSalary`方法不适用，我们需要重写。这个过程就叫方法覆盖。

```Java

public class Manager extends Employee {
  ...

  public double getSalary() {
    return super.getSalary() + bonus;
  }
}
```

* `super.getSalary()` 是`Employee`类的方法。也就是说，我们可以用`super`来调用父类方法。方法覆盖之后还是可以调用父类方法。


注意，重写一个方法，必须匹配准确的参数类型。

假如Employee类有一个方法

```Java
public boolean workdsFor (Employee supervisor){
  ...
}
```

我们现在要在`Manager`类重写这个方法，如果我们这样写：

```Java
public boolean workdsFor (Manager supervisor){
  ...
}
```

那这不是一个重写的方法，而是一个新方法，因为`类型参数`不一样！

正确的重写应该是：

```Java
public boolean workdsFor (Employee supervisor){
  ...
}
```

因此，为了避免发生这样的失误，最好在我们重写方法的前面加上`@Override`，以注明这是一个重写方法，当我们失误参数写错时，编译器会报错。

```Java
@Override
public boolean workdsFor (Employee supervisor){
  ...
}
```

* 注意，重写方法参数类型不能改，但是返回类型可以改。

---

# 子类构造函数

Manager的构造函数不能访问Employee的私有变量，所以我们要用`super`调用父类的构造函数来初始化。

```Java
public Manager (String name, double salary) {
  super(name, salary);
  bonus = 0;
}
```
---

# 父类赋值

在Java中，将一个子类对象赋给父类变量是可以的。Java有动态查找，即使是Employee类型，执行的时候还是会执行Manager的方法。

```Java
Manager boss = new Manager(...);
Employee empl = boss;  // it is ok.

//执行的是Manager.getSalary
double salary = empl.getSalary();
```

但是这也有一个缺点，那就是只能调用属于父类的方法（getSalary），而不能调用子类方法（getBonus）。

```Java
Employee empl = new Manager(...);
empl.setBonus(10010); //编译报错
```

解决这个问题，可以用`instanceof`操作符。

```Java
Employee empl = new Manager(...);

//如果empl不是Manager类型，则类型转换
if (empl instanceof Manager) {
  Manager mgr = (Manager)empl;
  mgr.setBonus(10010);
}
```

---

## final、abstract、interface

* final方法不能被覆盖，final类不能被继承。
* abstract方法没有实现，abstract类不能被实例化。

* Java中，类比接口优先（class win）。因此一个类继承了另一个类，又实现了某个接口，碰巧父类和接口有同名方法。这时，默认为父类的实现。

---

# 终极父类：Object

Object是Java中所有类的父类。Object类有几个重要的方法。


## ToString方法

许多toString方法都采用一种格式： 类名称后面跟中括号，里面是实例变量。如
调用Point类的toString将会输出：`Java.awt.Point[x=10, y=20]`

所以，在我们的Employee方法中，可以重写toString为

```java
public String toString() {
  return getClass().getName() + "[name=" + name + ",Salary=" + salary + "]"
}
```

* 打印多维数组用 Array.deepToString

## equals方法

equals方法用于判断一个对象是否与另一个对象相等。注意，判断的是对象引用是否相同。

* 一般情况下，我们不需要重写equals方法。
* 对于基本数据类型，用“==”，但是在double中，如果担心正负无穷大或NaN，用`Double.equals`。
* 对于对象，如果担心对象为null，用`Object.equals(x, y)`，如果x为空，返回false。而如果你常规的用`x.equals(y)则会抛出异常。`

## hashCode方法

哈希码是个来源于对象的整数。哈希码应该是杂乱无序的，如果x和y是两个不相等的对象，他们的`hashCode`方法很可能不同。
