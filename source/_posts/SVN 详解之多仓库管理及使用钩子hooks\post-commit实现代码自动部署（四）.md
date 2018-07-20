---
title: SVN 详解之多仓库管理及使用钩子hooks/post-commit实现代码自动部署（四）
date: 2018-03-16 10:42:33
categories:
- 版本控制
- SVN
tags:
- SVN
---
#### 多仓库管理
##### 1、首先，需要明白几个概念：
（1）多仓库管理，则每个仓库对应的配置都要单独管理。即该仓库允许那些人员访问，人员所具有的的权限等都需设定。

（2）每个仓库的配置文件只可管理本仓库的属性，不需要涉及到其他仓库，涉及到也没用。即仓库与仓库之间是独立的管理。
<!--more-->

##### 2、实战演示
首先，新建2个仓库，project和common。
```
svnadmin create /var/svn/project	//开放仓库
svnadmin create /var/svn/common		//共享资源仓库
```
分别对应仓库的conf/ 下文件进行配置，passwd与svnserve.conf配置类似，不予说明。其中，authz文件内容如下：
```
# project:
[groups]
admin = admin, ben
test = nana

[/] == [project:/]
@admin = rw
@test = r 	@只有读的权限，可用于测试

# common:
[common:/]
@test = rw
ben = rw
```
**注：对于用户享有单独某个目录权限，可如下操作：**
```
[svn:/special1] == [/special1]
ben = rw
[svn:/special1/special2]
nana = rw
```

##### 3、问题解决
>Invalid authz configuration
svn: Authorization failed解决办法

报此类错误，都是因为auth权限文件配置错误，仔细检查就可发现原因。


#### 使用钩子hooks/post-commit实现代码自动部署
配置了台svn服务器，用来保存公司项目的代码，同时svn服务器也是一台web服务器。因此希望当我本地代码commit到svn服务器时,能够触发svn服务器的钩子hooks/post-commit将新版本的代码自动update到站点目录上去。

>svn 目录：/var/svn/project
站点目录：/data/www/project/

##### 1、新建post-commit钩子
找到svn项目的hooks目录，这里是/var/svn/project/hooks。目录中默认会几个对应操作的钩子模板，我们需要创建一个post-commit的文件。
复制钩子文件，进行修改：
`cp hooks/post-commit.tmpl hooks/post-commit`
```
#!/bin/sh

export LANG=zh_CN.UTF-8
REPOS="$1"
REV="$2"

SVN_PATH=/usr/bin	#svn命令路径
WEB_PATH=/data/www/project/python_movie		#项目路径，即已经检出的项目，具体到项目目录
LOG_PATH=/data/www/project/python_movie/logs/svn_deploy.log		#日志文件
SVN_USER=ben
SVN_PASS=ben1234

echo `date "+%Y-%m-%d %H:%M:%S"` >> $LOG_PATH
echo `whoami`,$REPOS,%REV >> %LOG_PATH

$SVN_PATH/svn update $WEB_PATH --username $SVN_USER --password $SVN_PASS --no-auth-cache >> $LOG_PATH
```
<font color="red">说明：
whoami #执行此程序的用户
REPOS="$1" #svn项目绝对路径值
REV="$2" #最新版本号
--no-auth-cache #不保存账户认证信息</font>

##### 2、修改post-commit文件权限
```
chown www:www /var/svn/python_movie/hooks/post-commit #设置脚本所属用户组，www为web服务运行账户和组
chmod +x /var/svn/python_movie/hooks/post-commit #添加脚本执行权限
```

##### 3、客户端测试
测试的话，我这里是在本地修改版本库，点commit，然后再看web(nginx)服务器上的数据是否更新来测试。

##### 4、问题解决
Checkout一份代码到web服务器上
```
# cd /data/www/project/python_movie
# /usr/bin/svn checkout svn://127.0.0.1/python_movie
```
>在日志文件中提示
Skipped "/data/www/project/python_movie"
然后提交的文件并没有自动更新到web目录下

解决方法是：
首先，需要在Web目录下检出SVN项目，生成 .svn目录。原因是 Web目录下没有 .svn目录，更新时钩子不能识别Web目录下的 .svn（因为没有），因此会跳过Web目录。
```
cd /data/www/project/python_movie
svn checkout svn://服务器的ip地址/python_movie ./
```
然后再次提交的文件就可以自动更新到web目录下了。