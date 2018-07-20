---
title: FTP 服务器配置流程详解
date: 2018-03-19 11:19:06
categories:
- 服务器
- FTP
tags:
- FTP
---
#### 1、修改 FTP服务器配置文件
① 安装完之后在/etc/vsftpd/路径下会存在三个配置文件。
>**vsftpd.conf：** 主配置文件
**ftpusers：** 指定哪些用户不能访问FTP服务器,这里的用户包括root在内的一些重要用户。
**user\_list：** 指定的用户是否可以访问ftp服务器，通过vsftpd.conf文件中的userlist\_deny的配置来决定配置中的用户是否可以访问，userlist\_enable=YES ，userlist\_deny=YES ，userlist\_file=/etc/vsftpd/user\_list 这三个配置允许文件中的用户访问FTP。（<font color="red">注：文件中用户名一行一个</font>）

<!--more-->
>>* userlist\_enable=YES, userlist\_deny=NO(或缺省), user\_list文件中的用户允许登陆，未出现在此文件中的用户不允许（默认）
* userlist\_enable=NO(或缺省), userlist\_deny=YES, user\_list文件中的用户不允许登陆，未出现再次文件中的用户允许
* userlist\_enable=YES, userlist\_deny=YES, user\_list文件中的用户不能访问FTP
（<font color="red">注：ftpusers文件中的用户为禁止登陆用户，此文件的权限大于user\_list文件的权限</font>）


② vsftpd.conf 默认配置如下：
```
[root@VM_0_7_centos ~]# cat /etc/vsftpd/vsftpd.conf.bak | grep '^[^#]'
anonymous_enable=YES
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=NO
listen_ipv6=YES
pam_service_name=vsftpd
userlist_enable=YES
tcp_wrappers=YES
```

③ 设置 chroot 来限制用户根目录（一般此处是为了限制FTP用户到网站根目录）
```
chroot_local_user=YES
chroot_list_file=/etc/vsftpd/chroot_list
allow_writeable_chroot=YES
```
>* 选项 `chroot_local_user=YES` 意味着本地用户将进入 chroot 环境，当登录以后默认情况下是其 home 目录。
* 选项 `chroot_list_file=/etc/vsftpd/chroot_list` 指定在此文件中的用户被限制在根目录中。（此文件缺省时，表示所有用户都被限制）
* 选项 `allow_writeable_chroot=YES` 可以允许 chroot 目录具有可写权限。（注：vsftpd默认是不允许chroot目录有可写权限）


#### 2、创建 FTP用户
① 增加 FTP组
`groupadd ftpgroup`

② 增加用户并设置其目录为 /data/www/ceshi
`useradd -g ftpgroup -d /data/www/ceshi -M ftp1`

③ 设置用户口令
`passwd ftp1`

④ 更改用户根目录所属组及权限
```
chown -R :ftpgroup /data/www/ceshi
chmod -R g+w /data/www/ceshi
```

⑤ 将用户添加进 chroot_list文件：（即设置用户根目录）
`vi /etc/vsftpd/chroot_list`
（注：一行一个用户名）

⑥ 重启vsftpd
`systemctl restart vsftpd.service`


#### 3、测试 FTP
可通过 FileZilla 连接FTP服务器
![](/uploads/2018/03/ftp_filezilla_setting.png 'FileZilla设置')

>① 点击‘文件’，打开站点管理器
② 如果有跳过此步，如果没有设置新站点
③ 协议选择“FTP文件传输协议”即可
④ 加密选择“普通FTP”即可
⑤ 传输设置 选择主动模式（即PORT模式）
⑥ 其它信息填写完整，点击连接

如果报错，请看另一篇《FTP 客户端报错集锦》。


#### 4、常用 FTP命令总结
##### （1）建立用户账号
>useradd
　-c（备注）	加上备注文字
　-d（登入目录）		指定用户登入时的起始目录
　-g（群组）	指定用户所属的群组
　-s 		指定用户登入后所使用的shell
例如：`useradd -d /alidata/www81/ceshi -g ftp -s /sbin/nologin ftp3`

##### （2）删除用户账号
>userdel
语法： `userdel [-r] [用户账号]`
补充说明：userdel 可删除用户账号与相关的文件。若不加参数，则仅删除用户账号，而不删除相关文件。 
例如：`userdel -r ftp3`

##### （3）修改用户账号
>usermod
语法： `usermod [-LU][-c <备注>][-d <登入目录>][-g <群组>][-s ]`
补充说明：可用来修改用户账号的各项设定
例如：`usermod -s /sbin/nologin tes`


#### 5、附本人配置的 vsftpd.conf
```
anonymous_enable=YES
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
chroot_local_user=YES
chroot_list_file=/etc/vsftpd/chroot_list
listen=NO
listen_ipv6=YES
pam_service_name=vsftpd
userlist_enable=YES
tcp_wrappers=YES
allow_writeable_chroot=YES
```