---
title: CGI、FastCGI、PHP-CGI、PHP-FPM详解
date: 2017-12-12 15:05:11
categories:
- PHP
tags:
- PHP
- PHP原理
- 服务器
---
## CGI
　CGI全称是“公共网关接口”(Common Gateway Interface)，HTTP服务器与你的或其它机器上的程序进行“交谈”的一种工具，其程序须运行在网络服务器上。
　CGI可以用任何一种语言编写，只要这种语言具有标准输入、输出和环境变量。如php,perl,tcl等
<!--more-->
![](/uploads/2017/12/cgi_01.jpg)

## FastCGI
　FastCGI像是一个常驻(long-live)型的CGI，它可以一直执行着，只要激活后，不会每次都要花费时间去fork一次(这是CGI最为人诟病的fork-and-execute 模式)。它还支持分布式的运算, 即 FastCGI 程序可以在网站服务器以外的主机上执行并且接受来自其它网站服务器来的请求。
　FastCGI是语言无关的、可伸缩架构的CGI开放扩展，其主要行为是将CGI解释器进程保持在内存中并因此获得较高的性能。众所周知，CGI解释器的反复加载是CGI性能低下的主要原因，如果CGI解释器保持在内存中并接受FastCGI进程管理器调度，则可以提供良好的性能、伸缩性、Fail-Over特性等等。
　FastCGI也可以称为是一种协议标准，比如下面要说的php-fpm就是支持解析php的一个fastCGI进程管理器/引擎。
![](/uploads/2017/12/fastcgi_01.jpg)

> **FastCGI特点**
1. FastCGI具有语言无关性.
2. FastCGI在进程中的应用程序，独立于核心web服务器运行，提供了一个比API更安全的环境。APIs把应用程序的代码与核心的web服务器链接在一起，这意味着在一个错误的API的应用程序可能会损坏其他应用程序或核心服务器。 恶意的API的应用程序代码甚至可以窃取另一个应用程序或核心服务器的密钥。
3. FastCGI技术目前支持语言有：C/C++、Java、Perl、Tcl、Python、SmallTalk、Ruby等。相关模块在Apache, ISS, Lighttpd等流行的服务器上也是可用的。
4. FastCGI的不依赖于任何Web服务器的内部架构，因此即使服务器技术的变化, FastCGI依然稳定不变。

> **FastCGI的工作原理**
1. Web Server启动时载入FastCGI进程管理器（IIS ISAPI或Apache Module)
2. FastCGI进程管理器自身初始化，启动多个CGI解释器进程(可见多个php-cgi)并等待来自Web Server的连接。
3. 当客户端请求到达Web Server时，FastCGI进程管理器选择并连接到一个CGI解释器。Web server将CGI环境变量和标准输入发送到FastCGI子进程php-cgi。
4. FastCGI子进程完成处理后将标准输出和错误信息从同一连接返回Web Server。当FastCGI子进程关闭连接时，请求便告处理完成。FastCGI子进程接着等待并处理来自FastCGI进程管理器(运行在Web Server中)的下一个连接。 在CGI模式中，php-cgi在此便退出了。

　在上述情况中，你可以想象CGI通常有多慢。每一个Web请求PHP都必须重新解析php.ini、重新载入全部扩展并重初始化全部数据结构。使用FastCGI，所有这些都只在进程启动时发生一次。一个额外的好处是，持续数据库连接(Persistent database connection)可以工作。

> **FastCGI的不足**
　因为是多进程，所以比CGI多线程消耗更多的服务器内存，PHP-CGI解释器每进程消耗7至25兆内存，将这个数字乘以50或100就是很大的内存数。
　Nginx 0.8.46+PHP 5.2.14(FastCGI)服务器在3万并发连接下，开启的10个Nginx进程消耗150M内存（15M*10=150M），开启的64个php-cgi进程消耗1280M内存（20M*64=1280M），加上系统自身消耗的内存，总共消耗不到2GB内存。如果服务器内存较小，完全可以只开启25个php-cgi进程，这样php-cgi消耗的总内存数才500M。
上面的数据摘自Nginx 0.8.x + PHP 5.2.13(FastCGI)搭建胜过Apache十倍的Web服务器(第6版)

## php-cgi
　php-cgi是php提供给web serve也就是http前端服务器的cgi协议接口程序，当每次接到http前端服务器的请求都会开启一个php-cgi进程进行处理，而且开启的php-cgi的过程中会先要重载配置，数据结构以及初始化运行环境，如果更新了php配置，那么就需要重启php-cgi才能生效，例如phpstudy就是这种情况。

## PHP-CGI
PHP-CGI是PHP自带的FastCGI管理器。

> **PHP-CGI的不足：**
1. php-cgi变更php.ini配置后需重启php-cgi才能让新的php-ini生效，不可以平滑重启。
2. 直接杀死php-cgi进程，php就不能运行了。(PHP-FPM和Spawn-FCGI就没有这个问题，守护进程会平滑从新生成新的子进程。）

## PHP-FPM
PHP-FPM是一个PHP FastCGI管理器（FastCGI Process Manager），是只用于PHP的，可以在http://php-fpm.org/download下载得到。
PHP-FPM其实是PHP源代码的一个补丁，旨在将FastCGI进程管理整合进PHP包中。必须将它patch到你的PHP源代码中，在编译安装PHP后才可以使用。
PHP5.3.3已经集成php-fpm了，不再是第三方的包了。PHP-FPM提供了更好的PHP进程管理方式，可以有效控制内存和进程、可以平滑重载PHP配置，比spawn-fcgi具有更多有点，所以被PHP官方收录了。在./configure的时候带 –enable-fpm参数即可开启PHP-FPM。

## Spawn-FCGI
Spawn-FCGI是一个通用的FastCGI管理服务器，它是lighttpd中的一部份，很多人都用Lighttpd的Spawn-FCGI进行FastCGI模式下的管理工作。


## 总结
　　fastCGI是nginx和php之间的一个通信接口，该接口实际处理过程通过启动php-fpm进程来解析php脚本，即php-fpm相当于一个动态应用服务器，从而实现nginx动态解析php。因此，如果nginx服务器需要支持php解析，需要在nginx.conf中增加php的配置：将php脚本转发到fastCGI进程监听的IP地址和端口（php-fpm.conf中指定）。同时，php安装的时候，需要开启支持fastCGI选项，并且编译安装php-fpm补丁/扩展，同时，需要启动php-fpm进程，才可以解析nginx通过fastCGI转发过来的php脚本。

> php-fpm是一个完全独立的程序,不依赖php-cgi,也不依赖php.因为php-fpm是一个内置了php解释器的FastCGI服务,启动时能够自行读取php.ini配置和php-fpm.conf配置.

> 附:PHP FastCGI进程管理器PHP-FPM的架构
一个master进程,支持多个pool,每个pool由master进程监听不同的端口,pool中有多个worker进程.
每个worker进程都内置PHP解释器,并且进程常驻后台,支持prefork动态增加.
每个worker进程支持在运行时编译脚本并在内存中缓存生成的opcode来提升性能.
每个worker进程支持配置响应指定请求数后自动重启,master进程会重启挂掉的worker进程.
每个worker进程能保持一个到MySQL/Memcached/Redis的持久连接,实现"连接池",避免重复建立连接,对程序透明.
master进程采用epoll模型异步接收和分发请求,listen监听端口,epoll_wait等待连接,
然后分发给对应pool里的worker进程,worker进程accpet请求后poll处理连接,
如果worker进程不够用,master进程会prefork更多进程,
如果prefork达到了pm.max_children上限,worker进程又全都繁忙,
这时master进程会把请求挂起到连接队列backlog里(默认值是511).

　web server（比如说nginx）只是内容的分发者。比如，如果请求/index.html，那么web server会去文件系统中找到这个文件，发送给浏览器，这里分发的是静态数据。好了，如果现在请求的是/index.php，根据配置文件，nginx知道这个不是静态文件，需要去找PHP解析器来处理，那么他会把这个请求简单处理后交给PHP解析器。Nginx会传哪些数据给PHP解析器呢？url要有吧，查询字符串也得有吧，POST数据也要有，HTTP header不能少吧，好的，CGI就是规定要传哪些数据、以什么样的格式传递给后方处理这个请求的协议。仔细想想，你在PHP代码中使用的用户从哪里来的。

　当web server收到/index.php这个请求后，会启动对应的CGI程序，这里就是PHP的解析器。接下来PHP解析器会解析php.ini文件，初始化执行环境，然后处理请求，再以规定CGI规定的格式返回处理后的结果，退出进程。web server再把结果返回给浏览器。

　好了，CGI是个协议，跟进程什么的没关系。那fastcgi又是什么呢？Fastcgi是用来提高CGI程序性能的。

　提高性能，那么CGI程序的性能问题在哪呢？"PHP解析器会解析php.ini文件，初始化执行环境"，就是这里了。标准的CGI对每个请求都会执行这些步骤（不闲累啊！启动进程很累的说！），所以处理每个时间的时间会比较长。这明显不合理嘛！那么Fastcgi是怎么做的呢？首先，Fastcgi会先启一个master，解析配置文件，初始化执行环境，然后再启动多个worker。当请求过来时，master会传递给一个worker，然后立即可以接受下一个请求。这样就避免了重复的劳动，效率自然是高。而且当worker不够用时，master可以根据配置预先启动几个worker等着；当然空闲worker太多时，也会停掉一些，这样就提高了性能，也节约了资源。这就是fastcgi的对进程的管理。

　那PHP-FPM又是什么呢？是一个实现了Fastcgi的程序，被PHP官方收了。

　大家都知道，PHP的解释器是php-cgi。php-cgi只是个CGI程序，他自己本身只能解析请求，返回结果，不会进程管理（皇上，臣妾真的做不到啊！）所以就出现了一些能够调度php-cgi进程的程序，比如说由lighthttpd分离出来的spawn-fcgi。好了PHP-FPM也是这么个东东，在长时间的发展后，逐渐得到了大家的认可（要知道，前几年大家可是抱怨PHP-FPM稳定性太差的），也越来越流行。

　fastcgi是一个协议，php-fpm实现了这个协议

　php-fpm的 管理对象 是php-cgi。但不能说php-fpm是fastcgi进程的管理器，因为前面说了fastcgi是个协议，

　以前php-fpm没有包含在PHP内核里面，要使用这个功能，需要找到与源码版本相同的php-fpm对内核打补丁，然后再编译。

　后来PHP内核集成了PHP-FPM之后就方便多了，使用--enalbe-fpm这个编译参数即可。

　有的说，修改了php.ini配置文件后，没办法 平滑重启，所以就诞生了php-fpm

　是的，修改php.ini之后，php-cgi进程的确是没办法平滑重启的。php-fpm对此的处理机制是新的worker用新的配置，已经存在的worker处理完手上的活就可以歇着了，通过这种机制来平滑过度。


## 答疑解惑
网上有的说，fastcgi是一个协议，php-fpm实现了这个协议； 有的说，php-fpm是fastcgi进程的管理器，用来管理fastcgi进程的； 有的说，php-fpm是php内核的一个补丁; 有的说，修改了php.ini配置文件后，没办法平滑重启，所以就诞生了php-fpm； 还有的说PHP-CGI是PHP自带的FastCGI管理器，那这样的话干吗又弄个php-fpm出来，我就更晕了；

**答：**
首先，CGI是干嘛的？CGI是为了保证web server传递过来的数据是标准格式的，方便CGI程序的编写者。

web server（比如说nginx）只是内容的分发者。比如，如果请求/index.html，那么web server会去文件系统中找到这个文件，发送给浏览器，这里分发的是静态数据。好了，如果现在请求的是/index.php，根据配置文件，nginx知道这个不是静态文件，需要去找PHP解析器来处理，那么他会把这个请求简单处理后交给PHP解析器。Nginx会传哪些数据给PHP解析器呢？url要有吧，查询字符串也得有吧，POST数据也要有，HTTP header不能少吧，好的，CGI就是规定要传哪些数据、以什么样的格式传递给后方处理这个请求的协议。仔细想想，你在PHP代码中使用的用户从哪里来的。

当web server收到/index.php这个请求后，会启动对应的CGI程序，这里就是PHP的解析器。接下来PHP解析器会解析php.ini文件，初始化执行环境，然后处理请求，再以规定CGI规定的格式返回处理后的结果，退出进程。web server再把结果返回给浏览器。

好了，CGI是个协议，跟进程什么的没关系。那fastcgi又是什么呢？Fastcgi是用来提高CGI程序性能的。

提高性能，那么CGI程序的性能问题在哪呢？"PHP解析器会解析php.ini文件，初始化执行环境"，就是这里了。标准的CGI对每个请求都会执行这些步骤（不闲累啊！启动进程很累的说！），所以处理每个时间的时间会比较长。这明显不合理嘛！那么Fastcgi是怎么做的呢？首先，Fastcgi会先启一个master，解析配置文件，初始化执行环境，然后再启动多个worker。当请求过来时，master会传递给一个worker，然后立即可以接受下一个请求。这样就避免了重复的劳动，效率自然是高。而且当worker不够用时，master可以根据配置预先启动几个worker等着；当然空闲worker太多时，也会停掉一些，这样就提高了性能，也节约了资源。这就是fastcgi的对进程的管理。

那PHP-FPM又是什么呢？是一个实现了Fastcgi的程序，被PHP官方收了。

大家都知道，PHP的解释器是php-cgi。php-cgi只是个CGI程序，他自己本身只能解析请求，返回结果，不会进程管理（皇上，臣妾真的做不到啊！）所以就出现了一些能够调度php-cgi进程的程序，比如说由lighthttpd分离出来的spawn-fcgi。好了PHP-FPM也是这么个东东，在长时间的发展后，逐渐得到了大家的认可（要知道，前几年大家可是抱怨PHP-FPM稳定性太差的），也越来越流行。


**下面是对上面的问题进行的回答：**
1. 好了，最后来回来你的问题。 网上有的说，fastcgi是一个协议，php-fpm实现了这个协议
对。

2. 有的说，php-fpm是fastcgi进程的管理器，用来管理fastcgi进程的
对。php-fpm的管理对象是php-cgi。但不能说php-fpm是fastcgi进程的管理器，因为前面说了fastcgi是个协议，似乎没有这么个进程存在，就算存在php-fpm也管理不了他（至少目前是）。 

3. 有的说，php-fpm是php内核的一个补丁
以前是对的。因为最开始的时候php-fpm没有包含在PHP内核里面，要使用这个功能，需要找到与源码版本相同的php-fpm对内核打补丁，然后再编译。后来PHP内核集成了PHP-FPM之后就方便多了，使用--enalbe-fpm这个编译参数即可。

4. 有的说，修改了php.ini配置文件后，没办法平滑重启，所以就诞生了php-fpm
是的，修改php.ini之后，php-cgi进程的确是没办法平滑重启的。php-fpm对此的处理机制是新的worker用新的配置，已经存在的worker处理完手上的活就可以歇着了，通过这种机制来平滑过度。

5. 还有的说PHP-CGI是PHP自带的FastCGI管理器，那这样的话干吗又弄个php-fpm
不对。php-cgi只是解释PHP脚本的程序而已。
