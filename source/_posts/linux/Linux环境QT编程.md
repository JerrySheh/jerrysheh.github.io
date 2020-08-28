---
title: Linux环境QT编程
comments: true
categories: Linux
tags: linux
abbrlink: 10e2f7a0
date: 2018-04-11 13:26:40
---

由于课程需要，需要在 Linux 环境下开发 QT 程序，因此开一篇文章来记录QT的知识点。

<!-- more -->

# Ubuntu 16.04 QT安装

由于实验室和课程安排都是基于 QT4，因此这里安装QT4

```
sudo apt install qt4*
sudo apt install qtcreator
```

---

# QT 基本操作

## 屏幕自适应

先布局一个 layout，然后在全局框里，设置 layout

用以下代码自动全屏

```C++
QDesktopWidget* desktopWidget = QApplication::desktop();
QRect screenRect = desktopWidget->screenGeometry();
resize(screenRect.width(),screenRect.height());
this->showMaximized();
```

## 显示图片

1. 布局一个 label
2. 头文件添加 `#include <QPixmap>`
3. 构造函数添加 `ui->label->setPixmap("0.png")`

## 定时器

1. 主类声明一个指针 public: 下， `QTimer *display_timer;`
2. 头文件添加 `#include <QTimer>`
3. 构造函数添加

```c++
display_timer = new QTimer(this);

//槽函数
connect(display_timer,SIGNAL(timeout()), this, SLOT(doChange()));
display_timer->start(10000);
```

4. 头文件 public slots: 声明 `doChange();`
5. 实现`doChange();`函数

```c++
void MainWindow::doChange(){
    qDebug("doChange");
    ui->label->setPixmap(QPixmap(QString::number(num) + ".png"));
}
```

- 使用`display_timer->start(1000);`来开启定时器
- 使用`display_timer->stop(1000);`来关闭定时器



---

# 实验室环境QT交叉编译

将QT程序编译成实验室开发板 ARM-linux 下能执行的程序。

## 将工程文件拷贝到编译目录

```
cp myproject /usr/local/Trolltech/QtEmbedded-4.8.5-arm/examples/ -a
```

## 执行 qmake

这一步主要是生成 Makefile

实验室里为了区分，把 qmake 命令换成了 qmake-arm

```
qmake-arm
```

## 执行 make

```
make
```

make完成后，可执行文件就生成了。通过串口传输到ARM实验箱上。

串口连接过程略。

## 串口:本地 -> ARM

```
rx myAPP
```

选择 传输 -> 发生 Xmodem

## 解决ARM实验箱的坑

- kill掉以下三个进程
```
/opt/Qtopia/bin/qpe
/opt/Qtopia/bin/qss
/opt/Qtioua/bin/quicklauncher
```

- 初始化环境变量

```
. setqt4env  # 不要漏了中间的空格
```

- 运行

```
./myApp -qws
```
