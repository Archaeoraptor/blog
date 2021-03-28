---
title: Flask接锅记
tags:
 - Flask
abbrlink: ff2b
date: 2019-10-12 15:58:43
categories:
  - 笔记&札记
---
动态语言一时爽，维护重构火葬场
<!--more-->

<!-- ## 前言

国庆假期之后开始了偶滴搬砖生活，教研室有个师兄在拿Flask做微信公众号的文章推送后端。为了读祖传代码，准备折腾Flask。
据说Flask和Django差不了太多，之前Django用的很菜，也不打算拿Django或node之类的重构一边，索性就跳Flask的坑吧

## 配置

[1] 开启Debug模式。1.0之后直接在Pycharm的Configuration里勾选开启就好了。或者

    export Flask_Debug = 1 # win系统使用set命令

[2] 使用装饰器，在指定url或修改请求方式时常用，如

```python
@app.route('/hello')
def hello():
    return 'Hello, World'
```

指定请求方式

```python
@app.route('/login', methods=['GET', 'POST'])
```

[3] 处理请求、文件传输和cookies等
都在flask的request库里

```python
    from flask import request
```

[4] 数据库用SQLAlchemy可以像Django一样用python操作数据库，建表查询调用。

## 蓝图

我们项目再大一些，就要拆分模块，Flask一般使用蓝图(Blueprint)
这样注册

```python
from flask import Flask
from yourapplication.simple_page import simple_page

app = Flask(__name__)
app.register_blueprint(simple_page, url_prefix='/pages')
```

这样引用

```python
from flask import Flask
from yourapplication.simple_page import simple_page

app = Flask(__name__)
app.register_blueprint(simple_page)
```

也可以作为资源放置静态文件，详见官方文档

## 视图和模板

Flask的模板(template)用的Jinja2，基本都是标签的形式，和Django的模板非常像，类似这种

```html
{% block title %}{% endblock %}
```

(我刚刚又查了一下，Django的自带模板就是基于Jinja的魔改版，应该是Django模板像Jinja2，说反了)

视图函数一般用来处理入站请求，我们用装饰器把函数注册为事件处理程序，大概这个样子

```python
@app.route('/user/<name>')
def usr(name):
    return 'Hello World,{}!'.format(name)
``` -->

## 跨域

跨域请求：安装flask-cors, 设置一下就行了

```Python
from flask_cors import CORS
···
CORS(app, supports_credentials=True)
```

在chrome的新特性中，使用Same-Site Cookies来解决CSRF跨域攻击的问题，没设置会报这种警告, 在未来的版本中，没设置是不允许跨域的

```
A cookie associated with a cross-site resource at http://··· was set without the `SameSite` attribute. A future release of Chrome will only deliver cookies with cross-site requests if they are set with `SameSite=None` and `Secure`.
```

设置一下属性就好了

生成和导入requirements

```python
pip freeze > requirements.txt
pip install -r requirements.txt
```

使用哈希值存储用户密码

```python
from werkzeug.security import generate_password_hash, check_password_hash
```

## Flask部署

这里我用的Docker。照着[这里](https://zhuanlan.zhihu.com/p/78432719)部署的
这里没有用Nginx，直接上的gunicorn，回头看一下Nginx怎么搞
这里有个[官方文档](http://docs.gunicorn.org/en/stable/deploy.html)