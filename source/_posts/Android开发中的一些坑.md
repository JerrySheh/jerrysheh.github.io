---
title: Android开发中的一些坑
comments: true
categories: Android
tags: Android
abbrlink: cc537ae4
date: 2018-03-10 14:06:25
---

# Gradle 加载慢的问题

第一次加载项目很慢一直显示`Building “XXXX” Gradle project info`

解决办法：

打开{your project}/gradle/wrapper/gradle-wrapper.properties

查看distributionUrl中 gradle 版本

去 https://services.gradle.org/distributions/ 下载相应版本的Gradle（官网地址：https://gradle.org/install）

放到以下目录即可

- Linux：`~/.gradle/wrapper/dists`
- Windows：`C:\users\{user name}\.gradle\wrapper\dists`

---

# 运行时权限在部分Android手机上无效

## 问题

按照 Google 文档的开发模型，写的运行时权限模型代码，在一加手机5T上，点击拒绝后没有任何提示，也就是说依然返回了有权限的STATUE_CODE，但是什么都没发生。

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

## 解决办法

很多国产ROM都有这个坑。因此不要用上面的开发模型，推荐开源库：AndPermission

项目地址： https://github.com/yanzhenjie/AndPermission

使用方法可参考：[Android 6.0 运行时权限管理最佳实践](http://blog.csdn.net/yanzhenjie1003/article/details/52503533/)

---

# Android Device Monitor 打不开的问题

![](../../../../images/Learn_Android/monitor_error.png)


查看Log：C:\Users\Jerrysheh\AppData\Local\Android\Sdk\tools\lib\monitor-x86_64\configuration\1520661867795.log

关键报错

```
!ENTRY org.eclipse.osgi 4 0 2018-03-10 14:04:28.224
!MESSAGE Bundle reference:file:org.apache.ant_1.8.3.v201301120609/@4 not found.

!ENTRY org.eclipse.osgi 4 0 2018-03-10 14:04:28.255
!MESSAGE Bundle reference:file:org.apache.jasper.glassfish_2.2.2.v201205150955.jar@4 not found.

!ENTRY org.eclipse.osgi 4 0 2018-03-10 14:04:28.255
!MESSAGE Bundle reference:file:org.apache.lucene.core_2.9.1.v201101211721.jar@4 not found.

!ENTRY org.eclipse.osgi 4 0 2018-03-10 14:04:28.458
!MESSAGE Bundle reference:file:org.eclipse.help.base_3.6.101.v201302041200.jar@4 not found.

!ENTRY org.eclipse.osgi 4 0 2018-03-10 14:04:28.474
!MESSAGE Bundle reference:file:org.eclipse.help.ui_3.5.201.v20130108-092756.jar@4 not found.

!ENTRY org.eclipse.osgi 4 0 2018-03-10 14:04:28.474
!MESSAGE Bundle reference:file:org.eclipse.help.webapp_3.6.101.v20130116-182509.jar@4 not found.

!ENTRY org.eclipse.osgi 4 0 2018-03-10 14:04:28.474
!MESSAGE Bundle reference:file:org.eclipse.jetty.server_8.1.3.v20120522.jar@4 not found.

!ENTRY org.eclipse.osgi 4 0 2018-03-10 14:04:28.521
!MESSAGE Bundle reference:file:org.eclipse.ui.intro_3.4.200.v20120521-2344.jar@4 not found.

java.lang.IllegalStateException: Unable to acquire application service. Ensure that the org.eclipse.core.runtime bundle is resolved and started (see config.ini).
	at org.eclipse.core.runtime.internal.adaptor.EclipseAppLauncher.start(EclipseAppLauncher.java:74)
	at org.eclipse.core.runtime.adaptor.EclipseStarter.run(EclipseStarter.java:353)
	at org.eclipse.core.runtime.adaptor.EclipseStarter.run(EclipseStarter.java:180)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(Unknown Source)
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
	at java.base/java.lang.reflect.Method.invoke(Unknown Source)
	at org.eclipse.equinox.launcher.Main.invokeFramework(Main.java:629)
	at org.eclipse.equinox.launcher.Main.basicRun(Main.java:584)
	at org.eclipse.equinox.launcher.Main.run(Main.java:1438)
	at org.eclipse.equinox.launcher.Main.main(Main.java:1414)
```

## 解决办法

卸载 JRE 9。 （JDK 9 可以不用卸载） ， 然后装 JDK 8 。

然后启动 Android Studio 时，使用管理员权限打开。
