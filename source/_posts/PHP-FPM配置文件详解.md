---
title: PHP-FPM配置文件详解
date: 2018-10-24 17:15:18
categories:
- 服务器
- PHP-FPM
tags:
- PHP-FPM
---
>过滤php-fpm.conf文件中注释行。
`grep -Env "^$|;" php-fpm.conf`
grep - 过滤命令
　　-E - 使用正则表达示进行匹配
　　-n - 显示行号
　　-v - 剔除匹配的项（默认是筛选匹配的项）
　　^ - 开头匹配
　　$ - 代表空行
　　| - 正则中的或运算
　　; - ;开头行

<!--more-->

#### 配置文件： ####
```
[www]
user = www	#进程的发起用户和用户组，用户user是必须设置，group不是
group = www
listen = 127.0.0.1:9000		#监听ip和端口

pm = dynamic	#选择进程池管理器如何控制子进程的数量
pm.max_children = 5
pm.start_servers = 2
pm.min_spare_servers = 1
pm.max_spare_servers = 3
 
slowlog = /opt/webserver/logs/php/$pool.log.slow 	#用于记录慢请求
request_slowlog_timeout = 3	#慢日志请求超时时间，对一个php程序进行跟踪。
;request_terminate_timeout = 0　　#终止请求超时时间，在worker进程被杀掉之后，提供单个请求的超时间隔。由于某种原因不停止脚本执行时，应该使用该选项，0表示关闭不启用
```

>static： 对于子进程的开启数路给定一个锁定的值(pm.max_children)
dynamic： 子进程的数目为动态的，它的数目基于下面的指令的值(以下为dynamic适用参数)
　　pm.max_children： 同一时刻能够存货的最大子进程的数量
　　pm.start_servers： 在启动时启动的子进程数量
　　pm.min_spare_servers： 处于空闲"idle"状态的最小子进程，如果空闲进程数量小于这个值，那么相应的子进程会被创建
　　pm.max_spare_servers： 最大空闲子进程数量，空闲子进程数量超过这个值，那么相应的子进程会被杀掉。
ondemand： 在启动时不会创建，只有当发起请求链接时才会创建(pm.max_children, pm.process_idle_timeout)


#### 总结： ####
在php-fpm的配置文件中，有两个指令非常重要，就是"pm.max_children" 和 "request_terminate_timeout"

（1）第一个指令"pm.max_children" 确定了php-fpm的处理能力，原则上时越多越好，但这个是在内存足够打的前提下，每开启一个php-fpm进程要占用近30M左右的内存
如果请求访问较多，那么可能会出现502，504错误。对于502错误来说，属于繁忙进程而造成的，对于504来说，就是客户发送的请求在限定的时间内没有得到相应，过多的请求导致“504  Gateway  Time-out”。这里也有可能是服务器带宽问题。

（2）另外一个需要注意的指令"request_terminate_timeout"，它决定php-fpm进程的连接/发送和读取的时间，如果设置过小很容易出现"502 Bad Gateway" 和 “504  Gateway  Time-out”，默认为0，就是说没有启用，不加限制，但是这种设置前提是你的php-fpm足够健康，这个需要根据实际情况加以限定


#### 附属PHP-FPM配置文件 ####
```
pid = run/php-fpm.pid
#pid设置，默认在安装目录中的var/run/php-fpm.pid，建议开启

error_log = log/php-fpm.log
#错误日志，默认在安装目录中的var/log/php-fpm.log

log_level = notice
#错误级别. 可用级别为: alert（必须立即处理）, error（错误情况）, warning（警告情况）, notice（一般重要信息）, debug（调试信息）. 默认: notice.

emergency_restart_threshold = 60
emergency_restart_interval = 60s
#表示在emergency_restart_interval所设值内出现SIGSEGV或者SIGBUS错误的php-cgi进程数如果超过 emergency_restart_threshold个，php-fpm就会优雅重启。这两个选项一般保持默认值。

process_control_timeout = 0
#设置子进程接受主进程复用信号的超时时间. 可用单位: s(秒), m(分), h(小时), 或者 d(天) 默认单位: s(秒). 默认值: 0.

daemonize = yes
#后台执行fpm,默认值为yes，如果为了调试可以改为no。在FPM中，可以使用不同的设置来运行多个进程池。 这些设置可以针对每个进程池单独设置。

listen = 127.0.0.1:9000
#fpm监听端口，即nginx中php处理的地址，一般默认值即可。可用格式为: 'ip:port', 'port', '/path/to/unix/socket'. 每个进程池都需要设置.

listen.backlog = -1
#backlog数，-1表示无限制，由操作系统决定，此行注释掉就行。backlog含义参考：http://www.3gyou.cc/?p=41

listen.allowed_clients = 127.0.0.1
#允许访问FastCGI进程的IP，设置any为不限制IP，如果要设置其他主机的nginx也能访问这台FPM进程，listen处要设置成本地可被访问的IP。默认值是any。每个地址是用逗号分隔. 如果没有设置或者为空，则允许任何服务器请求连接

listen.owner = www
listen.group = www
listen.mode = 0666
#unix socket设置选项，如果使用tcp方式访问，这里注释即可。

user = www
group = www
#启动进程的帐户和组

pm = dynamic #对于专用服务器，pm可以设置为static。
#如何控制子进程，选项有static和dynamic。如果选择static，则由pm.max_children指定固定的子进程数。如果选择dynamic，则由下开参数决定：
pm.max_children #，子进程最大数
pm.start_servers #，启动时的进程数
pm.min_spare_servers #，保证空闲进程数最小值，如果空闲进程小于此值，则创建新的子进程
pm.max_spare_servers #，保证空闲进程数最大值，如果空闲进程大于此值，此进行清理

pm.max_requests = 1000
#设置每个子进程重生之前服务的请求数. 对于可能存在内存泄漏的第三方模块来说是非常有用的. 如果设置为 '0' 则一直接受请求. 等同于 PHP_FCGI_MAX_REQUESTS 环境变量. 默认值: 0.

pm.status_path = /status
#FPM状态页面的网址. 如果没有设置, 则无法访问状态页面. 默认值: none. munin监控会使用到

ping.path = /ping
#FPM监控页面的ping网址. 如果没有设置, 则无法访问ping页面. 该页面用于外部检测FPM是否存活并且可以响应请求. 请注意必须以斜线开头 (/)。

ping.response = pong
#用于定义ping请求的返回相应. 返回为 HTTP 200 的 text/plain 格式文本. 默认值: pong.

request_terminate_timeout = 0
#设置单个请求的超时中止时间. 该选项可能会对php.ini设置中的'max_execution_time'因为某些特殊原因没有中止运行的脚本有用. 设置为 '0' 表示 'Off'.当经常出现502错误时可以尝试更改此选项。

request_slowlog_timeout = 10s
#当一个请求该设置的超时时间后，就会将对应的PHP调用堆栈信息完整写入到慢日志中. 设置为 '0' 表示 'Off'

slowlog = log/$pool.log.slow
#慢请求的记录日志,配合request_slowlog_timeout使用

rlimit_files = 1024
#设置文件打开描述符的rlimit限制. 默认值: 系统定义值默认可打开句柄是1024，可使用 ulimit -n查看，ulimit -n 2048修改。

rlimit_core = 0
#设置核心rlimit最大限制值. 可用值: 'unlimited' 、0或者正整数. 默认值: 系统定义值.

chroot =
#启动时的Chroot目录. 所定义的目录需要是绝对路径. 如果没有设置, 则chroot不被使用.

chdir =
#设置启动目录，启动时会自动Chdir到该目录. 所定义的目录需要是绝对路径. 默认值: 当前目录，或者/目录（chroot时）

catch_workers_output = yes
#重定向运行过程中的stdout和stderr到主要的错误日志文件中. 如果没有设置, stdout 和 stderr 将会根据FastCGI的规则被重定向到 /dev/null . 默认值: 空.
```
