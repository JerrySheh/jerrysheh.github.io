---
title: Effective Java（二）对象通用的方法
comments: true
categories: Effective Java
tags: Java
abbrlink: f754c291
date: 2019-10-10 22:59:15
---

对象通用的方法，指的是 Object 类下的方法，即 toString、equals、hashCode 等等，合理地使用跟重写它们，可以避免很多坑。

# Item 10 重写 equals 时请遵守约定

重写 equals 很容易犯错，最好不要去重写，比如下面的情形：

1. 类的每个实例都是唯一的。显而易见，像 Thread 这样表示活动而不是值的类来说，每个实例都是不一样的。
2. 类不需要「逻辑相等（logical equality）」
3. 父类已经重写了 equals 方法，除非有必要，否则子类就不用再去重写了。例如，大多数 List 从 AbstractList 继承了 equals 实现，Map 从 AbstractMap 的 Map 继承了 equals 实现。

<!-- more -->

如果你不想这个类的 equals 方法被调用，可以给它抛异常

```java
@Override
public boolean equals(Object o) {
    throw new AssertionError(); // Method is never called
}
```

那什么时候需要重写 equals 方法呢？如果一个类需要逻辑相等，而不是引用相同的对象，那么需要重写 equals ，重写 equals 请遵守以下5个规则：

- **自反性**： 对于任何非空引用 x，`x.equals(x)` 必须返回 true。
- **对称性**： 对于任何非空引用 x 和 y，如果且仅当 `y.equals(x)` 返回 true 时 `x.equals(y)` 必须返回 true。
- **传递性**： 对于任何非空引用 x、y、z，如果` x.equals(y)` 返回 true，`y.equals(z)` 返回 true，则 x.equals(z) 必须返回 true。
- **一致性**： 对于任何非空引用 x 和 y，如果在 equals 比较中使用的信息没有修改，则 `x.equals(y)` 的多次调用必须始终返回 true 或始终返回 false。
- 对于任何非空引用 x，`x.equals(null)` 必须返回 false。

如何正确地使用 equals ：

1. **使用 == 运算符检查参数是否为该对象的引用**。如果是，返回 true。这只是一种性能优化，但是如果这种比较可能很昂贵的话，那就值得去做。
2. **使用 instanceof 运算符来检查参数是否具有正确的类型**。 如果不是，则返回 false。 通常，正确的类型就是 equals 方法所在的那个类，但有时是这个类实现的接口。 如果类实现了一个接口，且该接口允许实现接口的类进行比较，那么就使用接口。集合接口（如 Set，List，Map 和 Map.Entry）具有此特性。
3. **将参数转换为正确的类型**。因为转换操作在 instanceof 中已经处理过，所以它肯定会成功。
4. **对每个需要比对的属性，都去检查该参数属性是否与该对象对应的属性相匹配**。如果都匹配，返回 true，否则返回 false。如果步骤 2 中的类型是一个接口，那么必须通过接口方法访问参数的属性;如果类型是类，则可以直接访问属性，这取决于属性的访问权限。

JDK String 重写 equals 的例子就很值得学习：

```java
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

# Item 11 重写 equals ，必须重写 hashCode

重写 hashCode 的规范如下：

1. 在一个应用执行过程中，如果 equals 没有做任何修改，那么多次调用 hashCode 必须返回相同的值。
2. 如果 equals() 返回 true ，那么 hashcode 值必然相同，反之则不一定。

重写 equals 如果没有重写 hashCode，可能导致对象在集合（如 HashSet、HashMap）中出问题。因为两个逻辑相等 equals 的对象，若是不同的实例，会返回不同的 hashCode，如果一个实例插入到 HashMap 中，另一个作为判断相等的实例用来检索，put 方法把实例保存在了一个哈希桶（hash bucket）中，但 get 方法却是从不同的哈希桶中去查找，即使恰好两个实例放在同一个哈希桶中，get 方法几乎肯定也会返回 null。因为 HashMap 做了优化，缓存了与每一项（entry）相关的哈希码，如果哈希码不匹配，则不会检查对象是否相等了。

解决办法就只有重写 hashCode 方法，那么如何重写呢？去翻《Effective Java》原书吧，讲得很详细。也可以参考Guava 框架的 com.google.common.hash.Hashing [Guava] 方法，AutoValue 框架，或者 IDEA 的自动生成。

一个建议是不要试图从哈希码计算中排除重要的属性来提高性能，因为这样哈希质量会降低。

# Item 12 始终重写 toString 方法

toString 方法应该返回对象中包含的所有需要关注的信息。而 Object 类的 toString 却只返回 类名@十六进制数。因此我们最好重写它。例如当你把一个字符串放进 Map 里，输出时会 toString 方法会自动被调用， 输出 {Jenny=707-867-5309} 总比 {Jenny=PhoneNumber@163b91} 好吧？

附上阿里巴巴Java开发规范

【强制】关于 hashCode 和 equals 的处理，遵循如下规则：

1. 只要重写 equals ，就必须重写 hashCode 。
2. 因为 Set 存储的是不重复的对象，依据 hashCode 和 equals 进行判断，所以 Set 存储的对象必须重写这两个方法。
3. 如果自定义对象作为 Map 的键，那么必须重写 hashCode 和 equals 。
- 说明： String 重写了 hashCode 和 equals 方法，所以我们可以非常愉快地使用 String 对象作为 key 来使用。

# Item 13 谨慎地重写 clone 方法

Cloneable 接口是一个标记接口，只有实现该接口才可以调用 clone 方法。

clone的规范为： `x.clone() != x`，并且 `x.clone().getClass() == x.getClass()` ，通常情况下 `x.clone().equals(x)`。

但是请注意，clone() 是浅复制。也就是说只会复制对象本身，而对象引用的其他对象并不会被复制。考虑一个栈里面的元素，clone()出来的栈和原始栈引用的是相同的元素，因此改变克隆栈的某个元素，原始栈也会跟着改变，这是一个坑。要解决这个问题，要使用深复制，简而言之就是重写 clone() ，使对象中对其引用对象再使用clone()。

# Item 14 考虑实现 Comparable 接口

Comparable接口：

```java
public interface Comparable {
  int compareTo(T other);
}
```

如果你正在编写具有明显自然顺序（如字母顺序，数字顺序或时间顺序）的值类，那么就应该实现 Comparable 接口。如果一个对象按重要顺序要比较不同的属性，还可以递归调用：

```java
// Multiple-field `Comparable` with primitive fields
public int compareTo(PhoneNumber pn) {
    int result = Short.compare(areaCode, pn.areaCode);
    if (result == 0) {
        result = Short.compare(prefix, pn.prefix);
        if (result == 0)
            result = Short.compare(lineNum, pn.lineNum);
    }
    return result;
}
```

在 Java 8 中，还能用 比较器 Comparator 来定义比较行为：

```java
// Comparable with comparator construction methods
private static final Comparator<PhoneNumber> COMPARATOR =
        comparingInt((PhoneNumber pn) -> pn.areaCode)
          .thenComparingInt(pn -> pn.prefix)
          .thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
}
```

第三版中的建议是，比较 compareTo 方法的实现中的字段值时，请避免使用「<」和「>」运算符。相反，使用包装类中的静态 compare 方法或 Comparator 接口中的构建方法。
