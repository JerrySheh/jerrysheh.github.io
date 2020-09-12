---
title: Java简明笔记（八）Lambda和函数式编程
comments: true
categories:
- Java
- Java SE
tags: Java
abbrlink: 68278ec8
date: 2018-02-25 16:55:10
---

# 函数式编程

我们平时所采用的 **命令式编程**（OO也是命令式编程的一种）关心解决问题的步骤。你要做什么事情，你得把达到目的的步骤详细的描述出来，然后交给机器去运行。

而函数式编程关心数据的映射，或者说，关心类型（代数结构）之间的关系。这里的映射就是数学上“函数”的概念，即一种东西和另一种东西之间的对应关系。**所以，函数式编程的“函数”，是数学上的“函数”（映射），而不是编程语言中的函数或方法**。

函数式编程的思维就是如何将这个关系组合起来，用数学的构造主义将其构造出你设计的程序。用计算来表示程序，用计算的组合来表达程序的组合。

<!-- more -->

## 函数式编程思想

函数式编程有三个关键点：

1. **函数第一（Functions as first class objects）**：函数跟其他对象一样，一个引用变量可以指向一个函数，就像我们声明一个引用 s 指向一个字符串 String 一样。可惜的是，在 Java 中，函数不是第一的，但 Scala、Kotlin 里面是。
2. **纯函数（Pure functions）**：函数内部不依赖于外部变量。
3. **高阶函数（Higher order functions）**：函数可以作为参数传递进来，也可以作为返回值返回。在 Java 中，一个方法可以接受一个 lambda 表达式，也可以返回一个 lambda 表达式。

纯函数的四个关键点：

1. **无状态(No state)**：函数内部不能使用外部变量。
2. **无副作用（No side effects）**：函数内部不能修改外部变量。
3. **对象不可变（Immutable variables）**：使用不可变对象来避免副作用。如果要修改一个传进来的参数对象，那修改完毕后返回一个新的对象，而不是修改后的该对象本身。
4. **递归（Favour recursion over looping）**：使用递归，而非循环。

---

# Java 中的 Lambda 表达式

Lambda 表达式，也可称为闭包，或者匿名函数，它是推动 Java 8 发布的最重要新特性。Lambda 允许把函数作为参数传递进方法中，也可以在方法中返回一个函数。

使用 Lambda 表达式可以使代码变的更加简洁紧凑。

## 语法

我们用 `->` 来表达 Lambda 表达式，`->`之前是输入的参数，`->`之后是输出的结果。

```java
(parameters) -> expression
```

## 简单例子

比较字符串长度

```java
(String first, String second) -> first.length() - second.length()
```

给出两个数字，求和

```java
(int a, int b) -> a + b;
```

如果结果无法用一个表达式写出，则用大括号，并明确 return

```java
(int a, int b) -> {
  if (a > b) return a * b;
  else return a / b;
}
```

如果没有参数，`->`前面的参数给出空括号

```java
Runnable task = () -> { for (int i = 0; i < 100; i++) do(); };
```

## 实际例子

### 替代匿名类

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

### 事件处理

```java
// Java 8之前：
JButton show =  new JButton("Show");
show.addActionListener(new ActionListener() {
    @Override
    public void actionPerformed(ActionEvent e) {
    System.out.println("Event handling without lambda expression is boring");
    }
});

// Java 8方式：
show.addActionListener((e) -> {
    System.out.println("Light, Camera, Action !! Lambda expressions Rocks");
});
```

### 对列表进行迭代

```java
// Java 8之前：
List features = Arrays.asList("Lambdas", "Default Method", "Stream API", "Date and Time API");
for (String feature : features) {
    System.out.println(feature);
}

// Java 8之后：
List features = Arrays.asList("Lambdas", "Default Method", "Stream API", "Date and Time API");
features.forEach(n -> System.out.println(n));

// 使用Java 8的方法引用更方便，方法引用由::双冒号操作符标示，
// 看起来像C++的作用域解析运算符
features.forEach(System.out::println);
```

---

# Java中的函数式接口

lambda背后的奥秘在于，lambda表达式本质上是一个匿名类，这个匿名类实现了某个只有一个方法的接口。

## 自定义函数式接口

**只有一个未实现的抽象方法的接口称为函数式接口，static 和 default 不影响**。之所以规定不能有多个抽象方法，是因为 lambda 表达式只能接受一个方法。用`@FunctionalInterface`注解检查是否符合函数式接口规范。

```java
// 函数式接口
@FunctionalInterface
public interface Cal {
    int cal(int n1, int n2);
}
```

Cal是一个函数式接口，只有一个 cal 方法。使用的时候可以用 lambda 表达式定义方法做什么。

```java
public static void main(String[] args) {

    // 做加法
    Cal sum = (n1, n2) -> n1 + n2;
    int r1 = sum.cal(10, 20);
    System.out.println(r1); // 30

    // 做减法
    Cal sub = (n1, n2) -> n1 - n2;
    int r2 = sub.cal(10, 20);
    System.out.println(r2); // -10

}
```

当然，我们可以再做一层封装，提供 calculator 方法，接收两个数字和一个表示如何计算的 lambda，返回计算结果。

```java
public static int calculator(int num1, int num2, Cal c){
    return c.cal(num1, num2);
}

public static void main(String[] args) {
    int n = calculator(10, 20, (n1, n2)-> n1 + n2); // 30
}
```

可以看到，calculator 方法传入了一个 lambda 表达式，事实上，这个 lambda 就是 Cal 接口的一个匿名实现，在传参的时候“现场”声明罢了。

---

# Java预置的函数式接口

jdk 1.8 预置了一些函数式接口，在 java.util.function 包里。其中 6 个最常用的基本接口为：

## Consumer<T>

Consumer的中文意思是消费者，接收一个对象 T， 无返回。

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);

    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```

JDK 例子：System.out::println

```java
Consumer<Double> cal = (d) -> System.out.println(d*2);
cal.accept(3.5);
```

## Supplier<T>

Supplier的中文意思是提供者，不接收参数，返回一个对象 T

JDK例子：Instant::now

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

## Predicate<T>

我之前很难理解什么是 Predicate，直到看了 [知乎这个回答](https://www.zhihu.com/question/25404942/answer/53680068)。

其实很简单，接收一个对象T，返回 boolean，这种场景就是 Predicate 。

```java
@FunctionalInterface
public interface Predicate<T> {

    boolean test(T t);

    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }

    default Predicate<T> negate() {
        return (t) -> !test(t);
    }

    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }

    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
}
```

JDK 例子：Collection::isEmpty

## Function<T,R>

接收一个对象T，返回一个对象R

JDK 例子：Arrays::asList

## UnaryOperator<T>

接收一个对象T，返回一个对象T。这个接口实际上继承了 `Function<T,T>`

JDK 例子：String::toLowerCase

## BinaryOperator<T>

接收两个 T 对象，返回一个 T 对象。

这个接口实际上继承了 `BiFunction<T,T,T>`，在 BiFunction 中，接收 T，U 返回 R。

JDK 例子：BigInteger::add

```java
@FunctionalInterface
public interface BinaryOperator<T> extends BiFunction<T,T,T> {

    public static <T> BinaryOperator<T> minBy(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator);
        return (a, b) -> comparator.compare(a, b) <= 0 ? a : b;
    }

    public static <T> BinaryOperator<T> maxBy(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator);
        return (a, b) -> comparator.compare(a, b) >= 0 ? a : b;
    }
}
```

---

# 高阶函数

处理或返回函数的函数称为 **高阶函数**。

## 返回值为 lambda表达式 的方法

`Arrays.sort()`有第二个参数让我们以某种方式排序

升序排序

```java
Arrays.sort(sArr, (o1,o2) -> (o1.compareTo(o2)));
```

降序排序

```java
Arrays.sort(sArr, (o1,o2) -> ( -1 * o1.compareTo(o2)));
```

这样比较麻烦，怎么办呢？可以写一个产生比较器的方法：

```java
public static Comparator<String> compraeInDirection(int direction) {
    return (x,y) -> direction * x,compareTo(y);
}
```

这个方法返回了一个 lambda 表达式，决定了是采用升序排序还是降序排序。

当需要降序排序的时候，直接：

```java
Arrays.sort(sArr, compraeInDirection(-1));
```

`compraeInDirection(-1)` 返回了一个 lambda 表达式，即 `(x,y) -> -1 * x,compareTo(y)` ，这个 lambda 表达式又作为 `Arrays.sort()` 的第二个参数。

---


# 操作符 「::」 和方法引用

在学习lambda表达式之后，我们通常使用lambda表达式来创建匿名方法。然而，有时候我们仅仅是调用了一个已存在的方法。如下：
```java
Arrays.sort(stringsArray,(s1,s2)->s1.compareToIgnoreCase(s2));
```

在Java8中，我们可以直接通过方法引用来简写lambda表达式中已经存在的方法。这种特性就叫方法引用：
```java
Arrays.sort(stringsArray, String::compareToIgnoreCase);
```

操作符`::`将 方法名称 与 类或对象名称分隔开。

可用于
* 类::实例方法

 比如，`String::comparaToIgnoreCase` 等同于 `(x,y) -> x.comparaToIgnoreCase(y)`

* 类::静态方法

 比如，`Objects::isNull` 等同于 `x -> Objects.isNull(x)`

* 对象::实例方法

 比如，`System.out::println` 等同于 `x -> System.out::println(x)`


## 例子

打印列表的所有元素，可以这么做
```java
list.forEach(x -> System.out.println(x));
```

但也可以直接这么做
```java
list.forEach(System.out::println);
```

* this也是可以使用的，比如，`this::equals` 等同于 `x -> this.equals(x)`

---

# 访问闭合作用域的变量

注意，lambda 表达式只能引用 final 或 final 局部变量，这就是说不能在 lambda 内部修改定义在域外的局部变量，否则会编译错误。

考虑下面这个例子：

```java
public static void repeatMessage(String text, int count) {
    Runnable r = () -> {
        for (int i = 0; i < count; i++) {
            System.out.println(text);
        }
    };
}
```

可以发现，在Lambda表达式里，`count`和`text`既不属于Lambda表达式的参数，也不属于Lambda表达式内部定义的变量。但是我们仍然可以使用，原因是Lambda表达式捕获了`repeatMessage`方法的变量值。注意，是变量值，不是变量本身。

也就是说，Lambda表达式访问闭合作用域的变量，只能访问 final 局部变量。不会被修改的值。当然，我们也不能在Lambda表达式里去修改`count`和`text`的值。

假设我们：

```java
for (int i=0 ; i < n ; i++ ) {
  new Thread ( () -> System.out.println(i)).start();
}
```

则会编译出错，因为 i 是会变化的。

---

参考：

- [函数式编程](https://coolshell.cn/articles/10822.html)
- [Java Functional Programming](http://tutorials.jenkov.com/java-functional-programming/index.html)
