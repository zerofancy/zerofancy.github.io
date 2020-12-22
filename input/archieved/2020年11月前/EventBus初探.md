title: EventBus初探
author: 归零幻想
publishDate: 2020-09-08
editDate: 2020-09-08
tags: [android, Kotlin, EventBus, 事件驱动机制]

<!--config-->

抖音项目中大量运用了EventBus[^1]，这两天在做的需求相关的代码也用到了，于是还是要先去研究下这个EventBus是什么。

EventBus，顾名思义，事件总线，是一个安卓上的事件管理平台。用事件驱动的机制来简化事件传递的逻辑。在这层意义上也许可以类比`Qt`的信号槽机制，或者安卓的广播。当你需要把一个事件传递给多个对象，EventBus就变得非常有用。

![EventBus](https://github.com/greenrobot/EventBus/raw/master/EventBus-Publish-Subscribe.png)

*该图片来自EventBus的github仓库*

<!--summary-->

## EventBus的使用

首先当然是添加依赖

```groovy
implementation 'org.greenrobot:eventbus:3.2.0'
```

写个demo展示下：

![photo_2020-09-08_15-58-48.jpg](https://i.loli.net/2020/09/08/CzpLZxPkrJiqaDm.jpg)

![photo_2020-09-08_15-59-12.jpg](https://i.loli.net/2020/09/08/SwM6ljRtPkgdGLT.jpg)

点击按钮后，将两处文本的值修改为文本框中的值。注意：上面的那个红色的文本处于一个fragment中。

为此，我们先准备一个Event：

```kotlin
data class ButtonEvent(val text: String)
```

点击按钮时，发送这个Event：

```kotlin
        button.setOnClickListener {
            EventBus.getDefault().post(ButtonEvent(editTextTextPersonName.text.toString()))
        }
```

然后在需要接收这个请求的地方，比如我们在`MainActivity`的`onCreate()`中注册EventBus：

```kotlin
EventBus.getDefault().register(this)
```

在`onDestory()`中解除

```kotlin
EventBus.getDefault().unregister(this)
```

然后就可以接收那个Event了。

```kotlin
    @Subscribe
    fun onButtonEvent(buttonEvent: ButtonEvent){
        textView.text=buttonEvent.text
    }
```

同样对于Fragment，我们只需要做同样的事情，就可以监听这个Event了。很明显，这对于一个事件有多个处理的场景是很方便的。

## 订阅者的threadMode

安卓应用启动时，系统会创建一个主线程，负责向UI组件分发事件[^2]，所有对UI的操作应该放到主线程中。EventBus是一种重要的跨线程更新UI的方式。[^3]

`@Subscribe`注解有一个参数`threadMode`，有以下取值：

### ThreadMode.POSTING

默认值，在同一线程中调用，开销最小。

### ThreadMode.MAIN

在主线程中调用。

> 如果发送事件的是主线程，则直接调用。

### ThreadMode.MAIN_ORDERED

在主线程中调用，同步调用（排队）。

### ThreadMode.BACKGROUND

在后台线程中调用。

### ThreadMode.ASYNC

在单独线程中调用，用于耗时操作。

对于EventBus更新UI，我同样写了demo：

```kotlin
package com.example.eventbustimerdemo

import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import kotlinx.android.synthetic.main.activity_main.*
import org.greenrobot.eventbus.EventBus
import org.greenrobot.eventbus.Subscribe
import org.greenrobot.eventbus.ThreadMode

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        btnStart.setOnClickListener {
            Thread {
                while (true) {
                    Thread.sleep(1000)
                    EventBus.getDefault().post(TickEvent())
                }
            }.start()
            btnStart.isEnabled = false
        }
        EventBus.getDefault().register(this)
    }

    override fun onDestroy() {
        super.onDestroy()
        EventBus.getDefault().unregister(this)
    }

    @Subscribe(threadMode = ThreadMode.MAIN)//只有在UI线程中才能更新UI
    fun onTimeTick(event: TickEvent) {
        val currentNum = tvTime.text.toString().toInt()
        tvTime.text = (currentNum + 1).toString()
    }
}

class TickEvent

```

## StickyEvent

『粘性事件』[^4]，即在事件被广播后将长时间存在，新的订阅者仍然能收到。在一些场景下还是有用的，比如我们可能在一个详情页进行投票操作，在返回主页后才进行统计。这时粘性事件会带来帮助。

```kotlin
        button.setOnClickListener {
            EventBus.getDefault().postSticky(VoteEvent(true))
            finish()
        }
        button2.setOnClickListener {
            EventBus.getDefault().postSticky(VoteEvent(false))
            finish()
        }
```

```kotlin

    override fun onResume() {
        super.onResume()
        EventBus.getDefault().register(this)
    }

    override fun onPause() {
        super.onPause()
        EventBus.getDefault().unregister(this)
    }

    @Subscribe(threadMode = ThreadMode.MAIN,sticky = true)
    fun onVoteEvent(event: VoteEvent) {
        textView.text = if (event.result) "你投票赞同" else "你投票反对"
        EventBus.getDefault().removeStickyEvent(event)
    }
```



[^1]:[EventBus](https://github.com/greenrobot/EventBus)

[^2]:[Android UI线程和非UI线程 ](https://www.jianshu.com/p/0f5ed338feeb)
[^3]:[Android开发中更新UI的几种常用方式 ](https://blog.csdn.net/haoyuegongzi/article/details/78406342)
[^4]:[深入理解EventBus - ThreadMode、Sticky Event等](https://blog.csdn.net/IO_Field/article/details/52185717)
