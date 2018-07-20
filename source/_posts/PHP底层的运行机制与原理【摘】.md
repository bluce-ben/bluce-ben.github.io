---
title: PHP底层的运行机制与原理【摘】
date: 2017-12-11 16:27:41
categories:
- PHP
- PHP原理
tags:
- PHP原理
- PHP
---
## PHP简单运行过程：
　我们从未手动开启过PHP的相关进程，它是随着Apache的启动而运行的；
　PHP通过mod_php5.so模块和Apache相连（具体说来是SAPI，即服务器应用程序编程接口）；
　PHP总共有三个模块：内核、Zend引擎、以及扩展层；
　PHP内核用来处理请求、文件流、错误处理等相关操作；
　Zend引擎（ZE）用以将源文件转换成机器语言，然后在虚拟机上运行它；
　扩展层是一组函数、类库和流，PHP使用它们来执行一些特定的操作。比如，我们需要mysql扩展来连接MySQL数据库；
　当ZE执行程序时可能会需要连接若干扩展，这时ZE将控制权交给扩展，等处理完特定任务后再返还；
　最后，ZE将程序运行结果返回给PHP内核，它再将结果传送给SAPI层，最终输出到浏览器上。
<!--more-->

## PHP的执行流程&opcode
　我们先来看看PHP代码的执行所经过的流程。
![](/uploads/2017/12/php_opcode.jpg)
　从图上可以看到，PHP实现了一个典型的动态语言执行过程：拿到一段代码后，经过词法解析、语法解析等阶段后，源程序会被翻译成一个个指令(opcodes)，然后ZEND虚拟机顺次执行这些指令完成操作。PHP本身是用C实现的，因此最终调用的也都是C的函数，实际上，我们可以把PHP看做是一个C开发的软件。
　PHP的执行的核心是翻译出来的一条一条指令，也即opcode。
　Opcode是PHP程序执行的最基本单位。一个opcode由两个参数(op1,op2)、返回值和处理函数组成。PHP程序最终被翻译为一组opcode处理函数的顺序执行。


## PHP的四层体系
PHP的核心架构如下图：
![](/uploads/2017/12/php_core_framework.jpg)

从图上可以看出，PHP从下到上是一个4层体系：

* Zend引擎：Zend整体用纯C实现，是PHP的内核部分，它将PHP代码翻译（词法、语法解析等一系列编译过程）为可执行opcode的处理并实现相应的处理方法、实现了基本的数据结构（如hashtable、oo）、内存分配及管理、提供了相应的api方法供外部调用，是一切的核心，所有的外围功能均围绕Zend实现。
* Extensions：围绕着Zend引擎，extensions通过组件式的方式提供各种基础服务，我们常见的各种内置函数（如array系列）、标准库等都是通过extension来实现，用户也可以根据需要实现自己的extension以达到功能扩展、性能优化等目的（如贴吧正在使用的PHP中间层、富文本解析就是extension的典型应用）。
* Sapi：Sapi全称是Server Application Programming Interface，也就是服务端应用编程接口，Sapi通过一系列钩子函数，使得PHP可以和外围交互数据，这是PHP非常优雅和成功的一个设计，通过sapi成功的将PHP本身和上层应用解耦隔离，PHP可以不再考虑如何针对不同应用进行兼容，而应用本身也可以针对自己的特点实现不同的处理方式。
* 上层应用：这就是我们平时编写的PHP程序，通过不同的sapi方式得到各种各样的应用模式，如通过webserver实现web应用、在命令行下以脚本方式运行等等。

**构架思想：**
引擎(Zend)+组件(ext)的模式降低内部耦合
中间层(sapi)隔绝web server和PHP
**************************************************************************
如果php是一辆车，那么

车的框架就是php本身

Zend是车的引擎（发动机）

Ext下面的各种组件就是车的轮子

Sapi可以看做是公路，车可以跑在不同类型的公路上

而一次php程序的执行就是汽车跑在公路上。

因此，我们需要：性能优异的引擎+合适的车轮+正确的跑道


## Sapi
　如前所述，Sapi通过通过一系列的接口，使得外部应用可以和PHP交换数据并可以根据不同应用特点实现特定的处理方法，我们常见的一些sapi有：

* apache2handler：这是以apache作为webserver，采用mod_PHP模式运行时候的处理方式，也是现在应用最广泛的一种。
* cgi：这是webserver和PHP直接的另一种交互方式，也就是大名鼎鼎的fastcgi协议，在最近今年fastcgi+PHP得到越来越多的应用，也是异步webserver所唯一支持的方式。
* cli：命令行调用的应用模式

## LAMP架构
![](/uploads/2017/12/php_lamp_01.png)
从下往上四层：

1. liunx 属于操作系统的底层
2. apache服务器，属于次服务器，沟通linux和PHP
3. php:属于服务端编程语言，通过php_module 模块 和apache关联
4. mysql和其他web服务：属于应用服务，通过PHP的Extensions外 挂模块和mysql关联

**参考文章：**
[PHP的执行原理/执行流程](https://www.cnblogs.com/hongfei/archive/2012/06/12/2547119.html)
[PHP底层的运行机制与原理](http://www.nowamagic.net/librarys/veda/detail/102)