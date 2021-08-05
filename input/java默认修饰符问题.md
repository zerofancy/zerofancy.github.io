title: java默认修饰符问题
author: 归零幻想
publishDate: 2021-08-05
editDate: 2021-08-06
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

上述代码在执行后输出如下：

```plain
系统分享功能测试
Exception in thread "main" java.lang.NoClassDefFoundError: a/IPublishCallback
	at b.ShareFromSdkImpl.doShare(ShareFromSdkImpl.kt:8)
	at MainKt.main(main.kt:4)
	at MainKt.main(main.kt)
Caused by: java.lang.ClassNotFoundException: a.IPublishCallback
	at java.base/jdk.internal.loader.BuiltinClassLoader.loadClass(BuiltinClassLoader.java:581)
	at java.base/jdk.internal.loader.ClassLoaders$AppClassLoader.loadClass(ClassLoaders.java:178)
	at java.base/java.lang.ClassLoader.loadClass(ClassLoader.java:522)
	... 3 more

Process finished with exit code 1

```

在安卓中略有不同，异常类型为

java.lang.IllegalAccessError: Interface a.IPublishCallback implemented by class com.ss.android.ugc.aweme.plugin.xground.player……

## 排查

注意报错中提到的`a/IPublishCallback`，是我们前面定义的回调接口。

我们将lambda改为匿名内部类的写法，这才发现确实是找不到。

![2021-08-06_01-33.png](https://i.loli.net/2021/08/06/nrZt3QkqDLojcf5.png)

在使用Lambda形式实现回调时这个错误没有被编译器检查出来，运行时才报出来。

<!--summary-->

那么它为什么会找不到呢？

## 类可访问性修饰符

Java中，可以使用访问控制符来保护对类、变量、方法和构造方法的访问。Java 支持 4 种不同的访问权限。

|级别|当前类|同一包中的其他类|同一包中的子孙类|不同包中的子孙类|不同包中的其他类|
|---|---|---|---|---|---|
|public|是|是|是|是|是|
|protected（不能用于修饰类）|是|是|是|可访问继承来的方法，不能访问基类实例的protected方法|否|
|package-private（不需要修饰符）|是|是|是|否|否|
|private|是|否|否|否|否|

类、接口的默认级别为package-private，类中的方法和属性默认是package-private，接口中的为`public static final`。

在这里我们就可以看出来了，前面`a.IPublishCallback`是package-private的，在b包下的`ShareFromSdkImpl`就访问不到它。因为写Kotlin多了（默认public），我一时间没有很快认识到这一点。

Kotlin中的修饰符有四个级别：

1. private 意味着只在这个类内部（包含其所有成员）可见；
2. protected—— 和 private一样 + 在子类中可见。
3. internal —— 能见到类声明的 本模块内 的任何客户端都可见其 internal 成员；
4. public —— 能见到类声明的任何客户端都可见其 public 成员。

基本和java那边是对应的，不过不同的是`internal`级别，标识可以在当前模块中使用。与java那边默认的package-private不同，internal不是根据包名验证的，所以也不存在另外一个模块中的类通过定义在相同包下绕过限制的情况。

另外，Kotlin中的类和方法不写修饰符默认是public final的，要继承需要手动加上open。

## So？

所以问题就是工具线那边定义的时候忘记将回调接口设置成public了，我们与他们的类不在同一包下访问不到，但Kotlin的语法糖让我们用lambda写回调，很简单但却正好掩盖了这个问题。

排查花了比较多的时间，其实还是基础知识没掌握牢。