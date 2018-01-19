---
title: JAVA简明笔记（二） 面向对象
comments: true
categories: JAVA
tags: JAVA
abbrlink: ba7990ce
date: 2018-01-18 22:31:07
---


《Core Java for the Impatient》简明笔记。

本章要点：
* Mutator方法改变对象的状态，Accessor方法不改变对象状态；
* JAVA中，变量不持有对象，只能持有对象的引用；
* 实例变量和方法实现是在类的内部声明的；
* 实例方法是通过对象调用的，通过this引用可访问该对象；
* 构造函数和类名相同。一个类可以有多个构造函数（Overload,重载）；
* 静态变量不属于任何对象。静态方法不是通过对象调用的；
* 类是按包的形式来组织的。使用import声明，这样程序中就不必使用包名；
* 类可以嵌套在其他类中；
* 内部类是非静态嵌套类。它的实例有个外部类对象的引用，这个外部类构建了内部类；
* javadoc工具处理源代码文件，根据声明和程序员提供的注释产生HTML文件。


<!-- more -->

---

# 第二章：面向对象

---
* `Mutator`方法和`Accessor`方法
> 如果一个方法改变了调用它的对象，我们就说这是一个`Mutator`方法（更改器），反之如果不改变调用自己的对象，它就是`Accessor`方法 （访问器）。比如plusDays方法如果改变Date对象的状态，不返回结果，就是`Mutator`方法，如果plusDays不改变Date对象而是返回一个新构造的LocalDate对象，就是`Accessor`方法。

* JAVA中，变量只能持有对象的引用。引用是与实现相关的一种定位对象的方式。
* 在类的实例上运行的方法称为`实例方法`。
* JAVA中，所有没有被声明为`static`的方法都是实例方法。

---

* 在对象上调用方法时，this引用指向该对象。this清晰地区分了局部变量和实例变量。带有this的是实例对象。

```java
public void raiseSalary(double byPercent){
  double raise = this.salary * byPercent / 100;
  this.salary += raise;
}
```

* 不想给参数起不同的名称时，也可使用this

```java
public void setSalary(double salary){
  this.salary = salary;
}
```

---

* 当你将对象传递给方法，方法获得该对象引用的拷贝。
* JAVA中，所有参数，对象引用以及基本类型值都是值传递。

下面的例子无法工作，因为sales被复制进x,然后x增加，然而x只是局部变量，这并不更改sales

```java
// 无法工作
public void increaseRandomly(double x){
  double amount = x * generator.nextDouble();
  x += amount;
  }`

boss.increaseRandomly(sales);
```

同样的，不可能写出一个方法将对象引用修改成其他东西。下面的例子中，引用fred被复制进变量e，然后e被设置成不同的引用。当方法退出时，e退出作用域，fred一点都没改变。

```java
//无法工作
public class EvilManager{
  ...
  public void replaceWithZombie(Employee e){
    e = new Employee("",0);
  }
}

boss.replaceWithZombie(fred);
```

---

* 构造函数的名称与类名称相同。并且不返回任何类型！
* 构造函数没有返回类型，如果你不小心加了void，那这是一个名称跟类名相同的方法，不是构造函数。
* 构造函数可以重载。
* 如果有多个构造函数，将共有代码放在其中一个构造函数里，然后在其他构造函数可以调用另一个构造函数，这时候调用要使用this。且只能作为构造函数方法体的第一条语句。

```java
public Employee (double salary){
  this ("", salary);  //调用构造函数 Employee (String, salary)
  ...                 //其他内容
}
```

---

* 如果构造函数中没有设置实例变量的值，系统会自动设定：数字为0，boolean为False, 对象引用为null。
* 尽量不要忘记给对象引用初始化。假设我们没有在构造函数中将变量name设置为空字符串，那么当有人调用`getName`方法，如`if (e.getName().equals("Jerry"))`，就会导致空指针异常。

---

* 当使用可修改对象的引用时，fianl修饰符只是声明该引用永不改变。修改对象自身是完全合法的。

下面的例子中，方法可能修改friends引用的数组列表，但是它们不能用其他对象替代。特别是，friends不能变成null。

```java
public class Person{
  private final ArrayList<Person> friends = new ArrayList<>();
  //可以给该数组列表添加元素
  ...
}
```

---

* 如果在类中将变量声明为static，那么该变量属于类，而不是属于对象。
* 静态常量用static final，如: ` public static final double PI = 3.14159265358979323846;`
* 静态方法是指可以不用运行在对象上的方法。
* 静态方法常见的使用：`工厂方法`，也就是返回一个类的新实例的静态方法。


---

* 嵌套类：把一个类放进另一个类内部。
