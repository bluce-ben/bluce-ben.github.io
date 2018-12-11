---
title: HTTP之TIME_WAIT解析
date: 2018-12-11 11:58:23
categories:
 - 计算机网络
tags:
 - 计算机网络
 - HTTP
---
#### 1、查看服务器连接状态信息
查看服务器TCP链接状态信息：
`netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'`

它会显示例如下面的信息：
TIME_WAIT 814
CLOSE_WAIT 1
FIN_WAIT1 1
ESTABLISHED 634
SYN_RECV 2
LAST_ACK 1

常用的三个状态是：ESTABLISHED 表示正在通信（表示正常数据传输状态），TIME_WAIT 表示主动关闭（表示处理完毕，等待超时结束的请求数），CLOSE_WAIT 表示被动关闭。
<!--more-->
具体每种状态什么意思，看下图就明白了：
![](/uploads/2018/12/network_timewait_00.gif)

这么多状态不用都记住，只要了解到我上面提到的最常见的三种状态的意义就可以了。一般不到万不得已的情况也不会去查看网络状态，如果服务器出了异常，百分之八九十都是下面两种情况：
1. 服务器保持了大量TIME_WAIT状态
2. 服务器保持了大量CLOSE_WAIT状态

因为linux分配给一个用户的文件句柄是有限的，而TIME_WAIT和CLOSE_WAIT两种状态如果一直被保持，那么意味着对应数目的通道就一直被占着，而且是“占着茅坑不使劲”，一旦达到句柄数上限，新的请求就无法被处理了，接着就是大量Too Many Open Files异常，tomcat崩溃。。。

#### 2、文件句柄说明
在Linux里头“一切皆文件”，提示“Too Many Open Files异常”表示句柄不够用了。对应可能就是socket或设备打开太多导致的。

linux在文件句柄的数目上有两个级别的限制。一个是系统级别的总数限制，一个是针对用户的限制。
##### （1）系统级别的句柄操作
* 查看系统的句柄限制：
`sysctl -a | grep fs.file-max`
或者
`cat /proc/sys/fs/file-max`
* 修改句柄数限制：
`sysctl -w fs.file-max 797692`
或者
`echo "797692" > /proc/sys/fs/file-max`
两者作用是相同的，前者改内核参数，后者直接作用于内核参数在虚拟文件系统（procfs,psuedo file system）上对应的文件而已。
或者
修改内核参数 /etc/sysctl.conf
`echo "fs.file-max=797692" >> /etc/sysctl.conf`
（注：查看内核参数： sysctl -p）
*查看整个系统目前使用的文件句柄数量命令：
`cat /proc/sys/fs/file-nr`

##### （2）用户级别的句柄操作
* 查看当前用户的文件句柄限制： ulimit -a
* 修改句柄数限制：
修改 /etc/security/limits.conf 增加下面代码：
```
    *           soft   nofile       262140
    *           hard   nofile       262140
    *           soft   nproc        102400
    *           hard   nproc        102400
```
（注：*标识所有用户）
有两种限制，一种是soft软限制，在数目超过软限制的时候系统会给出warning警告，但是达到hard硬限制的时候系统将拒绝或者异常了。
（注：修改之后可能需要重启shell生效。）

##### （3）其它操作
查看某个进程开了那些句柄： `lsof -p pid`
某个进程开了几个句柄： `lsof -p pid | wc -l`
可以看到某个目录/文件被什么进程占用了，显示已打开该目录或文件的所有进程信息： `lsof path/filename`


#### 3、下面来讨论下这两种情况的处理方法：
##### （1）服务器保持了大量TIME_WAIT状态
这种情况比较常见，一些爬虫服务器或者WEB服务器（如果网管在安装的时候没有做内核参数优化的话）上经常会遇到这个问题，这个问题是怎么产生的呢？

从 上面的示意图可以看得出来，TIME_WAIT是主动关闭连接的一方保持的状态，对于爬虫服务器来说他本身就是“客户端”，在完成一个爬取任务之后，他就 会发起主动关闭连接，从而进入TIME_WAIT的状态，然后在保持这个状态2MSL（max segment lifetime）时间之后，彻底关闭回收资源。为什么要这么做？明明就已经主动关闭连接了为啥还要保持资源一段时间呢？这个是TCP/IP的设计者规定 的，主要出于以下两个方面的考虑：
1. 防止上一次连接中的包，迷路后重新出现，影响新连接（经过2MSL，上一次连接中所有的重复包都会消失）
2. 可靠的关闭TCP连接。在主动关闭方发送的最后一个 ack(fin) ，有可能丢失，这时被动方会重新发fin, 如果这时主动方处于 CLOSED 状态 ，就会响应 rst 而不是 ack。所以主动方要处于 TIME_WAIT 状态，而不能是 CLOSED 。另外这么设计TIME_WAIT 会定时的回收资源，并不会占用很大资源的，除非短时间内接受大量请求或者受到攻击。

>关于MSL引用下面一段话：
MSL为一个TCP Segment（某一块TCP网路封包）从来源送到目的之间可续存的时间（也就是一个网路封包再网路上传输时能存活的时间），由于RFC 793 TCP传输协议是在1981年定义的，当时的网路速度不像现在的网路那样发达，你可以想象你从浏览器输入等到第一个byte出现要等4分钟吗？在现在的网路环境下几乎不可能有这种事情发生，因此我们大可将 TIME_WAIT 状态的续存时间大幅调低，好让连接口能更快空出来给其它连线使用。

>再引用网络资源的一段话：
值 得一说的是，对于基于TCP的HTTP协议，关闭TCP连接的是Server端，这样，Server端会进入TIME_WAIT状态，可 想而知，对于访 问量大的Web Server，会存在大量的TIME_WAIT状态，假如server一秒钟接收1000个请求，那么就会积压 240*1000=240，000个 TIME_WAIT的记录，维护这些状态给Server带来负担。当然现代操作系统都会用快速的查找算法来管理这些 TIME_WAIT，所以对于新的 TCP连接请求，判断是否hit中一个TIME_WAIT不会太费时间，但是有这么多状态要维护总是不好。  
HTTP协议1.1版规定default行为是Keep-Alive，也就是会重用TCP连接传输多个 request/response，一个主要原因就是发现了这个问题

也就是说HTTP的交互跟上面画的那个图是不一样的，关闭连接的不是客户端，而是服务器，所以web服务器也是会出现大量的TIME_WAIT的情况的。这也说清楚了，为什么我的客户端服务器会出现大量CLOSE_WAIT的情况。

现在来说如何来解决这个问题。
解决思路很简单，就是让服务器能够快速回收和重用那些TIME_WAIT的资源。
下面来看一下我们对/etc/sysctl.conf文件的修改：
```
#对于一个新建连接，内核要发送多少个 SYN 连接请求才决定放弃,不应该大于255，默认值是5，对应于180秒左右时间   
net.ipv4.tcp_syn_retries=2  
#net.ipv4.tcp_synack_retries=2  
#表示当keepalive起用的时候，TCP发送keepalive消息的频度。缺省是2小时，改为300秒  
net.ipv4.tcp_keepalive_time=1200  
net.ipv4.tcp_orphan_retries=3  
#表示如果套接字由本端要求关闭，这个参数决定了它保持在FIN-WAIT-2状态的时间  
net.ipv4.tcp_fin_timeout=30    
#表示SYN队列的长度，默认为1024，加大队列长度为8192，可以容纳更多等待连接的网络连接数。  
net.ipv4.tcp_max_syn_backlog = 4096  
#表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭  
net.ipv4.tcp_syncookies = 1  
  
#表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭  
net.ipv4.tcp_tw_reuse = 1  
#表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭  
net.ipv4.tcp_tw_recycle = 1  
  
##减少超时前的探测次数   
net.ipv4.tcp_keepalive_probes=5   
##优化网络设备接收队列   
net.core.netdev_max_backlog=3000   
```
修改完之后执行/sbin/sysctl -p让参数生效。

这里头主要注意到的是
```
net.ipv4.tcp_tw_reuse 
net.ipv4.tcp_tw_recycle 
net.ipv4.tcp_fin_timeout 
net.ipv4.tcp_keepalive_*
```
这几个参数。
>net.ipv4.tcp_tw_reuse和net.ipv4.tcp_tw_recycle的开启都是为了回收处于TIME_WAIT状态的资源。
net.ipv4.tcp_fin_timeout这个时间可以减少在异常情况下服务器从FIN-WAIT-2转到TIME_WAIT的时间。
net.ipv4.tcp_keepalive_*一系列参数，是用来设置服务器检测连接存活的相关配置。

##### （2）服务器保持了大量CLOSE_WAIT状态
TIME_WAIT状态可以通过优化服务器参数得到解决，因为发生TIME_WAIT的情况是服务器自己可控的，要么就是对方连接的异常，要么就是自己没有迅速回收资源，总之不是由于自己程序错误导致的。

但是CLOSE_WAIT就不一样了，从上面的图可以看出来，如果一直保持在CLOSE_WAIT状态，那么只有一种情况，就是在对方关闭连接之后服务器程序自己没有进一步发出ack信号。换句话说，就是在对方连接关闭之后，程序里没有检测到，或者程序压根就忘记了这个时候需要关闭连接，于是这个资源就一直被程序占着。个人觉得这种情况，通过服务器内核参数也没办法解决，服务器对于程序抢占的资源没有主动回收的权利，除非终止程序运行。

简单来说CLOSE_WAIT数目过大是由于被动关闭连接处理不当导致的。
>我说一个场景，服务器A会去请求服务器B上面的apache获取文件资源，正常情况下，如果请求成功，那么在抓取完资源后服务器A会主动发出关闭连接的请求，这个时候就是主动关闭连接，连接状态我们可以看到是TIME_WAIT。如果一旦发生异常呢？假设请求的资源服务器B上并不存在，那么这个时候就会由服务器B发出关闭连接的请求，服务器A就是被动的关闭了连接，如果服务器A被动关闭连接之后自己并没有释放连接，那就会造成CLOSE_WAIT的状态了。

所以很明显，问题还是处在程序里头。

参考资料：
服务器TIME_WAIT和CLOSE_WAIT详解和解决办法：https://www.cnblogs.com/sunxucool/p/3449068.html
HttpClient4 TIME_WAIT和CLOSE_WAIT：https://www.cnblogs.com/caoyusongnet/p/9087633.html

