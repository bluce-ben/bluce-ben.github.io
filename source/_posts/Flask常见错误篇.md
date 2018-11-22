---
title: Flask 常见错误篇
date: 2018-04-16 13:31:49
categories:
- Python
- Flask
tags:
- Flask
- Python
---
**1、在 admin/forms.py 表单文件中使用如下语句报错：**
`auth_list = Auth.query.all()`

报错信息，如下：
>RuntimeError: No application found. Either work inside a view function or push an application contex.

报错信息意思是：没有找到应用程序。可以在视图函数内工作，也可以推动应用程序上下文。
<!--more-->
因此，我们就要建立程序上下文，如下：
```
app = Flask(__name__)
app.config.from_object(config[config_name])
config[config_name].init_app(app)

with app.app_context():
	db.init_app(app)
	...

return app
```
就是在初始化的时候，添加 `app_context()` 上下文。


**2、元组赋值报错，如下：**
>TypeError: ‘NoneType’ object is not iterable

这个错误提示一般发生在将None赋给多个值时。
在判断语句中，当if条件不满足，并且没有else语句时，函数默认返回None。
在没有return语句时，python也默认会返回None。
调用时，将None赋给多个值时，会出现提示：TypeError: 'NoneType' object is not iterable.


**3、模板中使用表单（wtf），报错如下：**
>TypeError: html_params() got multiple values for keyword argument 'name'

这个是由于表单中有一个变量为 “render_kw”，其值是键值对字典，且其键不能为 "name"。否则就报错！


**4、Flask 的 validate_on_submit() 老是false ??**
在 flask中提交表单时使用了validate_on_submit()来验证，但是每次提交时都是false，不知道什么原因啊？
但是只要把生成form表单的地方换成 quick_form自动生成，就正常了。
最后，通过上网搜索知道问题的原因了，就是CSRF的原因，直接在form里加上csrf_token就行了。如下：
```
{{ form.csrf_token }}
{{ form.submit() }}
```