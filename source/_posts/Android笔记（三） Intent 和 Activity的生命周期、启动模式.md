---
title: Android笔记（三） Intent 和 Activity的生命周期、启动模式
comments: true
categories: Android
tags: Android
abbrlink: 986c4cb5
date: 2018-01-29 17:19:39
---

# Intent

Intent 是一个消息传递对象。它是 Android 程序中各组件之间进行交互的一种重要方式，它不仅可以指明当前组件想要执行的动作，还可以在不同组件之间传递数据。

> 关键词： **指明要执行的动作**，**传递数据**

## Intent 的基本使用场景

- **启动 Activity**：
  Activity 表示应用中的一个屏幕。通过将 Intent 传递给 `startActivity()`，您可以启动新的 Activity 实例。Intent 描述了要启动的 Activity，并携带了任何必要的数据。

  如果希望在 Activity 完成后收到结果，请调用 `startActivityForResult()`。在 Activity 的 `onActivityResult()` 回调中，您的 Activity 将结果作为单独的 Intent 对象接收。

- **启动服务**：
  Service 是一个不使用用户界面而在后台执行操作的组件。通过将 Intent 传递给 `startService()`，您可以启动服务执行一次性操作（例如，下载文件）。Intent 描述了要启动的服务，并携带了任何必要的数据。

  如果服务旨在使用客户端-服务器接口，则通过将 Intent 传递给 `bindService()`，您可以从其他组件绑定到此服务。

- **传递广播**：
  广播是任何应用均可接收的消息。系统将针对系统事件（例如：系统启动或设备开始充电时）传递各种广播。通过将 Intent 传递给 sendBroadcast()、sendOrderedBroadcast() 或 sendStickyBroadcast()，您可以将广播传递给其他应用。

参阅：[官方文档 - Intent 和 Intent 过滤器](https://developer.android.com/guide/components/intents-filters.html?hl=zh-cn)

## 使用显式Intent

显式 Intent 指的是明确地按名称（完全限定类名）指定要启动的组件。比如说，如果我们想在FirstActivity这个活动中打开SecondActivity，我们可以在FirstActivity中的一个按钮点击中调用`StartActivity`，传入intent对象。

```java
Button buttonIntent = (Button) findViewById(R.id.button_intent);
buttonIntent.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        Intent intent = new Intent(MainActivity.this,SecondActivity.class);
        startActivity(intent);
    }
});
```

* 定义一个按钮
* 在按钮点击事件中 new 一个 intent 对象
* 调用 `startActivity`，传入 intent 对象

## 使用隐式Intent

隐式 Intent 不会指定特定的组件，而是声明要执行的常规操作，从而允许其他应用中的组件来处理它。 例如，如需在地图上向用户显示位置，则可以使用隐式 Intent，请求另一具有此功能的应用在地图上显示指定的位置。

创建隐式 Intent 时，Android 系统通过将 Intent 的内容与在设备上其他应用的清单文件中声明的 Intent 过滤器进行比较，从而找到要启动的相应组件。 如果 Intent 与 Intent 过滤器匹配，则系统将启动该组件，并向其传递 Intent 对象。 如果多个 Intent 过滤器兼容，则系统会显示一个对话框，支持用户选取要使用的应用。

在AndroidManifest.xml中，把SecondActivity段修改如下

```xml
<activity android:name=".SecondActivity">
    <intent-filter>
        <action android:name="com.jerrysheh.hello.ACTION_START" />
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="com.jerrysheh.hello.MY_CATEGORY"/>
    </intent-filter>
</activity>
```

在\<intent-filter\>中增添了`action`和`category`段，只有 action 和 category 同时匹配才能响应该Intent。每个Intent中只能指定一个action，但能指定多个category。

* **action**：声明接受的 Intent 操作。
* **category**：声明接受的 Intent 类别。

> 除了`action`和`category`外，还有一个`data`，请参考[官方文档](https://developer.android.com/guide/components/intents-filters.html?hl=zh-cn)


修改按钮点击事件， new Intent对象，因为我们想启动能到响应为`com.jerrysheh.hello.ACTION_START`这个action的活动，因此参数填入`com.jerrysheh.hello.ACTION_START`。

```java
Button buttonIntent = (Button) findViewById(R.id.button_intent);
buttonIntent.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        Intent intent = new Intent("com.jerrysheh.hello.ACTION_START");
        intent.addCategory("com.jerrysheh.hello.MY_CATEGORY");
        startActivity(intent);
    }
});
```

或者，可以这样写,利用`intent.setAction`方法。
```java
Button buttonIntent = (Button) findViewById(R.id.button_intent);
buttonIntent.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        Intent intent = new Intent();
        intent.setAction("com.jerrysheh.hello.ACTION_START");
        intent.addCategory("com.jerrysheh.hello.MY_CATEGORY");
        startActivity(intent);
    }
});
```

如果intent.addCategory指定的Category没有一个活动能够匹配，那么程序会抛出异常。稍作修改，用`resolveActivity()`方法来判断是否有应用能响应。假设没有活动匹配，就不启动`startActivity()`;

```java
Button buttonIntent = (Button) findViewById(R.id.button_intent);
buttonIntent.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        Intent intent = new Intent("com.jerrysheh.hello.ACTION_START");
        intent.addCategory("com.jerrysheh.hello.MY_CATEGORY");

        if (intent.resolveActivity(getPackageManager()) != null) {
            startActivity(intent);
        }

    }
});
```

---

# Intent的更多用法

## 调用浏览器和拨号

new 一个 Intnet 对象后， 用 `intent.setData(uri)` 方法可以调用其他程序

比如调用浏览器打开 github

```java
  Intent intent = new Intent("com.jerrysheh.hello.ACTION_START");
  intent.setData(Uri.parse("https://www.github.com"));
  startActivity(intent);
```

调用系统拨号拨打10010

```java
  Intent intent = new Intent(Intent.ACTION_DIAL);
  intent.setData(Uri.parse("tel:10010"));
  startActivity(intent);
```

可以在 AndroidManifest 的 <intent - filter>标签中配置 <data> 标签， 指定当前活动可以响应什么类型的数据。这样其他app响应这种数据的时候，Android系统会弹出选项，你的app会在可选列表里面

```xml
<activity android:name=".SecondActivity">
    <intent-filter>
        <action android:name="com.jerrysheh.hello.ACTION_START" />
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:schme="https" />
        <data android:host="www.zhihu.com" />
    </intent-filter>
</activity>
```
这样, 你的app可以响应知乎网站的浏览器调用

---

# 使用Intent传递数据

## 向下一个活动传递数据

可以用 intent 的 putExtra 方法向下一个活动传递数据。核心思想是，把数据存在String里，通过intent参数传递给下一个活动，下一个活动启动后从intent取出。

存放 (MainActivity.java)

```java
  String data = "this is data"
  Intent intent = new Intent("com.jerrysheh.hello.ACTION_START");
  intent.putExtra(Intent.EXTRA_TEXT, data);
  startActivity(intent);
```

取出 (SecondActivity.java)

```java
Intent intent = getIntent();
String data = intent.getStringExtra(Intent.EXTRA_TEXT);
```

## 返回数据给上一个活动

Activity中有一个`StartActivityForResult()`方法用于启动一个活动，但期望活动销毁后（通常是按下返回键或调用`finish()`方法）返回一个结果给上一个活动。


MainActivity.java

```java
Intent intent = new Intent(MainActivity.this,SecondActivity.class);
StartActivityForResult(intent, 1);
```

SecondActivity.java

```java
Intent intent = new Intent();
intent.putExtra("data_return", "this is back data");
setResult(RESULT_OK, intent);
finish();
```

当然，返回数据后，会回调MainActivity的`onActivityResult()`方法，因此我们还需要重写这个方法拿到SecondActivity返回来的数据。

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
  switch(requestCode) {
    case 1:
    if (requestCode == RESULT_OK) {
      String returnData = data.getStringExtra("data_return");
      ...
    }
    break;
  }
}
```

## 使用 intent 传递对象

上面的 intent 只能传 String， 如果我们有一个 javabean 对象需要传递，怎么做呢？

首先将实体类（bean）实现  Serializable 接口。**注意:** 如果 bean 里面嵌套了 bean，内部类也要声明为实现 Serializable 接口。

```java
public class bean implements Serializable {
    int a;
    int b;
    String c;
    Heybean d;
    public static class Heybean implements Serializable {
        ...
    }
}
```

传递 activity

```java
Intent intent = new Intent(context, DetailActivity.class);
intent.putExtra("name",detailbean);
context.startActivity(intent);
```

接收 activity

```java
Bean detailBean = (Bean) getIntent().getSerializableExtra("name");
```

---

# Activity的生命周期

Android 用 任务（Task）来管理活动，一个Task就是一组存放在返回栈（Back Stack）里的活动的集合。系统总是会显示处于栈顶的活动给用户。

## Activity的四种状态

* 运行状态
* 暂停状态（弹出式卡片，背景活动就是暂停状态）
* 停止状态
* 销毁状态

## Activity的生存期

![Toast](../../../../images/Learn_Android/activity_lifecycle.png)

* 完整生存期
 活动在`onCraete()`和`onDestroy()`之间经历的，就是一个完整生存期。

* 可见生存期
 活动在`onStart()`和`onStop()`之间经历的，就是一个可见生存期。onStart()方法在活动从不可见变为可见时调用，onStop()反之。

* 前台生存期
 活动在`onResume()`和`onPause()`之间经历的，就是一个前台生存期。onResume()方法在活动准备好和用户交互时调用。当系统准备去启动或恢复另一个活动时，onPause()将当前活动一些消耗CPU的资源释放，同时保存关键数据。

此外，还有一个`onRestart()`，用于活动从停止状态变为运行状态之前调用，也就是活动被重新启动。

> 当系统内存不足时，用户按下back键返回到上一个Activity，有可能上一个Activity已经被系统回收，这时不会执行`onRestart()`，而是执行`onCreate()`。遇到这种情况，如果上一个Activity有数据，那这些数据都丢失了，这是很影响用户体验的。解决办法是调用`onSaveInstantState()`回调方法。具体参见《第一行代码》第二版p62，以及[Activity Google官方文档](https://developer.android.com/guide/components/activities.html?hl=zh-cn#Creating)（推荐）


## Activity的启动模式

Activity有四种启动模式，可以在 AndroidManifest.xml 的 <activity> 标签中修改

```xml
<activity
  android:name=".SecondActivity"
  android:launchMode="singleTop">
    <intent-filter>
        <action android:name="com.jerrysheh.hello.ACTION_START" />
        <category android:name="android.intent.category.DEFAULT"/>
    </intent-filter>
</activity>
```

`android:launchMode`可填以下四种模式

* standard

 标准模式，在MainActivity中启动MainActivity，会重复创建MainActivity的新实例。如创建了3个MainActivity的实例，需要按3次返回键才能完全退出。

* singleTop

  如果MainActivity已经在栈顶，启动MainActivity则不会重复创建新实例。但MainActivity不在栈顶，还是会创建新实例。

* singleTask

 无论MainActivity是否在栈顶，在整个应用程序上下文中只存在一个MainActivity实例。

* singleInstance

  单独创建一个返回栈存放该实例。解决不同应用共享一个返回栈的问题。


  参考：[Google官方文档](https://developer.android.com/guide/components/tasks-and-back-stack.html?hl=zh-cn)
