---
title: MySQL调优三部曲
date: 2017-12-07 18:04:34
categories: 
- [MySQL]
- [SQL]
tags:
- MySQL
- SQL
---
1、慢查询 （分析出现出问题的sql）
2、Explain （显示了mysql如何使用索引来处理select语句以及连接表。可以帮助选择更好的索引和写出更优化的查询语句）
3、Profile（查询到 SQL 会执行多少时间, 并看出 CPU/Memory 使用量, 执行过程中 Systemlock, Table lock 花多少时间等等.）
<!--more-->

## 慢查询
　MySQL通过慢查询日志定位那些执行效率较低的SQL 语句，用--log-slow-queries[=file_name]选项启动时，mysqld 会写一个包含所有执行时间超过long_query_time 秒的SQL语句的日志文件，通过查看这个日志文件定位效率较低的SQL 。下面介绍MySQL中如何查询慢的SQL语句：
#### 配置开启
**配置选项：**

配置项 			|  说明
--------------- | -------------------
long_query_time |  设定慢查询的阀值，超出次设定值的SQL即被记录到慢查询日志，缺省值为10s  
slow_query_log  |  指定是否开启慢查询日志  
log_slow_queries |  指定是否开启慢查询日志(该参数要被slow_query_log取代，做兼容性保留)  
slow_query_log_file |  指定慢日志文件存放位置，可以为空，系统会给一个缺省的文件host_name-slow.log  
min_examined_row_limit |查询检查返回少于该参数指定行的SQL不被记录到慢查询日志  
log_queries_not_using_indexes | 不使用索引的慢查询日志是否记录到索引

**开启配置**
查询慢查询是否开启：
`show variables like '%slow%';`
开启慢查询日志：
`set global log_slow_queries=1;`
查询慢查询时间：
`show variables like '%long_query_time%';`

#### 查看方式
利用MySQL自带的慢查询日志分析工具mysqldumpslow分析日志。

	perlmysqldumpslow –s c –t 10 slow-query.log
	具体参数设置如下：
	-s 表示按何种方式排序，c、t、l、r分别是按照记录次数、时间、查询时间、返回的记录数来排序，ac、at、al、ar，表示相应的倒叙；
	-t 表示top的意思，后面跟着的数据表示返回前面多少条；
	-g 后面可以写正则表达式匹配，大小写不敏感。
例子：
`mysqldumpslow -s c -t 20 host-slow.log` 	#访问次数最多的20个sql语句
`mysqldumpslow -t 10 -s t -g “left join” host-slow.log` 	#按照时间返回前10条里面含有左连接的sql语句


## Explain
explain显示了mysql如何使用索引来处理select语句以及连接表。可以帮助选择更好的索引和写出更优化的查询语句。
使用方法，在select语句前加上explain就可以了：
例如：
explain select surname,first_name form a,b where a.id=b.id
**显示结果分析：**

	id | select_type | table |  type | possible_keys | key | key_len | ref | rows | Extra 
	EXPLAIN列的解释
	table
	显示这一行的数据是关于哪张表的
	type
	这是重要的列，显示连接使用了何种类型。从最好到最差的连接类型为const、eq_reg、ref、range、indexhe和ALL
	possible_keys
	显示可能应用在这张表中的索引。如果为空，没有可能的索引。可以为相关的域从WHERE语句中选择一个合适的语句
	key
	实际使用的索引。如果为NULL，则没有使用索引。很少的情况下，MYSQL会选择优化不足的索引。这种情况下，可以在SELECT语句 中使用USE INDEX（indexname）来强制使用一个索引或者用IGNORE INDEX（indexname）来强制MYSQL忽略索引
	key_len
	使用的索引的长度。在不损失精确性的情况下，长度越短越好
	ref
	显示索引的哪一列被使用了，如果可能的话，是一个常数
	rows
	MYSQL认为必须检查的用来返回请求数据的行数
	Extra
	关于MYSQL如何解析查询的额外信息。这里可以看到的坏的例子是Using temporary和Using filesort，意思MYSQL根本不能使用索引，结果是检索会很慢
**Extra列返回的描述的意义：**

	Distinct
	一旦MYSQL找到了与行相联合匹配的行，就不再搜索了
	Not exists
	MYSQL优化了LEFT JOIN，一旦它找到了匹配LEFT JOIN标准的行，就不再搜索了Range checked for each Record（index map:#）没有找到理想的索引，因此对于从前面表中来的每一个行组合，MYSQL检查使用哪个索引，并用它来从表中返回行。这是使用索引的最慢的连接之一
	Using filesort
	看到这个的时候，查询就需要优化了。MYSQL需要进行额外的步骤来发现如何对返回的行排序。它根据连接类型以及存储排序键值和匹配条件的全部行的行指针来排序全部行
	Using index
	列数据是从仅仅使用了索引中的信息而没有读取实际的行动的表返回的，这发生在对表的全部的请求列都是同一个索引的部分的时候
	Using temporary
	看到这个的时候，查询需要优化了。这里，MYSQL需要创建一个临时表来存储结果，这通常发生在对不同的列集进行ORDER BY上，而不是GROUP BY上Where used使用了WHERE从句来限制哪些行将与下一张表匹配或者是返回给用户。如果不想返回表中的全部行，并且连接类型ALL或index，这就会发生，或者是查询有问题不同连接类型的解释（按照效率高低的顺序排序）
**Type列不同连接类型的解释：（按照效率高低的顺序排序）**

	system 
	表只有一行：system表。这是const连接类型的特殊情况 
	const 
	表中的一个记录的最大值能够匹配这个查询（索引可以是主键或惟一索引）。因为只有一行，这个值实际就是常数，因为MYSQL先读这个值然后把它当做常数来对待 
	eq_ref 
	在连接中，MYSQL在查询时，从前面的表中，对每一个记录的联合都从表中读取一个记录，它在查询使用了索引为主键或惟一键的全部时使用 
	ref 
	这个连接类型只有在查询使用了不是惟一或主键的键或者是这些类型的部分（比如，利用最左边前缀）时发生。对于之前的表的每一个行联合，全部记录都将从表中读出。这个类型严重依赖于根据索引匹配的记录多少—越少越好 
	range 
	这个连接类型使用索引返回一个范围中的行，比如使用>或<查找东西时发生的情况 
	index 
	这个连接类型对前面的表中的每一个记录联合进行完全扫描（比ALL更好，因为索引一般小于表数据） 
	ALL 
	这个连接类型对于前面的每一个记录联合进行完全扫描，这一般比较糟糕，应该尽量避免

## Profile
　Query Profiler是MYSQL自带的一种query诊断分析工具，通过它可以分析出一条SQL语句的性能瓶颈在什么地方。通常我们是使用的explain,以及slow query log都无法做到精确分析，但是Query Profiler却可以定位出一条SQL语句执行的各种资源消耗情况，比如CPU，IO等，以及该SQL执行所耗费的时间等。
　Show profiles是5.0.37之后添加的，要想使用此功能，要确保版本在5.0.37之后。
　查看数据库版本方法：`show variables like "%version%";`  或者  `select version();`

1、确定支持show profile 后，查看profile是否开启，数据库默认是不开启的。变量profiling是用户变量，每次都得重新启用。
查看方法： `show variables like "%pro%";`
2、开启和关闭
```
mysql> set profiling=1;
mysql> set profiling=0; 
```
　information_schema 的 database 会建立一个PROFILING 的 table 记录. 
3、执行一些语句（自定义语句）
`mysql>select * from navigation_sub where navPId<6 and navSName='公司介绍';`
4、查询语句执行时间
`mysql>show profiles;`
5、查询语句详细执行时间
`mysql>show profile for query 2;`
**（注：此处的 2 表示再 `show profiles` 查询后获取的 Query_ID字段。）**