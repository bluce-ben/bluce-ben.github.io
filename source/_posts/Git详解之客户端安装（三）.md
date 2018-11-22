---
title: Git 详解之客户端安装（三）
date: 2018-03-14 17:47:43
categories:
- 版本控制
- Git
tags:
- Git
---
#### 一、Linux下安装
首先，你可以试着输入git，看看系统有没有安装Git：
```
$ git
The program 'git' is currently not installed. You can install it by typing:
sudo apt-get install git
```
像上面的命令，有很多Linux会友好地告诉你Git没有安装，还会告诉你如何安装Git。
如果你碰巧用Debian或Ubuntu Linux，通过一条`sudo apt-get install git`就可以直接完成Git的安装，非常简单。
<!--more-->

如果是其他Linux版本，可以直接通过源码安装。先从Git官网下载源码，然后解压，依次输入：`./config`，`make`，`sudo make install` 这几个命令安装就好了。


#### 二、windows下安装
windows下客户端分为两种：
（1）Git客户端程序。 
　　Git目前最新版本2.16.2
　　Git官网下载地址：[<font color="red">传送门</font>](http://git-scm.com/)
（2）Git客户端图形化操作程序 TortoiseGit。 
　　TortoiseGit目前最新版本2.6.0
　　TortoiseGit官网下载地址：[<font color="red">传送门</font>](http://tortoisegit.org/download/)

安装过程可以按照程序的默认选项，都选择“下一步”安装完成。

**下面以 Git客户端程序为例来安装：**
>1.双击安装程序“Git-2.10.2-64-bit.exe”
2.点击“Next” ，根据自己的情况，选择程序的安装目录。
3.继续点击“Next”，显示截图如下：
![](/uploads/2018/03/git_image_01.jpg)
**说明：**
（1）图标组件(Addition icons) : 选择是否创建桌面快捷方式。
（2）桌面浏览(Windows Explorer integration) : 浏览源码的方法，使用bash 或者 使用Git GUI工具。
（3）关联配置文件 : 是否关联 git 配置文件, 该配置文件主要显示文本编辑器的样式。
（4）关联shell脚本文件 : 是否关联Bash命令行执行的脚本文件。
（5）使用TrueType编码 : 在命令行中是否使用TruthType编码, 该编码是微软和苹果公司制定的通用编码。
4.选择完之后，点击“Next”，开始菜单快捷方式目录：设置开始菜单中快捷方式的目录名称, 也可以选择不在开始菜单中创建快捷方式。
5.点击“Next”，显示截图如下：
![](/uploads/2018/03/git_image_02.jpg)
**设置环境变量**
选择使用什么样的命令行工具，一般情况下我们默认使用Git Bash即可：
（1）Git自带：使用Git自带的Git Bash命令行工具。
（2）系统自带CMD：使用Windows系统的命令行工具。
（3）二者都有：上面二者同时配置，但是注意，这样会将windows中的find.exe 和 sort.exe工具覆盖，如果不懂这些尽量不要选择。
6.选择之后，继续点击“Next”，显示如下：
![](/uploads/2018/03/git_image_03.jpg)
**选择提交的时候换行格式**
（1）检查出windows格式转换为unix格式：将windows格式的换行转为unix格式的换行再进行提交。
（2）检查出原来格式转为unix格式：不管什么格式的，一律转为unix格式的换行再进行提交。
（3）不进行格式转换 : 不进行转换，检查出什么，就提交什么。
7.选择之后，点击“Next”
8.选择之后，点击“Next”
9.选择之后，点击“Install”，开始安装

这样，我们的Git客户端就安装完成了。

安装完成后，在开始菜单里面能够找到 "Git --> Git Bash",如下：
![](/uploads/2018/03/git_image_04.JPG)