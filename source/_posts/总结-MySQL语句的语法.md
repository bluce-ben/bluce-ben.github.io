---
title: 总结-MySQL语句的语法
date: 2017-12-07 16:05:45
categories: 
- [MySQL]
- [SQL]
tags: 
- MySQL
- SQL
---
## MySQL语句的语法：
1、`ALTER TABLE` 更新已存在的表的模式。
2、`COMMIT`  用来将事务处理写到数据库。
3、`CREATE INDEX`   用于在一个或多个列上创建索引。
4、`CREATE PROCEDURE`  用于创建存储过程。
5、`CREATE TABLE`   用于创建数据库表。
<!--more-->
6、`CREATE USER`   用于向系统中添加新的用户账号。
7、`CREATE VIEW`   用来创建一个或多个表上的新视图。
8、`DELETE`    从表中删除一行或多行。
9、`DROP`   永久地删除数据库对象（表、视图、索引等）。
10、`INSERT`   给表增加一行。
11、`INSERT SELECT`   插入SELECT的结果到一个表。
12、`ROLLBACK`   用于撤销一个事务处理块。
13、`SAVEPOINT`    为使用ROLLBACK语句设立保留点。
14、`SELECT`    用于从一个或多个表（视图）中检索数据。
15、`START TRANSACTION`   表示一个新的事务处理块的开始。
16、`UPDATE`    更新表中的一行或多行。

## 详解：
1、ALTER TABLE     更新已存在的表的模式。
	1. 删除列
	　ALTER TABLE 【表名字】 DROP 【列名称】
	2. 增加列
	　ALTER TABLE 【表名字】 ADD 【列名称】 【列类型】  COMMENT '注释说明'
	3. 修改列的类型信息
	　ALTER TABLE 【表名字】 MODIFY 【列名称】【列类型】
	4. 重命名列
	　ALTER TABLE 【表名字】 CHANGE 【列名称】【新列名称】【新列类型】
	5. 重命名表
	　ALTER TABLE 【表名字】 RENAME 【表新名字】
	6. 更改表存储引擎
	　ALTER TABLE ENGINE=InnoDB;
	7. 删除表中主键
	　Alter TABLE 【表名字】 DROP PRIMARY KEY
	　（注：该主键不能为自增，否则会报错。）
	8. 添加主键
	　ALTER TABLE 【表名字】 ADD PRIMARY KEY (`resid`,`resfromid`);
	　（注：如果表中有数据，直接添加主键会报错。需要①字段新建普通索引②字段增加自增属性③字段新建主键④删除字段的普通索引。）
	9. 添加索引
	　ALTER TABLE 【表名字】 ADD INDEX index_name (name);
	10. 添加唯一索引
	　ALTER TABLE 【表名字】 ADD UNIQUE emp_name2(cardnumber);
	11. 删除索引
	　ALTER TABLE 【表名字】 DROP INDEX emp_name;
	12. 添加外键索引
	　ALTER TABLE 【表名字】 ADD FOREIGN KEY index_name(name) REFERENCES foreign_table(name);
	　（注：添加外键需要两个表的存储引擎都是InnoDB型。）
	13. 删除外键索引
	　ALTER TABLE 【表名字】 DROP FOREIGN KEY index_name;
	14. 添加复合索引
	  ALTER TABLE 【表名字】 ADD INDEX index_name(`column1`, `column2`);

2、CREATE TABLE   创建数据库表
	```
	CREATE TABLE [IF NOT EXISTS] `tbl_name`(
	`col_name` type [NOT NULL | NULL] [DEFAULT default_value] [AUTO_INCREMENT]
	PRIMARY KEY (`index_col_name`,...)
	or KEY [index_name] (`index_col_name`,...)
	or INDEX [index_name] (`index_col_name`,...)
	or UNIQUE [INDEX] [index_name] (`index_col_name`,...)
	or FOREIGN KEY index_name (`index_col_name`,...）
	)ENGINE=MyISAM DEFAULT CHARSET=UTF8;
	```

3、CREATE USER    创建用户
	1. 创建用户
	　CREATE USER user01@'localhost/%' IDENTIFIED BY 'password1';
	2. 修改密码
	　SET PASSWORD FOR 'user01'@'localhost/%' = PASSWORD('password2');

4、GRANT    赋予权限
	1. 设置权限
	　grant 权限 on 数据库对象 to 用户
	2. 取消权限
	　revoke 权限 on 数据库对象 from 用户
	3. 查看权限
	　show grants for dba@localhost;

5、CREATE VIEW     创建视图
	1. 创建视图
	　CREATE VIEW  view_name AS SQL;
	2. 删除视图
	　DROP VIEW [IF EXISTS] view_name;
	　（注：视图不能修改，只能删除，然后重建。）

6、DROP      删除（表、视图、存储过程）
	`DROP TABLE/VIEW/PROCEDURE [IF EXISTS] name;`

7、CREATE PROCEDURE     创建存储过程
	1. 创建存储过程
	　CREATE PROCEDURE procedure_name(
	　IN  id int,
	　OUT price DECIMAL(8,2)
	 　)
	　BEGIN
	　SQL;
	　END;
	2. 调用存储过程
	　CALL  procedure_name();

8、触发器
	1. 创建触发器
	　CREATE TRIGGER trigger_name [AFTER/BEFORE] [INSERT/UPDATE/DELETE] ON table_name
	　FOR EACH ROW 操作;
	　（在对表操作之前或之后，执行操作，EACH ROW表示每行变动都需要执行操作）
	2. 删除触发器
	　DROP TRIGGER trigger_name;
	3. 如何使用触发器
	　此处不需要人为操作触发器，当对表操作的时候自动执行。

9、事务处理
```
	操作格式：
	START TRANSACTION;
	SQL;
	COMMIT;
	ROLLBACK;
```

## 数据表的基本操作：
1、SELECT
`select [ALL/DISTINCE] 字段表达式子句 [from子句] [where子句] [group by子句] [having子句] [order by子句] [limit子句];`

2、UPDATE
`UPDATE tbl_name SET col_name1=expr1 [, col_name2=expr2 ...][WHERE子句] [ORDER BY子句] [LIMIT子句];`

3、DELETE
`DELETE FROM tbl_name [WHERE子句] [ORDER BY子句] [LIMIT子句];`

4、INSERT
`INSERT INTO tbl_name(col_name1,col_name2...) VALUES(col_vallue1,col_value2...);`


## 附属：
修改数据表编码及查询数据库编码：
（1）查询数据库编码
```
mysql> show variables like '%char%';
+--------------------------+--------------------------------+
| Variable_name            | Value                          |
+--------------------------+--------------------------------+
| character_set_client     | utf8                           |
| character_set_connection | utf8                           |
| character_set_database   | utf8                           |
| character_set_filesystem | binary                         |
| character_set_results    | utf8                           |
| character_set_server     | utf8                           |
| character_set_system     | utf8                           |
| character_sets_dir       | D:\xampp\mysql\share\charsets\ |
+--------------------------+--------------------------------+
8 rows in set
```
如果想要修改编码的话，可使用命令：`set character_set_database = utf8;`

（2）修改数据表编码
`ALTER TABLE table_name character set utf8;`
