---
title: Docker之容器间通信
date: 2018-09-30 12:00:38
categories:
- Docker
tags:
- Docker
---
容器之间可通过 IP，Docker DNS Server 或 joined 容器三种方式通信。

#### IP 通信 ####
两个容器要能通信，必须要有属于同一个网络的网卡。
满足这个条件后，容器就可以通过 IP 交互了。具体做法是在容器创建时通过 --network 指定相应的网络，或者通过 docker network connect 将现有容器加入到指定网络。
<!--more-->

#### Docker DNS Server ####
通过 IP 访问容器虽然满足了通信的需求，但还是不够灵活。因为我们在部署应用之前可能无法确定 IP，部署之后再指定要访问的 IP 会比较麻烦。对于这个问题，可以通过 docker 自带的 DNS 服务解决。

从 Docker 1.10 版本开始，docker daemon 实现了一个内嵌的 DNS server，使容器可以直接通过“容器名”通信。方法很简单，只要在启动时用 --name 为容器命名就可以了。

下面启动两个容器 bbox1 和 bbox2：
```
docker run -it --network=my_net2 --name=bbox1 busybox
docker run -it --network=my_net2 --name=bbox2 busybox
```
然后，bbox2 就可以直接 ping 到 bbox1 了：
![](/uploads/2018/09/docker-network-communication-01.jpg)

使用 docker DNS 有个限制：只能在 user-defined 网络中使用。也就是说，默认的 bridge 网络是无法使用 DNS 的。下面验证一下：
创建 bbox3 和 bbox4，均连接到 bridge 网络。
```
docker run -it --name=bbox3 busybox
docker run -it --name=bbox4 busybox
```
bbox4 无法 ping 到 bbox3。
![](/uploads/2018/09/docker-network-communication-02.jpg)


#### joined 容器 ####
joined 容器是另一种实现容器间通信的方式。
joined 容器非常特别，它可以使两个或多个容器共享一个网络栈，共享网卡和配置信息，joined 容器之间可以通过 127.0.0.1 直接通信。请看下面的例子：

先创建一个 httpd 容器，名字为 web1。
`docker run -d -it --name=web1 httpd`
然后创建 busybox 容器并通过 `--network=container:web1` 指定 jointed 容器为 web1：
![](/uploads/2018/09/docker-network-communication-03.jpg)

请注意 busybox 容器中的网络配置信息，下面我们查看一下 web1 的网络：
![](/uploads/2018/09/docker-network-communication-04.jpg)

看！busybox 和 web1 的网卡 mac 地址与 IP 完全一样，它们共享了相同的网络栈。busybox 可以直接用 127.0.0.1 访问 web1 的 http 服务。
![](/uploads/2018/09/docker-network-communication-05.jpg)

joined 容器非常适合以下场景：
1. 不同容器中的程序希望通过 loopback 高效快速地通信，比如 web server 与 app server。
2. 希望监控其他容器的网络流量，比如运行在独立容器中的网络监控程序。

容器之间的通信我们已经搞清楚了，接下来要考虑的是容器如何与外部世界通信？


#### 容器如何访问到外部世界？ ####
**容器默认就能访问外网。**
详细信息查看博客：[传送门](https://www.cnblogs.com/CloudMan6/p/7107407.html)


#### 外部世界如何访问容器？ ####
答案是：**端口映射。**

docker 可将容器对外提供服务的端口映射到 host 的某个端口，外网通过该端口访问容器。容器启动时通过-p参数映射端口：
![](/uploads/2018/09/docker-network-communication-06.jpg)

容器启动后，可通过 docker ps 或者 docker port 查看到 host 映射的端口。在上面的例子中，httpd 容器的 80 端口被映射到 host 32773 上，这样就可以通过 `<host ip>:<32773>` 访问容器的 web 服务了。
![](/uploads/2018/09/docker-network-communication-07.jpg)

除了映射动态端口，也可在 -p 中指定映射到 host 某个特定端口，例如可将 80 端口映射到 host 的 8080 端口：
![](/uploads/2018/09/docker-network-communication-08.jpg)

每一个映射的端口，host 都会启动一个 docker-proxy 进程来处理访问容器的流量：
![](/uploads/2018/09/docker-network-communication-09.jpg)

以 0.0.0.0:32773->80/tcp 为例分析整个过程：
![](/uploads/2018/09/docker-network-communication-10.jpg)

1. docker-proxy 监听 host 的 32773 端口。
2. 当 curl 访问 10.0.2.15:32773 时，docker-proxy 转发给容器 172.17.0.2:80。
3. httpd 容器响应请求并返回结果。
