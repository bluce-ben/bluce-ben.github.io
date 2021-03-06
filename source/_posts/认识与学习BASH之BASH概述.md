---
title: 认识与学习BASH之BASH概述
date: 2017-12-28 19:10:35
categories:
- 书籍
- 《鸟哥的Linux私房菜基础篇（第三版）》
tags:
- 书籍
- BASH
- Shell
---
为什么BASH叫做壳程序？ 理解这个就理解了BASH与操作系统的关系，以及BASH是什么，有什么用。

### 1、认识BASH这个Shell
管理整个计算机硬件的其实是操作系统的核心 (kernel)，这个核心是需要被保护的。所以我们一般使用者就只能透过 shell 来跟核心沟通，以让核心达到我们所想要达到的工作。那么系统有多少 shell 可用呢？为什么我们要使用 bash啊？底下分别来谈一谈。
<!--more-->
### 2、硬件、核心 与 Shell
在认识 Shell 之前，我们先来了解一下计算机的运作状况吧。举个例子来说：当你要计算机传输出来【音乐】的时候，你的计算机需要什么东西呢？
	
	1. 硬件：当然就是需要你的硬件有【声卡芯片】这个配备，否则怎么会有声音；
	2. 核心管理：操作系统的核心可以支持这个芯片组，当然还需要提供芯片的驱动程序；
	3. 应用程序：需要使用者 (就是你) 输入发生声音的指令。
这就是基本的一个输出声音所需要的步骤。也就是说，你必须要【输入】一个指令之后，【硬件】才会透过你下达的指令来工作。那么硬件如何知道你下达的指令呢？那就是 kernel (核心) 的控制工作了，也就是说，我们必须要透过【Shell】将我们输入的指令与 Kernel 沟通，好让 Kernel 可以控制硬件来正确无误的工作。

曾经提到过，操作系统其实是一组软件，由于这组软件在控制整个硬件与管理系统的活动监测，如果这组软件能被用户随意的操作，若使用者应用不当，将会使得整个系统崩溃！因为操作系统管理的就是整个硬件功能，所以当然不能够随便被一些没有管理能力的终端用户随意使用。但是我们总是需要让用户操作系统的，所以就有了在操作系统上面发展的应用程序。用户可以透过应用程序来指挥核心，让核心达成我们所需要的硬件任务！

我们可以发现应用程序其实是在最外局，就如同鸡蛋的外壳一样，因此这个咚咚也就被称呼为<font color="red">壳程序 (shell)</font>。其实壳程序的功能只是提供用户操作系统的一个接口，因此这个壳程序需要可以呼叫其他软件才好。我们知道有很多指令，例如 man, chmod, chown, vi, fdisk, mkfs 等指令，这些指令都是独立的应用程序，但是我们可以透过壳程序 (就是指令列模式)来操作这些应用程序，让这些应用程序呼叫核心来运作所需的工作。这样对于壳程序是否有了一定的概念了。
> 也就是说，只要能够操作应用程序的接口都能够称为壳程序。狭义的壳程序指的是指令列方面的软件，包括本章要介绍的 bash 等。广义的壳程序则包括图形接口的软件，因为图形接口其实也能够操作各种应用程序来呼叫核心工作。


### 3、系统合法的shell与/etc/shells功能
知道什么是 Shell 之后，那么我们来了解一下 Linux 使用的是哪一个 shell 。由于早年的 Unix 年代，发展者众，所以由于 shell 依据发展者的不同就有很多的版本，例如常听到的 Bourne SHell (sh) 、在 Sun 里头预设的 C SHell、 商业上常用的 K SHell、还有 TCSH 等等，每一种 Shell 都各有其特点。至于 Linux 使用的这一种版本就称为【Bourne Again SHell (简称 bash) 】，这个 Shell 是 Bourne Shell 的增强版本，也是基准于GNU 的架构下发展出来的。

在介绍 shell 的优点之前，先来说一说 shell 的简单历史：第一个流行的 shell 是由 Steven Bourne 发展出来的，为了纪念他所以就称为 Bourne shell ，或直接简称为 sh 。而后来另一个广为流传的 shell 是由柏克莱大学的 Bill Joy 设计依附于 BSD 版的 Unix 系统中的 shell ，这个 shell 的语法有点类似 C 语言，所以才得名为 C shell ，简称为 csh 。由于在学术界 Sun 主机势力相当的庞大，而Sun 主要是 BSD 的分支之一，所以 C shell 也是另一个很重要而且流传很广的 shell 之一 。
> 由于 Linux 为 C 程序语言撰写的，很多程序设计师使用 C 来开发软件，因此 C shell相对的就很热门了。Sun 公司的创始人就是 Bill Joy，而 BSD 最早就是 Bill Joy 发展出来的。

那么目前我们的 Linux 有多少我们可以使用的 shells 呢？ 你可以检查一下/etc/shells 这个档案，至少就有底下这几个可以用的 shells：
```
	/bin/sh (已经被 /bin/bash 所取代)
	/bin/bash (就是 Linux 预设的 shell)
	/bin/ksh (Kornshell 由 AT&T Bell lab. 发展出来的，兼容于 bash)
	/bin/tcsh (整合 C Shell ，提供更多的功能)
	/bin/csh (已经被 /bin/tcsh 所取代)
	/bin/zsh (基于 ksh 发展出来的，功能更强大的 shell)
```
虽然各家 shell 的功能都差不多，但是在某些语法的下达方面则有所不同，因此建议你还是选择某一种 shell 来熟悉一下较佳。 Linux 预设就是使用 bash ，所以最初你只要学会 bash 就ok了。为什么我们系统上合法的 shell 要写入 /etc/shells 这个档案呢？ 这是因为系统某些服务在运作过程中，会去检查使用者能够使用的 shells ，而这些 shell 的查询就是藉由 /etc/shells 这个档案。

那么，我这个使用者取得shell工作后，预设会取得哪一个 shell 呢？当我登入的时候，系统就会给我一个 shell 让我来工作了。而这个登入取得的 shell 就记录在 /etc/passwd 这个档案内。这个档案的内容是啥？
```
[root@www ~]# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
.....(底下省略).....
```
如上所示，在每一行的最后一个数据，就是你登入后取得预设的 shell 。

### 4、Bash shell 的功能
既然 /bin/bash 是 Linux 预设的 shell，那么总是得了解一下这个玩意儿。bash 是 GNU 计划中重要的工具软件之一，目前也是 Linux distributions 的标准 shell 。 bash 主要兼容于 sh ，并且依据一些使用者的需求，而加强的 shell 版本。不论你使用的是那个 distribution ，你都难逃需要学习 bash 的宿命。那么这个 shell 有什么好处，干嘛 Linux 要使用他作为预设的 shell 呢？ bash 主要的优点有底下几个：

**（1）命令编修能力 (history)：**
就是记忆使用过的指令，只要在指令列按【上下键】就可以找到前/后一个输入的指令。而在很多 distribution 里头，默认的指令记忆功能可以多达 1000 个。也就是说，你曾经下达过的指令几乎都被记录下来了。

这么多的指令记录在哪里呢？在你的家目录内的 .bash_history。不过，需要留意的是，~/.bash_history 记录的是前一次登入以前所执行过的指令，而至于这一次登入所执行的指令都被暂存在内存中，当你成功的注销系统后，该指令记忆才会记录到 .bash_history 当中。

**（2）命令与档案补全功能： ([tab] 按键的好处)**
常常在 bash 环境中使用 [tab] 是个很棒的习惯。因为至少可以让你 1)少打很多字； 2)确定输入的数据是正确的。使用 [tab] 按键的时机依据 [tab] 接在指令后或参数后而有所不同。
如下所示：

	1. [Tab] 接在一串指令的第一个字的后面，则为命令补全；
	2. [Tab] 接在一串指令的第二个字以后时，则为【档案补齐】。
所以说，如果我想要知道我的环境中，所有可以执行的指令有几个？就直接在 bash 的提示字符后面连续按两次 [tab] 按键就能够显示所有的可执行指令了。 那如果想要知道系统当中所有以 c 为开头的指令呢？就按下【c[tab][tab]】就好了。

**（3）命令别名设定功能： (alias)**
假如我需要知道这个目录底下的所有档案 (包含隐藏档) 及所有的文件属性，那么我就必须要下达【ls -al】这样的指令串。如果使用命令别名，就可以自定义命令来取代长指令串命令，也就是说使用 alias 即可。你可以在指令列输入 alias 就可以知道目前的命令别名有哪些了。也可以直接下达命令来设定别名：
	`alias lm='ls -al'`

**（4）工作控制、前景背景控制： (job control, foreground, background)**
使用前、背景的控制可以让工作进行的更为顺利。至于工作控制(jobs)的用途则更广，可以让我们随时将工作丢到背景中执行。而不怕不小心使用了[Ctrl] + c 来停掉该程序。此外，也可以在单一登录的环境中，达到多任务的目的。

**（5）程序化脚本： (shell scripts)**
在 Linux 底下的 shell scripts 可以将你平时管理系统常需要下达的连续指令写成一个档案，该档案并且可以透过对谈交互式的方式来进行主机的侦测工作。也可以藉由 shell 提供的环境变量及相关指令来进行设计，整个设计下来几乎就是一个小型的程序语言。（该部分会单独详解）

**（6）通配符： (Wildcard)**
除了完整的字符串之外，bash 还支持很多的通配符来帮助用户查询与指令下达。举例来说，想要知道/usr/bin 底下有多少以 X 为开头的档案，使用：【ls -l /usr/bin/X*】就能够知道。

### 5、Bash shell 的内建命令： type
如何知道这个指令是来自于外部指令(指的是其他非 bash 所提供的指令) 或是内建在 bash 当中的，利用 type 这个指令来观察即可。
```
语法：`type [-tpa] name`
选项与参数：
	   ：不加任何选项与参数时，type 会显示出 name 是外部指令还是 bash 内建指令
	-t ：当加入 -t 参数时，type 会将 name 以底下这些字眼显示出他的意义：
		file ：表示为外部指令；
		alias ：表示该指令为命令别名所设定的名称；
		builtin ：表示该指令为 bash 内建的指令功能；
	-p ：如果后面接的 name 为外部指令时，才会显示完整文件名；
	-a ：会由 PATH 变量定义的路径中，将所有含 name 的指令都列出来，包含alias
```
