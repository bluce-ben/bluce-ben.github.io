---
title: MySQL 存储引擎详解
date: 2018-03-28 18:47:23
categories:
- MySQL
tags:
- MySQL
---
数据库存储引擎是数据库底层软件组织，数据库管理系统（DBMS）使用数据引擎进行创建、查询、更新和删除数据。不同的存储引擎提供不同的存储机制、索引技巧、锁定水平等功能，使用不同的存储引擎，还可以 获得特定的功能。现在许多不同的数据库管理系统都支持多种不同的数据引擎。MySql的核心就是存储引擎。

#### 存储引擎查看
MySQL给开发者提供了查询存储引擎的功能，我这里使用的是 10.1.28-MariaDB，可以使用：
`show engines;`
<!--more-->

Engine       | Suppert | Comment                     | Transactions | XA | Savepoints
-------------|---------|-----------------------------|--------------|----|--------------------------
CSV			 | YES     | CSV storage engine          | No           | No | No
InnoDB       | DEFAULT | Comment: Percona-XtraDB, Supports transactions, row-level locking, foreignkeys and encryption for tables  | YES  | YES | YES
MEMORY       | YES     | Hash based, stored in memory, useful for temporary tables | NO | NO | NO
MyISAM       | YES     | MyISAM storage engine       | NO           | NO | NO
MRG_MyISAM   | YES     | Collection of identical MyISAM tables      | NO | NO | NO
Aria         | YES     | Crash-safe tables with MyISAM heritage     | NO | NO | NO
PERFORMANCE_SCHEMA | YES | Performance Schema        | NO           | NO | NO | NO
SEQUENCE     | YES     | Generated tables filled with sequential values  | YES | NO | YES

看到MySQL给用户提供了这么多存储引擎，包括处理事务安全表的引擎和出来了非事物安全表的引擎。


#### InnoDB
InnoDB是事务型数据库的首选引擎，支持事务安全表（ACID），支持行锁定和外键，上图也看到了，InnoDB是默认的MySQL引擎。
InnoDB存储表和索引有一下两种方式：
1. 使用共享表空间存储，这种方式创建的表的表结构保存在.frm文件中，数据和索引保存在innodb_data_home_dir和innodb_data_file_path定义的表空间中，可以是多个文件。
2. 使用多表空间存储，这种方式创建的表的表结构仍然保存在.frm文件中，但是每个表的数据和索引单独保存在.ibd中，如果是个分区表，则每个分区对应单独的.ibd文件，文件名是"表名+分区名"，可以在创建分区的时候指定每个分区的数据文件位置，以此来将表的IO均匀分布在多个磁盘上。

**选择理由：**
用于事务处理应用程序，支持外键。如果应用对事务的完整性有比较高的要求，在并发条件下要求数据一致性，数据操作除了插入和查询意外，还包括很多的更新删除操作，那么InnoDB比较合适。InnoDB存储引擎除了有效的降低由于删除和更新操作导致的锁定，还可以确保事务的完整提交和回滚。


#### MyISAM 
它是在Web、数据仓储和其他应用环境下最常使用的存储引擎之一。MyISAM拥有较高的插入、查询速度，但不支持事物。
每个MyISAM在磁盘上存储成3个文件，文件名都和表名相同，但是扩展名不同，扩展名分别是：
1. .frm（存储表定义）；
2. .MYD（MYData，存储数据）；
3. .MYI（MYIndex，存储索引）；

MyISAM的表还支持3种不同的存储格式，分别是：
1. 静态（固定长度）表；
2. 动态表；
3. 压缩表；

**选择理由：**
如果应用是以读写操作和插入操作为主，只有很少的更新和删除操作，并且对事务的完整性、并发性要求不是很高可选用此种存储引擎。


#### MEMORY
MEMORY存储引擎将表中的数据存储到内存中，未查询和引用其他表数据提供快速访问。
MEMORY存储引擎使用存在于内存中的内容来创建表。每个MEMORY表只实际对应一个磁盘文件，格式.frm。MEMORY类型的表访问非常快，因为它的数据是存放在内存中的，并且默认使用HASH索引，但是一旦服务关闭，表中的数据就会丢失。

**选择理由：**
将所有的数据保存在RAM中，在需要快速定位记录和其他类似数据的环境下，可提供极快的访问。MEMORY的缺陷是对表的大小有限制，太大的表无法缓存在内存中，其次要确保表数据可以恢复，数据库异常终止后表中的数据是可以恢复的。MEMORY表通常用于更新不太频繁的小表，用以快速得到访问结果。


#### 下面对一些常用的引擎的特点进行汇总

功  能			| MYISAM | Memory |	InnoDB | Archive
----------------|--------|--------|--------|------------
存储限制	    | 256TB  |	RAM	  | 64TB   | None
支持事物	    | No	 | No	  | Yes	   | No
支持全文索引	| Yes	 | No	  | No	   | No
支持数索引	    | Yes	 | Yes    |	Yes    | No
支持哈希索引    | No     | Yes    |	No     | No
支持数据缓存	| No	 | N/A    |	Yes    | No
支持外键	    | No	 | No	  | Yes	   | No

**存储引擎的选择：**
如果要提供提交、回滚、崩溃恢复能力的事物安全（ACID兼容）能力，并要求实现并发控制，InnoDB是一个好的选择

如果数据表主要用来插入和查询记录，则MyISAM引擎能提供较高的处理效率

如果只是临时存放数据，数据量不大，并且不需要较高的数据安全性，可以选择将数据保存在内存中的Memory引擎，MySQL中使用该引擎作为临时表，存放查询的中间结果

如果只有INSERT和SELECT操作，可以选择Archive，Archive支持高并发的插入操作，但是本身不是事务安全的。Archive非常适合存储归档数据，如记录日志信息可以使用Archive

使用哪一种引擎需要灵活选择，一个数据库中多个表可以使用不同引擎以满足各种性能和实际需求，使用合适的存储引擎，将会提高整个数据库的性能


#### MyISAM与InnoDB的区别
1、InnoDB支持事务处理，而MyISAM不支持
2、InnoDB支持外键，而MyISAM不支持
3、InnoDB是行锁，而MyISAM是表锁（行锁开销大，高并发；表锁开销小，并发低）
4、MyISAM支持全文索引，而InnoDB不支持