[toc]

# Django基础

## 简介

### 1.为什么要使用框架来开发

+ 站在巨人的肩膀上

+ 提升开发效率

+ 只关注业务逻辑的实现，不关心底层建设

### 2.Django vs Flask

Django

+ 大而全
+ 入门简单
+ 开发效率极高
+ 最流行
+ 类似于精装修的房子

Flask

+ 轻量级
+ 定制化程度高
+ 流行
+ 高手的玩偶
+ 类似于毛坯房

### 3.特点

+ 提供创建项目工程自动化工具
+ 数据库ORM支持
+ 模板
+ 表单
+ Admin管理站点
+ 文件管理
+ 认证权限
+ session机制
+ 缓存



## 创建工程

### 1.创建虚拟环境

+ virtualenv
+ virtualenvwrapper
+ python -m venv 虚拟环境名

### 2.安装Django

+ 进入到虚拟环境中
+ ```pip install -i https://pypi.douban.com/simple Django```

### 3.创建项目

+ ```django-admin startproject 项目名```
+ 注意如果在pycharm的工程下创建同名的项目，要加一个点```django-admin startproject 项目名 .```

### 4.运行项目

+ 第一种方法：```python manage.py runserver```运行项目默认8080端口
+ 第二种方法：Tools-run manage.py task进入manage.py运行环境 直接输入 runserver启动项
+ 第三种方法：```python manage.py runserver ip:端口```修改运行的端口 ip:端口号（端口号要大于1024）
+ 第四种方法：可以创建运行器，在右上角 使用add configure来添加

### 5.项目结构
+ learn_django 项目同名的目录 主要存放相关配置信息
+ learn_django/init.py 当前mydev为一个包
+ learn_django/asgi.py 主要用于存放ASGI异步请求的入口配置信息
+ learn_django/settings.py 存放的是项目全局配置信息
+ learn_djangov/urls.py 主要存放项目的路由信息
+ learn_django/wsgi.py 主要用于存放WSGI协议服务的入口配置信息（一般在部署的时候使用）
+ db.sqlite3: 默认的关系型文本数据库
+ manage.py： 为命令行管理工具，用于开发阶段的项目的启动 管理数据迁移 静态文件收集等等

  

### 6.修改时区

learn_django/settings.py

```python
LANGUAGE_CODE = 'zh-hans'
TIME_ZONE = 'Asia/Shanghai'
USE_I18N = True
USE_L10N = True
USE_TZ = True
```



## 创建子应用

### 1.定义

+ 子应用的作用 · 实现业务功能模块的复用
+ 将工程项目拆分为不同的子功能模块，以子应用的形式存在
+ 各功能模块之间可以保持相对的独立
+ 实现路由的分发，便于管理各模块的url，减少代码维护成本

### 2.创建

```python manage.py startapp 项目名(如：projects)```



### 3.子应用结构

projects/migrations: 存放数据库迁移脚本和迁移历史记录等信息

projects/admin.py: admin后台站点的相关配置（需要后台站点时才会用到）

projects/apps.py: 为app_label的相关配置，很少使用

projects/models.py: 存放数据库模型相关信息

projects/tests.py: 对当前子应用进行自测，写单元测试

projects/views.py: 定义业务逻辑（向前端返回的页面）



### 4.注册

在全局配置文件settings.py中的INSTALLED_APPS中，对子应用进行注册

```python
# 子应用名.apps.子应用名首字母大写Config
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'projects.apps.ProjectsConfig'',
]

或者直接应用名 'projects'
```



### 5.创建视图函数

在projects/views.py下创建视图函数

views.py

```python
from django.shortcuts import render
from django.http import HttpResponse

# Create your views here.

def index(request):
    '''
    index视图
    :param request:
    :return:
    '''
    return HttpResponse("<h1>Hello, Python测试</h1>")
```

### 6.在全局路由表中添加路由信息

learn_django/urls.py

```python
from django.contrib import admin
from django.urls import path
from projects.views import index

urlpatterns = [
    path('admin/', admin.site.urls),
    path('index/', index)
]
```

运行后

![image-20210707231922801](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210707231922801.png)

## 路由

### 1.全局路由

全局路由：和项目名learn_django同名的目录下的urls.py为全局路由

全局路由匹配规则：

1.urlpatterns为固定名称的列表
2.列表中的一个元素，就代表一条路由
3.从上到下进行匹配，如果能匹配上，Django会导入和调用path函数第二个参数指定的视图或者去子路由中匹配
4.如果匹配不上，会自动抛出一个404异常（默认为404页面，状态码为404）



### 2.子路由

子路由匹配规则：

1.每一个应用（模块）都会维护一个子路由（当前应用的路由信息）
2.跟主路由一样，也是从上到下进行匹配
3.能匹配上，则执行path第二个参数指定的视图，匹配不上，则抛出404异常

### 3.示例

主路由learn_django/urls.py

```python
from django.contrib import admin
from django.urls import path, include
from projects.views import index


#全局路由配置
#1.urlpatterns为固定名称的列表
#2.列表中的一个元素，就代表一条路由
#3.从上到下进行匹配，如果能匹配上，Django会导入和调用path函数第二个参数指定的视图或者去子路由中匹配
#4.如果匹配不上，会自动抛出一个404异常（默认为404页面，状态码为404）
urlpatterns = [
    path('admin/', admin.site.urls),
    path('projects/', include('projects.urls')),
]
```

子路由projects/urls.py

```python
from django.urls import path
from .views import index

urlpatterns = [
    path('index/', index)
]
```

![image-20210710172835307](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210710172835307.png)

![image-20210710172935550](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210710172935550.png)

![image-20210710173017218](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210710173017218.png)

## 视图

### 1.函数视图

+ request是HttpRequest对象，包含前端用户的所有请求信息

+ 必须返回一个HttpResponse对象或子对象

```python
def index(request):
    return HttpResponse("<h1>Hello, Python测试</h1>")
```

**痛点**

if语句中有好几千行代码时，函数非常巨大，不利于维护，因此需要引入类视图

```python
#函数视图
def index(request):
    '''
    index视图
    :param request:request是HttpRequest对象，包含前端用户的所有请求信息
    :return:必须返回一个HttpResponse对象或子对象
    '''
    if request.method == "GET":
        # 此处有好几千行代码
        return HttpResponse("<h1>GET请求：Hello, Python测试</h1>")
    elif request.method == "POST":
        # 此处有好几千行代码
        return HttpResponse("<h1>POST请求：Hello, Python测试</h1>")
    else:
        # 此处有好几千行代码
        return HttpResponse("<h1>其他请求：Hello, Python测试</h1>")
```



### 2.类视图

- 一定要继承View父类 或者View的子类
- 可以定义get post put delete 方法 来实现get请求 post请求 put请求 delete请求
- get post put delete方法名称固定，且均为小写
- 实例方法的第二个参数为HttpRequest对象

views.py

```python
from django.http import HttpResponse
from django.views import View

##类视图
class IndexView(View):
    '''
    index主页类视图
    '''
    def get(self, request):
        #get请求
        return HttpResponse("<h1>GET请求：Hello, Python测试</h1>")

    def post(self, request):
        return HttpResponse("<h1>POST请求：Hello, Python测试</h1>")

    def delete(self, request):
        return HttpResponse("<h1>DELETE请求：Hello, Python测试</h1>")

    def put(self, request):
        return HttpResponse("<h1>PUT请求：Hello, Python测试</h1>")
```

projects/urls.py

如果为类视图，path第二个参数为类视图名.as_view()

```python
from django.urls import path
from projects import views

urlpatterns = [
    #如果为类视图，path第二个参数为类视图名.as_view()
    path('', views.IndexView.as_view()),
]
```



## 请求与响应

### 1. 请求参数类型

利用HTTP协议向服务器传参的几种途径

1）查询字符串传参

如：ip:port/index/?name=Tim&age=18  url后面的?参数，可以使用request.GET获取，返回QueryDict对象，可通过request.GET[key]、request.GET.get(key)、request.GET.getlist()获取参数值

**示例1**

pycharm打开debug模式，在浏览器中请求http://127.0.0.1:8000/projects?name=beck&age=29

![image-20210712233255424](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210712233255424.png)

![image-20210712233423255](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210712233423255.png)

![image-20210712233538475](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210712233538475.png)

**示例2**

查询字符串中如果有相同的key，后面的值会覆盖前面的值

比如在浏览器中请求：http://127.0.0.1:8000/projects/?name=beck&age=29&name=Tim

![image-20210712234030121](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210712234030121.png)

![image-20210712234105464](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210712234105464.png)

**示例3**

如果要获取多个相同key值的value，可以使用getlist('name')方法

![image-20210712234211308](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210712234211308.png)

2）请求体参数

 **form表单传参**

使用request.POST方法，获取application/x-www-form-urlencoded类型的参数

**示例1**

pycharm打开debug模式，使用postman发送一个post请求

![image-20210712235111723](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210712235111723.png)

![image-20210712235149112](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210712235149112.png)

![image-20210712235248899](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210712235248899.png)

![image-20210712235354514](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210712235354514.png)

或者使用request.POST.get('name')来获取name的值

![image-20210712235515753](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210712235515753.png)

**json格式参数**

使用request.body方法，获取application/json类型的参数

 **上传文件**

使用request.body方法，获取到文件的二进制格式数据,新建一个文件写入

3）路径参数

如：ip:port/index/10/

把参数伪装成路径，传递到后端

在路由中配置路径：< url类型转换器:路径参数名 > int path uuid slug；请求函数中配置请求参数：def post(self, request, pk)

```python
path('index04/<int:pk>/<username>/', views.IndexPage.as_view())
```



### 2.响应

视图中必须返回HTTPResponse对象或子对象

HttpResponse(content=响应体, content_type=响应体数据类型, status=状态码) HttpResponse对象 第一个参数为字符串或者字节类型 会将字符串内容返回到前端 jsonResponse

**模板渲染1**

render()主要用于渲染模板，生成一个html页面，第一个参数为request，第二个参数为在templates目录下的模板名，第三个参数为context，只能传字典

views.py

```python
from django.shortcuts import render
from django.views import View

##类视图
class IndexView(View):
    '''
    index主页类视图
    '''
    def get(self, request):
        #get请求
        return render(request, 'demo.html')
```

templates/demo.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>项目列表页</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css" integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" crossorigin="anonymous">
    <style>
        .container-fluid {
            margin-top: 100px;
        }
        .table-striped th. .table-striped td {
            text-align: center;
        }
    </style>
</head>
<body>
    <div class="container-fluid">
        <div class="row" style="margin-bottom: 30px">
            <div class="col"></div>
            <div class="col"><h2 class="text-center text-info">项目列表信息</h2></div>
            <div class="col"></div>
        </div>
        <div class="row">
            <div class="col"></div>
            <div class="col">
                <table class="table table-striped">
                    <thead class="thead-dark">
                        <tr>
                            <th scope="col">序号</th>
                            <th scope="col">项目名称</th>
                            <th scope="col">项目负责人</th>
                            <th scope="col">应用名称</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr>
                            <th scope="row">1</th>
                            <td>前程贷项目</td>
                            <td>可优</td>
                            <td>P2P平台应用</td>
                        </tr>
                         <tr>
                            <th scope="row">2</th>
                            <td>探索火星项目</td>
                            <td>优优</td>
                            <td>吊炸天应用</td>
                         </tr>
                         <tr>
                            <th scope="row">3</th>
                            <td>无比牛逼的项目</td>
                            <td>可可</td>
                            <td>神秘应用</td>
                         </tr>
                    </tbody>
                </table>
            </div>
            <div class="col"></div>
        </div>
    </div>
</body>
</html>
```

![image-20210710184146547](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210710184146547.png)

**模板渲染2**

views.py

```python
from django.shortcuts import render
from django.views import View

##类视图
class IndexView(View):
    '''
    index主页类视图
    '''
    def get(self, request):
        #get请求
        #return HttpResponse("<h1>GET请求：Hello, Python测试</h1>")
        #从数据库中读取项目数据
        datas = [
            {
                'project_name': '前程贷项目',
                'leader': '可优',
                'app_name': 'P2P平台应用'
            },
            {
                'project_name': '探索火星项目',
                'leader': '优优',
                'app_name': '吊炸天应用'
            },
            {
                'project_name': '无比牛逼的项目',
                'leader': '可可',
                'app_name': '神秘应用'
            }
        ]
        return render(request, 'index.html', locals())
```

templates/index.html

locals()，获取当前命名空间中的所有变量值，存放在一个字典中

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>项目列表页</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css" integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" crossorigin="anonymous">
    <style>
        .container-fluid {
            margin-top: 100px;
        }
        .table-striped th. .table-striped td {
            text-align: center;
        }
    </style>
</head>
<body>
    <div class="container-fluid">
        <div class="row" style="margin-bottom: 30px">
            <div class="col"></div>
            <div class="col"><h2 class="text-center text-info">项目列表信息</h2></div>
            <div class="col"></div>
        </div>
        <div class="row">
            <div class="col"></div>
            <div class="col">
                <table class="table table-striped">
                    <thead class="thead-dark">
                        <tr>
                            <th scope="col">序号</th>
                            <th scope="col">项目名称</th>
                            <th scope="col">项目负责人</th>
                            <th scope="col">应用名称</th>
                        </tr>
                    </thead>
                    <tbody>
                    {% for project in datas %}
                        <tr>
                            <th scope="row">{{ forloop.counter }}</th>
                            <td>{{ project.project_name }}</td>
                            <td>{{ project.leader }}</td>
                            <td>{{ project.app_name }}</td>
                        </tr>
                    {% endfor %}
                    </tbody>
                </table>
            </div>
            <div class="col"></div>
        </div>
    </div>
</body>
</html>
```

![image-20210710185759482](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210710185759482.png)

## MVT模式和两种开发模式

### MVT

M：Model，与MVC中的M功能相同，负责和数据库交互，进行数据处理

V：View，与MVC中的C功能相同，接收请求，进行业务处理，返回响应

T：Template，与MVC中的V功能相同，负责构造要返回的html页面



### 两种开发模式

1.前后不分离

前端数据的展示由后端来控制，后端渲染或重定向页面  耦合严重  返回的为html页面，实用性差，拓展性差，只能用于浏览器，与其他终端不适配



2.前后分离

前后端独立，后端只需对数据进行处理，向前端提供数据，由前端负责页面的展示  解耦合  前后端可同时进行开发，缩小业务上线周期  大部分情况下，前端发送json格式参数，后端同样以json格式数据返回  扩展性、适应性好，可多终端运行同一套接口，如PC、APP、小程序等