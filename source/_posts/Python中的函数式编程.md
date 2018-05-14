---
title: Python中的函数式编程
comments: true
abbrlink: 551bad5b
date: 2018-05-13 20:33:21
categories: Python
tags: Python
---

在 [Java简明笔记(九)](../post/372345f.html)中就提过函数式编程的概念，由于最近要用到 Python 的高阶函数，所以这一篇以 Python 为例，重新体会一下函数式编程的思想。

<!--more-->


---

# 高阶函数

## map

map 函数接收两个参数，一个是函数，一个是`Iterable`。

举个例子：

```python
def pow(x):
    return x*x;

r = map(pow, [1,2,3,4,5])
print(list(r))
```

输出

```
[1, 4, 9, 16, 25]
```

[1,2,3,4,5] 是一个可迭代对象，map会将函数pow分别作用于其中的每一个元素。结果r是一个`Iterator`。

## reduce

reduce函数能把结果继续和序列的下一个元素做累积计算。

```python
from functools import reduce

def sum(x, y):
    return x+y


r = reduce(sum, [1, 2, 3, 4, 5])
print(r)
```

输出

```
15
```

reduce的执行过程：

1. 1+2，结果为3
2. 3+3，结果为6
3. 6+4，结果为10
4. 10+5，结果为15

## filter

`filter()`函数用于过滤序列。和map()类似，filter()也接收一个函数和一个序列。

但是`filter()`把传入的函数依次作用于每个元素，然后根据返回值是 `True` 还是 `False` 决定保留还是丢弃该元素。其作用是筛选。

```python
def is_odd(n):
    return n % 2 == 1


r = filter(is_odd, [1,2,3,4,5])
print(list(r))
```

输出：

```
[1, 3, 5]
```

## sorted

> 在 Java 中，我们对一个类实现 Comparable 接口，那么它就有了默认的比较方法。但是我们有时候要自己定义比较的方式（比如String默认是根据首字母来比较，而我们现在想比较的是字符串的长度），就必须得实现 Comparator 接口的 compare 方法了。

在 Python 中， `sorted()`函数提供了默认的排序方法，但也支持自己定义，只需要指定第二个参数。

比如，按绝对值比较

```python
# 默认比较
sorted([36, 5, -12, 9, -21])

# 按绝对值比较
sorted([36, 5, -12, 9, -21], key=abs)
```

怎么实现的呢？实际上，是先对`[36, 5, -12, 9, -21]`进行`key=abs`操作，先生成一个key列表`[36, 5,  12, 9,  21]`，然后对key所对应的value进行 `sorted()`操作。所以最终的输出还是原始数字。

下面的例子为忽略大小写按首字母排列字符串

```python
sorted(['bob', 'about', 'Zoo', 'Credit'], key=str.lower)
```

> sorted还支持第三个参数，reverse=True 即可将结果反转。

---

# 返回函数

在 Python 高阶函数中，可以把函数作为结果值返回。


```python
def addx(x):
   def adder (y): return x + y
   return adder

add8 = addx(8)
add9 = addx(9)

print add8(100)
print add9(100)
```

## 闭包
