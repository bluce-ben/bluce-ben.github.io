---
title: windows下使用Hexo+导入Github博客
date: 2017-12-06 11:26:28
categories: Hexo
tags: Hexo
---
## 准备工作
### 1、安装Node.js
下载地址：[<font color="red">传送门</font>](https://nodejs.org/en/)
去 NodeJs 官网下载相应版本，进行安装即可。 
可以通过node -v的命令来测试NodeJS是否安装成功
<!--more-->
### 2、安装Git
下载地址：[<font color="red">传送门</font>](https://git-scm.com/)
去 Git 官网下载相应版本，进行安装即可。
可以通过git –version的命令来测试git是否安装成功

### 3、注册Github账号（如果已有GIthub账号可忽略）
去 Github 官网进行注册即可。 
注册完之后记得添加 SSH Key。 
这个 SSH Key是一个认证，让github识别绑定这台机器，允许这台机器提交。
在Git Bash执行如下命令：
`cd ~/. ssh`
~这个符号，表示在用户目录下 
执行代码如果提示：No such file or directory 说明你是第一次使用git。 
下面就说下怎么配置SSH Key，生产新的SSH Key配置（windows）。
在Git Bash执行代码：
`ssh-keygen -t rsa -C "zheng_benwu@163.com"`
记得修改成你自己邮箱地址。 
成功后会在家目录下生成一个隐藏文件夹.ssh，里面包含id_rsa 以及id_rsa.pub。

### 4、添加SSH Key到Github
打开刚才生成的文件id_rsa.pub文件，复制其中的公钥。
之后在Github添加SSH Key，在任意界面右上角，点击你的头像，选择Settings-> SSH keys->New SSH key
把公钥添加进去就ok了。
**至此，本地已经将环境搭建完成。**

## 搭建博客
### 1、安装Hexo
在本地新建一个Blog文件夹，文件右键，选择Git Bash（进入Bash界面）。
输入指令安装hexo（安装hexo命令）： 
`npm install -g hexo`
等安装完毕，通过输入hexo的命令来测试hexo是否安装成功。
接着初始化Hexo（安装Hexo博客）：
`hexo init hexo`
初始化成功会显示Start blogging with Hexo! 
这时在你刚才创建的Blog里面会多出一个hexo文件。
安装完成！

### 2、安装依赖文件
接着，进入到hexo目录，输入指令npm install，安装依赖文件以及部署形成文件。
安装依赖文件：
```
cd hexo
npm install
```
部署形成文件（生成静态文件）：
`hexo generate`

### 3、测试
启动server服务：
`hexo server`
这时提示Hexo is running at http://loalhost:4000/. 
接着我们打开浏览器，输入http://localhost:4000/便可看到默认的博客。
到这里，Hexo已经安装完毕。

## 配置Github Page
### 1、新建博客仓库
登录Github，点击”New repository”，新建一个版本库 
输入仓库名：你的Github名称.github.io。然后点击Create repository。
<font color="red">注意：这边的创建名字，一定要用的github的用户名，不然显示不出来，因为githubPage只能你的用户名。 </font>

### 2、启动GitHub Pages
点击右边的“Setting”菜单进入设置,点击”Choose a theme”，即可选择一个模板。
保存发布即可。 
到这里，可通过 https://bluce-ben.github.io 打开自己创建的静态站点模板。

## 将本地hexo项目托管到Github
### 1、修改配置文件
打开修改hexo目录下配置文件_config.yml（*网站配置文件*）。
编辑最后面的deploy属性，修改如下代码：
```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: https://github.com/bluce-ben/bluce-ben.github.io.git
  branch: master
```
**注意：此处type为git，不是github；repository仓库使用https形式的，不要使用SSH形式的。**

### 2、安装hexo-deployer-git插件
此插件的作用就是关联hexo与git部署问题，当使用`hexo deploy`部署到git上时报错，可能就是缺少该插件。
执行命令：
`npm install hexo-deployer-git --save`

### 3、部署你本地的主题到Github上
执行命令：
```
hexo clean
hexo generator #简写 hexo g
hexo deploy #简写 hexo d
```
此处命令可简写为：
```
hexo clean
hexo d -g
```
到此，就将本地Hexo博客与GitHub Pages关联成功。


## 异地管理hexo
首先，需要在这里说明一下我的问题。起初建博客的时候，当我将本地的hexo部署到GitHub上时，发现GitHub上版本库中的数据与本地是不一致的，GitHub上仅仅只是保存了页面显示部分，即.deploy_git下的内容（将本地部署到GitHub上的部分，命令为`hexo d`）。这并不是我想要的，当初没有异地管理的需求，就把这事忘了。

而此时，需要在异地管理，因此就把过程记录下来。
* 如果在搭建Hexo+GitHub Page的初始阶段，可阅读 https://blog.csdn.net/zwx2445205419/article/details/66970640 该篇文章。
* 如果是在后期新增异地管理的需求，可继续往下阅读。原理类似。

### 一、创建hexo文件存放分支
1、创建分支
![](/uploads/2018/07/hexo_01.png)

2、将hexo分支设为默认分支
![](/uploads/2018/07/hexo_02.png)
（注：设置为默认分支之后，下次再执行clone的时候，导入到本地的是hexo分支。）

### 二、将hexo分支仓库clone到本地，并清空分支数据
1、将分支clone到本地，并把数据清空，注意要保留` .git `文件，因为` .git `是版本库的信息。clone仓库的目的就是要取得 ` .git `文件。没有该文件，就不能对hexo分支进行管理了。

2、将清空后的hexo分支推到GitHub，即将GitHub上hexo分支数据清空。（注：此分支清空并不影响master分支数据）


### 三、将本地hexo文件数据添加到hexo分支版本下，并推到GitHub
1、将本地hexo文件数据中的` .git `文件删除，使用自己 clone下来的hexo分支中的 ` .git `文件。

2、然后把本地的hexo文件数据（不包含.git文件，否则两个会有冲突，一个文件夹下只能有一个同名文件）放到hexo分支下，并进行访问，查看是否可以使用。
```
hexo clean
hexo g
hexo s
```

3、之后，进行部署。访问博客是否修改成功。（注：可通过增加一篇文件测试）
注：hexo文件下 `_config.yml`中 deploy参数如下：
```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: git@github.com:xxx.github.io.git
  branch: master
```
仍然是部署到 master主分支。
开始部署：
`hexo d`

4、最后，再将本地的hexo分支同步到GitHub上的hexo分支。
```
git add .
git commit -m ''
git push origin hexo
```
可登录GitHub查看 hexo分支是否有变化。

### 四、异地clone hexo分支进行管理
异地通过 `git clone git@github.com:xxx.github.io.git`进行导入本地进行管理。
注：每次进行本地分支管理前，可使用`git pull`来同步分支数据。以防在其它地方进行了更改数据，造成数据冲突。


**原理说明：**
其实，说白了就是有两个分支，一个主分支用来存放GitHub Page页面，一个hexo分支用来存放本地hexo文件（即博客文件）。
异地管理的时候，导出hexo文件即可。当部署文件的时候就自动的部署到主分支的GitHub Page。然后再执行命令`git push origin hexo`，将本地hexo文件推到`origin/hexo`分支，用于其他电脑更新分支信息。
注：相当于比以前多了一步，即管理本地的hexo分支。
