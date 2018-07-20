---
title: Python3 支持MySQL
date: 2018-04-16 13:30:05
categories:
- Python
- Flask
tags:
- Flask
- Python
---
><font color="red">注：MySQLdb 只适用于Python2.x，在Python3的替代品是：pymysql</font>

1、安装： pip install pymysql

一些框架默认仍然用的是MySQLdb，但是python3已经不支持MySQLdb，取而代之的是pymysql，因此运行的时候会报 
`ModuleNotFoundError: No module named 'MySQLdb'`

2、使用方式：
1. 安装成功后，可使用MySQLdb的语法使用pymysql
2. 使用Flask集成的SQLAlchemy管理MySQL

<!--more-->

在框架中引入下面语句：
```
import pymysql
pymysql.install_as_MySQLdb()
```
然后，在配置 `SQLALCHEMY_DATABASE_URI` 参数时，直接使用 mysql的使用方式即可。
>使用方式：`mysql://username:password@hostname/database`

3、生成对应数据表
```
python manage.py db init
python manage.py db migrate
python manage.py db upgrade
```