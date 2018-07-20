---
title: Linux 源码包与RPM包详解
date: 2018-03-15 17:11:19
categories:
- Linux
tags:
- Linux
---
#### 源码包与RPM包的区别
##### 1、安装之前的区别：概念上的区别
比如说：源码包是开源的，比RPM包安装更自由，但是它安装更慢，更容易报错；RPM包是经过编译的，不能看到源代码，但是它安装更快，报错更容易解决，只有依赖性问题。

##### 2、安装之后的区别：安装位置不同
RPM包不需要指定安装位置，它会安装到系统默认位置；而源码包是人为手工设置的，下面我们就来看看到底位置有什么区别
<!--more-->

（1）RPM包安装位置
是按照在默认位置中
**RPM包默认安装路径**

安装位置        |   对应目录功能
----------------|------------------------
/etc/ 			|	配置文件安装目录
/usr/bin/ 		|	可执行的命令安装目录
/usr/lib/ 		|	程序所使用的函数库保存位置
/usr/share/doc/ |	基本的软件使用手册保存位置
/usr/share/man/ |	帮助文件保存位置

（2）源码包安装位置
安装在指定位置当中，一般是 `/usr/local/软件名/`

##### 3、安装位置不同带来的影响
RPM包安装的服务可以使用系统服务管理命令（service）来管理，例如RPM包安装的apache的启动方法是：
`/etc/rc.d/init.d/httpd start`
`service httpd start`

而源码包安装的服务则不能被服务管理命令管理，因为没有安装到默认路径中。所以只能用绝对路径进行服务的管理，如：
`/usr/local/apache2/bin/apachectl start`


#### RPM包详解
##### 1、rpm包命名结构
rpm包的组成：name-version-release.arch.rpm
>name：表示包的名称，包括主包名和分包名
version：表示包的版本信息
release：用于标识rpm包本身的发行号，可还包含适应的操作系统
arch:表示主机平台,noarch表示此包能安装到所以平台上面

例如：gd-devel-2.0.35-11.el6.x86_64.rpm
>gd是这个包的主包名，devel是这个包的分包名，
2.0.35是表示版本信息，2为主版本，0为此版本，35为修订号，
11.el6中的11是表示发行号，el6表示是RHEL6，
x86_64是表示包适合的平台，如果是noarch这表示与平台无关

##### 2、rpm包命令操作总结
<style type="text/css">
	table th:first-of-type {
		width: 15%;
	}
	table th:nth-of-type(2) {
		width: 17%;
	}
	table th:nth-of-type(3) {
		width: 50%;
	}
	table th:nth-of-type(4) {
		width: 40%;
	}
</style>

&nbsp;| option 					| 参数解释 							| Example
------|------------ ----------|-----------------------------|------------------
安装  | -i     					| 安装     							| &#32;
      |-v&#124;-vv&#124;-vvv 	| 显示详细信息 					| &#32;
      | -h    				 	| 以#显示安装进度一个#表示2%的进度  | rpm -ivh zsh-4.3.10-5.el6.x86_64.rpm
      | --nodeps 			   	| 忽略依赖关系 					| &#32;
      | --test  			   	| 测试安装 							| &#32;
      | --replacepkgs 			| 重新安装(安装的包已经安装了)| &#32;
升级  | -U   -Uvh  				| 升级+安装 						| &#32;
      |-F    -Fvh  				| 升级(此包已经安装了) 			| &#32;
      |--force     				| 有冲突强制升级  				| &#32;
      | --nodeps   				| 忽略包依赖性关系 				| &#32;
卸载  | -e        	 			| 卸载     							| rpm  -e  包名
      | --nodeps   				| 忽略包依赖性关系 				| &#32;
查询  | -q&#124;--query 		| &#32; 	    	               | rpm -q&#124;--query  包名
      | -qa         			| 查看所有已经安装的包 				| rpm -qa  查看所有包名 <br/>rpm -qa &#124; grep 包名查看某个包名
      | -qi         			| 查看包的摘要信息					| &#32;
      | -qf         			| 查看文件是有那个包安装的  		| rpm  -qf  /path/to/file
      | -ql         			| 查看包安装生成的文件清单			| &#32;
      | -qc         			| 查看包安装生成的配置文件			| &#32;
      | -qd         			| 查看包安装生成的帮助文档			| &#32;
      | -q  --scripts  		| 查看相关的脚本         			| rpm -q--script   包名
      | -qp[i&#124;l&#124;d&#124;c]   | 查看尚未安装包的详细信息 	| rpm -qpi /path/to/rpm_file
校验  | -V             		| &#32;		                     | rpm -V   包名
数据库管理 | --initdb  		| 新建 			                  | rpm  --initdb
           | --rebuilddb 	| 重建			                  | rpm  --rebuilddb


##### 3、RPM包安装方法之 yum 安装
yum(Yellowdog Update  Manager),yum是RPM的前端工具，是基于RPM的一个管理工具，他能自动的解决安装rpm包产生的依赖关系。

>yum 的配置文件    /etc/yum.conf
yum 的repository仓库的配置文件   /etc/yum.repos.d/*.repo

yum常用命令总结
<style type="text/css">
	table th:nth-of-type(2) {
		width: 40%;
	}
</style>

&#32;    |  操作命令 							 |  命令解释
---------|---------------------------------------|------------
列表     | `yum list <package_name>` 			 | 列出指定安装软件的清单
         | yum list installed      				 | 列出所有已安装的软件包
         | yum list extras     				     | 列出所有已安装但不在 Yum  仓库內的软件包
         | yum grouplist          		   	     | 列出所有的组
         | yum grouplist "Group1" 			     | 列出指定组的软件包列表
安装     | `yum -y install <package_name>`   	 | 安装指定的软件
         | yum -y groupinstall "Group1" "Group2" | 安装指定的组
         | `yum -y localinstall <package_name>`  | 用yum安装下载到本地的rpm包
卸载     | `yum -y remove <package_name>`     | 卸载指定的软件
更新     | yum check-update        			 	 | 列出所有可更新的软件清单
         | yum update              				 | 安装所有更新软件
         | `yum update <package_name>`			 | 更新指定的软件
信息     | yum info                			 	 | 显示所有包的信息
         | `yum info <package_name>` 			 | 显示指定包的信息
         | yum groupinfo "Group1" "Group2"  	 | 显示指定组的信息
清除     | yum clean all            			 | 清除所有yum所保存的信息
         | yum clean metadata       			 | 只清空保存的数据信息
其它操作 | yum repolist [all&#124;enable&#124;disable] 	 | 查看yum仓库的个数，默认显示启用的
         | yum   makecache          			 | 手动生成缓存
         | `yum search <package_name>`			 | 查询rpm包
         | `yum reinstall <package_name>` 		 | 重新安装一遍
         | `yum provides <package_name>`  		 | 列出软件包提供哪些文件


#### 源码包详解
##### 1、tar 源码包编译安装
编译安装的三部曲:
在安装三部曲之前，建议先看看解压之后目录里面的包含README, INSTALL文件，这里面的文件会告诉你详细安装步骤。
>（1）configure　　　检测编译环境
（2）make　　　　　进行编译
（3）make install　　编译安装