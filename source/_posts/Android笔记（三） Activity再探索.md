---
title: Android笔记（三） Activity再探索
comments: true
categories: Android
tags: Android
abbrlink: 986c4cb5
date: 2018-01-29 17:19:39
---

# 在Activity中使用Toast和Menu

## 使用Toast

Toast是一种显示在屏幕下方的提示，我们经常可以看到有时候app会提醒你没有联网，或者再按一次返回键退出应用之类，这些提醒在短时间内消失，不会打扰用户。

![Toast](../../../../images/Learn_Android/Toast.png)

假设我们已经在 xml 里添加了一个 button

然后转到MainActivity.java，重写`onCreate`方法

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    Button buttonOK = (Button) findViewById(R.id.toast_button);
    buttonOK.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View view) {
            Toast.makeText(MainActivity.this, "yeah, you click it", Toast.LENGTH_SHORT).show();
        }
    });
}
```

* 先实例化一个Button类型的变量 buttonOK，用`findViewById`指向button的ID
* 调用buttonOK的`setOnClickListener`，传入一个点击事件监听器
* 重写监听器的`onClick`方法为调用Toast.makeText
* Toast.makeText有3个参数，第一个是活动本身，第二个是提示内容，第三个是Toast的长度
* 最后`.show()` 让Toast显示出来

<!-- more -->

## 使用Menu

在 res 新建文件夹 menu ， 在 menu里面新建 MenuResource file， 命名为 menu.xml，内容如下

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/add_item"
        android:title="Add"
        />
    <item
        android:id="@+id/remove_item"
        android:title="Remove"/>
</menu>
```

这样我们就布局了两个按钮，一个 add ， 一个 remove

然后在 MainActivity.java 里 重写`onCreateOptionsmenu`方法和`onOptionItemSelected`方法，如下

```java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    getMenuInflater().inflate(R.menu.main, menu);
    return true;
}

@Override
public boolean onOptionsItemSelected(MenuItem item) {
    switch (item.getItemId()) {
        case R.id.add_item:
            Toast.makeText(this, "added", Toast.LENGTH_SHORT).show();
            break;
        case R.id.remove_item:
            Toast.makeText(this,"removed",Toast.LENGTH_SHORT).show();
            break;
        default:
            Toast.makeText(this,"nothing", Toast.LENGTH_SHORT).show();

    }
    return true;
}
```

`onCreateOptionsmenu`方法根据R.menu.main找到我们的布局

`onOptionsItemSelected`方法根据id定义了每个按键按下后的动作

---

# 使用Intent在活动中穿梭

Intent 是 Android 程序中各组件之间进行交互的一种重要方式，它不仅可以指明当前组件想要执行的动作，还可以在不同组件之间传递数据。Intent一般用于启动活动、启动服务以及发送广播等场景。

## 显式Intent

如果我们想在FirstActivity这个活动中打开SecondActivity，我们可以在FirstActivity中的一个按钮点击中调用`StartActivity`，传入intent对象。

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

## 隐式Intent

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

增添了`action`和`category`段，只有 action和category同时匹配才能响应该Intent

* 每个Intent中只能指定一个action，但能指定多个category

修改按钮点击事件， new Intent对象的时候，参数为`com.jerrysheh.hello.ACTION_START`

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

* 如果intent.addCategory指定的Category没有一个活动能够匹配，那么程序会抛出异常。

---

# Intent的更多用法

## 调用浏览器和拨号

new 一个 Intnet对象后， 用 intent.setData 方法可以调用其他程序

比如调用浏览器打开 github

```java
  Intent intent = new Intent("com.jerrysheh.hello.ACTION_START");
  intent.setData(Uri.parse("http://www.github.com"));
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
        <data android:schme="http" />
        <data android:host="www.zhihu.com" />
    </intent-filter>
</activity>
```
这样,你的app可以响应知乎网站的浏览器调用

---

# 传递数据

## 向下一个活动传递数据

可以用 intent 的 putExtra 方法向下一个活动传递数据。核心思想是，把数据存在String里，通过intent参数传递给下一个活动，下一个活动启动后从intent取出。

存放 (MainActivity.java)

```java
  String data = "this is data"
  Intent intent = new Intent("com.jerrysheh.hello.ACTION_START");
  intent.putExtra("Extra_data", data);
  startActivity(intent);
```

取出 (SecondActivity.java)

```java
Intent intent = getIntent();
String data = intent.getStringExtra("extra_data");
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

# Activity的生命周期

todo
