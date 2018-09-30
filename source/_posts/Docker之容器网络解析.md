---
title: Docker之容器网络解析
date: 2018-09-30 12:00:17
categories:
- Docker
tags:
- Docker
---
#### 一、Docker原生网络 ####
Docker 网络从覆盖范围可分为单个 host 上的容器网络和跨多个 host 的网络。

##### 1、单个 host 上的容器网络 #####
Docker 安装时会自动在 host 上创建三个网络，我们可用 docker network ls 命令查看：
![](/uploads/2018/09/docker-network-01.jpg)
<!--more-->

###### （1）none 网络 ######
故名思议，none 网络就是什么都没有的网络。挂在这个网络下的容器除了 lo，没有其他任何网卡。容器创建时，可以通过 --network=none 指定使用 none 网络。
![](/uploads/2018/09/docker-network-none.jpg)

我们不禁会问，这样一个封闭的网络有什么用呢？
其实还真有应用场景。封闭意味着隔离，一些对安全性要求高并且不需要联网的应用可以使用 none 网络。
比如某个容器的唯一用途是生成随机密码，就可以放到 none 网络中避免密码被窃取。

###### （2）host 网络 ######
连接到 host 网络的容器共享 Docker host 的网络栈，容器的网络配置与 host 完全一样。可以通过 --network=host 指定使用 host 网络。
![](/uploads/2018/09/docker-network-host.jpg)

在容器中可以看到 host 的所有网卡，并且连 hostname 也是 host 的。host 网络的使用场景又是什么呢？
直接使用 Docker host 的网络最大的好处就是性能，如果容器对网络传输效率有较高要求，则可以选择 host 网络。当然不便之处就是牺牲一些灵活性，比如要考虑端口冲突问题，Docker host 上已经使用的端口就不能再用了。

Docker host 的另一个用途是让容器可以直接配置 host 网路。比如某些跨 host 的网络解决方案，其本身也是以容器方式运行的，这些方案需要对网络进行配置，比如管理 iptables。

###### （3）bridge 网络 ######
应用最广泛也是默认的 bridge 网络。
Docker 安装时会创建一个 命名为 docker0 的 linux bridge。如果不指定--network，创建的容器默认都会挂到 docker0 上。
![](/uploads/2018/09/docker-network-02.jpg)
>使用brctl命令，需要安装

当前 docker0 上没有任何其他网络设备，我们创建一个容器看看有什么变化。
![](/uploads/2018/09/docker-network-03.jpg)
一个新的网络接口 veth28c57df 被挂到了 docker0 上，veth28c57df就是新创建容器的虚拟网卡。

下面看一下容器的网络配置。
![](/uploads/2018/09/docker-network-04.jpg)
容器有一个网卡 eth0@if34。大家可能会问了，为什么不是veth28c57df 呢？

实际上 eth0@if34 和 veth28c57df 是一对 veth pair。veth pair 是一种成对出现的特殊网络设备，可以把它们想象成由一根虚拟网线连接起来的一对网卡，网卡的一头（eth0@if34）在容器中，另一头（veth28c57df）挂在网桥 docker0 上，其效果就是将 eth0@if34 也挂在了 docker0 上。

我们还看到 eth0@if34 已经配置了 IP 172.17.0.2，为什么是这个网段呢？让我们通过 docker network inspect bridge 看一下 bridge 网络的配置信息:
![](/uploads/2018/09/docker-network-05.jpg)

原来 bridge 网络配置的 subnet 就是 172.17.0.0/16，并且网关是 172.17.0.1。这个网关在哪儿呢？大概你已经猜出来了，就是 docker0。
![](/uploads/2018/09/docker-network-06.jpg)

当前容器网络拓扑结构如图所示：
![](/uploads/2018/09/docker-network-07.jpg)
容器创建时，docker 会自动从 172.17.0.0/16 中分配一个 IP，这里 16 位的掩码保证有足够多的 IP 可以供容器使用。


##### 2、跨多个 host 的网络 #####
待补充...


#### 二、自定义网络 ####
除了 none, host, bridge 这三个自动创建的网络，用户也可以根据业务需要创建 user-defined 网络。

Docker 提供三种 user-defined 网络驱动：bridge, overlay 和 macvlan。overlay 和 macvlan 用于创建跨主机的网络。

我们可通过 bridge 驱动创建类似前面默认的 bridge 网络，例如
![](/uploads/2018/09/docker-network-bridge-01.jpg)

查看一下当前 host 的网络结构变化：
![](/uploads/2018/09/docker-network-bridge-02.jpg)

新增了一个网桥 br-eaed97dc9a77，这里 eaed97dc9a77 正好新建 bridge 网络 my_net 的短 id。执行 docker network inspect 查看一下 my_net 的配置信息：
![](/uploads/2018/09/docker-network-bridge-03.jpg)

这里 172.18.0.0/16 是 Docker 自动分配的 IP 网段。

我们可以自己指定 IP 网段吗？
答案是：可以。

只需在创建网段时指定 --subnet 和 --gateway 参数：
![](/uploads/2018/09/docker-network-bridge-04.jpg)

这里我们创建了新的 bridge 网络 my_net2，网段为 172.22.16.0/24，网关为 172.22.16.1。与前面一样，网关在 my_net2 对应的网桥 br-5d863e9f78b6 上：
![](/uploads/2018/09/docker-network-bridge-05.jpg)

容器要使用新的网络，需要在启动时通过 --network 指定：
![](/uploads/2018/09/docker-network-bridge-06.jpg)

容器分配到的 IP 为 172.22.16.2。

到目前为止，容器的 IP 都是 docker 自动从 subnet 中分配，我们能否指定一个静态 IP 呢？

答案是：可以，通过--ip指定。
![](/uploads/2018/09/docker-network-bridge-07.jpg)
注：**只有使用 --subnet 创建的网络才能指定静态 IP。**

my_net 创建时没有指定 --subnet，如果指定静态 IP 报错如下：
![](/uploads/2018/09/docker-network-bridge-08.jpg)

好了，我们来看看当前 docker host 的网络拓扑结构。
![](/uploads/2018/09/docker-network-bridge-09.jpg)


#### 三、理解容器之间的连通性 ####
通过上面的实践，当前 docker host 的网络拓扑结构如下图所示
![](/uploads/2018/09/docker-network-bridge-10.jpg)

两个 busybox 容器都挂在 my_net2 上，应该能够互通，我们验证一下：
![](/uploads/2018/09/docker-network-bridge-11.jpg)
可见同一网络中的容器、网关之间都是可以通信的。

my_net2 与默认 bridge 网络能通信吗？

从拓扑图可知，两个网络属于不同的网桥，应该不能通信，我们通过实验验证一下，让 busybox 容器 ping httpd 容器：
![](/uploads/2018/09/docker-network-bridge-12.jpg)
确实 ping 不通，符合预期。

原因：**iptables DROP 掉了网桥 docker0 与 br-5d863e9f78b6 之间双向的流量。**
从规则的命名 DOCKER-ISOLATION 可知 docker 在设计上就是要隔离不同的 netwrok。

那么接下来的问题是：怎样才能让 busybox 与 httpd 通信呢？
答案是：为 httpd 容器添加一块 net_my2 的网卡。这个可以通过docker network connect 命令实现。

![](/uploads/2018/09/docker-network-bridge-13.jpg)

我们在 httpd 容器中查看一下网络配置：
![](/uploads/2018/09/docker-network-bridge-14.jpg)

容器中增加了一个网卡 eth1，分配了 my_net2 的 IP 172.22.16.3。现在 busybox 应该能够访问 httpd 了，验证一下：
![](/uploads/2018/09/docker-network-bridge-15.jpg)

busybox 能够 ping 到 httpd，并且可以访问 httpd 的 web 服务。当前网络结构如图所示：
![](/uploads/2018/09/docker-network-bridge-16.jpg)
