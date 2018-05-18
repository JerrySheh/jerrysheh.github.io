---
title: Android笔记（七）连接网络
comments: true
categories: Android
tags: Android
abbrlink: 329b893e
date: 2018-05-04 16:10:43
---

当我们在知乎上面搜索“Android”的时候，可以看到地址栏的链接变化为：

`https://www.zhihu.com/search?type=content&q=Android`

其中，`https://www.zhihu.com/search`是 BASE_URL， 问号`?`后面的是参数。

这里的参数就是 type 为 content， q 为 Android 。

现在，我们要打造一个功能，用户在 EditText 上输入 Android ， 我们的app 可以构造出 `https://www.zhihu.com/search?type=content&q=Android` 这样的 URL 出来。 并对该地址进行 HTTP 访问，然后获取 Response 结果。

以下将以 Github 的 API 为例

<!--more-->

---

# 构建URL

访问 https://api.github.com/ ，可以看到如果我们要搜索 github 仓库，需要构造的链接是

https://api.github.com/search/repositories?q={query}{&page,per_page,sort,order}

假如搜索 hello，以 stars 数目排序，那么构造的链接是

https://api.github.com/search/repositories?q=Hello&sort=stars

创建一个工具类 NetworkUtils.java， 写一个 buildUrl 方法用来生成URL

```java
public class NetworkUtils {

    final static String GITHUB_BASE_URL =
            "https://api.github.com/search/repositories";
    final static String PARAM_QUERY = "q";
    final static String PARAM_SORT = "sort";
    final static String sortBy = "stars";

    public static URL buildUrl(String githubSearchQuery) {
        // TODO (1) Fill in this method to build the proper Github query URL
        Uri builtUri = Uri.parse(GITHUB_BASE_URL).buildUpon()
                .appendQueryParameter(PARAM_QUERY,githubSearchQuery)
                .appendQueryParameter(PARAM_SORT,sortBy)
                .build();

        URL url = null;

        try {
            url = new URL(builtUri.toString());
        } catch (MalformedURLException e) {
            e.printStackTrace();
        }

        return url;
    }
}
```

- 用 `Uri` 来生成 Uri ，再转换为 `URL`

然后在 MainActivity.java 中调用

```java
// 将 EditText 转为 String
String githubQuery = mSearchBoxEditText.getText().toString();

//构造 URL
URL githubSearchUrl = NetworkUtils.buildUrl(githubQuery);
```

---

# 获取 Response 内容

实际上从 Http 响应中获取数据，就是一个输入流。将输入流转换为 String 可以用 Scanner。或者IOUtils。

## 使用 Scanner

在上面的工具类 NetworkUtils.java 中添加一个静态方法，用于获取Http响应的内容

```java
public static String getResponseFromHttpUrl(URL url) throws IOException {
    // 建立一个 HTTP 连接
    HttpURLConnection urlConnection = (HttpURLConnection) url.openConnection();

    try {
        // 从 HTTP 连接中获取输入流
        InputStream in = urlConnection.getInputStream();

        // 以 \\A 为分隔符
        Scanner scanner = new Scanner(in);
        scanner.useDelimiter("\\A");

        boolean hasInput = scanner.hasNext();
        if (hasInput) {
            return scanner.next();
        } else {
            return null;
        }
    } finally {
        urlConnection.disconnect();
    }
}
```

## 扩展:使用 IOUtils 工具类

除了 Scanner 之外，还可以用IOUtils

```java
StringWriter writer = new StringWriter();
IOUtils.copy(inputStream, writer, encoding);
String theString = writer.toString();
```

- 或许还有其他的方法，参考：[read-convert-an-inputstream-to-a-string](https://stackoverflow.com/questions/309424/read-convert-an-inputstream-to-a-string)

---

# 发起 HTTP 请求

在 MainActivity 中， 使用以下语句来发起 HTTP 请求并得到结果

这个方法写在按钮点击事件里，点击时，将发生：

- 根据EditText的内容构建URL
- getResponseFromHttpUrl建立一个HTTP连接并返回String类型的结果
- 在TextView把 response 的结果显示出来

```java
private void makeGithubSearchQuery() {
    // 将 EditText 的内容转为 String
    String githubQuery = mSearchBoxEditText.getText().toString();

    // 用上面的NetworkUtils工具类的buildUrl方法，构建URL
    URL githubSearchUrl = NetworkUtils.buildUrl(githubQuery);

    //将构建好的 URL 显示在 TextView 上，作为直观测试
    mUrlDisplayTextView.setText(githubSearchUrl.toString());

    //用上面的NetworkUtils工具类的getResponseFromHttpUrl方法
    //获取 HTTP response 的内容，并显示在 TextView 上
    String githubSearchResults = null;
    try {
        githubSearchResults = NetworkUtils.getResponseFromHttpUrl(githubSearchUrl);
        mSearchResultsTextView.setText(githubSearchResults);
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

理论上这样做没问题，但如果真的运行，会发现抛出`NetworkOnMainThreadException`，应用程序强退。

原因是对于需要一定时间的任务（比如网络请求），需要开独立的线程来执行，不能在主线程执行。否则会阻塞UI绘制，导致app假死。

所以我们应该把获取 HTTP 请求写在后台线程里。

---

# 后台线程

AsyncTask是在 Android 上的线程之间进行线程和消息传递的抽象类。

使用AsyncTask，可以把网络请求在后台线程运行，然后把结果送到UI线程。

AsyncTask的使用方法是：

第一步：继承AsyncTask抽象类，指定3个泛型参数：
- Params:执行AsyncTask时传入的参数
- Progress：后台任务传给前台的进度条单位（如果不需要为Void）
- Result：后台任务执行完毕后返回给前台的返回值类型

第二步：重写AsyncTask类的以下（部分）方法
- `onPreExcute()`：前台执行。在后台任务开始前调用。
- `doInBackground(Params ...)`：后台执行。后台线程运行的具体内容。
- `onProgressUpdate(Progress ...)`： 前台执行。当后台线程调用`publicProgress(Progress ...)`后，在前台中该方法随即被调用。**用于对UI进行操作（比如改变进度百分比数字）**
- `onPostExecute(Result)`：前台执行。后台return的时候该方法被调用，返回的结果就是Result参数

![asyncTask](../../../../images/Learn_Android/asyncTask.png)

具体例子

- 创建一个类（可以是内部类）继承AsyncTask，泛型参数为URL, Void, String
- 重写`doInBackground()`方法，传入一个URL对象，以及它的参数，处理并返回结果集（该方法在后台线程运行）
- 重写`onPostExecute()`方法，传入结果集（该方法在主线程运行），将结果集显示在 TextView 上面
-

```java
public class GithubQueryTask extends AsyncTask<URL, Void, String>{

    @Override
    protected String doInBackground(URL... params) {
        URL searchUrl = params[0];
        String githubSearchResults = null;
        try {
            githubSearchResults = NetworkUtils.getResponseFromHttpUrl(searchUrl);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return githubSearchResults;
    }

    @Override
    protected void onPostExecute(String githubSearchResults) {
        if (githubSearchResults != null && !githubSearchResults.equals("")) {
            mSearchResultsTextView.setText(githubSearchResults);
        }
    }
}
```

## 添加网络访问权限

在 AndroidManifest.xml 中需要添加访问权限

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

否则会报`SecurityException`异常

## 在onCreate()中调用

因为GithubQueryTask是一个继承于AsyncTask的类，使用 new 语句来启动后台

```java
new GithubQueryTask().execute(githubSearchUrl);
```

效果：

![network](../../../../images/Learn_Android/network.jpg)


- 源码：[优达学城](https://github.com/udacity/ud851-Exercises/tree/student/Lesson02-GitHub-Repo-Search)

---

- 扩展：使用 okhttp 框架：http://square.github.io/okhttp/
