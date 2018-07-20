---
title: Flask 基础模块 - 模板(Jinjia2)
date: 2018-03-05 16:01:35
categories: 
 - Python
 - Flask
tags:
- Flask
- Python
---
#### 1、渲染模板
默认情况下，Flask 在程序文件夹中的templates 子文件夹中寻找模板。在下一个hello.py版本中，要把前面定义的模板保存在templates 文件夹中，并分别命名为index.html 和user.html。

示例3-3　hello.py：渲染模板
```
from flask import Flask, render_template
app = Flask(__name__)

@app.route('/')
def index():
    return render_template('index.html')
@app.route('/user/<name>')
def user(name):
    return render_template('user.html', name=name)

if __name__ == '__main__':
    app.run()
```
<!--more-->
Flask 提供的render_template 函数把Jinja2 模板引擎集成到了程序中。<font color="red">render_template 函数的第一个参数是模板的文件名。随后的参数都是键值对，表示模板中变量对应的真实值。</font>在这段代码中，第二个模板收到一个名为name 的变量。

#### 2、变量
在模板中使用的{{ name }} 结构表示一个变量，它是一种特殊的占位符，告诉模板引擎这个位置的值从渲染模板时使用的数据中获取。

Jinja2 能识别所有类型的变量，甚至是一些复杂的类型，例如列表、字典和对象。在模板中使用变量的一些示例如下：
```
<p>A value from a dictionary: {{ mydict['key'] }}.</p>
<p>A value from a list: {{ mylist[3] }}.</p>
<p>A value from a list, with a variable index: {{ mylist[myintvar] }}.</p>
<p>A value from an object's method: {{ myobj.somemethod() }}.</p>
```
可以使用过滤器修改变量，过滤器名添加在变量名之后，中间使用竖线分隔。例如，下述
模板以首字母大写形式显示变量name 的值：
　　`Hello, \{\{ name|capitalize \}\}`

**表3-1 列出了Jinja2 提供的部分常用过滤器。**

过滤器名  | 说　　明
----------|-------------------
safe      | 渲染值时不转义
capitalize| 把值的首字母转换成大写，其他字母转换成小写
lower     | 把值转换成小写形式
upper     | 把值转换成大写形式
title     | 把值中每个单词的首字母都转换成大写
trim      | 把值的首尾空格去掉
striptags | 渲染之前把值中所有的HTML 标签都删掉

safe 过滤器值得特别说明一下。默认情况下，出于安全考虑，Jinja2 会转义所有变量。例如，如果一个变量的值为`<h1>Hello</h1>`，Jinja2 会将其渲染成`&lt;h1&gt;Hello&lt;/h1&gt;`，浏览器能显示这个h1 元素，但不会进行解释。很多情况下需要显示变量中存储的HTML 代码，这时就可使用safe 过滤器。

完整的过滤器列表可在Jinja2 文档（http://jinja.pocoo.org/docs/templates/#builtin-filters）中查看。

#### 3、控制结构
Jinja2 提供了多种控制结构，可用来改变模板的渲染流程。
下面这个例子展示了如何在模板中使用条件控制语句：
```
{% if user %}
    Hello, {{ user }}!
{% else %}
    Hello, Stranger!
{% endif %}
```
另一种常见需求是在模板中渲染一组元素。下例展示了如何使用for 循环实现这一需求：
```
<ul>
    {% for comment in comments %}
        <li>{{ comment }}</li>
    {% endfor %}
</ul>
```
Jinja2 还支持宏。宏类似于Python 代码中的函数。例如：
```
{% macro render_comment(comment) %}
    <li>{{ comment }}</li>
{% endmacro %}
<ul>
    {% for comment in comments %}
        {{ render_comment(comment) }}
    {% endfor %}
</ul>
```
为了重复使用宏，我们可以将其保存在单独的文件中，然后在需要使用的模板中导入：
```
{% import 'macros.html' as macros %}
<ul>
    {% for comment in comments %}
        {{ macros.render_comment(comment) }}
    {% endfor %}
</ul>
```
需要在多处重复使用的模板代码片段可以写入单独的文件，再包含在所有模板中，以避免重复：
`include "comment.html"`

另一种重复使用代码的强大方式是模板继承，它类似于Python 代码中的类继承。首先，创建一个名为base.html 的基模板：
```
<html>
<head>
    {% block head %}
    <title>{% block title %}{% endblock %} - My Application</title>
    {% endblock %}
</head>
<body>
    {% block body %}
    {% endblock %}
</body>
</html>
```
block 标签定义的元素可在衍生模板中修改。在本例中，我们定义了名为head、title 和body 的块。注意，title 包含在head 中。下面这个示例是基模板的衍生模板：
```
{% extends "base.html" %}
{% block title %}Index{% endblock %}

{% block head %}
    {{ super() }}
    <style></style>
{% endblock %}

{% block body %}
    <h1>Hello, World!</h1>
{% endblock %}
```
extends 指令声明这个模板衍生自base.html。在extends 指令之后，基模板中的3 个块被重新定义，模板引擎会将其插入适当的位置。注意新定义的head 块，在基模板中其内容不是空的，所以使用super() 获取原来的内容。