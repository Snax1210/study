# Flask入门系列(四)–请求，响应及会话


- [Flask入门系列(四)–请求，响应及会话](#flask入门系列四请求响应及会话)
  - [Flask内建对象](#flask内建对象)
    - [请求对象request](#请求对象request)
    - [会话对象session](#会话对象session)
    - [构建相应](#构建相应)
    - [Cookie的使用](#cookie的使用)
    - [全局对象g](#全局对象g)

一个完整的HTTP请求，包括了客户端的请求Request，服务端的响应Response，会话Session等。一个基本的Web框架一定会提供内建的对象来访问这些信息，Flask当然也不例外。我们来看看在Flask中该怎么使用这些内建对象。

## Flask内建对象

Flask提供的内建对象常用的有request，session，g，通过request，你还可以获取cookie对象。这些对象不但可以在请求函数中使用，在模板中也可以使用。

### 请求对象request

引入flask包中的request对象，就可以直接在请求函数中直接使用该对象了。让我们改进下第二篇的`login()`方法：

```python
from flask import request

@app.route('/login', methods=['POST', 'GET'])
def login():
    if request.method == 'POST':
        if request.form['user'] == 'admin':
            return 'Admin login successfully!'
        else:
            return 'No such user!'
    title = request.args.get('title', 'Default')
    return render_template('login.html', title=title)
```

在第三篇的template目录下，添加“login.html”文件

```html
{% extends "layout.html" %}
{% block body %}
<form name="login" action="/login" method="post">
    Hello {{ title }}, please login by:
    <input type="text" name="user" />
</form>
{% endblock %}
```

执行上面的例子，结果我就不多描述了。简单解释下，`request`中`method`变量可以获取当前请求的方法，即”GET”, “POST”, “DELETE”, “PUT”等；`form`变量是一个字典，可以获取”Post”请求表单中的内容，在上例中，如果提交的表单中不存在`user`项，则会返回一个`KeyError`，你可以不捕获，页面会返回400错误（想避免抛出这`KeyError`，你可以用`request.form.get('user')`来替代）。而`request.args.get()`方法则可以获取”Get”请求URL中的参数，该函数的第二个参数是默认值，当URL参数不存在时，则返回默认值。`request`的详细使用可参阅Flask[官方API文档](http://https://flask.palletsprojects.com/en/2.0.x/)

### 会话对象session

会话可以用来保存当前请求的一些状态，以便于在请求之前共享信息。我们将上面的python代码改动下：

```python
from flask import request, session

@app.route('/login', methods=['POST', 'GET'])
def login():
    if request.method == 'POST':
        if request.form['user'] == 'admin':
            session['user'] = request.form['user']
            return 'Admin login successfully!'
        else:
            return 'No such user!'
    if 'user' in session:
        return 'Hello %s!' % session['user']
    else:
        title = request.args.get('title', 'Default')
        return render_template('login.html', title=title)

app.secret_key = '123456'
```

你可以看到，”admin”登陆成功后，再打开”login”页面就不会出现表单了。session对象的操作就跟一个字典一样。特别提醒，使用session时一定要设置一个密钥`app.secret_key`，如上例。不然你会得到一个运行时错误，内容大致是`RuntimeError: the session is unavailable because no secret key was set`。密钥要尽量复杂，最好使用一个随机数，这样不会有重复，上面的例子不是一个好密钥。

我们顺便写个登出方法，估计我不放例子，大家也都猜到怎么写，就是清除字典里的键值：

```python
from flask import request, session, redirect, url_for

@app.route('/logout')
def logout():
    session.pop('user', None)
    return redirect(url_for('login'))
```

关于`redirect`方法，我们会在下一篇介绍。

### 构建相应

在之前的例子中，请求的相应我们都是直接返回字符串内容，或者通过模板来构建响应内容然后返回。其实我们也可以先构建响应对象，设置一些参数（比如响应头）后，再将其返回。修改下上例中的“Get”请求部分：

```python
from flask import request, session, make_response

@app.route('/login', methods=['POST', 'GET'])
def login():
    if request.method == 'POST':
        ...
    if 'user' in session:
        ...
    else:
        title = request.args.get('title', 'Default')
        response = make_response(render_template('login.html', title=title), 200)
        response.headers['key'] = 'value'
        return response
```

打开浏览器调试，在”Get”请求用户未登录状态下，你会看到响应头中有一个`key`项。`make_response`方法就是用来构建`response`对象的，第二个参数代表响应状态码，缺省就是”200”。`response`对象的详细使用可参阅Flask的[官方API文档](http://https://flask.palletsprojects.com/en/2.0.x/)

### Cookie的使用

提到了Session，当然也要介绍Cookie，毕竟没有Cookie，Session就根本没法用（不知道为什么？查查去）。Flask中使用Cookie也很简单：

```python
from flask import request, session, make_response
import time

@app.route('/login', methods=['POST', 'GET'])
def login():
    response = None
    if request.method == 'POST':
        if request.form['user'] == 'admin':
            session['user'] = request.form['user']
            response = make_response('Admin login successfully!')
            response.set_cookie('login_time', time.strftime('%Y-%m-%d %H:%M:%S'))
        ...
    else:
        if 'user' in session:
            login_time = request.cookies.get('login_time')
            response = make_response('Hello %s, you logged in on %s' % (session['user'], login_time))
        ...

    return response
```

例子越来越长了，这次我们引入了`time`模块来获取当前系统时间。我们在返回响应时，通过`response.set_cookie()`函数，来设置Cookie项，之后这个项值会被保存在浏览器中。这个函数的第三个参数`max_age`可以设置该Cookie项的有效期，单位是秒，不设的话，在浏览器关闭后，该Cookie项即失效。

在请求中，`request.cookies`对象就是一个保存了浏览器Cookie的字典，使用其`get()`函数就可以获取相应的键值。

### 全局对象g

`flask.g`是Flask一个全局对象，这里有点容易让人误解，其实`g`的作用范围，就在一个请求（也就是一个线程）里，它不能在多个请求间共享。你可以在`g`对象里保存任何你想保存的内容。一个最常用的例子，就是在进入请求前，保存数据库连接。这个我们会在介绍数据库集成时讲到。