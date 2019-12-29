---
title: Android笔记（六） 运行时权限和内容提供器
comments: true
categories: Android
tags: Android
abbrlink: f85eb43b
date: 2018-03-10 22:34:43
---

# 什么是Content Provider ?

内容提供器（Content Provider）主要用于在不同的应用程序之间实现数据共享。它提供一套机制，允许在数据安全的条件下，让一个程序访问另一个程序的共享数据。比如，读取联系人app的电话号码数据等。

Content Provider可选择只对哪一部分数据进行共享，从而保证隐私。

但是在谈Content Provider之前，先来谈谈 Android 的`运行时权限（Runtime permissions）`。

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

危险权限则需要使用`运行时权限（runtime permssions）`。Android 6.0 引入了这个概念，简单地说，就是在应用程序运行的时候，由用户手动授权是否调用手机的某些数据（比如，麦克风、相机、电话）。

关于普通权限和危险权限的区别，以及分别有哪些权限，可以[参考 Google 官方文档](https://developer.android.com/guide/topics/security/permissions.html?hl=zh-cn#normal-dangerous)

## 申请运行时权限

可以调用`ContextCompat.checkSelfPermission()`方法来检查是否有相应的权限，如果有返回 PackageManager.PERMISSION_GRANTED，并且应用可以继续操作。如果没有，返回 PERMISSION_DENIED，且应用必须明确向用户要求权限。

可以调用`shouldShowRequestPermissionRationale()`方法来向用户解释需要某些权限。如果用户第一次拒绝了权限，第二个需要这个权限的时候，该方法返回true。

可以调用`requestPermissions()`来申请权限。（此时系统会显示标准对话框让用户选择是否授权，我们无法更改）

可以重写`onRequestPermissionsResult() `方法来了解用户选择的结果。

开发模型：

```java
if (ContextCompat.checkSelfPermission(this, Manifest.permission.READ_CONTACTS) != PackageManager.PERMISSION_GRANTED) {
    // 没有权限。
    if (ActivityCompat.shouldShowRequestPermissionRationale(this, Manifest.permission.READ_CONTACTS)) {
            // 用户拒绝过这个权限了，应该提示用户，为什么需要这个权限。
    } else {
        // 申请授权。
        ActivityCompat.requestPermissions(thisActivity, new String[]{Manifest.permission.READ_CONTACTS}, MMM);
    }
}

...

@Override
public void onRequestPermissionsResult(int requestCode, String permissions[], int[] grantResults) {
    switch (requestCode) {
        case MMM: {
            if (grantResults.length > 0
                && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                // 权限被用户同意，可以去放肆了。
            } else {
                // 权限被用户拒绝了，洗洗睡吧。
            }
            return;
        }
    }
}
```

- 上述代码详细参考[ Google 官方文档](https://developer.android.com/training/permissions/requesting.html?hl=zh-cn)



可以看到，上面的逻辑还是比较复杂的。而且，由于OEM厂商会在定制ROM上各种改，导致权限获取不正常，以致出现各种各样的 bug。我本人就因为在一加5T上测试上面的代码时，点击了拒绝权限，但没有任何的 Toast 提示，白白浪费了4个小时的时间研究。后来用 AOSP 测试通过了，才发现是一加改了系统的权限通知框。

解决上述问题，推荐开源库：`AndPermission`

项目地址： https://github.com/yanzhenjie/AndPermission

使用方法可参考：[Android 6.0 运行时权限管理最佳实践](http://blog.csdn.net/yanzhenjie1003/article/details/52503533/)


---

# 访问其他程序中的数据

未完待续

# 创建内容提供器

未完待续
