---
title: Java简明笔记（一） 基本编程结构
categories: JAVA
tags: Java
abbrlink: b3088ac5
date: 2017-11-05 22:40:39
---


《Core Java for the Impatient》简明笔记。


<!-- more -->

# 数据类型

九种基本数据类型

基本类型	|大小(字节)	|默认值	|封装类
---|---
byte|	1|	(byte)0|	Byte
short|	2|	(short)0	|Short
int|	4	|0	|Integer
long|	8	|0L|	Long
float|	4|	0.0f	|Float
double|	8|	0.0d	|Double
boolean|	-	|false	|Boolean
char|	2|	\u0000(null)	|Character
void	|-	|-	|Void

---

# 拆箱和装箱

## 什么是装箱？

一般我们要创建一个类的对象实例的时候，我们会这样：
```java
Class a = new Class(parameter);
```

当我们创建一个Integer对象时，却可以这样：
 ```java
 Integer i = 100; (注意：不是 int i = 100; )
 ```

实际上，执行上面那句代码的时候，系统为我们执行了：
```java
Integer i = Integer.valueOf(100);
```

这就是基本数据类型的`自动装箱`功能。

同理，拆箱就是把基本数据类型从Integer对象取出的过程。

## 基本数据类型与对象的差别

基本数据类型不是对象，也就是使用int、double、boolean等定义的变量、常量。
基本数据类型没有可调用的方法。

比如，

`int t = 1；`   ，  t.  后面没有方法。

`Integer t =1；` ， t.  后面就有很多方法可让你调用了。

---

# equals() 和 "=="

equals() 比较的是两个对象的值（内容）是否相同。

"==" 比较的是两个对象的引用（内存地址）是否相同，也用来比较两个基本数据类型的变量值是否相等。

前面说过，int 的自动装箱，是系统执行了 `Integer.valueOf(int i)`，看看Integer.java的源码：

```java
public static Integer valueOf(int i) {
    if(i >= -128 && i <= IntegerCache.high)　　// 没有设置的话，IngegerCache.high 默认是127
        return IntegerCache.cache[i + 128];
    else
        return new Integer(i);
}
```

对于–128到127（默认是127）之间的值，`Integer.valueOf(int i)` 返回的是缓存的Integer对象！！！


所以下面的现象也就不难理解了：

```java
//在-128~127 之外的数
Integer i1 =200;  
Integer i2 =200;          
System.out.println("i1==i2: "+(i1==i2));   // 输出 false


// 在-128~127 之内的数
Integer i3 =100;  
Integer i4 =100;  
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

循环ArrayList时，普通for循环比foreach循环花费的时间要少一点；循环LinkList时，普通for循环比foreach循环花费的时间要多很多。

当将循环次数提升到一百万次的时候，循环ArrayList，普通for循环还是比foreach要快一点；但是普通for循环在循环LinkList时，程序直接卡死。

结论：

需要循环数组结构的数据时，建议使用普通for循环，因为for循环采用下标访问，对于数组结构的数据来说，采用下标访问比较好。

需要循环链表结构的数据时，一定不要使用普通for循环，这种做法很糟糕，数据量大的时候有可能会导致系统崩溃。

原因：foreach使用的是迭代器


---

# 零碎知识

* 最小整数：Integer.MIN_VALUE
* 最大整数：Integer.MAX_VALUE
* long不够用，用BigInteger类
* long加L后缀（`400000000L`），Byte和short用类型转换`(byte)127`
* 1_000_000 = 1000000，编译器会自动删掉下划线（方便阅读）

---

* float有F后缀，没有F后缀默认为double
* `Double.POSITIVE_INFINITY`表示无穷大，`Double.NEGATIVE_INFINITY`表示负无穷大
* `Double.NaN`表示非数值
* 用 `if (Double.isNaN(x))`来检查 `x` 是否为 NaN，但不可以用`if (x == Double.NaN)`，因为NaN都是彼此不同的
* 浮点数不适合金融计算，用BigDecimal类
* `BigDecimal.ValueOf(n,e);`，其中n是一个整数数，e是小数位，如(588888,3)，就是 588.888

* Java不允许对象直接使用操作符，所以BigDecimal和BigInteer类需要用方法

```Java
BigDecimal next = bd.multiply(bd.add(BigDecimal.valueOf(l)));
```

---

* 最好每个变量名都有各自单独的声明
* 变量名建议用驼峰式命名，如 CountOfInvalidInputs
* 刚好正在首次需要变量的前一刻声明
* 常量用大写字母，如`DAYS_PRE_WEEK = 7;`
* 在其他类中使用Calendar类的常量，只需要前面加上类名，如`Calendar.DAYS_PRE_WEEK`

---

* 定义枚举类型`enum Weekday {MON, TUE, WED, THU, FRI, SAT, SUN};`，然后初始化它，`Weekday startDay = Weekday.MON;`
* 17/5 的结果是3， 而 17.0/5 的结果是3.4
* 整数除以零会导致异常，浮点数除以零会产生无限值或NaN
* 负数慎用 %
* Math类让算数运算更安全（p15）

---

* `Math.round`方法获得数字四舍五入的整数，`int n = (int) Math.round(x)`，若x=3.75，则n=4
* `( n!=0 && s+(100-s) / n ) < 50`，第二个条件除数等于零可能会报错，但第一个条件排除了这种可能性，所以第一个条件满足时，第二个条件不予评估，也就不会发生错误
* `time < 12 ? "am" : "pm"`， 若time<12，结果为am，否则为pm
* ` n & 0xF`的结果为n的最低四位
* 在32位的int类型中，1<<35的结果跟 1<<3 相同

---

# 格式化输出

* `System.out.printf("%8.2f", 1000.0 / 3.0)`, %8.2f指明输出的浮点数宽度为8，精度为小数点后两位。
* `System.out.printf("Hello, %s. Next year you will be %d.", name, age)`
* `String.format("Hello, %s. Next year you will be %d.", name, age);`

---

# 数组

* 声明一个有100个元素的字符串数组 `String[] names = new String[100];`
* new构造数组时的默认值：数字（0）， Boolean（flase）, 对象（空引用）
* 构造数组时，需要知道数组的长度，一旦构造出来长度便不能改变。在许多实际应用中很不方便。所以需要 Java.util 包中的 ArrayList 类来根据需求随时增长或者缩减数组的长度。
* ArrayList是泛型类
* == 和 != 比较的是对象引用，而不是对象的内容。所以不要用 `if (numbers.get(i) == numbers.get(j))`，要用 equals

```Java

ArrayList<String> friends;
friend = new ArrayList<>();
friend.add("Peter");
friend.add("Paul");
friend.remove(1);

String first = friends.get(0);
friends.set(1, "Mary");
```

---

# 可变长参数

* 可变长参数的声明：... 如 `public static double average(double... values)`
* 可变参数必须是方法的最后一个参数

```Java

public static double average(double... values) {
  double sum = 0;
  for(double v: values) sum+= v;
  return values.length == 0 ? 0 : sum / values.length;
}

```
