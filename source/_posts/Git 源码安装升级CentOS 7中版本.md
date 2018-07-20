---
title: Git 源码安装升级CentOS 7中版本
date: 2018-03-19 13:49:30
categories:
- 版本控制
- Git
tags:
- Git
---
服务器上报个漏洞警告：
>**漏洞名称：**Git 远程代码执行漏洞 (CVE-2016-2315)
**漏洞描述：**Git小于2.7.1的全版本中存在一个由于整数溢出导致的缓冲区边界溢出的远程代码执行漏洞，可使攻击者远程执行任意代码

因此，才有了这篇升级安装Git，使用源码编译安装指定版本。
#### 1、下载 Git最新版的源码包
登录https://github.com/git/git/releases查看git的最新版。不要下载带有-rc的，因为它代表了一个候选发布版本。 
安装指令：
`wget https://www.kernel.org/pub/software/scm/git/git-2.16.2.tar.gz `
<!--more-->
![](/uploads/2018/03/git_releases_download.png)

#### 2、解压
`tar -xvf git-2.16.2`

#### 3、进行目录配置
`cd git-2.16.2`
`./configure --prefix=/usr/local/git`
结果显示没有 configure 文件，尴尬了！
根据网上搜索才了解到查找解压包中没有对应的 configure 文件，因此应该找README 或者 INSTALL 之类的文档。

通过 INSTALL安装文档可知，执行如下命令即可：
```
$ make configure ;# as yourself
$ ./configure --prefix=/usr ;# as yourself
$ make all doc ;# as yourself
# make install install-doc install-html;# as root
```
① 执行 `make configure`：报错
>[root@VM_0_7_centos git-2.16.2]# make configure
GIT_VERSION = 2.16.2
    GEN configure
/bin/sh: autoconf: command not found
make: *** [configure] Error 127
**解决方法：**
`[root@VM_0_7_centos git-2.16.2]# yum install -y autoconf`

② `./configure --prefix=/usr/local/git` 成功执行

③ `make`：报错
>Can't locate ExtUtils/MakeMaker.pm in @INC (@INC contains: /usr/local/lib64/perl5 /usr/local/share/perl5 /usr/lib64/perl5/vendor_perl /usr/share/perl5/vendor_perl /usr/lib64/perl5 /usr/share/perl5 .) at Makefile.PL line 3.
BEGIN failed--compilation aborted at Makefile.PL line 3.
make[1]: *** [perl.mak] Error 2
make: *** [perl/perl.mak] Error 2
**解决方法：**
`[root@VM_0_7_centos git-2.16.2]# yum install -y perl-ExtUtils-MakeMaker`

④ `make install` 成功执行，安装完成。
（注：如何确认是否安装完成，只要看 `make` 执行后的结果没有报错，基本都是ok的。）

#### 4、配置全局路径
```
export PATH="/usr/local/git/bin:$PATH" 
source /etc/profile	# 使/etc/profile 文件中的环境变量立即生效
```
（注：此配置全局路径只是对当前生效，重启后就会失效。）
可使用软连接的形式，来使命令长久生效：
`ln -s /usr/local/git/bin/git /usr/local/bin/git`

#### 5、查看Git版本
```
[root@VM_0_7_centos git-2.16.2]# git --version
git version 2.16.2
```