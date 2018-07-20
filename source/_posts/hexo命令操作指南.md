---
title: hexo命令操作指南
date: 2017-12-06 15:34:14
categories: Hexo
tags: Hexo
---
## 基本命令
`hexo g == hexo generate`		#生成静态文件
`hexo s == hexo server`			#启动本地预览。默认情况下，访问网址为： http://localhost:4000/
`hexo d == hexo deploy`			#远程部署
`hexo n "文章标题" == hexo new "文章标题"`	#新建一篇博文
<!--more-->
## 组合命令
`hexo s -g` 	#等同于先输入 `hexo g`，再输入 `hexo s`
`hexo d -g` 	#等同于先输入 `hexo g`，再输入 `hexo d`

## 写作命令
`hexo new "postName"` 		#新建文章
`hexo new page "pageName"` 	#新建页面
`hexo new draft "filename"`	#新建草稿
`hexo publish draft "filename"` #发布草稿
**注意：`<!--more-->`之上的内容为摘要。**

## 其它命令
`hexo clean`	#清除缓存文件 (db.json) 和已生成的静态文件 (public)。
`hexo version`	#查看hexo版本号
`hexo publish filename`		#发表草稿。详解[<font color="red">传送门</font>](http://oakland.github.io/2016/05/02/hexo-%E5%A6%82%E4%BD%95%E7%94%9F%E6%88%90%E4%B8%80%E7%AF%87%E6%96%B0%E7%9A%84post/)
`hexo list [type]`		#列出网站资料 type:page、post、route、tag、category

官方命令：[<font color="red">传送门</font>](https://hexo.io/zh-cn/docs/commands.html)