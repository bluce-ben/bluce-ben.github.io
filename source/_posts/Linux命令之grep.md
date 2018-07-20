---
title: Linux命令之grep
date: 2017-12-12 18:27:29
categories:
- Linux
tags:
- Linux
- Linux命令
---
　grep命令文件过滤分割与合并 grep（global search regular expression(RE) and print out the line，全面搜索正则表达式并把行打印出来）是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来。
<!--more-->
**常用选项：**
-n	在显示符合范本样式的那一列之前，标示出该列的编号。
-v	反转查找。
-i	忽略字符大小写的差别。

**常见用法：**
1、在文件中搜索一个单词，命令会返回一个包含“match_pattern”的文本行：
grep match_pattern file_name 
grep "match_pattern" file_name

2、在多个文件中查找：
grep "match_pattern" file_1 file_2 file_3 ...
grep "match_pattern" ./*.php

3、输出除之外的所有行 -v 选项：
grep -v "match_pattern" file_name

4、输出包含匹配字符串的行数 -n 选项：
grep "text" -n file_name 
或 
cat file_name | grep "text" -n 
#多个文件 
grep "text" -n file_1 file_2

5、忽略匹配样式中的字符大小写： 
echo "hello world" | grep -i "HELLO" 
hello

**grep的一些进阶选项：**
```
[root@www ~]# grep [-A] [-B] [--color=auto] '搜寻字符串' filename
选项与参数：
-A ：后面可加数字，为 after 的意思，除了列出该行外，后续的 n 行也列出来；
-B ：后面可加数字，为 befer 的意思，除了列出该行外，前面的 n 行也列出来；
--color=auto 可将正确癿那个撷叏数据列出颜色
```
范例一：要将捉到的关键词显色，且加上行号来显示：
```
[root@www ~]# dmesg | grep -n --color=auto 'eth'
247:eth0: RealTek RTL8139 at 0xee846000, 00:90:cc:a6:34:84, IRQ 10
248:eth0: Identified 8139 chip type 'RTL-8139C'
294:eth0: link up, 100Mbps, full-duplex, lpa 0xC5E1
305:eth0: no IPv6 routers present
# 你会发现除了 eth 会有特殊颜色来表示之外，最前面还有行号显示
```
范例二：承上题，在关键词所在行的前两行与后三行也一起捉出来显示
```
[root@www ~]# dmesg | grep -n -A3 -B2 --color=auto 'eth'
245-PCI: setting IRQ 10 as level-triggered
246-ACPI: PCI Interrupt 0000:00:0e.0[A] -> Link [LNKB] ...
247:eth0: RealTek RTL8139 at 0xee846000, 00:90:cc:a6:34:84, IRQ 10
248:eth0: Identified 8139 chip type 'RTL-8139C'
249-input: PC Speaker as /class/input/input2
250-ACPI: PCI Interrupt 0000:00:01.4[B] -> Link [LNKB] ...
251-hdb: ATAPI 48X DVD-ROM DVD-R-RAM CD-R/RW drive, 2048kB
Cache, UDMA(66)
# 如上所示，你会发现关键词 247 所在的前两行及 248 后三行也都被显示出来！
# 这样可以让你将关键词前后数据捉出来进行分析
```
grep 是一个很常见也很常用的指令，他最重要的功能就是进行字符串数据的比对，然后将符合用户需求的字符串打印出来。需要说明的是【grep 在数据中查寻一个字符串时，是以 "整行" 为单位来进行数据的撷取的！】也就是说，假如一个档案内有 10行，其中有两行具有你所搜寻的字符串，则将那两行显示在屏幕上，其他的就丢弃了。
**（注：在关键词的显示方面，grep 可以使用 `--color=auto` 来将关键词部分使用颜色显示。如果感觉每次使用 grep 都得要自行加上 `--color=auto` 很麻烦，此时就可以使用 alias 来处理一下，你可以在 ~/.bashrc 内加上这行：`【alias grep='grep --color=auto'】`再以`【 source ~/.bashrc 】`来立即生效即可使用。这样每次执行 grep 都会自动帮你加上颜色显示。）**