---
title: Python 随记
comments: true
categories: Python
tags: Python
abbrlink: f9031a66
date: 2018-04-04 17:27:36
---

# Python 程序入口

有时候我们会看到

 ```python
 if __name__ == '__main__':
     dosomething()
     # ...
 ```

 这个语句中， `__name__` 是当前模块名，当模块被直接运行时模块名为 `__main__ `。

这句话的意思就是，当模块被直接运行时，`dosomething()`块将被运行，当模块是被导入时，`dosomething()`块不被运行。

<!-- more -->

## -m 参数

`python xxx.py` 与 `python -m xxx.py` ，这两种运行 Python 程序的方式的不同点在于，一种是直接运行，一种是当做模块来运行。

- 直接运行的时候，xxx.py 所在目录是 sys.path
- 以模块运行的时候，当前工作路径是 sys.path

参考：

- [Python 中的 if \_\_name__ == '\_\_main__' 该如何理解](http://blog.konghy.cn/2017/04/24/python-entry-program/)

---

# python 3.6 print新特性: Formatted string literals

## 使用 %

在 python 3.6 之前，我们想 print ， 一般是用 `%` 符号

```python
name = "jerry"
age = 18
print("my name is %s, and I am %d years old" % (name, age))
```

## 使用 {}

在 python 3.6 可以用 {} ，类似于 kotlin 语法

* **注意**： 引号前面有个 `f`

```python
name = "jerry"
age = 18
print(f"my name is {name}, and I am {age} years old")
```

可以在 {} 里运算

```python
width = 10
precision = 4  # 有效数字
value = decimal.Decimal("12.34567")
print(f"result: {value:{width}.{precision}}")  # nested fields
```

输出：12.34 （4位有效数字）

参考

- [what is New in python 3.6](https://docs.python.org/3/whatsnew/3.6.html)

---

# 获取输入参数

用 sys.argv[] 来获取输入参数

myinfo.py
```python
import sys

name = sys.argv[1]
age = sys.agrv[2]
print(f"my name is {name}, and I am {age} years old")
```

终端输入
```
$ python3 myinfo.py jerry 18
```

终端输出
```
$ my name is jerry, and I am 18 years old
```

- myinfo.py 本身是一个参数， 即 argv[0]
- jerry 是argv[1]
- 18 是argv[2]

* **注意**： 输入参数都是 string 类型， 这里的 18 其实是字符串18，不是数字18，可以用` int(sys.agrv[2]) ` 转换成数字18

---

持续更新
