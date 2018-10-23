---
title: Shell常用知识点总结
date: 2018-10-23 18:42:55
categories:
- Shell
tags:
- Shell
---
#### 1、整型变量自增的几种方法
```
a=1
a=$(($a+1))
a=$[$a+1]
a=`expr $a + 1`
let a++
let a+=1
((a++))
echo $a
```
<!--more-->
#### 2、读取文件中的内容
（1）直接读取对应文件
```
#!/bin/bash
while read line
do
      echo $line     #这里可根据实际用途变化
done < urfile
```

（2）使用管道符形式读取
```
#!/bin/bash
cat urfile | while read line
do
    echo $line
done
```
（注意：以上代码中urfile 为被读取的文件）

（3）将文件赋值给变量
```
#!/bin/bash
last | while read wOne wTwo wThree
do
	echo $wOne","$wTwo","$wThree
done
```

#### 3、判断文件是否为空
`if [[ ! -s filename ]]` # 如果文件存在且为空，-s代表存在不为空，！将它取反


#### 4、比较两个字符串的大小
<	小于，在ASCII字母顺序下，如：
	`if [[ "$a" < "$b" ]]`
	`if [ "$a" \< "$b" ]`
&#60;	大于，在ASCII字母顺序下，如：
	`if [[ "$a" > "$b" ]]`
	`if [ "$a" \> "$b" ]`
注意：在[]结构中">"需要被转义。
