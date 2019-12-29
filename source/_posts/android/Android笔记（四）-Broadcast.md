---
title: Android笔记（四） Broadcast
comments: true
categories: Android
tags: Android
abbrlink: 9e4d7c93
date: 2018-02-04 16:52:00
---

# 什么是 Broadcast Receiver

广播接收器（Broadcast Receiver）允许你的应用接收来自各处的广播消息，比如开机广播，电池电量变化广播，时间或地区变化广播，以及来自电话、短信和其他app发出的广播消息等等。同样，我们的应用也可以向外发出广播消息。

Android中的广播可以分为以下两种：

* `标准广播（Normal Broadcast）`，一种完全异步执行的广播，当广播发出后，所有的广播接收器几乎同一时刻接收到这条广播信息，没有先后顺序之分。这种广播效率高，但无法截断。

* `有序广播（Ordered Broadcast）`，一种同步执行的广播，当广播发出后，同一时刻只有一个广播接收器收到这条广播，当这个广播接收器的逻辑执行完毕后，广播才会继续传递。这样一来，优先级高的广播接收器可以先收到广播，并且前面的广播接收器可以截断正在传递的广播。

<!-- more -->

---

# 接收系统广播

我们可以对有需要的广播进行注册，这样，当有响应的广播发出时，广播接收器就能收到并处理。

注册广播的方法有动态和静态两种。

## 动态注册

即在逻辑代码中注册（动态注册的一定要记得手动取消注册），动态注册的广播接收器需要在程序运行后才能接收广播

动态注册创建一个广播接收器的方法：
* 新建一个类，继承自BroadcastReceiver
* Override 父类的 onReceive() 方法

实际例子，收到网络状态信息改变广播时，发出Toast
1. 新建一个继承自BroadcastReceiverd的 NetworkChangeReceiver 类
2. Override 父类的 onReceive() 方法，发出Toast
3. 分别定义一个IntentFilter类型和NetworkChangeReceiver类型的变量
4. new 一个 IntentFilter 的实例，该实例的`addAction`方法让intent过滤android.net.conn.CONNECTIVITY_CHANGE广播
5. new 一个 NetworkChangeReceiver 的实例
6. registerReceiver方法，传入networkChangeReceiver和intentFilter

```java
public class MainActivity extends AppCompatActivity {

    private IntentFilter intentFilter;
    private NetworkChangeReceiver networkChangeReceiver;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        intentFilter = new IntentFilter();
        intentFilter.addAction("android.net.conn.CONNECTIVITY_CHANGE");
        networkChangeReceiver = new NetworkChangeReceiver();
        registerReceiver(networkChangeReceiver, intentFilter);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        unregisterReceiver(networkChangeReceiver);
    }

    class NetworkChangeReceiver extends BroadcastReceiver {

        @Override
        public void onReceive(Context context, Intent intent) {
            Toast.makeText(context, "network change", Toast.LENGTH_SHORT).show();
        }
    }
}

```

## 静态注册

 即在AndroidManifest.xml中注册，应用程序没有运行也能接收。

静态注册创建一个广播接收器的方法：
* 在Android Studio中右键package → new → Other → Broadcast Receiver来创建广播接收器，这样Android Studio会自动帮我们在AndroidManfest.xml注册
* 会自动生成@Override的 `onReceive()`方法，在该方法中编写接收逻辑
* 在AndroidManfest.xml中的Receiver标签内再建intent-filiter标签


```xml
<Receiver
  android:name=".BootReceiver"
  android:enable="true"
  android:exported="true">
  <intent-filiter>
    <action android:name="android.intent.action.BOOT_COMPLETED" />
  </intent-filiter>
</Receiver>
```

* `android:exported`用于控制是否接收本程序以外的广播

Android中对敏感的操作必须在AndroidManifest.xml配置文件中声明权限，否则程序会崩溃。

比如，访问系统的`网络状态`和`监听手机开机`，需要在AndroidManifest.xml中注明：

```xml
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
```

> Broadcast Receiver通常扮演打开程序其他组件的角色，比如创建一条状态栏通知，或者启动一个服务等。不要在 `onReceive()`方法中写太多逻辑代码或耗时的操作，Broadcast Receiver中不允许开线程，`onReceive()`方法运行较长时间没有结束时，程序会报错。

---

# 发送广播

## 发送自定义标准广播

可以在按钮点击事件中加入发送自定义广播的逻辑。

* 构建一个 Intent 对象
* 传入要发送的广播的值
* 调用Context的`sendBroadcast(intent)`方法，将广播发送出去

## 发送有序广播

发送有序广播只需要把`sendBroadcast()`方法改成`sendOrderedBroadcast(intent, null)`。

第二个参数是权限相关字符串。

我们可以在Broadcast Receiver的AndroidManifest.xml中，加入

```xml
<intent-filter android:priority="100">
```

把这个Broadcast Receiver的优先级设为100

然后在这个Broadcast Receiver的`onReceive()`方法中，加入`abortBoradcast();`语句来截断这条广播，这样，这个有序广播不会被优先级更低的Broadcast Receiver收到。

---

# 本地广播

有时候，我们只希望我们的广播在自己的应用程序内部可以接收到，不希望被系统和其他app接收，这时候可以用本地广播。

本地广播使用方法：

## 接收

首先我们要定义一个Broadcast Receiver，用来接收我们的本地广播

```java
class LocalReceiver extends BroadcastReceiver {

  @Override
  public void onReceive(Context context, Intent intent) {
    // when recived, do something
  }
}
```

## 发送

首先在`onCreate()`方法中获取一个本地广播的实例

```java
localBroadcastManager = LocalBroadcastManager.getInstance(this);
```

然后可以在按钮点击事件中使用Intent,并注册本地广播监听器

```java
public void onClick(View v) {
  Intent intent = new Intent("com.jerrysheh.hello.LOCAL_BROADCAST");
  localBroadcastManager.sendBroadcast(intnet);
}

intentFiliter = new intentFiliter();
intentFiliter.addAction("com.jerrysheh.hello.LOCAL_BROADCAST");
localReceiver = new localReceiver();
localBroadcastManager.rigisterReceiver(localReceiver, intentFilter);
```

别忘记在`onDestroy()`方法中取消注册

```java
localBroadcastManager.unrigisterReceiver(localReceiver);
```
