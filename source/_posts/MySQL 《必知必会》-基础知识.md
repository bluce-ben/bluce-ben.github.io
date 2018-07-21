---
title: 《MySQL必知必会》-基础知识
date: 2017-12-07 10:57:12
categories: 
- [MySQL]
- [书籍]
tags: 
- MySQL
- 书籍
---
## 一、数据库基础
1、数据库：
数据库（database）保存有组织的数据的容器（通常是一个文件或一组文件）。
数据库软件应称为DBMS（数据库管理系统）。数据库是通过DBMS创建和操纵的容器。你使用的是DBMS，它替你访问数据库。

2、表：
表（table）某种特定类型数据的结构化清单。
*（注：表名：在不同的数据库中可以使用相同的表名。但是在同一数据库却不能使用相同的表名。 表名 = 数据库_表名）*
<!--more-->
3、列和数据类型：
列（column）表中的一个字段。所有的表都是由一个或多个字段构成的。
数据类型（datatype）所容许的数据的类型。每个表列都有相应的数据类型，它限制（或容许）该列中存储的数据。

4、行：
行（row）表中的一个记录（record）。

5、主键：
主键（primary key）一列（或一组列），其值能够唯一区分表中每个行。
*（注：主键列不允许NULL值。 还有一种非常重要的键，称为外键。）*

6、什么是SQL
SQL是结构化查询语言（Structured Query Language)的缩写。SQL是一种专门用来与数据库通信的语言。


## 二、MySQL简介
MySQL是一种DBMS，即它是一种数据库软件。
学习MySQL最好使用专门用途的实用程序，特别有3个工具需要提及：
 1. mysql命令行实用程序
 2. MySQL Administrator（MySQL管理器）是一个图形交互客户机，用来简化MySQL服务器的管理。
 3. MySQL Query Browser为一个图形交互客户机，用来编写和执行MySQL命令。


## 三、使用MySQL
mysql命令行命令：
  `USE database;`  #选择使用数据库
  `SHOW DATABASES;` #显示可用数据库的一个列表
  `SHOW TABLES;` #返回当前数据库内可用表的列表
  `SHOW STATUS;` #用于显示广泛的服务器状态信息
  `SHOW CREATE DATABASE和SHOWCREATE TABLES name`  #显示创建特定数据库或表的MySQL语句
  `SHOW GRANTS`  #用来显示授予用户（所有用户或特定用户）的安全权限
  `SHOW ERRORS和SHOWWARNINGS`  #用来显示服务器错误或警告信息


## 四、检索数据
关键字：
	SELECT、DISTINCT、LIMIT


## 五、排序检索数据
子句（clause）SQL语句由子句构成，有些子句是必须的，而有的是可选的。一个子句通常由一个关键字和所提供的数据组成。子句的例子有SELECT语句的FROM子句。
排序子句：ORDER BY


## 六、过滤数据
过滤子句：WHERE
*（注：在同时使用ORDER BY和WHERE子句时，应该让ORDER BY位于WHERE之后）*
WHERE子句操作符：
	\>、<、>=、<=、=、!=、<>、BETWEEN(必须用AND关键字分割)


## 七、数据过滤
操作符（operator）用来联结或改变WHERE子句中的子句的关键字。也称为逻辑操作符（logical operator)。
操作符：AND、OR、IN、NOT
*（注：SQL在处理OR操作符之前，优先处理AND操作符。如果需要加括号）*
*（注：IN的最大优点是可以包含其他SELECT语句，使得能够更动态地建立WHERE子句。）*
*（注：MySQL支持使用NOT对IN、BETWEEN和EXISTS子句取反）*


## 八、用通配符进行过滤
通配符（wildcard）用来匹配值的一部分的特殊字符。
*（在搜索子句中使用通配符，必须使用LIKE操作符。）*
通配符：
  1. 百分号（%）通配符：表示任何字符出现任意次数。
  *（注：尾空格可能会干扰通配符匹配。办法是去掉首尾空格。）*
  2. 下划线（\_）通配符：用途与%一样，但下划线只匹配单个字符而不是多个字符。


## 九、正则表达式
REGEXP 与 LIKE 使用类似


## 十、创建计算字段
字段拼接函数：Concat()
空格删除函数：RTrim()、LTrim()、Trim()
关键字：AS （别名用AS赋予）
MySQL算数操作符：+、-、\*、/


## 十一、使用数据处理函数
*（注：函数的可移植性不强。因此决定使用函数，应该保证做好代码的注释，以便以后能确切地知道所编写SQL代码的含义。）*
**文本处理函数：**

函数名 			| 说明
--------------- | --------------
Upper()			| 将文本转换为大写
Lower()			| 将串转换为小写
Length() 		| 返回串的长度
LTrim() 		| 去掉串左边的空格	
RTrim() 		| 去掉串右边的空格
Left() 			| 返回串左边的字符
Right() 		| 返回串右边的字符
Locate() 		| 找出串的一个子串
SubString()		| 返回子串的字符
Soundex() 		| 返回串的SOUNDEX值日期和时间处理函数：
AddDate() 		| 增加一个日期（天、周等）
AddTime() 		| 增加一个时间（时、分等）
CurDate() 		| 返回当前日期
CurTime()		| 返回当前时间
Date()			| 返回日期时间的日期部分
DateDiff()		| 计算两个日期之差
Date_Add()		| 高度灵活的日期运算函数
Date_Format()	| 返回一个格式化的日期或时间串
Day()			| 返回一个日期的天数部分
DayOfWeek()		| 对于一个日期，返回对应的星期几
Hour()			| 返回一个时间的小时部分
Minute()		| 返回一个时间的分钟部分
Month()			| 返回一个日期的月份部分
Now()			| 返回当前日期和时间
Second()		| 返回一个时间的秒部分
Time()			| 返回一个日期时间的时间部分
Year()			| 返回一个日期的年份部分
*（注：不管是插入或更新表值还是用WHERE子句进行过滤，日期必须为格式yyyy-mm-dd。防止歧义性。）*
***
例子：
（1）`SELECT cust_id,order_num FROM orders WHERE order_date='2005-09-01';`
此例子中值全部具有时间值 00:00:00，但实际中很可能并不总是这样。因此，需要比较日期部分，而把时间部分忽略。如下：
  `SELECT cust_id,order_num FROM orders WHERE Date(order_date)='2005-09-01';`
如果想要比较的是日期部分，Date()是一个很好的选择。
（2）如果你想检索出2005年9月下的所有订单？
  1. `SELECT cust_id,order_num FROM orders WHERE Date(order_date) BETWEEN '2005-09-01' AND '2009-09-30';`
    弊端：如果搜索的月份是闰年2月的话，还需要注意是否闰月。
  2. `SELECT cust_id,order_num FROM orders WHERE Year(order_date)=2005 AND Month(order_date)=9;`
***
**数值处理函数：**

| 函数名 | 说明                | 
| ------ | ------------------- |
| Abs()  | 返回一个数的绝对值  |
| Cos()  | 返回一个角度的余弦  |
| Exp()  | 返回一个数的指数值  |
| Mod()  | 返回除操作的余数    |
| Pi()   | 返回圆周率          |
| Rand() | 返回一个随机数      |
| Sin()  | 返回一个角度的正弦  |
| Sqrt() | 返回一个数的平方根  |
| Tan()  | 返回一个数的正切    |


## 十二、汇总数据
聚集函数（aggregate function)运行在行组上，计算和返回单个值的函数。
SQL聚集函数：
	AVG()、COUNT()、MAX()、MIN()、SUM()


## 十三、分组数据
分组数据设计两个SELECT语句子句：GROUP BY子句和HAVING子句
分组数据：`GROUP BY`
过滤分组：`HAVING`
**（区别：WHERE过滤行，HAVING过滤分组。且HAVING支持所有WHERE操作符。还有一种理解方法：WHERE在数据分组前进行过滤，HAVING在数据分组后进行过滤。）**
*（注：一般在使用GROUP BY子句时，应该也给出ORDER BY子句，这是保证数据正确排序的唯一方法，千万不要仅依赖GROUP BY排序数据。）*

**SELECT子句及其顺序：**

子句     |        说明         | 是否必须使用
-------- | ------------------- | -------------------
SELECT   | 要返回的列或表达式  | 是
FROM     | 从中检索数据的表    | 仅在从表选择数据时使用
WHERE    | 行级过滤            | 否
GROUP BY | 分组说明            | 仅在按组计算聚集时使用
HAVING   | 组级过滤            | 否
ORDER BY | 输出排序顺序        | 否
LIMIT    | 要检索的行数        | 否


## 十四、使用子查询
子查询最常见的使用是在WHERE子句的IN操作符中，以及用来填充计算列。


## 十五、联结表
外键（foreign key）：外键为某个表中的一列，它包含另一个表的主键值，定义了两个表之间的关系。
可伸缩性（scale）：能够适应不断增加的工作量而不失败。设计良好的数据库或应用程序称之为可伸缩性好（scale well）。
笛卡儿积（cartesian product）：由没有联结条件的表关系返回的结果为笛卡儿积。检索出的行的数数目将是第一个表中的行数乘以第二个表中的行数。
联结：联结是一种机制，用来在一条SELECT语句中关联表，因此称之为联结。
  1. 使用WHERE创建联结：
   `SELECT vend_name,prod_name,prod_price FROM vendors,products WHERE vendors.vend_id = products.vend_id ORDER BY vend_name,prod_name;`
*（注：如果不加WHERE子句，结果将变成笛卡儿积。）*
  2. 内部联结（INNER JOIN）
   `SELECT vend_name,prod_name,prod_price FROM vendors INNER JOIN products ON vendors.vend_id = products.vend_id;`
*（注：ANSI SQL规范首选INNER JOIN语法。）*
*（性能考虑：1. MySQL在运行时关联指定的每个表以处理联结。这种处理可能非常耗费资源的，因此不要联结不必要的表。联结的表越多，性能下降越厉害；2. 利用联结而不使用子查询。）*


## 十六、创建高级联结
使用表别名。
外部联结（OUTER JOIN）：包含没有关联行的那些行。
    `SELECT c.cust_id,o.order_num FROM customers as c LEFT OUTER JOIN orders as o ON c.cust_id = o.cust_id;`
在使用OUTER JOIN语法时，必须使用RIGHT或LEFT关键字指定包括其所有行的表。


## 十七、组合查询
MySQL允许执行多个查询（多条SELECT语句），并将结果作为单个查询结果集返回。这些组合查询通常称为并（union）或复合查询（compound query）。
关键字：UNION
  1. UNION和UNION ALL的区别：
	UNION从查询结果集中自动去除了重复的行。
	UNION ALL返回所有匹配的行。
  2. 在用UNION组合查询时，只能使用一条ORDER BY子句，它必须出现在最后一条SELECT语句之后。不允许使用多条ORDER BY子句。


## 十八、全文本搜索
*（注：并非所有引擎都支持全文本搜索，MyISAM支持，而InnoDB不支持。）*
全文索引：FULLTEXT
使用两个函数Match()和Against()执行全文本搜索，其中Match()指定被搜索的列，Against()指定要使用的搜索表达式。
例如： `SELECT note_text FROM productnotes WHERE Match(note_text) Against('rabbit');`
*（注：传递给Match()的值必须与FULLTEXT()定义中的相同。）*


## 十九、插入数据
关键字：INSERT
  `INSERT INTO table(fields) VALUES(values);`
*（注：多行插入时，values值可以是数组。）*


## 二十、更新和删除数据
关键字：UPDATE、DELETE
*（注：更新和删除操作时，不要省略WHERE子句，否则会删除或更新表中所有行。）*
  `UPDATE table SET field1=value1,...fieldn=valuen WHERE ...;`
  `DELETE FROM table WHERE ...;`


## 二十一、创建和操纵表
1、表的创建：CREATE TABLE 语句
```
CREATE TABLE `customers`(
   'cust_id' int(11) NOT NULL AUTO_INCREMENT,
   'username' varchar(50) NOT NULL DEFAULT='guest',
   'email' varchar(20) NOT NULL,
   'sex' enum('0', '1') NOT NULL,
   PRIMARY KEY('cust_id')
) ENGINE=MyISAM DEFAULT CHARSET=utf8;
```
几种常用引擎：
  1. InnoDB：是一个可靠的事务处理引擎，它不支持全文本搜索；
  2. MEMORY：在功能等同于MyISAM，但由于数据存储在内存（不是磁盘）中，速度很快（特别适合于临时表）；
  3. MyISAM：是一个性能极高的引擎，它支持全文本搜索，但不支持事务处理。
  *（注：引擎类型可以混用。外键不能跨引擎，即使用一个引擎的表不能引用具有使用不同引擎的表的外键。）*

2、表的更改：ALTER TABLE 语句
  1. 增加一个列：
    `ALTER TABLE customers ADD phone varchar(11);`
  2. 删除一个列：
    `ALTER TABLE customers DROP COLUMN phone;`
  3. 定义外键：

3、表的删除：DROP TABLE 语句
    `DROP TABLE customers;`
4、重命名表：RENAME TABLE 语句
    `RENAME TABLE customers TO guests;`