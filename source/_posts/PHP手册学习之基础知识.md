---
title: PHP手册学习之基础知识
date: 2019-07-19 16:59:10
categories:
- PHP
tags:
- PHP知识点
- 面试
---
#### PHP类型：支持9种原始数据类型。
##### 四种标量类型：
 boolean、integer、float(double)、string
##### 三种复合类型：
 array、object、callable(可调用)
##### 两种特殊类型：
 resource、NULL(无类型)

<!--more-->
#### 知识点：
1. 以下值被认为是false：
false、0、0.0、"0"、""、array()、NULL

2. 永远不要比较两个浮点数是否相等

3. 一个字符串的表达方式：单引号、双引号、heredoc语法结构、nowdoc语法结构

4. 在下列情况下一个变量被认为是NULL：被赋值为NULL；尚未被赋值；被unset()。
（注：使用unset()将一个变量转换为null将不会删除该变量或unset其值。仅是返回NULL值而已）

5. define()、const()定义常量，一个常量一旦被定义，就不能再改变或者取消定义。
 * 1）常量只能包含标量数据
 * 2）const()可以在类中定义常量

6. 常量和变量的不同点
 * 常量前面没有美元符号 $
 * 常量只能用define()函数定义，而不能通过赋值语句
 * 常量可以不用理会变量的作用域而在任何地方定义和访问
 * 常量一旦定义就不能被重新定义或者取消
 * 常量的值只能是标量。

7. isset()与empty()
isset()检测变量是否设置并且非NULL，注意，null字符并不等同于NULL常量。当变量被unset()之后，就变为NULL了。
empty()检测变量是否为空。当一个变量不存在，或者它的值等同于false，那么就会被被认为不存在。
以下的东西被认为是空的：
 ""、0、0.0、"0"、NULL、false、array()、$var(一个声明了，但是没有值的变量)
（注：不存在的变量，empty()检测也是为true。）


#### 其它：
1. 允许强制转换的有：
(int)、(bool)、(float)、(string)、(array)、(object)、(unset)
对应函数：intval()、strval()

2. 确定变量的类型：
gettype()、is\_array()、is\_bool()、is\_float()、is\_int()、is\_object()、is\_string()、is\_null()、is\_numeric()

3. 变量相关函数：
empty()、isset()、serialize()、unserialize()、unset()、var\_dump()、print\_r()、var\_export()


4. PHP预定义变量：
$\_GLOBALS
$\_SERVER
$\_GET
$\_POST
$\_REQUEST
$\_FILES
$\_SESSION
$\_COOKIE
$\_ENV 环境变量
$argc 传递给脚本的参数数目
$argv 传递给脚本的参数数组
$php\_errormsg 前一个错误信息
$HTTP\_RAW\_POST\_DATA 原声POST数据
$http\_response\_header HTTP响应头


5. 魔术常量：
\_\_LINE\_\_ 文件中当前行号
\_\_FILE\_\_ 文件的完整路径和文件名
\_\_DIR\_\_ 文件所在的目录
\_\_FUNCTION\_\_ 函数名称（区分大小写）
\_\_CLASS\_\_ 类的名称（区分大小写）
\_\_TRAIT\_\_ Trait的名字（区分大小写）
\_\_METHOD\_\_ 类的方法名（区分大小写）
\_\_NAMESPACE\_\_ 当前命名空间的名称（区分大小写）

6. 常见函数：
get\_class()、get\_object\_vars()、file\_exists()、function\_exists()
call\_user\_func - 把第一个参数作为回调函数调用
func\_get\_args - 返回一个包含函数参数列表的数组
