---
title: Java简明笔记（四） 继承
comments: true
categories:
- Java
- Java SE
tags: Java
abbrlink: 2fJava80
date: 2018-01-23 18:14:26
---

# 什么是继承

继承是在现有的类的基础上创建新类的过程。继承一个类，你也就重用了它的方法，而且还可以添加新的方法和域。

举个例子：员工有薪水，管理者有薪水+奖金， 管理者继承员工，增加 bounus 字段和 setBonus 方法即可。这种情况就是管理者类继承了员工类。

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

# 方法覆盖（Override，重写）

`Employee`类有个`getSalary`方法，返回员工的总薪水。对于管理层来说，除了工资外，还有奖金，于是`Employee`的`getSalary`方法不适用，我们需要重写。这个过程就叫方法覆盖（重写）。

```Java
public class Manager extends Employee {
  //...

  @Override
  public double getSalary() {
    return super.getSalary() + bonus;
  }
}
```

`super.getSalary()` 是 Employee 类的方法。也就是说，我们可以用 `super` 来调用父类方法。**方法覆盖之后还是可以调用父类方法。**

## 重写一个方法，必须匹配准确的参数类型

假如 Employee 类有一个方法

```Java
public boolean workdsFor (Employee supervisor){
  ...
}
```

我们现在要在`Manager`类重写这个方法，如果我们在 Manager 类这样写：

```Java
@Override
public boolean workdsFor (Manager supervisor){
  ...
}
```

那这不是一个重写的方法，而是一个新方法，因为`类型参数`不一样！

正确的重写应该是：

```Java
@Override
public boolean workdsFor (Employee supervisor){
  ...
}
```

因此，为了避免发生这样的失误，最好在我们重写方法的前面加上`@Override`，以注明这是一个重写方法，当我们失误参数写错时，编译器会报错。

## Override 和 Overload 的区别

Override 是方法重写，子类对父类方法的重写。**需要注意的是，重写方法参数类型不能改，但是返回类型可以改（比父类更小或相等）。**

Overload 是方法重载，同一个类中可以有多个名称相同但参数个数、类型或顺序不同的方法。**与函数的返回类型无关** 。

重写和重载都不要求返回类型，因为 Java 中调用函数并不需要强制赋值。

Overload 和 Overwrite 都与访问控制符（public private protected）无关！但一般不做修改。



---

# 初始化：子类构造函数调用父类构造函数

Manager的构造函数不能访问Employee的私有变量，所以我们要用`super`关键字调用父类的构造函数来初始化。

```Java
// 子类构造方法
public Manager (String name, double salary) {

  // 调用父类构造方法
  super(name, salary);

  bonus = 0;
}
```

---

# 父类赋值

在 Java 中，将一个子类对象赋给父类变量是可以的。Java有动态查找（多态），即使是 Employee 类型，执行的时候还是会执行 Manager 的方法。

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

//如果 empl 可以向下转型为 Manager，则类型转换
if (empl instanceof Manager) {
  Manager mgr = (Manager) empl;
  mgr.setBonus(10010);
}
```

参考：[Java简明笔记（二） 面向对象](../post/ba7990ce.html#多态)

---

# final、abstract、interface

* final方法不能被覆盖，final类不能被继承。
* abstract方法没有实现，abstract类不能被实例化。
* Java中，**类比接口优先（class win）**。因此一个类继承了另一个类，又实现了某个接口，碰巧父类和接口有同名方法。这时，默认为父类的实现。<font color="red"> 注意：声明一个类时，先 extends，再 implements，否则编译错误 </font>

```java
public class A extends B implements C {
  //...
}
```

---

# 终极父类：Object

Object 是 Java 中所有类的父类。可以把任何一种数据类型的变量赋给 Object 类型的变量（基本数据类型也可以，会自动装箱）。


Object类有几个重要的方法：

## clone方法

```java
protected Object clone()
```

用于创建并返回此对象的一个副本，实现对象的浅复制。**注意**：只有实现了 Cloneable 接口才可以调用该方法，否则抛出 CloneNotSupportedException 异常。


## ToString方法

```java
public String toString()
```

用于返回该对象的字符串表示。许多 toString 方法都采用一种格式： 类名后面跟中括号，里面是实例变量。

例如： Point 类的 toString 输出：

```java
Java.awt.Point[x=10, y=20]
```

所以，在我们的Employee方法中，可以重写 toString 为

```java
public String toString() {
  return getClass().getName() + "[name=" + name + ",Salary=" + salary + "]"
}
```

提示：打印多维数组用 `Array.deepToString` 方法。

## equals方法

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

用于判断一个对象是否与另一个对象相等。注意，判断的是对象引用是否相同。

考虑下面的例子：

```java
public static void main(String[] args) {
Object o = new Object();
Object oo = new Object();
System.out.println(o.equals(oo)); // 输出：false

String s = new String("aaa");
String ss = new String("aaa");
System.out.println(s.equals(ss)); // 输出：true
}
```

为什么同样是 new 对象， 两个 Object 返回 false， 两个 String 却返回 true 呢？ 原因是：String 重写了 equal() 方法，不是用 == 来判断的，而是比较值。具体看：[探究 String 类 equals 方法源码](../post/689b9445#%E6%8E%A2%E7%A9%B6-String-%E7%B1%BB-equals-%E6%96%B9%E6%B3%95%E6%BA%90%E7%A0%81)

提示：
* 一般情况下，我们不需要重写equals方法。
* 对于基本数据类型，用“==”，但是在 double 中，如果担心正负无穷大或NaN，用`Double.equals`。
* 对于对象，如果担心对象为null，用`Object.equals(x, y)`，如果x为空，返回false。而如果你常规的用`x.equals(y)`则会抛出异常。
* equals 方法的注释中提示我们，如果重写了 equals 方法，最好也重写 hashcode 方法。

### 什么时候需要重写 equals，什么时候需要重写 hashcode ？

#### 重写 equals
默认的，在 Object 类中比较的是两个对象的地址(==) ，地址相同，即同一个对象， equals 返回真。 但是，当某些类我们希望只要某个或某些属性相同，就认为他们是相同的，这时候就需要重写 equals 。

例如 String ，可能 new 了两个 String 对象，但是存的都是一样的字符数组。他们的地址是不一样的，但是我们也说是 equals 的。

#### 重写 hashcode

当重写了 equals 的时候，也必须重写 hashcode。 以免发生 equals 为真，hashcode 却为假的情况。这违背了 hashcode 的性质。

举例来说，HashSet 会用 hashcode 方法得到 hash 值，并以这个 hash 值决定存入 HashSet 的位置。当我们想用 HashSet 存储对象时，两个 equals 的对象都被存入了 HashSet ，这跟 HashSet 储存不重复元素的原则不符。

## wait方法

```java
public final void wait() throws InterruptedException
public final native void wait(long timeout) throws InterruptedException
public final void wait(long timeout, int nanos) throws InterruptedException
```

wait方法用于让当前线程等待，直到其他线程调用此对象的 `notify()` 方法或 `notifyAll()` 方法。

`wait()` 使当前线程等待该对象的锁。当前线程必须是该对象的拥有者，也就是具有该对象的锁。`wait()` 方法一直等待，直到获得锁或者被中断。

`wait(long timeout)` 设定一个超时间隔，如果在规定时间内没有获得锁就返回（继续执行后面的代码），不会抛超时异常。

调用`wait(long timeout)`后当前线程进入睡眠状态，直到以下事件发生：

1. 其他线程调用了该对象的 notify 方法
2. 其他线程调用了该对象的 notifyAll 方法
3. 其他线程调用了 interrupt 中断该线程
4. 时间间隔到了

此时该线程就可以被调度了，如果是被中断的话就抛出一个 InterruptedException 异常。

## notify方法

```java
public final native void notify();
```

唤醒在该对象上等待的某个线程

`notify()`是对对象锁的唤醒操作。但有一点需要注意的是：`notify()` 调用后，并不是马上就释放对象锁的，而是在相应的 `synchronized(){}` 语句块执行结束，自动释放锁后，JVM会在 `wait()` 对象锁的线程中随机选取一线程，赋予其对象锁，唤醒线程，继续执行。这样就提供了在线程间同步、唤醒的操作。

## notifyAll方法

```java
public final native void notifyAll();
```

唤醒在该对象上等待的所有线程。


## hashCode方法

```java
public native int hashCode();
```

哈希码是个来源于对象的整数。哈希码应该是杂乱无序的，如果 x 和 y 是两个不相等的对象，他们的`hashCode`方法很可能不同。

### 引申：hashcode的作用

hashCode用于返回对象的散列值，用于在散列函数中确定放置的桶的位置。

1. hashCode的存在主要是用于查找的快捷性，如Hashtable，HashMap等，hashCode是用来在散列存储结构中确定对象的存储地址的；
2. 如果两个对象相同，就是适用于equals(java.lang.Object) 方法，那么这两个对象的hashCode一定要相同；
3. 如果对象的equals方法被重写，那么对象的hashCode也尽量重写，并且产生hashCode使用的对象，一定要和equals方法中使用的一致，否则就会违反上面提到的第2点；
4. 两个对象的hashCode相同，并不一定表示两个对象就相同，也就是不一定适用于equals(java.lang.Object) 方法，只能够说明这两个对象在散列存储结构中，如Hashtable，他们“存放在同一个篮子里”。

参考：[数据结构（六）查找——哈希函数和哈希表](../post/c61c7436.html)


## getClass方法

```java
public final native Class<?> getClass();
```

native方法，也是 final 方法，用于返回此 Object 的运行时类型。

## finalize方法

```java
protected void finalize() throws Throwable {
  // 没有任何东西
}
```

一般不要使用finalize，最主要的用途是回收特殊渠道申请的内存。Java程序有垃圾回收器，所以一般情况下内存问题不用程序员操心。但有一种JNI(Java Native Interface)调用non-Java程序（C或C++），finalize()的工作就是回收这部分的内存。

当垃圾回收器确定不存在对该对象的更多引用时，由对象的垃圾回收器调用此方法。见 [Java虚拟机（四）垃圾回收策略](../post/2191536a.html)。
