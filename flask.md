# Flask Note 
2022/5/2 
Start learning Flask :)

# 安装
pipenv: 基于pip的Python包管理工具  
安装pipenv：pip install pipenv  
pipenv install：在当前工作目录创建虚拟环境, 还会在当前目录下创建Pipfile, Pipfile.lock文件，前者记录依赖包列表，后者记录固定版本的详细依赖包列表。当我们使用pipenv更新/创建/删除依赖包时，这两个文件会自动更新。这种方式代替了传统的pip+requirements.txt方式，requirements.txt需要手动维护。

pipenv shell：激活虚拟环境

![p1](p1.png)

安装flask:  
pipenv install flask  

更新flask:  
pipenv update flask  

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
