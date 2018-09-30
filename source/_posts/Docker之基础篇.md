---
title: Docker之基础篇
date: 2018-09-28 16:32:51
categories:
- Docker
tags:
- Docker
---
#### 一、常用命令 ####
##### 1、容器生命周期管理 #####
1. 运行容器
```
docker run [-it] [-d] [--name container_name] [-p container_port:host_port] imageId
参数详解：
-i  打开容器的标准输入STDIN。
-t 	为容器建立一个命令行终端。Ctrl+D退出容器
-d 	以deamon后台运行
-p  宿主机与容器端口建立映射
--name 	为容器命名一个name
```
<!--more-->
2. 进入运行中的容器
`docker exec -it container_id /bin/sh`

3. 查看运行中的容器日志信息
`docker logs container_id`

4. 停止所有容器
`docker stop $(docker ps -qa)`

5. 删除所有容器
`docker rm [-f] $(docker ps -qa)`

6. 删除所有镜像
`docker rmi [-f] $(docker ps -qa)`

7. 导入导出(未操作，待添加)
`docker load`
`docker save`
`docker export`
`docker import`


##### 2、容器操作 #####
1. 新建容器
`docker build -t container_name/container_tag` ./

2. 容器开启/关闭/重启
`docker start/stop/restart container_id	`

3. 查看容器详细信息
`docker inspect container_id`

4. 查看容器端口信息
`docker port container_id`

5. 查看运行中的容器
`docker ps`

6. 重新命名容器名称
`docker rename container_id new_name`

7. 删除容器
`docker rm [-f] container_id`

8. 进入容器（进入到容器启动命令的终端）
`docker attach 94ab7a046f7c`

##### 3、镜像仓库 #####
1. 查看docker信息
`docker info`
（要点：Registry: https://index.docker.io/v1/ 镜像仓库地址）

2. 查看本地镜像文件
`docker images`

3. 查看镜像
`docker search whalesay`  //从镜像仓库中查询镜像文件

4. 拉取镜像
`docker pull whalesay`

5. 推送本地镜像到远程仓库（注：必须先登录，之后才能成功推送。登录使用`docker login`）
`docker push myname/whalesay`

6. 删除镜像
`docker rmi [-f] container_id`

7. Docker重命名镜像名称和TAG
`docker tag IMAGEID(镜像id) REPOSITORY:TAG（仓库：标签）`

#### 二、Docker概念 ####
Docker是开发人员和系统管理员开发、部署和运行带有容器的应用程序的平台。使用Linux容器来部署应用程序称为“容器化”。容器不是一个新概念，但是使用它们能够轻松部署应用程序。

**容器越来越受欢迎，有如下优点：**
* 灵活：即使是最复杂的应用程序也可以被容器化
* 轻量级：容器利用并共享宿主内核
* 可互换：您可以动态地部署更新和升级
* 可移植：您可以在本地构建，部署到云中，并在任何地方运行
* 可伸缩：您可以增加并自动分发容器副本。

Docker有三个大的概念：_Images（镜像）_、_Containers（容器）_、_Registry（仓库）_
>**Docker 镜像**
Docker 镜像是Docker容器运行时的只读模板，每一个镜像由一系列的层 (layers) 组成。Docker 使用 UnionFS 来将这些层联合到单独的镜像中。UnionFS 允许独立文件系统中的文件和文件夹(称之为分支)被透明覆盖，形成一个单独连贯的文件系统。正因为有了这些层的存在，Docker 是如此的轻量。当你改变了一个 Docker 镜像，比如升级到某个程序到新的版本，一个新的层会被创建。因此，不用替换整个原先的镜像或者重新建立(在使用虚拟机的时候你可能会这么做)，只是一个新 的层被添加或升级了。现在你不用重新发布整个镜像，只需要升级，层使得分发 Docker 镜像变得简单和快速。

>**Docker 仓库**
Docker 仓库用来保存镜像，可以理解为代码控制中的代码仓库。同样的，Docker 仓库也有公有和私有的概念。公有的 Docker 仓库名字是 Docker Hub。Docker Hub 提供了庞大的镜像集合供使用。这些镜像可以是自己创建，或者在别人的镜像基础上创建。Docker 仓库是 Docker 的分发部分。
 
>**Docker 容器**
Docker 容器和文件夹很类似，一个Docker容器包含了所有的某个应用运行所需要的环境。每一个 Docker 容器都是从 Docker 镜像创建的。Docker 容器可以运行、开始、停止、移动和删除。每一个 Docker 容器都是独立和安全的应用平台，Docker 容器是 Docker 的运行部分。

**镜像和容器：**
一个容器是通过运行一个图像来启动的。图像是一个可执行的包，它包含运行应用程序所需的一切——代码、运行时、库、环境变量和配置文件。

**容器和虚拟机：**
一个容器在Linux上运行，并与其他容器共享主机的内核。它运行一个离散的过程，没有比任何其他可执行文件更少的内存，使它变得轻量级。
相比之下，虚拟机（VM）运行一个成熟的“客户”操作系统，通过虚拟机监控程序虚拟访问主机资源。一般来说，VMs提供的环境比大多数应用程序需要的资源都多。
<table>
  <tr>
    <td><center><img src="/uploads/2018/09/container_vm_01.png"></center></td>
    <td><center><img src="/uploads/2018/09/container_vm_02.png"></center></td>
  </tr>
</table>


#### 三、安装 ####
##### 1、Windows版 #####
win7、win8 等需要利用 docker toolbox 来安装，国内可以使用阿里云的镜像来下载，下载地址：[传送门](http://mirrors.aliyun.com/docker-toolbox/windows/docker-toolbox/)
win10 直接下载官方的安装包：[传送门](http://www.docker-cn.com/community-edition#/overview)
>docker toolbox 是一个工具集，它主要包含以下一些内容：
```
Docker CLI 客户端，用来运行docker引擎创建镜像和容器
Docker Machine. 可以让你在windows的命令行中运行docker引擎命令
Docker Compose. 用来运行docker-compose命令
Kitematic. 这是Docker的GUI版本
Docker QuickStart shell. 这是一个已经配置好Docker的命令行环境
Oracle VM Virtualbox. 虚拟机```

下载完成之后直接点击安装，安装成功后，桌边会出现三个图标，入下图所示：
![](/uploads/2018/09/docker-toolbox-01.png)


##### 2、Linux版 #####
1. 移除旧的版本
```
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

2. 安装一些必要的系统工具
`sudo yum install -y yum-utils device-mapper-persistent-data lvm2`

3. 添加软件源信息
`sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo`
>如果报错：
使用yum命令报错File "/bin/yum-config-manager", line 133 except yum.Errors.RepoError, e: SyntaxError: invalid syntax问题
>`vim /bin/yum-config-manager`打开，可看见首行为 /usr/bin/python ，由报错可看出使用的是Python2的语法，而我单独安装了Python3，且设置为默认版本导致的语法错误。将/usr/bin/python2 更改为使用版本2的即可。

4. 更新 yum 缓存
`sudo yum makecache fast`

5. 安装Docker-ce
`sudo yum -y install docker-ce`

6. 启动Docker后台服务
`sudo systemctl start docker`

7. 测试运行 hello-world
`docker run hello-world`

**镜像加速**
鉴于国内网络问题，后续拉取 Docker 镜像十分缓慢，我们可以需要配置加速器来解决。
比较常用的是网易的镜像中心和daocloud镜像市场。 
* 网易镜像中心：https://c.163.com/hub#/m/home/ 
* daocloud镜像市场：https://hub.daocloud.io/

我使用的是网易的镜像地址：http://hub-mirror.c.163.com。请在该配置文件中加入（没有该文件的话，请先建一个。配置文件默认是在 /etc/default/docker）：
```
{
  "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
```
所以刚开始我在寻找/etc/default/docker这个配置文件，一直找不到，后来发现是因为系统和版本的差异。 
在centos7上这个配置文件已经被更改为 /etc/docker/daemon.json 。
可以在这个配置中添加相应的registry-mirrors路径 。
原来是这样：
```
[root@localhost docker]# cat daemon.json 
{
    "live-restore": true
}
```
添加后：
```
{
    "registry-mirrors": ["http://ef017c13.m.daocloud.io"],
    "live-restore": true
}
```
可以手动vim添加，也可以使用daocloud给出的命令直接更改（建议使用命令）
`[root@localhost docker]# curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://ef017c13.m.daocloud.io`
更改后重启docker
`systemctl restart docker`


**删除Docker CE**
```
yum remove docker-ce
rm -rf /var/lib/docker
```

