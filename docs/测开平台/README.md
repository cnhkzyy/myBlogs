[toc]



# 测开平台



## 项目分析



### 开发流程

![test](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/test.png)



### 架构设计

+ 架构模式

  前后端分离

+ 前端架构

  vue + elementui + vue router + axios

+ 后端架构

  Django + Django restframework + mysql + swagger

+ 分析用到的技术点

+ 选择哪种数据库

+ 如何管理源代码



### 数据库设计

+ 数据库表的设计至关重要
+ 根据项目需求，设计合适的数据库表
+ 数据库表在前期如果设计不合理，后期随需求增加会变得难以维护



### 测试平台结构

+ 主要为接口测试平台
+ 项目模块
+ 接口模块
+ 用例模块
+ 配置模块
+ 内置函数模块
+ 环境变量模块
+ 套件模块
+ 报告模块
+ 用户模块



## 项目工程搭建



### 搭建项目

与之前搭建的项目步骤一致



### 基本配置

在setting.py文件中

```python
import os
import datetime


# 设置允许域名或ip访问
ALLOWED_HOSTS = ['*']


# 添加日志配置
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,   #是否禁用已存在的日志器
    'formatters': {
        'verbose': {
            'format': '%(asctime)s - [%(levelname)s] - [msg]%(message)s'
        },
        'simple': {
            'format': '%(asctime)s - [%(levelname)s] - %(name)s - [msg]%(message)s - [%(filename)s:%(lineno)d ]'
        },
    },
    'filters': {
        'require_debug_true': {
            ' ()': 'django.utils.log.RequireDebugTrue',
        },
    },
    'handlers': {
        'console': {
            'level': 'DEBUG',
            'filters': ['require_debug_true'],
            'class': 'logging.StreamHandler',
            'formatter': 'simple',
        },
        'file': {
            'level': 'INFO',
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': os.path.join(BASE_DIR, 'logs/mytest.log'),  #日志文件的位置
            'maxBytes': 100 * 1024 * 1024,
            'backupCount': 10,
            'formatter': 'verbose'
        },
    },
    'loggers': {
        'mytest': {   #定义了一个名为mytest的日志(收集)器
            'handlers': ['console', 'file'],
            'propagate': True,   #允许轮转
            'level': 'DEBUG',    #日志器接收的最低日志级别
        }
    }
}
```



## 用户模块



### 准备

1. 在Tools下的Run manage.py Task中输入startapp users，创建一个users的app

2. 在settings.py中配置INSTALLED_APPS```users.apps.UsersConfig```

   

### Browsable API页面认证

#### 添加rest_framwork.urls路由

1.在主路由study_django/urls.py中为接口文档添加认证的路由信息

```python
       path('/api', include('rest_framework.urls')),
```

 这里的认证接口是前后端不分离的

![image-20211012225807946](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211012225807946.png)

   

2.可以看到可浏览的API页面多了一个登录功能。实际上这里登不登录都可以访问接口，没有授任何权限

   ![image-20211012225308092](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211012225308092.png)

3.事实上，在rest_framework的settings.py中默认的授权类的字段值是AllowAny，即任何人都能访问

   ![image-20211012232315470](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211012232315470.png)



#### 指定permission_classes

1.在projects/views.py中进行授权

```python
from rest_framework import permissions



#APIView
#GenericAPIView
#ViewSet不再支持get、post、put、delete等请求方法，只支持action动作
#但是ViewSet这个类中未提供get_object()、get_serializer()等方法
#所以需要继承GenericViewSet
class ProjectsViewSet(viewsets.ModelViewSet):
    '''
     create: 创建项目
     retrieve: 获取项目详情数据
     update: 完整更新项目
     partial_update: 部分更新项目
     destroy: 删除项目
     list: 获取项目列表数据
     names: 获取所有项目名称
     interfaces: 获取指定项目的所有接口数据
    '''

    queryset = Projects.objects.all()
    serializer_class = ProjectModelSerializer
    ordering_fields = ['name', 'leader']
    filter_fields = ['name', 'leader', 'tester']

    permission_classes = [permissions.IsAuthenticated]
    
```

2.几种权限字段的含义

| 字段                      | 含义                                             | 备注             |
| ------------------------- | ------------------------------------------------ | ---------------- |
| IsAuthenticated           | 已经认证的用户才有权限                           |                  |
| AllowAny                  | 任何人都能访问                                   | 默认的是AllowAny |
| IsAdminUser               | 只有管理员才能访问                               |                  |
| IsAuthenticatedOrReadOnly | 已经认证的用户才有权限，未认证的用户只有读的权限 |                  |

3.运行之后发现未认证的用户没有权限

![image-20211012230438605](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211012230438605.png)

认证的用户才有权限

![image-20211012230517787](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211012230517787.png)

4.可以把rest_famework中的授权代码放在项目全局settings.py中，但所有的视图只有认证后才能访问，不建议这样使用（比如注册接口，未认证不能注册）

```pyt
REST_FRAMEWORK =  {
		'DEFAULT_PERMISSION_CLASSES': [
        	'rest_framework.permissions.IsAuthenticated',
    	],
}
```



### Json Web Token认证

+ 常用的认证机制
  + Session认证
  + Token认证
+ Session认证
  + 保存在服务端，增加服务器开销
  + 分布式架构中，难以维持Session会话同步
  + CSRF攻击风险
+ Token认证
  + 保存在客户端
  + 跨语言、跨平台
  + 扩展性强
  + 鉴权性能高