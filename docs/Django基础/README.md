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

### 全局路由

全局路由：和项目名learn_django同名的目录下的urls.py为全局路由

全局路由配置：

1.urlpatterns为固定名称的列表
2.列表中的一个元素，就代表一条路由

