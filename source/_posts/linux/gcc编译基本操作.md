---
title: gcc编译基本操作
categories: Linux
tags:
  - C/C++
  - Linux
abbrlink: bbe22ae6
date: 2017-11-21 08:10:47
---

假设我现在有3个文件，分别是：

* `mystrlen.c`: 是我自己实现的一个计算字符串长度的算法函数。
* `mystrlen.h`: 该算法的头文件。
* `test.c`: main函数，里面有一些字符串需要调用上面的算法来计算长度

那么在 Linux 下，如何用 gcc 把 mystrlen.c 编译成动态链接库，方便 test.c 去使用呢 ？


---

<!-- more -->

# 一、gcc各参数的用途

* -shared ：指定生成动态链接库。
* -static ：指定生成静态链接库。
* -fPIC ：表示编译为位置独立的代码，用于编译共享库。目标文件需要创建成位置无关码，就是在可执行程序装载它们的时候，它们可以放在可执行程序的内存里的任何地方。
* -L. ：表示要连接的库所在的目录。
* -l：指定链接时需要的动态库。编译器查找动态连接库时有隐含的命名规则，即在给出的名字前面加上lib，后面加上.a/.so来确定库的名称。
* -g ：编译器在编译的时候产生调试信息。
* -c ：只激活预处理、编译和汇编,也就是把程序做成目标文件(.o文件)。
* -o ：指定生成的文件名，如果不指定，默认为 a.out


* -ggdb ：此选项将尽可能的生成gdb的可以使用的调试信息。
* -Wl,options ：把参数(options)传递给链接器ld。如果options中间有逗号,就将options分成多个选项，然后传递给链接程序。
* -Wall ：生成所有警告信息。

---

# 二、使用gcc把 mystrlen 编译成动态库 libmystrlen.so

## 1. 把 mystrlen.c 编译成目标文件

```
gcc -c mystrlen.c -o mystrlen.o
```

在当前目录会生成 `mystrlen.o`文件

## 2. 把目标文件编译成动态链接库

```
gcc -shared -fPIC mystrlen.o -o libmystrlen.so
```

在当前目录会生成 `libmystrlen.so`文件

---

# 三、使用gcc把 test.c 编译成可执行文件

## 1.编译
```
gcc -L ./ test.c -lmystrlen -o test
```

在当前目录会生成 `test`可执行文件

## 2.运行

```
./test
```

程序里放了4个测试字符串，长度分别为0 3 10 26，如果匹配，则输出 pass

输出结果：

```
data_0 pass [0]
data_1 pass [3]
data_2 pass [10]
data_3 pass [26]
```

值得注意的地方
1. 我们的动态链接库文件名是`libmystrlen.so`，但在`-l`参数中，去掉lib和.so，只需要`mystrlen`就可以。 -l后面可以不用空格。
2. 在gcc编译的时候，如果文件a依赖于文件b，那么编译的时候必须把a放前面，b放后面。 所以， 命令中`test.c`一定要放在`-lmystrlen`前面。

---

# 三、可能出现的报错

##  error while loading shared libraries

```
jerrysheh@ubuntu:~/shiyan9$ ./test
./test: error while loading shared libraries: libmystrlen.so: cannot open shared object file: No such file or directory

```

这是因为程序运行时找不到我们自己的动态链接库文件，解决办法很简单：

### 1. 打开配置文件 /etc/ld.so.conf

```
sudo vim /etc/ld.so.conf
```

### 2. 在配置文件的最后追加一行你的库文件所在的路径即可
编辑完后类似这样：
```
include /etc/ld.so.conf.d/*.conf
/home/jerrysheh/shiyan9/

```

### 3. 刷新配置文件

```
sudo ldconfig
```

### 4. 重新运行 test 程序

```
./test
```

## 找不到头文件

如果头文件跟 .c 文件不在同一目录，使用 `-I` 参数，指定头文件的路径即可

如头文件在 `/home/jerrysheh/shiyan9/include` 里面，只需要

```
gcc -I /home/jerrysheh/shiyan9/include -L ./ test.c -lmystrlen -o test
```
