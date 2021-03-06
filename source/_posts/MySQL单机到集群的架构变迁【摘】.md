---
title: MySQL单机到集群的架构变迁【摘】
date: 2017-12-08 10:27:55
categories: 
- [MySQL, 架构]
tags:
- MySQL
- 架构
---
### 1、业务上线初期，web服务规模较小，一般具备以下特点：
1. 服务原型时期,用户基数小,多种业务公用资源,日均写入百万级别,读取千万级别
2. 数据规模小，单机性能能够满足需求
3. 用户规模小，开发重心偏向迭代速度
4. 考虑到上述小型业务特点，为节约资源成本及开发成本，可以采用多个业务混合部署形式
![](/uploads/2017/12/mysql_read_write_01.jpg)
<!--more-->
### 2、当用户增多，数据量、访问量升高(2倍以下)，数据库压力较大，怎样在有限程度提高MySQL吞吐量呢？
1. SQL优化
2. 硬件升级
　压力还在有限的范围内增长，通过简单、低成本优化，可以一定成都上提高有限的服务性能

### 3、业务持续发展，读取性能出现瓶颈&&各业务互相影响，多个业务出 现资源抢占，如何快速解决业务抢占问题以提高服务性能？
　垂直拆分——按业务进行数据拆分
![](/uploads/2017/12/mysql_read_write_02.jpg)

### 4、随着业务的继续发展，读取性能出现瓶颈,读写互相影响，如何确保读请求量的增加，不要影响写入性能？相反写入请求量增加如何确保不影响读取性能？(写入性能出现问题会造成数据丢失)
　读写分离，写入仅写master，master与slave自动同步；读取仅以slave作为来源
![](/uploads/2017/12/mysql_read_write_03.jpg)
　读写分离后，slave仅专注于承担读请求，读取性能得到优化；同里独立的master服务的写入性能也得到优化。

### 5、一台/一对M-S服务器性能终归是很有限的，当单实例服务性能无法承载线上的请求量时，如何进行优化？
1. 升级为一主多从架构
2. 一个master承载所有写入请求，理论上master性能不变
3. 多个slave分担读取请求，读取性能提升n倍
![](/uploads/2017/12/mysql_read_write_04.jpg)

### 6、随着业务量的增长，服务出现如下变化：
　数据量增长，意味着原本的存储空间不足
　写入量增长，意味着master写入性能存在瓶颈
　读取量增长，意味着slave读取性能也存在瓶颈，但通过扩充slave是有限制的：一方面M-Sreplication性能有风险；另一方面扩充slave的成本较高
**水平拆分**
![](/uploads/2017/12/mysql_read_write_05.jpg)
　业务经历数据量的增长、读写请求量的增长，数据库服务已经演进为分布式架构，一个业务的数据，怎样合理的分布到上述复杂的分布式数据库是下一个需要解决的问题

### 7、如何基于上述演进到最后的架构进行数据库设计呢？
1. 分布式数据库设计
2. hash拆分方式，既按hash规则，将数据读写请求分散到多个实例上，见上述水平拆分示意图
3. 时间拆分方式，基于确定好的时间划分规则，将数据按时间段分散存储再多个实例中
![](/uploads/2017/12/mysql_read_write_06.jpg)

### 8、当一台服务器宕机怎么办？
（1）Slave(一主多从)宕机？
	1. 剩余健康Slave无风险，则无需紧急操作，例行修复
	2. 切换流量到容灾机房(如果具备容灾机房)
	3. 紧急扩容[优先]、重启、替换
	4. 有损降级部分请求
（2）Master宕机？
　由于master数据的唯一性，致使master出现异常会直接造成数据写入失败
	1. 快速下线master
	2. 下线一台salve的读服务(如果slave性能有风险，则同时快速扩容)
	3. 提升slave为master
	4. 生效新master与slave的同步机制

### 9、注意事项
（1）MySQL设计应该注意的问题
	1. 表字符集选择UTF8
	2. 存储引擎使用InnoDB
	3. 使用Varchar/Varbinary存储变长字符串
	4. 不在数据库中存储图片、文件等
	5. 每张表数据量控制在20000W以下
	6. 提前对业务做好垂直拆分
（2）MySQL查询应该遇到的问题
	1. 避免使用存储过程、触发器、函数等
	　让数据库做最擅长的事
	　降低业务耦合度避开服务端BUG
	2. 避免使用大表的JOIN
	　MySQL最擅长的是单表的主键/索引查询
	　JOIN消耗较多内存,产生临时表
	3. 避免在数据库中进行数学运算
	　MySQL不擅长数学运算
	　无法使用索引
	4. 减少与数据库的交互次数
	　select条件查询要利用索引
	　同一字段的条件判定要用in而不要用or

*摘抄博客：*
**[新兵训练营系列课程——海量数据存储基础](https://weibo.com/p/1001643874615465508614)**
*分享一篇博客：*
**[QPS从0到4000请求每秒，谈达达后台架构演化之路](http://blog.csdn.net/czbing308722240/article/details/52350219)**