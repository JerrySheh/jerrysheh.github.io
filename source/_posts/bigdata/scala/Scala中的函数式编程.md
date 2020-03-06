---
title: Scala中的函数式编程
comments: true
abbrlink: bcfc34da
date: 2018-06-10 15:47:29
categories:
- 大数据
- Scala
tags:
- 大数据
- Scala
---

Scala是一门多范式编程语言，混合了 **面向对象编程** 和 **函数式编程** 的风格。

> 在大数据处理中为什么要函数式编程？
>
> 函数式编程的一个重要特性就是**值不可变性**，这对于编写可扩展的并发程序而言可以带来巨大好处。因为它避免了对公共的可变状态进行同步访问控制的复杂问题。简单地说，我们完全不需要进行传统并发编程里面的同步、加锁等操作。

<!-- more -->

---

# 函数字面量

在面向过程和面向对象编程中，数据包括：**类型** + **值**。在函数式编程中，函数也可以跟其他数据类型一样，进行传递和操作。函数的使用跟其他数据类型没什么区别。函数也包括 **类型** + **值**。函数的值，就是函数字面量。

## 函数的类型

考虑下面的例子

```scala
def counter(value: Int): Int = {
     value += 1
 }
```

输入一个 Int， 输出一个 Int ，因此这个函数的类型是：`Int => Int`

## 函数的值

把函数的类型声明部分（def counter）去掉，剩下的就是函数的值了。

在上面的例子中，输入 value， 输出 value+1 ，因此这个函数的值是：

```scala
(value) => {value += 1}
```

## 用函数式的方法定义函数

定义一个 Int 类型变量，可以：

```scala
val num: Int = 5
```

有了函数式编程的思想后，定义一个函数，可以：

```scala
val counter: Int => Int = { (value) => value += 1}
```

1. 我们定义了一个 counter 函数。
2. 这个函数的类型是`Int => Int`，输入一个 Int，输出一个 Int
3. 这个函数的值是`value => value +=1 `，输入一个 value，输出 value + 1

---

# Lambda表达式与闭包

像 `(参数) => 表达式` 这样的定义形式，我们称之为 Lambda 表达式。其实就是用函数式的方法定义函数。例如：

```scala
val myNumFunc: Int => Int = (num: Int) => num * 2
```

Scala有类型推断，因此可以简化为：

```scala
// 省略前面类型
val myNumFunc = (num: Int) => num * 2

// 省略后面类型
val myNumFunc: Int=>Int = (num) => num * 2
```

## 闭包

闭包是一种比较特殊的函数。闭包会引用函数外部的变量。

> vczh：闭包不是“封闭内部状态”，而是“封闭外部状态”。一个函数如何能封闭外部状态呢？当外部状态的 scope 失效的时候，还有一份留在内部状态里面。

例如，定义一个函数：

```scala
val addMore = (x: Int) => x + more
```

变量`more`并不是在函数内部定义的，而是函数外部的变量。

闭包（是一个函数）对捕获的外部变量做出的改变，**在闭包之外是可见的！**

---

# 高阶函数

一个接受其他函数作为参数 或者 返回一个函数 的函数就是高阶函数。

以求和函数为例：

## 不使用高阶函数

```scala
def sumInts(a: Int, b: Int): Int = {
    if (a > b) 0 else a + sumInts(a+1, b)
}
```

## 使用高阶函数

```scala
// 求和函数，传入一个函数、两个 Int
def sum(func: Int => Int, a: Int, b: Int): Int = {
    if (a > b) 0 else a + sum(func, a+1, b)
}

// self函数
def self(x: Int): Int = x

// 重新定义sumInts函数
def sumInts(a: Int, b: Int): Int = sum(self, a, b)
```

sum 函数的参数是 `（Int=>Int, Int, Int）`， 结果是 `Int`， 因此， sum 函数的类型是：

```
（Int=>Int, Int, Int） => Int
```

也就是说，函数sum是一个接受函数参数的函数，因此，是一个高阶函数。

---

# 占位符语法

当每个参数在函数中最多出现一次时，可以使用 `_` 占位符来表示一个或多个参数。

```scala
object Run {
  def main(args: Array[String]): Unit = {

    val list = List(-3, -5, 1, 9, 6, -99)

    val list2 = rList.filter(x=>x > 0)
    println(list2)

    val list3 = List.filter(_ > 0)
    println(list3)

  }
}
```
list2 和 list3 完全一样。也就是说，在这个例子中，`x => x` 和 `_` 完全一样。即把 List 中的每一个元素作为 x 传入。

---

# Scala内置的高阶函数

## map

map 对集合中的每个元素进行操作。

```scala
val numberList = List(-3, -5, 1, 9, 6, -99)

val numberList2 = numberList.map(x => x+1)

println(numberList2)
```

输出： `List(-2, -4, 2, 10, 7, -98)`

## flatMap

flatMap是map的一种扩展。跟 map 不同的是， flatMap 会将子集合进行合并。

```scala
val books = List("haddop", "spark", "scala")
println(books.map(s=>s.toList))
```

输出：`List(List(h, a, d, d, o, p), List(s, p, a, r, k), List(s, c, a, l, a))`

```scala
val books = List("haddop", "spark", "scala")
println(books.flatMap(s=>s.toList))
```

输出：`List(h, a, d, d, o, p, s, p, a, r, k, s, c, a, l, a)`

## filter

遍历一个集合，并从中获取满足指定条件的元素组成一个新的集合。

## reduce

reduce函数能把结果继续和序列的下一个元素做累积计算。结果就是计算的结果。

```scala
val numberList = List(-3, -5, 1, 9, 6, -99)

val numberList2 = numberList.reduce(_ + _)
println(numberList2)
```

输出：-91

### reduceLeft 和 reduceRight

```scala
val a = List(1,7,2,9)
val a1 = a.reduceLeft(_ - _)  //      ((1-7) - 2) - 9 = -17

val a4 = a.reduceRight(_ - _)//     1 - (7 - (2-9) ) = -13
```

## fold

fold意为折叠，跟reduce类似。只不过 fold 提供了一个种子值，集合的第一个元素首先会跟种子值进行运算。

```scala
val numberList = List(1, 9, 6)

val numberList2 = numberList.fold(4)(_ + _)
println(numberList2)
```

输出：20
