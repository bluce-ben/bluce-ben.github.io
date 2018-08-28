---
title: Linux 配置支持IPv6
date: 2018-08-28 17:56:05
categories:
- Linux
tags:
- Linux
---
由于ipv4已经不能满足当前的使用，ipv6出现了，也必将兼容ipv4的地位。因此，下面是配置ipv6的步骤：
#### 一、检查Linux是否已经开启ipv6 ####
`[root@iz2ze3oyrjbxg32wecre15z /]# ifconfig`
<!--more-->
![](/uploads/2018/08/network_ipv6_01.png)
 
从结果看出，输出结果没有 ipv6支持，如果支持ipv6，则输出结果会包含 “inet6”。
![](/uploads/2018/08/network_ipv6_02.png)

可以测试一下，如果环境已经支持，则可以不用往下看了！

#### 二、开启ipv6 ####
（1）找到配置sysctl.conf 文件，路径在：`/etc/sysctl.conf` ，找到如下配置：
![](/uploads/2018/08/network_ipv6_03.png)
   
如果已经存在，则直接修改，如果不存在，则新增。
将列出的ipv6相关配置更改为0
![](/uploads/2018/08/network_ipv6_04.png)

（2）找到 disable_ipv6.conf 文件，路径在: `/etc/modprobe.d/disable_ipv6.conf`
找到如下配置：
![](/uploads/2018/08/network_ipv6_05.png)

列出的配置更改为 0
![](/uploads/2018/08/network_ipv6_06.png)

（3）找到 network.conf 文件，路径在：`/etc/sysconfig/network`
找到如下配置：
![](/uploads/2018/08/network_ipv6_07.png)

将列出的配置更改为 yes
![](/uploads/2018/08/network_ipv6_08.png)


（4）重启网络服务
`[root@iz2ze3oyrjbxg32wecre15z /]# service network restart`

（5）通过ifconfig 命令检查是否已经启动ipv6
`[root@iz2ze3oyrjbxg32wecre15z /]# ifconfig|grep -i inet6`
![](/uploads/2018/08/network_ipv6_09.png)
 
结果显示，已经包含 inet6 相关信息。

