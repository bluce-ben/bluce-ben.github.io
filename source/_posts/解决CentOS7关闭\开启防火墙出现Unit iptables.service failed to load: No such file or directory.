---
title: >-
  解决CentOS7关闭/开启防火墙出现Unit iptables.service failed to load: No such file or
  directory.
date: 2018-03-15 17:11:04
categories:
- Linux
- Linux问题
tags:
- iptables
---
CentOS7中执行
`service iptables start/stop`

>会报错Failed to start iptables.service: Unit iptables.service failed to load: No such file or directory.

在CentOS 7或RHEL 7或Fedora中防火墙由firewalld来管理，当然你可以还原传统的管理方式。或则使用新的命令进行管理。
<!--more-->

1、还原传统的管理方式
执行一下命令：
```
systemctl stop firewalld  #停止 firewalld
systemctl mask firewalld  #禁用 firewalld
```

并且安装iptables-services：
`yum install iptables-services`

设置开机启动：
`systemctl enable iptables`

启动iptables：
`systemctl start iptables`

保存设置：
service iptables save 或者 /usr/libexec/iptables/iptables.init save

常用命令：
systemctl [stop|start|restart|reload] iptables（分开执行）

OK，再试一下传统管理方式应该就好使了。


2、使用新的防火墙firewalld进行管理

待补充...