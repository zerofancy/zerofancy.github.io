title: 在Windows10上安装血战上海滩
author: 归零幻想
publishDate: 2019-06-01
editDate: 2019-06-01
tags: [游戏, 纪念]

<!--config-->

《血战上海滩》是由北京欢乐亿派科技有限公司开发的单机FPS游戏，发行于2003年，是一个非常经典的老游戏了。
近来我打算把这个游戏找出来再玩一遍，可是在Windows10上安装还是有些问题……

![血战上海滩](/res/img/shanghai1.png "血战上海滩")

<!--summary-->

## 下载
下载不用多说，自己找资源吧，一共三百多兆的单机游戏。

> 现在很多单机游戏下载时网站都会给你带上一个“启动器”来显示广告，恶心的很。找到文件夹中，注意血战上海滩的游戏文件是一个帽子的图标，大小为`1.87M`，SHA1为`FE5A102AA9DE633FB6E388EAAEEA38BC43E8E7E4`

## 启动
直接双击exe，然后……它免费帮你调了一下分辨率，就没反应了

![直接双击，只有分辨率变化](/res/img/shanghai2.png "直接双击，只有分辨率变化")

貌似兼容性是个比较大的问题……于是在网上找了好久，终于知道，可以用命令行控制游戏窗口运行：

```cmd
shanghai.exe -windows
```

![窗口化运行](/res/img/shanghai3.png "窗口化运行")

## 修改屏幕分辨率实现全屏

倒是能运行了，只是……画面都在左上角，没法玩啊……

所以再加上自动修改屏幕分辨率就可以全屏了。修改屏幕分辨率可以用`setres`

[下载链接](https://www.majorgeeks.com/files/details/setres.html)下载并将exe文件放到血战上海滩的游戏文件夹。

于是用下面的命令：

```cmd
setres h800 v600
shanghai.exe -windows
setres h1920 v1080
```

`1920*1080`是我电脑的屏幕分辨率，注意换成自己的。

## 隐藏显示任务栏

然而，这样还是有问题，就是这样“全屏”后任务栏并没有消失，虽然可以设置自动隐藏任务栏但总归还是不爽，于是写了个简单C#程序在启动游戏隐藏任务栏（虽然很简单，但没找到只用命令实现的方法）

```C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Runtime.InteropServices;

namespace ShowTaskBar
{
    class Program
    {
        private const int SW_HIDE = 0;  //隐藏任务栏
        private const int SW_RESTORE = 9;//显示任务栏
        [DllImport("user32.dll")]
        public static extern int ShowWindow(int hwnd, int nCmdShow);
        [DllImport("user32.dll")]
        public static extern int FindWindow(string lpClassName, string lpWindowName);
        static void Main(string[] args)
        {
            ShowWindow(FindWindow("Shell_TrayWnd", null), SW_HIDE);

        }
    }
}

```

```C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Runtime.InteropServices;

namespace ShowTaskBar
{
    class Program
    {
        private const int SW_HIDE = 0;  //隐藏任务栏
        private const int SW_RESTORE = 9;//显示任务栏
        [DllImport("user32.dll")]
        public static extern int ShowWindow(int hwnd, int nCmdShow);
        [DllImport("user32.dll")]
        public static extern int FindWindow(string lpClassName, string lpWindowName);
        static void Main(string[] args)
        {
            ShowWindow(FindWindow("Shell_TrayWnd", null), SW_RESTORE);

        }
    }
}

```

编译并将生成文件也放到游戏文件夹。这样我们就可以用一个`start.cmd`打开游戏：
```cmd
setres h800 v600
hidetaskbar
shanghai.exe -windows
showtaskbar
setres h1920 v1080
```

体验相当完美。
