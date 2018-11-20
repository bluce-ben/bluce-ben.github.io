---
title: Linux命令之uniq
date: 2018-11-20 17:21:20
categories:
- Linux
tags:
- Linux
- Linux命令
---
uniq - 检查及删除文本文件中重复出现的行列，一般与 sort 命令结合使用。
```
语法：
　　uniq [-cdu][-f<栏位>][-s<字符位置>][-w<字符位置>][--help][--version][输入文件][输出文件]
参数：
　　-c或--count 	在每列旁边显示该行重复出现的次数。
　　-d或--repeated 仅显示重复出现的行列。
　　-f<栏位>或--skip-fields=<栏位> 忽略比较指定的栏位。
　　-s<字符位置>或--skip-chars=<字符位置> 忽略比较指定的字符。
　　-u或--unique 仅显示出一次的行列。
　　-w<字符位置>或--check-chars=<字符位置> 指定要比较的字符。
　　--help 显示帮助。
　　--version 显示版本信息。
　　[输入文件] 指定已排序好的文本文件。如果不指定此项，则从标准读取数据；
　　[输出文件] 指定输出的文件。如果不指定此选项，则将内容显示到标准输出设备（显示终端）。
```
<!--more-->

**描述：**
* uniq 命令删除文件中的重复行。
* uniq 命令读取由 InFile 参数指定的标准输入或文件。该命令首先比较相邻的行，然后除去第二行和该行的后续副本。重复的行一定相邻。（在发出 uniq 命令之前，请使用 sort 命令使所有重复行相邻。）

**实例：**
testfile中的原有内容为：
```
$ cat testfile      #原有内容  
test 30  
test 30  
test 30  
Hello 95  
Hello 95  
Hello 95  
Hello 95  
Linux 85  
Linux 85 
```
使用uniq 命令删除重复的行后，有如下输出结果：
```
$ uniq testfile     #删除重复行后的内容  
test 30  
Hello 95  
Linux 85 
```
检查文件并删除文件中重复出现的行，并在行首显示该行重复出现的次数。使用如下命令：
```
$ uniq -c testfile      #删除重复行后的内容  
3 test 30             #前面的数字的意义为该行共出现了3次  
4 Hello 95            #前面的数字的意义为该行共出现了4次  
2 Linux 85            #前面的数字的意义为该行共出现了2次
```
<font color="red">（当重复的行并不相邻时，uniq 命令是不起作用的，即若文件内容为以下时，uniq 命令不起作用。必须辅以sort排序后再过滤。）</font>

