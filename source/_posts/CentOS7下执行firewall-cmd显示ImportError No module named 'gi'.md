---
title: 'CentOS 7下执行firewall-cmd显示ImportError: No module named ''gi'''
date: 2018-03-16 13:17:26
categories:
- Linux
- Linux问题
tags:
- firewalld
---
>在命令行下执行报错提示：
```
[root@VM_0_7_centos ~]# firewall-cmd -h
Traceback (most recent call last):
  File "/usr/bin/firewall-cmd", line 24, in <module>
    from gi.repository import GObject
ModuleNotFoundError: No module named 'gi'
```
<!--more-->
由错误信息也能够看出来是因为缺少 'gi' 模块。
CentOS7 默认自带安装的是Python2.7版本，由于我前段时间安装了最新版的 Python3.6，且是共存的。更改了默认的Python版本为Python3，因此一些Linux命令不能使用，原因就是这些命令使用的Python2版本，由于我安装的Python3版本，并没有把所有的需要的包都安装，因此会提示缺少某些模块。（注：由此处就说明，升级Python2到Python3的时候，要保留Python2）

正确的做法就是，如果使用一些命令提示类似错误的时候，就把命令文件的头部Python版本改为CentOS默认的2.7版本即可。
更改方法如下：
```
第一步，vim /usr/bin/firewall-cmd
将#！/usr/bin/python -Es 改为 #！/usr/bin/python2 -Es（到目前为止，上面提到的问题已解决）
第二步，vim /usr/sbin/firewalld
将#！/usr/bin/python -Es 改为 #！/usr/bin/python2 -Es (这一步是针对于防火墙报错，进行的修改)
```