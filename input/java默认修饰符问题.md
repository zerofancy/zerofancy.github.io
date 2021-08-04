title: java默认修饰符问题
author: 归零幻想
publishDate: 2021-08-05
editDate: 2021-08-05
tags: [java, Kotlin, 踩坑, 基础知识]

<!--config-->

## 问题描述

和工具线配合完成某个需求，我这边的改动很少，但一鼓作气搞完后却遇到了奇怪的报错。已知工具线的代码大多是java的，而我这边自然是力推Kotlin。我们的代码参考如下：

### 代码参考

工具线定义了一个接口用于callback

```java
package a;

interface IPublishCallback {
    void onFinish();
}

```

工具线在执行完发布逻辑后无论成功还是失败都会调用我们的callback

```java
package a;

public class PublishUtil {
    public static void publishVideo(String videoName, IPublishCallback callback) {
        Runnable runnable = () -> {
            try {
                System.out.println("[" + videoName + "]开始执行耗时发布操作……");
                Thread.sleep(1000L);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            callback.onFinish();
        };
        Thread thread = new Thread(runnable);
        thread.start();
        try {
            thread.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

我们这边的实现是Kotlin的，就是调用了下工具线的方法

```kt
package b

import a.PublishUtil

class ShareFromSdkImpl {
    fun doShare() {
        println("系统分享功能测试")
        PublishUtil.publishVideo("测试视频") {
            println("发布视频完成回调")
        }
        println("完成系统分享方法")
    }
}
```

主函数

```kt
import b.ShareFromSdkImpl

fun main() {
    ShareFromSdkImpl().doShare()
}
```

## 问题

上述代码有无问题，将会得到什么样的运行结果？

如果有问题，问题出现在哪里，是我的问题还是工具线的问题？

## 答案解析

待补充，先占坑。

<!--summary-->
