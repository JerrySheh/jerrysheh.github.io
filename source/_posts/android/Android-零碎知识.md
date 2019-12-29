---
title: Android 零碎知识
comments: true
categories: Android
tags: Android
abbrlink: 335b62ef
date: 2018-05-11 00:08:52
---


# 使用WebView

如果要在一个 Activity 上显示图片， 可以用 imageView + ScrollView 组合。但如果是长图片，其实还可以用 WebView

但是 WebView 有内存泄漏的风险，使用时要谨慎。

<!-- more -->

WebView布局

```xml
<ScrollView
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fillViewport="true">
    <WebView
        android:id="@+id/detail_image"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
    </WebView>
</ScrollView>
```

调用

```java
String image = "http://read.html5.qq.com/image?src=share&imageUrl=http://s.cimg.163.com/i/abco1.heibaimanhua.com/wp-content/uploads/2018/05/20180510_5af3d93ebc915.jpg.0x0.auto.jpg"

detialImage = findViewById(R.id.detail_image);
detialImage.getSettings().setUseWideViewPort(true);
detialImage.loadUrl(image);

// 适应手机屏幕
detialImage.getSettings().setLoadWithOverviewMode(true);
```

---

# 使用 CardView

卡片布局，非常好用。

注意一点，添加下面这一行属性可以设置CardView和Button长按或者点击时的涟漪效果：

```xml
android:foreground="?attr/selectableItemBackgroundBorderless"
```

参考布局

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <android.support.v7.widget.CardView xmlns:card_view="http://schemas.android.com/apk/res-auto"
        android:id="@+id/card_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        card_view:cardBackgroundColor="#FFFFFF"
        card_view:cardCornerRadius="4dp"
        card_view:cardUseCompatPadding="true"
        android:foreground="?attr/selectableItemBackgroundBorderless"
        android:layout_gravity="center">

        <RelativeLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical">

            <TextView
                android:id="@+id/news_title"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:padding="5dp"
                android:textSize="20sp"
                android:lines="3"
                android:text="ss"/>

            <Button
                android:id="@+id/button_link"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:background="#00000000"
                android:layout_below="@id/news_title"
                android:textColor="?attr/colorAccent"
                android:text="@string/btn_link"
                android:layout_toStartOf="@id/button_share"
                android:foreground="?attr/selectableItemBackgroundBorderless"
                />

            <Button
                android:id="@+id/button_share"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:background="#00000000"
                android:text="@string/btn_share"
                android:textColor="?attr/colorAccent"
                android:layout_below="@id/news_title"
                android:layout_alignParentEnd="true"
                android:foreground="?attr/selectableItemBackgroundBorderless"
                />

        </RelativeLayout>

    </android.support.v7.widget.CardView>

</LinearLayout>
```

---

# 分享功能

```java
String shareContent = context.getString(R.string.share_content) + oneComic.getLink();
Intent intent = new Intent(Intent.ACTION_SEND);
intent.setType("text/plain");
intent.putExtra(Intent.EXTRA_SUBJECT, "share");
intent.putExtra(Intent.EXTRA_TEXT, shareContent);
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
context.startActivity(Intent.createChooser(intent,"分享到："));
```

---

# 在浏览器中打开

```java
Intent intent = new Intent(Intent.ACTION_VIEW);
Uri content_url = Uri.parse(oneComic.getLink());
intent.setData(content_url);
context.startActivity(intent);
```

---

# 隐藏状态栏和标题栏

AndroidManifest.xml

在对应的 activity 下添加：

```xml
<activity
  android:name=".DetailActivity"
  android:theme="@android:style/Theme.NoTitleBar.Fullscreen">
</activity>
```

如果 activity 是继承 AppCompatActivity 的，会导致报错，

可以修改为：

```xml
<activity
  android:name=".DetailActivity"
  android:theme="@style/Theme.AppCompat.Light.NoActionBar">
</activity>
```

或者把 activity 改为继承 Activity

---

# 使用 Glide 加载图片

简单用法

```java
Glide.with(context).load(imageUrl).into(mImageView);
```

如果需要获取图片属性

```java
String imageUrl = imageList.get(position);


Glide.with(context).load(imageUrl).into(new SimpleTarget<Drawable>() {
    @Override
    public void onResourceReady(@NonNull Drawable resource, @Nullable Transition<? super Drawable> transition) {

        // 获取到图片的高
        int height = resource.getIntrinsicHeight();

        // do more things

        // 把图片显示到 detialImageView 里面
        holder.detialImage.setImageDrawable(resource);
    }
});
}
```

如果需要高级功能(placeholder、firCenter之类，具体见官方文档)，需要写一个类继承AppGlideModule

```java
import com.bumptech.glide.annotation.GlideModule;
import com.bumptech.glide.module.AppGlideModule;

@GlideModule
public final class MyAppGlideModule extends AppGlideModule {
    // 无需写任何代码
}
```

然后 rebuild project

然后用 GlideApp

```java
GlideApp.with(fragment)
   .load(myUrl)
   .placeholder(placeholder)
   .fitCenter()
   .into(imageView);
```

- 项目地址：https://github.com/bumptech/glide
- 项目文档：https://bumptech.github.io/glide/doc/getting-started.html
