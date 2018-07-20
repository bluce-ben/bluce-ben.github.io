---
title: Linux命令之date、cal、bc
date: 2017-12-22 17:42:39
categories:
- 每日一个Linux命令
tags:
- Linux命令
---
### 1、显示日期的指令：date
如果在文字接口中想要知道目前Linux 系统的时间，那举就直接在指令列模式输入date 即可显示：
```
[bluce-ben@www ~]$ date
Fri Dec 22 16:06:43 CST 2017
上面显示的是：星期五, 12月22日, 16:06 分, 43 秒，在 2017 年的 CST 时区！
那如果我想要让这个程序显示出『2017/12/22』这样的日期显示方式呢？ 那就使用date 的格式化输出功能吧！
[bluce-ben@www ~]$ date +%Y/%m/%d
2017/12/22
[bluce-ben@www ~]$ date +%H:%M
16:09
```
<!--more-->
### 2、显示日历的指令：cal
```
[bluce-ben@www ~]$ cal
    December 2017
Su Mo Tu We Th Fr Sa 
                            1   2
3    4    5    6   7   8   9
10  11  12  13 14 15 16
17  18  19  20 21 22 23
24  25  26  27 28 29 30
31
```
cal (calendar)这个指令可以做的事情还很多，例如你可以显示整年的月历情况：`cal 2018`
基本上cal 这个指令可以接的语法为： `cal [month] [year]`

### 3、简单好用的计算器：bc
Linux提供了一支计算程序，就是bc。你在指令列输入bc 后，屏幕会显示出版本信息， 之后就进入到等待指示的阶段。如下所示：
```
[bluce-ben@www ~]$ bc
bc 1.06
Copyright 1991-1994, 1997, 1998, 2000 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'.
_ <==这个时候，光标会停留在这里等徃你的输入
```
常用运算符：
* \+ 加法
* \- 减法
* \* 乘法
* / 除法
* ^ 挃数
* % 余数

### 补充一些小Tips：
1. [Tab]：如果在command后按时，代表【命令补全】；如果在第二个字以后按，就变成【档案补齐】的功能了。
2. [Ctrl] + c：表示中断目前程序的按键。
3. [Ctrl] + d：表示【键盘输入结束】的意思，可以用来取代exit的输入。
4.  在文本模式下，你可以直接按下两个[Tab]按键，可以查看总共有多少指令可以使用。