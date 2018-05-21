---
title: Android笔记（九）使用ViewPager
comments: true
categories: Android
tags: Android
abbrlink: c60f0455
date: 2018-05-21 18:23:55
---

使用 RecyclerView 可以实现动态加载数据。但如果我们有很多个页面，需要通过左右滑动来切换，就可以使用 ViewPager。

需要实现的效果：

![](../../../../images/Learn_Android/ViewPager.gif)

<!--more-->

---

# Fragment

Fragment，可以称为碎片，或者片段。可以理解成就是小的 Activity， 我们可以在 Fragment 里编写布局，然后在一个 Activity 里面使用多个 Fragment，构成比较复杂的应用界面。

Fragment 同样有自己的生命周期。

## 创建一个 Fragment 布局

比如，如下布局是一个 recyclerView +  SwipeRefreshLayout (下拉刷新)

recycler_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:orientation="vertical"
    android:layout_height="wrap_content">


    <android.support.v4.widget.SwipeRefreshLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:id="@+id/swipeLayout" >

    <android.support.v7.widget.RecyclerView
        android:id="@+id/news_recycler_view"
        android:scrollbars="vertical"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    </android.support.v4.widget.SwipeRefreshLayout>

</RelativeLayout>
```

## 继承 Fragment 类

创建一个java类，继承 android.support.v4.app.Fragment，重写`onCreate()`和`onCreateView()`方法。

- `onCreate()`是在该Fragment被实例化的时候，比如`gsmhFragment = new MHFragment()`的时候调用。
- `onCreateView()`是创建该fragment对应的视图。在`onCreateView()`里，用inflater.inflate将上面的 xml 映射成 View

```java
public class MHFragment extends android.support.v4.app.Fragment {
    // 按需定义一些变量

    // 重写onCreate
    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        context = this.getActivity();
    }

    //重写 onCreateView
    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.recycler_view, container, false);
        recyclerView = view.findViewById(R.id.news_recycler_view);
        mSwipeRefreshLayout = view.findViewById(R.id.swipeLayout);

        // 其他逻辑，比如mSwipeRefreshLayout的监听
        return view;
    }
}
```

### 扩展：如果Activity需要传参数给Fragment

可以在MHFragment类里用静态工厂方法：

```java
public static MHFragment newInstance(String Comicstype)
{
    MHFragment fragment = new MHFragment();
    Bundle bundle = new Bundle();
    bundle.putString("Comicstype", Comicstype);
    fragment.setArguments(bundle);
    return fragment ;
}
```

然后在 Activity 里调用

```java
// Activity类中
MHFragment gsmhFragment;

// onCreate方法中
gsmhFragment = MHFragment.newInstance(ComicTypeEnum.GSMH.getID());
```

---

# 使用 ViewPager

ViewPager的使用也不难，先在Activity里布局 ViewPager，然后创建一个适配器，最后在 Activity 里设置。

## Activity布局

将 ViewPager 放在 LinearLayout 里面

```xml
<android.support.v4.view.ViewPager
    android:id="@+id/vp_content"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

</android.support.v4.view.ViewPager>
```

## 继承 FragmentPagerAdapter

自定义个一个`MyViewPagerAdapter`类，继承`FragmentPagerAdapter`类。在构造方法中传入参数（一个 MHFragment 类型的 List 和一个 String 数组用作标题）

> 之所以要标题，是因为要配合下面的 TabLayout 使用

重写`getItem()`、`getCount()`、`getPageTitle()`方法

```java
public class MyViewPagerAdapter extends FragmentPagerAdapter {
    private List<MHFragment> fragments;
    private String[] titleList;

    public MyViewPagerAdapter(FragmentManager fm, List<MHFragment> fragments, String[] titleList) {
        super(fm);
        this.fragments = fragments;
        this.titleList = titleList;
    }

    @Override
    public Fragment getItem(int arg0) {
        return fragments.get(arg0);
    }

    @Override
    public int getCount() {
        return fragments.size();
    }

    @Nullable
    @Override
    public CharSequence getPageTitle(int position) {
        //return titleList.get(position);
        return  titleList[position];
    }
}
```

## 在Activity中

首先要有一个 List， 里面放了几个 Fragment 实例

```java
MHFragment gxmhFragment ,gsmhFragment, qqmhFragment;
private List<MHFragment> fragments = new ArrayList<>();
gsmhFragment = MHFragment.newInstance(ComicTypeEnum.GSMH.getID());
gxmhFragment = MHFragment.newInstance(ComicTypeEnum.GXMH.getID());
qqmhFragment = MHFragment.newInstance(ComicTypeEnum.QQMH.getID());
fragments.add(gsmhFragment);
fragments.add(gxmhFragment);
fragments.add(qqmhFragment);
```

然后创建ViewPager和MyViewPagerAdapter实例

```java
// 类中
ViewPager mViewPager;
MyViewPagerAdapter mViewPagerAdapter;

// onCreate方法中

FragmentManager fragmentManager = getSupportFragmentManager();

// 参数1：Manager
// 参数2：Fragments类型的List
// 参数3：标题数组
mViewPagerAdapter = new MyViewPagerAdapter(fragmentManager, fragments, tabTitles);
mViewPager.setAdapter(mViewPagerAdapter);
```

这样就可以了。

---

# 使用 TabLayout

有时候我们滑动的时候，还希望上面有一个类似于 Tab 的标签，可以用 TabLayout

## 布局

在 LinearLayout 里面， ViewPager 之上，添加 TabLayout 的布局代码

```xml
<android.support.design.widget.TabLayout
    android:id="@+id/title_tab"
    android:layout_height="wrap_content"
    android:layout_width="match_parent"
    android:scrollbars="horizontal"
    xmlns:android="http://schemas.android.com/apk/res/android">

    <android.support.design.widget.TabItem
        android:id="@+id/tab_GXMH"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/tab_GXMH"/>

    <android.support.design.widget.TabItem
        android:id="@+id/tab_KBMH"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/tab_KBMH"/>

        <!-- 有几个 TabItem 就写几个 -->

</android.support.design.widget.TabLayout>

<android.support.v4.view.ViewPager
    android:id="@+id/vp_content"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

</android.support.v4.view.ViewPager>
```

## Activity 中

```java
// 类中
TabLayout mTabLayout;

// onCreate()方法中
mTabLayout.setupWithViewPager(mViewPager);
```

搞定。

---
