---
title: 总结-MySQL主从复制、读写分离配置详解
date: 2017-12-08 11:23:55
categories:
- [MySQL, 架构]
tags:
- 架构
- MySQL
---
## MySQL主从同步的机制：
![](/uploads/2017/12/mysql_master_slave_01.jpg)
<!--more-->
> 1. Slave 上面的IO线程连接上 Master，并请求从指定日志文件的指定位置(或者从最开始的日志)之后的日志内容;
> 2. Master 接收到来自 Slave 的 IO 线程的请求后，通过负责复制的 IO线程根据请求信息读取指定日志指定位置之后的日志信息，返回给 Slave 端的 IO线程。返回信息中除了日志所包含的信息之外，还包括本次返回的信息在 Master 端的 Binary Log 文件的名称以及在 BinaryLog 中的位置;
> 3. Slave 的 IO 线程接收到信息后，将接收到的日志内容依次写入到 Slave 端的RelayLog文件(MySQL-relay-bin.xxxxxx)的最末端，并将读取到的Master端的bin-log的文件名和位置记录到master-info文件中，以便在下一次读取的时候能够清楚的高速Master“我需要从某个bin-log的哪个位置开始往后的日志内容，请发给我”
> 4. Slave 的 SQL 线程检测到 Relay Log 中新增加了内容后，会马上解析该 Log 文件中的内容成为在 Master端真实执行时候的那些可执行的 Query 语句，并在自身执行这些 Query。这样，实际上就是在 Master 端和 Slave端执行了同样的 Query，所以两端的数据是完全一样的。

## MySQL主从同步的作用
Ø  可以作为一种备份机制，相当于热备份
Ø  可以用来做读写分离，均衡数据库负载

## MySQL主从同步的步骤
1、在Master MySQL上创建一个用户‘repl’，并允许其他Slave服务器可以通过远程访问Master，通过该用户读取二进制日志，实现数据同步。
> mysql>create user repl; //创建新用户
> //repl用户必须具有REPLICATION SLAVE权限，除此之外没有必要添加不必要的权限，密码为mysql。说明一下192.168.0.%，这个配置是指明repl用户所在服务器，这里%是通配符，表示192.168.0.0-192.168.0.255的Server都可以以repl用户登陆主服务器。当然你也可以指定固定Ip。
> mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'192.168.0.%' IDENTIFIED BY 'mysql';

2、主数据库服务器配置
　进入主数据库服务器安装目录,打开my.ini,在文件末尾增加如下配置：
```
#数据库ID号， 为1时表示为Master,其中master_id必须为1到232–1之间的一个正整数值;   
server-id = 1  
#启用二进制日志；  
log-bin=mysql-bin  
#需要同步的二进制数据库名；  
binlog-do-db=minishop  
#不同步的二进制数据库名,如果不设置可以将其注释掉;  
binlog-ignore-db=information_schema  
binlog-ignore-db=mysql  
binlog-ignore-db=personalsite  
binlog-ignore-db=test  
#设定生成的log文件名；  
log-bin="D:/Database/materlog"  
#把更新的记录写到二进制文件中； （注：此处不限制，是对所有操作进行记录日志）
#log-slave-updates  
```
　保存文件。重启Mysql服务。
<font color="red">注：主数据库配置文件命令详解：</font>

log-bin=mysql-bin | 启动二进制日志系统
------------------|--------------------------
binlog-do-db=test | 二进制需要同步的数据库名
server-id = 1     | 本机数据库ID 标示为主，该部分还应有一个server-id=Master_id选项，其中master_id必须为1到232–1之间的一个正整数值
log-bin=/var/log/mysql/updatelog | 设定生成log文件名，这里的路径没有mysql目录要手动创建并给于它mysql用户的权限。
binlog-ignore-db=mysql | 避免同步mysql用户配置，以免不必要的麻烦

查看日志：
```
mysql> SHOW MASTER STATUS;
+-------------------+----------+--------------+------------------+
| File | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+-------------------+----------+--------------+------------------+
| master-bin.000001 | 0| | |
+-------------------+----------+--------------+------------------+
1 row in set (0.00 sec)
```
*（注：此处的日志文件File、Position下面需要用到）*
3、从数据库服务器配置
　进入从数据库服务器安装目录,打开my.ini,在文件末尾增加如下配置：
```
#如果需要增加Slave库则，此id往后顺延；  
server-id = 2    
log-bin=mysql-bin  
#如果发现主服务器断线，重新连接的时间差；  
master-connect-retry=60  
#不需要备份的数据库；   
replicate-ignore-db=mysql  
#需要备份的数据库  
replicate-do-db=minishop  
#log-slave-update  
```
　保存文件。重启Mysql服务。
4、连接Master
```
mysql>change master to 
->master_host='192.168.0.104', //Master 服务器Ip
->master_port=3306,
->master_user='repl',
->master_password='mysql', 
->master_log_file='master-bin.000001',//Master服务器产生的日志
->master_log_pos=0;
```
<font color="red">从服务器配置命令详解：</font>

server-id = 2   |  从服务器ID号，不要和主ID相同 ，如果设置多个从服务器，每个从服务器必须有一个唯一的server-id值，必须与主服务器的以及其它从服务器的不相同。可以认为server-id值类似于IP地址：这些ID值能唯一识别复制服务器群集中的每个服务器实例。
--------------------------|---------------------------------------------
master-host = 172.31.70.51 | 指定主服务器IP地址
master-user = replication | 指定在主服务器上可以进行同步的用户名
master-password = 123456 | 密码
master-port = 3306      | 同步所用的端口
master-connect-retry=60 | 断点重新连接时间
replicate-ignore-db=mysql | 屏蔽对mysql库的同步，以免有麻烦
replicate-do-db=test   | 同步数据库名称
5、启动Slave
`start slave;`
查看从服务器状态
`mysql> show slave status \G;`
![](/uploads/2017/12/mysql_master_slave_02.png)
OK所有配置都完成了。

## 读写分离
　完成MySQL的主从配置，实现数据的实时同步，采用架构的方式实现MySQL的读写分离。
　统一认证平台完成数据的增删改的操作，保存数据到MySQL的Master的数据库中，Salve数据库从Master数据库中实时同步数据，应用系统从Salve数据库中读取书据。