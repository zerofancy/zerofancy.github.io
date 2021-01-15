title: android项目开发：项目结构
author: 归零幻想
publishDate: 2020-12-08
editDate: 2020-12-08
tags: [android, 第一行代码, Kotlin]

<!--config-->

虽然已经在字节实习并拿到转正offer，但实际我自己感受我现在对安卓基础知识掌握的程度还差很多，感觉写业务代码本身并不能带来多少提升。

恰逢前两天看到黄正楠那里有一本看上去不错的书[^1]，而在淘宝也在打折，就买了一本。

[^1]:《第一行代码》

那么从Hello Wrold开始，先看看安卓项目的项目结构。

<!--summary-->


[TOC]

## .gradle和.idea

Android Studio自动生成的文件，无需关心。

## app

项目中的代码、资源等内容。

### build

编译时自动生成的文件，不需要关心。

### libs

存放项目中的第三方jar包，这个目录下的jar包会自动添加到项目的构建路径下。

### src
#### androidTest

Android Test测试用例，可以对项目进行一些自动化测试。

> 实际上在公司里发现基本不写测试用例，全靠QA瞎几把点。

#### test

用来编写单元测试用例，对项目进行自动化测试。

> 这里的测试用例是不依赖安卓框架的。

#### main
##### java

存放所有java代码（和Kotlin代码）的地方。

##### res

项目的资源文件夹，项目中使用到的所有图片、布局、字符串等资源都存在这个目录下。

图片存放在`drawable`目录下，布局存放在`layout`目录下，字符串存放在`values`目录下。

`mipmap`存放图标，之所以有很多`mipmap`目录是为了适配各种设备。若只有一份图片，那么放在`xxhdpi`下就可以了。

##### AndroidManifest.xml

整个Android项目的配置文件，程序中定义的四大组件都需要在这个文件中注册。此外，还可以在这个文件中添加应用程序的权限声明。

### .gitignore

类似外层的`.gitignore`，用来在版本控制系统中排除app模块中的指定文件。

### build.gradle

app模块的gradle构建脚本，指定很多项目构建相关的配置。

插件`com.android.application`用于应用程序模块，`com.android.library`表示库模块。前者可以直接运行，后者只能作为代码库依附于应用程序模块运行。

### proguard-rules.pro

指定项目代码的混淆方式。

## build

编译时自动生成的内容。

## gradle
 
gradle wrapper的配置文件。Android Studio会根据本地的缓存i去=情况决定是否需要联网下载gradle。

## .gitignore

排除文件或文件夹的git版本控制。

> 对于所有项目都会用到的`.gitignore`条目，不妨加入`~/.gitignore`全局配置。

## build.gradle

项目全局的构建脚本。

repositories中，`google()`对应谷歌自家代码仓库依赖，`jcenter()`中则是很多第三方开源库。

## gradle.properties

全局gradle的配置文件。

## gradlew和gradlew.bat

这两个文件是用来在命令行界面执行gradle命令的，其中`gradlew`是在Linux或Mac系统中使用的，`gradlew.bat`是在Windows系统中使用的。

## local.properties

用于指定本机中的Android SDK路径。

## settings.gradle

用来指定项目中所有引入的模块。
