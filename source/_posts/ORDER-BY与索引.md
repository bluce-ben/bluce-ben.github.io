---
title: ORDER BY与索引
date: 2018-08-28 19:04:48
categories:
- [SQL]
- [SQL优化]
tags:
- SQL
- SQL优化
---
一条SQL实际上可以分为三步。
1. 得到数据
2. 处理数据
3. 返回处理后的数据

比如这条语句`select sid from zhuyuehua.student where sid < 50000 and id < 50000 order by id desc`
第一步：根据where条件和统计信息生成执行计划，得到数据。 
第二步：将得到的数据排序。 
**当执行处理数据（order by）时，数据库会先查看第一步的执行计划，看order by 的字段是否在执行计划中利用了索引。如果是，则可以利用索引顺序而直接取得已经排好序的数据。**如果不是，则排序操作。
第三步：返回排序后的数据。 

总结：
**当order by 中的字段出现在where条件中时，才会利用索引而不排序，更准确的说，order by 中的字段在执行计划中利用了索引时，不用排序操作。**

这个结论不仅对order by有效，对其他需要排序的操作也有效。比如**<font color="red">group by 、union 、distinct</font>**等。
