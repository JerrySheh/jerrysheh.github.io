---
title: Java简明笔记（二） 面向对象
comments: true
categories: JAVA
tags: Java
abbrlink: ba7990ce
date: 2018-01-18 22:31:07
---


《Core Java for the Impatient》简明笔记。

本章要点：
* Mutator方法改变对象的状态，Accessor方法不改变对象状态；
* Java中，变量不持有对象，只能持有对象的引用；
* 实例变量和方法实现是在类的内部声明的；
* 实例方法是通过对象调用的，通过this引用可访问该对象；
* 构造函数和类名相同。一个类可以有多个构造函数（Overload,重载）；
* 静态变量不属于任何对象。静态方法不是通过对象调用的；
* 类是按包的形式来组织的。使用import声明，这样程序中就不必使用包名；
* 类可以嵌套在其他类中；
* 内部类是非静态嵌套类。它的实例有个外部类对象的引用，这个外部类构建了内部类；
* Javadoc工具处理源代码文件，根据声明和程序员提供的注释产生HTML文件。


<!-- more -->

---

# 面向对象的三个特征

封装、继承、多态

---

# 对象和方法

* `Mutator`方法和`Accessor`方法
> 如果一个方法改变了调用它的对象，我们就说这是一个`Mutator`方法（更改器），反之如果不改变调用自己的对象，它就是`Accessor`方法 （访问器）。比如plusDays方法如果改变Date对象的状态，不返回结果，就是`Mutator`方法，如果plusDays不改变Date对象而是返回一个新构造的LocalDate对象，就是`Accessor`方法。

* Java中，变量只能持有对象的引用。引用是与实现相关的一种定位对象的方式。
* 在类的实例上运行的方法称为`实例方法`。
* Java中，所有没有被声明为`static`的方法都是实例方法。

---

# this

## 表示实例变量

 在对象上调用方法时，this引用指向该对象。this清晰地区分了局部变量和实例变量。带有this的是实例对象。

```Java
public void raiseSalary(double byPercent){
  double raise = this.salary * byPercent / 100;
  this.salary += raise;
}
```

不想给参数起不同的名称时，也可使用this

```Java
public void setSalary(double salary){
  this.salary = salary;
}
```

## 构造函数

一个类可以有多个构造函数，一个构造函数可以调用另一个构造函数，用this。且只能写在第一行。

```java
public Employee (double salary){
    this("", salary);
    //...
}

```

---

# 值传递

* 当你将对象传递给方法，方法获得该对象引用的拷贝。
* Java中，所有参数，对象引用以及基本类型值都是值传递。

下面的例子无法工作，因为sales被复制进x,然后x增加，然而x只是局部变量，这并不更改sales

```Java
// 无法工作
public void increaseRandomly(double x){
  double amount = x * generator.nextDouble();
  x += amount;
  }`

boss.increaseRandomly(sales);
```

同样的，不可能写出一个方法将对象引用修改成其他东西。下面的例子中，引用fred被复制进变量e，然后e被设置成不同的引用。当方法退出时，e退出作用域，fred一点都没改变。

```Java
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

# 构造函数

* 构造函数的名称与类名称相同。并且不返回任何类型！
* 构造函数没有返回类型，如果你不小心加了void，那这是一个名称跟类名相同的方法，不是构造函数。
* 构造函数可以重载。
* 如果有多个构造函数，将共有代码放在其中一个构造函数里，然后在其他构造函数可以调用另一个构造函数，这时候调用要使用this。且只能作为构造函数方法体的第一条语句。

```Java
public Employee (double salary){
  this ("", salary);  //调用构造函数 Employee (String, salary)
  //...                 //其他内容
}
```

* 如果构造函数中没有设置实例变量的值，系统会自动设定：数字为0，boolean为False, 对象引用为null。
* 尽量不要忘记给对象引用初始化。假设我们没有在构造函数中将变量name设置为空字符串，那么当有人调用`getName`方法，如`if (e.getName().equals("Jerry"))`，就会导致空指针异常。

---

# final

* 当使用可修改对象的引用时，fianl修饰符只是声明该引用永不改变。修改对象自身是完全合法的。

下面的例子中，方法可能修改friends引用的数组列表，但是它们不能用其他对象替代。特别是，friends不能变成null。

```Java
public class Person{
  private final ArrayList<Person> friends = new ArrayList<>();
  //可以给该数组列表添加元素
  ...
}
```

---

# static

* 如果在类中将变量声明为static，那么该变量属于类，而不是属于对象。
* 静态常量用static final，如: ` public static final double PI = 3.14159265358979323846;`
* 静态方法是指可以不用运行在对象上的方法。
* 静态方法常见的使用：`工厂方法`，也就是返回一个类的新实例的静态方法。

---

# 多态

多态就是**事物在运行过程中存在不同的状态。**

## 三个前提
- 有继承关系
- 子类要重写父类的方法
- 父类引用指向子类对象

## 例子

例如有 Cat 类继承了 Animal 类 ，并重写了 `eat()`、`sleep()` 方法(sleep是静态方法)，并增加了一个 Animal 没有的 `CatchMouse()` 方法。

然后在测试类中实例化
```java
// 这个语句在堆内存中开辟了子类(Cat)的对象，并把栈内存中的父类(Animal)的引用指向了这个Cat对象
Animal am = new Cat();

//调用实例方法
am.eat();

//调用静态方法
am.sleep();
```

可以发现，实例方法 `am.eat()` 输出的是 Cat 类重写后的方法，而静态方法`am.sleep()` 输出的是 Animal 类的方法（尽管 Cat 也重写了 sleep 方法，但运行时不被识别）

## 弊端

假如我们要执行父类没有而子类特有的`CatchMouse()`方法。

```java
am.CatchMouse();
```

结果却编译报错了。

可见，多态**不能使用子类特有的成员属性和子类特有的成员方法。**

那怎么办呢？ 这时候就要用到`向下转型`。

```java
Cat ca = (Cat) am;

ca.CatchMouse();
```

---

# 内部类

Java 中的内部类分为
- 非静态内部类
- 静态内部类
- 匿名类
- 本地类

## 非静态内部类

即在类里面嵌套另一个类，内部类可以直接访问外部类 private 实例属性。

实例化方法：

```
内部类名 实例名 = 外部类实例名.new 内部类名();
```

## 静态内部类

与非静态内部类不同，静态内部类水晶类的实例化 不需要一个外部类的实例为基础，可以直接实例化

实例化方法：
```
内部类名 实例名 = new 内部类名();
```

## 匿名类

通常情况下，要使用一个接口或者抽象类，都必须创建一个子类，有的时候，为了快速使用，直接实例化一个抽象类，并“当场”实现其抽象方法。这就是匿名类。

简单地说，匿名类就是声明一个类的同时实例化它。

- 匿名类可以用 lambda 表达式替代。
- 在匿名类中使用外部的局部变量，外部的局部变量必须修饰为final （但 jdk 1.8 以后不用声明为 final， jdk 会为你加）


```java
class jump implements Runnable {
    public void run(){
        System.out.println("jump now");
    }
}

public class test {
    public static void main(String[] args) {
        //不使用匿名类
        Runnable r = new jump();
        Thread t1 = new Thread(r);
        t1.start();

        //使用匿名类
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("swim now");
            }
        }).start();

        //使用 lambda 表达式
        new Thread( () -> System.out.println("go away now")).start();
    }
}

```

## 本地类

本地类可以理解为有名字的匿名类

与内部类不一样的是，内部类必须声明在成员的位置，即与属性和方法平等的位置。
本地类和匿名类一样，直接声明在代码块里面，可以是主方法，for循环里等等地方
