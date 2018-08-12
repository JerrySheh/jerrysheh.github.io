---
title: Java简明笔记（一） 基本知识
categories: JAVA
tags: Java
abbrlink: b3088ac5
date: 2017-11-05 22:40:39
---

![java](../../../../images/Java/java.jpg)

# Java 与 C++ 的区别

1. C++支持多重继承，Java不支持，但可以实现多接口。（引申：多重继承菱形问题）
2. 自动内存管理
3. java不支持goto语句
4. **引用与指针**：在Java中不可能直接操作对象本身，所有的对象都由一个引用指向，必须通过这个引用才能访问对象本身，包括获取成员变量的值，改变对象的成员变量，调用对象的方法等。而在 C++ 中存在引用，对象和指针三个东西，这三个东西都可以访问对象。其实，Java中的引用和C++中的指针在概念上是相似的，他们都是存放的对象在内存中的地址值，只是在Java中，引用丧失了部分灵活性，比如 Java 中的引用不能像 C++ 中的指针那样进行加减运算。

---

# 数据类型

在 Java 中，数据类型分为两类：**基本数据类型（primitive type）** 和 **引用类型（reference type）**。

## 九种基本数据类型

基本类型	|大小(字节)	|默认值	|封装类
---|---|---|---
byte|	1|	(byte)0|	Byte
short|	2|	(short)0	|Short
int|	4	|0	|Integer
long|	8	|0L|	Long
float|	4|	0.0f	|Float
double|	8|	0.0d	|Double
boolean|	-	|false	|Boolean
char|	2|	\u0000(null)	|Character
void	|-	|-	|Void


- 在数字后面加L后缀即表示long类型（如 `400000000L`），在数字前面类型转换即可表示Byte或short类型 （如 `(byte)127`）
- 在浮点数后面f后缀表示float，否则默认为double
- 关于 void，有些书认为不属于基本数据类型，虽然 Java api 中并未说明，但有些书籍如《Thinking in Java》将其也划进去。

<!-- more -->

---

# 拆箱和装箱

## 什么是装箱？

一般我们要创建一个类的对象实例的时候，我们会这样：
```java
Class a = new Class(parameter);
```

当我们创建一个Integer对象时，却可以这样：
 ```java
 Integer i = 100; //(注意：不是 int i = 100; )
 ```

实际上，执行上面那句代码的时候，系统为我们执行了：

```java
Integer i = Integer.valueOf(100);
```

这就是基本数据类型的`自动装箱`功能。

同理，拆箱就是把基本数据类型从 Integer 对象取出的过程。

## 基本数据类型与引用类型的区别

基本数据类型不是对象，也就是使用int、double、boolean等定义的变量、常量。基本数据类型没有可调用的方法，而引用类型有方法。比如：

```java
int t = 1;    // t. 后面没有方法
Integer u = 1;// u. 后面就有很多方法可让你调用了
```

---

# equals() 和 "=="

- **`equals()`**：比较的是两个对象的值（内容）是否相同。
- **`==`**：比较的是两个引用所指的对象（或者说内存地址）是否相同，也用来比较两个基本数据类型的变量值是否相等。

前面说过，int 的自动装箱，是系统执行了 `Integer.valueOf(int i)`，看看Integer.java的源码（在IDEA中，按住Ctrl键，鼠标点击 Integer）：

```java
public static Integer valueOf(int i) {
    // 没有设置的话，IngegerCache.high 默认是127
    if(i >= -128 && i <= IntegerCache.high)　　
        return IntegerCache.cache[i + 128];
    else
        return new Integer(i);
}
```

对于–128到127（默认是127）之间的值，`Integer.valueOf(int i)` 返回的是 **缓存的Integer对象！！！**


所以下面的现象也就不难理解了：

```java
//在-128~127 之外的数
Integer i1 = 200;  
Integer i2 = 200;          
System.out.println("i1==i2: "+(i1==i2));   // 输出 false


// 在-128~127 之内的数
Integer i3 = 100;  
Integer i4 = 100;  
System.out.println("i3==i4: "+(i3==i4));  // 输出 true
```

---

# 增强 for 循环 (forEach)

两个例子：

如果 `numbers` 是一个 int[] 数组列表，则使用 forEach 输出所有int
```Java
int sum = 0;
for ( int n : numbers){
  sum += 0;
}
```

如果 `friends` 是一个 String[] 数组列表，则使用 forEach 输出所有String
```java
for (String name : friends) {
  System.out.println(name);
}
```

## foreach与正常for循环效率对比

- 循环ArrayList时，普通for循环比foreach循环花费的时间要少一点；
- 循环LinkList时，普通for循环比foreach循环花费的时间要多很多。

当将循环次数提升到一百万次的时候，循环ArrayList，普通for循环还是比foreach要快一点；但是普通for循环在循环LinkList时，程序直接卡死。

结论：

- 需要循环数组结构的数据时，建议使用普通for循环，因为for循环采用下标访问，对于数组结构的数据来说，采用下标访问比较好。
- 需要循环链表结构的数据时，一定不要使用普通for循环，这种做法很糟糕，数据量大的时候有可能会导致系统崩溃。

原因：**foreach使用的是迭代器**

---

# 零碎知识

## 数学运算相关

1. 最小整数：`Integer.MIN_VALUE`
2. 最大整数：`Integer.MAX_VALUE`
3. long不够用，用 BigInteger 类
4. 1_000_000 = 1000000，编译器会自动删掉下划线（方便阅读）
5. `Double.POSITIVE_INFINITY`表示无穷大，`Double.NEGATIVE_INFINITY`表示负无穷大
6. `Double.NaN`表示非数值
7. 用 `if (Double.isNaN(x))`来检查 `x` 是否为 NaN，但不可以用`if (x == Double.NaN)`，因为NaN都是彼此不同的
8. 浮点数不适合金融计算，用BigDecimal类
9. `BigDecimal.ValueOf(n,e);`，其中n是一个整数数，e是小数位，如(588888,3)，就是 588.888
10. 17/5 的结果是3， 而 17.0/5 的结果是3.4
11. 整数除以零会导致异常，浮点数除以零会产生无限值或NaN
12. 负数慎用 %
13. Math类让算数运算更安全（p15）
14. `Math.round`方法获得数字四舍五入的整数，`int n = (int) Math.round(x)`，若x=3.75，则n=4
15. `( n!=0 && s+(100-s) / n ) < 50`，第二个条件除数等于零可能会报错，但第一个条件排除了这种可能性，所以第一个条件满足时，第二个条件不予评估，也就不会发生错误
16. `time < 12 ? "am" : "pm"`， 若time<12，结果为am，否则为pm
17. ` n & 0xF`的结果为n的最低四位
18.  在32位的int类型中，1<<35的结果跟 1<<3 相同
19.  Java不允许对象直接使用操作符，所以BigDecimal和BigInteer类需要用方法

 ```Java
 BigDecimal next = bd.multiply(bd.add(BigDecimal.valueOf(l)));
 ```

## 变量相关

* 最好每个变量名都有各自单独的声明
* 变量名建议用驼峰式命名，如 countOfInvalidInputs
* 刚好正在首次需要变量的前一刻声明
* 常量用大写字母，如`DAYS_PRE_WEEK = 7;`
* 在其他类中使用Calendar类的常量，只需要前面加上类名，如`Calendar.DAYS_PRE_WEEK`

---

# 格式化输出

## 使用 %0.0 输出浮点数

%8.2f指明输出的浮点数宽度为8，精度为小数点后两位。

```java
// 输出：333.33
System.out.printf("%8.2f", 1000.0 / 3.0);
```

## 使用 %s 输出字符串，%d 输出纯数字

```java
System.out.printf("Hello, %s. Next year you will be %d.", name, age);
String.format("Hello, %s. Next year you will be %d.", name, age);
```

---

# 类型转换

## String 转 其他

```java
Integer.parseInt(Str);
```

## 其他 转 String



```java
//取数字的值，转换为字符串，比如 3.10 转换为 "3.1"
//如果要格式化为 "3.10"，应该用下面的 DecimalFormat 类
String.ValueOf(num);

// Integer 转 String
Integer.toString(num);
```

## 使用 DecimalFormat 类

将数字格式化为字符串

```java
double d = 3.1;

// 预定格式
DecimalFormat df = new DecimalFormat("0.00");

// 把 3.10 格式化，返回 String，s = "3.10"
String s = df.format(d);
```

---

# 数组

## 声明数组

```java
//声明一个有100个元素的字符串数组
String[] names = new String[100];
```

使用 new 构造数组时的默认值：数字（0）， Boolean（flase）, 对象（空引用）

## 遍历数组

```java
// 使用 stream 遍历
Arrays.stream(names).forEach(System.out::println);
```

## 数组元素比较

== 和 != 比较的是对象引用，而不是对象的内容。所以比较数组不同下标元素内容是否相同时，不要用 == 应该用 **equals**。

```java
// DON'T
if (numbers.get(i) == numbers.get(j));

// DO
if (numbers.get(i).equals(numbers.get(j)));
```


## 使用 ArrayList 代替数组

构造数组时，需要知道数组的长度，一旦构造出来长度便不能改变。在许多实际应用中很不方便。所以需要 Java.util 包中的 ArrayList 类来根据需求随时增长或者缩减数组的长度。ArrayList是泛型类。

```Java
ArrayList<String> friends;
friend = new ArrayList<>();
friend.add("Peter");
friend.add("Paul");
friend.remove(1);

String first = friends.get(0);
friends.set(1, "Mary");
```

## 二维数组

```java
// 初始化二维数组
Integer[][] num = new Integer[2][3];

// 另一种初始化
int[][] numbers = {
        {1,2,3},
        {4,5,6}
};

// 遍历 fori 二维数组
for (int i = 0; i <numbers.length ; i++) {
    for (int j = 0; j <numbers[i].length ; j++) {
        System.out.print(numbers[i][j] + " ");
    }
    System.out.println();
}

// 使用 foreach 遍历二维数组
for (int[] number : numbers) {
    for (int aNumber : number) {
        System.out.print(aNumber + " ");
    }
    System.out.println();
}
```

遍历结果输出：

```
1 2 3
4 5 6
```

---

# 可变长参数

## 使用 ... 声明可变长参数

```Java
public static double average(double... values) {
  double sum = 0;
  for(double v: values) sum+= v;
  return values.length == 0 ? 0 : sum / values.length;
}
```

**注意**：可变参数必须是方法的最后一个参数
