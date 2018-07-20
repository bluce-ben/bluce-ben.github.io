---
title: 防火墙之iptables和firewalld
date: 2018-03-16 13:16:07
categories:
- Linux
tags:
- 网络安全
- iptables
- firewalld
---
在RHEL 7系统中，firewalld防火墙取代了iptables防火墙。对于接触Linux系统比较早或学习过RHEL 6系统的读者来说，当他们发现曾经掌握的知识在RHEL 7中不再适用，就需要全新学习firewalld。其实，iptables与firewalld都不是真正的防火墙，它们都只是用来定义防火墙策略的防火墙管理工具而已，或者说，它们只是一种服务。iptables服务会把配置好的防火墙策略交由内核层面的netfilter网络过滤器来处理，而firewalld服务则是把配置好的防火墙策略交由内核层面的nftables包过滤框架来处理。换句话说，当前在Linux系统中其实存在多个防火墙管理工具，旨在方便运维人员管理Linux系统中的防火墙策略，我们只需要配置妥当其中的一个就足够了。虽然这些工具各有优劣，但它们在防火墙策略的配置思路上是保持一致的。只要在这多个防火墙管理工具中任选一款并将其学透，就足以满足日常的工作需求了。
<!--more-->

#### 防火墙的分类：
根据工作的层次的不同来划分，常见的防火墙工作在OSI第三层，即网络层防火墙，工作在OSI第七层的称为应用层防火墙，或者代理服务器（代理网关）。
##### 1、网络层防火墙
网络层防火墙又称包过滤防火墙，在网络层对数据包进行选择，选择的依据是系统内设置的过滤逻辑，被称为访问控制列表（ACL），通过检查数据流中每个数据的源地址，目的地址，所用端口号和协议状态等因素,来确定是否允许该数据包通过。
特点：对用户来说透明，处理速度快且易于维护。但是一旦黑客突破防火墙，就可以轻易地伪造数据包的源地址，目的地址和IP的端口号，即“IP地址伪造”。

##### 2、应用层防火墙
代理服务型防火墙（Proxy Service）将所有跨越防火墙的网络通信链路分为两段。当代理服务器接收到用户对某个站点的访问请求后会检查该请求是否符合控制规则。如果规则允许，则代理服务器会替用户去那个站点取回所需要的信息，转发给用户。内外网用户的访问都是通过代理服务器上的“链接”来实现的，从而起到了隔离防火墙内外计算机系统的作用。
特点：在应用层对数据进行检查，比较安全。但是会增加防火墙的负载。

包过滤防火墙将对每一个接收到的包做出允许或拒绝的决定。具体地讲，它针对每一个数据包的包头，按照包过滤规则进行判定，与规则相匹配的包依据路由信息继续转发，否则就丢弃。包过滤是在IP层实现的，包过滤根据数据包的源IP地址、目的IP地址、协议类型（TCP包、UDP包、ICMP包）、源端口、目的端口等包头信息及数据包传输方向等信息来判断是否允许数据包通过。包过滤也包括与服务相关的过滤，这是指基于特定的服务进行包过滤，由于绝大多数服务的监听都驻留在特定TCP/UDP端口，因此，为阻断所有进入特定服务的链接，防火墙只需将所有包含特定TCP/UDP目的端口的包丢弃即可。

现实生产环境中所使用的防火墙一般都是二者结合体。即先检查网络数据，通过之后再送到应用层去检查。


#### iptables 防火墙
在早期的Linux系统中，默认使用的是iptables防火墙管理服务来配置防火墙。尽管新型的firewalld防火墙管理服务已经被投入使用多年，但是大量的企业在生产环境中依然出于各种原因而继续使用iptables。对于如何在CentOS7系统中，从firewalld防火墙更改为iptables防火墙，可查看我的一篇文章**《解决CentOS7关闭/开启防火墙出现Unit iptables.service failed to load: No such file or directory.》**。

##### 1、策略与规则链
<font color="red">防火墙会从上至下的顺序来读取配置的策略规则</font>，在找到匹配项后就立即结束匹配工作并去执行匹配项中定义的行为（即放行或阻止）。如果在读取完所有的策略规则之后没有匹配项，就去执行默认的策略。一般而言，防火墙策略规则的设置有两种：一种是“通”（即放行），一种是“堵”（即阻止）。当防火墙的默认策略为拒绝时（堵），就要设置允许规则（通），否则谁都进不来；如果防火墙的默认策略为允许时，就要设置拒绝规则，否则谁都能进来，防火墙也就失去了防范的作用。

iptables服务把用于处理或过滤流量的策略条目称之为规则，多条规则可以组成一个规则链，而规则链则依据数据包处理位置的不同进行分类，具体如下：
>在进行路由选择前处理数据包（PREROUTING）；
处理流入的数据包（INPUT）；
处理流出的数据包（OUTPUT）；
处理转发的数据包（FORWARD）；
在进行路由选择后处理数据包（POSTROUTING）。

一般来说，从内网向外网发送的流量一般都是可控且良性的，因此我们使用最多的就是INPUT规则链，该规则链可以增大黑客人员从外网入侵内网的难度。

##### 2、基本的命令参数
iptables是一款基于命令行的防火墙策略管理工具，具有大量参数，学习难度较大。好在对于日常的防火墙策略配置来讲，大家无需深入了解诸如“四表五链”的理论概念，只需要掌握常用的参数并做到灵活搭配即可，这就足以应对日常工作了。

iptables命令可以根据流量的源地址、目的地址、传输协议、服务类型等信息进行匹配，一旦匹配成功，iptables就会根据策略规则所预设的动作来处理这些流量。另外，再次提醒一下，<font color="red">防火墙策略规则的匹配顺序是从上至下的</font>，因此要把较为严格、优先级较高的策略规则放到前面，以免发生错误。表8-1总结归纳了常用的iptables命令参数。
**表8-1：iptables中常用的参数以及作用**
<style type="text/css">
	table th:first-of-type {
		width: 35%;
	}
</style>

参数            |	作用
----------------|----------------------
-P				| 设置默认策略
-F				| 清空规则链
-L				| 查看规则链
-A				| 在规则链的末尾加入新规则
-I num			| 在规则链的头部加入新规则
-D num			| 删除某一条规则
-s				| 匹配来源地址IP/MASK，加叹号“!”表示除这个IP外
-d				| 匹配目标地址
-i 网卡名称		| 匹配从这块网卡流入的数据
-o 网卡名称		| 匹配从这块网卡流出的数据
-p				| 匹配协议，如TCP、UDP、ICMP
--dport num		| 匹配目标端口号
--sport num		| 匹配来源端口号

至于，具体的示例可参考我的另一篇文章**《Linux 防火墙的设定》**。


#### firewalld 防火墙
RHEL 7系统中集成了多款防火墙管理工具，其中firewalld（Dynamic Firewall Manager of Linux systems，Linux系统的动态防火墙管理器）服务是默认的防火墙配置管理工具，它拥有基于CLI（命令行界面）和基于GUI（图形用户界面）的两种管理方式。

相较于传统的防火墙管理配置工具，firewalld支持动态更新技术并加入了区域（zone）的概念。简单来说，区域就是firewalld预先准备了几套防火墙策略集合（策略模板），用户可以根据生产场景的不同而选择合适的策略集合，从而实现防火墙策略之间的快速切换。例如，我们有一台笔记本电脑，每天都要在办公室、咖啡厅和家里使用。按常理来讲，这三者的安全性按照由高到低的顺序来排列，应该是家庭、公司办公室、咖啡厅。当前，我们希望为这台笔记本电脑指定如下防火墙策略规则：在家中允许访问所有服务；在办公室内仅允许访问文件共享服务；在咖啡厅仅允许上网浏览。在以往，我们需要频繁地手动设置防火墙策略规则，而现在只需要预设好区域集合，然后只需轻点鼠标就可以自动切换了，从而极大地提升了防火墙策略的应用效率。firewalld中常见的区域名称（<font color="red">默认为public</font>）以及相应的策略规则如表8-2所示。
**表8-2：firewalld中常用的区域名称及策略规则**

区域		|	默认规则策略
------------|---------------------------
trusted		| 允许所有的数据包
home		| 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh、mdns、ipp-client、amba-client与dhcpv6-client服务相关，则允许流量
internal	| 等同于home区域
work	    |拒绝流入的流量，除非与流出的流量数相关；而如果流量与ssh、ipp-client与dhcpv6-client服务相关，则允许流量
public		| 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh、dhcpv6-client服务相关，则允许流量
external	| 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh服务相关，则允许流量
dmz			| 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh服务相关，则允许流量
block		| 拒绝流入的流量，除非与流出的流量相关
drop		| 拒绝流入的流量，除非与流出的流量相关

##### 1、终端管理工具
命令行终端是一种极富效率的工作方式，firewall-cmd是firewalld防火墙配置管理工具的CLI（命令行界面）版本。它的参数一般都是以“长格式”来提供的，大家不要一听到长格式就头大，因为RHEL 7系统支持部分命令的参数补齐，其中就包含这条命令（很酷吧）。也就是说，现在除了能用Tab键自动补齐命令或文件名等内容之外，还可以用Tab键来补齐表8-3中所示的长格式参数了（这太棒了）。
**表8-3：firewall-cmd命令中使用的参数以及作用**

参数							|	作用
--------------------------------|----------------------
--get-default-zone				|	查询默认的区域名称
--set-default-zone=<区域名称>	|	设置默认的区域，使其永久生效
--get-zones						|	显示可用的区域
--get-services					|	显示预先定义的服务
--get-active-zones				|	显示当前正在使用的区域与网卡名称
--remove-source=				|	将源自此IP或子网的流量导向指定的区域
--remove-source=				|	不再将源自此IP或子网的流量导向某个指定区域
--add-interface=<网卡名称>		|	将源自该网卡的所有流量都导向某个指定区域
--change-interface=<网卡名称>	|	将某个网卡与区域进行关联
--list-all						|	显示当前区域的网卡配置参数、资源、端口以及服务等信息
--list-all-zones				|	显示所有区域的网卡配置参数、资源、端口以及服务等信息
--add-service=<服务名>			|	设置默认区域允许该服务的流量
--add-port=<端口号/协议>		|	设置默认区域允许该端口的流量
--remove-service=<服务名>		|	设置默认区域不再允许该服务的流量
--remove-port=<端口号/协议>		|	设置默认区域不再允许该端口的流量
--reload						|	让“永久生效”的配置规则立即生效，并覆盖当前的配置规则
--panic-on						|	开启应急状况模式
--panic-off						|	关闭应急状况模式

与Linux系统中其他的防火墙策略配置工具一样，使用firewalld配置的防火墙策略默认为运行时（Runtime）模式，又称为当前生效模式，而且随着系统的重启会失效。如果想让配置策略一直存在，就需要使用永久（Permanent）模式了，方法就是在用firewall-cmd命令正常设置防火墙策略时添加--permanent参数，这样配置的防火墙策略就可以永久生效了。但是，永久生效模式有一个“不近人情”的特点，就是使用它设置的策略只有在系统重启之后才能自动生效。如果想让配置的策略立即生效，需要手动执行firewall-cmd --reload命令。

##### 2、常用命令汇总
（1）安装firewalld
`yum install firewalld firewall-config`

（2）运行、停止、禁用firewalld
启动：# systemctl start  firewalld
查看状态：# systemctl status firewalld 或者 firewall-cmd --state
停止：# systemctl disable firewalld
禁用：# systemctl stop firewalld
systemctl mask firewalld
systemctl unmask firewalld

（3）配置firewalld
查看版本：`$ firewall-cmd --version`
查看帮助：`$ firewall-cmd --help`

查看设置：
显示状态：`$ firewall-cmd --state`
查看区域信息: `$ firewall-cmd --get-active-zones`
查看指定接口所属区域：`$ firewall-cmd --get-zone-of-interface=eth0`
拒绝所有包：`# firewall-cmd --panic-on`
取消拒绝状态：`# firewall-cmd --panic-off`
查看是否拒绝：`$ firewall-cmd --query-panic`

更新防火墙规则：
`# firewall-cmd --reload`
`# firewall-cmd --complete-reload`
两者的区别就是第一个无需断开连接，就是firewalld特性之一动态添加规则，第二个需要断开连接，类似重启服务

将接口添加到区域，默认接口都在public
`# firewall-cmd --zone=public --add-interface=eth0`
永久生效再加上 --permanent 然后reload防火墙

设置默认接口区域
`# firewall-cmd --set-default-zone=public`
立即生效无需重启

打开端口（貌似这个才最常用）
查看所有打开的端口：
`# firewall-cmd --zone=dmz --list-ports`
加入一个端口到区域：
`# firewall-cmd --zone=dmz --add-port=8080/tcp`
若要永久生效方法同上

打开一个服务，类似于将端口可视化，服务需要在配置文件中添加，/etc/firewalld 目录下有services文件夹，这个不详细说了，详情参考文档
`# firewall-cmd --zone=work --add-service=smtp`
移除服务
`# firewall-cmd --zone=work --remove-service=smtp`

##### 3、示例
接下来的实验都很简单，但是提醒大家一定要仔细查看使用的是Runtime模式还是Permanent模式。如果不关注这个细节，就算是正确配置了防火墙策略，也可能无法达到预期的效果。

查看firewalld服务当前所使用的区域：
```
[root@linuxprobe ~]# firewall-cmd --get-default-zone
public
```

查询eno16777728网卡在firewalld服务中的区域：
```
[root@linuxprobe ~]# firewall-cmd --get-zone-of-interface=eno16777728
public
```

把f默认irewalld服务中eno16777728网卡的区域修改为external，并在系统重启后生效。分别查看当前与永久模式下的区域名称：
```
[root@linuxprobe ~]# firewall-cmd --permanent --zone=external --change-interface=eno16777728
success
[root@linuxprobe ~]# firewall-cmd --get-zone-of-interface=eno16777728
public
[root@linuxprobe ~]# firewall-cmd --permanent --get-zone-of-interface=eno16777728
external
```

把firewalld服务的当前默认区域设置为public：
```
[root@linuxprobe ~]# firewall-cmd --set-default-zone=public
success
[root@linuxprobe ~]# firewall-cmd --get-default-zone 
public
```

启动/关闭firewalld防火墙服务的应急状况模式，阻断一切网络连接（当远程控制服务器时请慎用）：
```
[root@linuxprobe ~]# firewall-cmd --panic-on
success
[root@linuxprobe ~]# firewall-cmd --panic-off
success
```

查询public区域是否允许请求SSH和HTTPS协议的流量：
```
[root@linuxprobe ~]# firewall-cmd --zone=public --query-service=ssh
yes
[root@linuxprobe ~]# firewall-cmd --zone=public --query-service=https
no
```

把firewalld服务中请求HTTPS协议的流量设置为永久允许，并立即生效：
```
[root@linuxprobe ~]# firewall-cmd --zone=public --add-service=https
success
[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --add-service=https
success
[root@linuxprobe ~]# firewall-cmd --reload
success
```

把firewalld服务中请求HTTP协议的流量设置为永久拒绝，并立即生效：
```
[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --remove-service=http 
success
[root@linuxprobe ~]# firewall-cmd --reload 
success
```

把在firewalld服务中访问8080和8081端口的流量策略设置为允许，但仅限当前生效：
```
[root@linuxprobe ~]# firewall-cmd --zone=public --add-port=8080-8081/tcp
success
[root@linuxprobe ~]# firewall-cmd --zone=public --list-ports 
8080-8081/tcp
```

把原本访问本机888端口的流量转发到22端口，要且求当前和长期均有效：
流量转发命令格式为`firewall-cmd --permanent --zone=<区域> --add-forward-port=port=<源端口号>:proto=<协议>:toport=<目标端口号>:toaddr=<目标IP地址>`
```
[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --add-forward-port=port=888:proto=tcp:toport=22:toaddr=192.168.10.10
success
[root@linuxprobe ~]# firewall-cmd --reload
success
```

在客户端使用ssh命令尝试访问192.168.10.10主机的888端口：
```
[root@client A ~]# ssh -p 888 192.168.10.10
The authenticity of host '[192.168.10.10]:888 ([192.168.10.10]:888)' can't be established.
ECDSA key fingerprint is b8:25:88:89:5c:05:b6:dd:ef:76:63:ff:1a:54:02:1a.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[192.168.10.10]:888' (ECDSA) to the list of known hosts.
root@192.168.10.10's password:此处输入远程root管理员的密码
Last login: Sun Jul 19 21:43:48 2017 from 192.168.10.10
```

firewalld中的富规则表示更细致、更详细的防火墙策略配置，它可以针对系统服务、端口号、源地址和目标地址等诸多信息进行更有针对性的策略配置。它的优先级在所有的防火墙策略中也是最高的。比如，我们可以在firewalld服务中配置一条富规则，使其拒绝192.168.10.0/24网段的所有用户访问本机的ssh服务（22端口）：
```
[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.10.0/24" service name="ssh" reject"
success
[root@linuxprobe ~]# firewall-cmd --reload
success
```

在客户端使用ssh命令尝试访问192.168.10.10主机的ssh服务（22端口）：
```
[root@client A ~]# ssh 192.168.10.10
Connecting to 192.168.10.10:22...
Could not connect to '192.168.10.10' (port 22): Connection failed.
```