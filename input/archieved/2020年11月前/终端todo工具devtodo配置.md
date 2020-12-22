title: 终端todo工具devtodo配置
author: 归零幻想
publishDate: 2020-09-16
editDate: 2020-09-16
tags: [todo, 终端, 效率, git]

<!--config-->

## 背景

昨天突然发现前次提交的代码有一点小问题，然而我正在忙另外一个功能，就想先记下。看了看微软的待办事项，突然想起了之前了解的devtodo。

## todo工具

我用过的todo工具其实也就三款，其中还包括一个我自己写的[^1]。

[^1]:指ctodo。

<!--summary-->

### ctodo

一款简单的自己写的终端todo工具。

[https://github.com/zerofancy/ctodo](https://github.com/zerofancy/ctodo)

### 微软待办事项

前身奇妙清单，后被微软收购。界面简洁大方，使用方便，而且能直接用微软帐号同步数据。

支持Windows[^2]、OSX、iOS、Android[^3]平台，要不是我是个Linux用户，那就完美了。[^4]

[^2]:得在应用商店下载。
[^3]:在某次更新后开始提示要GMS，这对国内用户影响不小。
[^4]:没有官方的Linux版本，倒是也有[第三方的electron版本](https://github.com/klaussinani/ao)，勉强凑合用。

### devtodo

![devtodo](https://i.loli.net/2020/09/16/ziwSVOsohHguTjP.png)

正是发现了这个导致我没了再继续做ctodo的打算。是个比较流行的终端todo工具。

相对于GUI的todo工具，devtodo非常适合开发者。很多IDE都能很方便地打开终端，还正好定位到项目目录。这时我们可以把要做的事情加到devtodo，打开终端就能找到。

目前devtodo好像已经出了devtodo2，说是用go重写了，而原版本不再维护[^5]。但目前为止，debian软件源和OSX的homebrew上都搜不到devtodo2.反正devtodo1也能满足功能，暂时还是安装1吧。

[^5]:[https://swapoff.org/devtodo1.html](https://swapoff.org/devtodo1.html)

## 安装与配置

### 安装

debian系只要

```sh
sudo apt install devtodo
```

即可。OSX自然用Homebrew：

```sh
brew install devtodo
```

现在，你就可以在终端用`tda`命令添加todo，`todo`查看todo，`tdd`完成todo，`tdr`删除todo。具体的用法可以通过`man todo`查看。

### 配置cd后自动显示todo

TODO List主要是用来提醒我们有什么事情还没有做，但是如果我们连输入`todo`看一下都忘了咋办……

我们可以cd到某个目录下时自动列出当前位置的`todo`。

将下面的代码加入到`~/.bashrc`[^6]中即可。

[^6]: 如果你使用zsh则为`~/.zshrc`。

```sh
# 自动显示当前目录todo
cd_todo()
{
	\cd $1
	todo
}

alias cd="cd_todo"
# 刚打开终端时也显示下todo
todo
```

### 配置gitignore

devtodo会在当前目录生成一个`.todo`文件用于记录todo项目，当我们和别人合作开发时显然是不应该把这个文件提交到git仓库的。我们都知道这在`.gitignore`中添加`.todo`即可。

不过问题是我总是觉得这太高调了，说不定回头有人问你添加这么个文件是做什么的。而且每个git仓库都要添加，还是很麻烦的。

因此，建议添加到全局的`.gitignore`中。

即在`~/.gitignore`中添加

```gitignore
.todo
```
