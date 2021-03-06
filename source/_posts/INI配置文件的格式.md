---
title: INI配置文件的格式
date: 2018-10-24 17:14:55
categories:
- 服务器
- Nginx
tags:
- Nginx
---
最近学习Go语言的时候，发现好多Go框架中的配置文件都支持INI格式的。当然，不仅仅是Go框架，PHP语言的好多框架也使用INI格式的配置文件，例如Yaf等。因此，需要梳理一下INI格式。常用的配置文件格式还有很多，如XML配置文件等。
<!--more-->

>在早期的windows桌面系统中主要是用INI文件作为系统的配置文件，从win95以后开始转向使用注册表，但是还有很多系统配置是使用INI文件的。其实INI文件就是简单的text文件，只不过这种txt文件要遵循一定的INI文件格式。现在的WINCE系统上也常常用INI文件作为配置文件。“.INI ”就是英文 “initialization”的头三个字母的缩写；当然INI file的后缀名也不一定是".ini"也可以是".cfg"，".conf ”或者是".txt"。

#### INI文件由节、键、值组成 ####
**节**
　　[section]  

**参数（键=值） **
　　name=value

**注解** 
　　注解使用分号表示（;）。在分号后面的文字，直到该行结尾都全部为注解。

<font color="red">INI文件的格式很简单，最基本的三个要素是：parameters，sections和comments。</font>

#### 什么是parameters？ ####
INI所包含的最基本的“元素”就是parameter；每一个parameter都有一个name和一个value，name和value是由等号“=”隔开。name在等号的左边。
如：
　　name = value

#### 什么是sections ？ ####
所有的parameters都是以sections为单位结合在一起的。所有的section名称都是独占一行，并且sections名字都被方括号包围着（[ and ])。在section声明后的所有parameters都是属于该section。对于一个section没有明显的结束标志符，一个section的开始就是上一个section的结束，或者是end of the file。Sections一般情况下不能被nested，当然特殊情况下也可以实现sections的嵌套。
section如下所示：
　　[section]

#### 什么是comments ？ ####
在INI文件中注释语句是以分号“；”开始的。所有的所有的注释语句不管多长都是独占一行直到结束的。在分号和行结束符之间的所有内容都是被忽略的。
注释实例如下：
;comments text

当然，上面讲的都是最经典的INI文件格式，随着使用的需求INI文件的格式也出现了很多变种；

#### INI实例 ####
```
; last modified 1 April 2001 by John Doe
[owner]
name = John Doe
organization = Acme Products

[database]
server=192.0.2.42
; use IP address in case network name resolution is not working
port=143
file = "acme payroll.dat"
```
