---
title: 一个URL的漫游详解
date: 2019-08-01 17:28:41
categories:
- 计算机网络
tags:
- 计算机网络
- 面试
---

**详细分解网络请求**
一个网络请求通过TCP/IP五层架构（此处数据链路层和物理层是分开的）进行传输，都是怎么传输的，以及都干了些什么。此文将会从网络层、传输层和应用层进行详解，各个层都做了哪些操作，以及是如何进行传输的。

下面我们将从大的方向来说明一个请求从客户端到服务端都经历了哪些过程，此处可能会遗漏，以后达到该层次，再补充。
<!--more-->

#### 一、一个HTTP的URL的生命过程
（注：此处会涉及到一个客户端缓存的问题，如果请求的是静态页面且本机上有该缓存页面，则直接读取，不会向服务器发送请求。此处忽略该请求）
总体来说分为以下六个步骤：
##### 1、DNS解析
我们都知道，正常的url是通过域名来访问的，而在服务器之间则是通过IP地址来传输的。因此，首先客户端会进行URL解析——DNS解析。DNS会先在本机查找是否有缓存，没有的话，会向DNS服务器发送请求，而DNS服务器则会递归查询，直到找到该解析，则返回，客户端则会缓存该解析。
>扩展：
DNS缓存：DNS存在着多级缓存，从离浏览器的距离排序的话，有以下几种：浏览器缓存，系统缓存，路由缓存，IPS服务器缓存，根域名服务器缓存，顶级域名服务器缓存，
DNS负载均衡：DNS可以返回一个合适的机器的IP给用户，例如可以根据每台服务器的负载量，该机器离用户地理位置的距离等，这种过程就是DNS负载均衡，又可以叫做DNS重定向，大家耳熟能详的CDN就是利用DNS的重定向技术，DNS服务器会返回一个跟用户最接近的点的IP地址给用户，CDN节点的服务器负责响应用户的请求，提供所需的内容。

##### 2、TCP连接
由DNS解析后，客户端就能够找到服务器，首先会通过TCP建立连接。

##### 3、发送HTTP请求
建立连接后，客户端就向服务器发送请求。

##### 4、服务器处理请求并返回HTTP报文
服务器接收到请求后，首先服务器会进行处理，如果是静态页面，则直接返回；否则，将该动态请求转发给后端服务处理，等待后端服务处理结束，则返回处理结果。

##### 5、浏览器解析渲染页面
客户端接收到服务端返回的结果后，进行页面渲染。展示给用户。

##### 6、连接结束
客户端断开连接。到此，一个请求周期结束。
（注：如果是keepalive长连接模式，则该连接不会断开，在timeout时间结束后，连接才会断开。）
![](/uploads/2019/04/the_whole_url.jpg '一个完整的URL请求过程')


#### 二、一个请求的建立及断开都经历了什么
我们通过监听9000端口，来捕捉一个php-fpm的请求。仔细分析一下，该请求在服务端都进行了哪些处理，以及都进行了哪些步骤。（注：此分析相当于服务器处理请求并返回结果的过程）
下面是通过`tcpdump`捕获的一个php请求：
```
[root@VM_0_7_centos conf]# tcpdump -i lo port 9000 -S -XX
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on lo, link-type EN10MB (Ethernet), capture size 262144 bytes


15:07:47.787523 IP VM_0_7_centos.39980 > VM_0_7_centos.cslistener: Flags [S], seq 682182615, win 43690, options [mss 65495,sackOK,TS val 3797694546 ecr 0,nop,wscale 7], length 0
	0x0000:  0000 0000 0000 0000 0000 0000 0800 4500  ..............E.
	0x0010:  003c 7f85 4000 4006 bd34 7f00 0001 7f00  .<..@.@..4......
	0x0020:  0001 9c2c 2328 28a9 47d7 0000 0000 a002  ...,#((.G.......
	0x0030:  aaaa fe30 0000 0204 ffd7 0402 080a e25c  ...0...........\
	0x0040:  3852 0000 0000 0103 0307                 8R........
00:58:18.801473 IP VM_0_7_centos.cslistener > VM_0_7_centos.39980: Flags [S.], seq 1894922491, ack 682182616, win 43690, options [mss 65495,sackOK,TS val 3797694546 ecr 3797694546,nop,wscale 7], length 0
	0x0000:  0000 0000 0000 0000 0000 0000 0800 4500  ..............E.
	0x0010:  003c 0000 4000 4006 3cba 7f00 0001 7f00  .<..@.@.<.......
	0x0020:  0001 2328 9c2c 70f2 38fb 28a9 47d8 a012  ..#(.,p.8.(.G...
	0x0030:  aaaa fe30 0000 0204 ffd7 0402 080a e25c  ...0...........\
	0x0040:  3852 e25c 3852 0103 0307                 8R.\8R....
15:07:47.787544 IP VM_0_7_centos.39980 > VM_0_7_centos.cslistener: Flags [.], ack 1894922492, win 342, options [nop,nop,TS val 3797694546 ecr 3797694546], length 0
	0x0000:  0000 0000 0000 0000 0000 0000 0800 4500  ..............E.
	0x0010:  0034 7f86 4000 4006 bd3b 7f00 0001 7f00  .4..@.@..;......
	0x0020:  0001 9c2c 2328 28a9 47d8 70f2 38fc 8010  ...,#((.G.p.8...
	0x0030:  0156 fe28 0000 0101 080a e25c 3852 e25c  .V.(.......\8R.\
	0x0040:  3852                                     8R
15:07:47.787571 IP VM_0_7_centos.39980 > VM_0_7_centos.cslistener: Flags [P.], seq 682182616:682183504, ack 1894922492, win 342, options [nop,nop,TS val 3797694546 ecr 3797694546], length 888
	0x0000:  0000 0000 0000 0000 0000 0000 0800 4500  ..............E.
	0x0010:  03ac 7f87 4000 4006 b9c2 7f00 0001 7f00  ....@.@.........
	0x0020:  0001 9c2c 2328 28a9 47d8 70f2 38fc 8018  ...,#((.G.p.8...
	0x0030:  0156 01a1 0000 0101 080a e25c 3852 e25c  .V.........\8R.\
	0x0040:  3852 0101 0001 0008 0000 0001 0000 0000  8R..............
	0x0050:  0000 0104 0001 034f 0100 0f17 5343 5249  .......O....SCRI
	0x0060:  5054 5f46 494c 454e 414d 452f 686f 6d65  PT_FILENAME/home
	0x0070:  2f77 7777 726f 6f74 2f69 6e64 6578 2e70  /wwwroot/index.p
	0x0080:  6870 0c00 5155 4552 595f 5354 5249 4e47  hp..QUERY_STRING
	0x0090:  0e03 5245 5155 4553 545f 4d45 5448 4f44  ..REQUEST_METHOD
	0x00a0:  4745 540c 0043 4f4e 5445 4e54 5f54 5950  GET..CONTENT_TYP
	0x00b0:  450e 0043 4f4e 5445 4e54 5f4c 454e 4754  E..CONTENT_LENGT
	0x00c0:  480b 0a53 4352 4950 545f 4e41 4d45 2f69  H..SCRIPT_NAME/i
	0x00d0:  6e64 6578 2e70 6870 0b0a 5245 5155 4553  ndex.php..REQUES
	0x00e0:  545f 5552 492f 696e 6465 782e 7068 700c  T_URI/index.php.
	0x00f0:  0a44 4f43 554d 454e 545f 5552 492f 696e  .DOCUMENT_URI/in
	0x0100:  6465 782e 7068 700d 0d44 4f43 554d 454e  dex.php..DOCUMEN
	0x0110:  545f 524f 4f54 2f68 6f6d 652f 7777 7772  T_ROOT/home/wwwr
	0x0120:  6f6f 740f 0853 4552 5645 525f 5052 4f54  oot..SERVER_PROT
	0x0130:  4f43 4f4c 4854 5450 2f31 2e31 0e04 5245  OCOLHTTP/1.1..RE
	0x0140:  5155 4553 545f 5343 4845 4d45 6874 7470  QUEST_SCHEMEhttp
	0x0150:  1107 4741 5445 5741 595f 494e 5445 5246  ..GATEWAY_INTERF
	0x0160:  4143 4543 4749 2f31 2e31 0f0c 5345 5256  ACECGI/1.1..SERV
	0x0170:  4552 5f53 4f46 5457 4152 456e 6769 6e78  ER_SOFTWAREnginx
	0x0180:  2f31 2e31 342e 300b 0d52 454d 4f54 455f  /1.14.0..REMOTE_
	0x0190:  4144 4452 3232 332e 3230 2e31 3633 2e39  ADDR223.20.163.9
	0x01a0:  350b 0452 454d 4f54 455f 504f 5254 3437  5..REMOTE_PORT47
	0x01b0:  3839 0b0a 5345 5256 4552 5f41 4444 5231  89..SERVER_ADDR1
	0x01c0:  3732 2e32 372e 302e 370b 0253 4552 5645  72.27.0.7..SERVE
	0x01d0:  525f 504f 5254 3830 0b09 5345 5256 4552  R_PORT80..SERVER
	0x01e0:  5f4e 414d 456c 6f63 616c 686f 7374 0f03  _NAMElocalhost..
	0x01f0:  5245 4449 5245 4354 5f53 5441 5455 5332  REDIRECT_STATUS2
	0x0200:  3030 090c 4854 5450 5f48 4f53 5431 3138  00..HTTP_HOST118
	0x0210:  2e32 342e 382e 3232 390f 0a48 5454 505f  .24.8.229..HTTP_
	0x0220:  434f 4e4e 4543 5449 4f4e 6b65 6570 2d61  CONNECTIONkeep-a
	0x0230:  6c69 7665 1e01 4854 5450 5f55 5047 5241  live..HTTP_UPGRA
	0x0240:  4445 5f49 4e53 4543 5552 455f 5245 5155  DE_INSECURE_REQU
	0x0250:  4553 5453 310f 7948 5454 505f 5553 4552  ESTS1.yHTTP_USER
	0x0260:  5f41 4745 4e54 4d6f 7a69 6c6c 612f 352e  _AGENTMozilla/5.
	0x0270:  3020 284d 6163 696e 746f 7368 3b20 496e  0.(Macintosh;.In
	0x0280:  7465 6c20 4d61 6320 4f53 2058 2031 305f  tel.Mac.OS.X.10_
	0x0290:  3134 5f34 2920 4170 706c 6557 6562 4b69  14_4).AppleWebKi
	0x02a0:  742f 3533 372e 3336 2028 4b48 544d 4c2c  t/537.36.(KHTML,
	0x02b0:  206c 696b 6520 4765 636b 6f29 2043 6872  .like.Gecko).Chr
	0x02c0:  6f6d 652f 3733 2e30 2e33 3638 332e 3130  ome/73.0.3683.10
	0x02d0:  3320 5361 6661 7269 2f35 3337 2e33 360b  3.Safari/537.36.
	0x02e0:  7648 5454 505f 4143 4345 5054 7465 7874  vHTTP_ACCEPTtext
	0x02f0:  2f68 746d 6c2c 6170 706c 6963 6174 696f  /html,applicatio
	0x0300:  6e2f 7868 746d 6c2b 786d 6c2c 6170 706c  n/xhtml+xml,appl
	0x0310:  6963 6174 696f 6e2f 786d 6c3b 713d 302e  ication/xml;q=0.
	0x0320:  392c 696d 6167 652f 7765 6270 2c69 6d61  9,image/webp,ima
	0x0330:  6765 2f61 706e 672c 2a2f 2a3b 713d 302e  ge/apng,*/*;q=0.
	0x0340:  382c 6170 706c 6963 6174 696f 6e2f 7369  8,application/si
	0x0350:  676e 6564 2d65 7863 6861 6e67 653b 763d  gned-exchange;v=
	0x0360:  6233 140d 4854 5450 5f41 4343 4550 545f  b3..HTTP_ACCEPT_
	0x0370:  454e 434f 4449 4e47 677a 6970 2c20 6465  ENCODINGgzip,.de
	0x0380:  666c 6174 6514 0e48 5454 505f 4143 4345  flate..HTTP_ACCE
	0x0390:  5054 5f4c 414e 4755 4147 457a 682d 434e  PT_LANGUAGEzh-CN
	0x03a0:  2c7a 683b 713d 302e 3900 0104 0001 0000  ,zh;q=0.9.......
	0x03b0:  0000 0105 0001 0000 0000                 ..........
15:07:47.787577 IP VM_0_7_centos.cslistener > VM_0_7_centos.39980: Flags [.], ack 682183504, win 356, options [nop,nop,TS val 3797694546 ecr 3797694546], length 0
	0x0000:  0000 0000 0000 0000 0000 0000 0800 4500  ..............E.
	0x0010:  0034 00ae 4000 4006 3c14 7f00 0001 7f00  .4..@.@.<.......
	0x0020:  0001 2328 9c2c 70f2 38fc 28a9 4b50 8010  ..#(.,p.8.(.KP..
	0x0030:  0164 fe28 0000 0101 080a e25c 3852 e25c  .d.(.......\8R.\
	0x0040:  3852                                     8R
15:07:49.735400 IP VM_0_7_centos.cslistener > VM_0_7_centos.39980: Flags [P.], seq 1894922492:1894922596, ack 682183504, win 356, options [nop,nop,TS val 3797696494 ecr 3797694546], length 104
	0x0000:  0000 0000 0000 0000 0000 0000 0800 4500  ..............E.
	0x0010:  009c 00af 4000 4006 3bab 7f00 0001 7f00  ....@.@.;.......
	0x0020:  0001 2328 9c2c 70f2 38fc 28a9 4b50 8018  ..#(.,p.8.(.KP..
	0x0030:  0164 fe90 0000 0101 080a e25c 3fee e25c  .d.........\?..\
	0x0040:  3852 0106 0001 0050 0000 582d 506f 7765  8R.....P..X-Powe
	0x0050:  7265 642d 4279 3a20 5048 502f 372e 302e  red-By:.PHP/7.0.
	0x0060:  3333 0d0a 436f 6e74 656e 742d 7479 7065  33..Content-type
	0x0070:  3a20 7465 7874 2f68 746d 6c3b 2063 6861  :.text/html;.cha
	0x0080:  7273 6574 3d55 5446 2d38 0d0a 0d0a 4865  rset=UTF-8....He
	0x0090:  6c6c 6f20 576f 726c 6421 0103 0001 0008  llo.World!......
	0x00a0:  0000 0000 0000 003a 2074                 .......:.t
15:07:49.735419 IP VM_0_7_centos.39980 > VM_0_7_centos.cslistener: Flags [.], ack 1894922596, win 342, options [nop,nop,TS val 3797696494 ecr 3797696494], length 0
	0x0000:  0000 0000 0000 0000 0000 0000 0800 4500  ..............E.
	0x0010:  0034 7f88 4000 4006 bd39 7f00 0001 7f00  .4..@.@..9......
	0x0020:  0001 9c2c 2328 28a9 4b50 70f2 3964 8010  ...,#((.KPp.9d..
	0x0030:  0156 fe28 0000 0101 080a e25c 3fee e25c  .V.(.......\?..\
	0x0040:  3fee                                     ?.
15:07:49.735501 IP VM_0_7_centos.cslistener > VM_0_7_centos.39980: Flags [F.], seq 1894922596, ack 682183504, win 356, options [nop,nop,TS val 3797696494 ecr 3797696494], length 0
	0x0000:  0000 0000 0000 0000 0000 0000 0800 4500  ..............E.
	0x0010:  0034 00b0 4000 4006 3c12 7f00 0001 7f00  .4..@.@.<.......
	0x0020:  0001 2328 9c2c 70f2 3964 28a9 4b50 8011  ..#(.,p.9d(.KP..
	0x0030:  0164 fe28 0000 0101 080a e25c 3fee e25c  .d.(.......\?..\
	0x0040:  3fee                                     ?.
15:07:49.735547 IP VM_0_7_centos.39980 > VM_0_7_centos.cslistener: Flags [F.], seq 682183504, ack 1894922597, win 342, options [nop,nop,TS val 3797696494 ecr 3797696494], length 0
	0x0000:  0000 0000 0000 0000 0000 0000 0800 4500  ..............E.
	0x0010:  0034 7f89 4000 4006 bd38 7f00 0001 7f00  .4..@.@..8......
	0x0020:  0001 9c2c 2328 28a9 4b50 70f2 3965 8011  ...,#((.KPp.9e..
	0x0030:  0156 fe28 0000 0101 080a e25c 3fee e25c  .V.(.......\?..\
	0x0040:  3fee                                     ?.
15:07:49.735554 IP VM_0_7_centos.cslistener > VM_0_7_centos.39980: Flags [.], ack 682183505, win 356, options [nop,nop,TS val 3797696494 ecr 3797696494], length 0
	0x0000:  0000 0000 0000 0000 0000 0000 0800 4500  ..............E.
	0x0010:  0034 00b1 4000 4006 3c11 7f00 0001 7f00  .4..@.@.<.......
	0x0020:  0001 2328 9c2c 70f2 3965 28a9 4b51 8010  ..#(.,p.9e(.KQ..
	0x0030:  0164 fe28 0000 0101 080a e25c 3fee e25c  .d.(.......\?..\
	0x0040:  3fee                                     ?.
```
分析分为三个步骤，建立连接、传输数据、断开连接，及TCP的一个周期。

##### 1、三次握手建立连接
一个连接包含了哪些数据，包头和包体。包头又包含了mac头、IP头、TCP头。下面扩展中分析了各头部所占用的位数。
需要注意区分的是，IP头的协议版本、首部长度及IP地址是什么，TCP头的端口号、数据偏移及标识位是什么。
对应上面二进制的表示，则为：
IP头的版本号为`4500`中的'4'，表示IPv4，如果为'6'，表示IPv6。
IP头的首部长度为`4500`中的'5'，表示4x5=20，说明IP头部有20个字节。
IP头的IP地址为`7f00 0001`，表示'127.0.0.1'。
TCP头的端口号为`9c2c`，表示nginx起的端口号为'39980'；`2328 `，表示php-fpm的端口号为'9000'。
TCP头的长度为`a002`中的'a'，表示10x4=40，说明TCP头部有40个字节。
TCP头的标识位为`a002`中的'02'，表示‘SYN’；`a012`中的'12'，表示‘SYN’和‘ACK’。而断开连接的`8011`中的'11'，表示'FIN'和'ACK'。

因此，建立连接的步骤为：
1) 客户端向服务端发送了一个‘SYN’，seq 682182615。
2) 服务端回了客户端一个'ACK'，并向客户端发送了一个'SYN'，seq 1894922491, ack 682182616。
3) 客户端回了服务端一个'ACK'，ack 1894922492。（注：此seq是接着服务端的seq的）
至此，客户端与服务端建立了连接。下面就是客户端向服务端发送了请求。

>扩展：
Mac头包含什么？
mac头包含了14个字节。
对应上面的二进制，即为`0000 0000 0000 0000 0000 0000 0800`。
IP头包含什么？
![](/uploads/2019/04/ip_head.jpeg 'IP首部')
对应上面的二进制，即为`7f00 0001`结束。
TCP头包含什么？
![](/uploads/2019/04/tcp_head.png 'TCP首部')
对应上面的二进制，即为`fe30 0000`结束。
注：标识位区分
TCP的标识位一共有6个，分别为URG、ACK（响应）、PSH（数据传输）、RST（连接重置）、SYN（建立连接）、FIN（关闭连接）。
对应的位数分别表示为：10，01，1000，0100，0010，0001。转成16进制则为：2，1，8，4，2，1。
TCP/IP头部大小
通过TCP的数据偏移及IP的首部长度可说明头部的大小，因为都是4位，最大表示15，因此，TCP/IP头部最大头部位15x4=60个字节。由于最低头部有20字节，因此，可扩展的头部字节数为40字节。至于为什么要乘于4，因为4表示4字节，32位。而TCP/IP头部每一行占用32位。所以，TCP/IP的长度标识位最小数应为5。

##### 2、传输数据
在发送请求的时候，客户端给服务端回了一个'ACK‘，以及向服务端发送了数据'PSH'，seq 682182616:682183504, ack 1894922492
（注：此处的682182616:682183504，表示传输的数据大小。）
传输数据的协议都有哪些？HTTP、HTTPS、fastcgi,cgi

本次分析，以fastcgi协议来说明数据是如何传输的，以及如何保证数据的正确性。
博客：https://mengkang.net/668.html
传输数据时，nginx与PHP都会按照fastcig协议来进行通信，而最后，会通过nginx来进行组装数据，返回给客户端。

>**扩展：**
各协议详解？


##### 3、四次挥手断开连接
首先，是服务端主动断开连接的，向服务端发送了‘FIN’，客户端回了一个‘ACK’，并发送了一个‘FIN’，服务端回了客户端一个‘ACK’。断开连接。


>**扩展：**
1、在浏览器与服务器通信的时候，哪端主动断开连接
 那端都可主动断开连接，一般是客户端会断开连接。 如果服务端配置的有CLB，一般是CLB对服务端断开连接。 服务器上的TIME\_WAIT一般都是请求其它服务断开的连接。例如：mysql、redis、mq、rpc等等。
2、TIME\_WAIT的产生，以及影响
 那端主动断开，TIME\_WAIT产生在那端，且存留时间为2MSL（MSL：数据报在网络中存留的最大时间）。如果并发量太高的话，会导致服务器端口被用尽，其它请求会报5xx。 解决办法，就是只能增加机器，横向扩展，服务器降级，多配几台低端机器，增加并发量。
3、服务端连接都有几种状态，以及各个状态的含义
三次握手状态：LISTEN、CLOSED、SYN\_SENT、SYN\_RCVD、ESTABLISHED
四次挥手状态：FIN\_WAIT\_1、CLOSE\_WAIT、FIN\_WAIT\_2、LAST\_ACK、TIME\_WAIT、CLOSED

**扩展知识点：**
各层之间都是透明的，互不影响的，通过接口进行调用及传输。
网络层的作用是找到主机（通过IP+mac地址）；传输层的作用是找到那个进程，即服务（通过端口）；


**经典文章：**
https://www.cnblogs.com/jacklikedogs/articles/3848263.html

