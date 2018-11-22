---
title: 'Linux VPS、Linux系统防止DDOS攻击脚本[转]'
date: 2018-01-25 15:00:10
categories:
tags:
---
转载地址：http://www.1987.name/33.html

互联网上明争暗斗如同现实社会一样，DDOS攻击屡见不鲜，是很多站长烦恼的事情，尤其是个人站长，资金有限，没有硬件防火墙那只能使用软件。Linux系统上第一个想到的替代软件肯定是iptables，但是它不会智能屏蔽攻击IP，需要手动执行，这样也是不现实。本文主要介绍一款配合iptables来智能抵御DDOS攻击的软件：**DDoS Deflate。**
<!--more-->
#### 关于DDOS Deflate脚本
DDOS deflate是一个轻量级的脚本，以协助阻止拒绝服务攻击的过程中的bash shell脚本。它使用下面的命令来创建一个连接到服务器的IP地址列表，以及与它们的连接总数 。这是最简单的安装软件的解决方案之一。我已经使用一年多，抵御一般性的DDOS攻击效果还是不错的。

主要原理是超过了预先配置的连接数的IP地址自动被服务器防火墙（iptables）阻止！
`netstat -ntu | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -n`

#### DDOS Deflate的5个主要功能
1. 可以配置白名单的IP地址文件，配置文件路径：/usr/local/ddos/ignore.ip.list
2. 配置文件简单明了，文件路径：/usr/local/ddos/ddos.conf
3. 可以设置被防火墙（iptables）屏蔽的IP地址封锁时间（默认：600秒后自动解除封锁）
4. 可以修改配置文件，脚本可以定时周期性运行（默认是：1分钟）
5. 当遇到攻击，IP被封锁之后可以为指定的邮箱接收电子邮件警报。

#### DDOS Deflate的安装方法
DDOS Deflate非常简单，下载脚本后，直接执行脚本，结束后会显示安装信息，按ESC退出即可。
```
wget http://www.inetbase.com/scripts/ddos/install.sh
chmod +x install.sh
./install.sh
```
安装结束后，配置主配文件ddos.conf
```
##### Paths of the script and other files
PROGDIR="/usr/local/ddos" #软件文件存放位置
PROG="/usr/local/ddos/ddos.sh" #主要功能脚本路径
IGNORE_IP_LIST="/usr/local/ddos/ignore.ip.list" #白名单列表路径
CRON="/etc/cron.d/ddos.cron" #定时任务脚本路径
APF="/etc/apf/apf" #APF路径
IPT="/sbin/iptables" #iptables路径

##### frequency in minutes for running the script
##### Caution: Every time this setting is changed, run the script with --cron
##### option so that the new frequency takes effect
FREQ=1 #检查周期时间，默认1分钟

##### How many connections define a bad IP? Indicate that below.
NO_OF_CONNECTIONS=150 #允许客户端与服务器的最大连接数，超过IP就会被屏蔽，一般保持默认即可

##### APF_BAN=1 (Make sure your APF version is atleast 0.96)
##### APF_BAN=0 (Uses iptables for banning ips instead of APF)
APF_BAN=0  #数字1为使用APF，数字0为使用iptables，这里推荐使用iptables 

##### KILL=0 (Bad IPs are'nt banned, good for interactive execution of script)
##### KILL=1 (Recommended setting)
KILL=1 #是否屏蔽IP，当然是屏蔽，默认即可

##### An email is sent to the following address when an IP is banned.
##### Blank would suppress sending of mails
EMAIL_TO="admin@1987.name" #指定一个 电子邮件，用于发送警报 

##### Number of seconds the banned ip should remain in blacklist.
BAN_PERIOD=600 #屏蔽时间，这里自由设定
```
配置文件中提到的APF，它也是linux系统中防火墙之一，这里稍作介绍：APF（Advanced Policy Firewall），是 Rf-x Networks 出品的Linux环境下的软件防火墙。APF采用Linux系统默认的 iptables 规则。APF可以算是Linux中最出名的软件防火墙之一。

#### 为DDOS Deflate开启相关服务
开启iptables
`service iptables start`
开启crontab，定时任务
`service crond start`

#### 如何卸载DDOS Deflate
```
wget http://www.inetbase.com/scripts/ddos/uninstall.ddos
chmod +x uninstall.ddos
./uninstall.ddos
```
希望遇到DDOS攻击的朋友使用此软件能解决头疼问题，也感谢软件作者。