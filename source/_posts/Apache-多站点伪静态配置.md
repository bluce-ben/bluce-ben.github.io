---
title: Apache-多站点伪静态配置
date: 2017-12-13 11:07:22
categories:
- 服务器
- Apache
tags:
- Apache
---
## 一、Apache伪静态配置
1、开启http.conf中的rewrite模块
LoadModule rewrite_module modules/mod_rewrite.so  #前的注释去掉即可
检测：可使用phpinfo() 查看mod_rewrite模块是否加载
2、允许指定目录使用.htaccess
```
DocumentRoot "D:/xampp/htdocs"
<Directory "D:/xampp/htdocs">
Options Indexes FollowSymLinks Includes ExecCGI
AllowOverride All
Require all granted
</Directory>
```
3、上面两步操作完之后，后面即可在 /htdocs目录下使用.htaccess文件
<!--more-->

## 二、多站点伪静态配置
> 前提：根目录下有多个站点，且http.conf 配置文件中的网站根目录不能开启.htaccess文件识别。（否则，根目录开启.htaccess文件识别，那根目录下的子目录都会生效。）此处指的多站点配置伪静态是指对根目录下的某些站点配置。

因此多站点配置可分为两种方式：
  1. 根目录开启识别.htaccess文件
  2. 根目录下各子站开启识别.htaccess文件

多站点配置伪静态，首先需要有多站点配置。此处不予说明，前提配置与上面相同，直接上代码。
```
<VirtualHost *:80>
    ServerAdmin webmaster@admin.com
    DocumentRoot "D:/xampp/htdocs/CI"
    ServerName myblog.com
    ServerAlias www.myblog.com
    <Directory "D:/xampp/htdocs/CI">
        Options Indexes FollowSymLinks Includes ExecCGI
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog "logs/myblog.com-error.log"
    CustomLog "logs/myblog-access.log" common
</VirtualHost>
```
说明：
  * 此处的<Directory>节点必须要写，否则站点下的.htaccess文件不生效。
  * 有几个站点可以配置几个站点，都是同理。

## 三、遇到的问题及操作记录
1、启动Apache报错：
```
15:10:01  [Apache] 	Error: Apache shutdown unexpectedly.
15:10:01  [Apache] 	This may be due to a blocked port, missing dependencies, 
15:10:01  [Apache] 	improper privileges, a crash, or a shutdown by another method.
15:10:01  [Apache] 	Press the Logs button to view error logs and check
15:10:01  [Apache] 	the Windows Event Viewer for more clues
15:10:01  [Apache] 	If you need more help, copy and post this
15:10:01  [Apache] 	entire log window on the forums
```
解决办法：
1. 有错误可知最可能的原因就是端口被占用，因此可围绕端口占用查找
windows下查看端口情况及被占用情况命令：
列出所有端口的情况： netstat -ano
查找指定端口的情况： netstat -ano | findstr "80"
还可使用xampp管理界面直接查看使用端口情况，或者使用任务管理器查看进程，比对pid。
2. 此处，经查我的错误不是端口被占用，可**在命令行下面执行httpd.exe**，查看输出结果。
提示错误：
```
D:\xampp\apache\bin>httpd.exe
AH00526: Syntax error on line 47 of D:/xampp/apache/conf/extra/httpd-vhosts.conf:
ServerName takes one argument, The hostname and port of the server
```
可知是由于httpd-vhosts.conf配置文件修改错误，更改后重启成功。
