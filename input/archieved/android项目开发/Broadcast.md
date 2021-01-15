title: android项目开发：Broadcast
author: 归零幻想
publishDate: 2020-12-22
editDate: 2020-12-22
tags: [android, 第一行代码, Kotlin]

<!--config-->

仍然是《第一行代码》的学习笔记，安卓内置广播机制。

Android中每个应用程序都可以对自己感兴趣的广播进行注册，包括来自系统的，和其他应用程序的。

广播分为标准广播和有序广播。

- 标准广播异步执行，所有BroadcastReceiver几乎同时收到广播的消息。
- 有序广播 同步执行，只有前一个Receiver逻辑执行完后才会传递给下一个，且可以将广播截断。

<!--summary-->

## 接收系统广播

BroadcastReceiver的`onReceive()`方法是在主线程调用的，不应执行耗时操作。但开一个新线程操作也是不可靠的[^1]。

[^1]:[为什么不能在BroadcastReceiver中开启子线程](https://blog.csdn.net/lyabc123456/article/details/83104832)

### 监听时间变化（动态注册）

```kotlin

class MainActivity : AppCompatActivity() {
    private val timeReceiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context?, intent: Intent?) {
            text.text = Date(System.currentTimeMillis()).toString()
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val intentFilter = IntentFilter()
        intentFilter.addAction(Intent.ACTION_TIME_TICK)

        registerReceiver(timeReceiver, intentFilter)
    }

    override fun onDestroy() {
        super.onDestroy()
        unregisterReceiver(timeReceiver)
    }
}
```

### 开机启动（静态注册）

```xml

<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="top.ntutn.broadcasttest">

    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.BroadcastTest">
        <receiver
            android:name=".BootCompleteReceiver"
            android:enabled="true"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED" />
            </intent-filter>
        </receiver>

        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

```kotlin

package top.ntutn.broadcasttest

import android.content.BroadcastReceiver
import android.content.Context
import android.content.Intent
import android.widget.Toast

class BootCompleteReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        Toast.makeText(context, "Boot Complete", Toast.LENGTH_LONG).show()
    }
}
```

## 发送自定义广播
### 普通广播

```kotlin

val intent = Intent("top.ntutn.broadcasttest.MY_BROADCAST")
intent.`package` = packageName
sendBroadcast(intent)
```

intent中可以带上一些其他信息。

不指定package默认发送隐式广播，无法被静态注册的Receiver接收。指定package指定接收者。

### 有序广播

通过设置intent-filter的`android:priority`属性来调整各个接收者的优先级。

```kotlin

val intent = Intent("top.ntutn.broadcasttest.MY_BROADCAST")
intent.`package` = packageName
sendOrderedBroadcast(intent, null)
```

第二个参数是个有关权限的。

先接收到的可以用`abortBroadcast()`方法阻止广播继续传播。

## Broadcast和Eventbus

今天学习了这章之后我第一个联想到的是之前学习的EventBus，它们在这里起到的作用相似。便好奇它们的区别，实际项目中的选择。

不过在网上搜到的回答千篇一律，各个都是CV能手，说EventBus更轻量级更灵活，但逻辑复杂时难以理出一个清楚的时间流程什么的。

先这么理解吧，以后工程经验丰富后再对比学习。
