title: 写产品和写demo的区别
author: 归零幻想
publishDate: 2020-09-08
editDate: 2020-09-08
tags: [NPE, 空指针, demo, Kotlin]

<!--config-->

上次写的功能灰度报了几例crash，定位是我这里某个变量NPE了。

说实话，在Java中NPE是我们最常打交道的异常了，但Kotlin提供的类型机制下，NPE很少了。而这里出现NPE，其实是因为我认定在当前这个流程中这个变量不会为空——你总是要先弹出菜单再点击菜单项吧，所以用了`data!!.id`的方式使用。

这个变量被赋值的地方只有三处，不存在多线程问题，我想破脑袋也不知道哪里为空了。最后处理只得暂时加上判空，先不崩再说。

请教同事，同事说我这是还没有分清 **写产品和写demo的区别** 。

<!--summary-->

写产品和写demo的区别？仔细一想，的确有道理。『写demo』只要能跑通就行了，而写产品你是要为自己写的每一行代码负责的。我之前写过不少代码，但他们大概都算『写demo』。作业只要演示的时候不崩就行了，考试只要通过样例就行了，练手的项目写起来更是随心，反正自己就是用户，啥时候崩了啥时候debug，方便的很……

但『写产品』不同，你的程序不止要能完成需要的功能，还要在用户不按套路出牌的时候不出错。*今天你迟到一分钟，咱班四十个学生等你一分钟就是一节课，你浪费了大家一节课时间……* 虽然这么算不对，但也不能算全错。你的产品面向千千万万用户，任何小瑕疵都可能放大成一场事故。记得那个著名的ATM机的bug，就是因为用户选择了取消，然后插入了银行卡……

回到我这个Issue，虽然从业务逻辑的角度看用户操作后这个变量是不会为null的，但仍然应该有判空，有兜底的逻辑，这样你的程序才会更加健壮。

## Kotlin的空安全

与Java不同，Kotlin的类型系统在设计时就考虑了变量是否可空[^1]，其用一个?表示变量是否能为空。

```kt
var a: String = "abc" // 默认情况下，常规初始化意味着非空
a = null // 编译错误
```

```kt
var b: String? = "abc" // 可以设置为空
b = null // ok
print(b)
```

### 判空

Kotlin有一定的类型推断能力，判空后进行赋值操作前能识别出变量是非空的类型。

```kt
val b: String? = "Kotlin"
if (b != null && b.length > 0) {
    print("String of length ${b.length}")
} else {
    print("Empty string")
}
```

### 安全调用

Kotlin中提供了`?.`来安全调用一个可能为空的变量。`b?.test()`相当于`if (b != null) b.test()`。

值得注意的是，这个操作符是可以和赋值语句一起用的。如：

```kt
b?.name = "ByteDance"
```

换个形式可能更好理解：

```kt
if (b != null) b.setName("ByteDance")
```

这个操作符还可以与`.let`配合，达到非空时执行特定语句的目的。

```kt
b?.name?.let { println(it) }
```

### Elvis 操作符

在java中适当使用三元运算符也能让代码写起来更简洁一些：

```java
String title = article.title == null ? "未命名" : article.title;
```

在Kotlin中没有三元运算符了，但if和when语句可以有返回值了。于是可以写作

```kt
var title = if (article.title == null) "未命名" else article.title
```

不过还有更简单的表达方法：

```kt
var title = article.title ?: "未命名"
```

### !!操作符

终于说到我翻车的地方了。`!!`操作符表示“我保证这个变量不为空，否则就抛出NPE吧！”。尽管变量可能的确不会为空，两个感叹号上去IDE就不报错了……

这是很不好的，所有Kotlin的书都告诉你要少用这个符号。这从这个符号的设计就看出来了，写起来就像在对编译器咆哮“我知道自己在做什么！！”。

[^1]:[空安全 - Kotlin 语言中文站](https://www.kotlincn.net/docs/reference/null-safety.html)
