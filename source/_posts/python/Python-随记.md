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

## 直接参数

有时候我们用命令行运行 python 程序，如：

```
python test.py jerry 20
```

这里的 jerry 和 20 都是参数，在 python 里面用 sys 模块来获取参数：

```python
import sys

filename = sys.argv[0]  # test.py
user = sys.argv[1]  # jerry
age = sys.argv[2]  # 20
```

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

# 切片（Slice）操作

有一个 List

```python
L = ['Michael', 'Sarah', 'Tracy', 'Bob', 'Jack']
```

切片操作

```python
# 取出前3个元素
L[0:3]  # ['Michael', 'Sarah', 'Tracy']
L[:3]  # 其中，0可以省略

# 取出下标为1（包含）到 4（不包含）的元素
L[1:4]
```

倒数切片

```python
# 取出-2（包含）到0（不包含）
L[-2:0]  # ['Bob'，'Jack']
```

还可以每隔两个数取一个

```python
# 下标为0（包含）到10（不包含）
L[0:10:2]

```

如果看到下面这种操作，表示所有数每隔5个取一个

```python
L[::5]
```

## tuple

tuple 是另一种有序列表，中文叫元组。它也可以切片操作，操作结果仍为 tuple。 **tuple 跟 list 的区别在于一旦初始化便不可更改。**

看一个 tuple 例子

```python
>>> t = ('a', 'b', ['A', 'B'])
>>> t[2][0] = 'X'
>>> t[2][1] = 'Y'
>>> t
('a', 'b', ['X', 'Y'])
```

在这个例子中， t[0] 表示 tuple 的第一个元素，也就是 `'a'`， t[1] 表示 `'b'`， t[2]表示 `['A', 'B']`

---

# 连接数据库

```python
import pymysql


connect = pymysql.connect(host="127.0.0.1", port=3306, user="root", passwd="YOURPASSWD", db="YOURDBNAME")

cursor = connect.cursor()

sql = "SELECT * FROM rate"

cursor.execute(sql)
results = cursor.fetchall()

file = open("C:\\Users\\JerrySheh\\IdeaProjects\\mall\\dataset\\douban_large_clean.dat", "w")

for row in results:
    userid = str(row[0])
    bookid = str(row[1])
    rating = str(int(row[2]))
    line = str(userid + "::" + bookid + "::" + rating + "\n")
    print(line)
    file.write(line)

file.close()
```
