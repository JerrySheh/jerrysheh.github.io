---
title: Java 中几种关键字
categories: JAVA
tags: JAVA
abbrlink: 3d143648
date: 2017-09-07 23:25:47
---
## JAVA 几种关键字


<br />
### 一、 this

#### 1. 方法中，参数名和对象名一样时，用 this.variable 表示对象名

例子

```java
class A{
    private String name;

    public get(String name){
        this.name = name;
    }
}
```

#### 2. 构造方法调用其它方法时

例子


```java
class Person{

    Person(){
        this.eat();  //必须放在构造方法第一行
    }

    eat(){
        ...
    }
}
```
<!-- more -->

#### 3. 对象A传入对象B的参数

例子


```java
class Person{
    public boolean compareAge(Person p){
        if(this.age == p.age)   //在这个例子中，传入的 p 是 you， this.age指 me
        ...
    }
}

class main{
    public static void main(String[] args){
        Person me = new Person();
        Person you = new Person();
        me.compareAge(you);
    }
}
```

<br />

### 二、 static

#### 1. 无论一个类实例化多少对象，它的静态变量只创建一份
#### 2. main方法是静态的，只能调用静态方法
#### 3. 静态方法只能访问静态成员
#### 4. 静态代码块给类成员赋值

例子

```java
class StaticCodeExample{

	static int num;
	static double num2;
	static ...

	Static{
	num = 10;
	num2 = 6.66;
    ...
	}
}
```

<br />

### 三、 final

#### 1. final修饰的类不能被继承
#### 2. final修饰的常量常用大写，如 BIG_NUMBER = 9999；

例子

```java
// wtf 类不能被继承
final class wtf{
    int a;
    void method(){
        ...
    }
}

// wtf2 类可以被继承，但 method 方法不能被重写（Override）
class wtf2{
    int a;
    final void method(){
        ...
    }
}

// wtf3 类可以被继承，但 a 的值不能被修改
class wtf3{
    final int a;
    void method(){
        ...
    }
}
```

<br />

### 四、 abstract （抽象）

#### 1. 抽象类是不具体实现的类，不能被实例化
#### 2. 抽象类必须有子类覆盖所有抽象方法，否则该子类还是抽象类

<br />

#### 几个细节
	1. 抽象类有构造方法，用于给子类对象初始化
	2. 抽象类可以不定义抽象方法
	3. abstract 不可以和 private 同用（因为抽象方法必须被子类重写）
	4. abstract不可以和 final 同用（因为fianl是固定的）
	5. abstract不可以和 static 同用
	6. 抽象类一定是个父类，要被子类继承
<br />

例子


```java
abstract class animal{

    abstract void bark();  //抽象类中的抽象方法
}

class dogs extents animal{

    void bark(){
    System.out.println("wang wang wang")
    }

}
```
