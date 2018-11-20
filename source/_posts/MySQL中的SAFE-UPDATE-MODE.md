---
title: MySQL中的SAFE UPDATE MODE
date: 2018-11-20 17:20:17
categories:
- MySQL
- 优化
tags:
- MySQL优化
- MySQL
- 优化
---
写在前面：此`SET SQL_SAFE_UPDATES=1`命令可防止代码无WHERE条件更新或删除操作。

在业务中发现一个问题，报错如下：
![](/uploads/2018/11/mysql_safe_update_mode.png)
<!--more-->
在使用 `update` 的时候，报如上错误。 根据图示可发现是因为开启了 update safe mode，然后where条件中的字段没有索引导致的。

经过查找，确实发现，MySQL有SAFE UPDATE MODE模式，可使用SQL语句更改：
`SET SQL_SAFE_UPDATES = 0;`
相当于是解除SAFE MODE模式，则可以更新删除了。

sql_safe_updates参数可以限制不带where条件的update/delete语句执行失败，这个参数设置后，可以防止业务bug/漏洞导致把整个表都更新或者删除（线上发生过的案例），也可以防止DBA在线误操作更新/删除整张表。
>官方解释：
当sql_safe_updates设置为1时，UPDATE :要有where，并查询条件必须使用为索引字段，或者使用limit，或者两个条件同时存在，才能正常执行。DELETE:where条件中带有索引字段可删除，where中查询条件不是索引，得必须有limit。主要是防止UPDATE和DELETE 没有使用索引导致变更及删除大量数据。系统参数默认值为0
<font color="red">（经验证发现，并不是唯一索引才行（网上某些说法如此），只要是含有索引的就OK。）</font>

为了防止线上业务出现以下3种情况影响线上服务的正常使用和不小心全表数据删除:
1. 没有加where条件的全表更新操作
2. 加了where 条件字段，但是where 字段 没有走索引的表更新
3. 全表delete 没有加where 条件 或者where 条件没有 走索引

**建议:** DBA 开启此参数限制 ，可以避免线上业务数据误删除操作，但是需要先在测试库开启，这样可以可以先在测试库上补充短缺的表索引，测试验证没问题再部署到线上库 邮件部从去年开始已经在严选电商线上运行。


**业务逻辑：**
业务中开启SAFE MODE的场景是，由于出现过代码中有更新操作，而没有传where条件，导致全部更新（幸运的是，由于更新数据量过大，更新超时，导致数据并没有被更新成功。否则，数据就得回滚了。）。没有传where条件是因为没有进行过滤验证导致的。所以，DB那边就开启了UPDATE SAFE MODE模式。 
开启这个之后，线上数据库某些表中的字段没有索引，而<font color="red">代码中却使用了没有索引的字段做where条件进行更新。</font>才引发上面的错误。

