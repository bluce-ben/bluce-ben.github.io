---
title: 认识与学习BASH之Shell Scripts
date: 2018-01-03 17:14:14
categories:
- 书籍
- 《鸟哥的Linux私房菜基础篇（第三版）》
tags:
- 书籍
- BASH
- Shell
---
### 一、什么是Shell Script
什么是 shell script (程序化脚本) 呢？就字面上的意义，我们将他分为两部份。在【 shell 】部分，我们在 BASH 当中已经提过了，那是一个文字接口底下让我们与系统沟通的一个工具接口。那么【 script 】是啥？字面上的意义，script 是【脚本、剧本】的意思。整句话是说，shell script 是针对 shell 所写的【剧本！】

什么东西啊？其实，shell script 是利用 shell 的功能所写的一个【程序 (program)】，这个程序是使用纯文本文件，将一些 shell 的语法与指令(含外部指令)写在里面，搭配正规表示法、管线命令与数据流重导向等功能，以达到我们所想要的处理目的。
<!--more-->
所以，简单的说，shell script 就像是早期 DOS 年代的批处理文件 (.bat) ，最简单的功能就是将许多指令汇整写在一起，让使用者很轻易的就能够 one touch 的方法去处理复杂的动作 (执行一个档案"shell script" ，就能够一次执行多个指令)。而且 shell script 更提供数组、循环、条件与逻辑判断等重要功能，让用户也可以直接以 shell 来撰写程序，而不必使用类似 C 程序语言等传统程序撰写的语法。shell script 可以简单的被看成是批处理文件，也可以被说成是一个程序语言，且这个程序语言由于都是利用 shell 与相关工具指令，所以不需要编译即可执行，且拥有不错的除错 (debug) 工具，所以，他可以帮助系统管理员快速的管理好主机。

shell script 号称是程序 (program) ，但实际上，shell script 处理数据的速度上是不太够的。因为 shell script 用的是外部的指令与 bash shell 的一些默认工具，所以，他常常会去呼叫外部的函式库，因此，指令周期上面当然比不上传统的程序语言。所以，shell script 用在系统管理上面是很好的一项工具，但是用在处理大量数值运算上，就不够好了，因为 Shell scripts 的速度较慢，且使用的 CPU 资源较多，造成主机资源的分配不良。我们通常利用 shell script 来处理服务器的侦测，倒是没有进行大量运算的需求！所以不必担心。


#### Shell执行及注意事项
在 shell script 的撰写中还需要用到底下的注意事项：
1. 指令的执行是从上而下、从左而右的分析与执行；
2. 指令的下达就是： 指令、选项与参数间的多个空白都会被忽略掉；
3. 空白行也将被忽略掉，并且 [tab] 按键所推开的空白同样视为空格键；
4. 如果读取到一个 Enter 符号 (CR) ，就尝试开始执行该行 (或该串) 命令；
5. 至于如果一行的内容太多，则可以使用【 \[Enter] 】来延伸至下一行；
6. 【 # 】可做为批注！任何加在 # 后面的资料将全部被视为批注文字而被忽略！


如此一来，我们在 script 内所撰写的程序，就会被一行一行的执行。现在我们假设你写的这个程序文件名是 /home/dmtsai/shell.sh 好了，那如何执行这个档案？很简单，可以有底下几个方法：
* 直接指令下达： shell.sh 档案必须要具备可读与可执行 (rx) 的权限，然后：
	* 绝对路径：使用 /home/dmtsai/shell.sh 来下达指令；
	* 相对路径：假设工作目录在 /home/dmtsai/ ，则使用 ./shell.sh 来执行
	* 变量【PATH】功能：将 shell.sh 放在 PATH 指定的目录内，例如： ~/bin/(目录需要自行设定) 。此时，若 shell.sh 在 ~/bin 内且具有 rx 的权限，那就直接输入 shell.sh 即可执行该脚本程序！
* 以 bash 程序来执行：透过【 bash shell.sh 】或【 sh shell.sh 】来执行

反正重点就是要让那个 shell.sh 内的指令可以被执行的意思。

### 二、简单的shell script练习
#### （1）撰写第一支 script：输出Hello World！
```
[root@www ~]# mkdir scripts; cd scripts
[root@www scripts]# vi sh01.sh
#!/bin/bash
# Program:
# This program shows "Hello World!" in your screen.
# History:
# 2005/08/23 VBird First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH
echo -e "Hello World! \a \n"
exit 0
```
在本章当中，请将所有撰写的 script 放置到你家目录的 ~/scripts 这个目录内，未来比较好管理。
上面的写法当中，主要将整个程序的撰写分成数段，大致是这样：
>1.第一行 #!/bin/bash 在宣告这个 script 使用的 shell 名称：
因为我们使用的是 bash ，所以，必须要以【 #!/bin/bash 】来宣告这个档案内的语法使用 bash 的语法！那么当这个程序被执行时，他就能够加载 bash 的相关环境配置文件 (一般来说就是 non-login shell 的 ~/.bashrc)， 并且执行 bash 来使我们底下的指令能够执行！这很重要！(在很多状况中，如果没有设定好这一行，那么该程序很可能会无法执行，因为系统可能无法判断该程序需要使用什么 shell 来执行！)
>2.程序内容的说明：
整个 script 当中，除了第一行的【 #! 】是用来宣告 shell 的之外，其他的 # 都是【批注】用途！所以上面的程序当中，第二行以下就是用来说明整个程序的基本数据。一般来说，建议你一定要养成说明该 script 的：1. 内容与功能； 2. 版本信息； 3. 作者与联绚方式； 4. 建檔日期；5. 历史纪录 等等。这将有助于未来程序的改写与 debug。
>3.主要环境变量的宣告：
建议务必要将一些重要的环境变量设定好，PATH 与 LANG (如果有使用到输出相关的信息时) 是当中最重要的！如此一来，则可让我们这支程序在进行时，可以直接下达一些外部指令，而不必写绝对路径！
>4.主要程序部分
就将主要的程序写好即可！在这个例子当中，就是 echo 那一行！
>5.执行成果告知 (定义回传值)
是否记得我们在讨论一个指令的执行成功与否，可以使用 $? 这个变量来观察。那么我们也可以利用 exit 这个指令来让程序中断，并且回传一个数值给系统。在我们这个例子当中，使用 exit 0 ，这代表离开 script 并且回传一个 0 给系统，所以我执行完这个 script 后，若接着下达 echo $? 则可得到 0 的值！更聪明的读者应该也知道，利用这个 exit n (n 是数字) 的功能，我们还可以自定义错误讯息，让这支程序发得更加的 smart ！接下来透过刚刚上头介绍的执行方法来执行看看结果吧。

#### （2）简单范例
```
（1）对谈式脚本：变量内容由用户决定
[root@www scripts]# vi sh02.sh
#!/bin/bash
# Program:
# User inputs his first name and last name. Program shows his full
name.
# History:
# 2005/08/23 VBird First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH
read -p "Please input your first name: " firstname # 提示使用者输入
read -p "Please input your last name: " lastname # 提示使用者输入
echo -e "\nYour full name is: $firstname $lastname" # 结果由屏幕输出

（2）随日期变化：利用date进行档案的建立
[root@www scripts]# vi sh03.sh
#!/bin/bash
# Program:
# Program creates three files, which named by user's input
# and date command.
# History:
# 2005/08/23 VBird First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH
# 1. 让使用者输入文件名，并取得 fileuser 这个变量；
echo -e "I will use 'touch' command to create 3 files." # 纯粹显示信息
read -p "Please input your filename: " fileuser # 提示使用者输入
# 2. 为了避免使用者随意按 Enter ，利用变量功能分析档名是否有设定？
filename=${fileuser:-"filename"} # 开始判断有否配置文件名
# 3. 开始利用 date 指令来取得所需要的档名；
date1=$(date --date='2 days ago' +%Y%m%d) # 前两天的日期
date2=$(date --date='1 days ago' +%Y%m%d) # 前一天的日期
date3=$(date +%Y%m%d) # 今天的日期
file1=${filename}${date1} # 底下三行在配置文件名
file2=${filename}${date2}
file3=${filename}${date3}
# 4. 将档名建立
touch "$file1" # 底下三行在建立档案
touch "$file2"
touch "$file3"

（3）数值运算：简单的加减乘除
[root@www scripts]# vi sh04.sh
#!/bin/bash
# Program:
# User inputs 2 integer numbers; program will cross these two
numbers.
# History:
# 2005/08/23 VBird First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH
echo -e "You SHOULD input 2 numbers, I will cross them! \n"
read -p "first number: " firstnu
read -p "second number: " secnu
total=$(($firstnu*$secnu))
echo -e "\nThe result of $firstnu x $secnu is ==> $total"
# 在数值的运算上，我们可以使用【 declare -i total=$firstnu*$secnu 】
# 也可以使用上面的方式来进行。基本上，比较建议使用这样的方式来进行运算：
	var=$((运算内容))
# 不但容易记忆，而且也比较方便的多，因为两个小括号内可以加上空格符！
# 未来你可以使用这种方式来计算。至于数值运算上的处理，则有：【 +, -, *, /, % 】等等。
```

### 三、判断式
#### （1）利用 test 指令的测试功能
![](/uploads/2018/01/linux_shell_test_01.png)
![](/uploads/2018/01/linux_shell_test_02.png)
![](/uploads/2018/01/linux_shell_test_03.png)
![](/uploads/2018/01/linux_shell_test_04.png)
![](/uploads/2018/01/linux_shell_test_05.png)

OK！现在我们就利用 test 来帮我们写几个简单的例子。首先，判断一下，让使用者输入一个档名，我们判断：
1. 这个档案是否存在，若不存在则给予一个【Filename does not exist】的讯息，并中断程序；
2. 若这个档案存在，则判断他是个档案或目录，结果输出【Filename is regular file】或【Filename is directory】
3. 判断一下，执行者的身份对这个档案或目录所拥有的权限，并输出权限数据！
**（注意利用 test 与 && 还有 || 等标志！）**

```
[root@www scripts]# vi sh05.sh
#!/bin/bash
# Program:
# User input a filename, program will check the flowing:
# 1.) exist? 2.) file/directory? 3.) file permissions
# History:
# 2005/08/25 VBird First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH
# 1. 讥使用者输入档名，幵且刞断使用者是否真的有输入字符串？
echo -e "Please input a filename, I will check the filename's type and \
permission. \n\n"
read -p "Input a filename : " filename
test -z $filename && echo "You MUST input a filename." && exit 0
# 2. 刞断档案是否存在？若丌存在则显示讯息幵结束脚本
test ! -e $filename && echo "The filename '$filename' DO NOT exist" &&
exit 0
# 3. 开始刞断文件类型不属性
test -f $filename && filetype="regulare file"
test -d $filename && filetype="directory"
test -r $filename && perm="readable"
test -w $filename && perm="$perm writable"
test -x $filename && perm="$perm executable"
# 4. 开始输出信息！
echo "The filename: $filename is a $filetype"
echo "And the permissions are : $perm"
```

#### （2）利用判断符号 [ ]
除了我们很喜欢使用的 test 之外，其实，我们还可以利用判断符号【 [ ] 】(就是中括号) 来进行数据的判断。举例来说，如果我想要知道 $HOME 这个变量是否为空的，可以这样做：
`[root@www ~]# [ -z "$HOME" ] ; echo $?`
使用中括号必须要特别注意，因为中括号用在很多地方，包括通配符与正规表示法等等，所以如果要在 bash 的语法当中使用中括号作为 shell 的判断式时，必须要注意中括号的两端需要有空格符来分隔。
最好要注意：
* 在中括号 [] 内的每个组件都需要有空格键来分隔；
* 在中括号内的变数，最好都以双引号括号起来；
* 在中括号内的常数，最好都以单或双引号括号起来。

举例来说，假如我设定了 `name="VBird Tsai"` ，然后这样判定：
```
[root@www ~]# name="VBird Tsai"
[root@www ~]# [ $name == "VBird" ]
bash: [: too many arguments
```
怎么会发生错误？bash 还跟我说错误是由于【太多参数 (arguments)】所致。为什么呢？
因为 $name 如果没有使用双引号刮起来，那么上面的判定式会变成：
	`[ VBird Tsai == "VBird" ]`
上面肯定不对了。因为一个判断式仅能有两个数据的比对，上面 VBird 与 Tsai 还有 "VBird" 就有三个资料。这不是我们要的，我们要的应该是底下这个样子：
	`[ "VBird Tsai" == "VBird" ]`
这可是差很多。另外，中括号的使用方法与 test 几乎一模一样。


### 四、条件判断式
#### （1）利用if...then判断式
（1）单层、简单条件判断式
```
if [ 条件判断式 ]; then
当条件判断式成立时，可以进行的指令工作内容；
fi <==将 if 反过来写，就成为 fi ！结束 if 之意！

括号与括号之间，则以 && 或 || 来隔开，他们的意义是：
	&& 代表 AND ；
	|| 代表 or ；
例如：[ "$yn" == "Y" -o "$yn" == "y" ] 可替换为：[ "$yn" == "Y" ] || [ "$yn" == "y" ]
```
（2）多重、复杂条件判断式
```
# 一个条件判断，分成功进行与失败进行 (else)
if [ 条件判断式 ]; then
	当条件判断式成立时，可以进行的指令工作内容；
else
	当条件判断式不成立时，可以进行的指令工作内容；
fi

# 多个条件判断 (if ... elif ... elif ... else) 分多种不同情况执行
if [ 条件判断式一 ]; then
	当条件判断式一成立时，可以进行的指令工作内容；
elif [ 条件判断式二 ]; then
	当条件判断式二成立时，可以进行的指令工作内容；
else
	当条件判断式一与二均不成立时，可以进行的指令工作内容；
fi
```

#### （2）利用 case ..... esac 判断
```
case $变量名称 in <==关键词为 case ，还有变数前有钱字号
"第一个变量内容") <==每个变量内容建议用双引号括起来，关键词则为小括号 )
	程序段
	;; <==每个类别结尾使用两个连续的分号来处理！
"第二个变量内容")
	程序段
	;;
*) <==最后一个变量内容都会用 * 来代表所有其他值，不包含第一个变量内容与第二个变量内容的其他程序执行段
	exit 1
	;;
esac <==最终的 case 结尾！【反过来写】思考一下！
```
一般来说，使用【 case $变量 in 】这个语法中，当中的那个【 $变量 】大致有两种取得的方式：
* 直接下达式：例如上面提到的，利用【 script.sh variable 】的方式来直接给予 $1 这个变量的内容，这也是在 /etc/init.d 目录下大多数程序的设计方式。
* 交互式：透过 read 这个指令来让用户输入变量的内容。

### 五、循环
#### （1）while do done, until do done (不定循环)
一般来说，不定循环最常见的就是底下这两种状态了：
```
while [ condition ] <==中括号内的状态就是判断式
do 		<==do 是循环的开始！
	程序段落
done 	<==done 是循环的结束
```
while 的中文是【当....时】，所以，这种方式说的是【当 condition 条件成立时，就进行循环，直到condition 的条件不成立才停止】的意思。还有另外一种不定循环的方式：
```
until [ condition ]
do
	程序段落
done
```
这种方式恰恰与 while 相反，它说的是【当 condition 条件成立时，就终止循环，否则就持续进行循环的程序段。】

#### （2）for...do...done (固定循环)
相对于 while, until 的循环方式是必须要【符合某个条件】的状态，for 这种语法，则是【 已经知道要进行几次循环】的状态！他的语法是：
```
for var in con1 con2 con3 ...
do
	程序段
done
```
>以上面的例子来说，这个 $var 的变量内容在循环工作时：
	1. 第一次循环时， $var 的内容为 con1 ；
	2. 第二次循环时， $var 的内容为 con2 ；
	3. 第三次循环时， $var 的内容为 con3 ；
	4. ....

#### （3）for...do...done 的数值处理
除了上述的方法之外，for 循环还有另外一种写法！语法如下：
```
for (( 初始值; 限制值; 执行步阶 ))
do
	程序段
done
```
这种语法适合于数值方式的运算当中，在 for 后面的括号内的三串内容意义为：
* 初始值：某个变量在循环当中的起始值，直接以类似 i=1 设定好；
* 限制值：当变量的值在这个限制值的范围内，就继续进行循环。例如 i<=100；
* 执行步阶：每作一次循环时，变量的变化量。例如 i=i+1。

值得注意的是，在【执行步阶】的设定上，如果每次增加 1 ，则可以使用类似【i++】的方式，亦即是 i 每次循环都会增加一的意思。

### 六、shell script的追踪与debug
scripts 在执行之前，最怕的就是出现语法错误的问题了！那么我们如何 debug 呢？有没有办法不需要透过直接执行该 scripts 就可以来判断是否有问题呢？当然是有的，我们就直接以 bash 的相关参数来进行判断！
```
[root@www ~]# sh [-nvx] scripts.sh
选项与参数：
-n ：不要执行 script，仅查询语法的问题；
-v ：再执行 sccript 前，先将 scripts 的内容输出到屏幕上；
-x ：将使用到的 script 内容显示到屏幕上，这是很有用的参数！
范例一：测试 sh16.sh 有无语法的问题？
[root@www ~]# sh -n sh16.sh
# 若语法没有问题，则不会显示任何信息！
范例二：将 sh15.sh 的执行过程全部列出来～
[root@www ~]# sh -x sh15.sh
```
熟悉 sh 的用法，将可以使你在管理 Linux 的过程中得心应手！至于在 Shell scripts 的学习方法上面，需要【多看、多模仿、并加以修改成自己的样式！】是最快的学习手段了！网络上有相当多的朋友在
开发一些相当有用的 scripts ，若是你可以将对方的 scripts 拿来，并且改成适合自己主机的样子！那么学习的效果会是最快的！

另外，我们 Linux 系统本来就有很多的服务启动脚本，如果你想要知道每个 script 所代表的功能是什么？可以直接以 vim 进入该 script 去查阅一下，通常立刻就知道该 script 的目的了。 举例来说，我们之前一直提到的 /etc/init.d/syslog ，这个 script 是干嘛用的？ 利用 vi 去查阅最前面的几行字，他出现如下信息：
```
# description: Syslog is the facility by which many daemons use to log \
# messages to various system log files. It is a good idea to always \
# run syslog.
### BEGIN INIT INFO
# Provides: $syslog
### END INIT INFO
```
简单的说，这个脚本在启动一个名为 syslog 的常驻程序 (daemon)，这个常驻程序可以帮助很多系统服务记载她们的登录文件 (log file)，我们的 Linux 建议你一直启动 syslog 是个好主意！简单的看看您就知道啥是啥啦！

### 附属：
#### Shell Script中的特殊变量：
Shell script 的默认变数($0, $1...)
一些较为特殊的变量可以在 script 内使用来呼叫这些参数：
* $# ：代表后接的参数【个数】，以上表为例这里显示为【 4 】；
* $@ ：代表【 "$1" "$2" "$3" "$4" 】之意，每个变量是独立的(用双引号括起来)；
* $* ：代表【 "$1c$2c$3c$4" 】，其中 c 为分隔字符，默认为空格键，所以本例中代表【 "$1 $2 $3 $4" 】之意。
* $0 ：代表执行的脚本档名
* $n ：代表对应输入的变量

#### 常用的转义字符
**4个最常用的转义字符**，如下所示：
>**反斜杠（\）：**使反斜杠后面的一个变量变为单纯的字符串。
**单引号（''）：**转义其中所有的变量为单纯的字符串。
**双引号（""）：**保留其中的变量属性，不进行转义处理。
**反引号（``）：**把其中的命令执行后返回结果。