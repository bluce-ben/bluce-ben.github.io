---
title: 图解TCP连接及对TIME_WAIT的理解
date: 2018-12-19 19:14:58
categories:
- 计算机网络
tags:
- 计算机网络
---
#### 环境说明：
访问路径：http://118.24.8.229/index.html
index.html代码：
```
www.zhengbenwu.com
<img src="./apple.jpg" />
<img src="./ceshi.webp" />
```
监控工具：Wireshark
监控筛选条件：`ip.addr == 118.24.8.229 && tcp.port == 80`
<!--more-->

#### 监控说明：
##### （1）建立连接
![](/uploads/2018/12/network_tcp_wireshark_01.png)
客户端开启了3个连接请求，相应的客户端也会占用3个端口：59746、59747、59748。通过三次握手建立连接，图示连接建立过程很清晰。

客户端监控端口：
![](/uploads/2018/12/network_tcp_client_01.png)

服务端监控端口：
![](/uploads/2018/12/network_tcp_server_01.png)


##### （2）传输数据
index.html 文件传输：通过59747端口传输
![](/uploads/2018/12/network_tcp_file_01.png)

apple.jpg 文件传输：通过59747端口传输
![](/uploads/2018/12/network_tcp_file_02.png)

ceshi.webp 文件传输：通过59748端口传输
![](/uploads/2018/12/network_tcp_file_03.png)


##### （3）断开连接
![](/uploads/2018/12/network_tcp_unconnect.png)
① 59746端口连接
由图示可以看出，首先断开的是59746端口的连接，且是客户端主动断开连接。因此，客户端会产生TIME_WAIT。
![](/uploads/2018/12/network_tcp_client_02.png)
![](/uploads/2018/12/network_tcp_server_02.png)

>要注意的是，在此URL请求里，客户端建立的59746端口连接并没有用到；并且，很快就被断开了，并没有保持持久连接检测。

② 59747、59748端口连接
由上图示可以看出来，这两个端口是服务端主动断开连接的。中间客户端发送了 Keep-Alive保持连接，之后服务端在keepalive_timeout到时间后，自动断开了连接。因此，服务端会产生TIME_WAIT。
![](/uploads/2018/12/network_tcp_client_03.png)
![](/uploads/2018/12/network_tcp_server_03.png)

>由此可知道，网站如果量比较大的话，尽量不要使用长连接，会产生大量的TIME_WAIT，导致网站瘫痪。
量大的话，服务器可采用降配横向扩展，多部署几台服务器，提高端口数量，降低TIME_WAIT。


#### 问题思考：
##### 长连接中TCP是如何请求的？
实验一：客户端请求连接中，有多个额外请求时，TCP使用是怎么样的？
服务端程序改成：
（1）1个html和1个图片
```
www.zhengbenwu.com
<img src="./apple.jpg" />
```
结论：
客户端仍然会产生3个TCP连接，并且由于html的数据量少，导致服务端只使用了一个TCP连接就传输了html页面和图片数据。客户端产生2个TIME_WAIT，服务端产生1个TIME_WAIT。

（2）1个html
```
www.zhengbenwu.com
```
结论：
客户端仍然会产生3个TCP连接，传输数据的TCP连接会保持持久连接，其余未传输数据的客户端会主动断开连接，不检测持久连接。客户端产生2个TIME_WAIT，服务端产生1个TIME_WAIT。
![](/uploads/2018/12/network_tcp_wireshark_02.png)

（3）1个html和4个图片
```
www.zhengbenwu.com
<img src="./apple.jpg" />
<img src="./ceshi.webp" />
<img src="./timg.jpg" />
<img src="./test001.jpg" />
```
结论：
客户端初始仍会产生3个TCP连接，首先建立连接的TCP会先传输html数据，当传输完成后，会自动的继续传输前3个图片，当发现3个TCP连接都用完后，仍有一个图片未传输，客户端会再新建一个TCP连接，用于传输第四个图片。 此时，TCP连接建立了4个，且都会检测是否保持持久连接，因此最后服务端会产生4个TIME_WAIT。
![](/uploads/2018/12/network_tcp_server_04.png)

（4）1个html和7个图片
```
www.zhengbenwu.com
<img src="./apple.jpg">
<img src="./ceshi.webp">
<img src="./timg.jpg">
<img src="./test001.jpg">
<img src="./test002.jpg">
<img src="./test003.jpg">
<img src="./test004.jpg">
```
结论：
客户端初始会产生6个TCP连接，并且复用连接。断开TCP连接的是服务端，因此最后服务端会产生6个TIME_WAIT。
![](/uploads/2018/12/network_tcp_server_05.png)

解决方案：
线上环境可使用负载均衡。


##### 短连接中TCP是如何请求的？
实验二：使用短连接（在服务端Nginx配置 `keepalive_timeout 0;`）
（1）1个html和7个图片
```
www.zhengbenwu.com
<img src="./apple.jpg">
<img src="./ceshi.webp">
<img src="./timg.jpg">
<img src="./test001.jpg">
<img src="./test002.jpg">
<img src="./test003.jpg">
<img src="./test004.jpg">
```
结论：
在使用短连接的情况下，每个请求都会建立一个TCP连接，不会复用。并且，测试显示都是服务端断开的连接，因此服务端会产生TIME_WAIT。
![](/uploads/2018/12/network_tcp_server_06.png)


##### TIME_WAIT在服务端与客户端的理解
我理解的TIME_WAIT有两种，一种是作为客户端产生的TIME_WAIT，一种是作为服务端产生的TIME_WAIT。
这里，我们先说一下请求的过程。首先，客户端会新建一个进程，并随机分配一个端口号，来与服务端的固定端口号进行连接建立。因此，这一条请求就有两个端口，一个是客户端的随机端口，一个是服务端的固定端口。那么，下面来说结论：
① 对于客户端来说
客户端主动断开连接，TIME_WAIT产生在客户端。那么，服务端的这条TCP连接会被迅速回收；而客户端会耗时2MSL才被回收，端口才会被释放。

② 对于服务端来说
服务端主动断开连接，TIME_WAIT产生在服务端。客户端的TCP连接会被迅速回收，端口释放，重复使用；而服务端会耗时2MSL才会释放该条连接。

请注意，这里服务端与TCP建立连接并没有占用端口号，服务端的固定端口号一直在监听服务，并不会被请求占用，而是复用端口。这里与客户端建立连接的是服务端的线程或子进程，会复制主进程的资源，进行处理请求，然后，返回给客户端。
因此，服务端产生的TIME_WAIT并不会消耗端口号，只会消耗线程或子进程资源。当请求量达到服务器承载的极限值时，服务端会返回错误。
<font color="red">（注：由上可知，一般说的TIME_WAIT，是指服务端作为客户端发起的请求产生的TIME_WAIT。过多会导致端口号不够，造成错误。）</font>


##### 那端会产生TIME_WAIT
主动断开连接的那端会产生TIME_WAIT。 那又有个问题，那端会先断开连接呢？什么情况下客户端先断，什么情况下服务端先断？
1. 对于http1.0协议来说，如果响应头中有content-length头，则以content-length的长度就可以知道body的长度了，客户端在接收body时，就可以依照这个长度来接收数据，接收完后，就表示这个请求完成了。而如果没有content-length头，则客户端会一直接收数据，直到服务端主动断开连接，才表示body接收完了。
2. 而对于http1.1协议来说，如果响应头中的Transfer-encoding为chunked传输，则表示body是流式输出，body会被分成多个块，每块的开始会标识出当前块的长度，此时，body不需要通过长度来指定。如果是非chunked传输，而且有content-length，则按照content-length来接收数据。否则，如果是非chunked，并且没有content-length，则客户端接收数据，直到服务端主动断开连接。

**总结：**
* http1.0  
带content-length，body长度可知，客户端在接收body时，就可以依据这个长度来接受数据。接受完毕后，就表示这个请求完毕了。客户端主动调用close进入四次挥手。
不带content-length ，body长度不可知，客户端一直接受数据，直到服务端主动断开
* http1.1
带content-length，body长度可知，客户端主动断开
带Transfer-encoding：chunked，body会被分成多个块，每块的开始会标识出当前块的长度，body就不需要通过content-length来指定了。但依然可以知道body的长度 客户端主动断开
不带Transfer-encoding：chunked且不带content-length，客户端接收数据，直到服务端主动断开连接。
