---
title: Flask 基础模块 - 数据库之Flask-SQLAlchemy管理数据库
date: 2018-03-05 16:03:11
categories: 
 - Python
 - Flask
tags:
- Flask
- Python
---
Flask-SQLAlchemy 是一个Flask 扩展，简化了在Flask 程序中使用SQLAlchemy 的操作。SQLAlchemy 是一个很强大的关系型数据库框架，支持多种数据库后台。SQLAlchemy 提供了高层ORM，也提供了使用数据库原生SQL 的低层功能。

和其他大多数扩展一样，Flask-SQLAlchemy 也使用pip 安装：
　　`(venv) $ pip install flask-sqlalchemy`
<!--more-->
在Flask-SQLAlchemy 中，数据库使用URL 指定。最流行的数据库引擎采用的数据库URL格式如表5-1 所示。

**表5-1　FLask-SQLAlchemy数据库URL**

数据库引擎        | URL
------------------|---------------------------------------
MySQL             | mysql://username:password@hostname/database
Postgres          | postgresql://username:password@hostname/database
SQLite（Unix）    | sqlite:////absolute/path/to/database
SQLite（Windows） | sqlite:///c:/absolute/path/to/database

在这些URL 中，hostname 表示MySQL 服务所在的主机，可以是本地主机（localhost），也可以是远程服务器。数据库服务器上可以托管多个数据库，因此database 表示要使用的数据库名。如果数据库需要进行认证，username 和password 表示数据库用户密令。
*（注：SQLite 数据库不需要使用服务器，因此不用指定hostname、username 和password。URL 中的database 是硬盘上文件的文件名。）*

程序使用的数据库URL 必须保存到Flask 配置对象的SQLALCHEMY\_DATABASE\_URI 键中。配置对象中还有一个很有用的选项，即SQLALCHEMY\_COMMIT\_ON\_TEARDOWN 键，将其设为True时，每次请求结束后都会自动提交数据库中的变动。其他配置选项的作用请参阅Flask-SQLAlchemy 的文档。示例5-1 展示了如何初始化及配置一个简单的SQLite 数据库。

示例5-1　hello.py：配置数据库
```
from flask.ext.sqlalchemy import SQLAlchemy
basedir = os.path.abspath(os.path.dirname(__file__))
app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///' + os.path.join(basedir, 'data.sqlite')
app.config['SQLALCHEMY_COMMIT_ON_TEARDOWN'] = True
db = SQLAlchemy(app)
```
db 对象是SQLAlchemy 类的实例，表示程序使用的数据库，同时还获得了Flask-SQLAlchemy提供的所有功能。


#### 1、定义模型
模型这个术语表示程序使用的持久化实体。在ORM 中，模型一般是一个Python 类，类中的属性对应数据库表中的列。

Flask-SQLAlchemy 创建的数据库实例为模型提供了一个基类以及一系列辅助类和辅助函数，可用于定义模型的结构。图5-1 中的roles 表和users 表可定义为模型Role 和User，如示例5-2 所示。

示例5-2　hello.py：定义Role 和User 模型
```
class Role(db.Model):
    __tablename__ = 'roles'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(64), unique=True)
    def __repr__(self):
        return '<Role %r>' % self.name
class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), unique=True, index=True)
    def __repr__(self):
        return '<User %r>' % self.username
```
类变量\_\_tablename\_\_ 定义在数据库中使用的表名。如果没有定义\_\_tablename\_\_，Flask-SQLAlchemy 会使用一个默认名字，但默认的表名没有遵守使用复数形式进行命名的约定，所以最好由我们自己来指定表名。其余的类变量都是该模型的属性，被定义为db.Column类的实例。

db.Column 类构造函数的第一个参数是数据库列和模型属性的类型。表5-2 列出了一些可用的列类型以及在模型中使用的Python 类型。

**表5-2　最常用的SQLAlchemy列类型**

类型名       | Python类型         | 说　　明
-------------|--------------------|------------------
Integer      | int                | 普通整数，一般是32 位
SmallInteger | int                | 取值范围小的整数，一般是16 位
BigInteger   | int 或long         | 不限制精度的整数
Float        | float              | 浮点数
Numeric      | decimal.Decimal    | 定点数
String       | str                | 变长字符串
Text         | str                | 变长字符串，对较长或不限长度的字符串做了优化
Unicode      | unicode            | 变长Unicode 字符串
UnicodeText  | unicode            | 变长Unicode 字符串，对较长或不限长度的字符串做了优化
Boolean      | bool               | 布尔值
Date         | datetime.date      | 日期
Time         | datetime.time      | 时间
DateTime     | datetime.datetime  | 日期和时间
Interval     | datetime.timedelta | 时间间隔
Enum         | str                | 一组字符串
PickleType   | 任何Python 对象    | 自动使用Pickle 序列化
LargeBinary  | str                | 二进制文件

db.Column 中其余的参数指定属性的配置选项。表5-3 列出了一些可用选项。

**表5-3　最常使用的SQLAlchemy列选项**

选项名       | 说　　明
-------------|----------------------
primary_key  | 如果设为True，这列就是表的主键
unique       | 如果设为True，这列不允许出现重复的值
index        | 如果设为True，为这列创建索引，提升查询效率
nullable     | 如果设为True，这列允许使用空值；如果设为False，这列不允许使用空值
default      | 为这列定义默认值

**（注：Flask-SQLAlchemy 要求每个模型都要定义主键，这一列经常命名为id。）**

虽然没有强制要求，但这两个模型都定义了\_\_repr()\_\_ 方法，返回一个具有可读性的字符串表示模型，可在调试和测试时使用。


#### 2、关系
关系型数据库使用关系把不同表中的行联系起来。图5-1 所示的关系图表示用户和角色之间的一种简单关系。这是角色到用户的一对多关系，因为一个角色可属于多个用户，而每个用户都只能有一个角色。

图5-1 中的一对多关系在模型类中的表示方法如示例5-3 所示。

示例5-3　hello.py：关系
```
class Role(db.Model):
    # ...
    users = db.relationship('User', backref='role')

class User(db.Model):
    # ...
    role_id = db.Column(db.Integer, db.ForeignKey('roles.id'))
```
如图5-1 所示，关系使用users 表中的外键连接了两行。添加到User 模型中的role_id 列被定义为外键，就是这个外键建立起了关系。传给db.ForeignKey() 的参数'roles.id' 表明，这列的值是roles 表中行的id 值。

添加到Role 模型中的users 属性代表这个关系的面向对象视角。对于一个Role 类的实例，其users 属性将返回与角色相关联的用户组成的列表。db.relationship() 的第一个参数表明这个关系的另一端是哪个模型。如果模型类尚未定义，可使用字符串形式指定。

db.relationship() 中的backref 参数向User 模型中添加一个role 属性，从而定义反向关系。这一属性可替代role\_id 访问Role 模型，此时获取的是模型对象，而不是外键的值。

大多数情况下，db.relationship() 都能自行找到关系中的外键，但有时却无法决定把哪一列作为外键。例如，如果User 模型中有两个或以上的列定义为Role 模型的外键，SQLAlchemy 就不知道该使用哪列。如果无法决定外键，你就要为db.relationship() 提供额外参数，从而确定所用外键。表5-4 列出了定义关系时常用的配置选项。

**表5-4　常用的SQLAlchemy关系选项**

选项名            | 说　　明
------------------|-------------------------------------------
backref           | 在关系的另一个模型中添加反向引用
primaryjoin       | 明确指定两个模型之间使用的联结条件。只在模棱两可的关系中需要指定
lazy              | 指定如何加载相关记录。可选值有select（首次访问时按需加载）、immediate（源对象加载后就加载）、joined（加载记录，但使用联结）、subquery（立即加载，但使用子查询），noload（永不加载）和dynamic（不加载记录，但提供加载记录的查询）
uselist           | 如果设为Fales，不使用列表，而使用标量值
order_by          | 指定关系中记录的排序方式
secondary         | 指定多对多关系中关系表的名字
secondaryjoin     | SQLAlchemy 无法自行决定时，指定多对多关系中的二级联结条件

除了一对多之外，还有几种其他的关系类型。一对一关系可以用前面介绍的一对多关系表示，但调用db.relationship() 时要把uselist 设为False，把“多”变成“一”。多对一关系也可使用一对多表示，对调两个表即可，或者把外键和db.relationship() 都放在“多”这一侧。最复杂的关系类型是多对多，需要用到第三张表，这个表称为关系表。


#### 3、数据库操作
##### （1）创建表
首先，我们要让Flask-SQLAlchemy 根据模型类创建数据库。方法是使用db.create_all()函数：
```
(venv) $ python hello.py shell
>>> from hello import db
>>> db.create_all()
```
如果你查看程序目录，会发现新建了一个名为data.sqlite 的文件。这个SQLite 数据库文件的名字就是在配置中指定的。如果数据库表已经存在于数据库中，那么db.create_all()不会重新创建或者更新这个表。如果修改模型后要把改动应用到现有的数据库中，这一特性会带来不便。更新现有数据库表的粗暴方式是先删除旧表再重新创建：
```
>>> db.drop_all()
>>> db.create_all()
```
（注：遗憾的是，这个方法有个我们不想看到的副作用，它把数据库中原有的数据都销毁了。末尾将会介绍一种更好的方式用于更新数据库。）

##### （2）插入行
下面这段代码创建了一些角色和用户：
```
>>> from hello import Role, User
>>> admin_role = Role(name='Admin')
>>> mod_role = Role(name='Moderator')
>>> user_role = Role(name='User')
>>> user_john = User(username='john', role=admin_role)
>>> user_susan = User(username='susan', role=user_role)
>>> user_david = User(username='david', role=user_role)
```
模型的构造函数接受的参数是使用关键字参数指定的模型属性初始值。注意，role 属性也可使用，虽然它不是真正的数据库列，但却是一对多关系的高级表示。这些新建对象的id属性并没有明确设定，因为主键是由Flask-SQLAlchemy 管理的。现在这些对象只存在于Python 中，还未写入数据库。因此id 尚未赋值：
```
>>> print(admin_role.id)
None
>>> print(mod_role.id)
None
>>> print(user_role.id)
None
```
通过数据库会话管理对数据库所做的改动，在Flask-SQLAlchemy 中，会话由db.session表示。准备把对象写入数据库之前，先要将其添加到会话中：
```
>>> db.session.add(admin_role)
>>> db.session.add(mod_role)
>>> db.session.add(user_role)
>>> db.session.add(user_john)
>>> db.session.add(user_susan)
>>> db.session.add(user_david)
```
或者简写成：
```
>>> db.session.add_all([admin_role, mod_role, user_role,
... user_john, user_susan, user_david])
```
为了把对象写入数据库，我们要调用commit() 方法提交会话：
`>>> db.session.commit()`

再次查看id 属性，现在它们已经赋值了：
```
>>> print(admin_role.id)
1
>>> print(mod_role.id)
2
>>> print(user_role.id)
3
```
数据库会话能保证数据库的一致性。提交操作使用原子方式把会话中的对象全部写入数据库。如果在写入会话的过程中发生了错误，整个会话都会失效。如果你始终把相关改动放在会话中提交，就能避免因部分更新导致的数据库不一致性。

##### （3）修改行
在数据库会话上调用add() 方法也能更新模型。我们继续在之前的shell 会话中进行操作，下面这个例子把"Admin" 角色重命名为"Administrator"：
```
>>> admin_role.name = 'Administrator'
>>> db.session.add(admin_role)
>>> db.session.commit()
```

##### （4）删除行
数据库会话还有个delete() 方法。下面这个例子把"Moderator" 角色从数据库中删除：
```
>>> db.session.delete(mod_role)
>>> db.session.commit()
```
注意，删除与插入和更新一样，提交数据库会话后才会执行。

##### （5）查询行
Flask-SQLAlchemy 为每个模型类都提供了query 对象。最基本的模型查询是取回对应表中的所有记录：
```
>>> Role.query.all()
[<Role u'Administrator'>, <Role u'User'>]
>>> User.query.all()
[<User u'john'>, <User u'susan'>, <User u'david'>]
```

使用过滤器可以配置query 对象进行更精确的数据库查询。下面这个例子查找角色为"User" 的所有用户：
```
>>> User.query.filter_by(role=user_role).all()
[<User u'susan'>, <User u'david'>]
```

若要查看SQLAlchemy 为查询生成的原生SQL 查询语句，只需把query 对象转换成字符串：
```
>>> str(User.query.filter_by(role=user_role))
'SELECT users.id AS users_id, users.username AS users_username,
users.role_id AS users_role_id FROM users WHERE :param_1 = users.role_id'
```
filter_by() 等过滤器在query 对象上调用，返回一个更精确的query 对象。多个过滤器可以一起调用，直到获得所需结果。

表5-5 列出了可在query 对象上调用的常用过滤器。完整的列表参见SQLAlchemy 文档（http://docs.sqlalchemy.org）。

**表5-5　常用的SQLAlchemy查询过滤器**

过滤器        | 说　　明
--------------|-----------------------------------
filter()      | 把过滤器添加到原查询上，返回一个新查询
filter_by()   | 把等值过滤器添加到原查询上，返回一个新查询
limit()       | 使用指定的值限制原查询返回的结果数量，返回一个新查询
offset()      | 偏移原查询返回的结果，返回一个新查询
order_by()    | 根据指定条件对原查询结果进行排序，返回一个新查询
group_by()    | 根据指定条件对原查询结果进行分组，返回一个新查询

在查询上应用指定的过滤器后，通过调用all() 执行查询，以列表的形式返回结果。除了all() 之外，还有其他方法能触发查询执行。表5-6 列出了执行查询的其他方法。

**表5-6　最常使用的SQLAlchemy查询执行函数**

方　法          | 说　　明
----------------|-------------------------------------
all()           | 以列表形式返回查询的所有结果
first()         | 返回查询的第一个结果，如果没有结果，则返回None
first_or_404()  | 返回查询的第一个结果，如果没有结果，则终止请求，返回404 错误响应
get()           | 返回指定主键对应的行，如果没有对应的行，则返回None
get_or_404()    | 返回指定主键对应的行，如果没找到指定的主键，则终止请求，返回404 错误响应
count()         | 返回查询结果的数量
paginate()      | 返回一个Paginate 对象，它包含指定范围内的结果

关系和查询的处理方式类似。下面这个例子分别从关系的两端查询角色和用户之间的一对多关系：
```
>>> users = user_role.users
>>> users
[<User u'susan'>, <User u'david'>]
>>> users[0].role
<Role u'User'>
```
这个例子中的user_role.users 查询有个小问题。执行user_role.users 表达式时，隐含的查询会调用all() 返回一个用户列表。query 对象是隐藏的，因此无法指定更精确的查询过滤器。就这个特定示例而言，返回一个按照字母顺序排序的用户列表可能更好。如果修改了关系的设置，加入了lazy = 'dynamic' 参数，从而禁止自动执行查询。


#### 4、使用Flask-Migrate实现数据库迁移
在开发程序的过程中，你会发现有时需要修改数据库模型，而且修改之后还需要更新数据库。

仅当数据库表不存在时，Flask-SQLAlchemy 才会根据模型进行创建。因此，更新表的唯一方式就是先删除旧表，不过这样做会丢失数据库中的所有数据。

更新表的更好方法是使用数据库迁移框架。源码版本控制工具可以跟踪源码文件的变化，类似地，数据库迁移框架能跟踪数据库模式的变化，然后增量式的把变化应用到数据库中。

SQLAlchemy 的主力开发人员编写了一个迁移框架，称为Alembic（https://alembic.readthedocs.org/en/latest/index.html）。除了直接使用Alembic 之外，Flask 程序还可使用Flask-Migrate（http://flask-migrate.readthedocs.org/en/latest/）扩展。这个扩展对Alembic 做了轻量级包装，并集成到Flask-Script 中，所有操作都通过Flask-Script 命令完成。

##### （1）创建迁移仓库
首先，我们要在虚拟环境中安装Flask-Migrate：
　　`(venv) $ pip install flask-migrate`
这个扩展的初始化方法如示例5-8 所示。

示例5-8　hello.py：配置Flask-Migrate
```
from flask.ext.migrate import Migrate, MigrateCommand
# ...
migrate = Migrate(app, db)
manager.add_command('db', MigrateCommand)
```
为了导出数据库迁移命令，Flask-Migrate 提供了一个MigrateCommand 类，可附加到Flask-Script 的manager 对象上。在这个例子中，MigrateCommand 类使用db 命令附加。

在维护数据库迁移之前，要使用init 子命令创建迁移仓库：
```
(venv) $ python hello.py db init
Creating directory /home/flask/flasky/migrations...done
Creating directory /home/flask/flasky/migrations/versions...done
Generating /home/flask/flasky/migrations/alembic.ini...done
Generating /home/flask/flasky/migrations/env.py...done
Generating /home/flask/flasky/migrations/env.pyc...done
Generating /home/flask/flasky/migrations/README...done
Generating /home/flask/flasky/migrations/script.py.mako...done
Please edit configuration/connection/logging settings in
'/home/flask/flasky/migrations/alembic.ini' before proceeding.
```
这个命令会创建migrations 文件夹，所有迁移脚本都存放其中。

##### （2）创建迁移脚本
在Alembic 中，数据库迁移用迁移脚本表示。脚本中有两个函数，分别是upgrade() 和downgrade()。upgrade() 函数把迁移中的改动应用到数据库中，downgrade() 函数则将改动删除。Alembic 具有添加和删除改动的能力，因此数据库可重设到修改历史的任意一点。

我们可以使用revision 命令手动创建Alembic 迁移，也可使用migrate 命令自动创建。手动创建的迁移只是一个骨架，upgrade() 和downgrade() 函数都是空的，开发者要使用Alembic 提供的Operations 对象指令实现具体操作。自动创建的迁移会根据模型定义和数据库当前状态之间的差异生成upgrade() 和downgrade() 函数的内容。

migrate 子命令用来自动创建迁移脚本：
```
(venv) $ python hello.py db migrate -m "initial migration"
INFO [alembic.migration] Context impl SQLiteImpl.
INFO [alembic.migration] Will assume non-transactional DDL.
INFO [alembic.autogenerate] Detected added table 'roles'
INFO [alembic.autogenerate] Detected added table 'users'
INFO [alembic.autogenerate.compare] Detected added index
'ix_users_username' on '['username']'
Generating /home/flask/flasky/migrations/versions/1bc
594146bb5_initial_migration.py...done
```

##### （3）更新数据库
检查并修正好迁移脚本之后，我们可以使用db upgrade 命令把迁移应用到数据库中：
```
(venv) $ python hello.py db upgrade
INFO [alembic.migration] Context impl SQLiteImpl.
INFO [alembic.migration] Will assume non-transactional DDL.
INFO [alembic.migration] Running upgrade None -> 1bc594146bb5, initial migration
```

对第一个迁移来说， 其作用和调用db.create_all() 方法一样。但在后续的迁移中，upgrade 命令能把改动应用到数据库中，且不影响其中保存的数据。