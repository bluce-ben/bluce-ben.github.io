---
title: Flask - 认证扩展
date: 2018-03-06 18:43:25
categories: 
 - Python
 - Flask
tags:
- Flask
- Python
---
Flask-Login：管理已登录用户的用户会话。
Werkzeug：计算密码散列值并进行核对。
itsdangerous：生成并核对加密安全令牌。

#### 1、密码安全性 - 使用Werkzeug实现密码散列
众所周知，大多数用户都在不同的网站中使用相同的密码，因此，即便不保存任何敏感信息，攻击者获得存储在数据库中的密码之后，也能访问用户在其他网站中的账户。
<!--more-->
若想保证数据库中用户密码的安全，关键在于不能存储密码本身，而要存储密码的散列值。计算密码散列值的函数接收密码作为输入，使用一种或多种加密算法转换密码，最终得到一个和原始密码没有关系的字符序列。核对密码时，密码散列值可代替原始密码，因为计算散列值的函数是可复现的：只要输入一样，结果就一样。

Werkzeug 中的security 模块能够很方便地实现密码散列值的计算。这一功能的实现只需要两个函数，分别用在注册用户和验证用户阶段。
* generate_password_hash(password, method=pbkdf2:sha1, salt_length=8)：这个函数将原始密码作为输入，以字符串形式输出密码的散列值，输出的值可保存在用户数据库中。method 和salt_length 的默认值就能满足大多数需求。
* check_password_hash(hash, password)：这个函数的参数是从数据库中取回的密码散列值和用户输入的密码。返回值为True 表明密码正确。

示例8-1 展示了创建的User 模型为支持密码散列所做的改动。
示例8-1　app/models.py：在User 模型中加入密码散列
```
from werkzeug.security import generate_password_hash, check_password_hash
class User(db.Model):
    __tablename__ == 'users'
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), unique=True, index=True)
    password_hash = db.Column(db.String(128))

    @property
    def password(self):
        raise AttributeError('password is not a readable attribute')

    @password.setter
    def password(self, password):
        self.password_hash = generate_password_hash(password)
    
    def verify_password(self, password):
        return check_password_hash(self.password_hash, password)
    
    def __repr__(self):
        return '<User %r>' % self.username
```
计算密码散列值的函数通过名为password 的只写属性实现。设定这个属性的值时，赋值方法会调用Werkzeug 提供的generate_password_hash() 函数，并把得到的结果赋值给password_hash 字段。如果试图读取password 属性的值，则会返回错误，原因很明显，因为生成散列值后就无法还原成原来的密码了。

verify_password 方法接受一个参数（ 即密码）， 将其传给Werkzeug 提供的check_password_hash() 函数，和存储在User 模型中的密码散列值进行比对。如果这个方法返回True，就表明密码是正确的。


#### 2、使用Flask-Login认证用户
用户登录程序后，他们的认证状态要被记录下来，这样浏览不同的页面时才能记住这个状态。Flask-Login 是个非常有用的小型扩展，专门用来管理用户认证系统中的认证状态，且不依赖特定的认证机制。

##### （1）准备用于登录的用户模型
要想使用Flask-Login 扩展，程序的User 模型必须实现几个方法。需要实现的方法如表8-1所示。
表8-1　Flask-Login要求实现的用户方法

方　　法              | 说　　明
----------------------|-------------------------------------------
is_authenticated()    | 如果用户已经登录，必须返回True，否则返回False
is_active()           | 如果允许用户登录，必须返回True，否则返回False。如果要禁用账户，可以返回False
is_anonymous()        | 对普通用户必须返回False
get_id()              | 必须返回用户的唯一标识符，使用Unicode 编码字符串

这4 个方法可以在模型类中作为方法直接实现，不过还有一种更简单的替代方案。Flask-Login 提供了一个UserMixin 类，其中包含这些方法的默认实现，且能满足大多数需求。修改后的User 模型如示例8-6 所示。

示例8-6　app/models.py：修改User 模型，支持用户登录
```
from flask.ext.login import UserMixin
class User(UserMixin, db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key = True)
    email = db.Column(db.String(64), unique=True, index=True)
    username = db.Column(db.String(64), unique=True, index=True)
    password_hash = db.Column(db.String(128))
```
注意，示例中同时还添加了email 字段。在这个程序中，用户使用电子邮件地址登录，因为相对于用户名而言，用户更不容易忘记自己的电子邮件地址。

Flask-Login 在程序的工厂函数中初始化，如示例8-7 所示。
示例8-7　app/__init__.py：初始化Flask-Login
```
from flask.ext.login import LoginManager

login_manager = LoginManager()
login_manager.session_protection = 'strong'
login_manager.login_view = 'auth.login'

def create_app(config_name):
    # ...
    login_manager.init_app(app)
    # ...
```
LoginManager 对象的session_protection 属性可以设为None、'basic' 或'strong'，以提供不同的安全等级防止用户会话遭篡改。设为'strong' 时，Flask-Login 会记录客户端IP地址和浏览器的用户代理信息，如果发现异动就登出用户。login_view 属性设置登录页面的端点。如果登录路由在蓝本中定义，因此要在前面加上蓝本的名字。

最后，Flask-Login 要求程序实现一个回调函数，使用指定的标识符加载用户。这个函数的定义如示例8-8 所示。

示例8-8　app/models.py：加载用户的回调函数
```
from . import login_manager

@login_manager.user_loader
def load_user(user_id):
    return User.query.get(int(user_id))
```
加载用户的回调函数接收以Unicode 字符串形式表示的用户标识符。如果能找到用户，这个函数必须返回用户对象；否则应该返回None。

##### （2）保护路由
为了保护路由只让认证用户访问，Flask-Login 提供了一个login_required 修饰器。用法演示如下：
```
from flask.ext.login import login_required

@app.route('/secret')
@login_required
def secret():
    return 'Only authenticated users are allowed!'
```
如果未认证的用户访问这个路由，Flask-Login 会拦截请求，把用户发往登录页面。


#### 3、使用itsdangerous生成确认令牌
为验证电子邮件地址，用户注册后，程序会立即发送一封确认邮件。新账户先被标记成待确认状态，用户按照邮件中的说明操作后，才能证明自己可以被联系上。账户确认过程中，往往会要求用户点击一个包含确认令牌的特殊URL 链接。

确认邮件中最简单的确认链接是http://www.example.com/auth/confirm/<id> 这种形式的URL，其中id 是数据库分配给用户的数字id。用户点击链接后，处理这个路由的视图函数就将收到的用户id 作为参数进行确认，然后将用户状态更新为已确认。

但这种实现方式显然不是很安全，只要用户能判断确认链接的格式，就可以随便指定URL中的数字，从而确认任意账户。解决方法是把URL 中的id 换成将相同信息安全加密后得到的令牌。

下面这个简短的shell 会话显示了如何使用itsdangerous 包生成包含用户id 的安全令牌：
```
(venv) $ python manage.py shell
>>> from manage import app
>>> from itsdangerous import TimedJSONWebSignatureSerializer as Serializer
>>> s = Serializer(app.config['SECRET_KEY'], expires_in = 3600)
>>> token = s.dumps({ 'confirm': 23 })
>>> token
'eyJhbGciOiJIUzI1NiIsImV4cCI6MTM4MTcxODU1OCwiaWF0IjoxMzgxNzE0OTU4fQ.ey ...'
>>> data = s.loads(token)
>>> data
{u'confirm': 23}
```
itsdangerous 提供了多种生成令牌的方法。其中，TimedJSONWebSignatureSerializer 类生成具有过期时间的JSON Web 签名（JSON Web Signatures，JWS）。这个类的构造函数接收的参数是一个密钥，在Flask 程序中可使用SECRET_KEY 设置。

dumps() 方法为指定的数据生成一个加密签名，然后再对数据和签名进行序列化，生成令牌字符串。expires_in 参数设置令牌的过期时间，单位为秒。

为了解码令牌，序列化对象提供了loads() 方法，其唯一的参数是令牌字符串。这个方法会检验签名和过期时间，如果通过，返回原始数据。如果提供给loads() 方法的令牌不正确或过期了，则抛出异常。