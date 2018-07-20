---
title: Flask  整体概述-一个完整的程序
date: 2018-03-02 16:25:45
categories: 
 - Python
 - Flask
tags:
- Flask
- Python
---
### 一、Flask 简介
Flask(http://flask.pocoo.org/)是一种小型框架，小到可以称为“微框架”。但是，小并不意味着它比其他框架的功能少。Flask 自开发伊始就被设计为可扩展的框架，它具有一个包含基本服务的强健核心，其他功能则可通过扩展实现。你可以自己挑选所需的扩展包，组成一个没有附加功能的精益组合，从而完全精确满足自身需求。

Flask 有两个主要依赖：路由、调试和Web 服务器网关接口（Web Server Gateway Interface，WSGI）子系统由Werkzeug（http://werkzeug.pocoo.org/）提供；模板系统由Jinja2（http://jinja.pocoo.org/）提供。Werkzeug 和Jinjia2 都是由Flask 的核心开发者开发而成。

Flask 并不原生支持数据库访问、Web 表单验证和用户认证等高级功能。这些功能以及其他大多数Web 程序中需要的核心服务都以扩展的形式实现，然后再与核心包集成。开发者可以任意挑选符合项目需求的扩展，甚至可以自行开发。这和大型框架的做法相反，大型框架往往已经替你做出了大多数决定，难以（有时甚至不允许）使用替代方案。
<!--more-->

**Flask与django、tornado的区别：**
<style type="text/css">
	table th:first-of-type {
		width: 10%;
	}
	table th:nth-of-type(2) {
		width: 30%;
	}
</style>

框架   | 图标   |   特点
-------|--------|-------------------
flask  |![](/uploads/2018/03/logo-full.png)   | Flask扩展丰富，冗余度小，可自由选择组合各种插件，性能优越，相比其它web框架十分轻量级，其优雅的设计哲学易于学习掌握，小型项目快速开发，大项目毫无压力。Flask灵活开发，Python高手基本都会喜欢Flask
django |![](/uploads/2018/03/logo-django.jpg) | Django是重量级全栈型web框架，虽然功能强大，但冗余度高，自带ORM和模板引擎，灵活和自由度不够高，开发小型项目时显得过于臃肿与庞大。
tornado|![](/uploads/2018/03/tornado.png)     | Tornado是一个强大的、支持协程、高效并发且可扩展的Web服务器，发布于2009年9月，应用于FriendFeed、Facebook等社交网站。Tornado的强项在于可以利用它的异步协程机制开发高并发的服务器系统。

### 二、安装
#### 1、使用虚拟环境
安装Flask 最便捷的方式是使用虚拟环境。虚拟环境是Python解释器的一个私有副本，在这个环境中你可以安装私有包，而且不会影响系统中安装的全局Python 解释器。

虚拟环境非常有用，可以在系统的Python 解释器中避免包的混乱和版本的冲突。为每个程序单独创建虚拟环境可以保证程序只能访问虚拟环境中的包，从而保持全局解释器的干净整洁，使其只作为创建（更多）虚拟环境的源。

虚拟环境使用第三方实用工具virtualenv 创建。输入以下命令可以检查系统是否安装了virtualenv：
　　`$ virtualenv --version`
如果结果显示错误，你就需要安装这个工具。

##### （1）安装virtualenv
① 大多数Linux 发行版都提供了virtualenv 包。例如，Ubuntu 用户可以使用下述命令安装它：
　　`$ sudo apt-get install python-virtualenv`
② 如果你的电脑是Mac OS X 系统，就可以使用easy_install 安装virtualenv：
　　`$ sudo easy_install virtualenv`
③ 如果你使用微软的Windows 系统或其他没有官方virtualenv 包的操作系统，那么安装过程要稍微复杂一点。
在浏览器中输入网址https://bitbucket.org/pypa/setuptools，回车后会进入setuptools 安装程序的主页。在这个页面中找到下载安装脚本的链接，脚本名为ez_setup.py。把这个文件保存到电脑的一个临时文件夹中，然后在这个文件夹中执行以下命令：
```
$ python ez_setup.py
$ easy_install virtualenv
```

##### （2）创建虚拟环境
下一步是使用virtualenv 命令在flasky 文件夹中创建Python 虚拟环境。这个命令只有一个必需的参数，即虚拟环境的名字。创建虚拟环境后，当前文件夹中会出现一个子文件夹，名字就是上述命令中指定的参数，与虚拟环境相关的文件都保存在这个子文件夹中。
按照惯例，一般虚拟环境会被命名为venv：
```
$ virtualenv venv
New python executable in venv/bin/python2.7
Also creating executable in venv/bin/python
Installing setuptools............done.
Installing pip...............done.
```

##### （3）激活虚拟环境
现在，flasky 文件夹中就有了一个名为venv 的子文件夹，它保存一个全新的虚拟环境，其中有一个私有的Python 解释器。在使用这个虚拟环境之前，你需要先将其“激活”。如果你使用bash 命令行（Linux 和Mac OS X 用户），可以通过下面的命令激活这个虚拟环境：
　　`$ source venv/bin/activate`
如果使用微软Windows 系统，激活命令是：
　　`$ venv\Scripts\activate`
虚拟环境被激活后，其中Python 解释器的路径就被添加进PATH 中，但这种改变不是永久性的，它只会影响当前的命令行会话。为了提醒你已经激活了虚拟环境，激活虚拟环境的命令会修改命令行提示符，加入环境名：
　　`(venv) $`
当虚拟环境中的工作完成后，如果你想回到全局Python 解释器中，可以在命令行提示符下输入deactivate。

#### 2、使用pip安装Python包（Flask）
大多数Python 包都使用pip 实用工具安装，使用virtualenv 创建虚拟环境时会自动安装pip。激活虚拟环境后，pip 所在的路径会被添加进PATH。
执行下述命令可在虚拟环境中安装Flask：
　　`(venv) $ pip install flask`
执行上述命令，你就在虚拟环境中安装Flask 及其依赖了。要想验证Flask 是否正确安装，你可以启动Python 解释器，尝试导入Flask：
```
(venv) $ python
>>> import flask
>>>
```
如果没有看到错误提醒，那恭喜你——你已经可以安装完成了。


### 三、程序的基本结构
首先，来梳理一下Flask的工作流程。

#### 1、初始化
所有Flask 程序都必须创建一个程序实例。Web 服务器使用一种名为Web 服务器网关接口（Web Server Gateway Interface，WSGI）的协议，把接收自客户端的所有请求都转交给这个对象处理。程序实例是Flask 类的对象，经常使用下述代码创建：
```
from flask import Flask
app = Flask(__name__)
```
Flask 类的构造函数<font color="red">只有一个必须指定的参数，即程序主模块或包的名字</font>。在大多数程序中，Python 的__name__ 变量就是所需的值。

#### 2、路由和视图函数
客户端（例如Web 浏览器）把请求发送给Web 服务器，Web 服务器再把请求发送给Flask程序实例。程序实例需要知道对每个URL 请求运行哪些代码，所以保存了一个URL 到Python 函数的映射关系。<font color="red">处理URL和函数之间关系的程序称为路由</font>。

在Flask 程序中定义路由的最简便方式，是使用程序实例提供的app.route 修饰器，把修
饰的函数注册为路由。下面的例子说明了如何使用这个修饰器声明路由：
```
@app.route('/')
def index():
    return '<h1>Hello World!</h1>'
```
**（注：修饰器是Python 语言的标准特性，可以使用不同的方式修改函数的行为。惯常用法是使用修饰器把函数注册为事件的处理程序。）**

前例把index() 函数注册为程序根地址的处理程序，即将跟地址映射到index() 函数处理程序。在浏览器中访问http://www.example.com 后，会触发服务器执行index() 函数。这个函数的返回值称为响应，是客户端接收到的内容。如果客户端是Web 浏览器，响应就是显示给用户查看的文档。

像index() 这样的函数称为<font color="red">视图函数（view function）</font>。视图函数返回的响应可以是包含HTML 的简单字符串，也可以是复杂的表单，当然也可以是接口所需的数据格式（JSON或XML）。

此处再扩展一下，route 修饰器也支持动态URL，下例定义的路由中就有一部分是动态名字：
```
@app.route('/user/<name>')
def user(name):
    return '<h1>Hello, %s!</h1>' % name
```
尖括号中的内容就是动态部分，任何能匹配静态部分的URL 都会映射到这个路由上。调用视图函数时，Flask 会将动态部分作为参数传入函数。在这个视图函数中，参数用于生成针对个人的欢迎消息。

路由中的动态部分默认使用字符串，不过也可使用类型定义。例如，路由/user/<int:id>只会匹配动态片段id 为整数的URL。Flask 支持在路由中使用int、float 和path 类型。path 类型也是字符串，但不把斜线视作分隔符，而将其当作动态片段的一部分。

#### 3、启动服务器
程序实例用run 方法启动Flask 集成的开发Web 服务器：
```
if __name__ == '__main__':
app.run(debug=True)
```
\_\_name\_\_=='\_\_main\_\_' 是Python 的惯常用法，在这里确保直接执行这个脚本时才启动开发Web 服务器。如果这个脚本由其他脚本引入，程序假定父级脚本会启动不同的服务器，因此不会执行app.run()。

服务器启动后，会进入轮询，等待并处理请求。轮询会一直运行，直到程序停止，比如按Ctrl-C 键。

有一些选项参数可被app.run() 函数接受用于设置Web 服务器的操作模式。在开发过程中启用调试模式会带来一些便利，比如说激活调试器和重载程序。要想启用调试模式，我们可以把debug 参数设为True。

#### 4、一个完整的程序
```
# hello.py
from flask import Flask
app = Flask(__name__)

@app.route('/')
def index():
    return '<h1>Hello World!</h1>'

@app.route('user/<name>')
def user(name):
    return '<h1>Hello, %s</h1>' % self.name

if __name__ == '__main__':
    app.run()
```
然后使用下述命令启动程序：
```
(venv) $ python hello.py
* Running on http://127.0.0.1:5000/
* Restarting with reloader
```