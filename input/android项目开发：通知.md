title: android项目开发：通知
author: 归零幻想
publishDate: 2021-01-03
editDate: 2021-01-03
tags: [Android, Kotlin, 第一行代码, 通知]

<!--config-->

仍然是第一行代码的笔记，这篇是有关通知的，最基本的用法。

## 通知的相关知识

通知是什么不再赘述，这里只记录些重要但没接触的概念。

通知渠道在8.0（O）引入。要求APP将通知分类，通过不同*渠道*进行分发，用户可以选择性禁用某个渠道的通知，或者调整优先等级。

通知可以有不同的重要等级，有四种：`IMPORTANCE_HIGH`、`IMPORTANCE_DEFAULT`、`IMPORTANCE_LOW`、`IMPORTANCE_MIN`。根据重要等级不同，通知可能有不同的展现策略，比如在前台提示甚至播放声音。

在通知渠道创建时通知的重要等级也就确定了，之后不能再被APP修改。

<!--summary-->

## 通知dmeo

```kotlin

package top.ntutn.notificationtest

import android.app.NotificationChannel
import android.app.NotificationManager
import android.app.PendingIntent
import android.content.Context
import android.content.Intent
import android.graphics.BitmapFactory
import android.os.Build
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import androidx.core.app.NotificationCompat
import top.ntutn.notificationtest.databinding.ActivityMainBinding

class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        initView()
    }

    private fun initView(){
        val manager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
        if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.O){
            // 通知渠道
            val channel = NotificationChannel("normal","Normal",NotificationManager.IMPORTANCE_DEFAULT)
            manager.createNotificationChannel(channel)
        }
        binding.sendNotice.setOnClickListener {
            val intent = Intent(this,NotificationActivity::class.java)
            val pi = PendingIntent.getActivity(this,REQUEST_NOTIFICATION_ACTIVITY,intent,0)
            val notification = NotificationCompat.Builder(this,"normal")
                    .setContentTitle("This is content title")
                    .setContentText("This is content text. ")
                    .setSmallIcon(R.drawable.ic_launcher_foreground)
                    .setLargeIcon(BitmapFactory.decodeResource(resources,R.drawable.ic_launcher_foreground))
                    .setContentIntent(pi)
                    .setAutoCancel(true)
                    .build()
            manager.notify(CHANNEL,notification)
        }
    }

    companion object {
        private const val CHANNEL = 1
        private const val REQUEST_NOTIFICATION_ACTIVITY = 1
    }
}
```
