---
title: Java简明笔记（三） 接口
comments: true
abbrlink: 32811f1d
date: 2018-01-20 22:27:27
categories:
- Java
- Java SE
tags: Java
---

# 什么是接口

假设有一种整数序列服务，这种服务可以计算前n个整数的平均值。就像这样：

```Java
public static double average(IntSequence seq, int n){
  ...
  return average
}

```

我们传入一个序列seq，以及我们想计算这个序列的前n个数，它返回平均数。

然而，这样的序列可以有很多种形式，比如用户给出的序列、随机数序列、素数序列、整数数组中的元素序列......

<!-- more -->

现在，我们想实现一种单一机制，来处理所有的这些序列。要做到这一点，就得考虑上面序列的共性。

不难知道，我们需要两个方法。
1. 判断是否还有下一个元素
2. 获得下一个元素

我们暂时不去想这两个方法具体怎么实现，只是知道需要有这两个方法。于是，我们的average计算平均数服务可以是这样：

```Java
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

在Java中，我们把这两种方法声明出来，但不实现，这就是接口了。

```Java
public interface IntSequence{
  boolean hasNext();
  int next();
}
```

- 接口中所有的方法默认为public

---

# 实现接口

## 一个实现

现在有一个类，它的序列是一组无限平方数（0,1,4,9,16,25...），我们要用上面的average方法来计算这组平方数前n个数的平均值。那么，这个类必然有`hasNext()`和`next()`这两个方法的`具体实现`。我们就称这个类实现了上面的`IntSequence`接口。

```Java
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

```Java
SquareSequence squares = new SquareSequence();
double avg = average(squares, 100);
```
`squares`是`SquareSequence`类的一个对象，先new一个`squares`对象。然后在`average`方法中传入了这个对象作为序列，并且传入100表示前100个平方数。


## 又一个实现

现在又有一个类，它是一个有限序列。是正整数从个位开始每个位的值。比如1729，那么序列就是9，2，7，1。这个序列必然也有`hasNext()`和`next()`这两个方法的`具体实现`。因此，这个类也实现了上面的`IntSequence`接口。

```Java
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

```Java
IntSequence digits = new DigitSequence(1729);
double avg = average(digits, 100);  //虽然这里传入100，但实际只有4个数
```

值得注意的是，`IntSequence接口`是`DigitSequence类`的父类，所以我们可以指定`digits变量`的类型为`IntSequence接口`类型。


---

* 从父类转换为子类，用`cast`。
* 你只能将一个对象强制转换为它的实际类或者它的父类之一。
* 可以用`instanceof`测试对象是否期望的类型

```Java
// 如果DigitSequence是sequence的父类，if语句为true
if (sequence instanceof DigitSequence) {
  DigitSequence digits = (DigitSequence) sequence;
}
```

* 一个接口可以继承(extend)另外一个接口，在原有方法上提供额外的方法。
* 一个类可以实现多个接口，用逗号隔开。
* 定义在接口中的任何变量自动为 `public static final`。

---

# 接口中可以有哪些方法修饰符？

Java8 的接口方法可以有如下方法定义：

public, abstract, default, static，strictfp

## public

接口中所有的方法都是`public`的，不可以是 protected 或者 private。

## 接口中写 abstract 有什么意义？

其实接口中所有的方法都是 `public abstract` 的（静态方法除外），不写也默认是 abstract。只是可以省略而已。写了也不会错。

## static

接口可以有静态方法（Java 8新特性），但必须提供实现。

## default （默认方法）

可以给接口一个默认实现（默认方法），用`default`修饰。（Java 8新特性）

```Java
public interface IntSequence {

  default boolean hasNext(){
    return true;
   }

  int next();
}
```

默认方法的一个重要用途：**接口演化**

有一个旧接口，一个类实现了这个接口。新版Java中对旧接口增加了一个方法，那么这个类就无法编译了，因为这个类没有实现新增加的方法。这时，如果新增加的方法设为默认方法。那么在类的实例中调用这个方法时，执行的是接口的默认方法，即使这个类没有该方法也得以编译和运行。

### 解决冲突

- 如果一个类实现了两个接口，其中一个接口有默认方法，另一个接口有同名同参数的方法（默认或非默认），那么编译器会报错。可以用`父类.super.方法()`来决定要执行哪个方法。

```Java
//返回 Identified 接口的 getID，而不是 Persons 接口的
public class Employee implements Persons, Identified {
  public int getID() {
    return Identified.super.getID();
  }
}
```

## strictfp

strictfp, 即 strict float point (精确浮点)，这个用得比较少，暂时不深入研究。


---

# Java标准类库的几个常用接口

## Comparable接口

如果一个类想启用对象排序，它应该实现 Comparable 接口。

```Java
public interface Comparable<T> {
  int compareTo(T other);
}
```

String类实现`Comparable<String>`，它的 compareTo 方法是

```Java
int compareTo(String other)
```

Employee类实现`Comparable<Employee>`，它的compareTo方法可以这样写：

```Java
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

### 实现了Comparable后，如何使用？

我们的 Employee 类实现了 Comparable 接口，说明这个类的不同实例之间是可以比较的，比较的方法如下

```java
Employee s1 = new Employee(18,"Xiaoming");
Employee s2 = new Employee(20,"Luohao");
s1.compareTo(s2);
```

可以将这些实例放到一个数组中，然后用`Arrays.sort()`进行排序。

```java
// 一个Employee数组
Employee[] eArr = {e1,e2,e3};

// 对Employee数组进行排序，如何排序？根据上面我们写的compareTo方法
Arrays.sort(eArr);

//遍历输出
for (Employee ei:
     eArr) {
    System.out.println(ei.getID());
}
```

## Comparator接口

我们比较字符串的时候，通常是

```java
String s1 = "hello";
String s2 = "world";
s1.compare(s2);
```

这样会以首字母大小顺序对 s1,s2 进行比较。

但如果我们要比较的是字符串的长度，而不是根据首字母。就无法用Comparable接口的compareTo方法来实现。这时候就需要Comparator接口：

```java
public interface Comparator<T> {
  int compare(T first, T second);   
}
```

我们可以写一个类，叫`LenthComparator`，这个类实现了 `Comparator<>` 接口。 然后重写`compare`方法。

比较字符串长度的实现

```Java
class LenthComparator implements Comparator<String> {
  public int compare(String first, String second) {
    return first.lenth() - second.lenth()
  }
}
```

然后对这个类进行实例化，再应用在两个字符串中，这样就实现了对字符串的长度的比较。

```Java
Comparator<String> comp = new LenthComparator();
if (comp.compare(words[i],words[j]) > 0){
    //...
}
```

可以看到，这种调用是compare对象上的调用，而不是字符串自身。

### 扩展

当我们要对某个对象数组（比如student对象）进行排序的时候，不按comparable实现的比较方法来排序，而是要以另一种方式排序。这时候`Arrays.sort()`提供第二个参数，是一个 Comparator，意思是：以第二个参数（Comparator）制定的规则来对第一个参数（数组）进行排序。

```java
student[] sArr = {s1,s2,s3};
Arrays.sort(sArr, new Comparator<student>(){
    @Override
    public int compare(student o1, student o2) {
        return o1.getAge() - o2.getAge();
    }
});
```

这样的代码比较啰嗦，我们可以用 lambda 表达式替代：

```java
// lambda 表达式会自动进行类型判断
Arrays.sort(sArr, (o1,o2)->(o1.getAge() - o2.getAge()));
```

> 如果你要比较的不是数组，而是一个集合，用`Collections.sort`代替`Arrays.sort`

```java
// 第一种写法，IDEA会提示你可以用方法引用替代
Collections.sort(studentList, ((o1, o2) -> o1.getAge() - o2.getAge()));

// 用方法引用，IDEA会提示你可以用 实例.sort 替代
Collections.sort(studentList, Comparator.comparing(student::getAge));

// fine
studentList.sort(Comparator.comparing(student::getAge));
```

### 继续扩展

```java
//按照名字进行排序
Arrays.sort(arr, Comparator.comparing(Person::getName));

//按照名字长度进行排序
Arrays.sort(arr,Comparator.comparing(Person::getName,(s,t)->Integer.compare(s.length(),t.length())));
Arrays.sort(arr,Comparator.comparingInt(p->p.getName().length()));

//先按照名字进行排序,如果名字相同,再按照地址比较
Arrays.sort(arr,Comparator.comparing(Person::getName).thenComparing(Person::getAddress));
```

## Runable接口

Runable接口用来定义任务。比如我们想把特定的任务丢给一个单独的线程去做。

```Java
class HelloTask implements Runnable {
  public void run {
    // how to run
  }
}

{
Runnable task = new HelloTask();
Thread thread = new Thread(task);
thread.start();
}
```

这样，run方法就在一个单独的线程中去执行了，当前线程可以做别的事。

## Serializable 标记接口

## 什么是序列化

对象流是指将对象的内容进行流化。之后，我们就可以对流化后的对象进行读写操作或网络传输。序列化就是一种用来处理对象流的机制，为了解决在对对象流进行读写操作时所引发的问题。

## Serializable 接口的作用

将需要被序列化的类实现 Serializable 接口，该接口没有需要实现的方法，**implements Serializable 只是为了标注该对象是可被序列化的**。

之后，使用一个输出流(如：FileOutputStream)来构造一个 ObjectOutputStream(对象流) 对象，接着，使用 ObjectOutputStream 对象的 writeObject(Object obj) 方法就可以将参数为obj的对象写出(即保存其状态)，要恢复的话则用输入流。

## UI回调

在GUI中，当用户单击按钮、选择菜单项、拖动滑块等操作时，我们必须指定需要执行的行为。这种行为称为`回调`。

在Java GUI类库中，用接口来回调。如在JavaFX中，报告事件的接口：

```Java
public interface EventHandler<T> {
  void handle (T event);
}
```

一个CancelAction类实现上面的接口，指定按钮单击事件的行为ActionEvent，然后创建该类的对象。

```Java
class CancelAction implements EventHandler<ActionEvent> {
  public void handle (ActionEvent event) {
    System.out.println("Oh shit!");
  }
}

Button cancelButton = new Button("Cancel!");
cancelButton.setOnAction(new CancelAction());
```

---

# 接口(Interface)和抽象类(abstract class)的区别

接口是对动作(行为)的抽象，表示的是"like-a"关系。

抽象类是对类的抽象，表示的是"is-a"关系。

接口注重的是方法，而抽象类注重属性和方法。

抽象类|接口
---|---
可以有构造函数|没有构造函数
可以有普通成员变量|没有普通成员变量，只能有常量
可以有实现方法和抽象方法|有抽象方法，可以有静态方法（java8），如果方法被default修饰就可以实现（java8）
一个类只能继承一个抽象类|接口可以有多个实现

## 什么时候使用接口，什么时候使用抽象类

如果你想实现多继承，那么就用接口，Java不支持多继承，但是可以实现多个接口

接口主要用于模块与模块之间的调用

抽象类主要用于当做基础类使用，即基类

> 举个简单的例子，飞机和鸟是不同类的事物，但是它们都有一个共性，就是都会飞。那么在设计的时候，可以将飞机设计为一个类Airplane，将鸟设计为一个类Bird，但是不能将 飞行 这个特性也设计为类，因此它只是一个行为特性，并不是对一类事物的抽象描述。此时可以将 飞行 设计为一个接口Fly，包含方法fly( )，然后Airplane和Bird分别根据自己的需要实现Fly这个接口。然后至于有不同种类的飞机，比如战斗机、民用飞机等直接继承Airplane即可，对于鸟也是类似的，不同种类的鸟直接继承Bird类即可。从这里可以看出，继承是一个 "是不是"的关系，而 接口 实现则是 "有没有"的关系。如果一个类继承了某个抽象类，则子类必定是抽象类的种类，而接口实现则是有没有、具备不具备的关系，比如鸟是否能飞（或者是否具备飞行这个特点），能飞行则可以实现这个接口，不能飞行就不实现这个接口。 ([例子出处](https://www.cnblogs.com/NewDolphin/p/5397297.html))
