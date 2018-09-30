---
title: Docker之手动配置LNMP服务实战篇
date: 2018-09-30 12:01:07
categories:
- Docker
tags:
- Docker
---
参考博客：[传送门](https://blog.csdn.net/xy752068432/article/details/75975065)
##### 1、安装MySQL ##### 
1. 拉取MySQL5.6版本
`docker pull mysql:5.6`

2. 创建并启动一个容器
`docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=xy123456 --name test_mysql mysql:5.6`
<!--more-->

3. 进入到mysql容器后，我们通过创建一个远程可以访问的用户，这样我们就能从别的主机访问到我们的数据库了。
（注：容器中默认是没有vim的，所以我们首先要安装vim,需要注意的是安装前记得先执行apt-get update命令，不然安装会出现问题。）
`docker exec -it test_mysql /bin/bash`

>
```
进入数据库授权并新建用户
>mysql -uroot -p "xy123456"
>create database test;
>grant all privileges on test.* to 'test'@'%' identified by '123456';
>grant all privileges on *.* to 'root'@'%';
>exit;
```

##### 2、安装PHP #####
1. 拉取PHP版本
`docker pull php:7.0-fpm`

2. 创建一个phpfpm容器
`docker run -d -v $PWD/nginx/www/html:/var/www/html -p 9001:9000 --link test_mysql:mysql --name test_phpfpm php:7.0-fpm`

3. 进入容器中，并在 /var/www/html 下新建一个 index.php文件；查看宿主机中的 ./nginx/www/html下是否也有 index.php

4. 在容器中，需要安装`pdo_mysql`模块，在docker容器中可以这样来安装
`docker-php-ext-install pdo_mysql`
然后，通过命令 `php -m` 查看我们的PHP所有模块（后面会用到PDO来测试数据库的连通性）。


##### 3、安装Nginx #####
1. 拉取Nginx版本
`docker pull nginx:1.10.3`

2. 创建一个Nginx容器
`docker run -d -p 8080:80 --name test_nginx -v $PWD/nginx/www/html:/var/www/html --link test_phpfpm:phpfpm nginx:1.10.3`

3. 进入容器中
```
apt-get update
apt-get install -y vim
apt-get install -y net-tools
```
再容器里找到nginx的配置文件，并修改如下：
```
location ~ \.php$ {
    root           /var/www/html;
    fastcgi_index  index.php;
    fastcgi_pass   phpfpm:9000;//这里改成我们之前--link进来的容器，也可以直接用php容器的ip
    fastcgi_param  SCRIPT_FILENAME $document_root$fastcdi_script_name;//如果你的根目录和php容器的根目录不一样，这里的$document_root需要换成你php下的根目录，不然php就找不到文件了
    include        fastcgi_params;
}
```
**（注：docker容器中服务修改配置，需要重新启动，才能生效。）**


##### 4、测试连通性 #####
1. 直接访问nginx，检测nginx端口是否映射成功。（失败可在容器中查看nginx日志。）
2. 访问php文件，检测nginx与php的连通性。（失败可查看容器日志，`docker logs container_id -f`）
3. 通过php文件操作数据库，检测nginx、php与mysql的连通性。

检测代码：
```
<?php
try {
    $con = new PDO('mysql:host=mysql;dbname=test', 'xuye', 'xy123456');
    $con->query('SET NAMES UTF8');
    $res =  $con->query('select * from test');
    while ($row = $res->fetch(PDO::FETCH_ASSOC)) {
        echo "id:{$row['id']} name:{$row['name']}";
    }
} catch (PDOException $e) {
     echo '错误原因：'  . $e->getMessage();
}
```

总结说明：
1. 同一个host下容器之间通信，由于现在的版本，只要默认生成的容器，都会加入到Docker默认的docker0网络下。相互之间可通信。（缺点：相同端口的容器只可开启一个容器。）
2. 承接上面，在生成容器的时候，可不添加 --link仍可相互通信。（已验证。--link可更改容器中使用的容器名称）

**常用命令特殊安装：**

安装命令    |  安装包  |  命令
------------|----------|-----------
ps 			| procps   | apt-get install -y procps
netstat     | net-tools| apt-get install -y net-tools
ping 		| inetutils-ping| apt-get install inetutils-ping
