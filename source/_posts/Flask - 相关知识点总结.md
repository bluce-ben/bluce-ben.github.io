---
title: Flask - 相关知识点总结
date: 2018-03-06 18:46:40
categories: 
 - Python
 - Flask
tags:
- Flask
- Python
---
#### 1、Flask入口文件集成 shell
每次启动shell 会话都要导入数据库实例和模型，这真是份枯燥的工作。为了避免一直重复导入，我们可以做些配置，让Flask-Script 的shell 命令自动导入特定的对象。

若想把对象添加到导入列表中，我们要为shell 命令注册一个make\_context 回调函数，如示例5-7 所示。
<!--more-->
示例5-7　hello.py：为shell 命令添加一个上下文
```
from flask.ext.script import Shell
def make_shell_context():
    return dict(app=app, db=db, User=User, Role=Role)
manager.add_command("shell", Shell(make_context=make_shell_context))
```
make\_shell\_context() 函数注册了程序、数据库实例以及模型，因此这些对象能直接导入shell：
```
$ python hello.py shell
>>> app
<Flask 'app'>
>>> db
<SQLAlchemy engine='sqlite:////home/flask/flasky/data.sqlite'>
>>> User
<class 'app.User'>
```


#### 2、Flash消息
请求完成后，有时需要让用户知道状态发生了变化。这里可以使用确认消息、警告或者错误提醒。一个典型例子是，用户提交了有一项错误的登录表单后，服务器发回的响应重新渲染了登录表单，并在表单上面显示一个消息，提示用户用户名或密码错误。

这种功能是Flask 的核心特性。如示例所示，flash() 函数可实现这种效果。
```
from flask import Flask, render_template, session, redirect, url_for, flash
@app.route('/', methods=['GET', 'POST'])
def index():
	form = NameForm()
	if form.validate_on_submit():
	old_name = session.get('name')
	if old_name is not None and old_name != form.name.data:
		flash('Looks like you have changed your name!')
		session['name'] = form.name.data
		return redirect(url_for('index'))
	return render_template('index.html', form = form, name = session.get('name'))
```
在这个示例中，每次提交的名字都会和存储在用户会话中的名字进行比较，而会话中存储的名字是前一次在这个表单中提交的数据。如果两个名字不一样，就会调用flash() 函数，在发给客户端的下一个响应中显示一个消息。

仅调用flash() 函数并不能把消息显示出来，程序使用的模板要渲染这些消息。最好在基模板中渲染Flash 消息，因为这样所有页面都能使用这些消息。Flask 把get\_flashed\_messages() 函数开放给模板，用来获取并渲染消息，如示例所示。
```
{% block content %}
<div class="container">
	{% for message in get_flashed_messages() %}
	<div class="alert alert-warning">
		<button type="button" class="close" data-dismiss="alert">&times;</button>
		{{ message }}
	</div>
	{% endfor %}
	
	{% block page_content %}{% endblock %}
</div>
{% endblock %}
```
在模板中使用循环是因为在之前的请求循环中每次调用flash() 函数时都会生成一个消息，所以可能有多个消息在排队等待显示。get\_flashed\_messages() 函数获取的消息在下次调用时不会再次返回，因此Flash 消息只显示一次，然后就消失了。

**进阶用法：区别类别分别显示（使用category_filter字段）**
* 首先，在视图文件中使用如下：
```
flash(message, category)
# 例如
flash('Change your name success!', 'ok')
flash('Change your name faild!', 'error')
```
* 其次，在模板中使用如下：
```
{% for message in get_flashed_messages(category_filter=['ok']) %}
{% for message in get_flashed_messages(category_filter=['error']) %}
```



#### 3、使用Flask-Moment进行日期时间的管理
Flask-Moment又是一个flask的扩展模块，用来处理时间日期等信息。用这个模块主要是考虑到两点，第一是为了让不同时区的用户看到的都是各自时区的实际时间，而不是服务器所在地的时间。第二是对于一些时间间隔的处理，如果要手动处理很麻烦，如果有模块就很好了。

1、安装该模块
`pip install flask-moment`

2、初始化框架的时候引入
app文件 app/\_\_init\_\_.py：
```
from flask_moment import Moment
...

# 实例化
moment = Moment()

# 传入app进行初始化
moment.init_app(app)
```
（注：因为我使用了Flask-Scripts来管理项目，所以此处分为两步引入框架。）

3、在base.html模板中引入
```
{% block scripts %}
{{ super() }}
{{ moment.include_moment() }}
{% endblock %}
```



#### 4、Flask 前后台分离登录问题
**使用的扩展模块：**`Flask-Login`
**说明：**前台用户表：User; 后台管理员表：Manager;（注：前后台是一个app应用）
**代码实现，如下：**
```
@login_manager.user_loader
def load_user(user_id):
    if Manager.query.get(int(user_id)):
        return Manager.query.get(int(user_id))
    return User.query.get(int(user_id))
```
**注释：**`load_user(user_id)`,这个方法只负责加载用户对象，所以这里只要通过`user_id`能够返回正确的用户对象即可。
**实现效果：**前后台同时在一个浏览器中只能登录一个。因为同时都使用了`Flask-Login`，同一个session，相同的KEY，所以在同一浏览器中只能实现其中一个。相对的，可以在两个浏览器中同时分别登录前后台，使用的cookie就是不同的，可实现前后台同时使用`Flask-Login`验证的效果。
（注：为实现效果，前后台不能同时登录的问题。）



#### 5、Flask 中 `url_for()` 函数中参数问题
此问题是我在分页需要传递参数中遇到的，在此记录一下。
按照正常理解，`url_for()` 函数传递多参数，仅仅相应的在参数中增加即可。刚开始按照此思路测试，一直报错，传不到视图函数中。
然后，我查找官方文档，如下：
`flask.url_for(endpoint, **values)`
由文档可知，`url_for()` 函数的第一个参数为视图函数地址，第二个参数为关键字参数，即 `key=value` 键值对。
因此，依据我的理解，刚才的想法是正确的，然后继续梳理，最后验证成功。
附分页带参数代码如下：
**分页调用：**
```
{{ pg.page(page_data, "home.loginlog", id=current_user.id) }}
```
**分页函数相关代码：**
```
{% macro page(data, url, id=None) %}
...
<a href="{{ url_for(url, id=id, page=1) }}" aria-label="First">
    <span aria-hidden="true">首页</span>
</a>
...
（注：其它类似）
```
**小技巧：**对 id的初始值设为None，下面使用时不需要再判断是否传递 id，如果没有传递，`id=None`，下面`url_for()`函数的参数中 id就不传了。




#### 6、Flask 获取编辑器中的内容包含转义字符问题
编辑器可以使用 emotion特效图片，存储到 MySQL 数据表中的数据自动被 jinja2 模板转义了。转义的效果如下：
`<p>增加图片评论：&nbsp;<img src="http://img.baidu.com/hi/jx2/j_0062.gif"/></p>`

但是，当我取出来的时候做显示的时候，仍然是这样。那肯定是不行了。我查看了下取出来的数据类型是字符串。并没有实现反转义。原因应该是 模板文件不认为该数据为安全的，所以没有自动反转义。因此，需要让模板信任该数据，实现转义，做如下操作：
`{{ comment.content|safe }}`
即通过 `|safe` 过滤器来表示字符串是安全的，渲染的时候可进行反转义。




#### 常用技巧篇
**1、Flask 表单中上传文件的大小获取**
filesize = len(form.logo.data.read())

**2、Flask 获取用户ip**
ip = request.remote_addr

**3、Flask 前后台需要登录的问题**