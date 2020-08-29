---
title: Linux内核模块编程 HelloWorld
comments: true
categories: Linux
tags: Linux
abbrlink: 75b0adbf
date: 2018-03-07 13:03:26
---

# 微内核和宏内核

## 微内核

内核中只有最基本的调度、内存管理。其他的比如驱动、文件系统等都是用户态的守护进程去实现的。比如Windows NT、OS X

优点是超级稳定，驱动等的错误只会导致相应进程死掉，不会导致整个系统都崩溃，做驱动开发时，发现错误，只需要kill掉进程，修正后重启进程就行了，比较方便。

缺点是效率低。典型代表QNX，QNX的文件系统是跑在用户态的进程，称为resmgr的东西，是订阅发布机制，文件系统的错误只会导致这个守护进程挂掉。

## 宏内核

简单来说，就是把很多东西都集成进内核，例如Linux内核，除了最基本的进程、线程管理、内存管理外，文件系统，驱动，网络协议等等都在内核里面。优点是效率高。缺点是稳定性差，开发过程中的bug经常会导致整个系统挂掉。做驱动开发的应该经常有按电源键强行关机的经历。

<!--more-->

---

# 内核模块

由于 Linux 内核是宏内核，集成性比较高，随着内核版本的迭代，内核变得非常大（Linux内核约50M），我们想定制自己的内核时，需要整个重新编译，比较繁琐。而且，定制内核时，有些功能我们是不需要的。

因此 Linux 内核采用了模块化的方式。当我们编译内核时，可以只选择我们需要的模块。而且，我们自己编写的模块，可以采用安装/卸载的方式，集成到内核里。

内核模块的优点是：开发效率更高，而且可以在内核运行时`动态加载`。由于Linux内核模块是动态加载的，所以它也叫`可加载内核模块(Loadable Kernel Module, LKM)`。Linux内核镜像位于/boot目录下，启动时最先加载，LKM总是在内核启动之后加载。

LKM主要用于：设备驱动、文件系统驱动和系统调用。

* **注意**：LKM是内核空间程序，不是用户空间程序，你可以把它看成是内核的一部分。也就是LKM没有任何保护，一不小心可能就会导致系统崩溃。

---

# 编译LKM

## 安装C编译器和Linux内核头文件

```
sudo apt-get install build-essential linux-headers-$(uname -r)
```

## Hello World内核模块代码（hello.c）

```c
#include <linux/module.h>     /* 模块头文件，必不可少 */
#include <linux/kernel.h>     /* KERN_INFO在这里 */
#include <linux/init.h>       /* 使用的宏 */

// LICENSE
MODULE_LICENSE("GPL");

// 作者
MODULE_AUTHOR("blog.topspeedsnail.com");

// 描述
MODULE_DESCRIPTION("hello world");

// 模块版本
MODULE_VERSION("3.14");

static int __init hello_start(void)
{
    printk(KERN_INFO "Hello World\n");
    return 0;
}

static void __exit hello_end(void)
{
    printk(KERN_INFO "Go to Hell\n");
}

module_init(hello_start);
module_exit(hello_end);
```

* **module_init** 定义了模块的入口函数，在模块加载 insmoded (插入模块)时执行
* **module_exit** 定义了模块的退出函数，在模块卸载 rmmoded （移除模块）时执行

## 创建Makefile

```
obj-m = hello.o
all:
    make -C /lib/modules/$(shell uname -r)/build/ M=$(PWD) modules
clean:
    make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
# make命令前是tab，不是空格
```

## 编译

make命令

```
make
```

输出

```
jerrysheh@ubuntu:~/Desktop$ make
make -C /lib/modules/4.13.0-31-generic/build/ M=/home/jerrysheh/Desktop modules
make[1]: Entering directory '/usr/src/linux-headers-4.13.0-31-generic'
  CC [M]  /home/jerrysheh/Desktop/helloworld.o
  Building modules, stage 2.
  MODPOST 1 modules
  CC      /home/jerrysheh/Desktop/helloworld.mod.o
  LD [M]  /home/jerrysheh/Desktop/helloworld.ko
make[1]: Leaving directory '/usr/src/linux-headers-4.13.0-31-generic'
```

## 查看

使用`modinfo hello.ko`来查看模块信息

使用`sudo insmod hello.ko`来加载模块到内核

使用`lsmod`来查看已加载的内核（结合管道`lsmod | grep hello`）

使用`sudo rmmod hello`来卸载模块

## 输出

使用`tail /var/log/kern.log`来查看模块的输出

```
jerrysheh@ubuntu:~/Desktop$ tail /var/log/kern.log
Mar  7 12:53:59 ubuntu NetworkManager[863]: <info>  [1520398439.9770]   gateway 192.168.224.2
Mar  7 12:53:59 ubuntu NetworkManager[863]: <info>  [1520398439.9770]   server identifier 192.168.224.254
Mar  7 12:53:59 ubuntu NetworkManager[863]: <info>  [1520398439.9770]   lease time 1800
Mar  7 12:53:59 ubuntu NetworkManager[863]: <info>  [1520398439.9770]   nameserver '192.168.224.2'
Mar  7 12:53:59 ubuntu NetworkManager[863]: <info>  [1520398439.9770]   domain name 'localdomain'
Mar  7 12:53:59 ubuntu NetworkManager[863]: <info>  [1520398439.9770]   wins '192.168.224.2'
Mar  7 12:53:59 ubuntu NetworkManager[863]: <info>  [1520398439.9771] dhcp4 (ens33): state changed bound -> bound
Mar  7 12:56:41 ubuntu kernel: [ 1170.100050] helloworld: loading out-of-tree module taints kernel.
Mar  7 12:56:41 ubuntu kernel: [ 1170.100085] helloworld: module verification failed: signature and/or required key missing - tainting kernel
Mar  7 12:56:41 ubuntu kernel: [ 1170.100760] Hello World
```

看到最后一行输出了 Hello World，正是我们编写的 helloworld.ko 模块输出的。

可以使用`tail -f /var/log/kern.lo`来动态监控内核的输出

## 作为字符型驱动


在 insmod 的时候把设备的主设备号打印出来。

如果要把内核模块作为字符型驱动设备，首先

```
tail -f /var/log/messgaes
```

查看设备的主设备号，比如是 254

然后创建目录和节点

```
mkdir /dev/demo
mknod /dev/demo/newdev c 254 0
```

这样就创建了一个虚拟的字符型驱动设备。

---


本文引用的文章：

* [编写第一个Linux内核模块: Hello World](http://blog.topspeedsnail.com/archives/10053)

* [宏内核与微内核、Linux内核与Unix内核的区别](http://blog.csdn.net/silencegll/article/details/51496158)
