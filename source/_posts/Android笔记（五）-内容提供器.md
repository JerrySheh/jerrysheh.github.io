---
title: Android笔记（五） 内容提供器
comments: true
categories: Android
tags: Android
abbrlink: f85eb43b
date: 2018-03-03 22:34:43
---

# 什么是Content Provider ?

内容提供器（Content Provider）主要用于在不同的应用程序之间实现数据共享。它提供一套机制，允许在数据安全的条件下，让一个程序访问另一个程序的共享数据。比如，读取联系人app的电话号码数据等。

Content Provider可选择只对哪一部分数据进行共享，从而保证隐私。

---

# 运行时权限（Runtime permissions）

Android 6.0 之后，Android 系统的权限分为两类：

* 普通权限
* 危险权限

普通权限在 AndroidManifest.xml 文件里直接声明，如下：

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.jerrysheh.englishquickchecker">
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
    ...
</manifest>
```

危险权限则需要使用`运行时权限（runtime permssions）`。

Android 6.0 引入了`运行时权限（runtime permssions）`，简单地说，就是在应用程序运行的时候，由用户手动授权是否调用手机的某些数据（比如，麦克风、相机、电话）。

关于普通权限和危险权限的区别，以及分别有哪些权限，可以[参考 Google 官方文档](https://developer.android.com/guide/topics/security/permissions.html?hl=zh-cn#normal-dangerous)

## 申请运行时权限

可以调用`ContextCompat.checkSelfPermission()`方法来检查是否有相应的权限，如果有返回 PackageManager.PERMISSION_GRANTED，并且应用可以继续操作。如果没有，返回 PERMISSION_DENIED，且应用必须明确向用户要求权限。

可以调用`shouldShowRequestPermissionRationale()`方法来向用户解释需要某些权限。

可以调用`requestPermissions()`来申请权限。（此时系统会显示标准对话框让用户选择是否授权，我们无法更改）

可以调用`onRequestPermissionsResult() `来了解用户选择的结果。

[参考 Google 官方文档](https://developer.android.com/training/permissions/requesting.html?hl=zh-cn)

---

# 访问其他程序中的数据

# 创建内容提供器
