---
title: Linux 下二进制文件命令存放目录区别
date: 2018-03-20 18:48:21
categories:
- Linux
tags:
- Linux
---
>二进制文件命令：
* /bin　　# 系统的一些指令
* /sbin　　# 一般是指超级用户指令
* /usr/bin　　# 后期安装的一些软件的运行脚本
* /usr/sbin　　# 一些用户安装的系统管理的必备程序
* /usr/local/bin　　# 通常是源码编译的软件
* /usr/local/sbin　　# 通常是源码编译的软件，用来管理系统的程序

<!--more-->

**首先看下PATH变量在不同用户下的值：**
**root用户：**
```
[root@VM_0_7_centos /]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
```
**普通用户：**
```
[ben@VM_0_7_centos ~]$ echo $PATH
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/ben/.local/bin:/home/ben/bin
```
（注：bin是binary的缩写，二进制。sbin意义为system binary。）

>由上面可知道 root权限都没有 /bin、/sbin 的目录使用权。为什么呢？
原因就是，/bin以及/sbin是和/在同一文件系统，在挂载其他文件系统之前就可以使用/bin以及/sbin下的命令。
/usr/bin，/usr/sbin，/usr/local/bin，/usr/local/sbin可能与根文件系统不在同一文件系统之中，可能是在其他文件系统中后挂载上去的。
而我的服务器是 腾讯云，因此也是挂载上去的。


从命令功能来看，/sbin 下的命令属于基本的系统命令，如shutdown，reboot，用于启动系统，修复系统，/bin下存放一些普通的基本命令，如ls,chmod等，这些命令在Linux系统里的配置文件脚本里经常用到。

从用户权限的角度看，/sbin目录下的命令通常只有管理员才可以运行，/bin下的命令管理员和一般的用户都可以使用。

从可运行时间角度看，/sbin,/bin能够在挂载其他文件系统前就可以使用。


**下面来说一下常用的目录：**
>**/bin 是系统的一些指令。**
bin为binary的简写主要放置一些系统的必备执行档例如:cat、cp、chmod df、dmesg、gzip、kill、ls、mkdir、more、mount、rm、su、tar等。
**/sbin 一般是指超级用户指令。**
主要放置一些系统管理的必备程式例如:cfdisk、dhcpcd、dump、e2fsck、fdisk、halt、ifconfig、ifup、 ifdown、init、insmod、lilo、lsmod、mke2fs、modprobe、quotacheck、reboot、rmmod、 runlevel、shutdown等。
**/usr/bin 是你在后期安装的一些软件的运行脚本。**
主要放置一些应用软体工具的必备执行档例如c++、g++、gcc、chdrv、diff、dig、du、eject、elm、free、gnome*、 gzip、htpasswd、kfm、ktop、last、less、locale、m4、make、man、mcopy、ncftp、 newaliases、nslookup passwd、quota、smb*、wget等。
**/usr/sbin 放置一些用户安装的系统管理的必备程式**
例如:dhcpd、httpd、imap、in.*d、inetd、lpd、named、netconfig、nmbd、samba、sendmail、squid、swap、tcpd、tcpdump等。
>**/usr/local/bin **
很多时候我们自己安装的软件，可能在此处建立一个软连接（符号链接），指向实际的可执行文件。
>**/usr/local/sbin**
也是我们自己安装的软件，一般用来管理系统的程序

（注：以上所说的并不是绝对的，例如ifconfig在/sbin下，但是普通用户一般具有可执行权限。）