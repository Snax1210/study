[TOC]: # "Flask入门系列(一)–Hello World"

# Flask入门系列(一)–Hello World
- [HelloWorld](#helloworld)
  - [简单解释下这段代码](#简单解释下这段代码)


项目开发中，经常要写一些小系统来辅助，比如监控系统，配置系统等等。用系统的Java写，太笨重了，连PHP都嫌麻烦。
一直在寻找一个轻量级的后台框架，学习成本低，维护简单。发现Flask后，我立马被它的轻巧所吸引，它充分发挥了Python语言的
优雅和轻便，连Django这样强大的框架在它面前都觉得繁琐。可以说简单就是美。这里我们不讨论到底哪个框架语言更好，只是从简单
这个角度出发，Flask绝对是佼佼者。这一系列文章就会给大家展示Flask的轻巧之美。

## HelloWorld

程序员的经典学习方法，从Hello World开始。不要忘了，先安装python,pip，然后运行`pip install Flask`，环境就装好了。当然
本人还是强烈建议使用virtualenv来安装环境。细节就不多说了，让我们写一个Hello World吧：

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def index():
    return '<h1>Hello World</h1>'

if __name__ == '__main__':
    app.run()
```

一个Web应用的代码就写完了，对，就是这么简单！保存为“hello.py”，打开控制台，到该文件目录下，运行

```cmd
$ python hello.py
```

如果看到
```cmd
* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

字样后，就说明服务器启动完成。打开你的浏览器，访问`http://127.0.0.1:5000/`，一个硕大的“Hello World”映入眼帘。

### 简单解释下这段代码

- 首先是引入了Flask包，并创建一个Web应用的实例“app”

```python
from flask import Flask
app = Flask(__name__)
```
这里给的实例名称就是这个python模块名

- 定义路由规则

```python
@app.route('/')
```
这个函数级别的注解指明了当地址是根路径时，就调用下面的函数。可以定义多个路由规则，会在下篇文章里详细介绍。说的高大
上一些，这里就是MVC的Controller

- 处理请求

```python
def index():
    return  '<h1>Hello World</h1>'
```

当请求的地址符合路由规则时，就会进入该函数。可以说，这是MVC的Model层。你可以在里面获取请求的request对象，返回的内
容就是response。本例中的response就是大标题“HelloWorld”

- 启动Web服务器

```python
if __name__ == '__main__':
    app.run()
```

把文件为程序入口（也就是用python命令直接执行本文件）时，就会通过`app.run()`启动Web服务器。如果不是程序入口，那么
改文件就是一个模块。Web服务器会默认监听本地的5000端口，但不支持远程访问。如果你想支持远程，需要在`run ()`方法传
入`host=0.0.0.0`，想改变监听端口的话，传入`port=端口号`，你还可以这是调试模式。具体例子如下：

```python
if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8888, debug=True)
```

注意，Flask自带的Web服务器主要时候给开发人员调试用的，在生产环境下，你最好是通过WSGI将Flask工程部署到类似Apache
或Nginx的服务器上。

