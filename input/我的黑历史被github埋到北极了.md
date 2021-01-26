title: 我的黑历史被github埋到北极了
author: 归零幻想
publishDate: 2021-01-26
editDate: 2021-01-26
tags: [纪念, github, 代码]

<!--config-->

昨天整理自己的github仓库的时候发现自己的个人主页多了个徽章：`Arctic Code Vault Contributor`

![QQ20210126-143812@2x.png](https://i.loli.net/2021/01/26/CJOcmPn12o734KQ.png)

So what happened?

<!--summary-->

## GitHub Code Vault是什么

> GitHub Code Vault（GitHub 代码保险库）是由 GitHub Archive Program（GitHub 代码永久保存计划）设立的代码档案库，旨在保存开源软件以供未来使用。[^1]

[^1]:[GitHub Code Vault 指南](https://github.com/github/archive-program/blob/master/GUIDE_zh.md)

天灾人祸，世界末日，地球回到原始时代重新发展，或者[betacat](https://code2048.com/series/betacat/)污染了所有开源代码库啦，人类就可以找到这份代码库，重建网络世界。

该项目给github上活跃的开源代码库建立`快照`，将他们存储在胶片上运往北极。据称这些代码将被保存至少1000年。

每个细节都充满了科幻的味道，比如考虑一前年后的人类不一定读懂今天的代码，在[《GitHub Code Vault 指南》](https://github.com/github/archive-program/blob/master/GUIDE_zh.md)中介绍了二进制、计算机、软件、编译等基本概念，以及如何解码胶片上的信息。 **当然，这个指南本身并没有被压缩和编码，他们没有犯某些网站的`RARSetup.rar`的错误。**

考虑到地球文明重建，人类不一定造出了计算机，于是还放了一份[科技树](https://github.com/github/archive-program/blob/master/TheTechTree.md)指南，包含理解软件所需的多层技术基础，如微处理器、网络、电子、半导体，甚至工业社会前的技术。有了这些技术，让人类可以重新造出现代计算机。

## 我的被选中的代码库
### ctodo

[https://github.com/zerofancy/ctodo](https://github.com/zerofancy/ctodo)

一个终端todo工具。c不是指C语言，而是`console`（事实上这个项目是java写的）。

项目代码本身写的很垃圾，也就这个主意有点意思。在终端直接管理TODO List，听上去不错，但我没有坚持用太久。后来还是觉得有GUI的todo工具更好，比如微软的待办事项。

然而即使是这个主意，我也找到了更好的实现：[devtodo](https://swapoff.org/devtodo.html)，所以我的那个仓库也就不再维护了。

### noveldownloader

[https://github.com/zerofancy/noveldownloader](https://github.com/zerofancy/noveldownloader)

一个用Java+selenium写的小说下载器，基本原理就是模仿用户不断点“下一页”

没错，这个项目就是我看小说不想买正版又忍受不了盗版的广告时写的。当然因为是自用所以代码写的很随意，这不说放到1000年后，就是一两年后也是妥妥的黑历史啊……

## 吃瓜
### Dress被选中了

[你们的照片现在已经被冰封在北极了。。。](https://github.com/komeiji-satori/Dress/issues/910)

女装一时爽，破照留千年。坦白讲，除了公司的项目，我克隆过的最大的项目就是[Dress](https://github.com/komeiji-satori/Dress)

虽然该项目会忽略大于100KB的二进制文件，但这个限制会随着star数逐渐解除。考虑到这个项目的star数……为1000年后的考古学家的头发默哀3秒。

### 996.icu被选中了

[https://996.icu/#/zh_CN](https://996.icu/#/zh_CN)

我的代码留点bug给后世解决不算什么，但996.icu也被选中了。这下真的被钉到历史的耻辱柱上了……

### 面向Stackoverflow编程

不同时快照一份 stackoverflow ，后代也不会抄啊。[^2]

[^2]:[如何看待 GitHub 将公共存储库快照保存到北极地下？ - est的回答 - 知乎](https://www.zhihu.com/question/355902523/answer/894681961)

### 大家还贡献了什么

dotfile、Github Pages、个人演讲库和个人网站、给女朋友的信，听说鸿蒙的ppt也在里面……

## 参考

- [除了bug，GitHub可能还把你的女装照冻到了北极，1000年后还能读那种](https://zhuanlan.zhihu.com/p/162808193)
- [https://archiveprogram.github.com/](https://archiveprogram.github.com/)
- [GitHub Archive Program: the journey of the world’s open source code to the Arctic](https://github.blog/2020-07-16-github-archive-program-the-journey-of-the-worlds-open-source-code-to-the-arctic/)