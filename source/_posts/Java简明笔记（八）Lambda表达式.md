---
title: Java简明笔记（八）Lambda表达式
comments: true
categories: JAVA
tags: Java
abbrlink: 68278ec8
date: 2018-02-25 16:55:10
---

Lambda 表达式，也可称为闭包，或者匿名函数，它是推动 Java 8 发布的最重要新特性。

Lambda 允许把函数作为一个方法的参数（函数作为参数传递进方法中）。

使用 Lambda 表达式可以使代码变的更加简洁紧凑。

# 语法

我们用 `->` 来表达 Lambda 表达式，`->`之前是输入的参数，`->`之后是输出的结果。

```
(parameters) -> expression
或
(parameters) ->{ statements; }
```

<!-- more -->

# 简单例子

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

---

# 实际例子

## 替代匿名类

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

## 事件处理

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

## 对列表进行迭代

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

# 高阶函数

处理或返回函数的函数称为`高阶函数`。

往后学习到的时候再补充。
