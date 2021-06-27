---
title: Java中的浅拷贝和深拷贝
comments: false
categories:
  - Java
  - Java SE
tags: Java
abbrlink: 499617c5
date: 2021-06-26 13:33:48
---

## 引言

最近在复审代码时，发现组内一个同学写了这么一段代码：

```java
public void process(ResultContext rc){

    Map m1 = rc.getResult();
    Map m2 = rc.getResult();

    // m1 做一些get set处理
    // ..

    // m2 做一些get set处理
    // ..

    dao.insert(m1);
    dao.insert(m2);
}
```

这段代码，本意是想从同一个 `ResultContext` 获取到原始数据，然后对这份数据做不同的加工，再分别保存。但这段代码实际上是有问题的。**因为 `m1` 和 `m2` 虽然是不同的引用，但实际上都指向了同一个对象，在修改 `m1` 的同时， `m2` 实际上也发生了改变，反之亦然**。

<!-- more -->

---

## Java 中的浅拷贝(shallow copy)

在 java 中，如果简单地对一个对象进行赋值，那就是浅拷贝。如下面的例子， `m1` 和 `m2` 都是指向同一个 Map 容器的引用，其中一个改了，另一个也跟着改。

```java
Map m1 = rc.getResult();
Map m2 = rc.getResult();

// m2 key 为 "1" 的 value 也会更改
m1.set("1", obj);
```

如果你的 Map 存储的是基本数据类型或String，可以用 `putAll` 方法，来避免这种情况。

```java
private void process(Map<String,String> map){
    Map<String,String> m1 = new HashMap<>();
    m1.putAll(map);

    Map<String,String> m2 = new HashMap<>();
    m2.putAll(map);

    // m2 改了，m1 不会跟着改
    m2.set("1", "xin");

}
```

但假设你的 Map 存储的是引用对象类型，实际上 putAll 也是潜拷贝。因为虽然是不同的两个 map 容器，但 map 容器里存储的实际上是同一个对象，`m1` 里的对象改了， `m2` 里的对象也会改变。

```java
public static void main(String[] args) {

    Map<String,Person> map = new HashMap<>();
    map.put("1", new Person("jerry", 18));
    map.put("2",new Person("calm", 20));
    process(map);

}

private static void process(Map<String,Person> map){
    Map<String,Person> m1 = new HashMap<>();
    m1.putAll(map);
    Map<String, Person> m2 = new HashMap<>();
    m2.putAll(map);

    // m2 里面的某个对象改了，m1 里面的这个对象也跟着改
    m2.get("1").name = "xingj";


}
```

---

## Java 中的深拷贝(deep copy)

那如果希望将容器里面的内容也复制一份，就需要用到深拷贝。

jdk 并没有提供深拷贝的方法给我们使用，但 Obejct 类提供了 `clone()` 方法，`clone()` 方法是一个浅拷贝，我们可以重写该方法，实现自定义的深拷贝逻辑，基本思想就是在拷贝的时候对容器里的所有对象都进行 new 。

当然，也可以借助一些第三方库，如 `Apache Commons Lang` 库里有一个 `SerializationUtils#clone` 方法，但需要你的对象实现 `serializable` 接口。

```java
Map deepCopyMap = (Map) SerializationUtils.clone(map);
```

还有一种方式是通过 `Gson` 或 `jackson` 等框架将对象序列化后再反序列化到新的对象。