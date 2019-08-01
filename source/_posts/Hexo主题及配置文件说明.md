---
title: Hexo主题及配置文件说明
date: 2017-12-06 14:23:03
categories: Hexo
tags: Hexo
---
## Hexo主题
本网站使用的是 Next 主题，以此为例来说明如何配置主题。
Hexo官方主题：[<font color="red">传送门</font>](https://hexo.io/themes/)
Next主题：[<font color="red">传送门</font>](https://github.com/iissnan/hexo-theme-next)
### 1、下载主题
直接使用Git克隆到本地指定目录即可：
```
cd your-hexo-site
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```
<!--more-->
### 2、启用主题
进入Hexo目录，打开*站点配置文件*，找到theme字段，并将值改为 `next`。
`theme: next`
到此，NexT 主题安装完成。下一步我们将验证主题是否正确启用。在切换主题之后、验证之前， 我们最好使用 `hexo clean` 来清除 Hexo 的缓存。

### 3、验证主题
启动服务：
`hexo s`
此时即可使用浏览器访问 http://localhost:4000，检查站点是否正确运行。

### 4、主题设定
通过打开主题目录下面的*主题配置文件*修改即可。
1、选择 Scheme
```
# ---------------------------------------------------------------
# Scheme Settings
# ---------------------------------------------------------------

# Schemes
# scheme: Muse
# scheme: Mist
# scheme: Pisces
scheme: Gemini
```
2、设置语言
编辑*站点配置文件*，将 language 设置成你所需要的语言。例如选用简体中文，配置如下：
`language: zh-Hans`
3、设置菜单
编辑*主题配置文件*，修改以下内容：
```
menu:
  home: / || home
  categories: /categories/ || th
  tags: /tags/ || tags
  archives: /archives/ || archive
  about: /about/ || user
  #schedule: /schedule/ || calendar
  #sitemap: /sitemap.xml || sitemap
  commonweal: /404/ || heartbeat
```
此处 || 之后代表的是 Font Awesome 图标的名字。
**注意：如果上面设置了简体中文，而此处配置了相应栏目，而网站显示仍然是英文。那就是主题目录下的`languages/{language}.yml`语言包没有对应的翻译，需要手动添加即可。**
4、设置头像
编辑*主题配置文件*， 修改字段 avatar， 值设置成头像的链接地址。
头像的链接地址可以是：①完整的互联网URI（绝对路径） ②站点内的地址（相对路径）
`avatar: /images/IMG_2796.JPG`
**以上就是我自己修改的配置，如果还想了解更多配置详情，可到Next官网查看[Next](http://theme-next.iissnan.com/)**

### 5、插件安装
1、百度统计：[传送门](http://tongji.baidu.com/)
2、阅读次数统计：[传送门](https://leancloud.cn/)

## 配置文件
### 1、站点配置文件
站点配置：[传送门](https://hexo.io/zh-cn/docs/configuration.html)

### 2、主题配置文件
Next主题配置：[传送门](http://theme-next.iissnan.com/theme-settings.html)

## RSS
### 1、RSS是什么？
　　简易信息聚合（也叫聚合内容）是一种RSS基于XML标准，在互联网上被广泛采用的内容包装和投递协议。RSS(Really Simple Syndication)是一种描述和同步网站内容的格式，是使用最广泛的XML应用。RSS搭建了信息迅速传播的一个技术平台，使得每个人都成为潜在的信息提供者。发布一个RSS文件后，这个RSS Feed中包含的信息就能直接被其他站点调用，而且由于这些数据都是标准的XML格式，所以也能在其他的终端和服务中使用，是一种描述和同步网站内容的格式。RSS可以是以下三个解释的其中一个： Really Simple Syndication；RDF (Resource Description Framework) Site Summary； Rich Site Summary。但其实这三个解释都是指同一种Syndication的技术。
　　RSS目前广泛用于网上新闻频道，blog和wiki，主要的版本有0.91, 1.0, 2.0。使用RSS订阅能更快地获取信息，网站提供RSS输出，有利于让用户获取网站内容的最新更新。网络用户可以在客户端借助于支持RSS的聚合工具软件，在不打开网站内容页面的情况下阅读支持RSS输出的网站内容。
　　说白了，RSS就是获取订阅的feed流，且是最新的。
### 2、如何使用RSS？
　　RSS就是个数据源，要订阅RSS，就必须先知道RSS的地址。一般来说，各个网站的首页都会用显著位置标明。名称可能会有些不同，比如RSS、XML、FEED，大家知道它们指的都是同样的东西就可以了。有时RSS后面还会带有版本号，比如2.0、1.0，甚至0.92，这个不必理会，它们只是内部格式不同，内容都是一样。
　　获取到数据源之后，内容是一堆程序语言，想要转换成正常形式阅读，就需要阅读器。RSS的阅读器多种多样，大致分为两种，一种是桌面型的，需要安装；另一种是在线型，直接使用浏览器进行阅读。


## Hexo 增加阅读数
下面介绍两种方式：LeanClound 和 不蒜子统计。
### （1）LeanClound 统计
<font color="red">注：配置LeanClound统计需要提前配置好HTTPS协议。可参考下面的说明。</font>
1. 注册LeanClound账号
2. 创建应用 -> 可获取到APP ID和APP Key。（用于Hexo主题配置文件。）
3. 创建Class -> 用于专门保存博客的文章访问量等数据。此处Class名称必须为 **Counter**。
4. 添加安全域名 -> 只有添加了安全域名，才有权访问后台的数据。（注：此处安全域名需要使用HTTPS协议的域名）
5. 修改主题配置文件 -> 即把2步骤获取的APP ID和APP Key添加进去，并开启。

至此，重新部署代码后生效，不需要自己添加样式或加载JS等操作。

### （2）不蒜子统计
直接打开 themes/_config.yml 文件，搜索 “busuanzi_count” 即可找到主题中默认已经配置了不蒜子统计。
你要做的就是：
1. 开启不蒜子： `enable: true`
2. 进入 http://ibruce.info/2015/04/04/busuanzi/ 网站，依据上面说明更改对应的域名即可。

>更改域名：
如果依据网站说明的地址 `themes/你的主题/layout/_partial/footer.ejs` 找不到对应的文件。
可直接在“你的主题”路径下面直接搜索：`grep -r 'busuanzi' ./*` 即可找到对应的主题默认加载的不蒜子JS，更改其域名即可。

至此，重新部署代码即可生效。


## Hexo http升级到https
基于Hexo + GitHub Page搭建的个人博客，配置https超级简单。两步搞定：
1、进入GitHub Page所在的Repository，点击Settings
![](/uploads/2018/10/hexo_https_01.png)

2、在Options（默认）选项下方找到GitHub Page一栏，如下图所示，勾选 Enforce HTTPS设置即可。
![](/uploads/2018/10/hexo_https_02.png)


## Hexo 增加搜索功能
### 安装插件
`npm install hexo-generator-searchdb --save`

### 修改配置文件
**修改 站点配置 文件**
```
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```

**修改 主题配置 文件**
```
local_search:
  enable: true
```