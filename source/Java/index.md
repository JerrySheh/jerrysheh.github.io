---
title: Java了然于心
comments: false
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

强制转换需要显式声明，Java可以强制向下转型。比如double转int（会损失精度），子类转父类（丢失部分属性），父类转子类（前提是声明时必须用父类引用指向子类对象，即多态声明）。

# 3. i++ 和 ++i

- i++：先得到i，再把 i+1 赋给 i
- ++i: 先得到i+1，再把 i+1 赋给 i

# 4. 拆箱和装箱

什么时候自动装箱?

字面量直接赋值给 Integer，比如

Integer i = 100

什么时候自动拆箱？

用 == 比较Integer和int时，比如

int a = 5; Integer b = 50; a == b

# 5. 任意数据类型都可转 Object

基本数据类型也可以，会自动装箱

# 6. equals 和 ==

- Object类的 == 和 equals 没区别， equals 实际上判断的就是 ==
- String类的 == 比较的是内存地址，equals比较的是值

String 的 eauals 过程？

1. 先比较 == ，如果相同返回 true， 不同继续判断
2. 是否能转型为 String，不能直接返回 false
2. 比较长度，不同返回 false，相同继续判断
3. 逐个字符比较

# 7. Integer的缓存

《阿里巴巴Java开发手册》中提到，对于 `Integer var =?` 在 -128 至 127 之间的赋值， Integer 对象是在IntegerCache.cache 产生，会复用已有对象，这个区间内的 Integer 值可以直接使用 == 进行判断，但是这个区间之外的所有数据，都会在堆上产生，并不会复用已有对象，这是一个大坑，因此<font color="red">推荐使用 equals 方法进行判断</font>。

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

# 12. 局部变量和类变量的初始化

- 局部变量：初始化必须赋值，否则编译不通过
- 类变量：有默认值， int 0，char 空字符'\u0000'，String null

# 13. 成员内部类可以访问外部类的 private 属性，为什么？

成员内部类在编译时会生成单独的 .class 文件

1. Outter.class
2. Outter$Inner.class

编译器默认会为成员内部类添加了一个指向外部类对象的引用，也就是说，内部类对象被构造的时候，自动传入了一个外部类对象的引用，因此可以在成员内部类中随意访问外部类的成员。

# 14. 为什么局部内部类只能访问 final ？

避免外部作用域提前结束。

方法 A 中定义了局部内部类 B，当 方法A 执行完毕，已经结束作用域，如果内部类 B 的方法（可能由另一个线程执行）还没结束，但由于 A 结束作用域，方法 A 的变量 a 会不可访问。为了解决这一问题， Java 采用了 **复制** 的手段，即把方法 A 的变量 a 复制一份到内部类 B 的常量池。

但是复制过后会产生不一致的问题，也就是内部类的方法修改了 a ， 但是外部类的 a 没有改变。

因此，Java 规定，只能访问 fianl ，以避免上述问题。

# 15. 接口中的方法有哪些修饰符？

- public
- static（必须提供实现）
- default（必须提供实现）
- abstract

default有什么用？ 接口演化

# 16. JAVA 标准类库常用接口

- comparable接口，实现了这个接口的类，其对象能够进行比较
- comparator，比较器，用于提供多样的比较方法，自定义多种比较
- runnable，用于执行线程
- serializable，标记接口，用于序列化

# 17. 接口和抽象类的区别

抽象类是 is-a 关系，接口是 like-a 关系。抽象类一般用作基类，让具体类去实现。接口一般用作某个类具有哪些功能。

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

当多个对象之间，只需要某些字段相等而不必是同一个对象我们就认为他相等的时候，需要重写 equals 方法。重写了 equals 方法最好也重写 hashcode 方法，因为 hashcode 方法常用在 HashSet 或 HashMap 计算 key。 equals 的两个对象，却有不同的 hashcode，会被存入 Set 或 Map 中的不同位置，这和 HashSet 的设计初衷相悖。HashMap取的时候也可能因 HashCode 的改变而取不到。

# 21. HashMap如何存储键值对

HashMap底层是使用 Node 对象数组存储的，Node 是一个单项的链表。当这个链表长度超过 8 时，转换成红黑树 TreeNode 。

## put() 过程

1. 确定要存入的桶。先使用 hash() 函数获取该对象的 hash 值，高16位和低16位异或后跟 Entry对象数组大小-1 进行与操作，得到应该存入数组的下标。
2. 链表插入。假如该位置为空，就将value值插入，如果该下标不为空，则要遍历该下标上面的对象，使用equals方法进行判断，如果遇到equals()方法返回真则进行替换，否则将其插入。

## get() 过程

1. 根据 key 对象的 hash 值找到 Entry 对象数组的对应下标。
2. 判断Entry的 key 和 给定的 key 是否相同（equals或==），以及 hash 是否也相同，如果不是，访问链表下一个 Entry ，如果是，返回 Entry 的 value，如果遍历完了也没有，返回 null

# 22. HashMap 的 key 有什么要求？

1. 最好不要用可变对象。如果一定要是可变对象，也要保证 hashcode 方法的结果不会变。因为 HashMap 的 get 方法是会去判断 hashcode 值，如果 hash 值变了，有可能就取不到。
2. 使用不可变对象是明智的。

# 23. ArrayList 和 LinkList 的区别 ？

ArrayList 底层数组实现，遍历快，中间插入慢。

LinkList 底层链表实现，中间插入快，遍历慢。

# 24. Java.util.concurrent包（Java并发工具包）

concurrent包包含了一些帮助我们编写并发程序的有用的类（比如BlockingQueue阻塞队列，SynchronousQueue同步队列）以及线程安全的原子类（如AtomicInteger）。

- 学习参考：http://tutorials.jenkov.com/java-util-concurrent/index.html
- 中文：https://blog.csdn.net/axi295309066/article/details/65665090

# 25. fail-fast 和 fail-safe

java.util 包下的集合类都是快速失败（fail—fast）的，不能在多线程下发生并发修改（迭代过程中被修改）。java.util.concurrent包下的容器都是安全失败（fail—safe），可以在多线程下并发使用，并发修改。

用迭代器遍历一个java.util集合对象时，如果遍历过程中对集合对象的内容进行了修改（增加、删除、修改），则会抛出Concurrent Modification Exception。java.util.concurrent包下则不会。

# 26. Collection 和 Collections 的区别

Collection是集合类的上级接口，包含了 list、Set、Map 等子接口。Collections是集合工具类，提供了一些常用的集合操作。例如对各种集合的搜索、排序、线程安全化等操作。

# 27. 为什么 String 要设计成 final ？

1. 维护一个常量池，节省堆空间。
2. final类由于不可修改性，多线程并发访问也不会有任何问题
3. 支持hash映射和缓存

# 28. String s = new String("abc") 创建几个对象？

2个。第一在常量池中生成 abc 对象，第二在堆中生成 abc 对象。

# 29. String 的 + 号 如何连接字符串 ？

编译器优化，StringBuilder().append()

# 30. StringBuffer 和 StringBuilder

 StringBuilder 比 StringBuffer 快，但涉及线程安全必须用StringBuffer。它们两者与 String 的不同点在于对象能被多次修改。

# 31. Java中的异常

分为 Error 和 Exception

Error代表严重错误，比如 OutOfMemory 和 StackOverFlow

Exception 分为 checkException 和 uncheckException （也叫runtimeException）

checkException 是我们需要在程序中捕获处理或抛出的异常，比如IO、Network Exception

uncheckException 是可以通过优化程序逻辑来避免的，不应该捕获，常见的有 nullpointerException 和 ClassNotFoundException 和 ArrayIndexOutOfBoundsException

# 32. try里面有 return， finally 还执行吗？

执行。先保存 return 的内容，finally 里执行完之后再 return

但是 finally 里有 return， 会提前返回

# 33. 输入输出流？

InputStream 和 OutputStream 处理字节流（一个字节8位bit）， reader 或 writer 处理字符流（Unicode 字符）

BufferedWriter 和 BufferedReader 缓存流。

# 34. 线程的三种创建方式？

1. 继承Thread类
2. 实现Runnable方法（推荐）
3. 实现Callable方法

# 35. 线程Thread类的 join 方法是干什么用的？

让线程串行执行。

# 36. Java中线程同步有几种方式？

1. synchronized 关键字，解决竞争条件问题（多个线程同时访问一段内存区域）
2. Volatile 关键字，解决可见性问题（线程栈、CPU缓存），但不能保证原子性问题
3. java.util.concurrent包下的Atomic原子类，无锁保证原子性。synchronized开销太大，性能会下降。多线程 i++ 问题Atomic已足够。
4. ReentrantLock，是一个可重入、互斥、实现了Lock接口的锁。
5. ThreadLocal类，线程局部变量。
6. java.util.concurrent包下的阻塞队列

# 37. JVM的组成？

1. 类加载器
2. 内存区域
3. 执行引擎
4. 本地方法调用

# 38. 类加载器如何加载一个类？

1. 加载（读取.class二进制字节流，转换成方法区动态数据结构，堆中创建对象）
2. 链接（校验、解析（静态变量赋默认值）、准备（符号引用->直接引用））
3. 初始化（静态变量赋初值）

# 39. 什么时候初始化 ?

1. 遇到 new、getstatic、putstatic、invokestatic 字节码关键字
2. 反射
3. 父类未初始化先初始化父类
4. 虚拟机启动时，主类

# 40. 全盘负责双亲委派机制

1. 当一个 classloader 加载一个类时，其依赖和引用也由这个类加载器加载
2. 类加载器先委派父加载器加载，父加载器找不到目标类才由子加载器加载

好处？安全，保证所有基础的类都是由 RootClassLoader 来加载的。

有没有例外？有。线程上下文类加载器（Thread Context ClassLoader），父类加载器可以请求子类加载器去完成类加载的动作。

# 41. 类加载器如何判断两个类相同？

1. 类全限定名相同
2. 加载该类的类加载器相同

# 42. JVM的内存模型

1. 程序计数器（线程隔离）
2. 本地方法栈（线程隔离）
3. 方法栈（线程隔离）
4. 堆（线程共享）
5. 方法区（线程共享）

# 43. 什么是动态链接？

.class文件中有很多符号引用，一部分在类加载的时候转化为直接引用（称为静态链接），另一部分在每一次运行期间转化为直接引用，这部分被称为动态链接。

# 44. 垃圾回收算法

针对新生代，很多被清理，用标记-清除法，但效率低。用复制算法较好（Eden、Survior1、Survior2）

先只使用 Eden、Survior1，垃圾回收的时候，把幸存的复制到 Survior2，然后清空 Eden和Survior1，之后只使用 Eden、Survior2 。

针对老年代，只有很少被清理，标记-整理算法。将所有废弃的对象做上标记，然后将所有未被标记的对象移到一边，最后清空另一边区域即可。

# 45.Java IO 和 NIO

IO面向字节流和字符流，而NIO面向 channels 和 buffers 。

有许多连接，但是每个连接都只发送少量数据，选择NIO。（如网络聊天室、P2P网络）

如果只有少量的连接，但是每个连接同时发送很多数据，传统的IO会更适合。

# 46. 什么是CAS ？

CAS（Compare and swap）用于实现非阻塞并发算法。一个线程在修改一个变量时，先将当前值跟预期值进行比较，如果一致，则进行修改，如果不一致，说明这个变量被其他线程改了，就不进行修改。

在 java.util.concurrent.atomic 里面，像 AtomicBoolean 这些原子类就有 compareAndSet 方法。

参考：http://tutorials.jenkov.com/java-concurrency/compare-and-swap.html

# 47. 多线程应该注意哪些问题？如何避免？

三个问题：

1. 安全性问题（竞争条件、可见性）
2. 活跃性问题（死锁）
3. 性能问题（为了解决上述问题而导致的性能下降）

解决安全性问题，可以用 synchronized 关键字， 或者 ReentrantLock 同步锁，Volatile用于解决可见性问题，Threadlocal类等。

解决死锁问题，可以让线程一开始就持有所有需要的资源，但这样会造成资源浪费，变成一个性能问题。第二种方式是，当需要新的资源而不能满足时，必须先释放自己持有的锁。





# 48. 写出单例模式

单例模式是指一个类只能有一个对象实例。好的单例模式应该满足两点要求：**延时加载** 和 **线程安全**。

静态内部类写法：
1. 用静态内部类构造实例
2. 构造函数私有

```java
public class Singleton{

  private static class Holder{
    private static Singleton singleton = new Singleton();
  }

  private Singleton();

  public static Singleton get(){
    return Holder.singleton;
  }

}
```

# 49. 基本数据类型及其包装类有什么区别？

1. Java是一门纯粹的OO语言，但基本数据类型不是对象，为了让他们有对象的特征，Java设计了对应的包装类。包装类是对象，就要有对象的特征，有可调用的方法，而基本类型没有。
2. 包装类可放入如HashMap、HashSet等集合中，基本数据类型不可以。但是存入时会被Java自动装箱。
3. 基本数据类型初始化值为0（char为\u0000）,if 判断时要用 `if(i == 0)`，而包装类要用 `if(i==null)`
