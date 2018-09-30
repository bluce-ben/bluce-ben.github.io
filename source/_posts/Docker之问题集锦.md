---
title: Docker之问题集锦
date: 2018-09-29 17:26:13
categories:
- Docker
tags:
- Docker
---
#### 一、WINDOWS下错误集锦 ####
##### 问题一#####
解决 Docker pull 出现的net/http: TLS handshake timeout 的一个办法

解决思路：百度搜了下net/http: TLS handshake timeout

出现一个这个结果比较满意
`http://dockone.io/article/876?utm_source=tuicool&utm_medium=referral`
我不用官方的dockhub了，转而使用国内的仓库daocloud
```
$ echo "DOCKER_OPTS=\"\$DOCKER_OPTS --registry-mirror=http://f2d6cb40.m.daocloud.io\"" | sudo tee -a /etc/default/docker
$ sudo service docker restart
```
重启docker服务后，再次pull 就 ok了
<!--more-->

##### 问题二 #####
运行Dockerfile启动容器的时候报错，如下：
`docker: executable file not found in $PATH`

问题出现的原因：
主要原因是docker 执行的时候没有找到对应的Dockerfile文件位置。

解决方法：
在执行`docker run`的时候，要指定Dockerfile的运行目录位置。如果DOckerfile在当前目录，则在命令后添加“.”即可。（必须项，否则会报此错误。）


##### 问题三 #####
在Windows家庭版下安装了docker，并尝试在其中运行jupyter notebook等服务，但映射完毕之后，在主机的浏览器中，打开localhost:port无法访问对应的服务。

问题出现的原因：
>The reason you’re having this, is because on Linux, the docker daemon (and your containers) run on the Linux machine itself, so “localhost” is also the host that the container is running on, and the ports are mapped to.
On Windows (and OS X), the docker daemon, and your containers cannot run natively, so only the docker client is running on your Windows machine, but the daemon (and your containers) run in a VirtualBox Virtual Machine, that runs Linux.
>
>因为docker是运行在Linux上的，在Windows中运行docker，实际上还是在Windows下先安装了一个Linux环境，然后在这个系统中运行的docker。也就是说，服务中使用的localhost指的是这个Linux环境的地址，而不是我们的宿主环境Windows。

解决方法：
通过命令 `docker-machine ip default`
其中，default 是docker-machine的name，可以通过`docker-machine -ls` 查看
找到这个Linux的ip地址，一般情况下这个地址是192.168.99.100，然后在Windows的浏览器中，输入这个地址，加上服务的端口即可启用了。
（[参考博客](https://www.cnblogs.com/hypnus-ly/p/8683215.html)）
