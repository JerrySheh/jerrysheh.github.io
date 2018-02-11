---
title: 了解makefile
categories: linux
tags:
  - C/C++
  - Linux
abbrlink: 264103f9
date: 2017-11-28 09:15:30
---


使用 gcc 命令可以很方便地在Linux下编译C源代码，但是，当我们的工程变大了之后，项目下面有很多 `.c`、`.h`文件，各种依赖关系错综复杂。这时候，手工编译就不是那么划算了。makefile就是用于解决这个问题的。

假设我现在的工程下面有5个文件，分别是：

1. main.c 主函数所在源文件
2. hello.c 输出hello的函数
3. hello.h 头文件
4. world.c 输出world的函数
5. world.h 头文件

我们可以在工程目录下，新建一个名称为`makefile`的文件（没有扩展名），然后往这个文件里写一些规则，再在shell下面执行`make`命令，就可以实现自动编译了。

<!-- more -->
---

# makefile的四种规则写法

## 一、显式规则

makefile的规范：

目标 : 依赖
(tab)命令

make的执行:
make  默认搜索当前目录下的makefile，或者Makefile文件,可以指定特殊的makefile文件，比如
`make  -f  makefile_xxxx(文件名)`

make默认实现makefile里的第一个目标，一般是二进制可执行文件
也可以指定目标，比如`make  clean`


一个简单的方法：

```
main : main.o hello.o world.o
	gcc main.o hello.o world.o -o main

main.o : main.c hello.c world.c
	gcc -c main.c -o main.o

hello.o : hello.c
	gcc -c hello.c -o hello.o

world.o : world.c
	gcc -c world.c -o world.o

clean :
	rm *.o main
```

解析：
1. main是一个可执行文件（目标），它依赖于main.o hello.o world.o这三个文件（依赖），所以我们要用gcc编译main.o hello.o world.o，以生成main
2. 我们的目录下现在还没有main.o hello.o world.o这三个文件，makefile会根据依赖文件 xxxx.o 去寻找和执行下面的依赖关系。以此类推。一步步生成上来。


---

## 二、常量替换类似于C语言的宏定义

为了方便，我们可以这样写makefile：

```
cc 		= gcc
target 	= test
objects = test.o mystrlen.o

$(target) : $(objects)
	$(cc) $(objects) -o $(target)

test.o : test.c mystrlen.h
	$(cc) -c test.c -o test.o

mystrlen.o : mystrlen.c
	$(cc) -c  mystrlen.c -o mystrlen.o


clean :
	rm $(objects) $(target)
```


---

## 三、隐式规则


还可以再精简：

```
cc 		= gcc
target 	= test
headers = mystrlen.h
objects = test.o mystrlen.o

$(target) : $(objects)
	$(cc) -o $(target) $(objects)

%.o : %.c $(headers)
	$(cc) -c $< -o $@

clean :
	rm $(objects) $(target)

```

---


## 四、shell + 隐式规则

结合shell命令，使用函数完成makefile，使其能够自动寻找目录下h文件和c文件，同时把c文件替换成o文件共object常量使用

```
cc = gcc
target = main
headers = $(shell find ./ -name "*.h")
sources = $(shell find ./ -name "*.c")
objects = $(sources:%.c=%.o)

$(target) : $(objects)
	$(cc) -o $(target) $(objects)

%.o : %.c $(headers)
	$(cc) -c $< -o $@

clean:
	-rm -rf $(objects) $(target)
```


---


# gcc常用参数

1. gcc  -c  main.c  -o  main.o   编译main.c,生成main.o目标文件
2. gcc  test1.o  test2.o  -o  test  链接两个目标文件test1.o和test2.o生成可执行文件test
3. gcc  main.c  -o  main     编译main.c,然后链接生成main可执行文件
4. gcc  main.c  mystrlen.c  hello.c   -o  main  编译多个文件
5. gcc  -g  main.c  -o  main  打开调试
6. gcc  -I  /usr/include  main.c  -o  main  指定头文件目录
7. gcc  -L  /usr/lib  -l ffmpeg  main.c  -o  main  指定库文件目录和库文件libffmpeg.so
8. gcc  -MM  main.c  查看main.o 的依赖关系，用于生成makefile
