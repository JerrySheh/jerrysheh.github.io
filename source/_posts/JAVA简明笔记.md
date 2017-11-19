---
title: JAVA简明笔记（一） 基本编程结构
date: 2017-11-05 22:40:39
tags: JAVA
---


《Core Java for the Impatient》简明笔记。

<!-- more -->

---

# 第一章：基本编程结构

---
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
字符串

* `String name = String.join("-","hello","and","again"); `，输出 hello-and-again 。 第一个参数是连接符，第二到n个参数是需要连接的字符串
* `str.substring(7,12)`提取子字符串，如`Hello, World!` 第7（包括）到第12（不包括）位，即`World`这个单词。
* `str.split(" ")`，以空格为分隔符，将子字符串提取出来。最终结果为一个字符串数组。
* `str.equals("World")`，判断相等。
* 不要用 `==` 符号来判断字符串相等！！在Java虚拟机中，每个文字串只有一个实例，`"World" == "World"` 确实会返回真，但是如果前后比较的字符串是用分割提取等方法获取到的，它将会被存入一个新的对象当中，这时用==判断会出现假，因为不是同一个对象。
* `String middlename = null;` ，测试一个字符串对象是否为null，可以用`==`。
* null说明该变量没有引用任何对象。空字符串 `""`是长度为零的字符串， null不是字符串

* `if ("World".equals(location))` 是个好习惯， 把文字串放在前面
* `equalsIgnoreCase`方法，不考虑大小写比较字符串，`myStr.equalsIgnoreCase("world");`
* 数字转字符串，`str = Integer.toString(n,2)`，如果n是42，则把42转为二进制字符串 “101010”，第二个参数默认为10进制，范围在[2,36]
* 字符串转数字，`n = Integer.parseInt(str，2)`

---

格式化输出

* `System.out.printf("%8.2f", 1000.0 / 3.0)`, %8.2f指明输出的浮点数宽度为8，精度为小数点后两位。
* `System.out.printf("Hello, %s. Next year you will be %d.", name, age)`
* `String.format("Hello, %s. Next year you will be %d.", name, age);`

---

数组

* 声明一个有100个元素的字符串数组 `String[] names = new String[100];`
* new构造数组时的默认值：数字（0）， Boolean（flase）, 对象（空引用）
* 构造数组时，需要知道数组的长度，一旦构造出来长度便不能改变。在许多实际应用中很不方便。所以需要 java.util 包中的 ArrayList 类来根据需求随时增长或者缩减数组的长度。
* ArrayList是泛型类
* == 和 != 比较的是对象引用，而不是对象的内容。所以不要用 `if (numbers.get(i) == numbers.get(j))`，要用 equals

```java

ArrayList<Sstring> friends;
friend = new ArrayList<>();
friend.add("Peter");
friend.add("Paul");
friend.remove(1);

String first = friends.get(0);
friends.set(1, "Mary");
```

* 装箱和拆箱

```java

ArrayList<Integer> numbers = new ArrayList<>();
numbers.add(42);
int first = numbers.get(0);

```

* 增强 for 循环

```java

int sum = 0;
for ( int n : numbers){
  sum += 0;
}

```

---

* 可变长参数的声明：... 如 `public static double average(double... values)`
* 可变参数必须是方法的最后一个参数

```java

public static double average(double... values) {
  double sum = 0;
  for(double v: values) sum+= v;
  return values.length == 0 ? 0 : sum / values.length;
}

```
