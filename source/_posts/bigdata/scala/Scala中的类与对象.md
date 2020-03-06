---
title: Scala中的类与对象
comments: true
abbrlink: 230ce50d
categories:
- 大数据
- Scala
tags:
  - 大数据
  - Scala
date: 2018-06-20 16:29:29
---

# 类

## 定义类

```scala
class Counter {
    private var value = 0

    //大括号和返回类型可以省略
    def increment(): Unit = { value += 1}
    def current(): Int = {value}
}
```

- 字段没有 `private` 修饰的默认为 `public`
- 用`def`来声明方法
- `Unit`是 `increment()`方法的返回类型
- 方法的最后一条表达式就是返回值，不需要写`return`

<!-- more -->

## 创建对象

```scala
object theCounter{
  def main(args: Array[String]): Unit = {

    // 用 new 创建一个对象
    val myCounter = new theCounter

    //调用对象方法
    myCounter.increment()
    println(myCounter.current())
  }
}
```

- 在`object`里执行
- 调用方法时（）括号可以省略

## =_ （getter 和 setter）

Scala 中没有 getter 和 setter， 如果要修改 private 变量，用 `=_`

```scala
private var value = 0

// 首先定义一个方法，指向私有的 value 变量
def changeValue = value

// 用 _= 来修改
def changeValue_=(newValue: Int): Unit = {
  if (newValue > 0) value = newValue
}
```

---

# 构造器

跟Java构造方法不同，**Scala的主构造器是整个类体**。

需要在类名称后面罗列出构造器所需的所有参数，这些参数被编译成字段，字段的值就是创建对象时传入的参数的值。

```scala
class Counter(val name: String, val mode: Int) {
    //value用来存储计数器的起始值
    private var value = 0
    def increment(step: Int): Unit = { value += step}
    def current(): Int = {value}
    def info(): Unit = {printf("Name:%s and mode is %d\n",name,mode)}
}

object MyCounter{
    def main(args:Array[String]){       
        val myCounter = new Counter("Timer",2)

        //显示计数器信息
        myCounter.info  

        //设置步长
        myCounter.increment(1)  

        //显示计数器当前值
        printf("Current Value is: %d\n",myCounter.current)
    }
}
```

---

# 单例对象

Scala 没有 Java 的静态类或者静态方法。要实现单例，可用 `object`

```scala
object Person {
    private var lastId = 0  //一个人的身份编号
    def newPersonId() = {
        lastId +=1
        lastId
    }
}
```

---

# 伴生对象

Java 中，有些类既有实例方法又有静态方法。在Scala中用**伴生对象**来实现。

**当单例对象与某个类具有相同的名称时，它被称为这个类的“伴生对象”。**

```scala
class Person {
    //调用了伴生对象中的方法
    private val id = Person.newPersonId()

    // 给名字赋值
    private var name = ""
    def this(name: String) {
        this()
        this.name = name
    }

    // 打印信息
    def info() {
        printf("The id of %s is %d.\n",name,id)
    }
}

object Person {
    //一个人的身份编号
    private var lastId = 0  

    //被实例化一个Person时调用
    private def newPersonId() = {
        lastId +=1
        lastId
    }

    // main
    def main(args: Array[String]){
        val person1 = new Person("Ziyu")
        val person2 = new Person("Minxing")
        person1.info()
        person2.info()      
    }
}
```

- 在 `object` 中才可以使用 main

---

# apply方法和update方法

- 用括号传递给变量(对象)一个或多个参数时，Scala 会把它转换成对apply方法的调用；
- 当对带有括号并包括一到若干参数的对象进行赋值时，编译器将调用对象的update方法，在调用时，把括号里的参数和等号右边的对象一起作为update方法的输入参数来执行调用

---

# 继承

Scala中的继承与Java有着显著的不同：
- 重写方法必须用 override
- 只有主构造器可以调用父类的主构造器
- 可以重写父类中的字段，也是要用 override

---

# 特质（trait）

trait 类似 java 的接口。trait 可以同时有抽象方法 和 具体方法。

Scala中，一个类只能继承自一个父类，却可以实现多个trait，重用trait中的方法和字段，从而实现了多重继承。

## 定义

```scala
trait CarId{
  var id: Int
    def currentId(): Int     //定义了一个抽象方法
}
```
