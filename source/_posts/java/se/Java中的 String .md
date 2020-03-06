---
title: Java中的 String
comments: true
categories:
- Java
- Java SE
tags: Java
abbrlink: 689b9445
date: 2018-02-05 00:27:20
---

# String 的本质

在 Java8 中，分析 java.lang.String 类的源码，可以发现 String 内部维护的是一个 char 数组。同时可以发现，String类被 `final` 修饰，即不可变的。

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];

    //...
}
```

在 Java9 中，将 char 数组优化成了 byte 数组。

```java
private final byte value[];
```

<!-- more -->

## 为什么要将 char[] 改成 byte[] ？

char 占用16位，即两字节。每个 byte 占用8位，即1字节。如果我们要存储字符A，则为0x00 0x41，用 char 的话，前面的一个字节空间浪费了。

## 为什么要设计成不可变类 ？ 优缺点？

- **优点**：只读，所以 **多线程并发访问也不会有任何问题**。
- **缺点**：每个不同的状态都要一个对象来代表，可能会造成性能上的问题。（所以 Java 标准类库还提供了一个可变版本，即 StringBuffer）

具体到 String 类中，原因如下：

1. **字符串常量池的需要**
 维护一个字符串池，可以节省堆内存空间。不同的字符串变量都指向池中的同一个字符串。即字符串常量池数据共享。
2. **线程安全考虑**
3. **类加载器要用到字符串**，不可变性提供了安全性，以便正确的类被加载。
4. **支持hash映射和缓存**
 因为字符串是不可变的，所以在它创建的时候hashcode就被缓存了，不需要重新计算。这就使得字符串很适合作为Map中的键，字符串的处理速度要快过其它的键对象。这就是HashMap中的键往往都使用字符串。

---

# String str = new String("abc")创建了几个对象？

答案是：最多创建2个，最少创建1个。

在Java虚拟机（JVM）中存在着一个字符串池，其中保存着很多String对象，并且可以被共享使用，因此它提高了效率。**由于String类是final的，它的值一经创建就不可改变**，因此我们不用担心String对象共享而带来程序的混乱。

当我们执行：

```java
String str = "abc";
```

String类会先去字符串池寻找`abc`这个对象，如果`abc`存在，则把它的引用给str，如果`"abc"`不存在，则先创建`abc`对象。

看String类源码的构造方法中，其中一个是：

```java
// 源码第 151 行
/*
 * 初始化一个新创建的 String 对象，使其表示一个与参数相同的字符序列；
 * 换句话说，新创建的字符串是该参数字符串的副本。
 * Initializes a newly created {@code String} object so that it represents
 * the same sequence of characters as the argument; in other words, the
 * newly created string is a copy of the argument string. Unless an
 * explicit copy of {@code original} is needed, use of this constructor is
 * unnecessary since Strings are immutable.
 *
 * @param  original
 *         A {@code String}
 */
public String(String original) {
    this.value = original.value;
    this.hash = original.hash;
}
```

可以发现，被调用的构造器方法接受的参数也是一个String对象。也就是说，当我们执行：

```java
String str=new String("abc");
```

String类会先去字符串池寻找`abc`，发现`abc`不存在，于是创建`abc`这个对象，然后把`abc`作为构造方法的参数，传给String构造器`new String("abc")`相当于新创建了参数字符串的副本，于是又创建了一个对象。

只是，第一个`abc`对象存在于字符串池当中，第二个跟其他对象一样存在于内存的堆当中。

---

# String 的 intern 方法

`String.intern()`是一个 native 方法。如果字符串常量池里面已经包含一个等于此 String 对象的字符串，则返回池中的这个字符串String对象，否则，先将该String对象包含的字符串添加进常量池，然后返回此String对象的引用。

在 jdk 1.6 的实现里，intern会把首次出现的该实例内容复制进永久代里面的常量池，而在 jdk 1.7 以上的实现中，intern 方法只是在常量池记录首次出现该实例的引用，并不会把它复制一份。

```java
// base on jdk 1.8
public static void main(String[] args) {

    String str1 = new StringBuilder("oj").append("bk").toString();
    // "ojbk"是首次出现，记录的是 str1 的引用，因此返回 true
    System.out.println(str1.intern() == str1);


    String str2 = new StringBuilder("ja").append("va").toString();
    // "java"肯定在其他地方出现过了，记录的是别的引用，不是str2，因此返回 false
    System.out.println(str2.intern() == str2);

}
```

---

# String 的 “+” 号是怎么连接字符串的 ？

当我们在程序中输入：

```java
public static void main(String[] args) {
  String hello = "hello";
  String world = "world";
  String hw = hello + world;
}
```

编译之后，如果我们把 .class 文件反编译回 .java 文件，可以看到代码变成了：

```java
public static void main(String[] args) {
  String hello = "hello";
  String world = "world";
  String hw = (new StringBuilder()).append(hello).append(world).toString();
}
```

编译器自动引入了一个 java.lang.StringBuilder 类。虽然我们在源代码中并没有使用 StringBuilder 类，但是编译器却自作主张地使用了它，因为它更高效。这就是所谓的编译器优化。

如果字符串操作比较简单，那就可以信赖编译器，它会为你合理地构造最终的字符串结果。**但如果你还使用循环，多次地改变字符串的内容，那就更适合StringBuilder对象。**

参考：[Java学习笔记（3）—— String类详解](https://www.jianshu.com/p/e494552f2cf0)

---

# String 的比较

## == 和 equals 两种比较

```java
String s1 = "AAA";
String s2 = new String("AAA");

System.out.println(s1 == s2);   // 输出 false
System.out.println(s1.equals(s2)); // 输出 true
```

`==`比较的是引用的内存地址，而`equals`方法比较的是字符串的内容。

## 探究 String 类 equals 方法源码

```java
/**
 * Compares this string to the specified object.  The result is {@code
 * true} if and only if the argument is not {@code null} and is a {@code
 * String} object that represents the same sequence of characters as this
 * object.
 *
 * @param  anObject
 *         The object to compare this {@code String} against
 *
 * @return  {@code true} if the given object represents a {@code String}
 *          equivalent to this string, {@code false} otherwise
 *
 * @see  #compareTo(String)
 * @see  #equalsIgnoreCase(String)
 */
public boolean equals(Object anObject) {
    // 如果是同一个对象，返回 true
    if (this == anObject) {
        return true;
    }

    // 如果 anObject 可以向下转型为 String
    if (anObject instanceof String) {
        // 转型为 String 类型
        String anotherString = (String)anObject;

        // 原字符串长度
        int n = value.length;

        // 如果原字符串长度和要比较的字符串长度一致
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;

            // 逐个字符比较
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

从源码中可以知道，类型不同是返回 false 的，因此考虑下面的例子：

```java
String s = "hello";
String t = "hello";
char c[] = {'h', 'e', 'l', 'l', 'o'};
```

显然，`s==t`和 `s.equals(t)` 返回 true ，因为 s 和 t 都指向了同一个 String 常量池里面的常量，类型相同且值相同。但是，`t.equals(c)` 返回 false ， 因为类型不同。

引申，在 Java 9 中， String 的实现已经从 char[] 变成 byte[] 了。因此就更应该是 false 了。

## 字符串比较的几点建议

**建议一**：文字串和String对象比较的时候，好的习惯是把文字串放在前面，这样可以避免某些空指针异常。

```java
if ("World".equals(location))
```

**建议二**：不要用 `==` 符号来判断字符串相等！！在Java虚拟机中，每个文字串只有一个实例，`"World" == "World"` 确实会返回真，但是如果前后比较的字符串是用分割提取等方法获取到的，它将会被存入一个新的对象当中，这时用`==`判断会出现假，因为不是同一个对象。

**建议三**：测试一个字符串对象是否为null，可以用`==`。例如：

```java
String middlename == null;
```

null不是字符串，null说明该变量没有引用任何对象。而空字符串 `""`是长度为零的字符串。

**建议四**：如果想忽略大小写比较字符串，使用`equalsIgnoreCase`方法：

```java
myStr.equalsIgnoreCase("world");
```

---

# String 的用法

## 使用 join 连接字符串

join并不是用来取代“+”连接符的，更多是用于分隔符拼接。（参考：[stackoverflow](https://stackoverflow.com/questions/31817290/string-join-vs-other-string-concatenation-operations)）

```java
String name = String.join("-","hello","and","again");
```

输出 hello-and-again 。 第一个参数是连接符，第二到n个参数是需要连接的字符串

## 使用 substring 提取子串

```java
String str = "Hello, World!";
String str2 = str.substring(7,12);
```

提取出第 7（包括）到第12（不包括）位，即`World`这个单词。注意是从 0 开始的。

## 使用 split 分割字符串


```java
String[] subs = str.split(" ");
```

以空格为分隔符，将子字符串提取出来。**split的最终结果为一个字符串数组。**

## 使用 format 格式化输出

```java
String fs;
fs = String.format("浮点型变量为%f, 整型变量为%d, 字符串变量为%s",
                   floatVar, intVar, stringVar);

String hello;
hello = String.format("Hello, %s. Next year you will be %d.",
                      name, age);
```

## 使用 toString 将数字转为字符串

```java
str = Integer.toString(n,2);
```

`toString`接受2个参数，第一个参数是数字n，第二个参数是进制（默认为10进制，范围在[2,36]）。在这个例子中，如果n是42，则把42转为二进制字符串 “101010”。

## 使用 parseInt 将字符串转化为数字

```java
n = Integer.parseInt(str，2)
```

这实际上是`Integer`的方法而不是`String`的方法。这个例子将字符串 str 转化成二进制的 Integer 。

## 使用 replace 和 replaceAll 取代

```java
public String replace(char oldChar, char newChar)
public String replaceAll(String regex, String replacement)
```

注意，replaceAll 方法的第一个参数是一个正则表达式


---

# String 和 Char[] 之间的转换

String的底层就是 Char数组（JDK1.9之后是 Byte 数组），转换方式如下：

```java
//将 String 转换为 char[]
char[] cs = str.toCharArray();

//将 char[] 转换为 String
String str = String.valueOf(cs); // 方法一
String str2 = new String(cs);   // 方法二
```

---

# StringBuffer 和 StringBuilder

当需要对字符串进行修改，可以使用 StringBuffer 和 StringBuilder 类。

和 String 类不同的是，StringBuffer 和 StringBuilder 类的对象能够被多次的修改，并且不产生新的未使用对象。

StringBuilder 类在 Java 5 中被提出，它和 StringBuffer 之间的最大不同在于 StringBuilder 的方法不是线程安全的（不能同步访问）。但是速度快。

由于 StringBuilder 相较于 StringBuffer 有速度优势，所以多数情况下建议使用 StringBuilder 类。然而在应用程序要求线程安全的情况下，则必须使用 StringBuffer 类。 （[摘自菜鸟教程](http://www.runoob.com/java/java-stringbuffer.html)）

总结起来就是： **StringBuilder 比 StringBuffer 快，但涉及线程安全必须用StringBuffer。它们两者与 String 的不同点在于对象能被多次修改。**

## StringBuffer 的用法

StringBuffer有跟String类似的方法：

### 使用 append 追加字符串

```java
StringBuffer s = new StringBuffer("hello world，");
s.append("I am ");
s.append("Jerry.");
System.out.println(s);
```

输出：` hello world，I am Jerry.`

#### 扩展： String 的 “+” 和 StringBuffer的 append

问: 有没有哪种情况用 + 做字符串连接比调用 StringBuffer / StringBuilder 对象的 append 方法性能更好？

答：如果连接后得到的字符串在 **静态存储区中是早已存在的**，那么用+做字符串连接是优于 StringBuffer / StringBuilder 的 append 方法的。

### 使用 reverse 进行反转


```java
StringBuffer s = new StringBuffer("hello");
s.reverse;
System.out.println(s);
```

输出：`olleh`

### 使用 delete 删除字符串中间的字符

```java
public delete(int start, int end)

StringBuffer s = new StringBuffer("hello");
s.delete(1,3);
System.out.println(s);
```

输出：`ho`

### 使用 insert 插入

```java
StringBuffer s = new StringBuffer("hello");
str.insert(1,"ang");
System.out.println(s);
```

输出：`“hangello”`

### 使用 replace 取代

不举例了，给出原型:

```java
replace(int start, int end, String str)
```

---

参考：

- [互联网面试笔记
](https://www.bookstack.cn/read/note-of-interview/java-base.md)
