title: android项目开发：Activity
author: 归零幻想
publishDate: 2020-12-15
editDate: 2020-12-15
tags: [android, 第一行代码, Kotlin, Activity]

<!--config-->

> 仍然是《第一行代码》的读书笔记，可能引用原书的定义和描述，或代码案例。

# Activity

## Activity基本用法

Android讲究设计逻辑与视图分离，一般Activity都会对应一个布局文件（XML文件）。

所有的Activity都要在AndroidManifest中注册才生效。

Activity可以创建菜单。首先在`res/menu`下创建一个xml文件（Android Studio中也提供了可视化编辑的方法）：

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:app="http://schemas.android.com/apk/res-auto"
      xmlns:android="http://schemas.android.com/apk/res/android">

    <item
            android:id="@+id/add_item"
            android:title="Add"/>
    <item
            android:id="@+id/remove_item"
            android:title="Remove"/>
</menu>
```

重写两个方法

```kotlin
    override fun onCreateOptionsMenu(menu: Menu?): Boolean {
    menuInflater.inflate(R.menu.main, menu)
    return true
}

override fun onOptionsItemSelected(item: MenuItem): Boolean {
    when (item.itemId) {
        R.id.add_item -> Toast.makeText(this, "Add a book.", Toast.LENGTH_LONG).show()
        R.id.remove_item -> Toast.makeText(this, "Remove a book.", Toast.LENGTH_LONG).show()
    }
    return true
}
```

![photo_2020-12-10_21-00-32.jpg](https://i.loli.net/2020/12/10/a6wOcTFrHLhS4il.jpg)

用`finish()`方法可以关闭一个Activity。

<!--summary-->

## 在Activity之间跳转

### 显式Intent

```kotlin
jumpButton.setOnClickListener {
    startActivity(Intent(this, SecondActivity::class.java))
}
```

### 隐式Intent

首先在`AndroidManifest.xml`中注册

```xml

<activity android:name=".SecondActivity">
    <intent-filter>
        <action android:name="com.example.myapplication.ACTION_START"/>
        <category android:name="android.intent.category.DEFAULT"/>
    </intent-filter>
</activity>
```

修改启动Activity的代码：

```kotlin
startActivity(Intent("com.example.myapplication.ACTION_START"))
```

action是动作，category是一些附加信息。只有action和category完全匹配才能正确启动Activity。

接下来我们修改上述代码验证这一点。

```xml

<activity android:name=".SecondActivity">
    <intent-filter>
        <action android:name="com.example.myapplication.ACTION_START"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="com.example.myapplication.MY_CATEGORY"/>
    </intent-filter>
</activity>
```

```kotlin
val myIntent = Intent("com.example.myapplication.ACTION_START")
myIntent.addCategory("com.example.myapplication.MY_CATEGORY")
startActivity(myIntent)
```

#### 启动系统其他Activity

可以通过这样的代码启动系统浏览器：

```kotlin
val intent = Intent(Intent.ACTION_VIEW)
intent.data = Uri.parse("https://ntutn.top")
startActivity(intent)
```

`Intent.ACTION_VIEW`实际上是`android.intent.action.VIEW`，如果我们注册下这个action也能冒充系统浏览器。

```xml

<intent-filter>
    <action android:name="android.intent.action.VIEW"/>
    <category android:name="android.intent.category.DEFAULT"/>
    <data android:scheme="https"/>
</intent-filter>
```

![photo_2020-12-10_23-21-57.jpg](https://i.loli.net/2020/12/10/K7pyaXDeSVzqc8Y.jpg)

### 向下一个Activity传递数据

通过Intent的putExtra方法。在第一个Activity中：

```kotlin
            intent.putExtra("extra_data", "I'm IF. IF 3279. ")
```

在第二个Activity中：

```kotlin
        Toast.makeText(this, intent.getStringExtra("extra_data"), Toast.LENGTH_LONG).show()
```

### 返回数据给上一个Activity

首先看看书上的方法。 第一个Activity中：

```kotlin
val intent = Intent(this, SecondActivity::class.java)
startActivityForResult(intent, REQ_1)
```

然后重写一个方法接收结果

```kotlin
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    super.onActivityResult(requestCode, resultCode, data)
    when (requestCode) {
        REQ_1 -> if (resultCode == RESULT_OK) {
            Toast.makeText(this, "点击按钮返回数据：${data?.getStringExtra("data_return")}。", Toast.LENGTH_LONG).show()
        }
    }
}
```

用这样的方法在第二个Activity中返回数据

```kotlin

intent.putExtra("data_return", "Hello FirstActivity")
setResult(RESULT_OK, intent)
finish()
```

不过不得不说安卓迭代是真的快啊，现在AS已经提示我这种方法过时了。于是去找找现在最新的技术是啥。

```kotlin

private val testActivityResult =
    registerForActivityResult(ActivityResultContracts.StartActivityForResult()) {
        if (it.resultCode == Activity.RESULT_OK) {
            Toast.makeText(
                this,
                "点击按钮返回数据：${it.data?.getStringExtra("data_return")}。",
                Toast.LENGTH_LONG
            ).show()
        }
    }


//在需要启动另一个Activity的地方
testActivityResult.launch(intent)
```

## Activity的生命周期

### 返回栈

Activity可以层叠，每当启动一个Activity它将入栈，位于返回栈的栈顶。而如果销毁Activity，它将出栈，下一个栈顶的Activity则被显示给用户。

### Activity的状态

|状态|说明|回收策略|
|---|---|---|
|运行状态|Activity处于返回栈的栈顶|最不倾向于被回收|
|暂停状态|不处于栈顶，但仍然可见[^1]|只有在内存极低的情况下才会回收|
|停止状态|不处于栈顶，且完全不可见|可能被回收|
|销毁状态|从返回栈移除后|最倾向于被回收|

[^1]:Activity不一定会占满整个屏幕，所以下方没有被激活但仍然可见。很多应用用一像素Activity保活就是利用了这一点。

### Activity的生存期

![lifecycle.jpg](https://i.loli.net/2020/12/14/dNqfoY4XOaFC2Es.jpg)

|方法|说明|
|---|---|
|`onCreate()`|Activity**第一次**被创建。应完成初始化操作，创建布局，绑定事件|
|`onStart()`|Activity由不可见变为可见。|
|`onResume()`|Activity准备好和用户交互。此时Activity处于**运行状态**|
|`onPause()`|系统准备启动或恢复另一个Activity时。应释放资源，保存关键数据。|
|`onStop()`|Activity完全不可用时调用。|
|`onDestory()`|Activity被销毁前被调用。|
|`onRestart()`|停止状态变为运行状态前被调用。|

- 完整生存期 `onCreate()`——`onDestory()`
- 可见生存期 `onStart()`——`onStop()`
- 前台生存期 `onResume()`——`onPause()`

## Activity的启动模式

在`AndroidManifest.xml`中指定activity标签的`android:launchMode`属性。

### standard

默认模式。总是创建一个新实例。

### singleTop

若目标Activity已经在栈顶，则直接启动。

### singleTask

若目标Activity已经在栈中，则将其上Activity出栈并启动。

### singleInstance

启用一个新的返回栈管理Activity，一般用于多个程序共享Activity。

## Activity最佳实践

### 知道当前在哪个Activity

准备一个`BaseActivity`，让其他所有Activity继承自它。

```kotlin
package top.ntutn.activcitylifecycletest

import android.os.Bundle
import android.util.Log
import androidx.appcompat.app.AppCompatActivity

open class BaseActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        Log.d(TAG, "${javaClass.simpleName} onCreate.")
    }

    companion object {
        private const val TAG = "BaseActivity"
    }
}
```

### 随时随地退出程序

可以准备一个类将当前所有Activity管理起来。

```kotlin
package top.ntutn.activcitylifecycletest

import android.app.Activity

object ActivityHelper {
    private val activities = arrayListOf<Activity>()

    /**
     * onCreate时
     */
    fun addActivity(activity: Activity) {
        activities.add(activity)
    }

    /**
     * onDestory时
     */
    fun removeActivity(activity: Activity) {
        activities.remove(activity)
    }

    fun finishAll() {
        for (activity in activities) {
            if (!activity.isFinishing) {
                activity.finish()
            }
        }
    }
}
```

然后，在`BaseActivity`中将Activity注册到这里。

```kotlin

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    Log.d(TAG, "${javaClass.simpleName} onCreate.")

    ActivityHelper.addActivity(this)
}

override fun onDestroy() {
    super.onDestroy()
    Log.d(TAG, "${javaClass.simpleName} onDestory")

    ActivityHelper.removeActivity(this)
}

```

这样就可以随时使用`ActivityHelper.finishAll()`来关闭所有Activity了。

### 启动Activity的最佳写法

> 作为一个成熟的Activity，你应该会自己启动自己了。

建议Activity自己实现一个启动Activity的方法，这样外部调用时就能一目了然看出这个Activity需要什么参数。

```kotlin

companion object {
    fun actionStart(context: Context, title: String?, content: String?) {
        val intent = Intent(context, DialogActivity::class.java)
        intent.putExtra("title", title)
        intent.putExtra("content", content)
        context.startActivity(intent)
    }
}
```

这样外部调用只需要

```kotlin

DialogActivity.actionStart(this, "MyDialog", "作为一个成熟的Activity，你应该会自己启动自己了。")
```


