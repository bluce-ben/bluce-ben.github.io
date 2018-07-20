---
title: Flask 基础模块 - 大型程序的结构
date: 2018-03-05 18:04:46
categories: 
 - Python
 - Flask
tags:
- Flask
- Python
---
尽管在单一脚本中编写小型Web 程序很方便，但这种方法并不能广泛使用。程序变复杂后，使用单个大型源码文件会导致很多问题。

不同于大多数其他的Web 框架，Flask 并不强制要求大型项目使用特定的组织方式，程序结构的组织方式完全由开发者决定。在本章，我们将介绍一种使用包和模块组织大型程序的方式。本书后续示例都将采用这种结构。

#### 1、项目结构
Flask 程序的基本结构如示例7-1 所示。
<!--more-->
**示例7-1　多文件Flask 程序的基本结构**
```
|-flasky
　　|-app/
　　　　|-templates/
　　　　|-static/
　　　　|-main/
　　　　　　|-__init__.py
　　　　　　|-errors.py
　　　　　　|-forms.py
　　　　　　|-views.py
　　　　|-__init__.py
　　　　|-email.py
　　　　|-models.py
　　|-migrations/
　　|-tests/
　　　　|-__init__.py
　　　　|-test*.py
　　|-venv/
　　|-requirements.txt
　　|-config.py
　　|-manage.py
```
这种结构有4 个顶级文件夹：
* Flask程序一般都保存在名为app 的包中；
* 和之前一样，migrations文件夹包含数据库迁移脚本；
* 单元测试编写在 tests包中；
* 和之前一样，venv 文件夹包含 Python 虚拟环境。

同时还创建了一些新文件：
* requirements.txt列出了所有依赖包，便于在其他电脑中重新生成相同的虚拟环境；
* config.py 存储配置；
* manage.py用于启动程序以及其他的程序任务。

为了帮助你完全理解这个结构，下面几节讲解把hello.py 程序转换成这种结构的过程。

#### 2、配置选项
程序经常需要设定多个配置。这方面最好的例子就是开发、测试和生产环境要使用不同的数据库，这样才不会彼此影响。

我们不再使用hello.py 中简单的字典状结构配置，而使用层次结构的配置类。config.py 文件的内容如示例7-2 所示。

示例7-2　config.py：程序的配置
```
import os
basedir = os.path.abspath(os.path.dirname(__file__))

class Config:
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'hard to guess string'
    SQLALCHEMY_COMMIT_ON_TEARDOWN = True
    FLASKY_MAIL_SUBJECT_PREFIX = '[Flasky]'
    FLASKY_MAIL_SENDER = 'Flasky Admin <flasky@example.com>'
    FLASKY_ADMIN = os.environ.get('FLASKY_ADMIN')

    @staticmethod
    def init_app(app):
        pass

class DevelopmentConfig(Config):
    DEBUG = True
    MAIL_SERVER = 'smtp.googlemail.com'
    MAIL_PORT = 587
    MAIL_USE_TLS = True
    MAIL_USERNAME = os.environ.get('MAIL_USERNAME')
    MAIL_PASSWORD = os.environ.get('MAIL_PASSWORD')
    SQLALCHEMY_DATABASE_URI = os.environ.get('DEV_DATABASE_URL') or \
        'sqlite:///' + os.path.join(basedir, 'data-dev.sqlite')

class TestingConfig(Config):
    TESTING = True
    SQLALCHEMY_DATABASE_URI = os.environ.get('TEST_DATABASE_URL') or \
        'sqlite:///' + os.path.join(basedir, 'data-test.sqlite')

class ProductionConfig(Config):
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
        'sqlite:///' + os.path.join(basedir, 'data.sqlite')

config = {
    'development': DevelopmentConfig,
    'testing': TestingConfig,
    'production': ProductionConfig,
    'default': DevelopmentConfig
}
```
基类Config 中包含通用配置，子类分别定义专用的配置。如果需要，你还可添加其他配置类。

为了让配置方式更灵活且更安全，某些配置可以从环境变量中导入。例如，SECRET\_KEY 的值，这是个敏感信息，可以在环境中设定，但系统也提供了一个默认值，以防环境中没有定义。

在3 个子类中，SQLALCHEMY\_DATABASE\_URI 变量都被指定了不同的值。这样程序就可在不同的配置环境中运行，每个环境都使用不同的数据库。

配置类可以定义init\_app() 类方法，其参数是程序实例。在这个方法中，可以执行对当前环境的配置初始化。现在，基类Config 中的init\_app() 方法为空。

在这个配置脚本末尾，config 字典中注册了不同的配置环境，而且还注册了一个默认配置。


#### 3、程序包
程序包用来保存程序的所有代码、模板和静态文件。我们可以把这个包直接称为app（应用），如果有需求，也可使用一个程序专用名字。templates 和static 文件夹是程序包的一部分，因此这两个文件夹被移到了app 中。数据库模型和电子邮件支持函数也被移到了这个包中，分别保存为app/models.py 和app/email.py。

##### （1）使用程序工厂函数
在单个文件中开发程序很方便，但却有个很大的缺点，因为程序在全局作用域中创建，所以无法动态修改配置。运行脚本时，程序实例已经创建，再修改配置为时已晚。这一点对单元测试尤其重要，因为有时为了提高测试覆盖度，必须在不同的配置环境中运行程序。这个问题的解决方法是延迟创建程序实例，把创建过程移到可显式调用的工厂函数中。这种方法不仅可以给脚本留出配置程序的时间，还能够创建多个程序实例，这些实例有时在测试中非常有用。程序的工厂函数在app 包的构造文件中定义，如示例7-3 所示。

构造文件导入了大多数正在使用的Flask 扩展。由于尚未初始化所需的程序实例，所以没有初始化扩展，创建扩展类时没有向构造函数传入参数。create\_app() 函数就是程序的工厂函数，接受一个参数，是程序使用的配置名。配置类在config.py 文件中定义，其中保存的配置可以使用Flask app.config 配置对象提供的from\_object() 方法直接导入程序。至于配置对象，则可以通过名字从config 字典中选择。程序创建并配置好后，就能初始化扩展了。在之前创建的扩展对象上调用init\_app() 可以完成初始化过程。

示例7-3　app/\_\_init\_\_.py：程序包的构造文件
```
from flask import Flask, render_template
from flask.ext.bootstrap import Bootstrap
from flask.ext.mail import Mail
from flask.ext.moment import Moment
from flask.ext.sqlalchemy import SQLAlchemy
from config import config

bootstrap = Bootstrap()
mail = Mail()
moment = Moment()
db = SQLAlchemy()

def create_app(config_name):
    app = Flask(__name__)
    app.config.from_object(config[config_name])
    config[config_name].init_app(app)

    bootstrap.init_app(app)
    mail.init_app(app)
    moment.init_app(app)
    db.init_app(app)

    # 附加路由和自定义的错误页面

    return app
```

工厂函数返回创建的程序示例，不过要注意，现在工厂函数创建的程序还不完整，因为没
有路由和自定义的错误页面处理程序。

##### （2）在蓝本中实现程序功能
转换成程序工厂函数的操作让定义路由变复杂了。在单脚本程序中，程序实例存在于全局作用域中，路由可以直接使用app.route 修饰器定义。但现在程序在运行时创建，只有调用create\_app() 之后才能使用app.route 修饰器，这时定义路由就太晚了。和路由一样，自定义的错误页面处理程序也面临相同的困难，因为错误页面处理程序使用app.errorhandler 修饰器定义。

幸好Flask 使用蓝本提供了更好的解决方法。蓝本和程序类似，也可以定义路由。不同的是，在蓝本中定义的路由处于休眠状态，直到蓝本注册到程序上后，路由才真正成为程序的一部分。使用位于全局作用域中的蓝本时，定义路由的方法几乎和单脚本程序一样。

和程序一样，蓝本可以在单个文件中定义，也可使用更结构化的方式在包中的多个模块中创建。为了获得最大的灵活性，程序包中创建了一个子包，用于保存蓝本。示例7-4 是这个子包的构造文件，蓝本就创建于此。

示例7-4　app/main/\_\_init\_\_.py：创建蓝本
```
from flask import Blueprint
main = Blueprint('main', __name__)
from . import views, errors
```
通过实例化一个Blueprint 类对象可以创建蓝本。这个构造函数有两个必须指定的参数：蓝本的名字和蓝本所在的包或模块。和程序一样，大多数情况下第二个参数使用Python 的\_\_name\_\_ 变量即可。

程序的路由保存在包里的app/main/views.py 模块中，而错误处理程序保存在app/main/errors.py 模块中。导入这两个模块就能把路由和错误处理程序与蓝本关联起来。注意，这些模块在app/main/\_\_init\_\_.py 脚本的末尾导入，这是为了避免循环导入依赖，因为在views.py 和errors.py 中还要导入蓝本main。

蓝本在工厂函数create\_app() 中注册到程序上，如示例7-5 所示。
示例7-5　app/\_\_init\_\_.py：注册蓝本
```
def create_app(config_name):
    # ...
    from .main import main as main_blueprint
    app.register_blueprint(main_blueprint)
    
    return app
```

示例7-6 显示了错误处理程序。
示例7-6　app/main/errors.py：蓝本中的错误处理程序
```
from flask import render_template
from . import main

@main.app_errorhandler(404)
def page_not_found(e):
    return render_template('404.html'), 404

@main.app_errorhandler(500)
def internal_server_error(e):
    return render_template('500.html'), 500
```
在蓝本中编写错误处理程序稍有不同，如果使用errorhandler 修饰器，那么只有蓝本中的错误才能触发处理程序。要想注册程序全局的错误处理程序，必须使用app\_errorhandler。

在蓝本中定义的程序路由如示例7-7 所示。
示例7-7　app/main/views.py：蓝本中定义的程序路由
```
from datetime import datetime
from flask import render_template, session, redirect, url_for
from . import main
from .forms import NameForm
from .. import db
from ..models import User

@main.route('/', methods=['GET', 'POST'])
def index():
    form = NameForm()
    if form.validate_on_submit():
        # ...
        return redirect(url_for('.index'))
    return render_template('index.html', form=form, name=session.get('name'),
        known=session.get('known', False), current_time=datetime.utcnow())
```
在蓝本中编写视图函数主要有两点不同：第一，和前面的错误处理程序一样，路由修饰器由蓝本提供；第二，url\_for() 函数的用法不同。你可能还记得，url\_for() 函数的第一个参数是路由的端点名，在程序的路由中，默认为视图函数的名字。例如，在单脚本程序中，index() 视图函数的URL 可使用url\_for('index') 获取。

在蓝本中就不一样了，Flask 会为蓝本中的全部端点加上一个命名空间，这样就可以在不同的蓝本中使用相同的端点名定义视图函数，而不会产生冲突。命名空间就是蓝本的名字（Blueprint 构造函数的第一个参数），所以视图函数index() 注册的端点名是main.index，其URL 使用url\_for('main.index') 获取。

url\_for() 函数还支持一种简写的端点形式，在蓝本中可以省略蓝本名，例如url\_for('.index')。在这种写法中，命名空间是当前请求所在的蓝本。这意味着同一蓝本中的重定向可以使用简写形式，但跨蓝本的重定向必须使用带有命名空间的端点名。

为了完全修改程序的页面，表单对象也要移到蓝本中，保存于app/main/forms.py 模块。


#### 4、启动脚本
顶级文件夹中的manage.py 文件用于启动程序。脚本内容如示例7-8 所示。
示例7-8　manage.py：启动脚本
```
#!/usr/bin/env python
import os
from app import create_app, db
from app.models import User, Role
from flask.ext.script import Manager, Shell
from flask.ext.migrate import Migrate, MigrateCommand

app = create_app(os.getenv('FLASK_CONFIG') or 'default')

manager = Manager(app)
migrate = Migrate(app, db)

def make_shell_context():
    return dict(app=app, db=db, User=User, Role=Role)
manager.add_command("shell", Shell(make_context=make_shell_context))

manager.add_command('db', MigrateCommand)

if __name__ == '__main__':
    manager.run()
```
这个脚本先创建程序。如果已经定义了环境变量FLASK\_CONFIG，则从中读取配置名；否则使用默认配置。然后初始化Flask-Script、Flask-Migrate 和为Python shell 定义的上下文。出于便利，脚本中加入了shebang 声明，所以在基于Unix 的操作系统中可以通过./manage.py 执行脚本，而不用使用复杂的python manage.py。


#### 5、需求文件
程序中必须包含一个requirements.txt 文件，用于记录所有依赖包及其精确的版本号。如果要在另一台电脑上重新生成虚拟环境，这个文件的重要性就体现出来了，例如部署程序时使用的电脑。pip 可以使用如下命令自动生成这个文件：
　　`(venv) $ pip freeze >requirements.txt`

安装或升级包后，最好更新这个文件。需求文件的内容示例如下：
>Flask==0.10.1
Flask-Bootstrap==3.0.3.1
Flask-Mail==0.9.0
Flask-Migrate==1.1.0
Flask-Moment==0.2.0
Flask-SQLAlchemy==1.0
Flask-Script==0.6.6
Flask-WTF==0.9.4
Jinja2==2.7.1
Mako==0.9.1
MarkupSafe==0.18
SQLAlchemy==0.8.4
WTForms==1.0.5
Werkzeug==0.9.4
alembic==0.6.2
blinker==1.3
itsdangerous==0.23

如果你要创建这个虚拟环境的完全副本，可以创建一个新的虚拟环境，并在其上运行以下命令：
　　`(venv) $ pip install -r requirements.txt`

当你阅读本书时，该示例requirements.txt 文件中的版本号可能已经过期了。如果愿意，你可以试着使用这些包的最新版。如果遇到问题，你可以随时换回这个需求文件中的版本，因为这些版本和程序兼容。


#### 6、单元测试
这个程序很小，所以没什么可测试的。不过为了演示，我们可以编写两个简单的测试，如示例7-9 所示。

示例7-9　tests/test\_basics.py：单元测试
```
import unittest
from flask import current_app
from app import create_app, db

class BasicsTestCase(unittest.TestCase):
    def setUp(self):
        self.app = create_app('testing')
        self.app_context = self.app.app_context()
        self.app_context.push()
        db.create_all()
    
    def tearDown(self):
        db.session.remove()
        db.drop_all()
        self.app_context.pop()

    def test_app_exists(self):
        self.assertFalse(current_app is None)
        def test_app_is_testing(self):
        self.assertTrue(current_app.config['TESTING'])
```
这个测试使用Python 标准库中的unittest 包编写。setUp() 和tearDown() 方法分别在各测试前后运行，并且名字以test\_ 开头的函数都作为测试执行。

setUp() 方法尝试创建一个测试环境，类似于运行中的程序。首先，使用测试配置创建程序，然后激活上下文。这一步的作用是确保能在测试中使用current\_app，像普通请求一样。然后创建一个全新的数据库，以备不时之需。数据库和程序上下文在tearDown() 方法中删除。

第一个测试确保程序实例存在。第二个测试确保程序在测试配置中运行。若想把tests 文件夹作为包使用，需要添加tests/\_\_init\_\_.py 文件，不过这个文件可以为空，因为unittest 包会扫描所有模块并查找测试。

为了运行单元测试，你可以在manage.py 脚本中添加一个自定义命令。示例7-10 展示了如何添加test 命令。

示例7-10　manage.py：启动单元测试的命令
```
@manager.command
def test():
    """Run the unit tests."""
    import unittest
    tests = unittest.TestLoader().discover('tests')
    unittest.TextTestRunner(verbosity=2).run(tests)
```
manager.command 修饰器让自定义命令变得简单。修饰函数名就是命令名，函数的文档字符串会显示在帮助消息中。test() 函数的定义体中调用了unittest 包提供的测试运行函数。

单元测试可使用下面的命令运行：
```
(venv) $ python manage.py test
test_app_exists (test_basics.BasicsTestCase) ... ok
test_app_is_testing (test_basics.BasicsTestCase) ... ok
.----------------------------------------------------------------------
Ran 2 tests in 0.001s
OK
```


#### 7、创建数据库
重组后的程序和单脚本版本使用不同的数据库。

首选从环境变量中读取数据库的URL，同时还提供了一个默认的SQLite 数据库做备用。3种配置环境中的环境变量名和SQLite 数据库文件名都不一样。例如，在开发环境中，数据库URL 从环境变量DEV_DATABASE_URL 中读取，如果没有定义这个环境变量，则使用名为data-dev.sqlite 的SQLite 数据库。

不管从哪里获取数据库URL，都要在新数据库中创建数据表。如果使用Flask-Migrate 踪迁移，可使用如下命令创建数据表或者升级到最新修订版本：
　　`(venv) $ python manage.py db upgrade`