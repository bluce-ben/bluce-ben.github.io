---
title: 谈谈MySQL隐式类型转换【转】
date: 2017-12-18 11:52:21
categories:
- MySQL
tags:
- MySQL
---
### 前言
　　今天我们继续回到MySQL系列文章中,谈一谈MySQL中隐式类型转换。(其实我最早知道是在慢SQL优化中知道的隐式类型转换概念的),在说隐式类型转换之前,首先我们通过一个实例来看看是怎么回事。
<!--more-->
### 数据结构
本文中所有的操作,都是基于该数据结构(有兴趣的童鞋,可以实验):

	create table t_base_user(   
	oid bigint(20) not null primary key auto_increment,  
	name varchar(30) null comment "name",  
	email varchar(30) null comment "email",  
	age int null comment "age",  
	telephone varchar(30) null comment "telephone",  
	status tinyint(4) null comment "0 无效 1 有效",  
	created_at datetime null default now() comment "创建时间",  
	updated_at datetime null default now() comment "修改时间"  
	)

	### 新建索引
	alter table t_base_user add index idx_email(email);
	alter table t_base_user add index idx_name(name);
	alter table t_base_user add index idx_telephone(telephone);

	### 新增记录: 
	INSERT INTO `test`.`t_base_user` (`name`, `email`, `age`, `telephone`, `status`, `created_at`, `updated_at`) 
	VALUES ('111111', 'andytohome@gmail.com', '111', '12345678901', '1', now(),now());

### 引子
  首先我们基于上述数据结构中,我们来看看下面这个执行计划:
> explain select * from t_base_user where telephone=12345678901;

执行计划结果:

	id | select_type |    table    | type | possible_keys | key  | key_len | ref  | rows | Extra
	-- | ----------- | ----------- | ---- | ------------- | ---- | ------- | ---- | ---- | ------------
	1  | SIMPLE      | t_base_user | ALL  | idx_telephone | NULL | NULL    | NULL | 1    | Using where

细心的童鞋应该已经看出来了,为什么数据结构中已经在telephone字段上新建了idx_telephone，而上述语句并没有走索引,而是全表扫描。这是为什么呢？带着这疑问,我们来看看今天的主角–MySQL隐式类型转换

### 什么隐式类型转换？
  在MySQL中:
> 当操作符与不同类型的操作数一起使用时，会发生类型转换以使操作数兼容。则会发生转换隐式

也就是说,MySQL会根据需要自动将数字转换为字符串，将字符串转换数字。看到这个概念之后,是不是有一种茅塞顿开的感觉。哦… 原来在数据结构中telephone字段为字符串(varchar)类型,而我们传的手机号是数字类型。现在我们将SQL修改下:

> select * from t_base_user where telephone=”1234567890”;

再看看执行计划上述语句的执行计划:

> explain select * from t_base_user where telephone="1234567890";

结果:

	id | select_type |    table    | type | possible_keys |      key      | key_len | ref  | rows | Extra
	-- | ----------- | ----------- | ---- | ------------- | ------------- | ------- | ---- | ---- | ------------
	1  | SIMPLE      | t_base_user | ref  | idx_telephone | idx_telephone |   63    | const|   1  | Using index condition

从这里看,现在语句已经走索引了。为了加深我们对隐式类型转换的印象,我们再多看看几个隐式类型转换案例:
案例一: 字符串转换为数字
> mysql > SELECT 1+’1’;

结果:
> mysql > 2

案例二: 数字转换为字符串
> mysql -> SELECT CONCAT(1024,’ andyqian’);

结果:
‘1,’ test’;

此时CONCAT(字符拼接)函数就进行了隐式类型转换。

### 如何避免隐式类型转换?
　　只有当清楚的知道隐式类型转换的规则，才能从根本上避免产生隐式类型转换。MySQL也在官网描述了进行隐式类型转换的一些规则如下:

**1. 隐式类型转换规则:**

* 如果一个或两个参数都是NULL，比较的结果是NULL，除了NULL安全的<=>相等比较运算符。对于NULL <=> NULL，结果为true。不需要转换
* 如果比较操作中的两个参数都是字符串，则将它们作为字符串进行比较。
* 如果两个参数都是整数，则将它们作为整数进行比较。
* 如果不与数字进行比较，则将十六进制值视为二进制字符串
* 如果其中一个参数是十进制值，则比较取决于另一个参数。 如果另一个参数是十进制或整数值，则将参数与十进制值进行比较，如果另一个参数是浮点值，则将参数与浮点值进行比较
* 如果其中一个参数是TIMESTAMP或DATETIME列，另一个参数是常量，则在执行比较之前将常量转换为时间戳。
* 在所有其他情况下，参数都是作为浮点数（实数）比较的。

**2. 使用CAST函数显示转换**
我们可以使用CAST显示的将类型进行转换,如下所示:
> mysql> SELECT 38.8, CAST(38.8 AS CHAR);

结果:
> mysql > 38.8, ‘38.8’

如上述中:
> select * from t_base_user where telephone=cast(15608464487 as char);

查看执行计划,我们也可以看出,这个时候也走索引了。

**3. 类型一致**
  这里说的类型一致,指的是在写SQL时,参数类型一定要与数据库中的类型一致,避免产生隐式类型转换,就如刚才在文首时,如果多检查,写的SQL的参数类型与数据库中字段类型一致，也就不会不走索引了，你说是不是？

### 小心隐式类型转换
　　这里再重申一次,写SQL时一定要检查参数类型与数据库字段类型一致,(如果参数不一致,也要使用CAST函数显示转换成一致)否则造成隐式类型转换,不走索引,后果简直不堪设想, 在前面《写会MySQL索引》这篇文章中提到过,不走索引,轻则造成慢查询，重则造成数据库服务器CPU100%。唉,说到这里,不瞒你说，我就吃过不少MySQL隐式类型转换的亏,导致了慢查询。

### 小结
　　看到这里,是不是有一种，数据表设计还真不是件容易的事情。需要考虑的因素太多太多了,需要考虑字段类型,索引设计,还有各种约束条件等等。也一定要谨慎谨慎再谨慎！其实换个角度就更容易理解了,大家都知道高楼大厦都是需要一个好的地基的,在数据库表设计中,前期的表结构设计就是这个地基，其重要性可想而知。