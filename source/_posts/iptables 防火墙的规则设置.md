---
title: iptables 防火墙的规则设置
date: 2018-03-19 11:00:07
categories:
- Linux
tags:
- iptables
---
#### iptables 常用命令及参数注释：
>主要包含：
**命令表**
　　用来增加(-A、-I)删除(-D)修改(-R)查看(-L)规则等；
**常用参数**
　　用来指定协议(-p)、源地址(-s)、源端口(-\-sport)、目的地址(-d)、目的端口(\--dport)、进入网卡(-i)、出去网卡(-o)等设定包信息（即什么样的包）；用来描述要处理包的信息。
**常用处理动作**
　　用 -j 来指定对包的处理(ACCEPT、DROP、REJECT、REDIRECT等)。

<!--more-->

**常用参数注释：**
<font color="red">命令 -P, -\-policy</font>
范例 `iptables -P INPUT DROP`
说明 定义过滤政策。 也就是未符合过滤条件之封包，预设的处理方式。

<font color="red">参数 -\-line-numbers</font>
范例 `iptables -nL --line-numbers`
说明 查询规则链表的number号。

<font color="red">参数 -m state -\-state</font>
范例 `iptables -A INPUT -m state --state RELATED,ESTABLISHED`
说明 用来比对联机状态，联机状态共有四种：INVALID、ESTABLISHED、NEW 和 RELATED。
>**INVALID** 表示该封包的联机编号（Session ID）无法辨识或编号不正确。
**ESTABLISHED** 表示该封包属于某个已经建立的联机。
**NEW** 表示该封包想要起始一个联机（重设联机或将联机重导向）。
**RELATED** 表示该封包是属于某个已经建立的联机，所建立的新联机。例如：FTP-DATA 联机必定是源自某个 FTP 联机。


#### 常用规则总结
##### （1）启动、停止和重启 iptables
`systemctl start/stop/restart/status iptables.service`

##### （2）查看iptables 现有规则
`iptables -L -n`
`iptables -Ln --line-numbers` #查看对应的规则numbers
`iptables-save`

##### （3）屏蔽某个 IP
`iptables -A INPUT -s xxx.xxx.xxx.xxx -j DROP`
如果你只想屏蔽 TCP 流量，可以使用 -p 参数的指定协议，例如：
`iptables -A INPUT -p tcp -s xxx.xxx.xxx.xxx -j DROP`

##### （4）解封某个 IP地址（即删除）
`iptables -D INPUT -s xxx.xxx.xxx.xxx -j DROP`
或
`iptables -D INPUT numbers`

##### （5）关闭指定端口
阻止特定的传出连接：
`iptables -A OUTPUT -p tcp --dport xxx -j DROP`
阻止特定的传入连接：
`iptables -A INPUT -p tcp --dport xxx -j ACCEPT`

##### （6）使用Multiport控制多端口
使用 multiport 我们可以一次性在单条规则中写入多个端口，例如：
`iptables -A INPUT  -p tcp -m multiport --dports 22,80,443 -j ACCEPT`
`iptables -A OUTPUT -p tcp -m multiport --sports 22,80,443 -j ACCEPT`

##### （7）在规则中使用 IP 地址范围
在 IPtables 中 IP 地址范围是可以直接使用 CIDR 进行表示的，例如：
`iptables -A OUTPUT -p tcp -d 192.168.100.0/24 --dport 22 -j ACCEPT`

##### （8）禁止PING
对 Linux 禁 PING 可以使用如下规则屏蔽 ICMP 传入连接：
`iptables -A INPUT -p icmp -i eth0 -j DROP`

##### （9）允许访问回环网卡
环回访问（127.0.0.1）是比较重要的，建议大家都开放：
`iptables -A INPUT -i lo -j ACCEPT`
`iptables -A OUTPUT -o lo -j ACCEPT`

##### （10）丢弃无效数据包
很多网络攻击都会尝试用黑客自定义的非法数据包进行尝试，我们可以使用如下命令来丢弃无效数据包：
`iptables -A INPUT -m conntrack --ctstate INVALID -j DROP`

##### （11）插入指定位置一条规则
`iptables -I INPUT number -p tcp -m tcp --dport 21 -j ACCEPT`
（<font color="red">注：插入对应的number位置，原number位置往后移</font>）



#### 常用端口总结

服务  |  端口  |  是否开启  |  协议类型  |  描述
------|--------|------------|------------|------------
HTTP  | 80     | 建议开启   | tcp        | Web服务
HTTPS | 443    | 建议开启   | tcp        | Web加密服务
POP3  | 110    | 不建议开启 | tcp        | POP邮局协议
SMTP  | 25     | 不建议开启 | tcp        | SMTP邮件传输协议
FTP   | 21     | 不建议开启 | tcp        | FTP服务
SFTP  | 115    | 不建议开启 | tcp        | SFTP安全文本传输协议
SSH   | 22     | 必须开启   | tcp        | SSH服务
Telnet| 23     | 不建议开启 | tcp        | 不安全的文本传送
DNS   | 53     | 建议开启   | tcp        | DNS域名服务
TOMCAT| 8080   | 不建议开启 | tcp        | tomcat服务器端口
MySQL | 3306   | 不建议开启 | tcp        | MySQL服务器端口



#### 附本人主机iptables规则脚本
```
#!/bin/bash

iptables -F  # 清空所有默认规则
iptables -X  # 清空所有自定义规则
iptables -Z  # 所有封包计数器归0

# 设置默认的访问规则
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT

# 允许接受本机请求之后的返回数据（ESTABLISHED 是已经建立连接的状态）；RELATED 是为了FTP设置的
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A INPUT -p icmp -j ACCEPT  # 允许 ping
iptables -A INPUT -i lo -j ACCEPT    # 允许访问回环网卡

# 自定义开启一些端口
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
iptables -A INPUT -p tcp --dport 3690 -j ACCEPT

# 定义对应规则没有仍返回相应信息
iptables -A INPUT -j REJECT --reject-with icmp-host-prohibited
iptables -A FORWARD -j REJECT --reject-with icmp-host-prohibited

# 保存更改并重启服务
service iptables save
systemctl restart iptables.service
```