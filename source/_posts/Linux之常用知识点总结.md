---
title: Linux之常用知识点总结
date: 2018-10-12 12:24:05
categories:
- 书籍
- 《Linux就该这么学》
tags:
- Linux
- 书籍
---
#### 1、重置root管理员密码
`cat /etc/redhat-release`	查看系统信息
（1）重启 Linux 系统主机并出现引导界面时，按下键盘上的 e 键进入内核编辑界面
（2）在 linux16 参数这行的最后面追加“rd.break”参数，然后按下 Ctrl + X 组合键来运行修
改过的内核程序
（3）大约 30 秒过后，进入到系统的紧急求援模式
（4）依次输入以下命令，等待系统重启操作完毕，然后就可以使用新密码 linuxprobe 来登录
Linux 系统了。
```
mount -o remount,rw /sysroot
chroot /sysroot
passwd
touch /.autorelabel
exit
reboot
```
<!--more-->

#### 2、常用的RPM软件包命令

命令							|	作用
--------------------------------|----------------------
安装软件的命令格式				| rpm -ivh filename.rpm
升级软件的命令格式 				| rpm -Uvh filename.rpm
卸载软件的命令格式 				| rpm -e filename.rpm
查询软件描述信息的命令格式		| rpm -qpi filename.rpm
列出软件文件信息的命令格式		| rpm -qpl filename.rpm
查询文件属于哪个 RPM 的命令格式	| rpm -qf filename


#### 3、常见的Yum命令

命令 						|	作用
----------------------------|------------------------
yum repolist all			| 列出所有仓库
yum list all				| 列出仓库中所有软件包
yum info 软件包名称 查		| 看软件包信息
yum install 软件包名称		| 安装软件包
yum reinstall 软件包名称	| 重新安装软件包
yum update 软件包名称		| 升级软件包
yum remove 软件包名称		| 移除软件包
yum clean all				| 清除所有仓库缓存
yum check-update			| 检查可更新的软件包
yum grouplist			 	| 查看系统中已经安装的软件包组
yum groupinstall 软件包组	| 安装指定的软件包组
yum groupremove 软件包组	| 移除指定的软件包组
yum groupinfo 软件包组		| 查询指定的软件包组信息


#### 4、RHEL7 systemctl常用命令
**systemctl 管理服务的启动、重启、停止、重载、查看状态等常用命令**

System V init（RHEL 6）|		systemctl RHEL 7	  |	作用
---------------------- |------------------------------|---------------------
service foo start 	   |systemctl start foo.service   |	启动服务
service foo restart	   |systemctl restart foo.service |	重启服务
service foo stop 	   |systemctl stop foo.service 	  |	停止服务
service foo reload 	   |systemctl reload foo.service  |	重新加载配置文件（不终止服务）
service foo status 	   |systemctl status foo.service  |	查看服务状态

**systemctl 设置服务开机启动、不启动、查看各级别下服务启动状态等常用命令**

System V init（RHEL 6）|		systemctl RHEL 7	  		|	作用
---------------------- |------------------------------------|---------------------
chkconfig foo on 	   | systemctl enable foo.service 		| 开机自动启动
chkconfig foo off	   | systemctl disable foo.service		| 开机不自动启动
chkconfig foo 		   | systemctl is-enabled foo.service   | 查看特定服务是否为开机自动启动
chkconfig --list       | systemctl list-unit-files --type=service | 查看各个级别下服务的启动与禁用情况


#### 5、输入、输出重定向
**输入重定向中用到的符号及其作用**

符号					| 作用
------------------------|-------------------
命令 < 文件				| 将文件作为命令的标准输入
命令 << 分界符			| 从标准输入中读入，直到遇见分界符才停止
命令 < 文件 1 > 文件2	| 将文件 1 作为命令的标准输入并将标准输出到文件 2

**输出重定向中用到的符号及其作用**

符号					| 作用
------------------------|-------------------
命令 > 文件				| 将标准输出重定向到一个文件中（清空原有文件的数据）
命令 2> 文件			| 将错误输出重定向到一个文件中（清空原有文件的数据）
命令 >> 文件			| 将标准输出重定向到一个文件中（追加到原有内容的后面）
命令 2>> 文件			| 将错误输出重定向到一个文件中（追加到原有内容的后面）
命令 >> 文件 2>&1 或 命令 &>> 文件 | 将标准输出与错误输出共同写入到文件中（追加到原有内容的后面）


#### 6、常用的转义字符
**4 个最常用的转义字符如下所示。**
* 反斜杠（\）：使反斜杠后面的一个变量变为单纯的字符串。
* 单引号（''）：转义其中所有的变量为单纯的字符串。
* 双引号（""）：保留其中的变量属性，不进行转义处理。
* 反引号（\`\`）：把其中的命令执行后返回结果。


#### 7、Linux 系统中最重要的 10 个环境变量

变量名称			| 作用
--------------------|-------------------------
HOME				| 用户的主目录（即家目录）
SHELL				| 用户在使用的 Shell 解释器名称
HISTSIZE			| 输出的历史命令记录条数
HISTFILESIZE		| 保存的历史命令记录条数
MAIL				| 邮件保存路径
LANG				| 系统语言、语系名称
RANDOM				| 生成一个随机数字
PS1 				| Bash解释器的提示符
PATH				| 定义解释器搜索用户执行命令的路径
EDITOR				| 用户默认的文本编辑器


#### 8、Linux系统中常见的目录名称以及相应内容

目录名称	|	应放置文件的内容
----------------|--------------------------------------
/boot		| 开启所需文件——内核、开机菜单以及所需配置文件等
/dev		| 以文件形式存放任何设备与接口
/etc		| 配置文件
/home		| 用户家目录
/bin		| 存放单用户模式下还可以操作的命令
/lib		| 开机时用到的函数库，以及/bin与/sbin下面的命令要调用的函数
/sbin		| 开机过程中需要的命令
/media		| 用于挂载设备文件的目录
/opt		| 放置第三方的软件
/root		| 系统管理员的家目录
/srv		| 一些网络服务的数据文件目录
/tmp		| 任何人均可使用的“共享”临时目录
/proc		| 虚拟文件系统，例如系统内核、进程、外部设备及网络状态等
/usr/local	| 用户自行安装的软件
/usr/sbin	| Linux系统开机时不会使用到的软件/命令/脚本
/usr/share	| 帮助与说明文件，也可放置共享文件
/var		| 主要存放经常变化的文件，如日志
/lost+found	| 当文件系统发生错误时，将一些丢失的文件片段存放在这里

