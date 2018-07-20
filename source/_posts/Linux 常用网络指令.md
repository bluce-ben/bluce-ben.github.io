---
title: Linux 常用网络指令
date: 2018-01-12 17:41:44
categories:
- 书籍
- 《鸟哥的Linux私房菜服务器架设篇（第三版）》
tags:
- 书籍
- Linux命令
---
### 一、网络参数设定使用的指令
**ifconfig ：**查询、设定网络卡与 IP 网域等相关参数；
**ifup, ifdown：**这两个档案是 script，透过更简单的方式来启动网络接口；
**route ：**查询、设定路由表 (route table)
**ip ：**复合式的指令，可以直接修改上述提到的功能；
*（先会使用 ifconfig, ifup , ifdown 与 route 即可，等以后有经验了之后，再继续回来玩 ip 这个好玩的指令。）*
<!--more-->

#### 1、手动/自动设定与启动/关闭 IP 参数： ifconfig, ifup, ifdown
```
语法如下：
[root@www ~]# ifconfig {interface} {up|down} <== 观察与启动接口
[root@www ~]# ifconfig interface {options} <== 设定与修改接口
选项与参数：
interface：网络卡接口代号，包括 eth0, eth1, ppp0 等等
options ：可以接的参数，包括如下：
    up, down ：启动 (up) 或关闭 (down) 该网络接口(不涉及任何参数)
    mtu ：可以设定不同的 MTU 数值，例如 mtu 1500 (单位为 byte)
    netmask ：就是子屏蔽网络；
    broadcast：就是广播地址啊！
```
**范例一：观察所有的网络接口(直接输入 ifconfig)**
```
[root@www ~]# ifconfig
eth0      Link encap:Ethernet  HWaddr FA:16:3E:C9:BF:42  
          inet addr:10.99.192.224  Bcast:10.99.192.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:671391959 errors:0 dropped:0 overruns:0 frame:0
          TX packets:629925596 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:202240418477 (188.3 GiB)  TX bytes:161760425688 (150.6 GiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:19514029692 errors:0 dropped:0 overruns:0 frame:0
          TX packets:19514029692 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:4097035202575 (3.7 TiB)  TX bytes:4097035202575 (3.7 TiB)
```
>至于上表出现的各项数据是这样的(数据排列由上而下、由左而右)：
**eth0：**就是网络卡的代号，也有 lo 这个 loopback ；
**HWaddr：**就是网络卡的硬件地址，俗称的 MAC 是也；
**inet addr：**IPv4 的 IP 地址，后续的 Bcast, Mask 分别代表的是 Broadcast 与 netmask！
**inet6 addr：**是 IPv6 的版本的 IP ，我们没有使用，所以略过；
**MTU：**最大传输单位
**RX：**那一行代表的是网络由启动到目前为止的封包接收情况，packets 代表封包数、errors 代表封包发生错误的数量、dropped 代表封包由于有问题而遭丢弃的数量等等
**TX：**与 RX 相反，为网络由启动到目前为止的传送情况；
**collisions：**代表封包碰撞的情况，如果发生太多次，表示你的网络状况不太好；
**txqueuelen：**代表用来传输数据的缓冲区的储存长度；
**RX bytes, TX bytes：**总接收、发送字节总量

```
ifup/ifdown 语法如下：
[root@www ~]# ifup {interface}
```

#### 2、路由修改： route
主机之间一定要有路由才能够互通 TCP/IP 的协议，否则就无法进行联机啊！一般来说，只要有网络接口，该接口就会产生一个路由，所以我们安装的主机有一个 eth0 的接口，看起来就会是这样：
```
[root@www ~]# route [-nee]
[root@www ~]# route add [-net|-host] [网域或主机] netmask [mask] [gw|dev]
[root@www ~]# route del [-net|-host] [网域或主机] netmask [mask] [gw|dev]
观察的参数：
    -n ：不要使用通讯协议或主机名，直接使用 IP 或 port number；
    -ee ：使用更详细的信息来显示
增加 (add) 与删除 (del) 路由的相关参数：
    -net ：表示后面接的路由为一个网域；
    -host ：表示后面接的为连接到单部主机的路由；
    netmask ：与网域有关，可以设定 netmask 决定网域的大小；
    gw ：gateway 的简写，后续接的是 IP 的数值，与 dev 不同；
    dev ：如果只是要指定由那一块网络卡联机出去，则使用这个设定，后面接 eth0 等
```
**范例一：单纯的观察路由状态**
```
[root@www ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
10.99.192.0     0.0.0.0         255.255.255.0   U     0      0        0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U     0      0        0 eth0
0.0.0.0         10.99.192.1     0.0.0.0         UG    0      0        0 eth0
```
>上面的信息需要知道的一些参数含义：
**Destination, Genmask：**这两个玩意儿就是分别是 network 与 netmask ！所以这两个咚咚就组合成为一个完整的网域！
**Gateway：**该网域是通过哪个 gateway 连接出去的？如果显示 0.0.0.0 表示该路由是直接由本机传送，亦即可以透过局域网络的 MAC 直接传讯；如果有显示 IP 的话，表示该路由需要经过路由器 (通讯闸) 的帮忙才能够传送出去。
**Flags：**总共有多个旗标，代表的意义如下：
    U (route is up)：该路由是启动的；
    H (target is a host)：目标是一部主机 (IP) 而非网域；
    G (use gateway)：需要透过外部的主机 (gateway) 来转递封包；
    R (reinstate route for dynamic routing)：使用动态路由时，恢复路由信息的旗标；
    D (dynamically installed by daemon or redirect)：已经由服务或转 port 功能设定为动态路由
    M (modified from routing daemon or redirect)：路由已经被修改了；
    ! (reject route)：这个路由将不会被接受(用来抵挡不安全的网域！)
**Iface：**这个路由传递封包的接口。

#### 3、网络参数综合指令： ip
他就是整合了 ifconfig 与 route 这两个指令。ifup 就是利用 ip 这个指令来达成的。
```
[root@www ~]# ip [option] [动作] [指令]
选项与参数：
option ：设定的参数，主要有：
    -s ：显示出该装置的统计数据(statistics)，例如总接受封包数等；
动作：亦即是可以针对哪些网络参数进行动作，包括有：
    link ：关于装置 (device) 的相关设定，包括 MTU, MAC 地址等等
    addr/address ：关于额外的 IP 协议，例如多 IP 的达成等等；
    route ：与路由有关的相关设定
```
由上面的语法我们可以知道， ip 除了可以设定一些基本的网络参数之外，还能够进行额外的 IP 协议，包括多 IP 的达成，真是太完美了！底下我们就分三个部分 (link,addr, route) 来介绍这个 ip 指令吧！

**（1）关于装置接口 (device) 的相关设定： ip link**
ip link 可以设定与装置 (device) 有关的相关参数，包括 MTU 以及该网络接口的 MAC 等等，当然也可以启动 (up) 或关闭 (down) 某个网络接口啦！整个语法是这样的：
```
[root@www ~]# ip [-s] link show <== 单纯的查阅该装置相关的信息
[root@www ~]# ip link set [device] [动作与参数]
选项与参数：
show：仅显示出这个装置的相关内容，如果加上 -s 会显示更多统计数据；
set ：可以开始设定项目， device 指的是 eth0, eth1 等等界面代号；
动作与参数：包括有底下的这些动作：
up|down ：启动 (up) 或关闭 (down) 某个接口，其他参数使用默认的以太网络；
address ：如果这个装置可以更改 MAC 的话，用这个参数修改！
name ：给予这个装置一个特殊的名字；
mtu ：就是最大传输单元啊！
（使用 ip link show 可以显示出整个装置接口的硬件相关信息，如上所示，包括网卡地址(MAC)、MTU 等等）
```

**（2）关于额外的 IP 相关设定： ip address**
如果说 ip link 是与 OSI 七层协定 的第二层资料连阶层有关的话，那么 ip address (ip addr) 就是与第三层网络层有关的参数啦！ 主要是在设定与 IP 有关的各项参数，包括 netmask, broadcast 等等。
```
[root@www ~]# ip address show <==就是查阅 IP 参数啊！
[root@www ~]# ip address [add|del] [IP 参数] [dev 装置名] [相关参数]
选项与参数：
show ：单纯的显示出接口的 IP 信息啊；
add|del ：进行相关参数的增加 (add) 或删除 (del) 设定，主要有：
IP 参数：主要就是网域的设定，例如 192.168.100.100/24 之类的设定；
dev ：这个 IP 参数所要设定的接口，例如 eth0, eth1 等等；
相关参数：主要有底下这些：
    broadcast：设定广播地址，如果设定值是 + 表示『让系统自动计算』
    label ：亦即是这个装置的别名，例如 eth0:0 就是了！
    scope ：这个界面的领域，通常是这几个大类：
    global ：允许来自所有来源的联机；
    site ：仅支持 IPv6 ，仅允许本主机的联机；
    link ：仅允许本装置自我联机；
    host ：仅允许本主机内部的联机；
    所以当然是使用 global ！预设也是 global ！
```

**（3）关于路由的相关设定： ip route**
这个项目当然就是路由的观察与设定啰！事实上， ip route 的功能几乎与 route 这个指令差不多，但是，他还可以进行额外的参数设计，例如 MTU 的规划等等，相当的强悍啊！
```
[root@www ~]# ip route show <==单纯的显示出路由的设定而已
[root@www ~]# ip route [add|del] [IP 或网域] [via gateway] [dev 装置]
选项与参数：
show ：单纯的显示出路由表，也可以使用 list ；
add|del ：增加 (add) 或删除 (del) 路由的意思。
IP 或网域：可使用 192.168.50.0/24 之类的网域或者是单纯的 IP ；
via ：从那个 gateway 出去，不一定需要；
dev ：由那个装置连出去，这就需要了！
mtu ：可以额外的设定 MTU 的数值喔！
```

### 二、网络侦错与观察指令
#### 1、两部主机两点沟通： ping
```
[root@www ~]# ping [选项与参数] IP
选项与参数：
-c 数值：后面接的是执行 ping 的次数，例如 -c 5 ；
-n ：在输出数据时不进行 IP 与主机名的反查，直接使用 IP 输出(速度较快)；
-s 数值：发送出去的 ICMP 封包大小，预设为 56bytes，不过你可以放大此一数值；
-t 数值：TTL 的数值，预设是 255，每经过一个节点就会少一；
-W 数值：等待响应对方主机的秒数。
-M [do|dont] ：主要在侦测网络的 MTU 数值大小，两个常见的项目是：
    do ：代表传送一个 DF (Don't Fragment) 旗标，让封包不能重新拆包与打包；
    dont：代表不要传送 DF 旗标，表示封包可以在其他主机上拆包与打包
```
**范例一：侦测一下 172.24.170.43 这部 DNS 主机是否存在？**
```
[root@www ~]# ping -c 3 172.24.170.43
PING 172.24.170.43 (172.24.170.43) 56(84) bytes of data.
64 bytes from 172.24.170.43: icmp_seq=0 ttl=118 time=3.59 ms
64 bytes from 172.24.170.43: icmp_seq=1 ttl=118 time=4.41 ms
64 bytes from 172.24.170.43: icmp_seq=2 ttl=118 time=2.48 ms

--- 172.24.170.43 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 2.485/3.495/4.411/0.792 ms, pipe 2
```
>ping 最简单的功能就是传送 ICMP 封包去要求对方主机回应是否存在于网络环境中，上面的响应消息当中，几个重要的项目是这样的：
**64 bytes：**表示这次传送的 ICMP 封包大小为 64 bytes 这么大，这是默认值，在某些特殊场合中，例如要搜索整个网络内最大的 MTU 时，可以使用 -s 2000 之类的数值来取代；
**icmp_seq=0：**ICMP 所侦测进行的次数，第一次编号为 0 ；
**ttl=118：**TTL 与 IP 封包内的 TTL 是相同的，每经过一个带有 MAC 的节点 (node) 时，例如 router, bridge 时， TTL 就会减少一，预设的 TTL 为 255 ，你可以透过 -t 150 之类的方法来重新设定预设 TTL 数值；
**time=3.59 ms：**响应时间，单位有 ms(0.001 秒)及 us(0.000001 秒)，一般来说，越小的响应时间，表示两部主机之间的网络联机越良好！
**（注：如果你忘记加上 -c 3 这样的规定侦测次数，那就得要使用 [ctrl]-c 将他结束掉了！）**

**用 ping 追踪路径中的最大 MTU 数值**
现在我们知道网络卡的 MTU 修改可以透过 ifconfig 或者是 ip 等指令来达成，那么追踪整个网络传输的最大 MTU 时，又该如何查询？最简单的方法当然是透过 ping 传送一个大封包， 并且不许中继的路由器或 switch 将该封包重组，那就能够处理啦！没错！可以这样的：
**范例二：找出最大的 MTU 数值**
```
[root@www ~]# ping -c 2 -s 1000 -M do 172.24.170.43
PING 172.24.170.43 (172.24.170.43) 1000(1028) bytes of data.
1008 bytes from 172.24.170.43: icmp_seq=0 ttl=118 time=2.28 ms
1008 bytes from 172.24.170.43: icmp_seq=1 ttl=118 time=3.40 ms

--- 172.24.170.43 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 2.281/2.843/3.406/0.565 ms, pipe 2
# 如果有响应，那就是可以接受这个封包，如果无响应，那就表示这个 MTU 太大了。
[root@www ~]# ping -c 2 -s 8000 -M do 172.24.170.43
PING 172.24.170.43 (172.24.170.43) 8000(8028) bytes of data.
From 10.99.192.224 icmp_seq=0 Frag needed and DF set (mtu = 1500)
From 10.99.192.224 icmp_seq=0 Frag needed and DF set (mtu = 1500)

--- 172.24.170.43 ping statistics ---
0 packets transmitted, 0 received, +2 errors
# 这个错误讯息是说，本地端的 MTU 才到 1500 而已，你要侦测 8000 的 MTU
# 根本就是无法达成的！那要如何是好？用前一小节介绍的 ip link 来进行MTU 设定吧！
```
不过，你需要知道的是，由于 IP 封包表头 (不含 options) 就已经占用了 20bytes ，再加上 ICMP 的表头有 8 bytes ，所以当然你在使用 -s size 的时候，那个封包的大小就得要先扣除 (20+8=28) 的大小了。 因此如果要使用 MTU 为 1500 时，就得要下达【 ping -s 1472 -M do xx.yy.zz.ip 】才行啊！

另外，由于本地端的网络卡 MTU 也会影响到侦测，所以如果想要侦测整个传输媒体的 MTU 数值，那么每个可以调整的主机就得要先使用 ifcofig 或 ip 先将 MTU 调大，然后再去进行侦测， 否则就会出现像上面提供的案例一样，可能会出现错误讯息的！

如果是要连上 Internet 的主机，注意不要随便调整 MTU ，因为我们无法知道 Internet 上面的每部机器能够支持的 MTU 到多大，因为......不是我们能够管的到的。

#### 2、两主机间各节点分析： traceroute
我们前面谈到的指令大多数都是针对主机的网络参数设定所需要的，而 ping 是两部主机之间的回声与否判断， 那么有没有指令可以追踪两部主机之间通过的各个节点(node) 通讯状况的好坏呢？举例来说，如果我们联机到 yahoo 的速度比平常慢，你觉得是 (1)自己的网络环境有问题？ (2)还是外部的 Internet 有问题？如果是 (1)的话，我们当然需要检查自己的网络环境啊，看看是否又有谁中毒了？但如果是 Internet的问题呢？那只有『等等等』啊！ 判断是 (1) 还是 (2) 就得要使用 traceroute 这个指令！
```
[root@www ~]# traceroute [选项与参数] IP
选项与参数：
-n ：可以不必进行主机的名称解析，单纯用 IP ，速度较快！
-U ：使用 UDP 的 port 33434 来进行侦测，这是预设的侦测协议；
-I ：使用 ICMP 的方式来进行侦测；
-T ：使用 TCP 来进行侦测，一般使用 port 80 测试
-w ：若对方主机在几秒钟内没有回声就宣告不治...预设是 5 秒
-p 端号：若不想使用 UDP 与 TCP 的预设埠号来侦测，可在此改变埠号。
-i 装置：用在比较复杂的环境，如果你的网络接口很多很复杂时，才会用到这个参数；
    举例来说，你有两条 ADSL 可以连接到外部，那你的主机会有两个ppp，
    你可以使用 -i 来选择是 ppp0 还是 ppp1 
-g 路由：与 -i 的参数相仿，只是 -g 后面接的是 gateway 的 IP 就是了。
```
**范例一：侦测本机到 yahoo 去的各节点联机状态**
```
[root@www ~]# traceroute -n tw.yahoo.com
traceroute to tw.yahoo.com (119.160.246.241), 30 hops max, 40 byte
packets
1 192.168.1.254 0.279 ms 0.156 ms 0.169 ms
2 172.20.168.254 0.430 ms 0.513 ms 0.409 ms
3 10.40.1.1 0.996 ms 0.890 ms 1.042 ms
4 203.72.191.85 0.942 ms 0.969 ms 0.951 ms
5 211.20.206.58 1.360 ms 1.379 ms 1.355 ms
6 203.75.72.90 1.123 ms 0.988 ms 1.086 ms
7 220.128.24.22 11.238 ms 11.179 ms 11.128 ms
8 220.128.1.82 12.456 ms 12.327 ms 12.221 ms
9 220.128.3.149 8.062 ms 8.058 ms 7.990 ms
10 * * *
11 119.160.240.1 10.688 ms 10.590 ms 119.160.240.3 10.047 ms
12 * * * <==可能有防火墙装置等情况发生所致
```
这个 traceroute 挺有意思的，这个指令会针对欲连接的目的地之所有 node 进行 UDP 的逾时等待，例如上面的例子当中，由鸟哥的主机连接到 Yahoo 时，他会经过 12 个节点以上，traceroute 会主动的对这 12 个节点做 UDP 的回声等待，并侦测回复的时间，每节点侦测三次，最终回传像上头显示的结果。 你可以发现每个节点其实回复的时间大约在 50 ms 以内，算是还可以的 Internet 环境了。

比较特殊的算是第 10/12 个，会回传星号的，代表该 node 可能设有某些防护措施，让我们发送的封包信息被丢弃所致。 因为我们是直接透过路由器转递封包，并没有进入路由器去取得路由器的使用资源，所以某些路由器仅支持封包转递，并不会接受来自客户端的各项侦测啦！此时就会出现上述的问题。

#### 3、察看本机的网络联机与后门： netstat
如果你觉得你的某个网络服务明明就启动了，但是就是无法造成联机的话，那么应该怎么办？首先你应该要查询一下自己的网络接口所监听的端口 (port) 来看看是否真的有启动，因为有时候屏幕上面显示的 [OK] 并不一定是 OK 啊！
```
[root@www ~]# netstat -[rn] <==与路由有关的参数
[root@www ~]# netstat -[antulpc] <==与网络接口有关的参数
选项与参数：
与路由 (route) 有关的参数说明：
-r ：列出路由表(route table)，功能如同 route 这个指令；
-n ：不使用主机名与服务名称，使用 IP 与 port number ，如同 route -n
与网络接口有关的参数：
-a ：列出所有的联机状态，包括 tcp/udp/unix socket 等；
-t ：仅列出 TCP 封包的联机；
-u ：仅列出 UDP 封包的联机；
-l ：仅列出有在 Listen (监听) 的服务之网络状态；
-p ：列出 PID 与 Program 的檔名；
-c ：可以设定几秒钟后自动更新一次，例如 -c 5 每五秒更新一次网络状态的显示；
（netstat -rn 与 route -n 是相同的。）
```
>我们先来谈一谈关于网络联机状态的输出部分，他主要是分为底下几个大项：
**Proto：**该联机的封包协议，主要为 TCP/UDP 等封包；
**Recv-Q：**非由用户程序连接所复制而来的总 bytes 数；
**Send-Q：**由远程主机所传送而来，但不具有 ACK 标志的总 bytes 数，意指主动联机 SYN 或其他标志的封包所占的 bytes 数；
**Local Address：**本地端的地址，可以是 IP (-n 参数存在时)，也可以是完整的主机名。使用的格是就是『 IP:port 』只是 IP 的格式有 IPv4 及 IPv6 的差异。 如上所示，在 port 22 的接口中，使用的 :::22 就是针对 IPv6 的显示，事实上他就相同于 0.0.0.0:22 的意思。至于 port 25 仅针对 lo 接口开放，意指 Internet 基本上是无法连接到我本机的 25 埠口啦！
**Foreign Address：**远程的主机 IP 与 port number
**stat：**状态栏，主要的状态含有：
      ESTABLISED：已建立联机的状态；
      SYN_SENT：发出主动联机 (SYN 标志) 的联机封包；
      SYN_RECV：接收到一个要求联机的主动联机封包；
      FIN_WAIT1：该插槽服务(socket)已中断，该联机正在断线当中；
      FIN_WAIT2：该联机已挂断，但正在等待对方主机响应断线确认的封包；
      TIME_WAIT：该联机已挂断，但 socket 还在网络上等待结束；
      LISTEN：通常用在服务的监听 port ！可使用『 -l 』参数查阅。

基本上，我们常常谈到的 netstat 的功能，就是在观察网络的联机状态了，而网络联机状态中，又以观察【我目前开了多少的 port 在等待客户端的联机】以及 【目前我的网络联机状态中，有多少联机已建立或产生问题】最常见。