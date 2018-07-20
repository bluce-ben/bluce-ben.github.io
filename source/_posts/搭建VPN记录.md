---
title: 搭建VPS记录
date: 2018-07-18 19:43:42
categories:
tags: VPS
---
通过Stack Overflow搜索到一个靠谱点的搭建VPS方法，完成搭建，现整理如下：
#### 一、服务器端安装 ####
##### 1、软件安装 #####
Debian / Ubuntu:
　　apt-get install python-pip
　　pip install shadowsocks

CentOS:
　　yum install python-setuptools && easy_install pip
　　pip install shadowsocks

Windows:
　　See [Install Server on Windows]
<!--more-->

##### 2、账号配置及常用命令 #####
启动：
　　ssserver -p 443 -k password -m aes-256-cfb
后台启动：
　　sudo ssserver -p 443 -k password -m aes-256-cfb --user nobody -d start
停止：
　　sudo ssserver -d stop
检查log日志：
　　sudo less /var/log/shadowsocks.log


#### 二、客户端安装wiki ####
```
[Android]:           https://github.com/shadowsocks/shadowsocks-android
[Build Status]:      https://img.shields.io/travis/shadowsocks/shadowsocks/master.svg?style=flat
[Configuration]:     https://github.com/shadowsocks/shadowsocks/wiki/Configuration-via-Config-File
[Coverage Status]:   https://jenkins.shadowvpn.org/result/shadowsocks
[Coverage]:          https://jenkins.shadowvpn.org/job/Shadowsocks/ws/PYENV/py34/label/linux/htmlcov/index.html
[Debian sid]:        https://packages.debian.org/unstable/python/shadowsocks
[iOS]:               https://github.com/shadowsocks/shadowsocks-iOS/wiki/Help
[Issue Tracker]:     https://github.com/shadowsocks/shadowsocks/issues?state=open
[Install Server on Windows]: https://github.com/shadowsocks/shadowsocks/wiki/Install-Shadowsocks-Server-on-Windows
[Mailing list]:      https://groups.google.com/group/shadowsocks
[OpenWRT]:           https://github.com/shadowsocks/openwrt-shadowsocks
[OS X]:              https://github.com/shadowsocks/shadowsocks-iOS/wiki/Shadowsocks-for-OSX-Help
[PyPI]:              https://pypi.python.org/pypi/shadowsocks
[PyPI version]:      https://img.shields.io/pypi/v/shadowsocks.svg?style=flat
[Travis CI]:         https://travis-ci.org/shadowsocks/shadowsocks
[Troubleshooting]:   https://github.com/shadowsocks/shadowsocks/wiki/Troubleshooting
[Wiki]:              https://github.com/shadowsocks/shadowsocks/wiki
[Windows]:           https://github.com/shadowsocks/shadowsocks-csharp
```

#### 三、报错集锦 ####
##### 1、启动服务的时候报： #####
>AttributeError: /usr/lib/x86_64-linux-gnu/libcrypto.so.1.1: undefined symbol

这个问题是由于在openssl1.1.0版本中，废弃了`EVP_CIPHER_CTX_cleanup`函数，如官网中所说：
>EVP_CIPHER_CTX was made opaque in OpenSSL 1.1.0. As a result, EVP_CIPHER_CTX_reset() appeared and EVP_CIPHER_CTX_cleanup() disappeared.
>EVP_CIPHER_CTX_init() remains as an alias for EVP_CIPHER_CTX_reset().

**修改方法：**
>(1) 用vim打开文件：vim /usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/openssl.py (该路径请根据自己的系统情况自行修改，如果不知道该文件在哪里的话，可以使用find命令查找文件位置)
(2) 跳转到52行（shadowsocks2.8.2版本，其他版本搜索一下cleanup）
(3) 进入编辑模式
(4) 将第52行libcrypto.EVP_CIPHER_CTX_cleanup.argtypes = (c_void_p,) 
　　改为libcrypto.EVP_CIPHER_CTX_reset.argtypes = (c_void_p,)
(5) 再次搜索cleanup（全文件共2处，此处位于111行），将libcrypto.EVP_CIPHER_CTX_cleanup(self._ctx) 
　　改为libcrypto.EVP_CIPHER_CTX_reset(self._ctx)
(6) 保存并退出
(7) 启动shadowsocks服务：service shadowsocks start 或 sslocal -c ss配置文件目录
(8) 问题解决


**插曲：**
在 Vultr 上面购买的“东京”服务器用来搭建VPN，刚开始购买的是 $2.5/month，上面标注的是 only ipv6，服务器启动后，才发现国内目前还没有普及ipv6，ping都不通，在阿里云服务器上开启ipv6，然后再ping仍然不通。 应该是网络运营商方面没有开启ipv6，所以都出不去，才ping不通。迫不得已，又删除掉服务器，重新购买了 $5/month 的服务器，搭建完毕。

*下面查看电脑是否支持IPV6访问：*
登陆`http://test-ipv6.com/`（如果能上网的话）
根据网站给出的信息，判断是否支持IPv6。
![](/uploads/2018/07/vpn_01.png)
