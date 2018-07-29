---
title: Nginx 配置HTTPS及生成CA证书
date: 2018-07-29 18:23:12
categories:
- 服务器
- Nginx
tags:
- Nginx
---
#### 原理详解 ####
##### （1）HTTPS 采用对称加密和非对称加密两种形式。##### 
**对称加密：**
明文M -> 加密算法（包含加密秘钥） -> 生成密文Y发送 -> 解密算法（包含加密秘钥） -> 明文M
（注：加密算法中的秘钥 = 解密算法中的秘钥）

**非对称加密：**
发送方：明文M -> 加密（通过会话秘钥（服务器端生成的公钥）及加密算法） -> 生成密文Y
接收方：接收密文Y -> 解密（通过会话秘钥（服务器端生成的私钥）及加密算法） ->明文M
*（注：服务器端生成公私钥，公钥可以公开用于客户端加密传输数据，秘钥保管在服务器端，用于解密数据。私钥很重要，不能泄露。）*
<!--more-->
##### （2）HTTPS 加密协议原理： #####
客户端		-> 发起SSL连接		-> 服务器端（保管唯一私钥）
　　　　　　<- 发送公钥         
　　　　　　->发送对称密码（发送对称密码利用公钥加密）
　　　　　　<->
　　（利用对称秘钥传输数据） 

>此种方式安全漏洞：
会出现一种中间人：
① 在客户端“发起SSL连接”的时候，伪装成服务端接受连接，并再次伪装成客户端向服务端发送SSL连接；
② 当服务端接受SSL连接的时候，会向客户端发送“公钥”，然后中间人会伪装成客户端接受“公钥”，就会获取公钥；
③ 然后中间人会伪装成服务端向真正客户端发送“公钥”，获取客户端利用“公钥”返回的“对称密码”；
④ 到此，中间人就把传输数据的安全信息都获取了，可以进行伪装传输数据了。

**为解决此安全漏洞，HTTPS采用了数字证书的方式（即CA证书）。**
原理即是在发送公钥这一层，将服务端返回公钥改成发送“CA签名证书”，然后，客户端已经安装了服务端生成的证书。对服务端返回的证书进行验证即可。校验通过，客户端发送“对称密码”；校验失败则停止会话。


#### 秘钥及证书生成 ####
准备工作：Nginx必备模块
```
#openssl version
OpenSSL 1.0.2k-fips  26 Jan 2017
#nginx -V
--with-http_ssl_module
```

##### （1）生成key秘钥 #####
`openssl genrsa -idea -out jesonc.key 1024`

##### （2）生成证书签名请求文件（csr文件） ##### 
`openssl req -new -key jesonc.key -out jesonc.csr`

##### （3）生成证书签名文件（CA文件） ##### 
`openssl x509 -req -days 3650 -in jesonc.csr -signkey jesonc.key -out jesonc.crt`

##### （4）通过秘钥直接生成证书文件 ##### 
`openssl req -days 3650 -x509 -sha256 -nodes -newkey rsa:2048 -keyout jesonc.key -out jesonc_apple.crt`

##### （5）免密启动服务设置 ##### 
`openssl rsa -in ./jesonc.key -out ./jesonc_nopass.key`


#### HTTPS语法配置 ####
```
Syntax： ssl on | off;
Context：http, server
```

**示例：**
```
server{
	listen		443;
	server_name ip;
	ssl 	on;
	ssl_certificate 	/etc/nginx/ssl_key/jesonc.crt;
	ssl_certificate_key	/etc/nginx/ssl_key/jesonc.key;
	...
}
```

>场景 - 配置苹果要求的证书
1. 服务器所有的连接使用TLS1.2以上版本（openssl 1.0.2）
2. HTTPS证书必须使用SHA256以上哈希算法签名
3. HTTPS证书必须使用RSA 2048位或ECC 256位以上公钥算法
4. 使用向前加密技术

*（注：查看本地证书要求：`openssl x509 -noout -text -in ./jesonc.crt`）*


#### HTTPS服务优化 ####
（1）激活keepalive长连接
（2）设置 ssl session缓存
示例：
```
keepalive_timeout	100;

ssl on;
ssl_session_cache    shared:SSL:10m;
ssl_session_timeout  10m;
```

#### 升级SSL版本 ####
（1）下载：`wget http://www.openssl.org/source/openssl-1.0.2k.tar.gz`
（2）解压及安装
```
tar -zxvf openssl-1.0.2k.tar.gz
cd openssl-1.0.2k
./config --prefix=/usr/local/openssl
make && make install
```
（3）将旧的OpenSSL备份
```
mv /usr/bin/openssl /usr/bin/openssl.OFF
mv /usr/include/openssl /usr/include/openssl.OFF
```
（4）将新的OpenSSL软链接到指定位置
```
ln -s /usr/local/openssl/bin/openssl /usr/bin/openssl
ln -s /usr/local/openssl/include/openssl /usr/include/openssl
```
（5）将新安装的openssl的库路径追加到系统的库文件的搜索路径中
`echo "/usr/local/openssl/lib" >> /etc/ld.so.conf`
（6）加载系统库文件，使修改后的/etc/ld.so.conf生效 
`ldconfig -v`
（7）查看openssl是否已更新成功
`openssl version -a`