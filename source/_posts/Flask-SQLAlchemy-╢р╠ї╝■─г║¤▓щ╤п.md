---
title: Flask - SQLAlchemy 多条件模糊查询
date: 2018-04-23 18:53:25
categories:
- Python
- Flask
tags:
- Flask
- Python
---
在做项目的时候，总会遇到搜索的需求，平常写SQL语句很简单。但是，在Flask 使用SQLAlchemy管理数据库的时候，应该如何使用，做了一下笔记，如下。

#### 一、使用SQLALchemy
一般的搜索都需要模糊查询，如果存在多条查询的需求，可如下操作：
`users = User.query.filter(User.name.like("%"+搜索的内容+"%"), User.age.like("%"+搜索的内容+"%")).all()`

此种方式有一个弊端，就是不能 “or” 查询，只能 “and” 查询。
进行 “or” 查询，我的一个做法就是只能分开查询，然后合并数据即可。
<!--more-->

#### 二、使用原生SQL操作
使用 session会话执行 SQL语句。（注：还有一种支持线程安全的方式，可使用 `scoped_session()`实现）
```
from app import db

sql = "SELECT * FROM `moviecol` \
        LEFT JOIN `movie` ON moviecol.movie_id=movie.id \
        LEFT JOIN `user` ON moviecol.user_id=user.id \
        WHERE movie.title like '%"+keywords+"%' or user.name like '%"+keywords+"%' \
        ORDER BY moviecol.addtime DESC \
        LIMIT "+str(page)+","+str(page_config['moviecol_per_page'])
res = db.session.execute(sql).fetchall()
```
其中 Session类的相关方法如下：
```
public_methods = (
    '__contains__', '__iter__', 'add', 'add_all', 'begin', 'begin_nested',
    'close', 'commit', 'connection', 'delete', 'execute', 'expire',
    'expire_all', 'expunge', 'expunge_all', 'flush', 'get_bind',
    'is_modified', 'bulk_save_objects', 'bulk_insert_mappings',
    'bulk_update_mappings',
    'merge', 'query', 'refresh', 'rollback',
    'scalar')
```