---
title: Docker之Dockerfile详解
date: 2018-09-28 17:43:38
categories:
- Docker
tags:
- Docker
---
#### 一、Nginx 示例 ####
```
FROM  centos

MAINTAINER 2018-04-011 lipengcheng 777@qq.com

RUN  yum -y install gcc*  make pcre-devel zlib-devel

RUN  rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm && yum -y install nginx

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```
<!--more-->
#### 二、指令说明 ####

指令		| 说明
------------|----------------------
FROM 		| 指定所创建镜像的基础镜像
MAINTAINER  | 指定维护者信息
RUN			| 运行命令
CMD			| 指定启动容器时默认执行的命令
LABEL 		| 指定生成镜像的元数据标签信息
EXPOSE		| 声明镜像内服务所监听的端口
ENV			| 指定环境变量
ADD 		| 赋值指定的`<src>`路径下的内容到容器中的`<dest>`路径下，`<src>`可以为URL；如果为tar文件，会自动解压到`<dest>`路径下
COPY 		| 赋值本地主机的`<src>`路径下的内容到容器中的`<dest>`路径下；一般情况下推荐使用COPY而不是ADD
ENTRYPOINT  | 指定镜像的默认入口
VOLUME      | 创建数据挂载点
USER        | 指定运行容器时的用户名或UID
WORKDIR     | 配置工作目录
ARG         | 指定镜像内使用的参数（例如版本号信息等）
ONBUILD     | 配置当前所创建的镜像作为其他镜像的基础镜像时，所执行的创建操作的命令
STOPSIGNAL  | 容器退出的信号
HEALTHCHECK | 如何进行健康检查
SHELL       | 指定使用SHELL时的默认SHELL类型

##### 1. FROM
指定所创建的镜像的基础镜像，如果本地不存在，则默认会去Docker Hub下载指定镜像。
格式为：
`FROM <image>`，或`FROM <image>:<tag>`，或`FROM <image>@<digest>`
任何Dockerfile中的第一条指令必须为FROM指令。并且，如果在同一个Dockerfile文件中创建多个镜像，可以使用多个FROM指令(每个镜像一次)。

##### 2. MAINTAINER
指定维护者信息。
格式为：
`MAINTAINER <name>`
例如：
MAINTAINER image_creator@docker.com
该信息将会写入生成镜像的Author属性域中。

##### 3. RUN
运行指定命令。
格式为：
`RUN <command>` 或 `RUN ["executable", "param1", "param2"]`
<font color="red">注意：后一个指令会被解析为json数组，所以必须使用双引号。</font>
前者默认将在shell终端中运行命令，即/bin/sh -c；后者则使用exec执行，不会启动shell环境。
指定使用其他终端类型可以通过第二种方式实现，例如：
`RUN ["/bin/bash", "-c", "echo hello"]`
每条RUN指令将在当前镜像的基础上执行指定命令，并提交为新的镜像。当命令较长时可以使用\换行。例如：
```
RUN apt-get update \
        && apt-get install -y libsnappy-dev zliblg-dev libbz2-dev \
        && rm -rf /var/cache/apt
```

##### 4. CMD
CMD指令用来指定启动容器时默认执行的命令。它支持三种格式：
1. CMD ["executable", "param1", "param2"] 使用exec执行，是推荐使用的方式；
2. CMD param1 param2  在/bin/sh中执行，提供给需要交互的应用；
3. CMD ["param1", "param2"]  提供给ENTRYPOINT的默认参数。

每个Dockerfile只能有一条CMD命令。如果指定了多条命令，只有最后一条会被执行。如果用户启动容器时指定了运行的命令(作为run的参数)，则会覆盖掉CMD指定的命令。

##### 5. LABEL
LABEL指令用来生成用于生成镜像的元数据的标签信息。
格式为：`LABEL <key>=<value> <key>=<value> <key>=<value> ...`
例如：
```
LABEL version="1.0"
LABEL description="This text illustrates \ that label-values can span multiple lines."
```

##### 6. EXPOSE
声明镜像内服务所监听的端口。
格式为：`EXPOSE <port> [<port>...]`
例如：
`EXPOSE 22 80 443 3306`
<font color="red">
注意：
该命令只是起到声明租用，并不会自动完成端口映射。
在容器启动时需要使用-P(大写P)，Docker主机会自动分配一个宿主机未被使用的临时端口转发到指定的端口；使用-p(小写p)，则可以具体指定哪个宿主机的本地端口映射过来。</font>

##### 7. ENV
指定环境变量，在镜像生成过程中会被后续RUN指令使用，在镜像启动的容器中也会存在。
格式为：`ENV <key><value>或ENV<key>=<value>...`
例如：
```
ENV GOLANG_VERSION 1.6.3
ENV GOLANG_DOWNLOAD_RUL https://golang.org/dl/go$GOLANG_VERSION.linux-amd64.tar.gz
ENV GOLANG_DOWNLOAD_SHA256 cdd5e08530c0579255d6153b08fdb3b8e47caabbe717bc7bcd7561275a87aeb

RUN curl -fssL "$GOLANG_DOWNLOAD_RUL" -o golang.tar.gz && echo "$GOLANG_DOWNLOAD_SHA256 golang.tar.gz" | sha256sum -c - && tar -C /usr/local -xzf golang.tar.gz && rm golang.tar.gz

ENV GOPATH $GOPATH/bin:/usr/local/go/bin:$PATH

RUN mkdir -p "$GOPATH/bin" && chmod -R 777 "$GOPATH"
```
指令指定的环境变量在运行时可以被覆盖掉，如`docker run --env <key>=<value> built_image`。

##### 8. ADD
该指令将复制指定的<src>路径下的内容到容器中的<dest>路径下。
格式为：`ADD<src> <dest>`
其中`<src>`可以使Dockerfile所在目录的一个相对路径(文件或目录)，也可以是一个URL，还可以是一个tar文件(如果是tar文件，会自动解压到`<dest>`路径下)。`<dest>`可以使镜像内的绝对路径，或者相当于工作目录(WORKDIR)的相对路径。路径支持正则表达式，例如：
`ADD *.c /code/`

##### 9. COPY
复制本地主机的`<src>`(为Dockerfile所在目录的一个相对路径、文件或目录)下的内容到镜像中的`<dest>`下。目标路径不存在时，会自动创建。路径同样支持正则。
格式为：`COPY <src> <dest>`
当使用本地目录为源目录时，推荐使用COPY。

##### 10. ENTRYPOINT
指定镜像的默认入口命令，该入口命令会在启动容器时作为根命令执行，所有传入值作为该命令的参数。
支持两种格式：
1. `ENTRYPOINT ["executable","param1","param2"]` (exec调用执行)；
2. `ENTRYPOINT command param1 param2`(shell中执行)。

此时，CMD指令指定值将作为根命令的参数。
每个Dockerfile中只能有一个ENTRYPOINT，当指定多个时，只有最后一个有效。
在运行时可以被--entrypoint参数覆盖掉，如docker run --entrypoint。

##### 11. VOLUME
创建一个数据卷挂载点。
格式为：`VOLUME ["/data"]`
可以从本地主机或者其他容器挂载数据卷，一般用来存放数据库和需要保存的数据等。

##### 12. USER
指定运行容器时的用户名或UID，后续的RUN等指令也会使用特定的用户身份。
格式为：`USER daemon`
当服务不需要管理员权限时，可以通过该指令指定运行用户，并且可以在之前创建所需要的用户。例如：
`RUN groupadd -r nginx && useradd -r -g nginx nginx`
要临时获取管理员权限可以用gosu或者sudo。

##### 13. WORKDIR
为后续的RUN、CMD和ENTRYPOINT指令配置工作目录。
格式为：`WORKDIR /path/to/workdir`
可以使用多个WORKDIR指令，后续命令如果参数是相对的，则会基于之前命令指定的路径。例如：
```
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```
则最终路径为/a/b/c

##### 14. ARG
指定一些镜像内使用的参数(例如版本号信息等)，这些参数在执行`docker build`命令时才以`--build-arg<varname>=<value>`格式传入。
格式为：`ARG<name>[=<default value>]`
则可以用`docker build --build-arg<name>=<value>`来指定参数值。

##### 15. ONBUILD
配置当所创建的镜像作为其他镜像的基础镜像的时候，所执行创建操作指令。
格式为：`ONBUILD [INSTRUCTION]`
例如Dockerfile使用如下的内容创建了镜像image-A：
```
[...]
ONBUILD ADD . /app/src
ONBUILD RUN /usr/local/bin/python-build --dir /app/src
[...]
```
如果基于image-A镜像创建新的镜像时，新的Dockerfile中使用FROM image-A指定基础镜像，会自动执行ONBUILD指令的内容，等价于在后面添加了两条指令：
```
FROM image-A

# Automatically run the following
ONBUILD ADD . /app/src
ONBUILD RUN /usr/local/bin/python-build --dir /app/src
```
使用ONBUILD指令的镜像，推荐在标签中注明，例如：ruby:1.9-onbuild。


#### 三、后记 ####
从需求出发，定制适合自己需求、高效方便的镜像，可以参考他人优秀的Dockerfile文件，在构建中慢慢优化Dockerfile文件：
1. 精简镜像用途：                 尽量让每个镜像的用途都比较集中、单一，避免构造大而复杂、多功能的镜像；
2. 选用合适的基础镜像：            过大的基础镜像会造成构建出臃肿的镜像，一般推荐比较小巧的镜像作为基础镜像；
3. 提供详细的注释和维护者信息：     Dockerfile也是一种代码，需要考虑方便后续扩展和他人使用；
4. 正确使用版本号：               使用明确的具体数字信息的版本号信息，而非latest，可以避免无法确认具体版本号，统一环境；
5. 减少镜像层数：                减少镜像层数建议尽量合并RUN指令，可以将多条RUN指令的内容通过&&连接；
6. 及时删除临时和缓存文件：        这样可以避免构造的镜像过于臃肿，并且这些缓存文件并没有实际用途；
7. 提高生产速度：                合理使用缓存、减少目录下的使用文件，使用.dockeringore文件等；
8. 调整合理的指令顺序：           在开启缓存的情况下，内容不变的指令尽量放在前面，这样可以提高指令的复用性；
9. 减少外部源的干扰：             如果确实要从外部引入数据，需要制定持久的地址，并带有版本信息，让他人可以重复使用而不出错。
