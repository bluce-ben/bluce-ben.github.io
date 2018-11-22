---
title: Flask 基础模块 - 表单(Flask-WTF)
date: 2018-03-05 16:02:19
categories: 
 - Python
 - Flask
tags:
- Flask
- Python
---
#### 1、引入模块
Flask-WTF（http://pythonhosted.org/Flask-WTF/）扩展可以把处理Web 表单的过程变成一种愉悦的体验。这个扩展对独立的WTForms（http://wtforms.simplecodes.com）包进行了包装，方便集成到Flask 程序中。

Flask-WTF 及其依赖可使用pip 安装：
　　`(venv) $ pip install flask-wtf`
<!--more-->
#### 2、跨站请求伪造保护
默认情况下，Flask-WTF 能保护所有表单免受跨站请求伪造（Cross-Site Request Forgery，CSRF）的攻击。恶意网站把请求发送到被攻击者已登录的其他网站时就会引发CSRF 攻击。

为了实现CSRF 保护，Flask-WTF 需要程序设置一个密钥。Flask-WTF 使用这个密钥生成加密令牌，再用令牌验证请求中表单数据的真伪。设置密钥的方法如示例4-1 所示。

示例4-1　hello.py：设置Flask-WTF
```
app = Flask(__name__)
app.config['SECRET_KEY'] = 'hard to guess string'
```
**（注：SECRET_KEY 常量一般放在配置文件中，而常量的值一般放在环境变量中，配置文件从环境变量中获取值。防止他人获取配置文件拿到SECRET_KEY值）**

#### 3、表单类
使用Flask-WTF 时，每个Web 表单都由一个继承自Form 的类表示。这个类定义表单中的一组字段，每个字段都用对象表示。字段对象可附属一个或多个验证函数。验证函数用来验证用户提交的输入值是否符合要求。
示例4-2 是一个简单的Web 表单，包含一个文本字段和一个提交按钮。

示例4-2 是一个简单的Web 表单，包含一个文本字段和一个提交按钮。
```
示例4-2　hello.py：定义表单类
from flask_wtf import Form
from wtforms import StringField, SubmitField
from wtforms.validators import Required

class NameForm(Form):
    name = StringField('What is your name?', validators=[Required()])
    submit = SubmitField('Submit')
```
这个表单中的字段都定义为类变量，类变量的值是相应字段类型的对象。在这个示例中，NameForm 表单中有一个名为name 的文本字段和一个名为submit 的提交按钮。StringField类表示属性为type="text" 的`<input>` 元素。SubmitField 类表示属性为type="submit" 的`<input>` 元素。字段构造函数的第一个参数是把表单渲染成HTML 时使用的标号。

StringField 构造函数中的可选参数validators 指定一个由验证函数组成的列表，在接受用户提交的数据之前验证数据。验证函数Required() 确保提交的字段不为空。
**（注：Form 基类由Flask-WTF 扩展定义，所以从flask.ext.wtf 中导入。字段和验证函数却可以直接从WTForms 包中导入。）**

**表4-1　WTForms支持的HTML标准字段**

字段类型                | 说　　明
------------------------|--------------
StringField             | 文本字段
TextAreaField           | 多行文本字段
PasswordField           | 密码文本字段
HiddenField             | 隐藏文本字段
DateField               | 文本字段，值为datetime.date 格式
DateTimeField           | 文本字段，值为datetime.datetime 格式
IntegerField            | 文本字段，值为整数
DecimalField            | 文本字段，值为decimal.Decimal
FloatField              | 文本字段，值为浮点数
BooleanField            | 复选框，值为True 和False
RadioField              | 一组单选框
SelectField             | 下拉列表
SelectMultipleField     | 下拉列表，可选择多个值
FileField               | 文件上传字段
SubmitField             | 表单提交按钮
FormField               | 把表单作为字段嵌入另一个表单
FieldList               | 一组指定类型的字段

**表4-2  WTForms 字段支持的属性**

属性名称            | 说    明
--------------------|----------------------------------
label               | 字段的标签
validators          | 调用验证时调用的一系列验证器。
filters             | 由进程在输入数据上运行的一系列过滤器。
description         | 字段的描述，通常用于帮助文本。
id                  | 用于该字段的id。一个合理的默认设置是由表单设置的，您不需要手动设置它。
default             | 如果没有提供表单或对象输入，则将默认值分配给该字段。可能是一个可调用的。
widget              | 如果提供，则重写用于呈现字段的小部件。
render_kw (dict)    | 如果提供，则提供在呈现时向小部件提供默认关键字的字典。

**表4-2　WTForms验证函数**

验证函数                                              | 说　　明
------------------------------------------------------|---------------------
DataRequired(message=None)                            | 确保字段中有数据
Email(message=None)                                   | 验证电子邮件地址
EqualTo(fieldname, message=None)                      | 比较两个字段的值；常用于要求输入两次密码进行确认的情况
InputRequired(message=None)                           | Validates that input was provided for this field.
IPAddress(ipv4=True, ipv6=False, message=None)        | 验证IPv4 网络地址
Length(min=-1, max=-1, message=None)                  | 验证输入字符串的长度
MacAddress(message=None)                              | Validates a MAC address.
NumberRange(min=None, max=None, message=None)         | 验证输入的值在数字范围内
Optional(strip_whitespace=True)                       | 无输入值时跳过其他验证函数
Regexp(regex, flags=0, message=None)                  | 使用正则表达式验证输入值
URL(require_tld=True, message=None)                   | 验证URL
UUID(message=None)                                    | Validates a UUID.
AnyOf(values, message=None, values_formatter=None)    | 确保输入值在可选值列表中
NoneOf(values, message=None, values_formatter=None)   | 确保输入值不在可选值列表中

#### 4、把表单渲染成HTML
##### （1）手工方式渲染
表单字段是可调用的，在模板中调用后会渲染成HTML。假设视图函数把一个NameForm 实例通过参数form 传入模板，在模板中可以生成一个简单的表单，如下所示：
```
<form method="POST">
	{{ form.hidden_tag() }}
	{{ form.name.label }} {{ form.name() }}
	{{ form.submit() }}
</form>
```
当然，这个表单还很简陋。要想改进表单的外观，可以把参数传入渲染字段的函数，传入的参数会被转换成字段的HTML 属性。例如，可以为字段指定id 或class 属性，然后定义CSS 样式：
```
<form method="POST">
	{{ form.hidden_tag() }}
	{{ form.name.label }} {{ form.name(id='my-text-field') }}
	{{ form.submit() }}
</form>
```
即便能指定HTML 属性，但按照这种方式渲染表单的工作量还是很大，所以在条件允许的情况下最好能使用Bootstrap 中的表单样式。

##### （2）使用Bootstrap表单样式渲染
Flask-Bootstrap 提供了一个非常高端的辅助函数，可以使用Bootstrap 中预先定义好的表单样式渲染整个Flask-WTF 表单，而这些操作只需一次调用即可完成。使用Flask-Bootstrap，上述表单可使用下面的方式渲染：
```
{% import "bootstrap/wtf.html" as wtf %}
{{ wtf.quick_form(form) }}
```
import 指令的使用方法和普通Python 代码一样，允许导入模板中的元素并用在多个模板中。导入的bootstrap/wtf.html 文件中定义了一个使用Bootstrap 渲染Falsk-WTF 表单对象的辅助函数。wtf.quick\_form() 函数的参数为Flask-WTF 表单对象，使用Bootstrap 的默认样式渲染传入的表单。

#### 5、在视图函数中处理表单
```
示例4-4　hello.py：路由方法
@app.route('/', methods=['GET', 'POST'])
def index():
    name = None
    form = NameForm()
    if form.validate_on_submit():
        name = form.name.data
        form.name.data = ''
    return render_template('index.html', form=form, name=name)
```
app.route 修饰器中添加的methods 参数告诉Flask 在URL 映射中把这个视图函数注册为GET 和POST 请求的处理程序。如果没指定methods 参数，就只把视图函数注册为GET 请求的处理程序。

把POST 加入方法列表很有必要，因为将提交表单作为POST 请求进行处理更加便利。表单也可作为GET 请求提交，不过GET 请求没有主体，提交的数据以查询字符串的形式附加到URL 中，可在浏览器的地址栏中看到。基于这个以及其他多个原因，提交表单大都作为POST 请求进行处理。

局部变量name 用来存放表单中输入的有效名字，如果没有输入，其值为None。如上述代码所示，在视图函数中创建一个NameForm 类实例用于表示表单。提交表单后，如果数据能被所有验证函数接受，那么validate\_on\_submit() 方法的返回值为True，否则返回False。这个函数的返回值决定是重新渲染表单还是处理表单提交的数据。

用户第一次访问程序时，服务器会收到一个没有表单数据的GET 请求，所以validate\_on\_submit() 将返回False。if 语句的内容将被跳过，通过渲染模板处理请求，并传入表单对象和值为None 的name 变量作为参数。用户会看到浏览器中显示了一个表单。

用户提交表单后，服务器收到一个包含数据的POST 请求。validate\_on\_submit() 会调用name 字段上附属的Required() 验证函数。如果名字不为空，就能通过验证，validate\_on\_submit() 返回True。现在，用户输入的名字可通过字段的data 属性获取。在if 语句中，把名字赋值给局部变量name，然后再把data 属性设为空字符串，从而清空表单字段。最后一行调用render\_template() 函数渲染模板，但这一次参数name 的值为表单中输入的名字，因此会显示一个针对该用户的欢迎消息。