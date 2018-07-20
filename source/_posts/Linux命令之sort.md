---
title: Linux命令之sort
date: 2018-03-30 15:50:27
categories:
- Linux
tags:
- Linux
- Linux命令
---
sort命令是在Linux里非常有用，它将文件进行排序，并将排序结果标准输出。sort命令既可以从特定的文件，也可以从stdin中获取输入。

#### 语法格式
```
[root@www ~]# sort [选项] [参数]
选项：
-n：依照数值的大小排序；
-r：以相反的顺序来排序；
-k：是指定需要爱排序的栏位；
-t<分隔字符>：指定排序时所用的栏位分隔字符；
-o<输出文件>：将排序后的结果存入制定的文件；（注：直接使用重定向会覆盖原文件，将其清空。）
参数：
文件：指定待排序的文件列表。
```
<!--more-->

#### 用法解析
##### sort的工作原理
sort将文件的每一行作为一个单位，相互比较，比较原则是从首字符向后，依次按ASCII码值进行比较，最后将他们按升序输出。
```
[root@www ~]$ cat seq.txt
banana
apple
pear
orange
pear
[root@www ~]$ sort seq.txt
apple
banana
orange
pear
pear
```

##### sort的-u选项
它的作用很简单，就是在输出行中去除重复行。
```
[root@www ~]$ sort -u seq.txt
apple
banana
orange
pear
```

##### sort的-r选项
sort默认的排序方式是升序，如果想改成降序，就加个-r就搞定了。
```
[root@www ~]$ sort -r seq.txt
pear
pear
orange
banana
apple
```

##### sort的-n选项
sort排序默认是采用ASCII码值进行比较来排序的，因此对比10与2的大小时，10就会比2小了，因为比较10与2的时候，真实比较的是1与2的大小，显然1小，所以就将10放在2的前面了。
对于此类情况，我们只要使用-n选项，就限制sort使用数值来进行排序。
```
[root@www ~]$ cat number.txt
1
10
19
11
2
[root@www ~]$ sort number.txt
1
10
11
19
2
5
[root@www ~]$ sort -n number.txt
1
2
5
10
11
19
```

##### sort的-o选项
由于sort默认是把结果输出到标准输出，所以需要用重定向才能将结果写入文件，形如sort filename > newfile。
但是，如果你想把排序结果输出到原文件中，用重定向可就不行了。会将原文件覆盖，情况数据。
对于这种情况，sort提供了-o选项解决了这个问题，可将结果写入原文件。
```
[root@www ~]$ sort -r number.txt > number.txt
[root@www ~]$ cat number.txt
[root@www ~]$
[root@www ~]$ cat number.txt
1
10
19
11
2
[root@www ~]$ sort -nr number.txt -o number.txt
[root@www ~]$ cat number.txt
19
11
10
5
2
1
```

##### sort的-t选项和-k选项
>如果有一个文件的内容是这样：
```
[root@www ~]$ cat seq.txt
banana:30:5.5
apple:10:2.5
pear:90:2.3
orange:20:3.4
```
这个文件有三列，列与列之间用冒号隔开了，第一列表示水果类型，第二列表示水果数量，第三列表示水果价格。
那么我想以水果数量来排序，也就是以第二列来排序，如何利用sort实现？

此处就可使用sort提供的-t选项，指定以什么间隔符进行分割。然后，再使用-k选项指定对那一列或几列进行排序即可。
```
[root@www ~]$ sort -n -k 2 -t : seq.txt
apple:10:2.5
orange:20:3.4
banana:30:5.5
pear:90:2.3
```

##### sort的-k选项进阶版
>例如有一个文件内容是这样的：
第一个域是公司名称，第二个域是公司人数，第三个域是员工平均工资。（仅作演示）
```
[root@www ~]$ cat facebook.txt
google 110 5000
baidu 100 5000
guge 50 3000
sohu 100 4500
```
（1）首先，按照公司人数排序，人数相同的按照员工平均工资的升序排序。
```
[root@www ~]$ sort -n -t ' ' -k 2 facebook.txt
guge 50 3000
baidu 100 5000
sohu 100 4500
google 110 5000
[root@www ~]$ sort -n -t ' ' -k 2 -k 3 facebook.txt
guge 50 3000
sohu 100 4500
baidu 100 5000
google 110 5000
```
（2）下面，按照员工平均工资的降序排序，如果员工工资相同，则按照工资人数升序排序。
```
[root@www ~]$ sort -t ' ' -n -k 3r -k 3 facebook.txt
baidu 100 5000
google 110 5000
sohu 100 4500
guge 50 3000
```