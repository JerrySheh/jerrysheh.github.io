---
title: Effective Java（一）创建和销毁对象
comments: true
categories: Effective Java
tags: Java
abbrlink: 39fc1edf
date: 2019-09-09 23:35:58
---

《Effective Java》这本书算得上有口皆碑了，去年发现出了第三版，趁某东活动入手了一本英文版，粗略了过了一下，这本书给我最大的体会就是它教你如何成为一个真正的 Java 程序员，而不是 CRUD 程序员或 Spring 程序员，读这本书，能让你站在更高的角度和更深层次的视角去剖析 Java 的细节，让人豁然开朗。然而，上半年因为各种原因，瞎忙活了大半年，这本书一直没机会捡起来仔细看。好在最近工作不忙，想起来有这本书，决定一天看两个 Item 。

系列目录：

- [Effective Java（一）创建和销毁对象](../post/39fc1edf.html)
- [Effective Java（二）对象通用的方法](../post/f754c291.html)
- [Effective Java（三）类和接口](../post/20ef17da.html)
- [Effective Java（四）泛型](../post/53a4cf82.html)
- [Effective Java（五）枚举和注解](../post/acf36022.html)
- [Effective Java（六）Lambdas and Streams](../post/cc85a16e.html)
- [Effective Java（七）方法](../post/387fb533.html)
- [Effective Java（八）General Programming](../post/7d5810ff.html)
- [Effective Java（九）异常](../post/4e34dae4.html)
- Effective Java（十）并发

# Item 1 使用静态工厂方法替代构造器

我们平时编写一个类的构造方法，然后用 new 去获取一个对象。

```java
public class Student {
    String name;
    String age;

    Student(String name, String age){
        this.name = name;
        this.age = age;
    }
}

// client
Student jerry = new Student("jerry", 18);
```

有时候我们还可以把构造器私有化，禁止 new，取而代之的是用一个静态方法 newInstance 来获取对象：

```java
public class Student {
    String name;
    int age;

    private Student(String name, int age){
        this.name = name;
        this.age = age;
    }

    public static Student newInstance(String name, int age) {
        return new Student(name, age);
    }
}

// client
Student jerry = Student.newInstance("jerry", 18);
```

这么做的好处有五个：

1. 有名字，构造方法有多个时容易搞混，静态工厂方法就不会；
2. **静态工厂方法不要求每次都返回一个新对象，可以用来做单例（singleton）和不可实例化保证；**
3. 静态工厂方法可以返回一个对象的子类作为返回类型，而构造器不行，如 java.util.Collections；
4. 静态工厂方法返回对象的类可以根据输入参数的不同而不同；
5. 在编写包含该方法的类时，返回的对象的类不需要存在；

使用静态工厂方法，主要的不足是，没有 public 或 protected 构造器，因此也无法被子类化。但从另一个角度来说，这也是优点，因为这样做鼓励程序员 **多用组合，而不是继承**，这是好的习惯。第二个不足是程序员可能比较难找到他们，以下是静态工厂方法常用的名字：

- from
- of
- valueOf
- instance / getInstance
- create / newInstance
- getType
- newType
- type

例如 jdk 里，获取 Boolean 对象：

```java
public static Boolean valueOf(boolean b){
  return b ? Boolean.TRUE : Boolean.FALSE;
}
```

---

<!-- more -->


# Item 2 当构造器参数过多，考虑使用 Builder 模式

这里的 Builder 模式不是指设计模式。假设你要组装一台电脑，有品牌、价格、CPU、是否防水、屏幕尺寸等参数，有些是必选的，有些是可选的，如果用构造器，看起来会像是这样：

```java
public class Computer {
  private String brand;        // required
  private double price;        // required
  private int cpuGeneration;   // optional
  private boolean isWaterproof;// optional
  private int screenSize;      // optional

  Computer(String brand, double price){
    // something here
  }

  Computer(String brand, double price, int cpuGeneration){
    // something here
  }

  Computer(String brand, double price, int cpuGeneration, boolean isWaterproof){
    // something here
  }

  // ...

}
```

你会发现，你要写好多好多不同的构造方法，而且还容易搞混。

最好使用内部 builder 类，在调用方根据需要进行组合，如下：

```java
public class Computer {
    private String brand;        // required
    private double price;        // required
    private int cpuGeneration;   // optional
    private boolean isWaterproof;// optional
    private int screenSize;      // optional

    public static class Builder{
        private String brand;
        private double price;

        // Optional parameters - initialized to default values
        private int cpuGeneration = 3;
        private boolean isWaterproof = false;
        private int screenSize = 12;

        // constructor
        public Builder(String brand, double price) {
            this.brand = brand;
            this.price = price;
        }

        public Builder cpuGeneration(int value){
            cpuGeneration = value;
            return this;
        }

        public Builder isWaterproof(boolean value){
            isWaterproof = value;
            return this;
        }

        public Builder screenSize(int value){
            screenSize = value;
            return this;
        }

        public Computer build(){
            return new Computer(this);
        }
    }

    private Computer(Builder builder) {
        this.brand = builder.brand;
        this.price = builder.price;
        this.cpuGeneration = builder.cpuGeneration;
        this.isWaterproof = builder.isWaterproof;
        this.screenSize = builder.screenSize;
    }
}
```

调用方：

```java
public static void main(String[] args) {

    Computer computer = new Computer.Builder("MicroSoft", 6500.00)
            .cpuGeneration(7)
            .isWaterproof(false)
            .screenSize(24)
            .build();

    System.out.println(computer.toString());

}
```

---

# Item 3 使用私有构造器或枚举实现单例（Singleton）

> 注意: 不适用于多线程情况。

单例用于一个类只允许一个实例对象的情况，通常有两种方法实现单例：**公有域** 和 **公有静态方法**。两种方式都是通过 **私有构造器 + 公开静态成员** 来实现的。

第一种方法：公有域，客户端通过 `Elvis.INSTANCE` 来获取唯一对象。

```java
// Singleton with public final field
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
}
```

第二种方法：公有静态工厂方法。

```java
// Singleton with static factory
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTANCE; }
}
```

需要注意的是，有特权的客户端可以通过反射`AccessibleObject.setAccessible`的方式来调用私有构造方法。如果需要避免这个潜在的问题，可以修改构造函数，使其在请求创建第二个实例时抛出异常。

当需要序列化单例类对象时，仅仅用 `implements Serializable` 是不够的，因为每一次反序列化都会创建一个新的实例，解决办法是声明所有成员为 `transient`，然后用以下方法来返回实例。

```java
// readResolve method to preserve singleton property
private Object readResolve() {
     // Return the one true Elvis and let the garbage collector
     // take care of the Elvis impersonator.
    return INSTANCE;
}
```

最后还有一种用枚举实现单例的方式：

```java
// Enum singleton - the preferred approach
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() { ... }
}
```

用这种方式无需担心序列化问题和反射攻击，但是如果单例类需要继承除 enum 外的其他父类，就不能使用这种方法。

---

# Item 4 使用私有构造器实现不可实例化（Noninstantiable）

有些类（如工具类）只包含静态域和静态方法，为了避免被误用，可以将其构造器设置为私有，从而不可实例化。

```java
// Noninstantiable utility class
public class UtilityClass {
    // Suppress default constructor for noninstantiability
    private UtilityClass() {
        throw new AssertionError();
    }
    ... // Remainder omitted
}
```

为什么不用抽象类来实现不可实例化呢？因为抽象类可以被继承，其子类可以被实例化，并且会误导用户认为该类是为继承而设计的。

---

# Item 5 使用依赖注入，而不是硬编码所需资源

有些工具类或单例对象，依赖于一些底层资源。如单词拼写检查器，依赖于字典。

```java
// 静态工具类
// Inappropriate use of static utility - inflexible & untestable!
public class SpellChecker {
    // 依赖直接生成
    private static final Lexicon dictionary = ...;

    private SpellChecker() {} // Noninstantiable

    public static boolean isValid(String word) { ... }
    public static List<String> suggestions(String typo) { ... }
}

// 单例对象
// Inappropriate use of singleton - inflexible & untestable!
public class SpellChecker {
    // 依赖直接生成
    private final Lexicon dictionary = ...;

    private SpellChecker(...) {}
    public static INSTANCE = new SpellChecker(...);

    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}
```

然而，不同语言的单词拼写检查器，依赖于不同的字典，因此在 SpellChecker 直接依赖 dictionary，既不够灵活，又不便于测试（测试依赖其他字典时需要修改代码）。也就是说，**静态工具类和单例类都不适合直接引用底层资源**。

解决办法是 **依赖注入（Dependency injection）**。将 dictionary 依赖通过构造器，或静态工厂方法，或 item 2 中的 builder 传入 SpellChecker，从而实现解耦。

```java
// Dependency injection provides flexibility and testability
public class SpellChecker {
    private final Lexicon dictionary;

    // 依赖由构造器传入
    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}
```

一个更好的实践建议是，将资源工厂传递给构造器，再让工厂造资源。这也是设计模式中 **工厂方法模式（Factory Method pattern）** 的体现。Java 8 中引入的 `Supplier<T>` 接口非常适合代表工厂。

尽管依赖注入提高了灵活性和可测试性，但在大型项目中会让依赖变得十分混乱。使用依赖注入框架（如 Dagger、Guice  或 Spring）可以消除这些混乱。

---

# Item 6 避免创建不必要的对象

## 重用不可变对象

更多的时候，我们尽量要重用一个对象，而不是创建一个新的相同功能的对象，以节省资源。如果对象是不可变的（immutable, item 17），它就总是可以被重用。

一个反面例子：

```java
String s = new String("bikini");  // DON'T DO THIS!
```

用 new 构造一个 String 时，其参数本身就是一个String，这样白白创建了一个 bikini 对象。正确的做法应该像下面这样，使用单个 String 实例，而不是每次执行时创建一个新实例。此外，它可以保证对象运行在同一虚拟机上的任何其他代码重用。

```java
String s = "bikini"; // DO THIS !
```

## 使用静态工厂方法避免创建不需要的对象

使用静态工厂方法（static factory methods, item 1），可以避免创建不需要的对象。构造方法每次调用时都必须创建一个新对象，而工厂方法则不会。

## 缓存（预编译）实例

有些对象的创建很“昂贵”，例如检查一个字符串是否是一个有效的罗马数字：

```java
// Performance can be greatly improved!
static boolean isRomanNumeral(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
            + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```

我们发现，每次调用都要去匹配一次。`s.matches`在内部为正则表达式创建一个 Pattern 实例，并且只使用它一次，之后就可能被垃圾回收。然而，创建 Pattern 实例是昂贵的。解决方法是，将正则表达式显式编译为一个 Pattern 实例（不可变），缓存它，并在 isRomanNumeral 方法的每个调用中重复使用相同的实例：

```java
// Reusing expensive object for improved performance
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile(
            "^(?=.)M*(C[MD]|D?C{0,3})"
            + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```

## 自动装箱导致的创建不必要对象

注意自动装箱导致的创建不必要对象。如下面的代码，用 `Long` 会比 `long` 额外创建 2^31个不必要的 Long 实例。所以，优先使用基本类型而不是装箱的基本类型，也要注意无意识的自动装箱。

```java
// Hideously slow! Can you spot the object creation?
private static long sum() {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++)
        sum += i;
    return sum;
}
```

最后，避免创建不必要的对象这并不是说对象创建是昂贵的，应该避免创建对象。现代JVM可以很轻松应对廉价的对象，但是像数据库连接这样的重量级对象，就应该考虑重用的问题。

---

# Item 7 消除过期的对象引用

Java自带垃圾收集机制，但有时候得手动清除不需要的引用。

在一个栈的实现中，弹栈时我们返回 pop 之后的栈顶元素，但刚刚被弹出去的元素，其实已经不再被需要了，然而它的引用还在，垃圾收集器不会回收它。所以这个栈存在「内存泄漏」。

```java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    return elements[--size];
}
```

如下图所示，一个数组实现的栈，尽管 `s[5]` 已经被 pop 出去，但垃圾回收器不知道，它会认为 s[0] - s[7] 都是有用的。

![](../../../../images/Java/obsolete_ref.png)

解决办法很简单，对弹出去的元素置为 null 即可：

```java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // Eliminate obsolete reference
    return result;
}
```

> 请注意，清空对象引用应该是例外而不是规范。当一个类自己管理内存时，程序员才应该警惕内存泄漏问题。

---

# Item 8 避免使用 Finalizer 和 Cleaner 机制

 Finalizer 被设计用来当垃圾回收时关闭一些特殊的资源，可能有点像 C++ 的析构函数，但它并不稳定，因为我们无法确定什么时候垃圾回收。从 Java 9 开始，Finalizer 已被弃用，但仍被 Java 类库所使用。 Java 9 中 Cleaner 机制代替了 Finalizer 机制。但是 Cleaner 仍然是不可预测的。一般不要轻易尝试 Finalizer 和 Cleaner 。

 需要关闭一般资源时，请使用 `try-with-resource`。只有在使用 JNI(Java Native Interface) 调用non-Java程序（C或C++）时，才使用 finalize() 来回收这部分的内存。

---

# Item 9 使用 try-with-resource 代替 try-finally

像 `InputStream`，`OutputStream` 和 `java.sql.Connection` 这些资源时需要关闭的，通常我们用 `try-finally` 来关闭，但如果有多个资源需要关闭，情况会变得很糟糕：

```java
// try-finally is ugly when used with more than one resource!
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
```
内层的try块喝finally块都可能抛出异常，然而外层的 finally 块会吃掉内层的异常，导致在异常堆栈跟踪中没有第一个异常的记录，这会让调试变得非常复杂。

因此，在 Java 7 引入了 try-with-resources 语句，只要资源实现了 `AutoCloseable`接口就可以使用。当前， Java 类库和第三方类库中的许多类和接口现在都实现或继承了 `AutoCloseable`，所以无需担心。

```java
// try-with-resources on multiple resources - short and sweet
static void copy(String src, String dst) throws IOException {
    try (InputStream   in = new FileInputStream(src);
         OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
    }
}
```
