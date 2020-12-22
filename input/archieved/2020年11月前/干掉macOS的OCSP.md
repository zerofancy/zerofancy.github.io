title: 干掉macOS的OCSP
author: 归零幻想
publishDate: 2020-11-15
editDate: 2020-11-15
tags: [Apple, macOS, 隐私]

<!--config-->

苹果这两天摊上事了，有不少用户说自己的设备打开应用程序会卡好几分钟，然后分析发现是苹果的OCSP校验导致的。

当启动一个新应用程序的时候，系统会把其hash发送到`ocsp.apple.com`用于校验，而这次是这个服务挂了但是能ping通……

这个事情引起不小的讨论，主要集中在有关隐私的担忧上。本来我没有太在意，**但测试发现这好像是我每次休眠恢复后触摸板卡几秒的元凶……**

那对不起了。

```sh
echo "127.0.0.1 ocsp.apple.com" | sudo tee -a /etc/hosts
```

反正还有SEP把门呢。

<!--summary-->
