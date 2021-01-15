title: android项目开发：Kotlin
author: 归零幻想
publishDate: 2020-12-09
editDate: 2020-12-09
tags: [android, 第一行代码, Kotlin]

<!--config-->

# Kotlin

## 变量和函数

### 变量

用`var`和`val`定义变量，并有类型自动推导的支持。

`val`用来声明一个不可变的变量，`var`用来声明一个可变的变量。

```kotlin
val a = 10
var b: Int = 12
```

Kotlin完全抛弃了java中的基本数据类型，完全使用对象数据类型。

|java基本数据类型|Kotlin对象数据类型|数据类型说明|
|---|---|---|
|int|Int|整型|
|long|Long|长整型|
|short|Short|短整型|
|float|Float|单精度浮点型|
|double|Double|双精度浮点型|
|boolean|Boolean|布尔型|
|char|Char|字符型|
|byte|Byte|字节型|

### 函数

语法：

```kotlin
fun main(args: Array<String>) {
    println("Hello World!")
}
```

Kotlin中的函数在无必要时可以省略很多东西：

```kotlin
import kotlin.math.max

fun largerNumber(a: Int, b: Int) = max(a, b)

fun main(args: Array<String>) {
    val a = 3
    val b = 5
    println("The larger number of a and b is ${largerNumber(a, b)}")
}
```

## 程序的逻辑控制

### if

与java中的if语句相比，Kotlin中的if是可以有返回值的。

```kotlin
fun judge(score: Int) = if (score >= 60) "你及格了" else "你还需要多努力"

fun main() {
    println(judge(55))
    println(judge(66))
}
```

与此同时，Kotlin不再有java中的三元运算符[^1]，语义上清晰了很多。

[^1]:参考[Java中的三元运算符](https://www.w3cschool.cn/java/java-ternary-operator.html)

### when条件语句

类似于java中的switch语句，根据变量的值执行不同的逻辑。

```kotlin
fun judge(score: Int) = if (score >= 60) "你及格了" else "你还需要多努力"

fun getScore(name: String) = when (name) {
    "Tom" -> 78
    "Jack" -> 35
    "Jerry" -> 84
    "Lee" -> 57
    else -> 0
}

fun main() {
    println(judge(getScore("Tom")))
    println(judge(getScore("Jack")))
    println(judge(getScore("Bill")))
}
```

对我来说，最令人振奋的是再也不需要在每个分支里面都写个break了。其次when也是有返回值的，这和其他特性组合写出来的代码非常简洁优雅。

写个小demo吧。

```kotlin
interface Speakable {
    fun speak()
}

class Dog : Speakable {
    override fun speak() {
        println("汪汪汪")
    }
}

class Cat : Speakable {
    override fun speak() {
        println("喵喵喵")
    }

    fun climb() {
        println("小猫会爬树")
    }
}

fun generateAnimal(): Speakable? = when ((1..3).random()) {
    1 -> Dog()
    2 -> Cat()
    else -> null
}

fun main() {
    when (val animal = generateAnimal()) {
        is Dog -> {
            println("生成的动物是小狗")
            animal.speak()
        }
        is Cat -> {
            println("生成的动物是小猫")
            animal.speak()
            animal.climb()
        }
        else -> println("生成动物时出现问题")
    }
}
```

### 循环语句

Kotlin中有两类循环，其中`while`循环与java学过的while循环非常相似，只说下有差异的`for`循环吧。

Kotlin的for循环只有for..in式的了，如`for(i in list)`。

但有时对数组下标进行遍历还是有必要的。于是我们要先了解下Kotlin的`区间`的概念。

```kotlin
val range = 1..10
```

这表示`[1,10]`。但很多时候，我们需要左开右闭区间，比如数组有三个元素，我们需要`[0,3)`表示数组的下标。此时可以使用`util`关键字。

```kotlin
val indexRange = 0 util 3
```

有了range再和前面的for配合就完全可以替代之前java里面的for的作用了：

```kotlin
val array = arrayOf("Bob", "John", "Jackson")
for (i in 0 util array.size) {
    println("$i:${array[i]}")
}
```

此外，还可以用`step`指定步长值，实现”隔几个输出一次“的效果：

```kotlin
for (i in 0 util 10 step 2) {
    println(i)
}
```

如果需要10循环到1,则需要`downTo`关键字，

```kotlin
for (i in 10 downTo 1) {
    println(i)
}
```

<!--summary-->

## 面向对象编程

### 类与对象

与java的class相比Kotlin的class看上去没有多少改变，但创建对象不用new了。

```kotlin
class Person(val name: String) {
    fun speak() = "$name is speaking."
}

fun main() {
    Person("Jack").speak()
}
```

### 继承与构造函数

与java类似，Kotlin仍然是单继承，可以继承一个类实现多个接口。但继承的写法与java略有不同。

```kotlin
open class Animal//非抽象类只有带了open才可以继承

interface Speakable {
    fun speak()
}

class Dog : Animal(), Speakable {
    override fun speak() {
        println("汪汪汪")
    }
}

class Cat : Animal(), Speakable {
    override fun speak() {
        println("喵喵喵")
    }

    fun climb() {
        println("小猫会爬树")
    }
}
```

Kotlin中的构造函数分为主构造函数和次构造函数。如：

```kotlin
class Student(val sno: String, val grade: Int, name: String, age: Int) {
    constructor(name: String, age: Int) : this("", 0, name, age)

    constructor() : this("", 0)

    init {
        println("$age 的 $name 被初始化了。")
    }
}
```

次构造函数必须调用主构造函数（如果有），继承一个类必须调用他的构造函数（这也是为什么常常继承的类后面带着个括号）。

> Kotlin函数的参数可以有默认值的，大多数情况根本不需要使用多个构造函数。

主构造函数中`val`和`var`标记的变量将直接成为类的属性，没有这个标记的变量则只能在init block中访问。

### 接口

接口与java的接口是类似的。

```kotlin
interface Study {
    fun readBooks()

    fun doHomework() {
        println("Do homework default implementation")
    }
}
```

java与Kotlin中的可见性修饰略有不同。

|修饰符|java|Kotlin|
|---|---|---|
|public|所有类可见|所有类可见（默认）|
|private|当前类可见|当前类可见|
|protected|当前类、子类、同一包路径下的类可见|当前类、子类可见|
|default|同一包路径下的类可见|无|
|internal|无|同一模块中的类可见|

### 数据类与单例类

数据类在当今的系统设计中占据了重要的地位，他们格式非常固定，一般实现各种构造函数、getter和setter，重写`equals()`、`hashCode()`、`toString()`
这几个方法，真的写腻了。很多ide都提供了一键生成这些样板代码的方法，更有项目Lombok添加个注解在编译时生成这些方法[^2]。

[^2]:[Lombok](https://projectlombok.org/)，用注解的方式简化java代码，但要求使用的ide必须安装lombok插件才能正确识别lombok生成的代码，有人认为这是在”强奸队友“。另外lombok的实现调用了jdk未公开的方法也引发争议。

而Kotlin中对于数据类有了专门的支持。

```kotlin
data class Student(val sno: String, val name: String, var age: Int)
```

idea对于Kotlin有很好的支持。我们用下面的步骤将上面的代码转换为java形式：

1. Tools->Kotlin->Show Kotlin ByteCode
2. Decompile

得到的代码如下：

```java
import kotlin.Metadata;
import kotlin.jvm.internal.Intrinsics;
import org.jetbrains.annotations.NotNull;
import org.jetbrains.annotations.Nullable;

@Metadata(
        mv = {1, 4, 0},
        bv = {1, 0, 3},
        k = 1,
        d1 = {"\u0000\"\n\u0002\u0018\u0002\n\u0002\u0010\u0000\n\u0000\n\u0002\u0010\u000e\n\u0002\b\u0002\n\u0002\u0010\b\n\u0002\b\r\n\u0002\u0010\u000b\n\u0002\b\u0004\b\u0086\b\u0018\u00002\u00020\u0001B\u001d\u0012\u0006\u0010\u0002\u001a\u00020\u0003\u0012\u0006\u0010\u0004\u001a\u00020\u0003\u0012\u0006\u0010\u0005\u001a\u00020\u0006¢\u0006\u0002\u0010\u0007J\t\u0010\u000f\u001a\u00020\u0003HÆ\u0003J\t\u0010\u0010\u001a\u00020\u0003HÆ\u0003J\t\u0010\u0011\u001a\u00020\u0006HÆ\u0003J'\u0010\u0012\u001a\u00020\u00002\b\b\u0002\u0010\u0002\u001a\u00020\u00032\b\b\u0002\u0010\u0004\u001a\u00020\u00032\b\b\u0002\u0010\u0005\u001a\u00020\u0006HÆ\u0001J\u0013\u0010\u0013\u001a\u00020\u00142\b\u0010\u0015\u001a\u0004\u0018\u00010\u0001HÖ\u0003J\t\u0010\u0016\u001a\u00020\u0006HÖ\u0001J\t\u0010\u0017\u001a\u00020\u0003HÖ\u0001R\u001a\u0010\u0005\u001a\u00020\u0006X\u0086\u000e¢\u0006\u000e\n\u0000\u001a\u0004\b\b\u0010\t\"\u0004\b\n\u0010\u000bR\u0011\u0010\u0004\u001a\u00020\u0003¢\u0006\b\n\u0000\u001a\u0004\b\f\u0010\rR\u0011\u0010\u0002\u001a\u00020\u0003¢\u0006\b\n\u0000\u001a\u0004\b\u000e\u0010\r¨\u0006\u0018"},
        d2 = {"LStudent;", "", "sno", "", "name", "age", "", "(Ljava/lang/String;Ljava/lang/String;I)V", "getAge", "()I", "setAge", "(I)V", "getName", "()Ljava/lang/String;", "getSno", "component1", "component2", "component3", "copy", "equals", "", "other", "hashCode", "toString", "kotlin-learning.main"}
)
public final class Student {
    @NotNull
    private final String sno;
    @NotNull
    private final String name;
    private int age;

    @NotNull
    public final String getSno() {
        return this.sno;
    }

    @NotNull
    public final String getName() {
        return this.name;
    }

    public final int getAge() {
        return this.age;
    }

    public final void setAge(int var1) {
        this.age = var1;
    }

    public Student(@NotNull String sno, @NotNull String name, int age) {
        Intrinsics.checkNotNullParameter(sno, "sno");
        Intrinsics.checkNotNullParameter(name, "name");
        super();
        this.sno = sno;
        this.name = name;
        this.age = age;
    }

    @NotNull
    public final String component1() {
        return this.sno;
    }

    @NotNull
    public final String component2() {
        return this.name;
    }

    public final int component3() {
        return this.age;
    }

    @NotNull
    public final Student copy(@NotNull String sno, @NotNull String name, int age) {
        Intrinsics.checkNotNullParameter(sno, "sno");
        Intrinsics.checkNotNullParameter(name, "name");
        return new Student(sno, name, age);
    }

    // $FF: synthetic method
    public static Student copy$default(Student var0, String var1, String var2, int var3, int var4, Object var5) {
        if ((var4 & 1) != 0) {
            var1 = var0.sno;
        }

        if ((var4 & 2) != 0) {
            var2 = var0.name;
        }

        if ((var4 & 4) != 0) {
            var3 = var0.age;
        }

        return var0.copy(var1, var2, var3);
    }

    @NotNull
    public String toString() {
        return "Student(sno=" + this.sno + ", name=" + this.name + ", age=" + this.age + ")";
    }

    public int hashCode() {
        String var10000 = this.sno;
        int var1 = (var10000 != null ? var10000.hashCode() : 0) * 31;
        String var10001 = this.name;
        return (var1 + (var10001 != null ? var10001.hashCode() : 0)) * 31 + this.age;
    }

    public boolean equals(@Nullable Object var1) {
        if (this != var1) {
            if (var1 instanceof Student) {
                Student var2 = (Student) var1;
                if (Intrinsics.areEqual(this.sno, var2.sno) && Intrinsics.areEqual(this.name, var2.name) && this.age == var2.age) {
                    return true;
                }
            }

            return false;
        } else {
            return true;
        }
    }
}

```

孰优孰劣，一目了然。

类似的，Kotlin也对单例类提供了支持。如：

```kotlin
object Singleton {
    fun singletonTest() {
        println("Singleton test is called.")
    }
}
```

这和你费了半天劲写出来的java代码作用是一样的：

```java
import kotlin.Metadata;

@Metadata(
        mv = {1, 4, 0},
        bv = {1, 0, 3},
        k = 1,
        d1 = {"\u0000\u0012\n\u0002\u0018\u0002\n\u0002\u0010\u0000\n\u0002\b\u0002\n\u0002\u0010\u0002\n\u0000\bÆ\u0002\u0018\u00002\u00020\u0001B\u0007\b\u0002¢\u0006\u0002\u0010\u0002J\u0006\u0010\u0003\u001a\u00020\u0004¨\u0006\u0005"},
        d2 = {"LSingleton;", "", "()V", "singletonTest", "", "kotlin-learning.main"}
)
public final class Singleton {
    public static final Singleton INSTANCE;

    public final void singletonTest() {
        String var1 = "Singleton test is called.";
        boolean var2 = false;
        System.out.println(var1);
    }

    private Singleton() {
    }

    static {
        Singleton var0 = new Singleton();
        INSTANCE = var0;
    }
}

```

## Lambda编程

```kotlin
fun main() {
    val map = mapOf("Apple" to 1, "Banana" to 2, "Orange" to 3, "Pear" to 4, "Grape" to 5)
    for ((fruit, number) in map) {
        println("fruit is $fruit, number is $number")
    }
}
```

### 函数式API

#### maxBy

集合中最长的字符串。

### map

将集合中每个元素都映射到另一个元素。

#### filter

过滤集合中的数据。

#### any

至少有一个元素满足条件。

#### all

所有元素都满足条件。

### Java函数式API

对于java中的单抽象方法可使用函数式API。

```kotlin
Thread {
    println("Thread is running")
}.start()
```

## 空指针检查

写java时最常见的错误就是`java.lang.NullPointerException`了吧。在Kotlin中，情况有一些改善。

与java不同，Kotlin中的变量默认是不可空的。

```kotlin
fun doStudy(study: Study) {
    study.readBooks()
    study.doHomework()
}
```

如果你给这个函数传一个可能为null的值，在编译期间就会得到错误提示。

当然，很多情况我们还是需要让我们的函数接受一个可空的值的，则可以用这样的写法：

```kotlin
fun doStudy(study: Study?) {
    if (study != null) {
        study.readBooks()
        study.doHomework()
    }
}
```

但在Kotlin中还有更好的方法：

```kotlin
fun doStudy(study: Study?) {
    study?.readBooks()
    study?.doHomework()
}
```

用上`let`我们还可以写得更优雅：

```kotlin
fun doStudy(study: Study?) {
    study?.let {
        it.readBooks()
        it.doHomework()
    }
}
```

`?.`表示只有不为空时才正常调用。

有时我们需要这样的逻辑：

```kotlin
fun getTextLength(text: String?): Int {
    if (text != null) {
        return text.length
    }
    return 0
}
```

Kotlin也有个方便的操作符：

```kotlin
fun getTextLength(text: String?) = text?.length ?: 0
```

`?:`表示前面为空时返回后面的值。

当然，有些业务逻辑中Kotlin不一定能正确推断出你的变量是否可能为空，这时Kotlin也提供了让你自己操纵它的机会：

```kotlin
var content: String? = null

fun initContent() {
    content = "https://ntutn.top"
}

fun main() {
    initContent()
    if (content != null) {
        printContent()
    }
}

fun printContent() {
    println(content!!)
}
```
