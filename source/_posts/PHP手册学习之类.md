---
title: PHP手册学习之类
date: 2019-07-19 17:00:56
categories:
- PHP
tags:
- PHP知识点
- 面试
---
#### 类
PHP5中的新特性包括：访问控制、抽象类、final类与方法，附加的魔术方法、接口、对象复制和类型约束。

##### 基本概念：
1. 一个类可以包含有属于自己的常量、变量（称为“属性”）以及函数（称为“方法”）。
2. 一个类可以在声明中用 extends 关键字继承另一个类的方法和属性。PHP不支持多重继承。
3. 访问控制关键字 public、protected、private、static、final
4. 静态成员、类常量访问使用“::“
5. self,parent和static这是那个特殊的关键字是用于在类定义的内部对其属性或方法进行访问的。
6. 对象比较
 `==` 比较两个对象时，如果两个对象的属性和属性值 都相等，而且两个对象是同一个类的实例，那么这两个对象变量相等。
 `===` 使用全等运算符，这两个对象变量一定要指向某个类的同一个实例（即同一个对象）。

<!--more-->

##### static
1. 声明类属性或方法为静态，就可以不实例化类而直接访问。静态属性不能通过一个已实例化的类对象来访问（但静态方法可以）。
2. 由于静态方法不需要通过对象即可调用，所以伪变量 $this在静态方法中不可用。
3. 静态属性不可以由对象通过 -> 操作符来访问。
4. 用静态方式调用一个非静态方法会导致一个 E_STRICT级别的错误


##### 接口
1. 通过关键字 interface来定义，通过implements关键字来实现接口。（类可以实现多个接口）
2. 接口定义的所有方法都必须是公有的，这是接口的特性。
3. 类中必须实现接口定义的所有方法，否则会报一个致命错误


##### Trait
1. Trait是为类似PHP的单继承语言而准备的一种代码复用机制。利用组合的方式来复用代码。
2. 优先级问题：
 从基类继承的成员会被trait插入的成员所覆盖。优先顺序是来自当前类的成员覆盖了trait的方法，而trait则覆盖了被继承的方法。
3. 多个trait，通过都好分隔，在use声明列出多个trait，可以都插入到一个类中。


##### 重载（类魔术方法）
###### 属性重载
\_\_set() 在给不可访问属性赋值时，会调用
\_\_get() 读取不可访问属性的值时，会调用
\_\_isset() 当对不可访问属性调用isset()或empty()时，会调用
\_\_unset() 当对不可访问属性调用unset()时，会调用
###### 方法重载
\_\_call() 在对象中调用一个不可访问方法时，会调用
\_\_callStatic() 在静态上下文中调用一个不可访问方法时，会调用


##### 魔术方法
\_\_construct()、\_\_destruct()、\_\_call()、\_\_callStatic()、
\_\_get()、\_\_set()、\_\_isset()、\_\_unset()、\_\_sleep()、\_\_wakeup()、
\_\_toString()、\_\_invoke()、\_\_clone()、\_\_set_state()、\_\_debugInfo()
\_\_toString() 用于当一个类被当成字符串时应该怎么回应。例如echo $obj;应该显示什么
\_\_invoke() 当尝试以调用函数的方式调用一个对象时，该方法会自动调用。
\_\_clone() 当克隆对象的时候，会调用该方法。


##### 类型约束（PHP5可以使用）
1. 函数的参数可以指定必须为对象（在函数原型里面指定类的名字），接口，数组或者callable
2. 如果一个类或接口指定了类型约束，则其所有的子类或者实现也都如此。


##### 类/对象函数
\_\_autoload	- 尝试加载为定义的类 sql\_autoload\_register()
class\_exists	- 检查类是否已定义
method\_exists	- 检查类的方法是否存在
trait\_exists	- 检查指定的trait是否存在
property\_exists - 检查对象或类是否具有该属性
get\_class	- 返回对象的类名
is\_a		- 如果对象属于该类或该类是此对象的父类则返回TRUE





#### 其它
##### 一、错误处理和日志记录
1、预定义常量
E\_ERROR		致命的运行时错误
E\_WARNING	运行时警告
E\_PARSE		编译时语法解析错误
E\_NOTICE	运行时通知
E\_STRICT	启用PHP对代码的修改建议，以确保代码具有最佳的互操作性和向前兼容性
E\_ALL		除E_STRICT外所有错误和警告信息

2、运行时配置项： 可通过ini\_set()函数设置运行时配置
error\_reporting		设置错误报告的级别。运行时可直接使用error\_reporting()函数设置
 PHP5.3及以上版本中，默认值为E\_ALL & ~E\_NOTICE & ~E\_STRICT & ~E\_DEPRECATED
该设置不会显示E\_NOTICE、E\_STRICT、E\_DEPRECATED级错误提示。
display\_errors		该选项设置是否将错误信息作为输出的一部分显示到屏幕，或者对用户隐藏而不显示。
log\_errors		设置是否将脚本运行的错误信息记录到服务器错误日志或者error\_log之中。
error\_log		设置脚本错误将被记录到文件。
ignore\_repeated\_errors	不记录重复的信息

3、错误处理函数：
debug\_backtrace 产生一条回溯信息
debug\_print\_backtrace 打印一条回溯信息
error\_clear\_last 清楚最近一次错误
error\_get\_last	获取最后发生的错误
error\_log	发送错误信息到某个地方
error\_reporting	设置应该报告何种PHP错误
set\_error\_handler	设置用户自定义的错误处理函数
set\_exception\_handler	设置用户自定义的异常处理函数


##### 二、PHP选项/信息函数
set\_time\_limit	- 设置脚本最大执行时间
ini\_set		- 为一个配置选项设置值
getenv		- 获取一个环境变量的值
getlastmod	- 获取页面最后修改的时间
ini\_get		- 获取一个配置选项的值
memory\_get\_usage 	- 返回分配给PHP的内存量
php\_ini\_loaded\_file	- 取得已加载的php.ini文件的路径
php\_uname	- 返回运行PHP的系统的有关信息
phpinfo		- 输出关于PHP配置的信息
phpversion	- 获取当前的PHP版本
putenv		- 设置环境变量的值



##### 三、扩展：
**压缩与归档扩展：**
 Bzip2		对.bz2的压缩文件
 Rar
 Zip
 Zlib		对gzip的编译
**加密：**
 Hash		无需安装，自带
 Mcrypt		加密扩展，需安装
 OpenSSL		对称/非对称加解密
**数据库：**
 Mysql(原始)	从PHP5.5.0起这个扩展已经被废弃，并且从PHP7.0.0开始被移除。可使用mysqli或者PDO_MySQL扩展代替。
 Mysqli		MySQL增强版扩展
 Mysqlnd		mysql的客户端
 PDO_MYSQL	mysql的PDO驱动
**国际化与字符编码支持：**
 iconv		包含了iconv字符集转换功能的接口
 gettext	实现了NLS API
**图像生成和处理：**
 GD		图像处理和GD
 Gmagick、ImageMagick、Exif
**邮件相关扩展：** 一般使用第三方邮件系统
**数学扩展：**
 BC Math	BCMath 任意精度数学
 Math		
**其它基本扩展：**
 JSON、Sessions、libxml
 Lua、Swoole、Yaf、Yaml
**其它服务：**
 cURL、Memcache、Memcached、Sockets
