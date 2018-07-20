---
title: Nginx + FastCGI运行原理
date: 2018-04-09 15:17:41
categories:
- 服务器
- Nginx
tags:
- Nginx
---
#### 1、什么是 FastCGI
FastCGI是一个可伸缩地、高速地在HTTP server和动态脚本语言间通信的接口。多数流行的HTTP server都支持FastCGI，包括Apache、Nginx和lighttpd等。同时，FastCGI也被许多脚本语言支持，其中就有PHP。

FastCGI是从CGI发展改进而来的。传统CGI接口方式的主要缺点是性能很差，因为每次HTTP服务器遇到动态程序时都需要重新启动脚本解析器来执行解析，然后将结果返回给HTTP服务器。这在处理高并发访问时几乎是不可用的。另外传统的CGI接口方式安全性也很差，现在已经很少使用了。
<!--more-->
FastCGI接口方式采用C/S结构，可以将HTTP服务器和脚本解析服务器分开，同时在脚本解析服务器上启动一个或者多个脚本解析守护进程。当HTTP服务器每次遇到动态程序时，可以将其直接交付给FastCGI进程来执行，然后将得到的结果返回给浏览器。这种方式可以让HTTP服务器专一地处理静态请求或者将动态脚本服务器的结果返回给客户端，这在很大程度上提高了整个应用系统的性能。

#### 2、Nginx+FastCGI运行原理
Nginx不支持对外部程序的直接调用或者解析，所有的外部程序（包括PHP）必须通过FastCGI接口来调用。FastCGI接口在Linux下是socket（这个socket可以是文件socket，也可以是ip socket）。

wrapper：为了调用CGI程序，还需要一个FastCGI的wrapper（wrapper可以理解为用于启动另一个程序的程序），这个wrapper绑定在某个固定socket上，如端口或者文件socket。当Nginx将CGI请求发送给这个socket的时候，通过FastCGI接口，wrapper接收到请求，然后Fork(派生）出一个新的线程，这个线程调用解释器或者外部程序处理脚本并读取返回数据；接着，wrapper再将返回的数据通过FastCGI接口，沿着固定的socket传递给Nginx；最后，Nginx将返回的数据（html页面或者图片）发送给客户端。这就是Nginx+FastCGI的整个运作过程，如图1-3所示。
![](/uploads/2018/04/nginx_fastcgi.png)

所以，我们首先需要一个wrapper，这个wrapper需要完成的工作：
1. 通过调用fastcgi（库）的函数通过socket和ningx通信（读写socket是fastcgi内部实现的功能，对wrapper是非透明的）
2. 调度thread，进行fork和kill
3. 和application（php）进行通信

#### 3、spawn-fcgi与PHP-FPM
FastCGI接口方式在脚本解析服务器上启动一个或者多个守护进程对动态脚本进行解析，这些进程就是FastCGI进程管理器，或者称为FastCGI引擎。 spawn-fcgi与PHP-FPM就是支持PHP的两个FastCGI进程管理器。因此HTTPServer完全解放出来，可以更好地进行响应和并发处理。

>spawn-fcgi与PHP-FPM的异同：
1）spawn-fcgi是HTTP服务器lighttpd的一部分，目前已经独立成为一个项目，一般与lighttpd配合使用来支持PHP。但是ligttpd的spwan-fcgi在高并发访问的时候，会出现内存泄漏甚至自动重启FastCGI的问题。即：PHP脚本处理器当机，这个时候如果用户访问的话，可能就会出现白页(即PHP不能被解析或者出错)。
2）Nginx是个轻量级的HTTP server，必须借助第三方的FastCGI处理器才可以对PHP进行解析，因此其实这样看来nginx是非常灵活的，它可以和任何第三方提供解析的处理器实现连接从而实现对PHP的解析(在nginx.conf中很容易设置)。nginx也可以使用spwan-fcgi(需要一同安装lighttpd，但是需要为nginx避开端口，一些较早的blog有这方面安装的教程)，但是由于spawn-fcgi具有上面所述的用户逐渐发现的缺陷，现在慢慢减少用nginx+spawn-fcgi组合了。

由于spawn-fcgi的缺陷，现在出现了第三方(目前已经加入到PHP core中)的PHP的FastCGI处理器PHP-FPM，它和spawn-fcgi比较起来有如下优点：
* 由于它是作为PHP的patch补丁来开发的，安装的时候需要和php源码一起编译，也就是说编译到php core中了，因此在性能方面要优秀一些；
* 同时它在处理高并发方面也优于spawn-fcgi，至少不会自动重启fastcgi处理器。因此，推荐使用Nginx+PHP/PHP-FPM这个组合对PHP进行解析。
* 相对Spawn-FCGI，PHP-FPM在CPU和内存方面的控制都更胜一筹，而且前者很容易崩溃，必须用crontab进行监控，而PHP-FPM则没有这种烦恼。
* FastCGI 的主要优点是把动态语言和HTTP Server分离开来，所以Nginx与PHP/PHP-FPM经常被部署在不同的服务器上，以分担前端Nginx服务器的压力，使Nginx专一处理静态请求和转发动态请求，而PHP/PHP-FPM服务器专一解析PHP动态请求。


#### 4、Nginx+PHP-FPM
PHP-FPM是管理FastCGI的一个管理器，它作为PHP的插件存在，在安装PHP要想使用PHP-FPM时在老php的老版本（php5.3.3之前）就需要把PHP-FPM以补丁的形式安装到PHP中，而且PHP要与PHP-FPM版本一致，这是必须的）

PHP-FPM其实是PHP源代码的一个补丁，旨在将FastCGI进程管理整合进PHP包中。必须将它patch到你的PHP源代码中，在编译安装PHP后才可以使用。
PHP5.3.3已经集成php-fpm了，不再是第三方的包了。PHP-FPM提供了更好的PHP进程管理方式，可以有效控制内存和进程、可以平滑重载PHP配置，比spawn-fcgi具有更多优点，所以被PHP官方收录了。在./configure的时候带 –enable-fpm参数即可开启PHP-FPM。

fastcgi已经在php5.3.5的core中了，不必在configure时添加 --enable-fastcgi了。老版本如php5.2的需要加此项。

**当我们安装Nginx和PHP-FPM完后，配置信息：**
>PHP-FPM的默认配置php-fpm.conf：
```
listen_address  127.0.0.1:9000 #这个表示php的fastcgi进程监听的ip地址以及端口
start_servers
min_spare_servers
max_spare_servers
```

>Nginx配置运行php： 编辑nginx.conf加入如下语句：
```
location ~ \.php$ {
    root html;   
    fastcgi_pass 127.0.0.1:9000; 指定了fastcgi进程侦听的端口,nginx就是通过这里与php交互的
    fastcgi_index index.php;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME   /usr/local/nginx/html$fastcgi_script_name;
}
```

Nginx通过location指令，将所有以php为后缀的文件都交给127.0.0.1:9000来处理，而这里的IP地址和端口就是FastCGI进程监听的IP地址和端口。

**其整体工作流程：**	
1)  FastCGI进程管理器php-fpm自身初始化，启动主进程php-fpm和启动start_servers个CGI 子进程。
    主进程php-fpm主要是管理fastcgi子进程，监听9000端口。
    fastcgi子进程等待来自Web Server的连接。
2)  当客户端请求到达Web Server Nginx是时，Nginx通过location指令，将所有以php为后缀的文件都交给127.0.0.1:9000来处理，即Nginx通过location指令，将所有以php为后缀的文件都交给127.0.0.1:9000来处理。
3)  FastCGI进程管理器PHP-FPM选择并连接到一个子进程CGI解释器。Web server将CGI环境变量和标准输入发送到FastCGI子进程。
4)  FastCGI子进程完成处理后将标准输出和错误信息从同一连接返回Web Server。当FastCGI子进程关闭连接时，请求便告处理完成。
5)  FastCGI子进程接着等待并处理来自FastCGI进程管理器（运行在 WebServer中）的下一个连接。


#### 5、Nginx+PHP正确配置
一般web都做统一入口：把PHP请求都发送到同一个文件上，然后在此文件里通过解析「REQUEST_URI」实现路由。

Nginx配置文件分为好多块，常见的从外到内依次是「http」、「server」、「location」等等，缺省的继承关系是从外到内，也就是说内层块会自动获取外层块的值作为缺省值。
例如：
```
server {  
    listen 80;  
    server_name foo.com;  
    root /path;  
    location / {  
        index index.html index.htm index.php;  
        if (!-e $request_filename) {  
            rewrite . /index.php last;  
        }  
    }  
    location ~ \.php$ {  
        include fastcgi_params;  
        fastcgi_param SCRIPT_FILENAME /path$fastcgi_script_name;  
        fastcgi_pass 127.0.0.1:9000;  
        fastcgi_index index.php;  
    }  
}   
```
**1) 不应该在location 模块定义index**
一旦未来需要加入新的「location」，必然会出现重复定义的「index」指令，这是因为多个「location」是平级的关系，不存在继承，此时应该在「server」里定义「index」，借助继承关系，「index」指令在所有的「location」中都能生效。

**2) 使用try_files**
接下来看看「if」指令，说它是大家误解最深的Nginx指令毫不为过：
```
if (!-e $request_filename) {
    rewrite . /index.php last;
}
```
很多人喜欢用「if」指令做一系列的检查，不过这实际上是「try\_files」指令的职责：
`try_files $uri $uri/ /index.php;`

除此以外，初学者往往会认为「if」指令是内核级的指令，但是实际上它是rewrite模块的一部分，加上Nginx配置实际上是声明式的，而非过程式的，所以当其和非rewrite模块的指令混用时，结果可能会非你所愿。

**3）fastcgi_params」配置文件：**
`include fastcgi_params;`

Nginx有两份fastcgi配置文件，分别是「fastcgi\_params」和「fastcgi.conf」，它们没有太大的差异，唯一的区别是后者比前者多了一行「SCRIPT\_FILENAME」的定义：
`fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;`

注意：$document\_root 和 $fastcgi\_script\_name 之间没有 /。
原本Nginx只有「fastcgi\_params」，后来发现很多人在定义「SCRIPT\_FILENAME」时使用了硬编码的方式，于是为了规范用法便引入了「fastcgi.conf」。

不过这样的话就产生一个疑问：为什么一定要引入一个新的配置文件，而不是修改旧的配置文件？这是因为「fastcgi_param」指令是数组型的，和普通指令相同的是：内层替换外层；和普通指令不同的是：当在同级多次使用的时候，是新增而不是替换。换句话说，如果在同级定义两次「SCRIPT_FILENAME」，那么它们都会被发送到后端，这可能会导致一些潜在的问题，为了避免此类情况，便引入了一个新的配置文件。

此外，我们还需要考虑一个安全问题：在PHP开启「cgi.fix_pathinfo」的情况下，PHP可能会把错误的文件类型当作PHP文件来解析。如果Nginx和PHP安装在同一台服务器上的话，那么最简单的解决方法是用「try_files」指令做一次过滤：
`try_files $uri =404;`

依照前面的分析，给出一份改良后的版本，是不是比开始的版本清爽了很多：
```
server {  
    listen 80;  
    server_name foo.com;  
    root /path;  
    index index.html index.htm index.php;  
    location / {  
        try_files $uri $uri/ /index.php;  
    }  
    location ~ \.php$ {  
       try_files $uri =404;  
       include fastcgi.conf;  
       fastcgi_pass 127.0.0.1:9000;  
   }  
}  
```