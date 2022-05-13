# Flask Note 
2022/5/2 
Start learning Flask :)

# 安装
pipenv: 基于pip的Python包管理工具  
安装pipenv：
```
pip install pipenv  
```
新建虚拟环境：  
```
pipenv install
```
在当前工作目录创建虚拟环境, 还会在当前目录下创建Pipfile, Pipfile.lock文件，前者记录依赖包列表，后者记录固定版本的详细依赖包列表。当我们使用pipenv更新/创建/删除依赖包时，这两个文件会自动更新。这种方式代替了传统的pip+requirements.txt方式，requirements.txt需要手动维护。

激活虚拟环境:
```
pipenv shell
```

![p1](p1.png)

安装flask:  
```
pipenv install flask  
```

更新flask:
```
pipenv update flask  
```

# 最小的flask app
```python
from flask import Flask

app = Flask(__name__)
# Flask第一个参数是模块或包名，使用__name__的话，python会根据所处的模块命名，如当前程序在app.py，那么这个值为app


# the minimal Flask application
@app.route('/')
def index():
    return '<h1>Hello, World!</h1>'
```

## 客户端与服务器上的flask程序交互步骤
1. 用户输入URL访问某个资源
2. Flask收到请求并分析URL
3. 为这个URL找到处理函数
4. 执行函数并生成响应，返回给浏览器
5. 浏览器接收并解析响应，在页面上显示信息

@app.route() 装饰器根据传入的URL规则，与处理函数建立联系。这个过程称为“注册路由”(route)，这个函数称为“视图函数”(view function)。  
URL规则是字符串，以斜杠开始，是相对URL，不包含域名。

## 一个视图可以绑定多个URL
```python
# bind multiple URL for one view function
@app.route('/hi')
@app.route('/hello')
def say_hello():
    return '<h1>Hello, Flask!</h1>'
```
## 动态URL
URL规则中可以添加变量，Flask会把变量传入视图函数。
```python
# dynamic route, URL variable default
@app.route('/greet', defaults={'name': 'Programmer'})
@app.route('/greet/<name>')
def greet(name):
    return '<h1>Hello, %s!</h1>' % name
```
当URL规则包含变量，但用户没有添加变量时，那么Flask会在匹配失败时返回404错误响应。为避免这种情况，可以为变量添加一个默认值，为视图添加一个新的装饰器处理没有传参的情况。
```python
@app.route('/greet')
@app.route('/greet/<name>')
def greet(name='Programmer'):
return '<h1>Hello, %s!</h1>' % name
```
# 启动Flask
![p2](p2.png)
flask run命令默认监听 127.0.0.1:5000 地址。  
![p3](p3.png)  
使用flask run命令前，需要提供程序所在位置，flask会自动从两个位置寻找：  
 - 当前目录下寻找app.py和wsgy.py，并从中寻找app或application对象
 - 从环境变量FLASK_APP对应的值寻找名为app或application的程序实例  

当前使用的app对象就在app.py，所以flask通过第一种方法能探测到；如果我们使用的是别的名字，如hello.py，那就需要在环境变量设置FLASK_APP。  
Linux:
```bash
$ export FLASK_APP=hello
```
Windows:
```
> set FLASK_APP=hello
```

为了避免每次重启都需要重新设置FLASK_APP环境变量，或者在开发多个Flask程序需要切换环境变量，可以使用pythondotenv
管理项目的环境变量。  
```
pipenv install python-dotenv
```
在项目根目录创建两个文件，.env和.flaskenv。  
.env: 存储敏感信息环境变量，如email账户密码。  
.flaskenv：存储公共环境变量，如FLASK_APP。

Flask加载环境变量的优先级：  
手动设置的环境变量>.env中设置的环境变量>.flaskenv设置的环境变量

## 设置运行环境
为了区分production和development环境，Flask提供了一个FLASK_ENV环境变量，默认为production。  
在开发时，可以在.flaskenv文件中将它设置为development。
```
FLASK_ENV=development
```
开发模式下，调试模式（Debug Mode）将被开启，这时执行flask run启动程序会自动激活Werkzeug内置的调试器（debugger）和重载器 （reloader）。  
调试器非常强大，当程序出错时，我们可以在网页上看到详细的错误追踪信息。  
重载器的作用就是监测文件变动，然后重新启动开发服务器。  
默认会使用Werkzeug内置的stat重载器，它的缺点是耗电较严重，而且准确性一般。
为了获得更优秀的体验，我们可以安装另一个用于监测文件变动的Python库Watchdog：
```bash
$ pipenv install watchdog --dev
```
-dev声明了这个包只在开发时被用到。

## 使用flask shell启动Python Shell
```
D:\workspace\helloflask\demos\hello> flask shell
Python 3.8.2 (tags/v3.8.2:7b3ab59, Feb 25 2020, 22:45:29) [MSC v.1916 32 bit (Intel)] on win32
App: app [development]
Instance: D:\workspace\helloflask\demos\hello\instance
>>>
```

## 自定义flask命令
除了flask run, flask shell等命令外，还可以在app.py中通过函数创建flask命令。  
```python
# custom flask cli command
@app.cli.command()
def hello():
    """Just say hello."""
    click.echo('Hello, Human!')
```
函数的名称即为命令名称，通过flask hello来触发函数。
```
D:\workspace\helloflask\demos\hello> flask hello
Hello, Human!
```

# Flask & HTTP
## Request 对象
请求URL和报文中的其他信息都可以通过Flask的Request对象提供的属性和方法获取。
URL: http://helloflask.com/hello?name=Grey
![p4](p4.png)  
```python
from flask import Flask, request
app = Flask(__name__)
@app.route('/hello')
def hello():
    name = request.args.get('name', 'Flask')  # 通过request对象得到传入的name，若未传入，默认值为Flask
    # 使用request.args.get而不是args['name']的原因是
    # 如果args中没有name这个key，那么会返回HTTP 400错误响应(Bad Request，表示请求无效)，而不是抛出KeyError异常
    # 为了避免这个错误，应该使用get方法
    return '<h1>Hello, %s!<h1>' % name
```
## Flask 处理请求
当前运行的app实例会存储一张路由表(app.url_map)，其中定义了URL规则和视图函数的映射关系。
当请求发来后，Flask会根据请求报文中的URL(path部分)来尝试与这个表中的所有URL规则进行匹配，调用匹配成功的视图函数。
如果没有找到匹配的URL规则，说明程序中没有处理这个URL的视图函数，Flask会自动返回404错误响应(Not Found，表示资源未找到)。  

使用flask routes命令可以查看程序中定义的所有路由，这个列表由app.url_map解析得到：  
```
$ flask routes
Endpoint  Methods  Rule
--------  -------  -----------------------
hello     GET
go_back   GET
hi        GET
...
static GET
/hello
/goback/<int:age>
/hi
/static/<path:filename>
```

### 设置监听的HTTP方法
在app.route()装饰器的methods参数可以设置监听方法：  
```python
@app.route('/hello', methods=['GET', 'POST'])
def hello():
   return '<h1>Hello, Flask!</h1>'
```
如果使用了不被允许的监听方法，访问该URL时视图函数会返回405错误响应（Method Not Allowed，表示请求方法不允许）。  

### URL处理
app.route()装饰器中可以对传入的变量进行类型转化，当变量无法转化时，会返回404错误响应，间接验证了传入变量类型是否正确。  
![p5](p5.png) 
```python
@app.route('goback/<int:year>')
def go_back(year):
   return '<p>Welcome to %d!</p>' % (2018 - year)
```
any 提供了一个可选的value列表：  
```python
@app.route('/colors/<any(blue, white, red):color>')
def three_colors(color):
    ...
```
可以使用格式化字符串的方式来设置这个列表，用%s或format都可以：  
```python
colors = ['blue', 'white', 'red']
@app.route('/colors/<any(%s):color>' % str(colors)[1:-1])
...
```
### 请求钩子
请求钩子用于预处理、后处理。  
![p6](p6.png) 

```python
@app.before_request
def do_something():
    pass # 这里的代码会在每个请求处理前执行
```
常见的应用是建立数据库连接，通常会有多个视图函数需要建立和关闭数据库连接，这些操作基本相同。
一个理想的解决方法是在 请求之前(before_request)建立连接，在请求之后(teardown_request) 关闭连接。  

## HTTP 响应
响应报文主要由协议版本、状态码(status code)、原因短语 (reason phrase)、响应首部和响应主体组成。
以发向localhost: 5000/hello的请求为例：  
![p7](p7.png) 

响应报文的首部包含一些关于响应和服务器的信息，这些内容由Flask生成，而我们在视图函数中返回的内容即为响应报文中的主体内容。
浏览器接收到响应后，会把返回的响应主体解析并显示在浏览器窗口上。
常见的HTTP状态码：  
![p8](p8.png) 

### 在Flask中生成响应
Flask会先判断是否可以找到与请求URL相匹配的路由，如果没有则返回404响应。
如果找到，则调用对应的视图函数，视图函数的返回值构成了响应报文的主体内容，正确返回时状态码默认为200。
Flask会调用make_response()方法将视图函数返回值转换为响应对象。  
视图函数可以返回最多由三个元素组成的元组:响应主体、状态码、首部字段。其中首部字段可以为字典，或是两元素元组组成的列表。  

只包含主体的响应：  
```python
@app.route('/hello')
def hello():
    ...
    return '<h1>Hello, Flask!</h1>'
```
默认状态码为200，可以指定：  
```python
@app.route('/hello')
def hello():
    ...
    return '<h1>Hello, Flask!</h1>', 201
```
有时你会想附加或修改某个首部字段。比如，要生成状态码为3XX的重定向响应，需要将首部中的Location字段设置为重定向的目标URL:
```python
@app.route('/hello')
def hello():
    ...
    return '', 302, {'Location', 'http://www.example.com'}
```
也可以通过使用Flask提供的redirect()函数来生成重定向响应：
```python
from flask import Flask, redirect
# ...
@app.route('/hello')
def hello():
    return redirect('http://www.example.com')
```
如果要在程序内重定向到其他视图，那么只需在redirect()函数中使用url_for()函数生成目标URL即可:
```python
from flask import Flask, redirect, url_for
...
@app.route('/hi')
def hi():
    ...
    return redierct(url_for('hello'))

@app.route('/hello')
def hello():
```
在abort()函数中传入状态码即可返回对应的错误响应:  
```python
from flask import Flask, abort
...
@app.route('/404')
def not_found():
    abort(404)
```

### 响应格式
MIME类型(又称为media type或content type)是一种用来标识文件类型的机制，比如，HTML的MIME类型为“text/html”，png图片的MIME类型为“image/png”。
默认的响应格式是HTML，如果你想使用其他MIME类型，可以通过Flask提供的make_response()方法生成响应对象，传入响应的主体作为参数，
然后使用响应对象的mimetype属性设置MIME类型，比如:
```python
from flask import make_response
@app.route('/foo')
def foo():
   response = make_response('Hello, World!')
   response.mimetype = 'text/plain'
   return response
```

### Cookie
Cookie指Web服务器为了存储某些数据(比如用户信息)而保存在浏览器上的小型文本数据。
浏览器会在一定时间内保存它，并在下一次向同一个服务器发送请求时附带这些数据。
Cookie通常被用来进行用户会话管理(比如登录状态)，保存用户的个性化信息(比如语言偏好， 
视频上次播放的位置，网站主题选项等)以及记录和收集用户浏览数据 以用来分析用户行为等。  

在Flask中，如果想要在响应中添加一个cookie，最方便的方法是使用Response类提供的set_cookie()方法。
要使用这个方法，我们需要先使用make_response()方法手动生成一个响应对象，传入响应主体作为参数。这个响应对象默认实例化内置的Response类。  
![p9](p9.png)
set_cookie()方法支持多个参数来设置Cookie的选项:
![p10](p10.png)
将name变量设置到名为name的cookie中：
```python
from flask import Flask, make_response
...
@app.route('/set/<name>')
def set_cookie(name):
   response = make_response(redirect(url_for('hello')))
   response.set_cookie('name', name)
   return response
```
set_cookie视图会在生成的响应报文首部中创 建一个Set-Cookie字段，即“Set-Cookie:name=Grey;Path=/”。  
在Flask中，Cookie可以通过请求对象的cookies属性读取。在修改后的hello视图中，如果没有从查询参数中获取到name的值，就从Cookie中寻找:
```python
from flask import Flask, request
@app.route('/')
@app.route('/hello')
def hello():
   name = request.args.get('name')
   if name is None:
      name = request.cookies.get('name', 'Human') # 从Cookie中获取name值 
   return '<h1>Hello, %s</h1>' % name
```

### session
如果直接把认证信息以明文的方式存储在Cookie里，那么恶意用户就可以通过伪造cookie的 内容来获得对网站的权限，冒用别人的账户。
为了避免这个问题，我们需要对敏感的Cookie内容进行加密。
在Flask中，session对象用来加密Cookie。默认情况下，它会把数据存储在浏览器上一个名为session的cookie里。


