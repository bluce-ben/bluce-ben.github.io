---
title: PHP错误日志简单配置
date: 2019-01-08 16:23:42
categories:
- PHP
tags:
- PHP
---
php.ini
```
; 错误日志
log_errors = On
; 显示错误
display_errors = Off
; 日志路径
error_log = "/usr/local/lnmp/php/var/log/error_log"
; 错误等级
error_reporting = E_ALL&~E_NOTICE
```
<!--more-->
php-fpm.conf
```
[global]
; php-fpm pid文件
pid = /usr/local/php/var/run/php-fpm.pid
; php-fpm 错误日志路径
error_log = /usr/local/php/var/log/php-fpm.log
; php-fpm 记录错误日志等级
log_level = notice
[www]
; 记录错误到php-fpm的日志中
;catch_workers_output = yes
; 慢日志
slowlog = var/log/slow.log
; 关闭打印日志
php_flag[display_errors] = off
; 错误日志
php_admin_value[error_log] = /usr/local/php/var/log/www.log
; 记录错误
php_admin_flag[log_errors] = on
; 内存使用量
php_admin_value[memory_limit] = 32M
```
注：如果错误没有写入到文件，查看网站用户对`php_admin_value[error_log]`的路径是否有写入权限。
