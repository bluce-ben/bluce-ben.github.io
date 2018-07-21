---
title: Nginx 实战场景配置
date: 2018-07-21 18:26:00
categories:
- 服务器
- Nginx
tags:
- Nginx
---
#### 1、静态资源配置 ####
配置语法 - sendfile
```
Syntax : sendfile on | off;
Contaxt: http, server, location, if in location
```
<!--more-->
配置语法 - tcp\_nopush （注：sendfile 开启的情况下，提高网络包的传输效率）
```
Syntax : tcp_nopush on | off;
Contaxt: http, server, location
```
配置语法 - tcp\_nodelay（注：keepalive 连接下，提高网络包的传输实时性）
```
Syntax : tcp_nodelay on | off;
Contaxt: http, server, location
```
配置语法 - 压缩
```
Syntax : gzip on | off;
Contaxt: http, server, location, if in location
```
**示例：**
```
location ~ .*\.(jpg|gif|png)$ {
	#gzip on;
	#gzip_http_version 1.1;
	#gzip_comp_level 2;
	#gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
	root /opt/app/code/images;
}
```

#### 2、浏览器缓存配置 ####
客户端校验过期机制：

校验是否过期			 |	Expires、Cache-Control(max-age)
-------------------------|-------------------------------------
协议中Etag头信息校验	 |	Etag
Last-Modified头信息校验	 |	Last-Modified

服务端配置语法：
```
Syntax : expires [modified] time;
		 expires epoch | max | off;
Contaxt: http, server, location, if in location
```
**示例：**
```
location ~ .*\.(html|htm)$ {
	expires 24h;
	root /opt/app/code;
}
```

#### 3、跨站访问配置 ####
客户端显示跨域信息：
`Access-Control-Allow-Origin`
服务端设置语法：
```
Syntax : add_header name value [always];
Contaxt: http, server, location, if in location
```
**示例：**
```
location ~ .*\.(html|htm)$ {
	add_header Access-Control-Allow-Origin http://www.url.com;
	add_header Access-Control-Allow-Methods GET,POST,PUT,DELETE,OPTIONS;
	root /opt/app/code;
}
```

#### 4、防盗链配置 ####
基于http_refer防盗链配置模块
```
Syntax : valid_referers none | blocked | server_names | string ...;
Contaxt: server, location
```
**示例：**
```
location ~ .*\.(jpg|gif|png)$ {
	valid_referers none blocked 116.62.103.228; （注：也可使用正则匹配）
	if ($invalid_referer) {
		return 443;
	}
	root /opt/app/code/images;
}
```

#### 5、代理配置 ####
>代理区别：区别在于代理的对象不一样
　　正向代理代理的对象是客户端
　　反向代理代理的对象是服务端

配置语法：
```
Syntax : proxy_pass URL;
Contaxt: location, if in location, limit_execpt
```

##### （1）正向代理 #####
服务端需限制访问路径，下面服务端只开启对应的IP能够访问：
```
location / {
	if ( $http_x_forwarded_for !~* "^116\.62\.104\.228" ) {
		return 403;
	}
	root /opt/app/code;
	index index.html index.htm;
}
```
*（注：$http_x_forwarded_for 为代理的所有信息，包括客户端的IP。）*
然后再对应的116.62.104.228服务器上配置正向代理即可。配置如下：
```
resolver 8.8.8.8;	//Google的一个dns解析
location / {
	proxy_pass http://$http_host$request_uri;
}
```
之后再客户端配置好代理服务器（116.62.104.228）即可进行访问。
（注：此处服务端配置可不限制访问也可。理解正向代理就是客户端通过代理服务器向外访问。）

##### （2）反向代理 #####
**示例：**
```
location ~ /test_proxy.html$ {
	proxy_pass http://127.0.0.1:8080;
}
```
其它代理配置语法：可浏览Nginx官网进行查看。


#### 6、负载均衡配置 ####
配置语法：
```
Syntax : upstream name {...}
Context: http
```
**示例：**
```
upstream imooc {
	server 116.62.103.228:8001 down;	//不提供服务
	server 116.62.103.228:8002 backup;	//备份结点
	server 116.62.103.228:8003 max_fails=1 fail_timeout=10s; //能够访问，最大试错为1次，服务暂停时间为10s
}
server {
	location / {
		proxy_pass http://imooc;
		include proxy_params;
	}
}
```
负载均衡默认规则是轮询。会依据访问请求来依次进行分配。当其中一台机器宕机，会停止对其访问。
**后端服务器在负载均衡调度中的状态：**

down		|	当前的server暂时不参与负载均衡
------------|--------------------------------------
backup		|	预留的备份服务器
max_fails	|	允许请求失败的次数
fail_timeout|	经过max_fails失败后，服务暂停的时间
max_conns	|	限制最大的接收的连接数

##### 调度算法：#####

轮询 		|	按时间顺序逐一分配到不同的后端服务器
------------|-------------------------------------------------------
加权轮询	|	weight值越大，分配到的访问几率越高
ip_hash		|	每个请求按访问IP的hash结果分配，这样来自同一个IP的固定访问一个后端服务器
url_hash	|	按照访问的URL的hash结果分配请求，是每个URL定向到同一个后端服务器
least_conn  |	最少链接数，那个机器连接数少就分发
hash关键数值|	hash自定义的key

url\_hash 配置语法：
```
Syntax : hash key [consistent];
Context: upstream
（This directive appeared in version 1.7.2）
```
**示例：**
```
upstream imooc {
	hash $request_uri;
	server 116.62.103.228:8001;
	server 116.62.103.228:8002;
	server 116.62.103.228:8003; 
}
```
上面示例的意思就是根据 $request_uri的值来计算hash值。此处的key可自定义，只要在配置文件中定义即可。


#### 7、缓存服务配置（Nginx） ####
proxy_cache 配置语法。