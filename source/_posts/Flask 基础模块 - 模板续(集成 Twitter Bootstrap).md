---
title: Flask 基础模块 - 模板续(集成 Twitter Bootstrap)
date: 2018-03-05 16:02:02
categories: 
 - Python
 - Flask
tags:
- Flask
- Python
---
#### 1、集成 Bootstrap
Bootstrap（http://getbootstrap.com/）是Twitter 开发的一个开源框架，它提供的用户界面组件可用于创建整洁且具有吸引力的网页，而且这些网页还能兼容所有现代Web 浏览器。

Bootstrap 是客户端框架，因此不会直接涉及服务器。服务器需要做的只是提供引用了Bootstrap 层叠样式表（CSS） 和JavaScript 文件的HTML 响应， 并在HTML、CSS 和JavaScript 代码中实例化所需组件。这些操作最理想的执行场所就是模板。

Flask-Bootstrap 使用pip安装：
`(venv) $ pip install flask-bootstrap`
<!--more-->
Flask 扩展一般都在创建程序实例时初始化。示例3-4 是Flask-Bootstrap 的初始化方法。
示例3-4　hello.py：初始化Flask-Bootstrap
```
from flask_bootstrap import Bootstrap
# ...
bootstrap = Bootstrap(app)
```

初始化Flask-Bootstrap 之后，就可以在程序中使用一个包含所有Bootstrap 文件的基模板。这个模板利用Jinja2 的模板继承机制，让程序扩展一个具有基本页面结构的基模板，其中就有用来引入Bootstrap 的元素。

Jinja2 的模板继承机制可以帮助我们解决这一问题。Flask-Bootstrap 提供了一个具有页面基本布局的基模板，同样，程序可以定义一个具有更完整页面布局的基模板，其中包含导航条，而页面内容则可留到衍生模板中定义。示例3-7 展示了templates/base.html 的内容，这是一个继承自bootstrap/base.html 的新模板，其中定义了导航条。这个模板本身也可作为其他模板的基模板，例如templates/user.html、templates/404.html 和templates/500.html。

示例3-7　templates/base.html：包含导航条的程序基模板
```
{% extends "bootstrap/base.html" %}
{% block title %}Flasky{% endblock %}

{% block navbar %}
<div class="navbar navbar-inverse" role="navigation">
    <div class="container">
        <div class="navbar-header">
            <button type="button" class="navbar-toggle" data-toggle="collapse" 
            data-target=".navbar-collapse">
                <span class="sr-only">Toggle navigation</span>
                <span class="icon-bar"></span>
				<span class="icon-bar"></span>
				<span class="icon-bar"></span>
			</button>
			<a class="navbar-brand" href="/">Flasky</a>
		</div>
		<div class="navbar-collapse collapse">
			<ul class="nav navbar-nav">
				<li><a href="/">Home</a></li>
			</ul>
		</div>
	</div>
</div>
{% endblock %}

{% block content %}
<div class="container">
	{% block page_content %}{% endblock %}
</div>
{% endblock %}
```
这个模板的content 块中只有一个`<div>` 容器，其中包含了一个名为page\_content 的新的空块，块中的内容由衍生模板定义。

Jinja2 中的extends 指令从Flask-Bootstrap 中导入bootstrap/base.html， 从而实现模板继承。Flask-Bootstrap 中的基模板提供了一个网页框架，引入了Bootstrap 中的所有CSS 和JavaScript 文件。

基模板中定义了可在衍生模板中重定义的块。block 和endblock 指令定义的块中的内容可添加到基模板中。

上面这个base.html 模板定义了3 个块，分别名为title、navbar 和content。这些块都是基模板提供的，可在衍生模板中重新定义。title 块的作用很明显，其中的内容会出现在渲染后的HTML 文档头部，放在`<title>` 标签中。navbar 和content 这两个块分别表示页面中的导航条和主体内容。

在这个模板中，navbar 块使用Bootstrap 组件定义了一个简单的导航条。content 块中有个`<div>` 容器，其中包含一个页面头部。之前版本的模板中的欢迎信息，现在就放在这个页面头部。

Flask-Bootstrap 的base.html 模板还定义了很多其他块，都可在衍生模板中使用。表3-2 列出了所有可用的快。
**表3-2 　Flask-Bootstrap基模板中定义的块**

块　　名      | 说　　明
--------------|--------------------
doc           | 整个HTML 文档
html_attribs  | <html> 标签的属性
html          | <html> 标签中的内容
head          | <head> 标签中的内容
title         | <title> 标签中的内容
metas         | 一组<meta> 标签
styles        | 层叠样式表定义
body_attribs  | <body> 标签的属性
body          | <body> 标签中的内容
navbar        | 用户定义的导航条
content       | 用户定义的页面内容
scripts       | 文档底部的JavaScript 声明

表3-2 中的很多块都是Flask-Bootstrap 自用的，如果直接重定义可能会导致一些问题。例如，Bootstrap 所需的文件在styles 和scripts 块中声明。如果程序需要向已经有内容的块中添加新内容，必须使用Jinja2 提供的super() 函数。例如，如果要在衍生模板中添加新的JavaScript 文件，需要这么定义scripts 块：
```
{% block scripts %}
    {{ super() }}
    <script type="text/javascript" src="my-script.js"></script>
{% endblock %}
```

#### 2、自定义错误页面
像常规路由一样，Flask 允许程序使用基于模板的自定义错误页面。最常见的错误代码有两个：404，客户端请求未知页面或路由时显示；500，有未处理的异常时显示。为这两个错误代码指定自定义处理程序的方式如示例3-6 所示。

示例3-6　hello.py：自定义错误页面
```
@app.errorhandler(404)
def page_not_found(e):
    return render_template('404.html'), 404
@app.errorhandler(500)
def internal_server_error(e):
    return render_template('500.html'), 500
```
和视图函数一样，错误处理程序也会返回响应。它们还返回与该错误对应的数字状态码。

错误处理程序中引用的模板也需要编写。这些模板应该和常规页面使用相同的布局，因此要有一个导航条和显示错误消息的页面头部。

现在，程序使用的模板继承自 templates/base.html 模板，而不直接继承自Flask-Bootstrap 的基模板。通过继承templates/base.html 模板编写自定义的404 错误页面很简单，如示例3-8 所示。

示例3-8　templates/404.html：使用模板继承机制自定义404 错误页面
```
{% extends "base.html" %}
{% block title %}Flasky - Page Not Found{% endblock %}
{% block page_content %}
    <div class="page-header">
        <h1>Not Found</h1>
	</div>
{% endblock %}
```

templates/user.html 现在可以通过继承这个基模板来简化内容，如示例3-9 所示。
示例3-9　templates/user.html：使用模板继承机制简化页面模板
```
{% extends "base.html" %}
{% block title %}Flasky{% endblock %}
{% block page_content %}
	<div class="page-header">
		<h1>Hello, {{ name }}!</h1>
	</div>
{% endblock %}
```

#### 3、链接
在模板中直接编写简单路由的URL 链接不难，但对于包含可变部分的动态路由，在模板中构建正确的URL 就很困难。而且，直接编写URL 会对代码中定义的路由产生不必要的依赖关系。如果重新定义路由，模板中的链接可能会失效。

为了避免这些问题，Flask 提供了url_for() 辅助函数，它可以使用程序URL 映射中保存的信息生成URL。

url_for() 函数最简单的用法是以视图函数名（或者app.add_url_route() 定义路由时使用的端点名）作为参数，返回对应的URL。例如，在当前版本的hello.py 程序中调用url_for('index') 得到的结果是/。调用url_for('index', _external=True) 返回的则是绝对地址，在这个示例中是http://localhost:5000/。

使用url_for() 生成动态地址时， 将动态部分作为关键字参数传入。例如，url_for('user', name='john', _external=True) 的返回结果是http://localhost:5000/user/john。传入url_for() 的关键字参数不仅限于动态路由中的参数。函数能将任何额外参数添加到查询字符串中。例如，url_for('index', page=2) 的返回结果是/?page=2。


#### 4、静态文件
Web 程序不是仅由Python 代码和模板组成。大多数程序还会使用静态文件，例如HTML代码中引用的图片、JavaScript 源码文件和CSS。

默认设置下，Flask 在程序根目录中名为static 的子目录中寻找静态文件。如果需要，可在static 文件夹中使用子文件夹存放文件。服务器收到前面那个URL 后，会生成一个响应，包含文件系统中static/css/styles.css 文件的内容。

示例3-10 展示了如何在程序的基模板中放置favicon.ico 图标。这个图标会显示在浏览器的地址栏中。
示例3-10　templates/base.html：定义收藏夹图标
```
{% block head %}
{{ super() }}
<link rel="shortcut icon" href="{{ url_for('static', filename = 'favicon.ico') }}"
 type="image/x-icon">
<link rel="icon" href="{{ url_for('static', filename = 'favicon.ico') }}"
 type="image/x-icon">
{% endblock %}
```
图标的声明会插入head 块的末尾。注意如何使用super() 保留基模板中定义的块的原始内容。


#### 5、使用Flask-Moment 本地化日期和时间
服务器需要统一时间单位，这和用户所在的地理位置无关，所以一般使用协调世界时（Coordinated Universal Time，UTC）。不过用户看到UTC 格式的时间会感到困惑，他们更希望看到当地时间，而且采用当地惯用的格式。

要想在服务器上只使用UTC 时间，一个优雅的解决方案是，把时间单位发送给Web 浏览器，转换成当地时间，然后渲染。Web 浏览器可以更好地完成这一任务，因为它能获取用户电脑中的时区和区域设置。

有一个使用JavaScript 开发的优秀客户端开源代码库，名为moment.js（http://momentjs.com/），它可以在浏览器中渲染日期和时间。Flask-Moment 是一个Flask 程序扩展，能把moment.js 集成到Jinja2 模板中。Flask-Moment 可以使用pip 安装：
`(venv) $ pip install flask-moment`

这个扩展的初始化方法如示例3-11 所示。
示例3-11　hello.py：初始化Flask-Moment
```
from flask.ext.moment import Moment
moment = Moment(app)
```

除了moment.js，Flask-Moment 还依赖jquery.js。要在HTML 文档的某个地方引入这两个库，可以直接引入，这样可以选择使用哪个版本，也可使用扩展提供的辅助函数，从内容分发网络（Content Delivery Network，CDN）中引入通过测试的版本。Bootstrap 已经引入了jquery.js，因此只需引入moment.js 即可。示例3-12 展示了如何在基模板的scripts 块中引入这个库。
示例3-12　templates/base.html：引入moment.js 库
```
{% block scripts %}
{{ super() }}
{{ moment.include_moment() }}
{% endblock %}
```

为了处理时间戳，Flask-Moment 向模板开放了moment 类。示例3-13 中的代码把变量current_time 传入模板进行渲染。

示例3-13　hello.py：加入一个datetime 变量
```
from datetime import datetime
@app.route('/')
def index():
    return render_template('index.html', current_time=datetime.utcnow())
```

示例3-14 展示了如何在模板中渲染current_time。
代码3-14　templates/index.html：使用Flask-Moment 渲染时间戳
```
<p>The local date and time is {{ moment(current_time).format('LLL') }}.</p>
<p>That was {{ moment(current_time).fromNow(refresh=True) }}</p>
```

format('LLL') 根据客户端电脑中的时区和区域设置渲染日期和时间。参数决定了渲染的方式，'L' 到'LLLL' 分别对应不同的复杂度。format() 函数还可接受自定义的格式说明符。

第二行中的fromNow() 渲染相对时间戳，而且会随着时间的推移自动刷新显示的时间。这个时间戳最开始显示为“a few seconds ago”，但指定refresh 参数后，其内容会随着时间的推移而更新。如果一直待在这个页面，几分钟后，会看到显示的文本变成“a minuteago”“2 minutes ago”等。

Flask-Moment 实现了moment.js 中的format()、fromNow()、fromTime()、calendar()、valueOf()和unix() 方法。你可查阅文档（http://momentjs.com/docs/#/displaying/）学习moment.js 提供的全部格式化选项。

Flask-Moment 渲染的时间戳可实现多种语言的本地化。语言可在模板中选择，把语言代码传给lang() 函数即可：
`\{\{ moment.lang('es') \}\}`