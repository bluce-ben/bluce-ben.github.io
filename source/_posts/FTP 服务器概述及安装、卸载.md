---
title: FTP 服务器概述及安装、卸载
date: 2018-03-19 11:18:10
categories:
- 服务器
- FTP
tags:
- FTP
---
#### 1、FTP概述
FTP（文件传输协议）是一个较老且最常用的标准网络协议，用于在两台计算机之间通过网络上传/下载文件。
FTP采用客户/服务器模式，客户机与服务器之间利用TCP建立连接，客户可以从服务器上下载文件，也可以把本地文件上传至服务器。
FTP服务器有匿名的和授权的两种。匿名的FTP服务器向公众开放，用户可以用“ftp”或“anonymous”为帐号，用电子邮箱地址为密码登录服务器；授权的FTP服务器必须用授权的账户名和密码才能登录服务器。通常匿名的用户权限较低，只能下载文件，不能上传文件。
客户机访问FTP服务器通常有两种方法：用FTP命令访问和用FTP客户端软件访问。

然而， FTP 最初的时候并不安全，因为它仅通过用户凭证（用户名和密码）传输数据，没有进行加密。如果你打算使用 FTP，需要考虑通过 SSL/TLS 配置 FTP 连接。否则，使用安全 FTP，比如 SFTP 会更好一些。
<!--more-->

#### 2、知识点介绍
##### （1）FTP采用双TCP 连接方式
>**控制连接-使用TCP端口号21**
　　用于在FTP客户端和FTP服务器之间传输FTP控制命令及命令执行信息。控制连接在整个FTP会话期间一直保持打开。
**数据连接-使用TCP端口号20**
　　用于传输数据，包括数据上传、下载、文件列表发送等。数据传输结束后数据连接将终止。

##### （2）FTP传输方式
>FTP主动数据传输方式
主动方式也称为PORT方式，是FTP协议最初定义的数据传输连接方式，主要特点是：
* FTP客户端通过向FTP服务器发送PORT命令，告诉服务器该客户端用于传输数据的临时端口号（大于1024的随机端口）
* 当需要传送数据时，<font color="red">服务器通过TCP端口号20与客户端的临时端口建立数据传输通道</font>，完成数据传输

在建立数据连接的过程中，由服务器主动发起连接，因此被称为主动方式。
>FTP被动数据传输方式
被动方式也称为PASV方式，被动方式的主要特点是：
* FTP客户端通过向FTP服务器发送PASV命令，告诉服务器进入被动方式。<font color="red">服务器选择临时端口号(1024~5000之间的随机端口)并告知客户端</font>
* 当需要传送数据时，客户端主动与服务器的临时端口号建立数据传输通道，完成数据传输。

在整个过程中，由于服务器总是被动接收客户端的数据连接，因此被称为被动方式。

##### （3）FTP用户的类型
**匿名用户：**anonymous或ftp
**本地用户：**
　　帐号名称、密码等信息保存在passwd、shadow文件中
**虚拟用户：**
　　使用独立的帐号/密码数据文件
　　user\_list zhangsan 123456 /var/pub


#### 3、在Centos 7中安装FTP 服务器
非常简单，使用 yum只需要一条命令即可。
`yum -y install vsftpd`


#### 4、卸载FTP 服务器
① 查看当前服务器中的vsftpd 包
`rpm -qa|grep vsftpd `
例如结果为：vsftpd-2.2.2-13.el6\_6.1.x86\_64

② 执行卸载
`rpm -e vsftpd-2.2.2-13.el6_6.1.x86_64`
返回：卸载时自动备份vsftp的用户列表文件
>warning: /etc/vsftpd/vsftpd.conf saved as /etc/vsftpd/vsftpd.conf.rpmsave
warning: /etc/vsftpd/user_list saved as /etc/vsftpd/user_list.rpmsave

③ 删除上面的文件
`rm -rf /etc/vsftpd`

④ 查看vsftpd是否还在开机启动项中
`chkconfig --list`

⑤ 查看vsftpd运行状态
`service vsftpd status`
>返回：vsftpd: unrecognized service（无法识别vsftpd，说明卸载了vsftpd了）