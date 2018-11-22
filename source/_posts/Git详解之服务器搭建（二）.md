---
title: Git 详解之服务器搭建（二）
date: 2018-03-14 17:46:31
categories:
- 版本控制
- Git
tags:
- Git
---
服务器环境：CentOS7 + Git(version 1.8.3.1)

#### 1、安装Git
　　`yum install -y git`
安装完后，查看 Git 版本
```
[root@VM_0_7_centos ~]# git version
git version 1.8.3.1
```
<!--more-->

#### 2、新建Git账号，用来管理Git服务
```
[root@VM_0_7_centos ~]# id git
id: git：无此用户
[root@VM_0_7_centos ~]# useradd git
[root@VM_0_7_centos ~]# passwd git
```

#### 3、服务器端创建Git仓库
设置 /data/git/gittest.git 为 Git 仓库
然后把 Git 仓库的 owner 修改为 git
```
[root@VM_0_7_centos ~]# mkdir -p /data/git/gittest.git
[root@VM_0_7_centos ~]# git init --bare /data/git/gittest.git
Initialized empty Git repository in /data/git/gittest.git/
[root@VM_0_7_centos ~]# cd /data/git/
[root@VM_0_7_centos git]# chown -R git:git gittest.git/
```

#### 4、客户端首次 clone 远程仓库
进入 Git Bash 命令行客户端，创建项目地址（设置在  /d/MyProjects/gittest.git）并进入；
然后从 Linux Git 服务器上 clone 项目：
`$ git clone git@118.24.8.229:/data/git/gittest.git`

如果SSH用的不是默认的22端口，则需要使用以下的命令（假设SSH端口号是7700）：
`$ git clone ssh://git@118.24.8.229:7700/home/data/gittest.git`

当第一次连接到目标 Git 服务器时会得到一个提示：
```
Cloning into 'gittest'...
The authenticity of host '118.24.8.229 (118.24.8.229)' can't be established.
ECDSA key fingerprint is SHA256:witKSLOMzfgWBcs78t5LfqmSJ+JEm2/PBEaR0d9mnqA.
Are you sure you want to continue connecting (yes/no)? yes
```
选择 yes：
```
Warning: Permanently added '118.24.8.229' (ECDSA) to the list of known hosts.
git@118.24.8.229's password:
warning: You appear to have cloned an empty repository.
```
后面提示要输入密码，可以采用 SSH 公钥来进行验证（下面会进行说明）。


#### 5、服务器端Git打开RSA认证
进入 /etc/ssh 目录，编辑 sshd\_config，打开以下三个配置的注释：
```
#RSAAuthentication yes
#PubkeyAuthentication yes
#AuthorizedKeysFile     .ssh/authorized_keys
```
>##RSAAuthentication用来设置是否开启RSA密钥验证，只针对SSH1<font color="red">（注：对于SSH2来说，没有此项配置）</font>
##PubkeyAuthentication用来设置是否开启公钥验证，如果使用公钥验证的方式登录时，则设置为yes
##AuthorizedKeysFile用来设置公钥验证文件的路径，与PubkeyAuthentication配合使用,默认值是".ssh/authorized_keys"。

保存并重启 sshd 服务：
`[root@VM_0_7_centos git]# systemctl restart sshd`

由AuthorizedKeysFile 得知公钥的存放路径是 `.ssh/authorized_keys`，实际上是 `$Home/.ssh/authorized_keys`，由于管理 Git 服务的用户是 git，所以实际存放公钥的路径是 `/home/git/.ssh/authorized_keys`。


#### 6、客户端创建SSH公钥和私钥并将公钥导入服务器端
`$ ssh-keygen -t rsa`
此时 C:\Users\用户名\.ssh 下会多出两个文件 id\_rsa 和 id\_rsa.pub

>id\_rsa 是私钥
id\_rsa.pub 是公钥

回到 Git Bash下，导入文件：（注：需要输入服务器端 git 用户的密码）
```
$ ssh git@118.24.8.229
git@118.24.8.229's password:
Last login: Sun Mar 18 09:43:25 2018 from 113.46.182.66
[git@VM_0_7_centos ~]# ll -h .ssh/authorized_keys
-rw-rw-r-- 1 git git 0 Mar 18 09:37 .ssh/authorized_keys
[git@VM_0_7_centos ~]$ cat >> ~/.ssh/authorized_keys < ~/.ssh/id_rsa.pub
[git@VM_0_7_centos ~]# ll -h .ssh/authorized_keys
-rw-rw-r-- 1 git git 399 Mar 18 09:44 .ssh/authorized_keys
[git@VM_0_7_centos ~]$ exit
logout
Connection to 118.24.8.229 closed.
```
由上面可知道 公钥已经被导入到 git用户的认证文件中。
<font color="red">
>重要（对于服务器端）：
修改 .ssh 目录的权限为 700
修改 .ssh/authorized_keys 文件的权限为 600</font>


#### 7、客户端再次 clone 远程仓库
```
git clone git@118.24.8.229:/data/git/gittest.git
fatal: destination path 'gittest' already exists and is not an empty directory.
```
项目已经 clone 了。（注：已经不需要输入密码了）


#### 8、禁止 Git 用户 SSH 登录服务器
编辑 /etc/passwd
找到：
`git:x:502:504::/home/git:/bin/bash`
修改为
`git:x:502:504::/home/git:/bin/git-shell`
此时 git 用户可以正常通过 ssh 使用 git，但无法通过 ssh 登录系统：
```
$ ssh git@118.24.8.229
git@118.24.8.229's password:
Last login: Sun Mar 18 09:44:11 2018 from 113.46.182.66
fatal: Interactive git shell is not enabled.
hint: ~/git-shell-commands should exist and have read and execute access.
Connection to 118.24.8.229 closed.
```

***

#### 问题集锦
##### （1）Git本地公钥导入服务器失败
>本地 Git Bash 登陆服务器报错：
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that the RSA host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
d0:00:7c:bc:88:5c:dc:de:89:61:44:30:00:60:f9:b2.
Please contact your system administrator.
Add correct host key in /root/.ssh/known_hosts to get rid of this message.
Offending key in /root/.ssh/known_hosts:1
RSA host key for 192.168.4.222 has changed and you have requested strict checking.
Host key verification failed.

**解决方法：**
先查一下本地家目录的 .ssh/know_hosts 文件中是否存在服务器端的信息
```
$ cat ~/.ssh/known_hosts
118.24.8.229 ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBHrKfe2igdpfAMhukTCC0ki3gZEO4bFQviqsO6T+F/8Uqjt+XoVzKs7zmNgMEkodfpkZJ93bZyRMc2JdDRlCp9w=
```
由此可知，本地已经保存了服务器端的信息，因为我是重装了Git 服务器，所以此时保存的信息已经过时了，才会报次错误。
解决方法就是删除已存在信息：
由于我本地保存的只有一条服务器端信息，可使用如下命令：
`$ cat /dev/null > ~/.ssh/known_hosts`
如果有多条服务器信息，可使用 `$ vim ~/.ssh/known_hosts`手动删除，或者window下直接打开文件删除对应信息即可。
