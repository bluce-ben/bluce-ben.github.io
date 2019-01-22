---
title: Docker使用docker-compose.yml管理LNMP环境
date: 2019-01-22 18:47:03
categories:
- Docker
tags:
- Docker
- Docker实战
---
#### 一、生成必备的镜像文件 - php版本 #####
说明：由于官方的镜像文件不满足实际使用需求，因此需要自定义Dockerfile文件来配置PHP。
<!--more-->

##### 1、基于php5.6.40镜像版本 #####
```
ROM php:5.6-fpm
ENV PHPREDIS_VERSION 3.0.0
RUN apt-get update && apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng-dev \
    && docker-php-ext-install iconv mcrypt \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install gd pdo pdo_mysql mysqli opcache

# 安装memcached
RUN apt-get update \
        # 手动安装依赖
        && apt-get install -y libmemcached-dev zlib1g-dev \
    # 安装需要的扩展
    && pecl install memcached-2.2.0 \
    # 启用扩展
    && docker-php-ext-enable memcached

# 安装memcache
RUN pecl install memcache-2.2.7 \
    && docker-php-ext-enable memcache

# 安装redis
RUN pecl install -o -f redis \
    && rm -rf /tmp/pear \
    && docker-php-ext-enable redis

WORKDIR /usr/local/
CMD ["./sbin/php-fpm", "-c", "/usr/local/etc/php-fpm.conf"]
```
**注意点：**
* （1）安装扩展
由于php的pecl支持安装memcached、memcache、redis扩展，则使用pecl安装即可。
注：要使用docker命令 `docker-php-ext-enable` 来开启扩展。

##### 2、基于php7.2.14镜像版本 #####
```
FROM php:7.2.14-fpm
ENV PHPREDIS_VERSION 3.0.0
RUN apt-get update && apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
    && docker-php-ext-install iconv \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install gd pdo pdo_mysql mysqli opcache

# 拷贝php.ini文件
RUN cp /usr/local/etc/php/php.ini-development /usr/local/etc/php/php.ini

# 注：php7不支持pecl安装memcached memcache
# 安装memcached
RUN apt-get install -y libmemcached-dev
COPY ./memcached /tmp/memcached
RUN cd /tmp/memcached && /usr/local/bin/phpize \
    && ./configure -with-php-config=/usr/local/bin/php-config \
    && make \
    && make install
RUN docker-php-ext-enable memcached

# 安装memcache
COPY ./memcache /tmp/memcache
RUN cd /tmp/memcache && /usr/local/bin/phpize \
    && ./configure -with-php-config=/usr/local/bin/php-config \
    && make \
    && make install
RUN docker-php-ext-enable memcache

# 安装redis
RUN pecl install -o -f redis \
    && rm -rf /tmp/pear \
    && docker-php-ext-enable redis

# 清空脏数据
RUN rm -rf /tmp/*

WORKDIR /usr/local/
CMD ["./sbin/php-fpm", "-c", "/usr/local/etc/php-fpm.conf"]
```
**注意点：**
* （1）安装memcache扩展
php7不支持pecl安装memcached 和memcache扩展，因此只能使用源码安装，所以要下载源码，地址如下：
Memcached:  git clone https://github.com/php-memcached-dev/php-memcached memcached
Memcache: git clone https://github.com/websupport-sk/pecl-memcache memcache

* （2）安装memcache后，开启扩展时
不要使用下面命令把扩展添加到php.ini文件，因为镜像中设置的`--with-config-file-path`参数找不到php.ini。
`RUN echo extension=memcached.so >> /usr/local/etc/php/php.ini`
 因此，要用`docker-php-ext-enable`命令来开启扩展。

* （3）清空脏数据时
不要使用`rm -rf /tmp/`命令，该命令会删除tmp文件。导致docker构建错误：`Fatal Error Unable to create lock file: Bad file descriptor (9)`


##### 3、构建镜像文件 #####
* （1）PHP
基于上面书写的Dockerfile文件来构建镜像文件：
`docker build -t lnmp/php5.6:redis-memcache-001 .`

* （2）Nginx、mysql 
使用官方镜像即可，此处不用构建！下面会贴出对应的版本。


#### 二、使用docker-compose.yml管理镜像 #####
##### 1、docker-compose.yml文件 #####
```
version: '2'

services:
    php-fpm7:
        container_name: php-fpm7
        image: "lnmp/php7.2:redis-memcache-001"
        expose:
            - "9000"
        volumes:
            - /Users/zhengbenwu/Docker/wwwroot/:/usr/share/nginx/html
        networks:
            - lnmp

    php-fpm5:
        container_name: php-fpm5
        image: "lnmp/php5.6:redis-memcache-001"
        depends_on:
            - mysql
        expose:
            - "9000"
        volumes:
            - /Users/zhengbenwu/Docker/wwwroot/:/usr/share/nginx/html
        networks:
            - lnmp

    nginx:
        container_name: nginx
        image: "nginx:1.14.2"
        depends_on:
            - php-fpm5
        links:
            - php-fpm5:fpm5
            - php-fpm7:fpm7
        ports:
            - "80:80"
        volumes:
            - /Users/zhengbenwu/Docker/nginx/nginx.conf:/etc/nginx/nginx.conf
            - /Users/zhengbenwu/Docker/nginx/conf.d:/etc/nginx/conf.d
            - /Users/zhengbenwu/Docker/nginx/logs:/var/log/nginx
            - /Users/zhengbenwu/Docker/wwwroot/:/usr/share/nginx/html
        networks:
            - lnmp

    mysql:
        container_name: mysql
        image: mysql:5.7
        ports:
            - "3306:3306"
        environment:
            - MYSQL_ROOT_PASSWORD=123456
        volumes:
            - /Users/zhengbenwu/Docker/mysql/data:/var/lib/mysql

networks:
    lnmp:
        driver: bridge
```

#### 附常用命令： ####
##### 1、docker-compose #####
注： `-f docker-compose.yml` 参数可省略，若省略默认找docker-compose.yml文件，找不到会报错。且此参数必须紧跟docker-compse后面
开启/关闭：`docker-compose [-f docker-compose.yml] start/stop`
删除镜像：`docker-compose rm`
构建并开启：`docker-compose up -d` （注：`-d`参数表示后台执行）

##### 2、docker #####
注：下面说明的`container_id`或`image_id` 可相应的使用`container_name`或`image_name`
查看docker详细安装信息：`docker info`
查看镜像文件：`docker images`
查看容器：`docker ps [-a] （注：添加-a表示查看所有）`
查看镜像或容器详细信息：`docker inspect container_id/image_id`
进入容器：`docker exec -it container_id /bin/bash `
容器关闭/开启：`docker stop/start container_id `
删除容器：`docker rm container_id`
删除镜像：`docker rmi image_id`
构建镜像（Dockerfile）：`docker build -t image_name . （注：“.” 表示使用当前目录的Dockerfile）`
生成容器：`docker run -itd - -name container_name - -v $PWD/data:/var/data -p 8888:8888 image_id`
查看构建容器日志：`docker logs container_id`


#### 错误记录：####
1、Error response from daemon: configured logging driver does not support reading
参考文章：https://docs.docker.com/config/containers/logging/configure/
解决方法：
在生成容器的时候多加一个参数：
`docker run -itd --log-driver json-file --name container_name image_id`
注： `--log-driver`参数含义是指定日志驱动。 报错显示守护进程配置日志不支持，通过`docker inspect container_id`可查看到容器的 LoggingDriver 参数为null。因此需要指定一个日志驱动即可。


**参考博客：**
使用docker创建集成服务-lnmp：https://www.cnblogs.com/s-b-b/p/8624491.html
Dockerfile构建LNMP平台：http://blog.51cto.com/ganbing/2074640
秒懂Docker中安装扩展PHP：https://blog.csdn.net/u014389734/article/details/79683136
Dockerfile文件中添加redis扩展：https://blog.csdn.net/xiaobinqt/article/details/83105807
PHP7下安装memcache和memcached扩展：https://blog.csdn.net/u011547570/article/details/78325556
