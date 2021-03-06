---
title: Linux命令之awk
date: 2017-12-29 17:56:49
categories:
- Linux
tags:
- Linux
- Linux命令
---
#### awk：好用的数据处理工具
awk 也是一个非常棒的数据处理工具。相较于 sed 常常作用于一整个行的处理，awk 则比较倾向于一行当中分成数个【字段】来处理。因此，awk 相当的适合处理小型的数据数据处理。awk 通常运作的模式是这样的：
`[root@www ~]# awk '条件类型1{动作1} 条件类型2{动作2} ...' filename`

>**awk脚本基本结构**
`awk 'BEGIN{ commands } pattern{ commands } END{ commands }' file`
**工作原理：**
第一步：执行BEGIN{ commands }语句块中的语句；**BEGIN语句块**在awk开始从输入流中<font color="red">读取行之前被执行</font>，这是一个可选的语句块，比如变量初始化、打印输出表格的表头等语句通常可以写在BEGIN语句块中。
第二步：从文件或标准输入(stdin)读取一行，然后执行pattern{ commands }语句块，它逐行扫描文件，从第一行到最后一行重复这个过程，直到文件全部被读取完毕。**pattern语句块**中的通用命令是最重要的部分，它也是可选的。<font color="red">如果没有提供pattern语句块，则默认执行{ print }，即打印每一个读取到的行</font>，awk读取的每一行都会执行该语句块。
第三步：当读至输入流末尾时，执行END{ commands }语句块。**END语句块**在awk从输入流中<font color="red">读取完所有的行之后即被执行</font>，比如打印所有行的分析结果这类信息汇总都是在END语句块中完成，它也是一个可选语句块。


awk 后面接两个单引号（<font color="red">双引号一般在动作中使用</font>）并加上大括号 {} 来设定想要对数据进行的处理动作。awk 可以处理后续接的档案，也可以读取来自前个指令的 standard output 。但如前面说的，awk 主要是处理【每一行的字段内的数据】，而默认的【字段的分隔符为 "空格键" 或 "[tab]键"】。
<!--more-->
举例来说，我们用 last 可以将登入者的数据取出来，结果如下所示：
```
[root@www ~]# last -n 5 <==仅取出前五行
root pts/1 192.168.1.100 Tue Feb 10 11:21 still logged in
root pts/1 192.168.1.100 Tue Feb 10 00:46 - 02:28 (01:41)
root pts/1 192.168.1.100 Mon Feb 9 11:41 - 18:30 (06:48)
dmtsai pts/1 192.168.1.100 Mon Feb 9 11:41 - 11:41 (00:00)
root tty1 Fri Sep 5 14:09 - 14:10 (00:01)
```
若我想要取出账号与登入者的 IP ，且账号与 IP 之间以 [tab] 隔开，则会变成这样：
```
[root@www ~]# last -n 5 | awk '{print $1 "\t" $3}'
root 192.168.1.100
root 192.168.1.100
root 192.168.1.100
dmtsai 192.168.1.100
root Fri
```
上表是 awk 最常使用的动作。透过 print 的功能将字段数据列出来，字段的分隔则以空格键或 [tab] 按键来隔开。因为不论哪一行我都要处理，因此，就不需要有 "条件类型" 的限制。我所想要的是第一栏以及第三栏，但是，第五行的内容怪怪的，这是因为数据格式的问题。所以，使用 awk 的时候，请先确认一下你的数据当中，如果是连续性的数据，请不要有空格或 [tab] 在内，否则，就会像这个例子这样，会发生误判。<font color="red">（注：条件类型是可有可无的）</font>

另外，由上面这个例子你也会知道，在每一行的每个字段都是有变量名称的，那就是 $1, $2... 等变量名称。以上面的例子来说，root 是 $1 ，因为他是第一栏。至于 192.168.1.100 是第三栏，所以他就是 $3，后面以此类推。还有个变数，那就是 $0 ，$0 代表【一整列资料】的意思。以上面的例子来说，第一行的 $0 代表的就是【root ....】那一行。 由此可知，刚刚上面五行当中，整个 awk 的处理流程是：

	1. 读入第一行，并将第一行的资料填入 $0, $1, $2.... 等变数当中；
	2. 依据 "条件类型" 的限制，判断是否需要进行后面的 "动作"；
	3. 做完所有的动作与条件类型；
	4. 若还有后续的【行】的数据，则重复上面 1~3 的步骤，直到所有的数据都读完为止。
经过这样的步骤，你会晓得，awk 是【以行为一次处理的单位】，而【以字段为最小的处理单位】。那么 awk 怎么知道我到底这个数据有几行？有几栏？这就需要 awk 的内建变量的帮忙了。

变量名称 | 代表意义
---------|------------------
$0       | 这个变量包含执行过程中当前行的文本内容。
NF 		 | 表示字段数，在执行过程中对应于当前的字段数。
NR 		 | 表示记录数，在执行过程中对应于当前的行号。
FS 		 | 目前的分隔字符，默认是空格键

我们继续以上面 last -n 5 的例子来做说明，如果我想要：
* 列出每一行的账号(就是 $1)；
* 列出目前处理的行数(就是 awk 内的 NR 变量)
* 并且说明，该行有多少字段(就是 awk 内的 NF 变量)

*（注：awk 后续的所有动作是以单引号【 ' 】括住的，由于单引号与双引号都必须是成对的，所以，awk的格式内容如果想要以print打印时，记得非变量的文字部分，包含printf的格式中，都需要使用双引号来定义出来。因为单引号已经是 awk 的指令固定用法了！）*

```
[root@www ~]# last -n 5| awk '{print $1 "\t lines: " NR "\t columes: " NF}'
root lines: 1 columes: 10
root lines: 2 columes: 10
root lines: 3 columes: 10
dmtsai lines: 4 columes: 10
root lines: 5 columes: 9
# 注意，在 awk 内的 NR, NF 等变量要用大写，且不需要有钱字号 $
```

#### awk 的逻辑运算字符
既然有需要用到 "条件" 的类别，自然就需要一些逻辑运算。例如底下这些：
<font color="red">（注：条件类型一般都使用逻辑运算符来进行判断。）</font>
![](/uploads/2017/12/linux_shell_awk_01.png)
值得注意的是那个【 == 】的符号，因为：	
* 逻辑运算上面亦即所谓的大于、小于、等于等判断式上面，习惯上是以【 == 】来表示；
* 如果是直接给予一个值，例如变量设定时，就直接使用 = 而已。

举例来说，在 /etc/passwd 当中是以冒号 ":" 来作为字段的分隔，该档案中第一字段为账号，第三字段则是UID。那假设我要查阅，第三栏小于 10 以下的数据，并且仅列出账号与第三栏，那么可以这样做：
```
[root@www ~]# cat /etc/passwd | awk '{FS=":"} $3 < 10 {print $1 "\t " $3}'
root:x:0:0:root:/root:/bin/bash
bin 1
daemon 2
....(以下省略)....
```
<font color="red">（注：分隔符上述不好使可以使用如下形式：`awk FS=':' '{}'`）</font>

另外，如果要用 awk 来进行【计算功能】呢？以底下的例子来看，假设我有一个薪资数据表档名为 pay.txt ，内容是这样的：
```
Name 1st 2nd 3th
VBird 23000 24000 25000
DMTsai 21000 20000 23000
Bird2 43000 42000 41000
```
如何帮我计算每个人的总额呢？而且我还想要格式化输出。我们可以这样考虑：
* 第一行只是说明，所以第一行不要进行加总 (NR==1 时处理)；
* 第二行以后就会有加总的情况出现 (NR>=2 以后处理)

```
[root@www ~]# cat pay.txt | \
> awk 'NR==1{printf
"%10s %10s %10s %10s %10s\n",$1,$2,$3,$4,"Total" }
NR>=2{total = $2 + $3 + $4
printf "%10s %10d %10d %10d %10.2f\n", $1, $2, $3, $4, total}'
Name 1st 2nd 3th Total
VBird 23000 24000 25000 72000.00
DMTsai 21000 20000 23000 64000.00
Bird2 43000 42000 41000 126000.00
```
上面的例子有几个重要事项应该要先说明的：
* awk 的指令间隔：所有 awk 的动作，亦即在 {} 内的动作，如果有需要多个指令辅助时，可利用分号【;】间隔，或者直接以 [Enter] 按键来隔开每个指令
* 逻辑运算当中，如果是【等于】的情况，则务必使用两个等号【==】！
* 格式化输出时，在 printf 的格式设定当中，务必加上 \n ，才能进行分行！
* 与 bash shell 的变量不同，在 awk 当中，变量可以直接使用，不需加上 $ 符号。

利用 awk 这个玩意儿，就可以帮我们处理很多日常工作了呢！真是好用的很。此外，awk 的输出格式当中，常常会以 printf 来辅助，所以，最好你对 printf 也稍微熟悉一下比较好。另外，awk 的动作内 {} 也是支持 if (条件) 的。举例来说，上面的指令可以修订成为这样：
```
[root@www ~]# cat pay.txt | \
> awk '{if(NR==1) printf
"%10s %10s %10s %10s %10s\n",$1,$2,$3,$4,"Total"}
NR>=2{total = $2 + $3 + $4
printf "%10s %10d %10d %10d %10.2f\n", $1, $2, $3, $4, total}'
```

#### awk示例
##### awk给文本文件每行添加行号
假定文本内容如下：
```
first line
second line
third line
```
现在要在每行前面添加行号，格式为："1 "。
可通过如下命令实现：
`awk '$0=NR" "$0'`
最后，通过重定向符可导入某个文件中。

##### awk查看文本文件的某一行数据
可使用如下命令实现：
`awk 'NR==num' filename`


#### 格式化打印：printf
```
[root@www ~]# printf '打印格式' 实际内容
选项与参数：
关于格式方面的几个特殊样式：
	\a 警告声音输出
	\b 退格键(backspace)
	\f 清除屏幕 (form feed)
	\n 输出新的一行
	\r 亦即 Enter 按键
	\t 水平的 [tab] 按键
	\v 垂直的 [tab] 按键
	\xNN NN 为两位数的数字，可以转换数字成为字符。
关于 C 程序语言内，常见的变数格式
	%ns 那个 n 是数字， s 代表 string ，亦即多少个字符；
	%ni 那个 n 是数字， i 代表 integer ，亦即多少整数字数；
	%N.nf 那个 n 与 N 都是数字， f 代表 floating (浮点)，如果有小数字数，假设我共要十个位数，但小数点有两位，即为 %10.2f
```
**（注：printf 不是管线命令）**
