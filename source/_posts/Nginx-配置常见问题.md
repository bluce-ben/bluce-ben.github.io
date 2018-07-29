---
title: Nginx 配置常见问题
date: 2018-07-29 18:19:56
categories:
- 服务器
- Nginx
tags:
- Nginx
---
#### 1. 相同server_name多个虚拟主机优先级访问
答： 相同的server_name，在nginx.conf文件中，会由上往下匹配，匹配成功则不往下继续匹配；而在vhost/下，则依据文件顺序由上往下进行匹配（即文件名称的顺序）。


#### 2. location匹配优先级
location方式有三种：
1. = 	进行普通字符精确匹配，也就是完全匹配
2. ^~	表示普通字符匹配，使用前缀匹配方式
3. ~/~*	表示执行一个正则匹配()
4. @ 	用来定义“Named Location”,是专门用来处理“内部重定向”请求的
说明：
（1）1、2优先级最高，匹配成功后，就不继续往下匹配。而3的优先级低，会一直往下匹配，直到找到最匹配的为止。
（2）~与~*的区别：~ 表示区分大小写；~*则表示不区分大小写。
<!--more-->

#### 3. try_files使用
答：按顺序检查文件是否存在
示例：
```
location / {
	root /opt/app/code/cache;
	try_files $uri @php_page;
}
location @php_page {
	proxy_pass http://127.0.0.1:9090;
}
```
解释：检测$uri文件是否存在，如果不存在，则重定向到@php_page模块。


#### 4. Nginx的alias和root区别
（1）root
```
location /request_path/image/ {
	root /local_path/image/;
}
```
请求链接：`http://hostname/request_path/image/cat.png`
会指向：`/local_path/image/request_path/image/cat.png`
（2）alias
```
location /request_path/image/ {
	alias /local_path/image/;
}
```
请求链接：`http://hostname/request_path/image/cat.png`
会指向：`/local_path/image/cat.png`


#### 5. 用什么方法传递用户的真实IP
可通过第一层代理增加http头变量来传递用户真实IP，后端服务可通过该变量来获取。
`set x_real_ip=$remote_addr`
*（注：`x-forwarded-for` 可被用户篡改）*

**附属一个工具用例：Nginx压测工具 ab**
示例： `ab -n 2000 -c 2 url`
说明：
-n 	总的请求数
-c 	并发数
-k 	是否开启长连接