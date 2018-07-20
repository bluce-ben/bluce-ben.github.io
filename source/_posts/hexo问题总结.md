---
title: hexo问题总结
date: 2017-12-06 17:11:37
categories: 
- [Hexo, Hexo问题]
tags: 
- Hexo
- Hexo问题
---
### 博客栏目分类、标签不显示内容
问题描述：
网站栏目通过 `hexo new page 'pagename'` 新增了分类、标签、关于等页面。
然后陆陆续续写了几篇文章。但是，点击分类、标签页面确没有生成对应的分类和标签信息。
解决方案：
需要了解，通过 `hexo new page 'pagename'` 生成的文件内容（以tags为例）：
<!--more-->
首先执行命令：
　`hexo new page 'tags'` 
生成页面，之后再更改生成页面的内容
```
---
title: tags
date: 2017-12-05 18:47:42
type:
---
```
改为：
```
---
title: TagCloud
date: 2017-12-05 18:47:42
type: "tags"
comments: false	//多说或者Disqus评论
---
```
再次生成静态文件，访问成功。
同理，分类页面照此修改即可！

***

### hexo部署后，CNAME会被自动删除，怎么办？
*（自己的独立域名需要用CNAME连接到github pages）*
解决方案：
将需要上传至github的内容放在source文件夹，例如CNAME、favicon.ico、images等，每次上传就不会消失了

***

### Hexo 部署到GitHub报错：fatal: could not read Username for 'https://github.com': No error
执行 `hexo d` 部署时报错，信息如下：
![](/uploads/2018/03/hexo_error_01.JPG)

由错误信息可知，GitHub 不能够读取 Username。因此我验证了一下本地的SSH KEY是否能够连接到GitHub，执行命令：
```
$ ssh -T git@github.com
Hi bluce-ben! You've successfully authenticated, but GitHub does not provide shell access.
```
出现上面的语句说明你的ssh key已经配置好了，并没有错误，能够与 GitHub 连通。 从网上查询说是使用 ssh 登录方式来代替 https，因此进行了代码调整，如下：
```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  #repository: https://github.com/bluce-ben/bluce-ben.github.io.git
  repository: git@github.com:bluce-ben/bluce-ben.github.io.git
  branch: master
```
然后，再次执行 `hexo d` 命令部署成功！