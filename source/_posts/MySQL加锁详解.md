---
title: MySQL加锁详解
date: 2018-11-20 17:19:29
categories:
- MySQL
tags:
- MySQL
---
#### 1. 新建数据表：
```
| test  | CREATE TABLE `test` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(50) NOT NULL DEFAULT '',
  `age` tinyint(3) NOT NULL,
  `hobby` varchar(200) NOT NULL DEFAULT '',
  PRIMARY KEY (`id`),
  UNIQUE KEY `name` (`name`),
  KEY `hobby` (`hobby`)
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8 |
```
<!--more-->

#### 2. 设置事物隔离级别：
```
mysql> set tx_isolation='READ-COMMITTED';
Query OK, 0 rows affected

mysql> select @@tx_isolation;
+-----------------+
| @@tx_isolation  |
+-----------------+
| READ-COMMITTED  |
+-----------------+
1 row in set
```

#### 3. 测试索引对于锁的影响（事务级别：READ-COMMITTED，默认级别（REPEATABLE-READ））
##### ① 只含有一个主键
`delete from test where id = 10; `
这个毋庸置疑，锁的肯定是主键。在id = 10的记录上加上锁即可。
![](/uploads/2018/11/mysql_index_00.jpg)

##### ② 只含有一个唯一键
`delete from test where id = 10; `
这个id不是主键，而是一个Unique的二级索引键值。 
![](/uploads/2018/11/mysql_index_01.jpg)
由于id是unique索引，因此delete语句会选择走id列的索引进行where条件的过滤，在找到id=10的记录后，首先会将unique索引上的id=10索引记录加上X锁，同时，会根据读取到的name列，回主键索引(聚簇索引)，然后将聚簇索引上的name = ‘d’ 对应的主键索引项加X锁。
>为什么聚簇索引上的记录也要加锁？试想一下，如果并发的一个SQL，是通过主键索引来更新：update t1 set id = 100 where name = ‘d’; 此时，如果delete语句没有将主键索引上的记录加锁，那么并发的update就会感知不到delete语句的存在，违背了同一记录上的更新/删除需要串行执行的约束。

##### ③ 仅含有一个普通索引
`delete from test where id = 10; `
相对于前两个的变化，id列上的约束又降低了，id列不再唯一，只有一个普通的索引。
![](/uploads/2018/11/mysql_index_02.jpg)
根据此图，可以看到，首先，id列索引上，满足id = 10查询条件的记录，均已加锁。同时，这些记录对应的主键索引上的记录也都加上了锁。与②唯一的区别在于，②最多只有一个满足等值查询的记录，而③会将所有满足查询条件的记录都加锁。

##### ④ 即含有唯一键，也含有普通索引
（唯一索引和普通索引都会 加锁，其它地方使用主键加锁，死锁概率会增加）

##### ⑤ 不含有索引 （聚簇索引上全部加锁）
`delete from test where id = 10; `
(没有索引，只能走聚簇索引，进行全部扫描。)
相对于前面三个组合，这是一个比较特殊的情况。id列上没有索引，where id = 10;这个过滤条件，没法通过索引进行过滤，那么只能走全表扫描做过滤。
![](/uploads/2018/11/mysql_index_03.jpg)
由于id列上没有索引，因此只能走聚簇索引，进行全部扫描。从图中可以看到，满足删除条件的记录有两条，但是，聚簇索引上所有的记录，都被加上了X锁。无论记录是否满足条件，全部被加上X锁。既不是加表锁，也不是在满足条件的记录上加行锁。


#### 4. 死锁
死锁是指两个或者多个事务在同一资源上相互作用，并请求锁定对方占用的资源，从而导致恶性循环的现象。当多个事务试图以不同的顺序锁定资源时，就可能产生死锁。多个事务同时锁定同一个资源时，也会产生死锁。
##### （1）互锁
![](/uploads/2018/11/mysql_deadlock_01.jpg)
最常见的死锁，每个事务执行两条SQL，分别持有了一把锁，然后加另一把锁，产生死锁。

##### （2）数据查询顺序导致加锁
![](/uploads/2018/11/mysql_deadlock_02.jpg)
虽然每个Session都只有一条语句，仍旧会产生死锁。要分析这个死锁，首先必须用到本文前面提到的MySQL加锁的规则。针对Session 1，从name索引出发，读到的[hdc, 1]，[hdc, 6]均满足条件，不仅会加name索引上的记录X锁，而且会加聚簇索引上的记录X锁，加锁顺序为先[1,hdc,100]，后[6,hdc,10]。而Session 2，从pubtime索引出发，[10,6],[100,1]均满足过滤条件，同样也会加聚簇索引上的记录X锁，加锁顺序为[6,hdc,10]，后[1,hdc,100]。发现没有，跟Session 1的加锁顺序正好相反，如果两个Session恰好都持有了第一把锁，请求加第二把锁，死锁就发生了。

<font color="red">（注：死锁的发生与否，并不在于事务中有多少条SQL语句，死锁的关键在于：两个(或以上)的Session加锁的顺序不一致。）</font>

