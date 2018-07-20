---
title: FTP 客户端报错集锦
date: 2018-03-19 11:19:53
categories:
- 服务器
- FTP
tags:
- FTP
---
#### 1、ftp 客户端连接报错，如下：
>![](/uploads/2018/03/ftp_connect_error.png 'ftp连接超时')

经查看是因为 服务器端对应的 21号端口未开启，开启即可，命令如下：
`iptables -I INPUT 4 -p tcp -m tcp --dport 21 -j ACCEPT`
<!--more-->

#### 2、ftp 客户端连接成功，而读取目录失败，如下：
>![](/uploads/2018/03/ftp_writedir_error.png 'ftp读取目录失败')

经过查找，发现是因为ftp数据连接模式的问题，ftp连接模式有两种：PORT（主动模式）和PASV（被动模式）。两者的前面文章中已经说明两者的区别，在这里我选择“主动模式”，只开放ftp服务端20端口传输数据，客户端随机产生大于1024端口号。
然后，开启服务端20端口权限即可，命令如下：
`iptables -I INPUT 4 -p tcp -m tcp --dport 20 -j ACCEPT`

#### 3、ftp 客户端能够下载数据，却不能上传数据，如下：
>![](/uploads/2018/03/ftp_trans_error.png 'ftp文件传输错误')

经过一番查找，发现应该是我本身一堆错误导致的。包括ftp 用户组使用但未创建，ftp用户根目录权限未给予正确的ftp用户组（注：根目录创建是root用户，因此所有者及所属组都是root。所以其他用户都不能操作）。因此，我做了如下修改：
```
# 首先，创建用户组
groupadd ftpgroup
# 将用户添加进用户组
usermod -g ftpgroup ftp1
# 更改用户根目录所属组
chown -R :ftpgroup ceshi/
# 更改用户根目录所属组权限为可写
chmod -R g+w ceshi/
```

#### 4、ftp 拒绝登陆，如下：
>![](/uploads/2018/03/ftp_refuse_error.png 'ftp拒绝登陆')

原因是配置文件中 `allow_writeable_chroot=YES` 不存在，而`chroot_local_user=YES`已经开启。所以导致 ftp 拒绝登陆。