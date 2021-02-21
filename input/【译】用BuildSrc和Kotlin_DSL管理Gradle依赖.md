title: 【译】用BuildSrc和Kotlin_DSL管理Gradle依赖
author: Siva Ganesh Kantamani
publishDate: 2021-02-21
editDate: 2021-02-21
tags: [翻译, 安卓, gradle, Kotlin]

<!--config-->

![](https://miro.medium.com/max/1400/1*ZKNUGwUmBjukpnTrHCeGJA.png)

> 多模块工程中一个更好的引入依赖的方法

翻译自[Gradle Dependency Management With BuildSrc and Kotlin DSL](https://medium.com/better-programming/gradle-dependency-management-with-buildsrc-and-kotlin-dsl-1de958eab166)

第一次尝试翻译英文文章……

## 要点

主要集中在如何用`buildSrc`目录和Kotlin DSL脚本构建一个Gradle依赖管理系统，你也会学到这样做相对使用传统Groovy代码的好处。

如果你倾向于通过视频来看这篇博客，文末附有一个Youtube视频。

## 问题

众所周知在一个快速发展的项目中维护依赖是一个乏味的工作，而传统的Groovy脚本没有code navigation、自动补全，再加上性能问题和运行时错误让这一切变得更糟糕。

更重要的是，多数安卓开发者不懂Groovy，甚至我也不知道我之前在用Groovy做啥。

感谢Gradle团队和社区的工作提供了一个顺畅安全的构建流程，他们提出的最棒的主意之一就是用Kotlin DSL脚本写buildSrc目录。

# 解决

依赖库引入和自定义task不应该放到构建脚本中，它们应该被声明到一个独立文件中再被构建脚本使用。在这个实现的早期，开发者习惯于创建一个Gradle文件来声明所有库并在构建脚本中使用。

这确实在一定程度上解决了问题，你可以在[这篇文章](https://medium.com/@sgkantamani/next-level-of-dependencies-declaration-with-kotlin-dsl-scripits-48bfe1cb1f10)读到这种方法。但这个简单方案不能解决类似自动补全和code navigation的问题，这使得在长远上看这个方案不够可靠。在这之外，buildSrc似乎有希望解决这个问题。

> 这个目录被当作一个[included build](https://docs.gradle.org/current/userguide/composite_builds.html#composite_build_intro)看待。在发现这个目录之后，Gradle自动编译和测试它的代码，并将编译结果放到你的构建脚本的class path中。在一个多模块的工程中只能有一个这样的目录，并且要放到工程的顶级目录中。应该优先通过[script plugins](https://docs.gradle.org/current/userguide/plugins.html#sec:script_plugins)因为这样更便于管理、重构和测试代码。
> ——Gradle团队

## 创建buildSrc目录

使用Kotlin DSL脚本不但能解决构建脚本中的这些问题，还能得到先进的IDE支持，包括code navigation、编译时错误提示等。最重要的，我们再也不用使用Groovy了。

<!--summary-->

我们要做的第一件事是创建一个buildSrc目录：

- 在工程上右键
- 点击New并选择Directory
- 把它命名为`buildSrc`

如果你仍然不懂怎么创建这个目录，请看这里：

![](https://miro.medium.com/max/1400/1*rDhflrwxcwwr4mcefTrQEg.gif)


然后我们需要在这个目录里创建一个叫做`build.gradle.kts`的文件，在这个文件里导入插件和存储库。

```kotlin
plugins{
    `kotlin-dsl`
}

repositories {
    jcenter()
}
```

完成后你还需要点击gradle的“sync now”按钮，因为gradle把它当作了一个新建目录中的普通文件。现在你可以实现Kotlin DSL脚本了。

![](https://miro.medium.com/max/870/1*L1zYKShhvKrrgTyn07S3aw.png)

下一步是创建一个类似这样的目录结构`src>main>java`，完成后如上图所示。

现在我们可以创建Kotlin文件来声明依赖库，管理版本或者实现自定义task。

现在我们的目标是实现一个依赖管理系统，所以我们创建一个叫做`Dependencies.kt`的文件（你可以用你喜欢的任何名字）。

完成后我们就可以通过Kotlin代码来声明依赖库和版本了。这里我们用object来声明特定的类型，例如版本号、AndroidX依赖库等。

首先我们创建一个用来用Kotlin风格定义所有依赖库版本号的object。

```kotlin
object Versions{

    val constraint_layout_version = "1.1.3"
    val lifecycle_version = "2.2.0"
    val coroutines_version = "1.3.8"
    val kotlin_version = "1.3.41"
  
    val appcompat = "1.1.0"
    val recycelerview_version = "1.0.0"
    val material_design_version = "1.2.0"
    val cardview_version = "1.0.0"
    val viewpager2_version = "1.0.0"
  
    val play_core_version = "1.8.0"
}
```

下一步我们创建各种依赖库类型的专属object文件，例如Kotlinlibraries、AndroidXLibraries和UiLibraries等。

```kotlin
object kotlinDependencies {
    val kotlin = "org.jetbrains.kotlin:kotlin-stdlib-jdk7:${Versions.kotlin_version}"
    val coroutines = "org.jetbrains.kotlinx:kotlinx-coroutines-android:${Versions.coroutines_version}"
}

object androidxsupportDependencies{
    val appcompat = "androidx.appcompat:appcompat:${Versions.appcompat}"
    val cardview = "androidx.cardview:cardview:${Versions.cardview_version}"
    val contraintLayout = "androidx.constraintlayout:constraintlayout:${Versions.constraint_layout_version}"
    val recyclerview = "androidx.recyclerview:recyclerview:${Versions.recycelerview_version}"
    val viewpager2 = "androidx.viewpager2:viewpager2:${Versions.viewpager2_version}"
}

object materialDesignDependencies{
    val materialDesign = "com.google.android.material:material:${Versions.material_design_version}"
}

object playcoreDependencies{
    val play_core =  "com.google.android.play:core:${Versions.play_core_version}"
}
```

最后把完整代码片段放到这里

```kotlin
object Versions{

    val constraint_layout_version = "1.1.3"
    val lifecycle_version = "2.2.0"
    val coroutines_version = "1.3.8"
    val kotlin_version = "1.3.41"
  
    val appcompat = "1.1.0"
    val recycelerview_version = "1.0.0"
    val material_design_version = "1.2.0"
    val cardview_version = "1.0.0"
    val viewpager2_version = "1.0.0"
  
    val play_core_version = "1.8.0"
}

object kotlinDependencies {
    val kotlin = "org.jetbrains.kotlin:kotlin-stdlib-jdk7:${Versions.kotlin_version}"
    val coroutines = "org.jetbrains.kotlinx:kotlinx-coroutines-android:${Versions.coroutines_version}"
}

object androidxsupportDependencies{
    val appcompat = "androidx.appcompat:appcompat:${Versions.appcompat}"
    val cardview = "androidx.cardview:cardview:${Versions.cardview_version}"
    val contraintLayout = "androidx.constraintlayout:constraintlayout:${Versions.constraint_layout_version}"
    val recyclerview = "androidx.recyclerview:recyclerview:${Versions.recycelerview_version}"
    val viewpager2 = "androidx.viewpager2:viewpager2:${Versions.viewpager2_version}"
}

object materialDesignDependencies{
    val materialDesign = "com.google.android.material:material:${Versions.material_design_version}"
}

object playcoreDependencies{
    val play_core =  "com.google.android.play:core:${Versions.play_core_version}"
}
```

最后一步就是在project、app和模块级的build.gradle文件中使用这些object了。这相当简单，只要object的name和成员变量的name就行了。

```kotlin
dependencies {
    /** Kotlin */
    implementation( kotlinDependencies.kotlin)

    /** Coroutines */
    implementation( coroutineDependencies.coroutines)
    
    /** support androidx ibraries */
    implementation( androidxsupportDependencies.appcompat)
    implementation( androidxsupportDependencies.contraintLayout)
    implementation( androidxsupportDependencies.recyclerview)
    implementation( androidxsupportDependencies.cardview)
    implementation( androidxsupportDependencies.viewpager2)

    /*** Material design */
    implementation( materialDesignDependencies.materialDesign)
    
    /** Playcore */
    implementation( playcoreDependencies.play_core)
}
```

这种方法看上去简单，但拥有很多优点，比如code navigation、自动补全、运行时错误提示等。看下面的gif。（gif见原文）

此外，我们可以在整个项目的所有模块中使用这些定义的依赖。

## 更多

目前为止，我们实现了与依赖有关的所有东西，但我们确实可以在编译脚本中做更多事情，比如替换掉defaultConfig块，它看上去这样

```kotlin
compileSdkVersion(29)
defaultConfig {
    applicationId = "com.example.app"
    minSdkVersion(16)
    targetSdkVersion (29)
    multiDexEnabled = true
    versionCode = 1
    versionName = "1.0"
    testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
    vectorDrawables.useSupportLibrary = true
}
```

如我所说，把常量移到编译脚本外更安全。所以我们可以在下面另外创建一个Kotlin文件，或者在Dependencies.kt文件中创建一个新的object。然后声明所有常量，比如min、编译SDK版本等。

```kotlin
object buildConfigVersions {
    val compileSdkVersion = 29
    val buildToolsVersion = "29.0.3"
    val minSdkVersion = 16
    val targetSdkVersion = 29
    val versionCode = 6
    val versionName = "2.0"
}
```

现在我们可以在block中使用这个object了

```kotlin
compileSdkVersion(buildConfigVersions.compileSdkVersion)
defaultConfig {
    applicationId = "com.example.app"
    minSdkVersion(buildConfigVersions.minSdkVersion)
    targetSdkVersion (buildConfigVersions.targetSdkVersion)
    multiDexEnabled = true
    versionCode = buildConfigVersions.versionCode
    versionName = buildConfigVersions.versionName
    testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
    vectorDrawables.useSupportLibrary = true
}
```

如果你的工程中有任何顶级字符串或常量需要维护，你可以创建一个单独的文件，而不是在编译脚本中写的一团糟。

（Youtube视频见[https://youtu.be/w5qCmvS9JGE](https://youtu.be/w5qCmvS9JGE)）