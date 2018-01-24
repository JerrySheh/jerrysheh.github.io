---
title: JAVA简明笔记（三） 接口
comments: true
abbrlink: 32811f1d
date: 2018-01-20 22:27:27
categories: JAVA
tags: JAVA
---

《Core Java for the Impatient》简明笔记。

本章要点：

* 接口指定了一组实现类必须提供的方法
* 接口是任何实现该接口的父类。因此，可以将类的实例赋值给接口类型的变量。
* 接口可以包含静态方法。接口中的所有变量默认是 static final 的。
* 接口可以包含默认方法，实现类可以继承或覆盖该默认方法。
* Comparable和Comparator接口是用来比较对象的。

<!-- more -->

---

# 什么是接口

假设有一种整数序列服务，这种服务可以计算前n个整数的平均值。就像这样：

```java
public static double average(IntSequence seq, int n){
  ...
  return average
}

```

我们传入一个序列seq，以及我们想计算这个序列的前n个数，它返回平均数。

然而，这样的序列可以有很多种形式，比如用户给出的序列、随机数序列、素数序列、整数数组中的元素序列......

现在，我们想实现一种单一机制，来处理所有的这些序列。要做到这一点，就得考虑上面序列的共性。

不难知道，我们需要两个方法。
1. 判断是否还有下一个元素
2. 获得下一个元素

我们暂时不去想这两个方法具体怎么实现，只是知道需要有这两个方法。于是，我们的average计算平均数服务可以是这样：

```java
public static double average(IntSequence seq, int n) {
  int count = 0;
  double sum = 0;
  while (seq.hasNext() && count < n){
    count ++;
    sum += seq.next();
  }
  return count == 0 ? 0 : sum / count;
}
```

在JAVA中，我们把这两种方法声明出来，但不实现，这就是接口了。

```java
public interface IntSequence{
  boolean hasNext();
  int next();
}
```

* 接口中所有的方法默认为public

---

# 实现接口

## 一个实现

现在有一个类，它的序列是一组无限平方数（0,1,4,9,16,25...），我们要用上面的average方法来计算这组平方数前n个数的平均值。那么，这个类必然有`hasNext()`和`next()`这两个方法的`具体实现`。我们就称这个类实现了上面的`IntSequence`接口。

```java
public class SquareSequence implements IntSequence {
  private int i;

  public boolean hasNext() {
    return true;
  }

  public int next() {
    i++;
    return i * i;
  }
}
```

获得前100个平方数的平均值：

```java
SquareSequence squares = new SquareSequence();
double avg = average(squares, 100);
```
`squares`是`SquareSequence`类的一个对象，先new一个`squares`对象。然后在`average`方法中传入了这个对象作为序列，并且传入100表示前100个平方数。


## 又一个实现

现在又有一个类，它是一个有限序列。是正整数从个位开始每个位的值。比如1729，那么序列就是9，2，7，1。这个序列必然也有`hasNext()`和`next()`这两个方法的`具体实现`。因此，这个类也实现了上面的`IntSequence`接口。

```java
public class DigitSequence implements IntSequence {
  private int number;

  public DigitSequence(int n) {
    number = n;
  }

  public boolean hasNext() {
    return number !=0;
  }

  public int next() {
    int result = number % 10;
    number /= 10;
    return result;
  }

  public int rest() {
    return number;
  }
}
```

计算1729位数序列的平均值

```java
IntSequence digits = new DigitSequence(1729);
double avg = average(digits, 100);  //虽然这里传入100，但实际只有4个数
```

值得注意的是，`IntSequence接口`是`DigitSequence类`的父类，所以我们可以指定`digits变量`的类型为`IntSequence接口`类型。


---

* 从父类转换为子类，用`cast`。
* 你只能将一个对象强制转换为它的实际类或者它的父类之一。
* 可以用`instanceof`测试对象是否期望的类型

```java
// 如果DigitSequence是sequence的父类，if语句为true
if (sequence instanceof DigitSequence) {
  DigitSequence digits = (DigitSequence) sequence;
}
```

* 一个接口可以继承(extend)另外一个接口，在原有方法上提供额外的方法。
* 一个类可以实现多个接口，用逗号隔开。
* 定义在接口中的任何变量自动为 `public static final`。

---

# 静态方法和默认方法

* 接口可以有静态方法

## 默认方法和作用

* 可以给接口一个默认实现（默认方法），用`default`修饰。

```java
public interface IntSequence {

  default boolean hasNext(){
    return true;
   }

  int next();
}
```

* 默认方法的一个重要用途：接口演化

> 有一个旧接口，一个类实现了这个接口。新版java中对旧接口增加了一个方法，那么这个类就无法编译了，因为这个类没有实现新增加的方法。这时，如果新增加的方法设为默认方法。那么在类的实例中调用这个方法时，执行的是接口的默认方法，即使这个类没有该方法也得以编译和运行。

## 解决冲突

* 如果一个类实现了两个接口，其中一个接口有默认方法，另一个接口有同名同参数的方法（默认或非默认），那么编译器会报错。可以用`父类.super.方法()`来决定要执行哪个方法。

```java
//返回Identified接口的getID，而不是Persons接口的
public class Employee implements Persons, Identified {
  public int getID() {
    return Identified.super.getID();
  }
}
```
---

# Java标准类库的四个使用接口情况

## Comparable接口

如果一个类想启用对象排序，它应该实现Comparable接口。

Comparable接口的一个技术要点：

```java
public interface Comparable<T> {
  int compareTo(T other);
}
```

String类实现`Comparable<String>`，它的compareTo方法是

```java
int compareTo(String other)
```

Employee类实现`Comparable<Employee>`，它的compareTo方法可以这样写：

```java
public class Employee implements Comparable<Employee> {
  ...
  public int compareTo(Employee other) {
    return getID() - other.getID();
  }
}
```

返回正整数（不一定是1），表示x应该在y后面；返回负整数（不一定是-1），表示x应该在y前面；返回0，说明相等。

* 如果返回负整数，有可能溢出，导致结果变成一个大正整数，用`Interger.compare`方法解决。
* 比较浮点数时，应该用静态方法`Double.compare`，不能用两数之差。

## Comparator接口

如果我们要比较的是String字符串的长度，而不是大小。无法用Comparable接口的compareTo来实现。这时候就需要Comparator接口。

```
public interface Comparator<T> {
  int compare(T first, T second)
}
```

比较字符串长度的实现

```java
class LenthComparator implements Comparator<String> {
  public int compare(String first, String second) {
    return first.lenth() - second.lenth()
  }
}
```

实例

```java
Comparator<String> comp = new LenthComparator();
if (comp.compare(words[i],words[j]) > 0)
```

可以看到，这种调用是compare对象上的调用，而不是字符串自身。

## Runable接口

Runable接口用来定义任务。比如我们想把特定的任务丢给一个单独的线程去做。

```java
class HelloTask implements Runnable {
  public void run {
    // how to run
  }
}

Runnable task = new HelloTask();
Thread thread = new Thread(task);
thread.start();
```

这样，run方法就在一个单独的线程中去执行了，当前线程可以做别的事。

## UI回调

在GUI中，当用户单击按钮、选择菜单项、拖动滑块等操作时，我们必须指定需要执行的行为。这种行为称为`回调`。

在JAVA GUI类库中，用接口来回调。如在JavaFX中，报告事件的接口：

```java
public interface EventHandler<T> {
  void handle (T event);
}
```

一个CancelAction类实现上面的接口，指定按钮单击事件的行为ActionEvent，然后创建该类的对象。

```java
class CancelAction implements EventHandler<ActionEvent> {
  public void handle (ActionEvent event) {
    System.out.println("Oh shit!");
  }
}

Button cancelButton = new Button("Cancel!");
cancelButton.setOnAction(new CancelAction());
```
