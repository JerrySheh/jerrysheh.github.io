---
title: Java了然于心
comments: true
categories: JAVA
tags: JAVA
abbrlink: 650d39e8
date: 2018-09-12 21:13:51
---


# 1. 八种基本数据类型

- boolean
- 1字节：byte
- 2字节：char、short
- 4字节：int、float
- 8字节：long、double

# 2. 自动转换和强制转换

自动转换是系统悄然进行的，从低位转换到高位。例如 int + short ，结果为 int， int + double，结果为 double 。

强制转换需要显式声明，Java可以强制向下转型。比如double转int，父类转子类。

# 3. i++ 和 ++i

- i++：先得到i，再把 i+1 赋给 i
- ++i: 先得到i+1，再把 i+1 赋给 i

# 4. 拆箱和装箱

Integer i = 100 ，自动装箱
int a = 5; Interget b = 50; a == b 自动装箱

# 5. 任意数据类型都可转 Object

基本数据类型也可以，会自动装箱

# 6. equals 和 ==

- Object类的 == 和 equals 没区别
- String类的 == 比较的是内存地址，equals比较的是值

# 7. Integer的缓存

-127 ~ 128

# 8. JAVA创建对象的过程

简要回答：
1. 在堆中创建对象
2. 在栈中创建引用
3. 引用指向对象

从虚拟机角度回答:
1. 判断对象所属的类是否被虚拟机加载和链接
2. 判断是否有父类，父类是否被初始化
3. 初始化

# 9. 类块加载顺序

1. 父类静态块
2. 子类静态块
3. 父类构造块
4. 父类构造方法
5. 子类构造块
6. 子类构造方法

# 10. 面向对象的三个特征

1. 封装
2. 继承
3. 多态

多态前提：有继承关系、子类重写父类方法、父类引用指向子类对象
多态好处：屏蔽子类间的差异，写出通用的代码（SQLDao dao = new MySQLDao()）
多态弊端：不能用子类特有的方法，解决办法：instanceof操作符，向下转型

# 11. public、protected、private

- public：所有类可访问
- protected：子类、同包
- 无修饰符：同包
- private：私有，内部类可访问

# 12. 局部遍历和类变量的初始化

- 局部变量：初始化必须赋值
- 类变量：有默认值， int 0，char 空字符'\u0000'，String null

# 13. 成员内部类可以访问外部类的 private 属性，为什么？

成员内部类在编译时会生成单独的 .class 文件

1. Outter.class
2. Outter$Inner.class

编译器默认会为成员内部类添加了一个指向外部类对象的引用，也就是说，内部类对象被构造的时候，自动传入了一个外部类对象的引用，因此可以在成员内部类中随意访问外部类的成员。

# 14. 为什么局部内部类只能访问 final ？

方法 A 中定义了局部内部类 B，当 方法A 执行完毕，已经结束作用域，如果内部类 B 的方法（可能由另一个线程执行）还没结束，但由于 A 结束作用域，方法 A 的变量 a 会不可访问。为了解决这一问题， Java 采用了 **复制** 的手段，即把方法 A 的变量 a 复制一份到内部类 B 的常量池。

但是复制过后会产生不一致的问题，也就是内部类的方法修改了 a ， 但是外部类的 a 没有改变。

因此，Java 规定，只能访问 fianl ，以避免上述问题。

# 15. 接口中的方法有哪些修饰符？

- public
- statict（必须提供实现）
- default（必须提供实现）
- abstract

default有什么用？ 接口演化

# 16. JAVA 标准类库常用接口

- comparable接口，实现了这个接口的类，其对象能够进行比较
- comparator，比较器，用于提供多样的比较方法，自定义多种比较
- runnable，用于执行线程
- serializable，标记接口，用于序列化

# 17. 接口和抽象类的区别

接口是 like-a 关系， 抽象类是 is-a 关系。抽象类一般用作基类，让具体类去实现。接口一般用作某个类具有哪些功能。

抽象类可以有成员变量和实现方法，接口只能有常量，static 和 default 方法有实现。

抽象类可以有构造器，接口没有。

# 18. Overload和Override的区别

- Overload是重载，一个类中可以多个名字一样，但参数类型或个数不一样的方法
- Override是重写，子类重写父类的方法

# 19. Object类有哪些方法？

1. clone，用于对象复制
2. toString
3. equals，比较
4. hashcode，哈希
5. wait，让一个线程放弃锁，进入等待状态
6. notify/notifyAll，唤醒线程，让线程进入就绪状态
7. getClass，获取类对象
8. finalize，垃圾回收相关

# 20. 什么时候重写 equals，什么时候重写 hashcode ？

当一个类的多个对象之间，只需要某些字段相等而不必是同一个对象我们就认为他相等的时候，需要重写 equals 方法。重写了 equals 方法最好也重写 hashcode 方法，因为 hashcode 方法常用在 HashSet 或 HashMap 计算 key。 equals 的两个对象，却有不同的 hashcode，会被存入 Set 或 Map 中的不同位置，这和 HashSet 的设计初衷相悖。

# 21. HashMap如何存储键值对

HashMap底层是使用 Entry 对象数组存储的，Entry是一个单项的链表。当这个链表长度超过 8 时，转换成红黑树。

## put() 过程

1. 先使用 hash() 函数获取该对象的 hash 值，高16位和低16位异或后跟 Entry对象数组大小-1 进行与操作，得到应该存入数组的下标。
2. 假如该位置为空，就将value值插入，如果该下标不为空，则要遍历该下标上面的对象，使用equals方法进行判断，如果遇到equals()方法返回真则进行替换，否则将其插入。

## get() 过程

1. 根据 key 对象的 hash 值找到 Entry 对象数组的对应下标。
2. 判断Entry的 key 和 给定的 key 是否相同（equals或==），以及 hash 是否也相同，如果不是，访问链表下一个 Entry ，如果是，返回 Entry 的 value，如果遍历完了也没有，返回 null

# 22. HashMap 的 key 有什么要求？

1. 最好不要用可变对象。假设一定要是可变对象，也要保证 hashcode 方法的结果不会变。因为 HashMap 的 get 方法是会去判断 hashcode 值，如果 hash 值变了，有可能就取不到。
2. 使用不可变对象是明智的。

# 23. ArrayList 和 LinkList 的区别 ？

ArrayList 底层数组实现，遍历快，中间插入慢。

LinkList 底层链表实现，中间插入快，遍历慢。

# 24. Java.util.concurrent包（Java并发工具包）

待补充

# 25.
