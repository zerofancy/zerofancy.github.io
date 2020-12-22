title: 给博客添加个RSS订阅功能
author: 归零幻想
publishDate: 2020-09-11
editDate: 2020-09-11
tags: [博客, RSS, Thymeleaf, XML]

<!--config-->

虽然RSS阅读器开始变得小众了，但我还是很喜欢的，这样我能划出每天固定的时间看我关注的博客有没有更新，看我关注的人有没有发知乎动态[^1]，这两天想起来给自己博客也加上了RSS订阅功能，虽然不见得有人看就是了。

[^1]:这个实现起来要一些奇技淫巧……

<!--summary-->

## RSS订阅的格式

按照[RSS 2.0 Specification](https://www.rssboard.org/rss-specification)的说明，就是提供个XML文件的事情。我的博客是用Thymeleaf渲染HTML的，就顺便让他渲染个XML也问题不大。

我最后用了这样的模板[^2]：

[^2]:参考了[给博客添加rss订阅](https://blog.lindexi.com/post/%E7%BB%99%E5%8D%9A%E5%AE%A2%E6%B7%BB%E5%8A%A0rss%E8%AE%A2%E9%98%85.html)和[菜鸟教程](https://www.runoob.com/rss/rss-tutorial.html)的内容。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0">
    <channel>
        <title th:text="${sitename}">林德熙</title>
        <description th:text="${rssdescription}">Windows 10 Developer</description>
        <link>[(${siteurl})]</link>
        <item th:each="i:${htmls}">
            <title th:text="${i['title']}">第一篇</title>
            <description th:text="${i['abs']}">摘要</description>
            <author th:text="${i['author']}">作者</author>
            <pubDate th:text="${i['editTime2822']}">发布时间</pubDate>
            <link>[(${siteurl}+'/'+${i['url']})]</link>
        </item>
    </channel>
</rss>
```

注意这里有些字段是用内联的方式写的，因为`th:text`失效了，也许是因为HTML没有这样的节点吧。

## 发布RSS订阅源

只看CSDN和菜鸟教程，它们告诉我只要在博客首页放个RSS的图片，设置个超链接指向xml文件就算发布了。但这样“发布”的订阅并不能被浏览器识别出来，肯定还是有没有配置正确的地方。

如图，用Firefox浏览某些博客，地址栏会提示你发现了RSS订阅：

![2020-09-11_01-32.png](https://i.loli.net/2020/09/11/kzeDmVCAQiaGHfl.png)

但我们“发布”的明显是不识别的。

最终Google提高生产力，我找到了[相关的说明](https://developer.mozilla.org/en-US/docs/Archive/RSS/Getting_Started/Syndicating)。

只要添加这样的代码就可以了。

```xml
<link rel="alternate" type="application/rss+xml" title="归零幻想" href="https://ntutn.top/rss.xml" />
```
