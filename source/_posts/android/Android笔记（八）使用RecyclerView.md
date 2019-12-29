---
title: Android笔记（八）使用RecyclerView
comments: true
abbrlink: 37cb0c4f
date: 2018-05-06 18:03:21
categories: Android
tags: Android
---

在 [Android笔记（二） Activity和布局](../post/7b1dcae5.html) 中，使用一个 String[] 模拟了一些数据，然后写入到一个 TextView 或者 ScrollView 中。

但这样有两个弊端：

- ScrollView是一次性将内容绘制完毕，如果数据量很大，会导致内存消耗。
- 无法通过点击 String[] 里面的某一个 String 进入详细页面

于是我们引入了 RecyclerView 。想象一下，我们平时刷微博、刷知乎，随着我们不断地向下刷，数据是动态加载出来的。 这就是RecyclerView。 当然，如果在 RecyclerView 里面嵌套 CardView 就能显示很丰富的内容了。

<!--more-->

# RecyclerView原理

RecyclerView 有一个适配器 Adapter

Adapter 用于在必要时将某些数据源与 View 绑定，同时向 RecyclerView 提供新的 View。

那它如何提供呢？ Adapter 是通过一个叫 ViewHolder 的对象来提供。 ViewHolder 包含了那些 View 的 Root View 。并且，ViewHolder 缓存了一些 View， 以降低请求更新的成本。

最后，Layout Manager 会告诉 RecyclerView 如何布局所有得到的这些 View ， 例如，是垂直排列，还是水平、网格之类。

![recyclerView](../../../../images/Learn_Android/recyclerview.png)

知道了以上原理之后，开发步骤就很明朗了：

0. 添加 recyclerView 的依赖
1. 在 layout 中创建一个 RecyclerView
2. 创建单个 item 的layout resource file
3. 创建 Adapter 类，实现内部类AdapterHolder
4. 重写 Adapter 的三个方法
5. 添加 Layout Manager

---

# 添加依赖

在 app/build.gradle 里面的 dependencies 添加依赖

```
dependencies {
    implementation 'com.android.support:recyclerview-v7:27.1.1'
}
```

> 注意: 在新版本的 Gradle 中， compile 命令已经变更为 implementation ，或者 API

---

# Layout

## 添加 recyclerView

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <!-- A RecyclerView with some commonly used attributes -->
    <android.support.v7.widget.RecyclerView
        android:id="@+id/news_recycler_view"
        android:scrollbars="vertical"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>


</FrameLayout>
```

## 为每个子项添加布局

在 res/layout 创建一个新的 layout resource file

number_list_item.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <android.support.v7.widget.CardView xmlns:card_view="http://schemas.android.com/apk/res-auto"
        android:id="@+id/card_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        card_view:cardBackgroundColor="#FFFFFF"
        card_view:cardCornerRadius="8dp"
        card_view:cardUseCompatPadding="true"
        android:layout_gravity="center">

        <RelativeLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical">

            <ImageView
                android:id="@+id/news_photo"
                android:layout_width="match_parent"
                android:layout_height="240dp"
                android:layout_alignParentTop="true"
                android:scaleType="centerCrop" />

            <TextView
                android:id="@+id/news_title"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_below="@id/news_photo"
                android:gravity="center"
                android:maxLines="1"
                android:padding="5dp"
                android:textColor="#ffffff"
                android:textSize="20sp" />
        </RelativeLayout>
    </android.support.v7.widget.CardView>
</LinearLayout>
```

以上布局为一个 CardView 里面有一个 ImageView 和 TextView

---

# 创建 Adapter 类

当 RecyclerView 需要显示内容的时候，它首先会去找 Adapter 问应该显示哪些 items ，然后 RecyclerView 要求 Adapter 创建 ViewHolder 对象。

具体来说，Adapter主要做以下几件事:

- 为每个 RecyclerView 项目创建 ViewHolder 对象。
- 将数据来源的数据与每个项目绑定
- 返回数据来源中的项目数量
- 扩展将显示的每个项目视图

Adapter 类需要我们重写三个方法：

- `onCreateViewHolder()`：ViewHolder将被创建的时候调用，负责从xml中映射并创建View，并返回一个 ViewHolder 对象。
- `onBindViewHolder()`：数据源与View进行绑定的时候调用
- `getItemCount()`：返回计数器表示第几个 item

在写这三个方法前，我们先定义一个内部类作为 ViewHolder：

 **ViewHolder 的作用是将 xml 中的内容映射成 View 对象，它决定如何显示单个item**。

ViewHolder 将在 `onCreateViewHolder()`方法中被实例化。之后，在 `onBindViewHolder()` 方法中填充每个项的数据。

```java
// 自定义 ViewHolder 类
static class NewsViewHolder extends RecyclerView.ViewHolder{
    CardView cardView;
    ImageView newsImage;
    TextView newsTitle;

    // 构造器
    NewsViewHolder(final View itemView){
        super(itemView);
        cardView = itemView.findViewById(R.id.card_view);
        newsImage = itemView.findViewById(R.id.news_photo);
        newsTitle = itemView.findViewById(R.id.news_title);

        // 设置标题背景为半透明
        newsTitle.setBackgroundColor(Color.argb(20, 0, 0, 0));
    }
}
```

然后重写 Adapter 的三个方法

```java
public class NewsAdapter extends RecyclerView.Adapter<NewsAdapter.NewsViewHolder> {

    private List<News> newsList;
    private Context context;

    // 构造器
    NewsAdapter(List<News> newsList, Context context) {
        this.newsList = newsList;
        this.context = context;
    }

    // 自定义 ViewHolder 类
    // 代码在上面
    // 此处省略

    @NonNull
    @Override
    public NewsViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View v = LayoutInflater.from(context).inflate(R.layout.news_item, parent, false);
        return new NewsViewHolder(v);
    }

    @Override
    public void onBindViewHolder(@NonNull NewsViewHolder holder, int position) {
        News oneNews = newsList.get(position);
        holder.newsTitle.setText(oneNews.getTitle());
        Glide.with(context).load("https://jerrysheh.github.io/images/Learn_Android/Android.jpg").into(holder.newsImage);

    }

    @Override
    public int getItemCount() {
        return newsList.size();
    }
}

```

---

# LayoutManager

ViewHolder 决定如何显示单个 item， 而 LayoutManager 则决定如何显示一堆 item。包括以下三种方式：

![recyclerView](../../../../images/Learn_Android/layoutmanager.png)

LayoutManager 同时负责回收不再需要的 view。

MainActivity.java

```java
public class MainActivity extends AppCompatActivity {

    NewsAdapter mAdapter;
    RecyclerView mRecyclerView;
    List<News> newsList;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        newsList = new ArrayList<>();
        String url = "https://jerrysheh.github.io/images/Learn_Android/Android.jpg";
        for (int i = 1; i < 11; i++) {
            newsList.add(new News(String.valueOf(i), url));
        }

        mRecyclerView = findViewById(R.id.news_recycler_view);

        // 实例化一个 LinearLayoutManager
        LinearLayoutManager layoutManager = new LinearLayoutManager(this);

        // 实例化一个 Adapter
        mAdapter = new NewsAdapter(newsList, this);

        mRecyclerView.setLayoutManager(layoutManager);
        mRecyclerView.setHasFixedSize(true);
        mRecyclerView.setAdapter(mAdapter);

    }

}

```

---

# 实现点击事件

> 其实可以在 RecyclerView 里嵌套 CardView， CardView 自带点击事件，就可以不用自己实现了。

## Adapter类添加点击监听接口

在我们的 Adapter 中加入一个内部接口，然后定义一个 ListItemClickListener 点击监听器

```java
final private ListItemClickListener mOnClickListener;

public interface ListItemClickListener {
    void onListItemClick(int clickedItemIndex);
}
```

## 修改Adapter构造函数

修改 Adapter 类的构造函数，添加第二个参数（ListItemClickListener类型的监听器listener）

```java
public GreenAdapter(int numberOfItems, ListItemClickListener listener) {
    mNumberItems = numberOfItems;
    mOnClickListener = listener;
    viewHolderCount = 0;
}
```

## ViewHolder内部类实现点击监听接口

- 用 `implements OnClickListener` 语句实现接口
- 重写点击方法
- ViewHolder的构造函数中调用点击方法


```java
//implements 监听接口
class NumberViewHolder extends RecyclerView.ViewHolder
    implements OnClickListener {

    TextView listItemNumberView;
    TextView viewHolderIndex;

    public NumberViewHolder(View itemView) {
        super(itemView);

        listItemNumberView = (TextView) itemView.findViewById(R.id.tv_item_number);
        viewHolderIndex = (TextView) itemView.findViewById(R.id.tv_view_holder_instance);
        //调用点击方法
        itemView.setOnClickListener(this);
    }

    void bind(int listIndex) {
        listItemNumberView.setText(String.valueOf(listIndex));
    }

    //重写点击方法
    @Override
    public void onClick(View v) {
        int clickedPosition = getAdapterPosition();
        mOnClickListener.onListItemClick(clickedPosition);
    }
}
```

MainActivity.java

- implements GreenAdapter.ListItemClickListener
- 重写按钮监听方法
- 实例化 Adapter 的时候传入 this

```java
public class MainActivity extends AppCompatActivity
        implements GreenAdapter.ListItemClickListener {

    private static final int NUM_LIST_ITEMS = 100;

    private GreenAdapter mAdapter;
    private RecyclerView mNumbersList;
    private Toast mToast;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mNumbersList = (RecyclerView) findViewById(R.id.rv_numbers);

        LinearLayoutManager layoutManager = new LinearLayoutManager(this);
        mNumbersList.setLayoutManager(layoutManager);

        mNumbersList.setHasFixedSize(true);

        // 用this传入监听器
        mAdapter = new GreenAdapter(NUM_LIST_ITEMS, this);
        mNumbersList.setAdapter(mAdapter);
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.main, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        int itemId = item.getItemId();
        switch (itemId) {
            case R.id.action_refresh:
                mAdapter = new GreenAdapter(NUM_LIST_ITEMS, this);
                mNumbersList.setAdapter(mAdapter);
                return true;
        }
        return super.onOptionsItemSelected(item);
    }

    // 重写按钮监听方法
    @Override
    public void onListItemClick(int clickedItemIndex) {
        if (mToast != null) {
            mToast.cancel();
        }
        String toastMessage = "Item #" + clickedItemIndex + " clicked.";
        mToast = Toast.makeText(this, toastMessage, Toast.LENGTH_LONG);

        mToast.show();
    }
```
