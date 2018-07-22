---
title: 认识与学习BASH之管线命令
date: 2017-12-28 19:11:21
categories:
- 书籍
- 《鸟哥的Linux私房菜基础篇（第三版）》
tags:
- 书籍
- BASH
- Shell
---
就如同《数据流导向》所说的，bash 命令执行的时候有输出的数据会出现，那么如果这群数据必需要经过几道手续之后才能得到我们所想要的格式，应该如何来设定？这就牵涉到管线命令的问题(pipe) ，管线命令使用的是【 | 】这个界定符号。另外，管线命令与【连续下达命令】是不一样的。这点底下我们会再说明。底下我们先举一个例子来说明一下简单的管线命令。
```
示例：显示/etc/地下有多少个档案，并且利用less指令来分页显示
[root@www ~]# ls -al /etc | less
```
<!--more-->
其实这个管线命令【 | 】仅能处理经由前面一个指令传来的正确信息，也就是 standard output 的信息，对于 stdandard error 并没有直接处理的能力。那么整体的管线命令可以使用下图表示：
![](/uploads/2017/12/linux_pipe_command.JPG)

<font color="red">在每个管线后面接的第一个数据必定是【指令】(管线指令)。</font>而且这个指令必须要能够接收 standard input 的数据才行，这样的指令才可以是为【管线命令】，例如 less, more, head, tail 等都是可以接收 standard input 的管线命令。至于例如 ls, cp, mv 等就不是管线命令。因为 ls, cp, mv 并不会接收来自 stdin的数据。也就是说，管线命令主要有两个比较需要注意的地方：
>1.管线命令仅会处理 standard output，对于 standard error output 会予以忽略
2.管线命令必须要能够接收来自前一个指令的数据成为 standard input 继续处理才行。

### 1、撷取命令： cut, grep
什么是撷取命令？就是将一段数据经过分析后，取出我们所想要的。或者是经由分析关键词，取得我们所想要的那一行。不过，要注意的是，一般来说，撷取讯息通常是针对【一行一行】来分析的，并不是整篇讯息分析的，底下我们介绍两个很常用的讯息撷取命令：
#### cut
这个指令可以将一段讯息的某一段给他【切】出来，处理的讯息是以【行】为单位，底下我们就来谈一谈：
```
[root@www ~]# cut -d'分隔字符' -f fields <==用于有特定分隔字符
[root@www ~]# cut -c 字符区间 <==用于排列整齐的讯息
选项与参数：
    -d ：后面接分隔字符。与 -f 一起使用；
    -f ：依据 -d 的分隔字符将一段讯息分割成为数段，用 -f 取出第几段的意思；
    -c ：以字符 (characters) 的单位取出固定字符区间；
范例一：将 PATH 变量取出，我要找出第五个路径。
[root@www ~]# echo $PATH
/bin:/usr/bin:/sbin:/usr/sbin:/usr/local/bin:/usr/X11R6/bin:/usr/games:
# 1 | 2 | 3 | 4 | 5 | 6 | 7

[root@www ~]# echo $PATH | cut -d ':' -f 5
# 如同上面的数字显示，我们是以【 : 】作为分隔，因此会出现 /usr/local/bin
# 那么如果想要列出第 3 与第 5 呢？，就是这样：
[root@www ~]# echo $PATH | cut -d ':' -f 3,5
范例二：将 export 输出癿讯息，取得第 12 字符以后的所有字符串
[root@www ~]# export
declare -x HISTSIZE="1000"
declare -x INPUTRC="/etc/inputrc"
declare -x KDEDIR="/usr"
declare -x LANG="zh_TW.big5"
.....(其他省略).....
# 注意看，每个数据都是排列整齐的输出！如果我们不想要【declare -x】时，就得这么做：
[root@www ~]# export | cut -c 12-
HISTSIZE="1000"
INPUTRC="/etc/inputrc"
KDEDIR="/usr"
LANG="zh_TW.big5"
.....(其他省略).....
# 知道怎么回事了吧？用 -c 可以处理比较具有格式的输出数据！
# 我们还可以指定某个范围的值，例如第 12-20 的字符，就是 cut -c 12-20 等等！
```
*cut 主要的用途在于将【同一行里面的数据进行分解！】最常使用在分析一些数据或文字数据的时候！这是因为有时候我们会以某些字符当作分割的参数，然后来将数据加以切割，以取得我们所需要的数据。*

#### grep
刚刚的 cut 是将一行讯息当中，取出某部分我们想要的，而 grep 则是分析一行讯息，若当中有我们所需要的信息，就将该行拿出来，简单的语法是这样的：
```
[root@www ~]# grep [-acinv] [--color=auto] '搜寻字符串' filename
选项与参数：
    -a ：将 binary 档案以 text 档案的方式搜寻数据
    -c ：计算找到 '搜寻字符串' 的次数
    -i ：忽略大小写的不同，所以大小写视为相同
    -n ：顺便输出行号
    -v ：反向选择，亦即显示出没有 '搜寻字符串' 内容的那一行！
    --color=auto ：可以将找到的关键词部分加上颜色的显示。

范例一：将 last 当中，有出现 root 的那一行就取出来；
[root@www ~]# last | grep 'root'
范例二：与范例一相反，只要没有 root 的就取出！
[root@www ~]# last | grep -v 'root'
范例三：在 last 的输出讯息中，只要有 root 就取出，并且仅取第一栏
[root@www ~]# last | grep 'root' |cut -d ' ' -f1
范例四：取出 /etc/man.config 内含 MANPATH 的那几行
[root@www ~]# grep --color=auto 'MANPATH' /etc/man.config
....(前面省略)....
MANPATH_MAP /usr/X11R6/bin /usr/X11R6/man
MANPATH_MAP /usr/bin/X11 /usr/X11R6/man
MANPATH_MAP /usr/bin/mh /usr/share/man
（如果加上 --color=auto 的选项，找到的关键词部分会用特殊颜色显示。）
```

### 2、排序命令： sort, wc, uniq
#### sort
他可以帮我们进行排序，而且可以依据不同的数据型态来排序。例如数字与文字的排序就不一样。此外，排序的字符与语系的编码有关，因此，如果您需要排序时，建议使用LANG=C来让语系统一，数据排序会比较好一些。
```
[root@www ~]# sort [-fbMnrtuk] [file or stdin]
选项与参数：
    -f ：忽略大小写的差异，例如 A 与 a 规为编码相同；
    -b ：忽略最前面的空格符部分；
    -M ：以月份的名字来排序，例如 JAN, DEC 等等的排序方法；
    -n ：使用【纯数字】进行排序(默认是以文字型态来排序的)；
    -r ：反向排序；
    -u ：就是 uniq ，相同的数据中，仅出现一行代表；
    -t ：分隔符，预设是用 [tab] 键来分隔；
    -k ：以那个区间 (field) 来进行排序的意思

范例一：个人账号都记录在 /etc/passwd 下，请将账号进行排序。
[root@www ~]# cat /etc/passwd | sort
adm:x:3:4:adm:/var/adm:/sbin/nologin
apache:x:48:48:Apache:/var/www:/sbin/nologin
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
#由上面的数据看起来，sort 是预设【以第一个】数据来排序，而且默认是以【文字】型态来排序的。所以由a开始排到最后。
范例二：/etc/passwd 内容是以 : 来分割的，我想以第三栏来排序，该如何？
[root@www ~]# cat /etc/passwd | sort -t ':' -k 3
root:x:0:0:root:/root:/bin/bash
uucp:x:10:14:uucp:/var/spool/uucp:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
bin:x:1:1:bin:/bin:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
# 如果是以文字型态来排序的，就是会这样，想要使用数字排序：
# cat /etc/passwd | sort -t ':' -k 3 -n
# 这样才行。用那个 -n 来告知 sort 以数字来排序。
```
#### uniq
如果我排序完成了，想要将重复的资料仅列出一个显示，可以怎么做呢？
```
[root@www ~]# uniq [-ic]
选项与参数：
    -i ：忽略大小写字符的不同；
    -c ：进行计数
范例一：使用 last 将账号列出，仅取出账号栏，进行排序后仅取出一位；
[root@www ~]# last | cut -d ' ' -f1 | sort | uniq
范例二：承上题，如果我还想要知道每个人的登入总次数
[root@www ~]# last | cut -d ' ' -f1 | sort | uniq -c
1
12 reboot
41 root
1 wtmp
# 从上面的结果可以发现 reboot 有 12 次， root 登入则有 41 次。wtmp与第一行的空白都是last的默认字符，那两个可以忽略。
```
*这个指令用来将【重复的行删除掉只显示一个】，举个例子来说，你要知道这个月份登入你主机的用户有谁，而不在乎他的登入次数，那么就使用上面的范例，(1)先将所有的数据列出；(2)再将人名独立出来；(3)经过排序；(4)只显示一个！由于这个指令是在将重复的东西减少，所以当然需要【配合排序过的档案】来处理。*

### 3、统计命令：wc
如果我想要知道 /etc/man.config这个档案里面有多少字？多少行？多少字符的话，可以利用wc这个指令来达成，他可以帮我们计算输出的讯息的整体数据。
```
[root@www ~]# wc [-lwm]
选项与参数：
    -l ：仅列出行；
    -w ：仅列出多少字(英文单字)；
    -m ：多少字符；
```
*当你要知道目前你的账号档案中有多少个账号时，就使用这个方法：【cat /etc/passwd | wc -l】。因为/etc/passwd里头一行代表一个使用者。所以知道行数就晓得有多少账号在里头。而如果要计算一个档案里头有多少个字符时，就使用 wc -c 这个选项。*


### 4、字符转换命令： tr, col, join, paste, expand
我们在 vim 程序编辑器当中，提到过 DOS 断行字符与 Unix 断行字符的不同，并且可以使用 dos2unix 与 unix2dos 来完成转换。当然，还有其他的替代方案，底下我们就来介绍一下这些字符转换命令在管线当中的使用方法：

#### tr
tr 可以用来删除一段讯息当中的文字，或者是进行文字讯息的替换。
```
[root@www ~]# tr [-ds] SET1 ...
选项与参数：
    -d ：删除讯息当中的 SET1 这个字符串；
    -s ：取代掉重复的字符！
范例一：将 last 输出的讯息中，所有的小写变成大写字符：
[root@www ~]# last | tr '[a-z]' '[A-Z]'
# 事实上，没有加上单引号也是可以执行的，如：【 last | tr [a-z] [A-Z] 】
范例二：将 /etc/passwd 输出的讯息中，将冒号 (:) 删除
[root@www ~]# cat /etc/passwd | tr -d ':'
```
相信处理 Linux & Windows 系统中的人们最麻烦的一件事就是 DOS 底下会自动的在每行行尾加入^M 这个断行符号。使用 tr 将 ^M 可以使用 \r 来代替就可以去除 DOS 档案留下来的 ^M 这个断行的符号。这东西相当的有用！

#### col
```
[root@www ~]# col [-xb]
选项与参数：
    -x ：将 tab 键转换成对等的空格键
    -b ：在文字内有反斜杠 (/) 时，仅保留反斜杠最后接的那个字符
```

#### join
join 看字面上的意义 (加入/参加) 就可以知道，他是在处理两个档案之间的数据，而且，主要是在处理【两个档案当中，有"相同数据"的那一行，才将他加在一起】的意思。我们利用底下的简单例子来说明：
```
[root@www ~]# join [-ti12] file1 file2
选项与参数：
    -t ：join 默认以空格符分隔数据，并且比对【第一个字段】的数据，如果两个档案相同，则将两笔数据联成一行，且第一个字段放在第一个
    -i ：忽略大小写的差异；
    -1 ：这个是数字的 1 ，代表【第一个档案要用那个字段来分析】的意思；
    -2 ：代表【第二个档案要用那个字段来分析】的意思。

范例一：用 root 的身份，将 /etc/passwd 与 /etc/shadow 相关数据整合成一栏
[root@www ~]# join -t ':' /etc/passwd /etc/shadow
# 透过上面这个动作，我们可以将两个档案第一字段相同者整合成一行。第二个档案的相同字段并不会显示
范例二：我们知道 /etc/passwd 第四个字段是 GID ，那个 GID 记录在 /etc/group 当中的第三个字段
[root@www ~]# join -t ':' -1 4 /etc/passwd -2 3 /etc/group
# 同样的，相同的字段部分被移动到最前面了。所以第二个档案的内容就没再显示
```

#### paste
这个 paste 就要比 join 简单多了。相对于 join 必须要比对两个档案的数据相关性，paste 就直接【将两行贴在一起，且中间以 [tab] 键隔开】而已。简单的使用方法：
```
[root@www ~]# paste [-d] file1 file2
选项与参数：
    -d ：后面可以接分隔字符。预设是以 [tab] 来分隔的
    - ：如果 file 部分写成 - ，表示来自 standard input 的资料的意思。
```

#### expand
将 [tab] 按键转成空格键
```
[root@www ~]# expand [-t] file
选项与参数：
    -t ：后面可以接数字。一般来说，一个 tab 按键可以用 8 个空格键取代。我们也可以自行定义一个 [tab] 按键代表多少个字符
```

### 5、分割命令： split
```
[root@www ~]# split [-bl] file PREFIX
选项与参数：
    -b ：后面可接欲分割成的档案大小，可加单位，例如 b, k, m 等；
    -l ：以行数来进行分割。
    PREFIX ：代表前导符的意思，可作为分割档案的前导文字。
```

### 6、参数代换： xargs
```
[root@www ~]# xargs [-0epn] command
选项与参数：
    -0 ：如果输入的 stdin 含有特殊字符，例如 `, \, 空格键等等字符时，这个 -0 参数可以将他还原成一般字符。这个参数可以用于特殊状态
    -e ：这个是 EOF (end of file) 的意思。后面可以接一个字符串，当 xargs 分析到这个字符串时，就会停止继续工作
    -p ：在执行每个指令的 argument 时，都会询问使用者的意思；
    -n ：后面接次数，每次 command 指令执行时，要使用几个参数的意思。
当 xargs 后面没有接任何的指令时，默认是以 echo 来进行输出。
```
