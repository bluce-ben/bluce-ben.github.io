---
title: Flask 使用session
date: 2018-04-16 13:30:53
categories:
- Python
- Flask
tags:
- Flask
- Python
---
1、首先安装session
`pip install flask-session`

2、引入框架中
```
from flask_session import Session
sess = Session()
sess.init_app(app)
```

3、下面就可以在视图中使用引用使用了
`from flask import session`

<!--more-->

**报错集锦：**
>Flask  session报下面错误：
RuntimeError: The session is unavailable because no secret key was set.  Set the secret_key on the application to something unique and secret.

查找代码发现：
SERECT_KEY 已经设定。但是，仍然报此错误。原因是 SESSION_TYPE 未设置，如果不使用 内存缓存的话，可以使文件缓存。即：
`SESSION_TYPE = 'filesystem'`