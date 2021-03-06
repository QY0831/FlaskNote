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

![p1](pics/p1.png)

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
![p2](pics/p2.png)
flask run命令默认监听 127.0.0.1:5000 地址。  
![p3](pics/p3.png)  
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
