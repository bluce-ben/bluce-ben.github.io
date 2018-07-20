---
title: Linux命令之sed
date: 2017-12-29 17:56:35
categories:
- Linux
tags:
- Linux
- Linux命令
---
sed 本身也是一个管线命令，可以分析 standard input 的，而且 sed 还可以将数据进行取代、删除、新增、撷取特定行等等的功能。我们先来了解一下 sed 的用法，再来聊他的用途。
<!--more-->
```
[root@www ~]# sed [-nefr] [动作]
选项与参数：
    -n ：使用安静(silent)模式。在一般 sed 的用法中，所有来自 STDIN 的数据一般都会被列出到屏幕上。
         但如果加上 -n 参数后，则只有经过 sed 特殊处理的那一行(或者动作)才会被列出来。
    -e ：直接在指令列模式上进行 sed 的动作编辑；
    -f ：直接将 sed 的动作写在一个档案内，-f filename 则可以执行 filename 内的 sed 动作；
    -r ：sed 的动作支持的是延伸型正规表示法的语法。(预设是基础正规表示法语法)
    -i ：直接修改读取的档案内容，而不是由屏幕输出。

动作说明： [n1[,n2]]function
n1, n2 ：不见得会存在，一般代表【选择进行动作的行数】，
        举例来说，如果我的动作是需要在 10 到 20 行之间进行的，则【 10,20[动作行为] 】
function 有底下这些咚咚：
    a ：新增， a 的后面可以接字符串，而这些字符串会在新的一行出现(目前的下一行)
    c ：取代， c 的后面可以接字符串，这些字符串可以取代 n1,n2 之间的行！
    d ：删除，因为是删除，所以 d 后面通常不接任何咚咚；
    i ：插入， i 的后面可以接字符串，而这些字符串会在新的一行出现(目前的上一行)；
    p ：打印，亦即将某个选择的数据打印出。通常 p 会与参数 sed -n 一起运作
    s ：取代，可以直接进行取代的工作。通常这个 s 的动作可以搭配正规表示法。例如 1,20s/old/new/g 就是
```

下面进行一些示例：

#### 1、以行为单位的新增/删除功能
##### 范例一：将 /etc/passwd 的内容列出并且打印行号，同时，请将第 2~5 行删除
```
[root@www ~]# nl /etc/passwd | sed '2,5d'
1 root:x:0:0:root:/root:/bin/bash
6 sync:x:5:0:sync:/sbin:/bin/sync
7 shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
.....(后面省略).....
```
sed 的动作为 '2,5d' ，那个 d 就是删除。因为 2-5 行给他删除了，所以显示的数据就没有 2-5 行了。另外，注意一下，原本应该是要下达 sed -e 才对，没有 -e 也行。同时也要注意的是，sed 后面接的动作，请务必以 '' 两个单引号括住。

如果题型变化一下，举例来说，如果只要删除第 2 行，可以使用`【 nl /etc/passwd | sed '2d' 】`来达成，至于若是要删除第 3 到最后一行，则是`【 nl /etc/passwd | sed '3,$d' 】`，那个钱字号【 $ 】代表最后一行。

##### 范例二：承上题，在第二行后(亦即是加在第三行)加上【drink tea?】字样
```
[root@www ~]# nl /etc/passwd | sed '2a drink tea'
1 root:x:0:0:root:/root:/bin/bash
2 bin:x:1:1:bin:/bin:/sbin/nologin
drink tea
3 daemon:x:2:2:daemon:/sbin:/sbin/nologin
.....(后面省略).....
```
在 a 后面加上的字符串就已将出现在第二行后面。那如果是要在第二行前呢？`【 nl /etc/passwd | sed '2i drink tea' 】`就对了。就是将【 a 】变成【 i 】即可。增加一行很简单，那如果是要增将两行以上呢？

##### 范例三：在第二行后面加入两行字，例如【Drink tea or .....】与【drink beer?】
```
[root@www ~]# nl /etc/passwd | sed '2a Drink tea or ......\
> drink beer ?'
1 root:x:0:0:root:/root:/bin/bash
2 bin:x:1:1:bin:/bin:/sbin/nologin
Drink tea or ......
drink beer ?
3 daemon:x:2:2:daemon:/sbin:/sbin/nologin
.....(后面省略).....
```
这个范例的重点是【我们可以新增不只一行，可以新增好几行】但是每一行之间都必须要以反斜杠【 \ 】来进行新行的增加。所以，上面的例子中，我们可以发现在第一行的最后面就有 \ 存在，那是一定要的。

#### 2、以行为单位的取代与显示功能
##### 范例四：我想将第 2-5 行的内容取代成为【No 2-5 number】
```
[root@www ~]# nl /etc/passwd | sed '2,5c No 2-5 number'
1 root:x:0:0:root:/root:/bin/bash
No 2-5 number
6 sync:x:5:0:sync:/sbin:/bin/sync
.....(后面省略).....
```
透过这个方法我们就能够将数据整行取代了，非常容易吧，sed 还有更好用的。我们以前想要列出第 11~20 行，得要透过`【head -n 20 | tail -n 10】`之类的方法来处理，很麻烦。sed 则可以简单的直接取出你想要的那几行，是透过行号来的。看看底下的范例：
##### 范例五：仅列出 /etc/passwd 档案内的第 5-7 行
```
[root@www ~]# nl /etc/passwd | sed -n '5,7p'
5 lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
6 sync:x:5:0:sync:/sbin:/bin/sync
7 shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
```
上述的指令中有个重要的选项【 -n 】，按照说明文件，这个 -n 代表的是【安静模式】。那么为什么要使用安静模式呢？你可以自行下达 sed '5,7p' 就知道了 (5-7 行会重复输出)。有没有加上 -n 的参数时，输出的数据可是差很多的。你可以透过这个 sed 的以行为单位的显示功能，就能够将某一个档案内的某些行号捉出来查阅。很棒的功能！

#### 3、部分数据的搜寻并取代的功能
除了整行的处理模式之外，sed 还可以用行为单位进行部分数据的搜寻并取代的功能。基本上 sed 的搜寻与取代的与 vi 相当的类似！他有点像这样：
`sed 's/要被取代的字符串/新的字符串/g'`

我们使用底下这个取得 ifconfig 中 IP 数据的范例来了解一下什么是咱们所谓的搜寻并取代：
```
步骤一：先观察原始讯息，利用 /sbin/ifconfig 查询 IP 为何？
[root@www ~]# /sbin/ifconfig eth0
eth0 Link encap:Ethernet HWaddr 00:90:CC:A6:34:84
inet addr:192.168.1.100 Bcast:192.168.1.255 Mask:255.255.255.0
inet6 addr: fe80::290:ccff:fea6:3484/64 Scope:Link
UP BROADCAST RUNNING MULTICAST MTU:1500 Metric:1
.....(以下省略).....
# 我们的重点在第二行，也就是 192.168.1.100 这一行。先利用关键词捉出那一行！

步骤二：利用关键词配合 grep 撷取出关键的一行数据
[root@www ~]# /sbin/ifconfig eth0 | grep 'inet addr'
inet addr:192.168.1.100 Bcast:192.168.1.255 Mask:255.255.255.0
# 当场仅剩下一行，接下来，我们要将开始到 addr: 通通删除，就是像底下这样：
# inet addr:192.168.1.100 Bcast:192.168.1.255 Mask:255.255.255.0
# 上面的删除关键在于【 ^.*inet addr: 】。正规表示法出现！

步骤三：将 IP 前面的部分予以删除
[root@www ~]# /sbin/ifconfig eth0 | grep 'inet addr' | \
> sed 's/^.*addr://g'
192.168.1.100 Bcast:192.168.1.255 Mask:255.255.255.0
# 仔细与上个步骤比较一下，前面的部分不见了！接下来则是删除后续的部分，亦即：
# 192.168.1.100 Bcast:192.168.1.255 Mask:255.255.255.0
# 此时所需的正规表示法为：【 Bcast.*$ 】

步骤四：将 IP 后面的部分予以删除
[root@www ~]# /sbin/ifconfig eth0 | grep 'inet addr' | \
> sed 's/^.*addr://g' | sed 's/Bcast.*$//g'
192.168.1.100
```

#### 4、直接修改档案内容(危险动作)
sed 甚至可以直接修改档案的内容。而不必使用管线命令或数据流重导向。不过，由于这个动作会直接修改到原始的档案，所以千万不要随便拿系统配置文件来测试，可以使用测试档案 regular_express.txt 来测试看看。
##### 范例六：利用 sed 将 regular_express.txt 内每一行结尾若为 . 则换成 !
```
[root@www ~]# sed -i 's/\.$/\!/g' regular_express.txt
# 上头的 -i 选项可以让你的 sed 直接去修改后面接的档案内容而不是由屏幕输出
# 这个范例是用在取代，可自行 cat 该档案去查阅结果
```
##### 范例七：利用 sed 直接在 regular_express.txt 最后一行加入【# This is a test】
```
[root@www ~]# sed -i '$a # This is a test' regular_express.txt
# 由于 $ 代表的是最后一行，而 a 的动作是新增，因此该档案是最后新增
```
sed 的【 -i 】选项可以直接修改档案内容，这功能非常有帮助。举例来说，如果你有一个 100 万行的档案，你要在第 100 行加某些文字，此时使用 vim 可能会疯掉！因为档案太大了！那怎办？就利用 sed 啊！透过 sed 直接修改/取代的功能，你甚至不需要使用 vim 去修订。