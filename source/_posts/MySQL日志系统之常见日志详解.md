---
title: MySQL日志系统之常见日志详解
date: 2019-08-11 18:34:57
categories:
- MySQL
tags:
- MySQL
- 面试
---

innoDB事务日志包括redo log和undo log。redo log是重做日志，提供前滚操作，undo log是回滚日志，提供回滚操作，这两个日志都是innoDB特有的。而binlog（二进制日志）是MySQL数据库Server层实现的，所有引擎都可以使用。
下面会对 redo log、undo log及binlog进行说明基本概念、相关作用及之间区别。

<!--more-->

首先，我们先来看看一次查询/更新语句流程图：
![](/uploads/2019/08/mysql_select_or_update_process.png)
**本文主要说明 执行器 <-> 存储引擎之间的交互。**
>mysql不是每次数据更改都立刻写到磁盘，而是会先将修改后的结果暂存在内存中，当一段时间后，再一次性将多个修改写到磁盘上，减少磁盘io成本，同时提高操作速度。


### redo log（重做日志）
#### 基本概念
重做日志用来实现事务的持久性，即事务ACID中的D。其由两部分组成：
1. 一是内存中的重做日志缓存（redo log buffer）,其是易失的；
2. 二是重做日志文件（redo log file)，其是持久的。

InnoDB是事务的存储引擎，其通过Force Log at Commit机制实现事务的持久性，即当事务提交（COMMIT）时，必须先将该事务的所有日志写入到重做日志文件进行持久化，待事务的COMMIT操作完成才算完成。

>扩展：主从复制的原理：
mysql中是基于binlog，来进行主从复置的。
oracle中就没有binlog，只用redo log，复制是基于redo log来做。


#### redo log与二进制日志的区别
redo log不是二进制日志。虽然二进制日志中也记录了innodb表的很多操作，也能实现重做的功能，但是它们之间有很大的区别。
1. 二进制日志是在**存储引擎的上层（即Server层）**产生的，不管是什么存储引擎，对数据库进行了修改都会产生二进制日志。而redo log是innodb层产生的，只记录该存储引擎中表的修改。**并且二进制日志先于redo log被记录**。
2. 二进制日志记录操作的方法是逻辑性的语句。即便它是基于行格式的记录方式，其本质也还是逻辑的SQL设置，如该行记录的每列的值是多少。而redo log是在物理格式上的日志，它记录的是数据库中每个页的修改。
3. 二进制日志只在每次事务提交的时候一次性写入缓存中的日志“文件”。而redo log在数据准备修改前写入缓存中的redo log中，然后才对缓存中的数据执行修改操作；而且保证在发出事务提交指令时，先向缓存中的redo log写入日志，写入完成后才执行提交动作。
4. 因为二进制日志只在提交的时候一次性写入，所以二进制日志中的记录方式和提交顺序有关，且一次提交对应一次记录。而redo log中是记录的物理页的修改，redo log文件中同一个事务可能多次记录，最后一个提交的事务记录会覆盖所有未提交的事务记录。
 例如：在binlog日志中记录方式如下：T1,T4,T3,T2，每个事务都会在提交之后被写入到二进制日志中。而redo log是并发写入的，不同事务之间的不同版本的记录会穿插写入到redo log 文件中，例如可能redo log的记录方式如下：T1-1,T1-2,T2-1,T2-2,T2\*,T1-3,T1\*，其中T1* 表示最后提交时的日志记录，所以对应的数据页最终状态是 T1\*对应的操作结果。
5. 事务日志记录的是物理页的情况，它具有幂等性，因此记录日志的方式极其简练。幂等性的意思是多次操作前后状态是一样的，例如新插入一行后又删除该行，前后状态没有变化。而二进制日志记录的是所有影响数据的操作，记录的内容较多。例如插入一行记录一次，删除该行又记录一次。


#### innodb的恢复行为
在启动innoDB的时候，不管上次是正常关闭还是异常关闭，总是会进行恢复操作。且InnoDB可以保证数据库在异常重启后后的状态和使用binlog文件恢复出来的数据库状态保持一致。
我们可以假设没有redo log，只有binlog，那么数据文件更新和写入binlog的顺序由两种可能：
* 第一种，更新数据文件； 写入binlog
* 第二种，写入binlog；更新数据文件

第一种情况如果在完成步骤1后服务器异常关闭，则导致binlog中缺少最后更新的数据；第二种情况如果在完成步骤1后服务器异常关闭，则数据库中比binlog中少了最后的数据变更记录。此时如果使用binlog文件进行恢复数据库（比如备库），则会导致数据库不一致的情况。
**redo log是怎么做的？**
先看下面的图片，是InnoDB更新数据时update语句的执行流程，橙色的流程在InnoDB内部执行，蓝色的部分在MySQL Server层执行器中执行。图片以下条SQL语句为例。
`update T set c=c+1 where ID=2;`
![](/uploads/2019/08/mysql_update_process.png)
如上图所示，redo log的写入分为两个阶段（prepare 和 commit），这个称作两阶段提交，保证了数据的正确性。下面我们从上图4个可能发生异常关闭的时间点来分析InnoDB如何在MySQL启动时做崩溃恢复。
1. Point A
 如果服务器异常关闭发生在Point A 以及之前的时间点，这个时候redo log和binlog 都没有任何记录，事务还未提交，不会造成任何影响。
2. Point B
 当服务器启动的时候发现redo log里处于prepare状态的记录，这个时候需要检查binlog是否完整包含此条redo log的更新内容（通过全局事务ID对应），发现binlog中还未包含此事务变更，则丢弃此次变更。
3. Point C
 和Point B基本相同，只不过此时发现binlog中包含redo log的更新内容，此时事务会进行提交。
4. Point D
 binlog中和数据库中均含有此事务的变更，没有任何影响。

**组提交**
上面关于崩溃恢复部分只是讲了写redo log和binlog 的步骤，那么一定很疑惑数据是何时被写入到磁盘文件中的呢，这里就要说下InnoDB通过redo log实现的组提交的策略了。
因为更新数据时写磁盘的操作是随机写，这部分的IO消耗很大，而通过组提交（多个事务的变更统一写磁盘）的方式可以提升系统的吞吐量。

**组提交实现**
下图是一组redo log文件的工作示意图，如图所示，一组redo log文件是一个类似环形的状态，循环利用。
![](/uploads/2019/08/mysql_redo_log.png)
write pos表示日志当前记录的位置，当ib_logfile_4写满后，会从ib_logfile_1从头开始记录； checkpoint表示将日志记录的修改写进磁盘，完成数据落盘，数据落盘后checkpoint会将日志上的相关记录擦除掉，即write pos -> checkpoint之间的部分是redo log空着的部分，用于记录新的记录，checkpoint -> write pos之间是redo log待落盘的数据修改记录。当write pos追上checkpoint时，得先停下记录，先推动checkpoint向前移动，空出位置记录新的日志。
（注：有了redo log，当数据库发生宕机重启后，可通过redo log将未落盘的数据恢复，即保证已经提交的事务记录不会丢失。）


#### 日志刷盘的规则
内存中（buffer pool，即log buffer）未刷到磁盘的数据称为脏数据（dirty data）。由于数据和日志都以页的形式存在，所以脏页表示脏数据和脏日志。
在innoDB中，数据刷盘的规则只有一个：checkpoint。 但是触发checkpoint的情况却有集中。不管怎样，checkpoint触发后，会将buffer中脏数据页和脏日志页都刷到磁盘。
innoDB存储引擎中checkpoint分为两种：
1. sharp checkpoint：在重用redo log文件（例如切换日志文件）的时候，将所有已记录到redo log中对应的脏数据刷到磁盘。
2. fuzzy checkpoint：一次只刷一小部分的日志到磁盘，而非将所有脏日志刷盘。有以下几种情况会触发该检查点：
 * master thread checkpoint： 由master线程控制，每秒或每10秒刷入一定比例的脏页到磁盘。
 * `flush_lru_list` checkpoint： 从MySQL5.6开始可通过 `innodb_page_cleaners` 变量指定专门负责脏页刷盘的page cleaner线程的个数，该线程的目的是为了保证lru列表有可用的空闲页。
 * async/sync flush checkpoint： 同步刷盘还是异步刷盘。 例如还有非常多的脏页没刷到磁盘（非常多是多少，有比例控制），这时候会选择同步刷到磁盘，但这很少出现；如果脏页不是很多，可以选择异步刷到磁盘，如果脏页很少，可以暂时不刷脏页到磁盘。
 * dirty page too much checkpoint： 脏页太多时强制出发检查点，目的是为了保证缓存有足够的空闲空间。too much的比例由变量`innodb_max_dirty_pages_pct`控制，MySQL5.6默认的值为75，即当脏页占缓冲池的百分之75后，就强制刷一部分脏页到磁盘。

（注：由于刷脏页需要一定的时间来完成，所以记录检查点的位置是在每次刷盘结束之后才在redo log中标记的。）


#### 相关配置
redo log的大小是固定的，在mysql中可以通过修改配置参数 innodb_log_files_in_group和innodb_log_file_size 配置日志文件数量和每个日志文件大小，redolog采用循环写的方式记录，当写到结尾时，会回到开头循环写日志。
>注：binlog文件是通过追加的方式写入的。

1. `innodb_flush_log_at_trx_commit={0|1|2}` #指定何时将事务日志刷到磁盘，默认为1.
 * 0 表示每秒将“log buffer”同步到“os buffer"且从"os buffer"刷到磁盘日志文件中。
 * 1 表示每事务提交都将“log buffer”同步到“os buffer“且从”os buffer“刷到磁盘日志文件中。
 * 2 表示每事务提交都将“log buffer”同步到“os buffer“但每秒才从”os buffer“刷到磁盘日志文件中。
2. `innodb_log_buffer_size:` #log buffer的大小，默认8M
3. `innodb_log_file_size:` #事务日志的大小，默认5M
4. `innodb_log_fiel_group=2` #事务日志组中的事务日志文件个数，默认2个
5. `innodb_log_group_home_dir=./` #事务日志组路径，当前目录表示数据目录
6. `innodb_mirrored_log_groups=1` #指定事务日志组的镜像组个数，但镜像功能好像是强制关闭的，所以只有一个log group。在MySQL5.7中该变量已经移除。


### undo log（回滚日志）
#### 基本概念
undo log有两个作用：提供回滚和多个行版本控制（MVCC）。
在数据修改的时候，不仅记录了redo，还记录了相对应的undo，如果因为某些原因导致事务失败或回滚了，可以借助该undo进行回滚。
undo log不是redo log的逆向过程，其实它们都算是用来恢复的日志：
1. redo log通常是物理日志，记录的是数据页的物理修改，而不是某一行或某几行修改成怎样怎样，他用来恢复提交后的物理数据页（恢复数据页，且只能恢复到最后一次提交的位置）。
2. undo用来回滚行记录到某个版本。undo log一般是逻辑日志，根据每行记录进行记录。


### binlog（binary log，二进制日志）
#### 基本概念
MySQL的二进制日志binlog可以说是MySQL最重要的日志，它记录了所有的DDL和DML语句（除了数据查询语句select），以事件形式记录，还包含语句所执行的消耗的时间，MySQL的二进制日志是事务安全型的。

binlog日志有两个最重要的使用场景：
1. mysql主从复制： mysql replication在master端开启binlog，master把它的二进制日志传递给slaves来达到master-slave数据一致的目的。
2. 数据恢复： 通过mysqlbinlog 工具来恢复数据。

>binlog 日志包括两类文件：
1. 二进制日志索引文件（文件名后缀为 .index）用于记录所有的二进制文件。
2. 二进制日志文件（文件名后缀为 .00000\*）记录数据库所有的DDL和DML（除了数据查询语句select）语句事件。
>>DDL（Data Definition Language 数据库定义语言），主要命令有create、alter、drop等，ddl主要是用在定义或改变表（table）的结构，数据类型，表之间的连接和约束等初始工作上，他们大多在建表时候使用。
>>DML（Data Manipulation Language 数据操纵语言），主要命令是select、update、insert、delete，就像他的名字一样，这4条命令是用来对数据库里的数据进行操作的语言。

#### 相关配置
binlog文件是通过追加的方式写入的，可通过配置参数`max_binlog_size`设置每个binlog文件的大小，当文件大小大于给定值后，日志会发生滚动，之后的日志记录到新的文件上。
binlog有两种记录模式，statement格式的话是记sql语句，row格式会记录行的内容，记两条，更新前和更新后都有。

注：binlog文件配置主从复制可查看我的另一篇文章。

博客：
[详细分析MySQL事务日志（redo log和undo log）](https://www.cnblogs.com/f-ck-need-u/archive/2018/05/08/9010872.html#auto_id_11)
[MySQL崩溃恢复功臣——Redo Log](https://cloud.tencent.com/developer/article/1417482)
[MySQL日志系统之redo log和bin log](https://www.jianshu.com/p/4bcfffb27ed5)

