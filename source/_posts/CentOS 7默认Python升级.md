---
title: CentOS 7默认Python升级
date: 2018-03-13 18:01:05
categories:
- Python
tags:
- Python
---
### 简述
CentOS 7 中默认安装了 Python，版本比较低（2.7.5），为了使用新版 3.x，需要对旧版本进行升级。

由于很多基本的命令、软件包都依赖旧版本，比如：yum。所以，在更新 Python 时，建议不要删除旧版本（新旧版本可以共存）。

### 安装Python3.6.4
<!--more-->
#### （1）下载并解压缩
下载地址：[<font color="red">传送门</font>](https://www.python.org/downloads/release/python-364/ 'Python3.x')
此处我选择的版本是Python3.6.4：
　　`wget https://www.python.org/ftp/python/3.6.4/Python-3.6.4.tgz`
解压缩：
　　`tar -xvf Python-3.6.4.tgz`

#### （2）检测编译环境
```
cd Python-3.6.4/
./configure
```
执行 ./configure 时，如果报错：
>configure: error: no acceptable C compiler found in $PATH

说明没有安装合适的编译器。这时，需要安装/升级 gcc 及其它依赖包。
　　`yum install make gcc gcc-c++`
完成之后，重新执行：
　　`./configure`

#### （3）编译 & 安装
　　`make & make install`
安装中如果报错：
>zipimport.ZipImportError: can't decompress data; zlib not available

说明是因为缺少zlib 的相关工具包导致的，知道了问题所在，那么我们只需要安装相关依赖包即可， 
① 打开终端，输入一下命令安装zlib相关依赖包：
　　`yum -y install zlib*` 或 `yum -y install zlib zlib-devel`（未试）

② 进入 python安装包,修改Module路径的setup文件：
　　`vim Module/Setup.dist`
找到以下这行代码，去掉注释：
>#zlib zlibmodule.c -I$(prefix)/include -L$(exec_prefix)/lib -lz

然后再次执行编译 & 安装。

### 设置Python默认版本
#### （1）查看Python版本
```
[root@VM_0_7_centos ~]# python -V
Python 2.7.5
[root@VM_0_7_centos ~]# python3 -V
Python 3.6.4
```
查看Python命令路径：
```
[root@VM_0_7_centos ~]# which python
/usr/bin/python
[root@VM_0_7_centos ~]# which python3
/usr/local/bin/python3
```

#### （2）设置Python3.x为默认版本
查看Python默认版本：
```
[root@VM_0_7_centos ~]# ll /usr/bin/ | grep python
-rwxr-xr-x. 1 root root      11232 Aug 10  2017 abrt-action-analyze-python
lrwxrwxrwx. 1 root root          7 Jan  9 18:20 python -> python2
lrwxrwxrwx. 1 root root          9 Jan  9 18:20 python2 -> python2.7
-rwxr-xr-x. 1 root root       7136 Aug  4  2017 python2.7
```
更改Python默认版本，将原来 python 的软链接重命名：
　　`mv /usr/bin/python /usr/bin/python.bak`
将 python 链接至 python3：
　　`ln -s /usr/local/bin/python3 /usr/bin/python`
这时，再查看 Python 的版本：
```
[root@VM_0_7_centos ~]# python -V
Python 3.6.4
```
输出的是 3.x，说明已经使用的是 python3了。


### 配置yum
```
[root@VM_0_7_centos ~]# yum
  File "/usr/bin/yum", line 30
    except KeyboardInterrupt, e:
                            ^
SyntaxError: invalid syntax
```
升级 Python 之后，由于将默认的 python 指向了 python3，yum 不能正常使用，需要编辑 yum 的配置文件：
　　`vi /usr/bin/yum`
同时修改：
　　`vi /usr/libexec/urlgrabber-ext-down`
将 #!/usr/bin/python 改为 #!/usr/bin/python2.7，保存退出即可。