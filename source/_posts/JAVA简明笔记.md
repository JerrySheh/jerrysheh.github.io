---
title: JAVA简明笔记
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

* `Math.round`方法获得数字四舍五入的整数


未完待续。。。
