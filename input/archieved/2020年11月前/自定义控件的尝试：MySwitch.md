title: 自定义控件的尝试：MySwitch
author: 归零幻想
publishDate: 2020-09-27
editDate: 2020-09-27
tags: [安卓, View, demo, 控件]

<!--config-->

第一次尝试自定义控件，柿子挑软的捏，我决定做一个Switch。

所以首先确定目标是：实现一个Switch，它有三种状态：`ON`、`OFF`和`DISABLED`。

<!--summary-->

## 自定义属性

自定义控件，肯定要自定义属性嘛。首先在`res/values`中建立一个xml文件，如`attrs.xml`，内容如下

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
<declare-styleable name="MySwitch">
    <attr name="state">
        <enum name="DISABLED" value="0" />
        <enum name="OFF" value="1" />
        <enum name="ON" value="2" />
    </attr>
</declare-styleable>
</resources>
```

这里定义了一个枚举类型state，接下来我们去代码中读取这个属性：

```kt
package com.example.myswitch

import android.content.Context
import android.content.res.TypedArray
import android.util.AttributeSet
import android.util.Log
import android.view.View


class MySwitch(context: Context, attributeSet: AttributeSet) : View(context, attributeSet) {
    companion object {
        const val STATE_DISABLED = 0
        const val STATE_OFF = 1
        const val STATE_ON = 2
    }

    var state = STATE_OFF

    init {
        val attrs: TypedArray = context.obtainStyledAttributes(attributeSet, R.styleable.MySwitch)
        state = attrs.getInt(R.styleable.MySwitch_state, state)
        attrs.recycle()
        Log.d(javaClass.simpleName, "读取到state=$state")
    }
}
```

注意读这个自定义属性有一个坑，就是如果你R点不出styleable，那么你可能导错包了。要导入自己项目的R文件而不是`android.R`。

## 绘制三种不同状态的开关

switch还是很好画的，一条横线做背景，按钮就一个圆圈。

```kt
    private val backgroundPaint = Paint()
    private val foregroundPaint = Paint()

    override fun draw(canvas: Canvas?) {
        super.draw(canvas)

        backgroundPaint.strokeWidth = height.toFloat()
        foregroundPaint.strokeWidth = height.toFloat()

        when (state) {
            STATE_DISABLED -> {
                backgroundPaint.color = Color.DKGRAY
                foregroundPaint.color = Color.WHITE
            }
            STATE_OFF -> {
                backgroundPaint.color = Color.GRAY
                foregroundPaint.color = Color.WHITE
            }
            else -> {
                backgroundPaint.color = Color.GREEN
                foregroundPaint.color = Color.WHITE
            }
        }

        canvas?.let {
            drawBackground(it)
            drawForeground(it, state == STATE_ON)
        }
    }

    private fun drawBackground(canvas: Canvas) {
        canvas.drawLine(
            height / 2f,
            height / 2f,
            width.toFloat() - height / 2f,
            height / 2f,
            backgroundPaint
        )
    }

    private fun drawForeground(canvas: Canvas, switchOn: Boolean) {
        canvas.drawCircle(
            if (switchOn) width.toFloat() - height / 2f else height / 2f,
            height / 2f,
            height / 2f,
            foregroundPaint
        )
    }
```

## 点击事件

其实标准说点击事件应该自己重写下`onTouchEvent`，但我这里简单点直接设置了下`onClickListener`

```kt
setOnClickListener {
            state = when (state) {
                STATE_ON -> {
                    STATE_OFF
                }
                STATE_OFF -> {
                    STATE_ON
                }
                else -> return@setOnClickListener
            }
            invalidate()
        }
```
