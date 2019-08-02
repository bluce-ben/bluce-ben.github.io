---
title: PHP安装yaf扩展
date: 2019-08-02 17:46:46
categories:
- PHP
tags:
- PHP
---
Yaf，全称 Yet Another Framework，是一个C语言编写的PHP框架，是一个用PHP扩展形式提供的PHP开发框架, 相比于一般的PHP框架, 它更快. 它提供了Bootstrap, 路由, 分发, 视图, 插件, 是一个全功能的PHP框架。最大特点就是简单、高效、快速，已经在百度和新浪微博经过大平台验证。

#### 一、下载
Git仓库：`https://github.com/laruence/yaf`

>资源整理：
1. yaf官方文档 ：`http://www.laruence.com/manual/`
2. yaf手册：`http://www.laruence.com/manual/index.html`

<!--more-->

#### 二、安装
依据git仓库中的说明，有两种方式可安装（也可使用其它方式）。
##### 1、Yaf是PECL的一个扩展，因此，可以直接使用PECL安装即可
`pecl install yaf`
（注：没有pecl命令，可先安装该命令。）
##### 2、编译安装（Linux）
```
/path/to/phpize
./configure --with-php-config=/path/to/php-config
Make & make install
```
>成功安装提示：
Build process completed successfully
Installing '/usr/local/lib/php/extensions/no-debug-non-zts-20170718/yaf.so'
install ok: channel://pecl.php.net/yaf-3.0.8
configuration option "php_ini" is not set to php.ini location
You should add "extension=yaf.so" to php.ini


##### 三、使用yaf框架
###### 1、启用yaf扩展
由上面“成功安装提示”可知道，下面需要配置php.ini，将yaf.so添加进PHP扩展中，重新启动PHP即可。
>下面是常用yaf配置选项：
```
[yaf]
yaf.environ = product
yaf.library = NULL
yaf.cache_config = 0
yaf.name_suffix = 1
yaf.name_separator = ""
yaf.forward_limit = 5
yaf.use_namespace = 0
yaf.use_spl_autoload = 0
```

##### 2、生成yaf框架代码
Yaf提供了代码生成工具yaf_code generator（在git项目中）, 所以也可以通过使用代码生成工具yaf_cg来完成这个简单的入门Demo（即yaf框架）。
`./yaf/tools/cg/yaf_cg sample`
会在 cg/output/目录下生成 sample文件夹，即yaf初版框架。将其粘贴到自己的项目中使用

##### 3、访问yaf框架
注：需要将'yaf_' 更改为'/yaf/'，才能识别出yaf的位置，否则会报找不到相应的类。
然后就可以访问成功了。
```
Hello World! I am Stranger
```

