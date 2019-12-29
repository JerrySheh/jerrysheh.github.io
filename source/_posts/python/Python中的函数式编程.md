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

# 高阶函数概念

在 Python 中，`abs(-10)`用于取绝对值， 其中，-10 是参数，abs 是函数本身，`abs()`才是函数调用。我们可以把函数本身作为变量，赋值给另一变量。

如假设我们定义`f = abs`，现在，`f(-10)` 和 `abs(-10)` 完全一样。

既然函数本身可以作为变量赋值，那么它就可以作为参数，传给另一个函数。 如

```python
def add(x, y, f):
    return f(x) + f(y)

f = abs
add(-5, 6, f)
```

当我们调用` add()` 的时候， 把函数`f`本身传递了进去。

像这样，如果有一个函数，它可以接收另一个函数作为参数，这种函数就称之为`高阶函数（Higher-order function）`。


---

#  Iterable 和 Iterator

可以直接作用于 for 循环的对象统称为`可迭代对象（Iterable）`。

可以被 `next()` 函数调用并不断返回下一个值的对象称为`迭代器（Iterator）`。

Python的 Iterator 对象表示的是一个数据流，Iterator对象可以被 `next()` 函数调用并不断返回下一个数据，直到没有数据时抛出 StopIteration 错误。

可以把这个数据流看做是一个有序序列，**但我们却不能提前知道序列的长度**，只能不断通过 `next()` 函数实现按需计算下一个数据，**所以 Iterator 的计算是惰性的**，只有在需要返回下一个数据时它才会计算。

list、dict、str等数据类型不是 Iterator，因为他们的长度是可知的。

---

# Python内置的高阶函数

## map

map 函数接收两个参数，一个是函数，一个是`Iterable`。返回一个`Iterator`。

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

reduce函数能把结果继续和序列的下一个元素做累积计算。结果就是计算的结果。

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

`filter()`函数用于过滤序列。和`map()`类似，`filter()`也接收一个函数和一个序列。

但是`filter()`把传入的函数依次作用于每个元素，然后根据返回值是 `True` 还是 `False` 决定保留还是丢弃该元素。其作用是筛选。

`filter()`返回的也是一个`Iterator`。

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
def lazy_sum(*args):
    def sum():
        ax = 0
        for n in args:
            ax = ax + n
        return ax
    return sum
```

当我们调用 `lazy_sum()` 时，返回的是 `sum` 函数本身，而不是一个具体的数据类型。

## 闭包

在函数中定义另一函数，内部函数可以引用外部函数的参数和局部变量。当外部函数返回内部函数时，相关的参数和变量都保存在返回的内部函数中。这种程序结构，称为`闭包`。

> vczh：闭包不是“封闭内部状态”，而是“封闭外部状态”。一个函数如何能封闭外部状态呢？当外部状态的 scope 失效的时候，还有一份留在内部状态里面。

---

# 匿名函数

在使用 `map()` 函数的时候，我们传入一个函数参数和一个`Iterable`。我们还要特地去定义传入的函数，比如：

```python
def f(x):
    return x * x

map(f(x), [1, 2, 3, 4, 5, 6, 7, 8, 9])
```

我们可以使用 lambda 表达式，直接表示 f(x) ，而不用特地去定义。

```python
map(lambda x: x * x, [1, 2, 3, 4, 5, 6, 7, 8, 9])
```

这就是匿名函数和lambda表达式的用法。

---

# 装饰器（Decorator）

## 函数对象

在 Python 中，函数是对象，可以赋值给变量

```python
def now():
    print("2018-05-17")

myfun = now

# 可以通过 __name__ 属性获取函数的名字
now.__name__
myfun.__name__
```

## 装饰器

现在，我们要在调用 now() 函数前，执行日志打印，但又不希望修改 `now()` 函数的内容。这时候可以用`装饰器（Decorator）`。

在代码运行期间动态增加功能的方式就是 Decorator，本质上 Decorator 就是一个返回函数的高阶函数。

```python
def log(func):
    def wrapper(*args, **kw):
        print('call %s():' % func.__name__)
        return func(*args, **kw)
    return wrapper

def now():
    print('2018-05-17')

now = log(now)

now()
```

我们把 `now()` 函数作为参数，传给 `log()` 函数，然后`log()` 函数返回一个内部函数 wrapper。 而 wrapper 在返回前打印了日志，然后返回了传进去的 `now()` 函数。

现在， now 这个变量保存了 wrapper() 函数。（注意：wrapper并没有执行，只是保存了状态）

最后，我们调用 `now()`，`now()` 执行刚刚保存的 wrapper() ，也就是打印日志，然后返回 now() 本身，也就是打印`2018-05-17`。

所以，我们最后看到的结果是：

```
call now():
2018-05-17
```

这就实现了我们想要的功能：在调用 now() 函数前，执行日志打印。

事实上，为了方便，我们可以用 `@log` 来修饰函数 ，如

```python
@log
def now():
    print('2015-3-25')
```

等同于

```python
def now():
    print('2018-05-17')

now = log(now)
```

用装饰器装饰过后的函数， `__name__` 会变成装饰函数的内部函数，以这个例子为例， `now.__name__` 从 `now` 变成了 `wrapper`。所以我们要在装饰器函数里把原始函数的 __name__ 属性复制到 `wrapper()` 内部函数中。

你可能马上想到用`wrapper.__name__ = func.__name__`，事实可以直接`@functools.wraps(func)`。

一个完整的Decorator：

```python
import functools

def log(func):
    @functools.wraps(func)
    def wrapper(*args, **kw):
        print('call %s():' % func.__name__)
        return func(*args, **kw)
    return wrapper
```
