---
title: Linux命令之netstat
date: 2018-06-11 16:06:20
categories:
- Linux
tags:
- Linux
- Linux命令
---
```
netstat命令各个参数说明如下：
　　-a : 显示全部端口
　　-t : 指明显示TCP端口
　　-u : 指明显示UDP端口
　　-l : 仅显示监听套接字(所谓套接字就是使应用程序能够读写与收发通讯协议(protocol)与资料的程序)
　　-p : 显示进程标识符和程序名称，每一个套接字/端口都属于一个程序。
　　-n : 不进行DNS轮询，显示IP(可以加速操作)
```
<!--more-->

即可显示当前服务器上所有端口及进程服务，于grep结合可查看某个具体端口及服务情况··
`netstat -ntlp   //查看当前所有tcp端口·`
`netstat -ntulp |grep 80   //查看所有80端口使用情况·`
`netstat -an | grep 3306   //查看所有3306端口使用情况·`


**查看一台服务器上面哪些服务及端口**
`netstat  -lanp`

**查看一个服务有几个端口。比如要查看mysqld**
`ps -ef |grep mysqld`

**查看某一端口的连接数量,比如3306端口**
`netstat -pnt |grep :3306 |wc`

**查看某一端口的连接客户端IP 比如3306端口**
`netstat -anp |grep 3306`

`netstat -an 查看网络端口 `

`lsof -i :port，使用lsof -i :port就能看见所指定端口运行的程序，同时还有当前连接。 `

`nmap 端口扫描`
`netstat -nupl  (UDP类型的端口)`
`netstat -ntpl  (TCP类型的端口)`
`netstat -anp 显示系统端口使用情况`

Netstat 命令用于显示各种网络相关信息，如网络连接，路由表，接口状态 (Interface Statistics)，masquerade 连接，多播成员 (Multicast Memberships) 等等。
>输出信息含义：
```
[rd@qbj3-dwxk-ark-api-00 server_conf]$ netstat
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State
tcp        0      0 10.40.10.219:21462          10.40.120.140:5511          TIME_WAIT   
tcp        0      0 10.40.10.219:35814          10.40.120.140:5511          TIME_WAIT
...

Active UNIX domain sockets (w/o servers)
Proto RefCnt Flags       Type       State         I-Node Path
unix  3      [ ]         STREAM     CONNECTED     286798166 /usr/local/yd.socket.client
unix  2      [ ]         DGRAM                    8807   /var/run/portreserve/socket
unix  5      [ ]         DGRAM                    8837   /dev/log
unix  2      [ ]         DGRAM                    7933   @/org/kernel/udev/udevd
unix  3      [ ]         STREAM     CONNECTED     1144400163 
...
```
从整体上看，netstat的输出结果可以分为两个部分：
* 一个是Active Internet connections，称为有源TCP连接，其中"Recv-Q"和"Send-Q"指%0A的是接收队列和发送队列。这些数字一般都应该是0。如果不是则表示软件包正在队列中堆积。这种情况只能在非常少的情况见到。

* 另一个是Active UNIX domain sockets，称为有源Unix域套接口(和网络套接字一样，但是只能用于本机通信，性能可以提高一倍)。
Proto显示连接使用的协议,RefCnt表示连接到本套接口上的进程号,Types显示套接口的类型,State显示套接口当前的状态,Path表示连接到套接口的其它进程使用的路径名。