title: 在github pages部署你的博客
author: 归零幻想
publishDate: 2020-08-21
editDate: 2020-08-21
tags: [github, 博客]

<!--config-->

废话不多说，直接开搞。

## 部署

1. 在github创建一个仓库，名为`[uzername].github.io`，如`zerofancy.github.io`
2. 将你的网页提交到这个仓库。
3. 直接访问`[username].github.io`

## 绑定域名

1. 将你的域名@和www以CNAME方式绑定到`[username].github.io`
2. 在你的仓库根目录提交一个`CNAME`文件，内容为你的域名，如`ntutn.top`
3. 在仓库的设置中找到`Custom domain`，填入并保存。如果下面的`Enforce HTTPS`不可用，清除`Custom domain保存并重新设置一次试试。

<!--summary-->
