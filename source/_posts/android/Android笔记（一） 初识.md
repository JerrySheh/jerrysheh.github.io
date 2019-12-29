---
title: Android笔记（一） 初识
comments: true
abbrlink: 4afa9fc3
date: 2018-01-21 20:21:23
categories: Android
tags: Android
---

![Android 8](../../../../images/Learn_Android/Android.jpg)

# Android 简介

> Android，常见的非官方中文名称为安卓，是一个基于Linux内核的开放源代码移动操作系统，由Google成立的Open Handset Alliance（OHA，开放手持设备联盟）持续领导与开发，主要设计用于触屏移动设备如智能手机和平板电脑与其他便携式设备。

> Android最初由安迪·鲁宾等人开发制作，最初开发这个系统的目的是创建一个数码相机的先进操作系统；但是后来发现市场需求不够大，加上智能手机市场快速成长，于是Android成为一款面向智能手机的操作系统。

> 于2005年7月11日被美国科技企业Google收购。

> 2007年11月，Google与84家硬件制造商、软件开发商及电信营运商成立开放手持设备联盟来共同研发改良Android，随后，Google以Apache免费开放源代码许可证的授权方式，发布了Android的源代码（[Android OpenSource Projet](https://source.android.com)），开放源代码加速了Android普及，让生产商推出搭载Android的智能手机，Android后来更逐渐拓展到平板电脑及其他领域上。

> 2010年末数据显示，仅正式推出两年的Android操作系统在市场占有率上已经超越称霸逾十年的诺基亚Symbian系统，成为全球第一大智能手机操作系统。

> 在2014年Google I/O开发者大会上Google宣布过去30天里有10亿台活跃的安卓设备，相较于2013年6月则是5.38亿。

> 2017年3月，Android全球网络流量和设备超越Microsoft Windows，正式成为全球第一大操作系统。

> 2017年8月，Android O发布。

以上简介来自Wikipedia，可见 Android 前景还是非常大的。作为一个移动开发者，学习Android也是势在必行了。

本次Android的学习，我采用书本+视频的方式。

参考书籍是：
* [第一行代码（第二版）（郭霖）](https://book.douban.com/subject/26915433/)


参考视频：

对于没有编程基础的:
* [Android 基础：用户界面](https://classroom.udacity.com/courses/ud834)
* [Android 基础：用户输入](https://cn.udacity.com/course/android-basics-user-input--ud836)
* [Android 基础：多屏应用](https://cn.udacity.com/course/android-basics-multi-screen-apps--ud839)
* [Android 基础：网络](https://cn.udacity.com/course/android-basics-networking--ud843)
* [Android 基础：数据存储](https://cn.udacity.com/course/android-basics-data-storage--ud845)

有编程基础可以直接从这个开始：
* [Android应用开发](https://classroom.udacity.com/courses/ud851)

<!-- more -->

---

# Android 系统架构

Android系统架构可以分为五层，分别是：


* **应用层（System app）**，所有安装在手机上的app都是应用层的，包括系统自带app和第三方开发的app。
* **应用框架层（Java API Framework）**，提供了构建应用程序时需要的API。
* **系统运行库（Native C/C++ Libraries） 和 Android Runtime**， 通过C/C++库来为Android系统提供主要的特性支持。如SQLite库提供数据库支持，OpenGL/ES库提供3D绘图支持，Webkit库提供浏览器内核支持等。Android Runtime是Android的核心库，为Android应用跨平台使用提供的可靠方案，每个app都会有自己独立的运行空间和虚拟机。允许开发者使用Java、Kotlin编写Android应用。
* **硬件抽象层（Hardware Abstraction Layer, HAL）** 主要与manufacture和chip vendor相关，manufacture提供HAL的实现以及各种硬件设备的驱动和集成chip vendor提供的firmware。
* **Linux内核层（Linux Kernel）**，为硬件提供了底层驱动，如显示驱动，照相机驱动，音频驱动，蓝牙驱动等。

![Android Framework](../../../../images/Learn_Android/Android_Framework.png)

关于 Framework 的一些知识点：

> 隐藏在每个应用后面的是一系列的服务和系统, 其中包括；
>
>a. **丰富而又可扩展的视图（Views）**，可以用来构建应用程序， 它包括列表（lists），网格（grids），文本框（text boxes），按钮（buttons）， 甚至可嵌入的web浏览器。
>
>b. **内容提供器（Content Providers）** 使得应用程序可以访问另一个应用程序的数据（如联系人数据库）， 或者共享它们自己的数据。
>
>c. **资源管理器（Resource Manager）** 提供非代码资源的访问，如本地字符串，图形，和布局文件（layout files）。
>
>d. **通知管理器（Notification Manager）** 使得应用程序可以在状态栏中显示自定义的提示信息。
>
>e. **活动管理器（Activity Manager）** 用来管理应用程序生命周期并提供常用的导航回退功能。

---

# Android 四大组件

Android 系统的四大组件分别是活动（Activity）、服务（Service）、广播接收器（Broadcast Receiver）、内容提供器（Content Providers）。

## 活动（Activity）

在 Android 应用中我们能看到的东西，都是放在活动当中的。活动是一种能够包含用户界面的组件，主要用于和用户交互。值得注意的是，在Android中，有一种能够嵌入在活动当中的UI片段，称为`碎片（Fragment）`，它和活动十分相似。

## 服务（Service）

服务是 Android 中实现程序后台运行的解决方案。它非常适合去执行那些不需要和用户交互而且需要长期运行的任务。服务依赖于所在应用程序的进程，不单独运行。值得注意的是，服务不会自动开启线程，所有的代码默认在主线程运行。因此需要我们手动在服务内部创建子线程。

## 广播接收器（Broadcast Receiver）

广播接收器允许你的应用接收来自各处的广播消息，比如电话、短信等。同样也可以向外发出广播消息。Android 中的广播可以分为以下两种：

* `标准广播（Normal Broadcast）`，一种完全异步执行的广播，当广播发出后，所有的广播接收器几乎同一时刻接收到这条广播信息，没有先后顺序之分。这种广播效率高，但无法截断。

* `有序广播（Ordered Broadcast）`，一种同步执行的广播，当广播发出后，同一时刻只有一个广播接收器收到这条广播，当这个广播接收器的逻辑执行完毕后，广播才会继续传递。这样一来，优先级高的广播接收器可以先收到广播，并且前面的广播接收器可以截断正在传递的广播。

## 内容提供器（Content Providers）

内容提供器则是为应用程序之间共享数据提供可能的机制。比如，你的应用程序需要读取系统通讯录中的联系人信息，就需要通过内容提供器实现。

---

# Material Design

Material Design(材料设计语言)，是由Google推出的全新的设计语言，谷歌表示，这种设计语言旨在为手机、平板电脑、台式机和“其他平台”提供更一致、更广泛的“外观和感觉”。

![MD](http://www.comestsavatino.com/wp-content/uploads/2014/12/finance_3d_web.jpg)


---

# Android Studio

Android Studio 是 Google 官方提供的 Android IDE，提供用于为各类 Android 设备开发应用的最快速的工具。

下载地址：https://developer.android.com/studio/index.html


![Studio](https://developer.android.com/images/tools/studio/studio-feature-devices_2x.png)

下载和使用 Android Studio 需要连接 Goole 服务器下载数据，然而根据众所周知的原因，无法科学上网的童鞋需要另辟蹊径了。最好还是学会如何科学上网。（当然，是为了学习和开发）

在第一次创建Android Studio project的时候，如果没有科学上网，有可能会遇到卡在`building “project name”gradle project info`的问题。可以通过下载离线gradle包手动安装。 [参考链接](http://blog.csdn.net/liuhuiyi/article/details/21861733)

---

# HelloWorld

使用 Android Studio 创建第一个 helloworld project之后，会自动生成一些代码。先来看一下这些自动生成的代码都表示什么。

## AndroidManifest.xml

在工程目录下可以看到AndroidManifest.xml，在这个xml里面有如下语句

```xml
<activity android:name=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />

        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

Android 应用程序基于活动，而每一个活动都要在 AndroidManifest.xml 进行注册，没有注册的活动是不能使用的。这段代码就是注册了 `MainActivity`这个活动。


## MainActivity.java

MainActivity.java中自动生成的代码如下：

```java
package com.jerrysheh.hello;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
}

```

分析：
* `MainActivity`继承自`AppCompatActivity`，让这个活动支持系统版本向下兼容。
> `Activity`是 Android 系统提供的一个活动基类，我们项目中所有的活动都必须继承它或者它的子类。`AppCompatActivity`是`Activity`的子类。

* `onCreate`方法第二行调用了`setContentView()`，参数指向了`layout.activity_main`。也就是说，`setContentView()`方法将我们的`activity_main.xml`和`MainActivity.java`关联了起来。

不妨打开activity_main.xml看看。

## activity_main.xml

打开activity_main.xml，Android Studio默认是Design视图，在下面按钮切换到text视图。

可以看到，activity_main中有几行关键代码：

```xml
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Hello World!"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintLeft_toLeftOf="parent"
    app:layout_constraintRight_toRightOf="parent"
    app:layout_constraintTop_toTopOf="parent" />
```

不用说，定义了视图界面以及显示的信息。 我们熟悉的`Hello World!`正是在这里。

## 小结

Android程序设计讲究逻辑和视图分离，所以逻辑代码写在MainActivity.java中，视图界面写在activity_main.xml中。在逻辑代码中的活动，需要先到AndroidManifest.xml进行注册。

> 方便的是，当我们创建一个新的 Activity，Android Studio会自动帮我们注册。如果你用的是 Eclipse 或其他IDE，可能要手动进行注册。

---

# build.gradle

Gradle是一个先进的项目构建工具，它使用一种基于Groovy的领域特定语言（DSL）来声明项目设置，摒弃了传统基于XML（如Ant和Maven）的各种繁琐设置。

Android Project 会首先经过gradle构建，gradle会将你的代码构建成字节码和资源，打包成APK，然后经过jar签名，最后通过adb安装到虚拟或真实的Android设备上。

![project](../../../../images/Learn_Android/stru.png)

在Android Studio中，有多种项目构造目录，其中默认的Android方式是IDE自动调整的，方便我们查看文件。而只有project目录结构是真实存放在我们硬盘里的组织结构。

![project](../../../../images/Learn_Android/Android_project.png)


切换到project目录结构，可以看到 Helloworld项目有两个 build.gradle 文件。

## 外层build.gradle

```
buildscript {

    repositories {
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.0.1'
    }
}

allprojects {
    repositories {
        google()
        jcenter()
    }
}
```

* jcenter是一个代码托管仓库，声明了`jcenter()`后我们可以轻松引用任何jcenter上的开源项目。`google()`同理。

* `classpath`声明了一个 Gradle插件。用于构建 Android 项目。

## 内层build.gradle

```
apply plugin: 'com.android.application'

android {
    compileSdkVersion 26
    defaultConfig {
        applicationId "com.jerrysheh.hello"
        minSdkVersion 15
        targetSdkVersion 26
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'com.android.support:appcompat-v7:26.1.0'
    implementation 'com.android.support.constraint:constraint-layout:1.0.2'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'com.android.support.test:runner:1.0.1'
    androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.1'
}

```

* 第一行`com.android.application`表示这是一个应用程序模块，如果是`com.android.library`表示这是一个库模块。应用程序模块可以直接运行，库模块需要依附别的应用程序模块。

* `android`闭包配置了项目构建的属性。

* `buildTypes`闭包用于指定生成安装文件的相关配置。

* `dependencies`闭包指定当前项目的所有依赖关系。Android Studio有3种依赖方式：本地依赖、库依赖、远程依赖。

---


# Android Debug Bridge （ADB）

ADB 是 Android 的 SDK 包含的命令行工具。用于调试。

要从命令行中启动 Android 应用，你可以输入：
```
adb shell am start -n com.package.name/com.package.name.ActivityName
```
