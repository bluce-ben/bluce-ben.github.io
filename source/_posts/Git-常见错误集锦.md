---
title: Git 常见错误集锦
date: 2018-07-21 18:40:22
categories:
- [版本控制, Git, Git问题]
tags:
- Git
- Git问题
---
#### 一、git add出现 "in unpopulated submodule 'A' " 问题 ####
**问题场景：**
我博客Hexo+GitHub中用到了Next主题，而Next主题是我直接从GitHub上下载使用的。因此我Hexo博客中有.git文件，而Next主题也有.git文件，且这两者文件含义不同。因此，在我git push的时候，报此错误。
<!--more-->

**原因解释：**
在 ./ 下有一文件夹 命名为“A”，A/ 有之前建立的仓库，我在 ./ 下add commit push 后发现远程仓库内并没有A/的内容，于是我在 A/ 下执行 ”git add .” 提示：“in unpopulated submodule ‘A’ ”（翻译为”在一个无人居住的子模块“，感觉意思是说位于子模块下，无法 add 0.0） 

**解决方法是：**
（1）删除 A/ 的.git 文件夹
（2）在 ./ 下输入”git rm -rf –-cached A/“ //谨记：是 A/ ，意为A目录下
（3）在 ./ 下输入”git add A”
（4）git commit -m “”
（5）git push origin master


#### 二、GitHub上提示package-lock.json中依赖文件需要更新
**问题场景：**
将文章更新到GitHub上时，偶然间发现提示一个Warning！package-lock.json 依赖文件有漏洞，提示更新。一下就蒙圈了，package-lock.json是啥？怎么更新？
经过查找发现，package-lock.json 是npm的包管理文件，即你下载的npm包都记录在里面。
那需要怎么更新呢？度娘说直接使用“npm install” 即可，会自动的从package.json里面更新对应包，尝试失败。

**解决方法：**
直接在Hexo根目录下使用命令：
`npm install packagename`
直接填写对应包名，更新即可。

