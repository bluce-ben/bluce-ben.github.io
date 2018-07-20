---
title: PHP的几种运行模式
date: 2017-12-13 17:29:42
categories:
- PHP
- PHP原理
tags:
- PHP原理
- PHP
---
首先，我们来看看PHP的架构图：
![](/uploads/2017/12/php_core_framework.jpg)
<!--more-->
　　**SAPI（Server Application Programming Interface服务端应用编程端口）提供了一个和外部通信的接口，使得PHP可以和其他应用进行交互数据。** php默认提供了很多种SAPI，常见的给apache的mod_php，CGI，FastCGI，还有Shell的CLI。

## CGI
　　以CGI的方式运行，CGI英文叫做公共网关接口，就是Apache在遇到PHP脚本的时候会将PHP程序提交给CGI应用程序（php-cgi.exe）解释，CGI方式在遇到连接请求（用户 请求）先要创建cgi的子进程，激活一个CGI进程（解析器会解析php.ini文件，初始化执行环境），然后处理请求，处理完后再以CGI规定的格式返回处理后的结果给Apache，结束这个子进程。之后Apache再将结果响应给请求的用户。这就是fork-and-execute模式。所以用cgi 方式的服务器有多少连接请求就会有多少cgi子进程，子进程反复加载是cgi性能低下的主要原因。都会当用户请求数量非常多时，会大量挤占系统的资源如内 存，CPU时间等，造成效能低下。
<font color="red">（注：php-cgi 是PHP的解释器。）</font>

## FastCGI
　　以FastCGI的方式运行，目的是提高CGI程序性能。这种形式是CGI的加强版本，CGI是单进程，多线程的运行方式，程序执行完成之后就会销毁，所以每次都需要加载配置和环境变量fork-and-execute（创建-执行）。而FastCGI则不同，FastCGI 像是一个常驻 (long-live) 型的 CGI，它可以一直执行着，只要激活后，不会每次都要花费时间去 fork 一次。PHP使用PHP-FPM(FastCGI Process Manager)，全称PHP FastCGI进程管理器进行管理。
　　Web Server启动时载入FastCGI进程管理器(IIS ISAPI或Apache Module)。FastCGI进程管理器自身初始化，启动多个CGI解释器进程 (在任务管理器中可见多个php-cgi.exe)并等待来自Web Server的连接。
　　当客户端请求到达Web Server时，FastCGI进程管理器选择并连接到一个CGI解释器。Web server将CGI环境变量和标准输入发送到FastCGI子进程php-cgi。
　　FastCGI子进程完成处理后将标准输出和错误信息从同一连接返回Web Server。当FastCGI子进程关闭连接时，请求便告处理完成。FastCGI子进程接着等待并处理来自FastCGI进程管理器(运行在Web Server中)的下一个连接。 在CGI模式中，php-cgi在此便退出了。

## APACHE2HANDLER（mod_php）
　　PHP作为Apache模块，Apache服务器在系统启动后，预先生成多个进程副本驻留在内存中，一旦有请求出 现，就立即使用这些空余的子进程进行处理，这样就不存在生成子进程造成的延迟了。这些服务器副本在处理完一次HTTP请求之后并不立即退出，而是停留在计算机中等待下次请求。对于客户浏览器的请求反应更快，性能较高。
*注：实现FastCGI协议的还有 PHP-FPM（Nginx）、Spawn-FCGI（Lighttp）。*

## CLI（Command Line Interface，即命令行接口）
　　cli是php的命令行运行模式，大家经常会使用它，但是可能并没有注意到（例如：我们在linux下经常使用 “php -m”查找PHP安装了那些扩展就是PHP命令行运行模式；