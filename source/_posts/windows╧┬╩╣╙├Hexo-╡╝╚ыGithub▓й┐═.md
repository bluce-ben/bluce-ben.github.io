---
title: windows下使用Hexo+导入Github博客
date: 2017-12-06 11:26:28
categories: Hexo
tags: Hexo
---
# 准备工作
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

# 搭建博客
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

# 配置Github Page
### 1、新建博客仓库
登录Github，点击”New repository”，新建一个版本库 
输入仓库名：你的Github名称.github.io。然后点击Create repository。
<font color="red">注意：这边的创建名字，一定要用的github的用户名，不然显示不出来，因为githubPage只能你的用户名。 </font>

### 2、启动GitHub Pages
点击右边的“Setting”菜单进入设置,点击”Choose a theme”，即可选择一个模板。
保存发布即可。 
到这里，可通过 https://bluce-ben.github.io 打开自己创建的静态站点模板。

# 将本地hexo项目托管到Github
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

# 导入Github
待补充...