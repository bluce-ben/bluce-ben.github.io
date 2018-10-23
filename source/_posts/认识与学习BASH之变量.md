---
title: 认识与学习BASH之变量
date: 2017-12-28 19:10:43
categories:
- 书籍
- 《鸟哥的Linux私房菜基础篇（第三版）》
tags:
- 书籍
- BASH
- Shell
---
### 1、Shell 的变量
变量是 bash 环境中非常重要的一个知识点，我们知道 Linux 是多人多任务的环境，每个人登入系统都能取得一个 bash ，每个人都能够使用 bash 下达 mail 这个指令来收取【自己】的邮件，问题是，bash 是如何得知你的邮件信箱是哪个档案？这就需要【变量】的帮助。底下我们将介绍重要的环境变量、变量的取用与设定等数据。
<!--more-->

#### 命令的执行原理：
在用户执行了一条命令之后，Linux系统中到底发生了什么事情呢？简单来说，命令在Linux中的执行分为4个步骤。
1. **第1步：**判断用户是否以绝对路径或相对路径的方式输入命令（如/bin/ls），如果是的话则直接执行。
2. **第2步：**Linux系统检查用户输入的命令是否为“别名命令”，即用一个自定义的命令名称来替换原本的命令名称。可以用alias命令来创建一个属于自己的命令别名，格式为“alias 别名=命令”。若要取消一个命令别名，则是用unalias命令，格式为“unalias 别名”。我们之前在使用rm命令删除文件时，Linux系统都会要求我们再确认是否执行删除操作，其实这就是Linux系统为了防止用户误删除文件而特意设置的rm别名命令，接下来我们把它取消掉：
```
[root@linuxprobe ~]# ls
anaconda-ks.cfg Documents initial-setup-ks.cfg Pictures Templates
Desktop Downloads Music Public Videos
[root@linuxprobe ~]# rm anaconda-ks.cfg 
rm: remove regular file ‘anaconda-ks.cfg’? y
[root@linuxprobe~]# alias rm
alias rm='rm -i'
[root@linuxprobe ~]# unalias rm
[root@linuxprobe ~]# rm initial-setup-ks.cfg 
[root@linuxprobe ~]#
```
3. **第3步：**Bash解释器判断用户输入的是内部命令还是外部命令。内部命令是解释器内部的指令，会被直接执行；而用户在绝大部分时间输入的是外部命令，这些命令交由步骤4继续处理。可以使用“type命令名称”来判断用户输入的命令是内部命令还是外部命令。
4. **第4步：**系统在多个路径中查找用户输入的命令文件，而定义这些路径的变量叫作PATH，可以简单地把它理解成是“解释器的小助手”，作用是告诉Bash解释器待执行的命令可能存放的位置，然后Bash解释器就会乖乖地在这些位置中逐个查找。PATH是由多个路径值组成的变量，每个路径值之间用冒号间隔，对这些路径的增加和删除操作将影响到Bash解释器对Linux命令的查找。
```
[root@linuxprobe ~]# echo $PATH
/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin
[root@linuxprobe ~]# PATH=$PATH:/root/bin
[root@linuxprobe ~]# echo $PATH
/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin:/root/bin
```
这里有比较经典的问题：“为什么不能将当前目录（.）添加到PATH中呢? ” 原因是，尽管可以将当前目录（.）添加到PATH变量中，从而在某些情况下可以让用户免去输入命令所在路径的麻烦。但是，如果黑客在比较常用的公共目录/tmp中存放了一个与ls或cd命令同名的木马文件，而用户又恰巧在公共目录中执行了这些命令，那么就极有可能中招了。

### 2、什么是变量
简单的说，就是让某一个特定字符串代表不固定的内容。
**（1）环境变量**
当我们登录到Linux之后，就会有一个bash的执行程序用来跟Linux沟通，而在进入 shell 之前，由于系统需要一些变量来提供其他数据的存取 (或者是一些环境的设定参数值，例如是否要显示彩色等等)，所以就有一些所谓的【环境变量】需要来读入系统中。这些环境变量例如 PATH、HOME、MAIL、SHELL 等等，都是很重要的，为了区别与自定义变量的不同，环境变量通常以大写字符来表示。

**（2）变量的取用与设定：echo, 取消变量设定规则：unset**
可以利用 echo 这个指令来取用变量，但是，变量在被取用时，前面必须要加上钱字号【$】才行，举例来说，要知道 PATH 的内容，该如何是好？
1. 变量的取用：
> [root@www ~]# echo $PATH
/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin
变量的取用就如同上面的范例，利用 ehco 就能够读出，只是需要在变量名称前面加上 $，或者是以${变量}的方式来取用都可以。

2. 变量的设定规则：
>1.变量与变量内容以一个等号【=】来连结，如下所示：
　　【myname=ben】
>2.等号两边不能直接接空格符，如下所示为错误：
　　【myname = ben】或【myname=ben Tsai】
>3.变量名称只能是英文字母与数字，但是开头字符不能是数字，如下为错误：
　　【2myname=ben】
>4.变量内容若有空格符可使用双引号【"】或单引号【'】将变量内容结合起来，但双引号能够识别变量，单引号原样输出。
>5.可用跳脱字符【\】将特殊符号(如 [Enter], $, \, 空格符, '等)变成一般字符；
>6.在一串指令中，还需要藉由其他的指令提供的信息，可以使用反单引号【`指令`】或【$(指令)】。
　　【version=$(uname -r)】再【echo $version】可得【2.6.32_1-16-0-0_virtio】
>7.若该变量为扩增变量内容时，则可用 "$变量名称" 或 ${变量} 累加内容，如下所示：
　　【PATH="$PATH":/home/bin】
>8.若该变量需要在其他子程序执行，则需要以 export 来使变量变成环境发量：
　　【export PATH】
>9.通常大写字符为系统默认变量，自行设定变量可以使用小写字符，方便判断；
>10.取消变量的方法为使用 unset ：【unset 变量名称】例如取消 version 的设定：
　　【unset version】

**（3）其它**
1. 什么是【子程序】呢？就是说，在我目前这个 shell 的情况下，去启用另一个新的 shell ，新的那个shell 就是子程序。在一般的状态下，父程序的自定义变量是无法在子程序内使用的。但是透过export 将变量变成环境发量后，就能够在子程序底下应用了。
2. 在指令下达的过程中，反单引号( \` )这个符号代表的意义为何？
在一串指令中，在 \` 之内的指令将会被先执行，而其执行出来的结果将做为外部的输入信息。

**（4）常见引用方式**
![](/uploads/2018/10/shell_variable_01.png)
![](/uploads/2018/10/shell_variable_02.png)
![](/uploads/2018/10/shell_variable_03.png)

### 3、环境变量的功能
用 env 观察环境变量与常见环境变量说明。env 是 environment (环境) 的简写，是列出来所有的环境变量的命令（此处就不贴出 env 命令显示数据）。
底下我们对一些常见变量来做一个说明：

>HOME
	代表用户的家目录。还记得我们可以使用 cd ~ 去到自己的家目录吗？或者利用 cd 就可以直接回到用户家目录了。那就是取用这个变量。有很多程序都可能会取用到这个变量的值。

>SHELL
	告知我们，目前这个环境使用的 SHELL 是哪支程序？ Linux 预设使用 /bin/bash 。

>HISTSIZE
	这个与【历史命令】有关，亦即是，我们曾经下达过的指令可以被系统记录下来，而记录的【笔数】则是由这个值来设定的。

>MAIL
	当我们使用 mail 这个指令在收信时，系统会去读取的邮件信箱档案 (mailbox)。

>PATH
	就是执行文件搜寻的路径，目录与目录中间以冒号(:)分隔，由于档案的搜寻是依序由 PATH 的变量内的目录来查询，所以目录的顺序也是重要的。

>LANG
	这个重要。就是语系数据。很多讯息都会用到他，举例来说，当我们在启动某些 perl 的程序语言档案时，他会主动的去分析语系数据文件，如果发现有他无法解析的编码语系，可能会产生错误。一般来说，我们中文编码通常是 zh_TW.Big5 或者是 zh_TW.UTF-8，这两个编码偏偏不容易被解译出来，所以，有的时候，可能需要修订一下语系数据。

>RANDOM
	这个就是【随机随机数】的变量。目前大多数的 distributions 都会有随机数生成器，那就是 /dev/random 这个档案。 我们可以透过这个随机数档案相关的变量 ($RANDOM) 来随机取得随机数值。在 BASH 的环境下，这个 RANDOM 变量的内容，介于 0~32767 之间，所以，你只要 echo $RANDOM 时，系统就会主动的随机取出一个介于 0~32767 的数值。万一我想要使用 0~9 之间的数值，利用 declare 宣告数值类型，然后这样做就可以了：
		`declare -i number=$RANDOM*10/32768 ; echo $number ##此时会随机取出 0~9 之间的数值`

Linux作为一个多用户多任务的操作系统，能够为每个用户提供独立的、合适的工作运行环境，因此，一个相同的变量会因为用户身份的不同而具有不同的值。

其实变量是由固定的变量名与用户或系统设置的变量值两部分组成的，我们完全可以自行创建变量，来满足工作需求。例如设置一个名称为WORKDIR的变量，方便用户更轻松地进入一个层次较深的目录：
```
[root@linuxprobe ~]# mkdir /home/workdir
[root@linuxprobe ~]# WORKDIR=/home/workdir
[root@linuxprobe ~]# cd $WORKDIR 
[root@linuxprobe workdir]# pwd
/home/workdir
```
但是，这样的变量不具有全局性，作用范围也有限，默认情况下不能被其他用户使用。如果工作需要，可以使用export命令将其提升为全局变量，这样其他用户也就可以使用它了：
```
[root@linuxprobe ~]# export WORKDIR
[root@linuxprobe ~]# su linuxprobe
Last login: Fri Mar 20 21:52:10 CST 2017 on pts/0
[linuxprobe@linuxprobe ~]$ cd $WORKDIR
[linuxprobe@linuxprobe workdir]$ pwd
/home/workdir
```


### 4、变量键盘的读取、数组与宣告
（1）要读取来自键盘输入的变量，就是用 read 这个指令。这个指令最常被用在 shell script 的撰写当中，想要跟使用者对谈？用这个指令就对了。

	语法：read [-pt] variable
	选项与参数：
		-p ：后面可以接提示字符；
		-t ：后面可以接等待的【秒数！】这个比较有趣。不会一直等待使用者
read 之后不加任何参数，直接加上变量名称，那么底下就会主动出现一个空白行等待你的输入。如果加上 -t 后面接秒数，那么 t 秒之内没有任何动作时，该指令就会自动略过。如果是加上 -p ，在输入的光标前就会有比较多可以用的提示字符给我们参考。在指令的下达里面，比较美观。
```
示例：
[root@www ~]# echo $atest
This is a test <==你刚刚输入的数据已经变成一个变量内容
[root@www ~]# read -p "Please keyin your name: " -t 30 named
Please keyin your name: VBird Tsai <==提示使用者 30 秒内输入自己的大名，将该输入字符串作为名为 named的变量内容
```
（2）declare 或 typeset 是一样的功能，就是在【宣告变量的类型】。如果使用 declare 后面并没有接任何参数，那么 bash 就会主动的将所有的变量名称与内容通通叫出来，就好像使用 set 一样。
declare / typeset

	语法：declare [-aixr] variable
	选项与参数：
		-a ：将后面名为 variable 的变量定义成为数组 (array) 类型
		-i ：将后面名为 variable 的变量定义成为整数数字 (integer) 类型
		-x ：用法与 export 一样，就是将后面的 variable 变成环境变量；取消的话将[-x]变为[+x]即为取消。
		-r ：将变量设定成为 readonly 类型，该变量不可被更改内容，也不能 unset

### 5、变量内容的删除、取代与替换
![](/uploads/2017/12/linux_shell_variable_01.png)

### 6、用户登入 shell 后读取的两个配置文件：
1. /etc/profile：这是系统整体的设定，你最好不要修改这个档案；
2. ~/.bash_profile 或 ~/.bash_login 或 ~/.profile：属于使用者个人设定，你要改自己的数据，就写入这里。

### 7、常见的一些特殊变量
<style type="text/css">
	table th:first-of-type {
		width: 100px;
	}
</style>

参数处理  |  说明
----------|---------------------------
$0 | 当前脚本的名称
$# | 传递到脚本的参数个数
$? | 获取上一个命令的执行结果。0表示没有错误，其他任何值表明有错误。
$* | 以一个单字符串显示所有向脚本传递的参数。如"$*"用「"」括起来的情况、以"$1 $2 … $n"的形式输出所有参数。
$@ | 与$*相同，但是使用时加引号，并在引号中返回每个参数。如"$@"用「"」括起来的情况、以"$1" "$2" … "$n" 的形式输出所有参数。
$$ | 脚本运行的当前进程ID号
$! | 后台运行的最后一个进程的ID号
$- | 显示Shell使用的当前选项，与set命令功能相同。
