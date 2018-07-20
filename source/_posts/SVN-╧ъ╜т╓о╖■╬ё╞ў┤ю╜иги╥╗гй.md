---
title: SVN 详解之服务器搭建（一）
date: 2018-03-15 14:09:57
categories:
- 版本控制
- SVN
tags:
- SVN
---
#### 1、使用 yum 安装SVN
`[root@singledb ~]# yum install -y subversion`

#### 2、验证安装版本及是否成功安装
```
[root@singledb ~]# svnserve --version
svnserve, version 1.4.2 (r22196)
```
<!--more-->
#### 3、创建 SVN 版本库
```
[root@singledb ~]# mkdir /var/svn  //创建存放版本库文件的目录
[root@singledb ~]# svnadmin create /var/svn/testsvn  //创建版本库
```

#### 4、SVN 配置
创建版本库后，在这个目录下会生成3个配置文件：
```
[root@singledb conf]# pwd
/var/svn/testsvn/conf
[root@singledb conf]# ls
authz  passwd  svnserve.conf
```
>（1）svnserve.conf：  svn服务配置文件。
（2）passwd： 用户名口令文件，负责账号和密码的用户名单管理。
（3）authz： 权限配置文件，控制账号的读写权限。 

**svnserve.conf 文件， 该文件配置项分为以下5项：**
* anon-access： 控制非鉴权用户访问版本库的权限。
* auth-access：  控制鉴权用户访问版本库的权限。
* password-db： 指定用户名口令文件名。
* authz-db：指定权限配置文件名，通过该文件可以实现以路径为基础的访问控制。
* realm：指定版本库的认证域，即在登录时提示的认证域名称。若两个版本库的认证域相同，建议使用相同的用户名口令数据文件

<font color="red">（注意：将此五项都给去注释的时候，一定要顶格，不然会报错； realm指要认证的版本库，填写自己要配置的版本库即可）</font>

**authz 文件 ：**
```
#这里把不同用户放到不同的组里面，下面在设置目录访问权限的时候，用用户组来操作就可以了。
（注：此处可不使用groups概念，在下方可直接添加可操作的人员即可。）
[groups]
admin = admin
dev = ben,nana
test = test

# 为所有库指定默认访问规则
# 所有人可以读，管理员可以写
[/]
* = r
@admin = rw

# 允许开发人员可以完全访问他们的项目版本库
[devproject:/]
@dev = rw

# 允许测试人员访问其中的test目录
[devproject:/test]
@test = rw
```

**Passwd 文件 ：**
``` 
[users]
# harry = harryssecret
# sally = sallyssecret
ben = ben1234
nana = nana1234
```

#### 5、启动和停止SVN服务
启动SVN服务： 
`svnserve -d -r /home/svn   （/home/svn 为版本库的根目录）`
关闭SVN服务：
① 使用以下命令查找进程 
```
[root@VM_0_7_centos ~]# ps aux | grep svn
root      8530  0.0  0.0 112660   972 pts/0    R+   14:44   0:00 grep --color=auto svn
root     31964  0.0  0.0 166336   908 ?        Ss   11:28   0:00 svnserve -d -r /var/svn
```
② 使用Kill命令杀死进程 
`kill -s 9 31964		（31964为进程ID）`


#### 6、客户端检出访问 svn 服务器
`svn co svn://118.24.8.229/<repo>` 即可检出代码（注：repo为代码库名称）。会弹出输入用户名和密码。

**注：我们搭建的SVN是独立服务器形式运行的，没有和Apache整合，所以SVN地址为：svn://xxx/xxx，而不是http://xxx/xxx或https://xxx/xxx。**
Apache 搭建HTTP方式访问SVN服务器：[<font color="red">传送门</font>](http://blog.csdn.net/robertohuang/article/details/55504445)


***

#### 问题集锦
##### （1）SVN 检出错误
>在 Windows下检出SVN仓库报错：
>Unable to connect to a repository at URL 'svn://118.24.8.229/testsvn'
Can't connect to host '118.24.8.229': 由于连接方在一段时间后没有正确答复或连接的主机没有反应，连接尝试失败。

经查找知道，是因为服务器端的对应端口未开启：3690.
在服务端开启即可：
`iptables -A INPUT -p tcp -m tcp --dport 3690 -j ACCEPT`