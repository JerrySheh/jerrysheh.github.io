---
title: Android笔记（五）持久化技术
comments: true
abbrlink: 1d084fbe
date: 2018-03-09 11:30:34
categories: Android
tags: Android
---

# 什么是持久化技术

持久化技术指的是将内存中产生的瞬时数据保存到存储设备中，这样，当手机关机再重启，这些数据不会丢失。Android的持久化技术提供了一套机制，让数据在瞬时状态和持久状态之间进行转换。

Android 主要提供了 3 种数据持久化功能，分别是：

- **文件存储**：顾名思义，用于保存文本或二进制数据等文件


- **SharedPreference存储**：保存相对较小的键值集合


- **数据库存储**：将数据保存到数据库

---

# 文件存储

File 对象适合按照从开始到结束的顺序不跳过地读取或写入大量数据。 例如，它适合于图片文件或通过网络交换的任何内容。

> 早期的Android设备，由于内置存储空间非常有限（2011年买的Samung Galaxy S+只有 8 G 存储空间），因此通常都会外置SD卡。所以，Android的存储分为内部和外部。也有一些设备，虽然不支持SD卡，但依然人为地把存储空间分为外部和内部。一般来说，我们推荐把文件保存在内置存储当中，因为它始终可用，且只有自己的应用才能访问此处保存的文件，当自己的应用被卸载时，这些文件也会被移除。

如果要将文件写入外部存储中，请参考[官方文档](https://developer.android.com/training/basics/data-storage/files.html?hl=zh-cn#WriteInternalStorage)

## 将文件保存在内部存储中

无需任何权限，即可在内部存储中保存文件。

* 可以用 `getFilesDir()` 方法返回表示应用的内部目录的 File 。 用`getCacheDir()`方法返回表示应用临时缓存文件的内部目录的 File。

> 关于写入文件和写入缓存，请参考[官方文档](https://developer.android.com/training/basics/data-storage/files.html?hl=zh-cn#WriteInternalStorage)

### 保存文本示例

首先定义一个 EditText

activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <EditText
        android:id="@+id/edit_text_name_input"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="input your name"
        android:padding="4dp"
        android:textSize="24sp" />

</LinearLayout>
```

然后重写onCreate方法和onDestroy方法

Context类提供了一个 FileOutputStream 类型的 `openFileOutput()`方法，用于输出数据到文件。

MainActivity.java
```java
public class MainActivity extends AppCompatActivity {
    private EditText edit;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // edit 实例
        edit = findViewById(R.id.edit_text_name_input);

    }

    @Override
    protected void onDestroy(){
        super.onDestroy();

        // 获取 edit 的内容
        String inputText = edit.getText().toString();
        save(inputText);
    }

    public void save(String inputText){
        FileOutputStream outputStream;

        try {
            // 输出流
            outputStream = openFileOutput("myName", Context.MODE_PRIVATE);
            outputStream.write(inputText.getBytes());
            outputStream.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

这样当我们退出app，edittext的文本就会被自动保存起来。

### 从保存的文件中读取数据

类似于输出流，Context类也提供了一个 FileinputStream 类型的 `openFileinput()`方法，用于输出数据到文件。

> `openFileinput(filename)`的参数是文件名，一旦指定，系统会自动从 /data/data/<package name>/files/ 目录下加载这个文件，并返回一个 FileinputStream 对象。


## 一个完整的示例

```
public class MainActivity extends AppCompatActivity {

    private EditText dataEditText;
    private String inputText;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        dataEditText = findViewById(R.id.editText);
        inputText = load();

        // TextUtils.isEmpty方法：当 inputText 为 null 或 空 时返回 true
        if (!TextUtils.isEmpty(inputText)){
            dataEditText.setText(inputText);
            dataEditText.setSelection(inputText.length());
            Toast.makeText(this, "恢复数据成功", Toast.LENGTH_SHORT).show();
        }

    }

    // 从文件加载数据
    private String load() {
        StringBuilder content = new StringBuilder();
        String line;

        try (
                FileInputStream fileInputStream = openFileInput("data");
                BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(fileInputStream))
        ){
            while( (line = bufferedReader.readLine()) != null ){
                content.append(line);
            }
        } catch (IOException IOe){
            IOe.printStackTrace();
        }
        return content.toString();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        String inputText = dataEditText.getText().toString();
        saveText(inputText);
    }

    //保存数据到文件
    private void saveText(String inputText) {
        try(
             FileOutputStream outputStream = openFileOutput("data",Context.MODE_PRIVATE);
             BufferedWriter bufferedWriter = new BufferedWriter(new OutputStreamWriter(outputStream))
           ){
            bufferedWriter.write(inputText);
        } catch ( IOException e){
            e.printStackTrace();
        }
    }
}
```

思路：

- 在 `onCreate()`方法中调用 `load()` 来读取文件中的数据
- 在 `onDestroy()`方法中调用 `saveText()` 来保存数据到文件


---

# SharedPreference存储

文件存储还是比较麻烦的，SharedPreference可以用键值对的方式来存储数据。

- 保存数据时，给数据提供一个键
- 读取数据时，根据键找到值

在 Android 中，有三种方法得到 SharedPreference 对象:

- Context 类中的 `getSharePreference(filename, mode) `方法

> 参数1是存储的文件名，目录在`/data/data/<package name>/share_prefs/` ，参数2是模式，默认 MODE_PRIVATE

- Activity 类中的 `getPreferences(mode)` 方法

> 只有一个mode参数，因为这个方法会把当前类名作为 filename

- PreferenceManager 类中的 ` static getDefaultSharedPreferences(Context)`方法
