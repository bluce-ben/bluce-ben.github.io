---
title: SVN 详解之客户端安装（二）
date: 2018-03-15 14:10:26
categories:
- 版本控制
- SVN
tags:
- SVN
---
#### Linux 下安装
CentOS 系统可使用yum 软件包管理器直接安装，Debian、Ubuntu 系统可使用apt-get 软件包管理器直接安装。此处，我们使用一种通用的安装方式，源码安装。
##### 1、下载及解压
下载地址：[<font color="red">传送门</font>](http://subversion.apache.org/download/)
`tar xvf subversion-1.9.7.tar.bz2`
<!--more-->

##### 2、配置文件
```
mkdir /usr/local/subersion
cd subversion-1.9.7
./configure --prefix=/usr/local/subversion  #指定安装路径
```

##### 3、解析及安装文件
```
make & make install
```

##### 4、测试及建立软链接
```
cd /usr/local/subversion/bin
svn
ln -s /usr/local/subversion/bin/svn /sbin/svn
cd /
svn
```

#### Windows下安装
##### 1、SVN客户端下载及安装
下载地址：[<font color="red">传送门</font>](https://tortoisesvn.net/downloads.html)
安装根据下一步操作即可完成...

##### 2、SVN客户端配置
这里要特别声明一下——<font color="red">SVN客户端不是指一个桌面应用程序，而是集成到系统的右键菜单中的插件。</font>因此使用客户端向资源库下载项目资源、提交项目资源等都是通过右键菜单来完成的。

在桌面空白处右键：
![](/uploads/2018/03/svn_image_01.png)

选择  设置 ，打开设置面板：
可以设置语言：
![](/uploads/2018/03/svn_image_02.png)
也可以设置 项目资源的图标，通过不同图标来指示下载到本地的项目资源文件发生了什么变化，比如：修改、新增、删除等等。


##### 3、SVN 基础操作
###### （1）检出
在你的本地项目文件夹或随便一个地方，右键空白处弹出菜单，选择 SVN检出。然后，通过从SVN服务端获取的 <font color="red">资源库URL+具体的项目文件夹名</font> 下载相应项目，并可以知道下载项目的保存位置
![](/uploads/2018/03/svn_image_03.png)

###### （2）修改及提交
1. 把项目下载到本机后，其实就是一个普通的项目文件而已，你可以在里面添加文件、修改文件、删除文件等等。
2. 提交修改
在项目文件空白处右键，选择 SVN提交。然后，输入 <font color="red">本次提交的版本更新信息（所作修改的注释）、勾选要提交的操作内容，点击 确定，即可把本机项目提交到SVN服务器资源库，覆盖掉资源库项目从而实现更新。</font>
（如果发生提交冲突，即两人都提交修改，后提交者由于版本落后会提交失败。这时可以先把自己的项目备份，然后从服务端下载最新的项目（下面有讲SVN更新），再把自己的项目覆盖到本地项目文件夹，最后SVN提交即可成功提交）
（SVN不提供历史版本功能，所以项目被覆盖后就找不回来了，所以切记备份。如果需要历史版本的保存功能，推荐使用Git）
![](/uploads/2018/03/svn_image_04.png)

###### （3）更新
如果别人修改了SVN服务端资源库上的项目，你想下载最新的项目，则在 本机项目文件空白处单击鼠标右键，选择 SVN更新 ，即可自动完成下载，并会提示所作的更新有哪些。注意：<font color="red">在原项目文件夹内选择SVN更新的话，会自动覆盖掉原有内容。建议：先备份，再更新，防止自己本来的项目内容丢失。</font>
![](/uploads/2018/03/svn_image_05.png)