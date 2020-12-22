title: 禁用macOS的窗口标签
author: 归零幻想
publishDate: 2020-12-22
editDate: 2020-12-22
tags: [Apple, macOS, bug, 系统]

<!--config-->

作为一个对更新相对激进的用户，我当然是第一时间升级了最新的macOS Big Sur。说实在的，这名字给我的第一印象并不好，因为被我看成了『Big Bug』。

圆角变大了，还有我一开始比较喜欢的功能，姑且称之为窗口标签。在BigSur中，当你打开两个全屏的Android Studio，它们将出现在同一个窗口，窗口上方出现不同的标签页，和浏览器一样。

好景不长，这个功能表现很不稳定，我不得不考虑干掉这个功能。如果只是没有成功触发也就算了，大不了当没升级用，但它常常会把一些弹出窗口也搞成和原窗口并列的标签。比如当你rename一个文件时，弹出的窗口有时就会并列到标签上，然后Android Studio就卡死了。

好吧，既然它开始影响我的工作效率了，我就找了找禁用的方法[^1]。

[^1]:来自[https://www.reddit.com/r/androiddev/comments/jtbl4m/has_anyone_updated_to_macos_big_sur_and_is/](https://www.reddit.com/r/androiddev/comments/jtbl4m/has_anyone_updated_to_macos_big_sur_and_is/)

```bash

defaults write com.google.android.studio AppleWindowTabbingMode manual
```

<!--summary-->
