title: Kotlin中的单例模式
author: 归零幻想
publishDate: 2020-08-07
editDate: 2020-08-14
tags: [Kotlin, java, 单例模式]

<!--config-->

这两天要学习埋点相关的知识，做抖音某页面跳转的埋点，于是就去看下单例模式在`Kotlin`的实现。

## 单例模式
单例模式是一种常用的软件设计模式，属于创建型模式的一种。许多时候整个系统只需要拥有某个类的一个全局对象，用于协调系统整体的行为。比如我正打算做的这个埋点，这段计时的逻辑就比较适合用单例模式。

单例模式创建的对象会长期存在，所以一般比较关注它的内存占用问题，希望只有方法被调用才创建对象。这样就在多线程的场景会遇到一些问题，即当两个线程同时调用`getInstance()`时，都检测到`instance`为`null`并各自创建对象，这样就有两个实例被创建，违背了单例模式的原则。

<!--summary-->

## java中的单例模式

单例模式有所谓『懒汉式』和『饿汉式』之分，java中最典型的线程安全的两种单例模式大概如此：

### 饿汉式

饿汉式指没有懒加载，在类加载时就创建实例的单例模式。由于利用了类加载机制，天生线程安全。

```java
/**
 * 饿汉式
 */
public final class SingletoneJava {
    public static final SingletoneJava INSTANCE;

    private SingletoneJava() {
    }

    static {
        System.out.println("创建实例！");
        INSTANCE = new SingletoneJava();
    }

    public void test() {
        System.out.println("测试方法执行！");
    }
}
```

### 懒汉式

首先贴出代码

```java
/**
 * 懒汉式
 */
public class SingletoneJavaLazy {
    private volatile static SingletoneJavaLazy instance;

    private SingletoneJavaLazy() {
    }

    private static SingletoneJavaLazy getInstance() {
        if (instance == null) {
            synchronized (SingletoneJavaLazy.class) {
                if (instance == null) {
                    instance = new SingletoneJavaLazy();
                }
            }
        }
        return instance;
    }

    public void test() {
        System.out.println("Hello World by java " + getClass().getSimpleName());
    }
}
```

首先我们注意到的是这里有两次`instance`为`null`的判断。内层的判断很简单，`instance`不存在创建实例，加锁解决线程安全。但加锁的开销不可忽略，为了避免每次`getInstance()`都加锁，我们只需要在`instance`为`null`时加锁就可以了。

然后注意到这里有一个`volatile`关键字。搜索相关资料得知这是因为`instance = new SingletoneJavaLazy()`不是原子操作，执行时有三步操作

1. `instance`分配内存
2. 调用构造函数
3. 将`instance`指向内存区域

jvm可能会进行指令重排导致`instance`不是`null`但没有初始化的情况，导致空指针。`volatile`禁止了指令重新排序，并做了一些底层工作保证可见性。

## Kotlin中的通常做法

### object

`Kotlin`引入了一个叫做`object`的类型，实现单例模式就非常简单。

```kotlin
object SingletoneKotlin {
    fun test(){
        println("Kotlin测试！")
    }
}
```

那么它是『懒汉式』的，还是『饿汉式』的？我们可以用idea提供的工具将他转换为java的实现。

1. Tools -> Kotlin -> Show Kotlin Bytecode
2. Decompile

```java
public final class SingletoneKotlin {
   public static final SingletoneKotlin INSTANCE;

   public final void test() {
      String var1 = "Kotlin测试！";
      boolean var2 = false;
      System.out.println(var1);
   }

   private SingletoneKotlin() {
   }

   static {
      SingletoneKotlin var0 = new SingletoneKotlin();
      INSTANCE = var0;
   }
}
```

一个标准的饿汉式嘛。

### 懒汉式实现？

我们可以参考java中的实现写出`Kotlin`的单例模式懒汉式实现。

```kotlin
/**
 * 懒汉式
 */
class SingletoneKotlinLazy private constructor() {
    fun test() {
        println("Hello World by Kotlin ${javaClass.simpleName}")
    }

    companion object {
        @Volatile
        private var instance: SingletoneKotlinLazy? = null

        fun getInstance(): SingletoneKotlinLazy {
            return instance?: synchronized(SingletoneKotlinLazy::class.java){
                return instance?:SingletoneKotlinLazy()
            }
        }
    }
}
```

不过还有更简单的实现。

```kotlin
class SingletoneKotlinLazy {
    companion object {
        val instance by lazy {
            SingletoneKotlinLazy()
        }
    }

    fun test(){
        println("Hello World by Kotlin ${javaClass.simpleName}")
    }
}
```

`lazy()`是接受一个lambda返回一个`Lazy<T>`的函数，默认参数内部实现正是双重检测的机制。所以，起飞！

## 懒汉式与饿汉式差别有多大

~~饿汉式创建实例在类加载时~~，那么jvm在何时进行类加载呢？我们调用下看看。

> 查资料并[写demo](https://github.com/zerofancy/singletone)验证得知static代码块执行并不是在类加载时，而是在初始化时。**所以懒汉式那还有啥用？我现在很疑惑。**

```kotlin
fun main(){
    println("Hello World!")
    SingletoneJava.INSTANCE.test()
    SingletoneKotlin.test()
}
```

输出结果：

```
Hello World!
创建实例！
测试方法执行！
Kotlin测试！

```

好像……也是在实际执行方法时创建的实例？那么和懒汉式也没区别啊。

事实上，jvm并没有强制的规范规定何时进行类加载，这取决于jvm的具体实现。

## 参考

- [单例模式 - 维基百科](https://zh.wikipedia.org/wiki/%E5%8D%95%E4%BE%8B%E6%A8%A1%E5%BC%8F)
- [漫画：什么是单例模式？（整合版）](https://blog.csdn.net/bjweimengshu/article/details/78716839)
- [理解 Java volatile 关键字](https://lotabout.me/2019/Java-volatile-keyword/)
