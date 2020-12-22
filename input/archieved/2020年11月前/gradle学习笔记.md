title: gradle学习笔记
author: 归零幻想
publishDate: 2020-11-13
editDate: 2020-11-15
tags: [编程, groovy, gradle, 安卓, java]

<!--config-->

新建一个安卓项目，`Android Studio`已经为我们生成的模板中，`build.gradle`总是很显眼。我也终于下定决心来了解下这个构建整合工具。

gradle脚本使用`groovy`编写，最近也开始支持用`Kotlin`编写。Kotlin大法好，但我暂时只找到了groovy的教程[^1]，暂时不想去啃英文，先将就学着。

[^1]:[Gradle User Guide 中文版](https://wiki.jikexueyuan.com/project/GradleUserGuide-Wiki/)

## gradle安装

### 命令

对于OSX用户来说，`brew install gradle`就可以搞定这个步骤了。但是这里有个问题，就是这样会把openjdk当做依赖装上。也不算很大的问题，也能将就用。

### gradle wrapper

gradle有一个很方便的命令，`gradle wrapper`，使用后会创建如下的目录结构

```
.
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
└── gradlew.bat
```

此时即使你没有安装gradle，仍然可以使用`./gradlew`代替`gradle`执行命令。如`./gradlew clean install`。在实际的项目中，这种方式使用比较普遍，因为gradle脚本很多特性受版本影响比较大，一般gradle的版本也会在项目中指定。

<!--summary-->

## task

### 基本使用

task即构建任务，一个最简单的构建任务如下

```groovy
task hello {
    doLast {
        println 'Hello world!'
    }
}
```

此时输入`gradle -q hello`就可以看到输出了。`-q`指`quiet`，会忽略一些gradle自身的输出。

> 教程中还写了一种更简洁的写法[^2]，但我发现这种写法已经被废弃[^3]，不再写出。

[^2]:[https://wiki.jikexueyuan.com/project/GradleUserGuide-Wiki/build_script_basics/a_shortcut_task_definition.html](https://wiki.jikexueyuan.com/project/GradleUserGuide-Wiki/build_script_basics/a_shortcut_task_definition.html)

[^3]:[https://www.cnblogs.com/yuanchaoyong/p/11421595.html](https://www.cnblogs.com/yuanchaoyong/p/11421595.html)

### 任务依赖

声明任务间的依赖关系。

```groovy
task taskY {
	doLast {
		println 'taskY'
	}
}
task taskX(dependsOn: taskY) {
	doLast {
		println 'taskX'
	}
}
```

或

```groovy
task taskX(dependsOn: 'taskY') { //因为taskY还未定义，只能这样使用
	doLast {
		println 'taskX'
	}
}
task taskY {
	doLast {
		println 'taskY'
	}
}
```

```
➜  task gradle -q taskX
taskY
taskX
➜  task
```

### 动态任务

可以动态创建任务，使用已经存在的任务。

```groovy
4.times { counter ->
    task "task$counter" {
		doLast{
			println "I'm task number $counter"
		}
	}
}
task0.dependsOn task2, task3
```

```
➜  task gradle -q task0
I'm task number 2
I'm task number 3
I'm task number 0
➜  task 
```

### 自定义属性

自定义属性和默认任务

```groovy
defaultTasks 'information','hello'

task information {
	doLast {
		println 'I use this script to learn gradle.'
	}
}

task hello {
	ext {
		computer = 'IF'
		user = 'Haidee'
	}

	doLast {
		println 'Hello Susan'
	}	
	
	doFirst {
		println 'Hello Zero'
	}

	doLast {
		println "There are also $hello.computer and $hello.user in our organization."
	}
}
```

```
➜  task gradle -q
I use this script to learn gradle.
Hello Zero
Hello Susan
There are also IF and Haidee in our organization.
➜  task 
```

### 判断发布任务是否在要被执行的任务中

```groovy
task distribution << {
    println "We build the zip with version=$version"
}

task release(dependsOn: 'distribution') << {
    println 'We release now'
}

gradle.taskGraph.whenReady {taskGraph ->
    if (taskGraph.hasTask(release)) {
        version = '1.0'
    } else {
        version = '1.0-SNAPSHOT'
    }
}
```

```
➜  task gradle -q distribution
We build the zip with version=1.0-SNAPSHOT
➜  task gradle -q release
We build the zip with version=1.0
We release now
➜  task 
```

> 使用`gradle tasks --all`可以列出当前项目所有构建任务。

## java构建

### 基础java项目

```sh
mkdir javaproject
cd javaproject
mkdir -p src/main/java
mkdir -p src/test/java
echo "apply plugin: 'java'" > build.gradle
```

```
.
├── build.gradle
└── src
    ├── main
    │   └── java
    └── test
        └── java

5 directories, 1 file
```

#### 添加依赖

```groovy
repositories {
    mavenCentral()
}

dependencies {
    implementation group: 'commons-collections', name: 'commons-collections', version: '3.2'
    testImplementation group: 'junit', name: 'junit', version: '4.+'
}
```

#### 定制 MANIFEST.MF 文件

```groovy
sourceCompatibility = 1.8
version = '1.0'

jar {
    manifest {
        attributes 'Implementation-Title': 'Gradle Quickstart', 'Implementation-Version': version
    }
}
```

#### 发布jar文件

```groovy
uploadArchives {
    repositories {
       flatDir {
           dirs 'repos'
       }
    }
}
```

### 多项目java构建

```sh
mkdir multiproject
cd multiproject
mkdir api
mkdir -p services/webservice
mkdir shared
```

#### 项目结构

```
.
├── api
│   ├── build
│   └── build.gradle
├── build.gradle
├── services
│   └── webservice
│       └── build.gradle
├── settings.gradle
└── shared
    └── build.gradle
```

`settings.gradle`指定要包括哪个项目。

```groovy
include "shared", "api", "services:webservice", "services:shared"
```

#### 通用配置

根项目的`build.gradle`中

```groovy
subprojects {
    apply plugin: 'java'

    repositories {
        mavenCentral()
    }

    dependencies {
        testImplementation 'junit:junit:4.11'
    }

    version = '1.0'

    jar {
        manifest.attributes provider: 'gradle'
    }
}
```

#### 项目之间的依赖

api/build.gradle

```groovy
dependencies {
    compile project(':shared')
}
```

## 依赖管理基础知识

> 依赖类型，待整理，当前找到的资料有点乱。

引用外部依赖需要`group`、`name`和`version`属性。

仓库（未完全验证）。

```groovy
repositories {
    mavenCentral()
    maven {
        url "http://repo.mycompany.com/maven2"
    }
    ivy {
        url "http://repo.mycompany.com/repo"
    }
    ivy {
        // URL can refer to a local directory
        url "../local-repo"
    }
}
```

## groovy快速入门

就一个例子，入门入了个寂寞。

```groovy
apply plugin: 'groovy'

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.codehaus.groovy:groovy-all:2.3.3'
    testImplementation 'junit:junit:4.11'
}
```

## 编写构建脚本

### 标准项目属性

project对象提供了一些标准属性，可以在对象中很方便地使用。

|Name	|Type	|Default Value|
|---|---|---|
|project	|Project	|Project 实例对象|
|name	|String	|项目目录的名称|
|path	|String	|项目的绝对路径|
|description	|String	|项目描述|
|projectDir	|File	|包含构建脚本的目录|
|build	|File	|projectDir/build|
|group	|Object	|未具体说明|
|version	|Object	|未具体说明|
|ant	|AntBuilder	|Ant实例对象|

> 来自Gradle User Guide[^1]，未验证。

### 变量

声明变量

```groovy
def dest = "dest"
```

拓展属性

```groovy
ext {
    springVersion = "3.1.0.RELEASE"
    emailNotification = "build@master.org"
}
```

### 方法

圆括号可选。