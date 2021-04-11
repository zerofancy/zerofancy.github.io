title: ClassLoader双亲委托机制探究
author: 归零幻想
publishDate: 2021-04-11
editDate: 2021-04-11
tags: [Java, 安卓, 反射]

<!--config-->

最近在研究抖音进入热点内流的耗时问题，种种线索指向了类加载耗时上。为此，我研究了Java类加载的双亲委托机制，并尝试给出了优化建议。

## 双亲委托机制

双亲委托机制中最重要的是loadClass方法，让我们看看它是怎么实现的。

```java
    protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name); // 已加载过直接返回
            if (c == null) {
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false); //尝试让parent加载
                    } else {
                        c = findBootstrapClassOrNull(name); // bootstrap class loader是否加载过
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    c = findClass(name); // 自己加载（找不到会抛出异常）
                }
            }
            return c;
    }
```

findClass就是根据类名去加载具体类的方法，则整个加载机制就很清楚了。

1. 首先部分类会被BootstrapClassLoader加载，这部分是native实现。
2. 会先尝试让parent加载
3. parent找不到时才会自己加载

如果我们能创建一个ClassLoader，将它插入到PathClassLoader和BootstrapClassLoader之间，那么所有被PathClassLoader加载的类就都可以记录下来了。

<!--summary-->

## PathClassLoaderProxy实现

> Talk is cheap, show me the code.

```kotlin

class ClassLoaderProxy : ClassLoader() {
    override fun findClass(name: String?): Class<*> {
        if (name != null && recording) loadedClass.add(name)
        throw ClassNotFoundException()
    }

    companion object {
        private val loadedClass = mutableSetOf<String>()
        private var recording = false
        private var hooked = false

        @JvmStatic
        fun inject(context: Context): Boolean {
            Log.w(ClassLoaderWrapper2::class.java.simpleName, "Hook开始")
            try {
                val originClassLoader = context.classLoader
                Log.d("ClassLoader类型", originClassLoader.javaClass.simpleName)
                val delegateClassLoader = ClassLoaderWrapper2()
                setParent(delegateClassLoader, originClassLoader.parent)
                setParent(originClassLoader, delegateClassLoader)
                Log.w(ClassLoaderWrapper2::class.java.simpleName, "Hook成功")
                hooked = true
                return true
            } catch (e: Throwable) {
            }
            Log.e(ClassLoaderWrapper2::class.java.simpleName, "Hook没有成功")
            return false
        }

        private fun setParent(classLoader: ClassLoader, newParent: ClassLoader) {
            val parentField = ClassLoader::class.java.getDeclaredField("parent")
            parentField.isAccessible = true
            parentField.set(classLoader, newParent)
        }

        @Throws(IllegalStateException::class)
        private fun checkHookState() {
            if (!hooked) {
                throw IllegalStateException("hookPackageClassLoader没有执行成功！")
            }
        }

        @Throws(IllegalStateException::class, RuntimeException::class)
        fun startRecord() {
            checkHookState()
            synchronized(recording) {
                if (recording) {
                    throw RuntimeException("已经存在正在监听的任务！")
                }
                loadedClass.clear()
                recording = true
            }
        }

        @Throws(IllegalStateException::class, RuntimeException::class)
        fun endRecord(block: (Set<String>) -> Unit) {
            checkHookState()
            synchronized(recording) {
                if (!recording) {
                    throw RuntimeException("当前没有在监听！")
                }
                recording = false
                block.invoke(loadedClass)
            }
        }
    }
}
```

因为我们并不需要做实际的加载，只要抛出异常，加载还是交给我们儿子PathClassLoader就可以了。

## 记录加载时间

```kotlin
package top.ntutn.hookpackageclassloader

import android.content.Context
import android.util.Log

class ClassLoaderWrapper(private val realClassLoader: ClassLoader) : ClassLoader() {

    private val findClassMethod by lazy {
        ClassLoader::class.java.getDeclaredMethod("findClass", String::class.java).apply {
            isAccessible = true
        }
    }

    override fun loadClass(name: String?, resolve: Boolean): Class<*> {
        val startTime = System.currentTimeMillis()

        // First, check if the class has already been loaded
        var c: Class<*>? = null
        try {
            c = parent.loadClass(name)
        } catch (e: ClassNotFoundException) {
            // ClassNotFoundException thrown if class not found
            // from the non-null parent class loader
        }

        if (c == null) {
            c = findClassMethod.invoke(realClassLoader, name) as Class<*>?
        }

        Log.e("classloader", "load class: $name")
        loadedClasses.add(name.toString() to System.currentTimeMillis() - startTime)

        return c!!
    }

    companion object {
        private val loadedClasses = mutableSetOf<Pair<String, Long>>()
        private var recording = false
        private var hooked = false
        private val preloadClasses = setOf(
            "kotlin.collections.CollectionsKt",
            "kotlin.Triple",
            "top.ntutn.hookpackageclassloader.ClassLoaderWrapper",
            "top.ntutn.hookpackageclassloader.ClassLoaderWrapper\$Companion"
        )

        @JvmStatic
        fun inject(context: Context): Boolean {
            // 提前加载避免死循环
            preloadClasses.forEach { Class.forName(it) }
            try {
                val originClassLoader = context.classLoader
                val originParent = originClassLoader.parent
                val applicationInfo = context.applicationInfo
                applicationInfo.sourceDir

                val delegateClassLoader = ClassLoaderWrapper3(originClassLoader)
                setParent(delegateClassLoader, originParent)
                setParent(originClassLoader, delegateClassLoader)
                hooked = true
                return true
            } catch (e: Exception) {
                Log.e(ClassLoaderWrapper3::class.simpleName, "hook失败")
            }
            return false
        }

        private fun setParent(classLoader: ClassLoader, newParent: ClassLoader) {
            val parentField = ClassLoader::class.java.getDeclaredField("parent")
            parentField.isAccessible = true
            parentField.set(classLoader, newParent)
        }

        @Throws(IllegalStateException::class)
        private fun checkHookState() {
            if (!hooked) {
                throw IllegalStateException("hook没有执行成功！")
            }
        }

        @Throws(IllegalStateException::class, RuntimeException::class)
        fun startRecord() {
            checkHookState()
            synchronized(recording) {
                if (recording) {
                    throw RuntimeException("已经存在正在监听的任务！")
                }
                loadedClasses.clear()
                recording = true
            }
        }

        @Throws(IllegalStateException::class, RuntimeException::class)
        fun endRecord(block: (Set<Pair<String, Long>>) -> Unit) {
            checkHookState()
            synchronized(recording) {
                if (!recording) {
                    throw RuntimeException("当前没有在监听！")
                }
                recording = false
                block.invoke(loadedClasses)
            }
        }
    }
}
```

相比前面方案来说，注意点就是不要搞出死循环来，另外就是Kotlin自己的类似乎并不会被BootstrapClassLoader加载，需要在计时前加载一下。

## 反射进行类加载

非常简单，就是用`Class.forName()`去加载了下这些类。

既然用了反射，混淆问题怎么解决？

幸运的是，对于一些简单的场景，ProGuard是可以帮我们处理的，其中就包括了直接使用`Class.forName("some class")`的情况。参见[ProGuard manual | Introduction | Guardsquare](https://www.guardsquare.com/en/products/proguard/manual/introduction)。

遗憾的是，当这个字符串穿上马夹（比如放到了某个变量里再传给这个函数）似乎ProGuard就不认识了。因而，我只好用笨方法生成一大堆Class.forName()了。

在demo中，这个方法正常work了，但在抖音的release包多半类会反射失败，此法不可行。按照我的思路之后如果还想继续看就只好看下怎样设计一个gradle插件，编译时从生成的mapping.txt解析出这些Class混淆后的名字再打到抖音的包里。成本很高而且我目前大概没有能力完成。

一次失败的尝试呢。但总归也算学到一些东西。