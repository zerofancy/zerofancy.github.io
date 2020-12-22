title: Typora+git自动同步你的笔记
author: 归零幻想
publishDate: 2020-03-02
editDate: 2020-03-02
tags: [笔记, 工具, git, markdown]

<!--config-->

这两天看了群里某个大佬的笔记，突然觉得自己用VSCode记笔记的方案不香了。VSCode虽然对`markdown`的支持不错，但毕竟本职工作不是这个，直接拿来做笔记还是不够方便，太笨重了。于是我终于想起之前同学给我安利的`markdown`编辑器：Typora。

## dalao的笔记

首先看看大佬的笔记：

![1.png](https://i.loli.net/2020/08/20/1Lr3BfMvSw8haYy.png)

好想去偷他的笔记。不过话说他这笔记软件也不错啊，看起来简约清晰，我也有试一试的想法了。

他用的笔记软件：[https://github.com/tsujan/FeatherNotes](https://github.com/tsujan/FeatherNotes)

然后就被编译安装劝退了。一个是不太想这么折腾，另一个是这个软件没有提供编译好的包，感觉还是不够放心啊，毕竟如果以后开发者不维护了要再折腾一遍会非常麻烦。

## Typora，同学安利的markdown编辑工具

与多数markdown编辑工具不同，Typora是所见即所得的markdown编辑工具。为什么程序员偏爱markdown？就是因为markdown可以让我们写作的时候只关注内容本身，而不用太在意排版的问题。而Typora又改变了传统的左右分栏或者点击切换预览的传统markdown编辑模式，用起来就更舒服了。

![2.png](https://i.loli.net/2020/08/20/V7LzFNRXyT1hUbj.png)

<!--summary-->

### 安装

在Ubuntu下安装还是非常方便的：

```bash
# or run:
# sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys BA300B7755AFCFAE
wget -qO - https://typora.io/linux/public-key.asc | sudo apt-key add -
# add Typora's repository
sudo add-apt-repository 'deb https://typora.io/linux ./'
sudo apt-get update
# install typora
sudo apt-get install typora
```

其他系统参考[官网](https://www.typora.net/#download)。

### 复制图片

不过有一点要注意：我们写文档常常要插入一些图片，为了后面和git好配合，我们需要将这些图片也放到当前文件夹里。按下`Ctrl+逗号`打开设置，将图片复制到当前文件夹下。如图。

![3.png](https://i.loli.net/2020/08/20/VWcp6qEGOF9beZf.png)

### 重要的快捷键

都是重点等于没有重点。我们只要记住能让我们离开鼠标提高效率的就行了。

| 快捷键               | 功能           |
| -------------------- | -------------- |
| `Ctrl+S`             | 保存           |
| `Ctrl+Z`             | 撤销           |
| `Ctrl+Y`             | 重做           |
| `Ctrl+Shift+L`       | 侧边栏         |
| `Ctrl+/`             | 切换源代码模式 |
| 表格中，`Ctrl+Enter` | 添加一行       |

## 用git管理笔记

首先什么是git？有些少年区分不了git和github，这是姿势水平还不够啊。听说过GitLab没？听说过码云没？不是修福报的那个马云哦。

- git：当前最流行的分布式版本控制软件
- github：通过Git进行版本控制的软件源代码托管服务平台

鉴于本文并不是为了介绍这二者，本人就简单粗暴放个链接了。

https://zh.wikipedia.org/wiki/Git

https://zh.wikipedia.org/wiki/GitHub

git虽然大多用于管理程序源代码，但用来管理我们的笔记却也正好。虽然笔记对于版本控制的需求不是非常大，但聊胜于无嘛。最重要的是，你可以把笔记借助git同步到github上，什么时候换工作环境完全可以全pull下来，岂不美滋滋？

git版本库托管平台有很多，这里我并没有选择github，而是选择了[阿里云的Code平台](https://code.aliyun.com)。毕竟在国内速度比较快啊。阿里云单个仓库容量足有2G，足够我们放笔记和笔记涉及的图片了。

怎么注册账号，怎么建立仓库我就不说了。clone下来，这就是以后笔记安家的地方了。

## 自动同步笔记

然而，我们还是需要运行`git add`、`git commit`、`git push`，而且在别处修改了笔记还得手动pull一下。执行的命令这么固定，肯定是要写成脚本(.sync.sh)了。

```bash
cd /home/zero/Documents/Notes
git pull &
typora . TODO.md
git add .
git commit -m "Sync"
git push
```

注意上面的路径改为你自己笔记文件夹的路径。第二行的`&`不要省，我们没必要等pull完才打开Typora，这样可以加快一点启动速度。

好了，现在我们执行脚本就能打开Typora，而且所有编辑还能自动同步。由于git默认不允许保留空提交，没有修改的时候也不会产生大量无用的记录，完美。

最后写个桌面启动器就大功告成了：

```ini
[Desktop Entry]
Type=Application
Icon=typora
Name=笔记
Exec=/home/zero/Documents/Notes/.sync.sh
Terminal=false
Hidden=false
```

`Note.desktop`文件名保存到桌面就成了。

## Windows下的配置

Git是跨平台的，Typora也是跨平台的，那么我们这个方案自然同样可以跨平台。

> Windows下的Typora并没有默认添加环境变量，请手动添加。添加后需要重启。

首先对应上面的sh，写个cmd(.sync.cmd)：

```cmd
start /b git pull
typora . TODO.md
git add .
git commit -m "Sync"
git push
```

第一行不一样，这是因为cmd后台执行是用`start /b`的，与shell脚本的`&`效果是一样的。

此时双击这个cmd已经能看到效果了。

但cmd执行时会有黑窗口，很影响美观，我们写个vbs脚本(.sync.vbs)来调用它，这样可以不显示黑窗口。

```vbs
WScript.CreateObject("WScript.Shell").Run ".sync.cmd",0
```

最后创建个桌面快捷方式，指向`.sync.vbs`，就大功告成了。

> 注意：快捷方式的起始目录要指向笔记的目录。

---

美中不足的是，在Android上我没找到比较合适的git客户端。虽然看到有个商用的评价不错，但为了这点笔记花那么多银子还是不值得。Android上的笔记软件倒是找到一个不错的：[Markor](https://github.com/gsantner/markor)。虽然没有Typora的所见即所得那么厉害，但他的编辑界面做的很不错，配上蓝牙键盘，非常适合某些科目上课做笔记。
