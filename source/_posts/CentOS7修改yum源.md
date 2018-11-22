---
title: CentOS 7修改yum源
date: 2018-03-20 18:48:03
categories:
- Linux
tags:
- Linux
---
官方的yum源在国内访问效果不佳。
需要改为国内比较好的阿里云或者网易的yum源。

1、配置 yum源
```
# 备份当前的yum源
cp /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS.Base.repo.bak

# 下载阿里云的yum配置源
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

2、然后更新缓存
```
yum clean all
yum makecache
```