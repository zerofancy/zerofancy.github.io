title: android项目开发：多线程编程
author: 归零幻想
publishDate: 2021-01-06
editDate: 2021-01-06
tags: [android, Kotlin, 第一行代码, Service, 多线程]

<!--config-->

仍然是《第一行代码》的笔记，不过略过了deprated的内容，并探究了下Handler的工作机制。

> 上班了，果然没有那么多大块时间写博客了。

## Handler

主线程不能进行耗时处理，子线程不能访问UI，所以我们需要异步消息处理机制。

### 使用

```kotlin

class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding

    private val handler = object : Handler(Looper.getMainLooper()) {
        override fun handleMessage(msg: Message) {
            when(msg.what){
                MSG_UPDATE_TEXT -> binding.textView.text = "Nice to meet you. "
            }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        initView()
    }

    private fun initView() {
        binding.apply {
            changeTextButton.setOnClickListener {
                thread {
                    val msg = Message()
                    msg.what = MSG_UPDATE_TEXT
                    handler.sendMessage(msg)
                }
            }
        }
    }

    companion object {
        private const val MSG_UPDATE_TEXT = 1
    }
}
```

### 原理

图片有点多，懒得一张张转移了，去我整理的文档看吧：

[安卓Handler异步消息处理机制](https://bytedance.feishu.cn/docs/doccnRaxHFiTJDYLBuYMzpSSJAd)

<!--summary-->

## Service

三个回调：

1. `onCreate()` 在Service创建时调用
2. `onStartCommand()`在Service每次启动时被调用
3. `onDestory()`在Service销毁时调用

通过`startService(intent)`和`stopService(intent)`的方式启动和停止Service。

Binder用于和View绑定通信。Binder和前台Service具体参见下面例子。

```kotlin

package top.ntutn.servicetest

import android.app.NotificationChannel
import android.app.NotificationManager
import android.app.PendingIntent
import android.app.Service
import android.content.Context
import android.content.Intent
import android.graphics.BitmapFactory
import android.os.Binder
import android.os.Build
import android.util.Log
import androidx.core.app.NotificationCompat

class MyService : Service() {
    private val mBinder = DownloadBinder()

    class DownloadBinder : Binder() {
        fun startDownload() {
            Log.d(javaClass.simpleName, "startDownload() executed")
        }

        fun getProcess(): Int {
            Log.d(javaClass.simpleName, "getProcess() executed")
            return 0
        }
    }

    override fun onBind(intent: Intent) = mBinder

    override fun onCreate() {
        super.onCreate()
        Log.d(javaClass.simpleName, "onCreate() executed")
        val manager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(
                "my_service",
                "前台Service通知",
                NotificationManager.IMPORTANCE_DEFAULT
            )
            manager.createNotificationChannel(channel)
        }
        val intent = Intent(this, MainActivity::class.java)
        val pi = PendingIntent.getActivity(this, REQ_MY_SERVICE, intent, 0)
        val notification = NotificationCompat.Builder(this, "my_service")
            .setContentTitle("This is content title")
            .setContentText("This is content text. ")
            .setContentIntent(pi)
            .setSmallIcon(R.drawable.ic_launcher_foreground)
            .setLargeIcon(BitmapFactory.decodeResource(resources,R.drawable.ic_launcher_foreground))
            .build()
        startForeground(1, notification)
    }

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        Log.d(javaClass.simpleName, "onStartCommand() executed")
        return super.onStartCommand(intent, flags, startId)
    }

    override fun onDestroy() {
        super.onDestroy()
        Log.d(javaClass.simpleName, "onDestroy() executed")
    }

    companion object {
        private const val REQ_MY_SERVICE = 0
    }
}
```

```kotlin

package top.ntutn.servicetest

import android.content.ComponentName
import android.content.Context
import android.content.Intent
import android.content.ServiceConnection
import android.os.Bundle
import android.os.IBinder
import androidx.appcompat.app.AppCompatActivity
import top.ntutn.servicetest.databinding.ActivityMainBinding

class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding
    private lateinit var downloadBinder: MyService.DownloadBinder

    private val connection = object : ServiceConnection {
        override fun onServiceConnected(name: ComponentName, service: IBinder) {
            downloadBinder = service as MyService.DownloadBinder
            downloadBinder.startDownload()
            downloadBinder.getProcess()
        }

        override fun onServiceDisconnected(name: ComponentName?) = Unit
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        initView()
    }

    private fun initView() {
        binding.apply {
            startServiceButton.setOnClickListener {
                val intent = Intent(this@MainActivity, MyService::class.java)
                startService(intent)
            }
            stopServiceButton.setOnClickListener {
                val intent = Intent(this@MainActivity, MyService::class.java)
                stopService(intent)
            }
            bindServiceButton.setOnClickListener {
                val intent = Intent(this@MainActivity, MyService::class.java)
                bindService(intent, connection, Context.BIND_AUTO_CREATE)
            }
            unbindServiceButton.setOnClickListener {
                unbindService(connection)
            }
        }
    }
}
```
