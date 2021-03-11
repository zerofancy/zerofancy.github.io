title: 关于
author: 归零幻想
publishDate: 2020-07-25
editDate: 2020-12-09
tags: [博客, CMS]

<!--config-->

## 归零幻想

<img alt="归零幻想" src="/res/img/avatar.jpg" width="200px">

---

归零幻想是本人的网名啦，这里是我的博客小站。本人在其他地方帐号大多也叫这个名字。

博客系统至今我写过三个版本[^1]，毕竟写出个CRUD就可以厚着脸皮说自己是个博客了。当前版本最初是个Kotlin练手项目，源代码发布在[github](https://github.com/zerofancy/kmdblog)。不过也就自用罢了，不会有人对我的代码感兴趣吧……

<!--summary-->

[^1]: 第一个版本是很遥远的过去了，[第二个版本](https://github.com/zerofancy/blogSystem)是用springboot写的，颇有杀鸡用牛刀之嫌。一方面不想继续交服务器租金了，另一方面原来部分代码写得很烂，当真懒得改了。加上新学了Kotlin，正想练手，这就有了[第三个版本](https://github.com/zerofancy/kmdblog)。

站点结构的设计上其实有一点小失误，就是文章比较多后根目录会比较乱，于是我把之前发布的文章*归档*到`archieved`文件夹中，于是文章链接还会改变一次，之前分享出去的/被搜索引擎爬的还会失效。

## 联系方式

- E-mail:[ntutn.top@gmail.com](mailto:ntutn.top@gmail.com)
- Telegram:[@zerofancy](https://t.me/zerofancy)

## 采用的工具、技术

本网站开发部署离不开以下工具技术和第三方组件的支持。

|技术|说明|链接|
|---|---|---|
|Bootstrap|前端模板。|[Bootstrap](https://getbootstrap.com/)|
|commons-io|&nbsp;|[commons-io](https://commons.apache.org/proper/commons-io/)|
|dom4j|XML解析。|[dom4j](https://dom4j.github.io/)|
|flexmark-java[^2]|java版的markdown渲染引擎。|[flexmark-java](https://github.com/vsch/flexmark-java)|
|github-markdown-css|仿githubmarkdown样式。|[github-markdown-css](https://github.com/sindresorhus/github-markdown-css)|
|github pages|本站部署在Github Pages上。|[Github Pages](https://pages.github.com/)|
|公益 404|公益404页面是由腾讯公司员工志愿者自主发起的互联网公益活动。|[公益 404](https://www.qq.com/404/)|
|Hitokoto|一言。|[Hitokoto](https://hitokoto.cn/)|
|jQuery|&nbsp;|[jQuery](https://jquery.com/)|
|Kotlin|Kotlin是一种在Java虚拟机上运行的静态类型编程语言，本站来自一个Kotlin练手项目。|[Kotlin](https://kotlinlang.org/)|
|kotlin-argparser|一个Kotlin编写的解析命令行参数的工具。|[kotlin-argparser](https://github.com/xenomachina/kotlin-argparser)|
|kotlinpoet|用Kotlin生成Kotlin代码。|[kotlinpoet](https://square.github.io/kotlinpoet/)|
|Markdown|markdown是一种轻量级的文本标记语言，它可以让人们更关注要写的内容而不是文章的排版和格式。本站文章用markdown编写。|[Markdown](https://zh.wikipedia.org/wiki/Markdown)|
|Slf4J|日志门面，Thymeleaf要用到，只保留了空实现。|[Slf4j](http://www.slf4j.org/)|
|Thymeleaf|优雅的java HTML模板引擎。|[Thymeleaf](https://www.thymeleaf.org/)|
|unsplash|使用了unsplash提供的api实现背景图片。|[Unsplash](https://unsplash.com/)|
|utterances|用Github Issues做评论功能的项目，配置简单，用起来也舒服。|[utterances](https://utteranc.es/)|
|Vali Admin|使用了其样式文件和插件。|[Vali Admin](https://github.com/pratikborsadiya/vali-admin)|
|zoomify|点击进行图片缩放。|[zoomify](https://github.com/indrimuska/zoomify)|

文章参数的保存参考了 [Hexo](https://hexo.io/zh-cn/) 的设计。

[^2]:flexmark-java对于我这个小项目实在太庞大了，因此只引用了其中的一部分组件。具体引入的插件有：表格插件`TablesExtension`、目录插件`TocExtension`、删除线插件`StrikethroughExtension`、脚注插件`FootnoteExtension`、任务列表插件`TaskListExtension`

博客首页『阅读全文』改用ajax加载全文，但脚注、目录等显示异常且无法评论。

## 版权和知识共享

本站主页有称作『一言』的功能，其数据来自[https://hitokoto.cn/](https://hitokoto.cn/)，非本站原创，且不在本站服务器上存储，标注的作者也获取自该网站提供的API。若有侵权，请自行与该站站长协商。

在未另行说明的情况下，本站内容遵守

<a href="https://creativecommons.org/licenses/by-sa/4.0/deed.zh"><img alt="CC-BY-SA" src="/res/img/cc-by-sa.png">&nbsp;署名-相同方式共享 4.0 国际 (CC BY-SA 4.0) </a>。