---
title: Java简明笔记（十七）注解
comments: true
categories:
- Java
- Java SE
tags: Java
abbrlink: b3112bec
date: 2019-06-04 11:21:31
---

# 注解魔法

注解是一种标记。在 Java 中，随处可见`@Override`、`@Deprecated`这样的注解。说实话，Java的注解经常不被重视，以至于学习的时候习惯性略过。在学了Spring框架后发现Spring使用了大量的注解来简化开发和配置，回过头来才发现注解的魅力。

<!-- more -->

插句题外话，一开始让我感受到注解的强大和优雅的，不是Java，而是在学习 Python 时遇到的 **decorator**，如下：

```python
def draw_lighting_decorator(func):
    @functools.wraps(func)
    def wrapper():
        func()
        print("lighting")
    return wrapper


@draw_lighting_decorator
def draw():
    print("draw")
```

只需通过`@draw_lighting_decorator`标记，就能让`draw()`函数执行后自动执行`print("lighting")`，而无需修改`draw()`函数本身。

在 Spring 框架中也是，如下：

```java
@Service
public class ProductService {
    @Autowired
    private ProductMapper productMapper;

    //...
}
```

只需使用`@Service`标记，就能让框架知道这是一个MVC中的Service，只需通过`@Autowired` 就能实现在Service中自动注入一个mapper组件。

这到底是什么魔法？

---

# Override注解探究

Override是一个注解接口，在 java.lang 包下：

```java
package java.lang;
import java.lang.annotation.*;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```

它似乎什么都没干，就能检查父类中有一个同名的方法。然而，不是这样的！**Annotations仅仅是元数据，和业务逻辑无关**。也就是说，Annotations只是一个标记，而对做了这个标记的地方要做什么业务逻辑，是由其他代码来实现的。就`@Override`这个注解来说，用户是JVM，它在字节码层面来实现检查重写的业务逻辑。

我们完全可以写自己的注解，在需要的地方标记上，然后在另一个地方提供业务逻辑。

---

# 自定义注解

## 定义注解

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface IpAddress {
    String ip() default "127.0.0.1";
}
```

- `@Target`表示该注解可以作用在哪里（方法、类、属性）
- `@Retention`表示该注解的生命周期

> RetentionPolicy.SOURCE – 在编译阶段丢弃。这些注解在编译结束之后就不再有任何意义，所以它们不会写入字节码。@Override, @SuppressWarnings都属于这类注解。
>
> RetentionPolicy.CLASS – 在类加载的时候丢弃。在字节码文件的处理中有用。注解默认使用这种方式。
>
> RetentionPolicy.RUNTIME– 始终不会丢弃，运行期也保留该注解，因此可以使用反射机制读取该注解的信息。我们自定义的注解通常使用这种方式。

## 使用注解

```java
public class AnnotationExample {

    @IpAddress(ip = "192.168.1.1")
    public void doSomething(){
        System.out.println("do");
    }

    public static void main(String[] args) throws NoSuchMethodException {
        AnnotationExample exam = new AnnotationExample();
        IpAddress addr = exam.getClass().getMethod("doSomething").getAnnotation(IpAddress.class);
        String ip = addr.ip(); // 192.168.1.1
    }

}
```

在 `doSomething()` 方法上的`@IpAddress` 注解只是标记，真正的业务逻辑在`main()`方法里做。通过反射的方式，获取类中某个方法的注解，再对注解进行解析。

---

# 注解 + AOP 实现业务增强

在 Spring 框架中，经常可以见到使用 `@Translaction` 注解就可以自带事务，叫做 **声明式事务**。事实上，被 `@Translaction` 标记的方法，Spring 在另一个地方会通过反射+动态代理的方式，对该方法进行增强，从而加入事务功能，确实很“魔法”。

未完待续

---

参考：
- [Java中的注解到底是怎么工作的？](https://zhuanlan.zhihu.com/p/67967745)
