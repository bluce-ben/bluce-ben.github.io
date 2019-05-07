---
title: PHP错误处理机制
date: 2019-05-06 20:51:44
categories:
- PHP
tags:
- PHP
---
#### 错误简介 ####
  PHP提供了错误处理和日志记录的功能. 这些函数允许你定义自己的错误处理规则，以及修改错误记录的方式. 这样，你就可以根据自己的需要，来更改和加强错误输出信息以满足实际需要。
  常见的错误类型有：语法错误、环境错误、逻辑错误。平时遇到的warning、notice都是错误，只是级别不同而已。
  常见的错误级别有：Deprecated（最低级别）、Notice（通知）、Warning（警告）、Fatal（致命）、Parser（语法解析），E_USER_相关错误。

>注：在 PHP 中，默认的错误处理很简单。一条错误消息会被发送到浏览器，这条消息带有文件名、行号以及描述错误的消息。

<!--more-->
#### 错误处理函数（可自定义处理错误函数） ####
1. error_reporting  设置PHP的报错级别
1）通过php.ini设置如下： error_reporting = E_ALL
2）通过error_reporting()函数设置，如：
  error_reporting(0); //关闭所有PHP错误报告
  error_reporting(-1); //报告所有PHP错误
  error_reporting(E_ALL); //和error_reporting(-1)一样
3）通过ini_set()函数运行时设置
  `ini_set('error_reporting', E_ALL)`;

2. display_errors 设置是否将错误信息
1）在php.ini设置如下：display_errors = On 
（注：不管是On还是Off都会记录到你错误日志里面，前提是配置了错误日志log_errors和error_log）
2）通过`ini_set('display_errors', 1);` 设置运行时状态

3. set_error_handler  设置一个用户定义的错误处理函数
1）`set_error_handler('my_error');` //my_error()函数为自定义的错误处理方法
2）如果把自定义的错误封装到一个类上，则使用数组的方式调用：
  `set_error_handler(array('MyErrorHander', 'deal'));` //MyErrorHander为错误类，deal为处理方法
3）set_error_handler() 参数介绍如下：
![](/uploads/2019/05/set_error_handler.png "set_error_handler")

>注：(1)以下级别的错误不能由用户定义的函数来处理： E_ERROR、 E_PARSE、 E_CORE_ERROR、 E_CORE_WARNING、 E_COMPILE_ERROR、E_COMPILE_WARNING，和在调用 set_error_handler() 函数所在文件中产生的大多数 E_STRICT。 官方没有给出原因，但不难看出这些错误要么是运行时的致命错误，要么是php核心或编译时的错误，因此也不难猜想：对于运行时的致命错误，php直接中断，导致了错误处理函数没有机会执行。
(2)如果错误发生在脚本执行之前（比如文件上传时），将不会调用自定义的错误处理程序，因为它尚未在那时注册。


4. trigger_error()  产生一个用户级别的 error/warning/notice 信息

5. error_log —发送错误信息到某个地方
1）在配置文件中： error_log = E:\phpStudy\MyError\test_error.txt
2）运行时设置：`int_set('error_log', 'E:\phpStudy\MyError\test_error.txt')`;
3）使用error_log函数：`error_log("You messed up!", 3, "./error/my-errors.log")`;

6. error_get_last()  获取最后发生的错误
返回一个关联数组，描述了最后错误的信息，以该错误的“type”、“message”、“file”和“line”为数组的键。 如果该错误由PHP内置函数导致的，“message”会以该函数名开头。如果还没有错误则返回NULL。


#### 错误控制配置 ####
我们按照php+php-fpm的模型来说，会影响php错误显示的其实是有两个配置文件，一个是php本身的配置文件php.ini，另外一个是php-fpm的配置文件，php-fpm.conf。
##### 1）php.ini中的配置 #####
```
error_reporting = E_ALL  // 报告错误级别，什么级别的
error_log = /tmp/php_errors.log // php中的错误显示的日志位置
display_errors = On // 是否把错误展示在输出上，这个输出可能是页面，也可能是stdout
display_startup_errors = On // 是否把启动过程的错误信息显示在页面上，记得上面说的有几个Core类型的错误是启动时候发生的，这个就是控制这些错误是否显示页面的。
log_errors = On // 是否要记录错误日志
log_errors_max_len = 1024 // 错误日志的最大长度
ignore_repeated_errors = Off // 是否忽略重复的错误
track_errors = Off // 是否使用全局变量$php_errormsg来记录最后一个错误
xmlrpc_errors = 0 //是否使用XML-RPC的错误信息格式记录错误
xmlrpc_error_number = 0 // 用作 XML-RPC faultCode 元素的值。
html_errors = On  // 是否把输出中的函数等信息变为HTML链接
docref_root = http://manual/en/ // 如果html_errors开启了，这个链接的根路径是什么
fastcgi.logging = 0 // 是否把php错误抛出到fastcgi中
```

##### 2）php-fpm中的配置 #####
```
error_log = /var/log/php-fpm/error.log // php-fpm自身的日志
log_level = notice // php-fpm自身的日志记录级别
php_flag[display_errors] = off // 覆盖php.ini中的某个配置变量，可被程序中的ini_set覆盖
php_value[display_errors] = off // 同php_flag
php_admin_value[error_log] = /tmp/www-error.log // 覆盖php.ini中的某个配置变量，不可被程序中的ini_set覆盖
php_admin_flag[log_errors] = on // 同php_admin_value
catch_workers_output = yes // 是否抓取fpmworker的输出
request_slowlog_timeout = 0 // 慢日志时长
slowlog = /var/log/php-fpm/www-slow.log // 慢日志记录
```

php-fpm的配置中也有一个error_log配置，这个很经常会和php.ini中的error_log配置弄混。但他们记录的东西是不一样的，php-fpm的error_log只记录php-fpm本身的日志，比如fpm启动，关闭。
而php.ini中的error_log是记录php程序本身的错误日志。

那么在php-fpm中要覆盖php.ini中的error_log配置，就需要使用到下面几个函数：
* php_flag
* php_value
* php_admin_flag
* php_admin_value
这四个函数admin的两个函数说明这个变量设置完之后，不能在代码中使用ini_set把这个变量重新赋值了。而php_flag/value就仍然以php代码中的ini_set为准


#### 错误处理应该遵循的规则 ####
* 一定要让PHP报告错误；
* 在开发环境中要显示错误；
* 在生产环境中不能显示错误；
* 在开发和生产环境中都要记录错误。

参考博文：
1. [PHP错误机制总结](http://www.cnblogs.com/yjf512/p/5314345.html)
2. [PHP错误处理机制](https://www.jianshu.com/p/8752a7339022)
