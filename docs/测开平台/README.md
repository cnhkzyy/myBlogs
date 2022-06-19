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

1. 在主路由study_django/urls.py中为接口文档添加认证的路由信息

```python
       path('api/', include('rest_framework.urls')),
```

 这里的认证接口是前后端不分离的

![image-20211012225807946](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211012225807946.png)

   

2. 可以看到可浏览的API页面多了一个登录功能。实际上这里登不登录都可以访问接口，没有授任何权限

   ![image-20211012225308092](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211012225308092.png)

3. 事实上，在rest_framework的settings.py中默认的授权类的字段值是AllowAny，即任何人都能访问

   ![image-20211012232315470](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211012232315470.png)



#### 指定permission_classes

1. 在projects/views.py中进行授权

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

2. 几种权限字段的含义

| 字段                      | 含义                                             | 备注             |
| ------------------------- | ------------------------------------------------ | ---------------- |
| IsAuthenticated           | 已经认证的用户才有权限                           |                  |
| AllowAny                  | 任何人都能访问                                   | 默认的是AllowAny |
| IsAdminUser               | 只有管理员才能访问                               |                  |
| IsAuthenticatedOrReadOnly | 已经认证的用户才有权限，未认证的用户只有读的权限 |                  |

3. 运行之后发现未认证的用户没有权限

![image-20211012230438605](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211012230438605.png)

认证的用户才有权限

![image-20211012230517787](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211012230517787.png)

4. 可以把rest_famework中的授权代码放在项目全局settings.py中，但所有的视图只有认证后才能访问，不建议这样使用（比如注册接口，未认证不能注册）

```pyt
REST_FRAMEWORK =  {
		'DEFAULT_PERMISSION_CLASSES': [
        	'rest_framework.permissions.IsAuthenticated',
    	],
}
```



### Json Web Token认证

#### 常用的认证机制

+ Session认证
+ Token认证

#### Session认证

+ 保存在服务端，增加服务器开销
+ 分布式架构中，难以维持Session会话同步
+ CSRF攻击风险

#### Token认证

+ 保存在客户端
+ 跨语言、跨平台
+ 扩展性强
+ 鉴权性能高

### JWT


#### JWT介绍

1. 由三部分组成：header、playload、signture（以.分隔）

```json
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImFkbWluIiwiZXhwIjoxNjM0MzEzMzMwLCJlbWFpbCI6IjEwNjk5NjY0NzZAcXEuY29tIn0.IcHH8BR7qi6gzONHC8_RXN34zHcUxoYl5ISbey_v684
```

header：eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9

playload：eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImFkbWluIiwiZXhwIjoxNjM0MzEzMzMwLCJlbWFpbCI6IjEwNjk5NjY0NzZAcXEuY29tIn0

signture：IcHH8BR7qi6gzONHC8_RXN34zHcUxoYl5ISbey_v684

2. header

    + 声明类型
    + 声明加密算法，默认为HS256
    + base64加密，可以解密

    base64解密第一部分

    ![image-20211015000025068](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211015000025068.png)

3. playload

    + 存放过期时间，签发用户等
    + 可以添加用户的非敏感信息
    + base64加密，可以解密

    base64解密第二部分

    ![image-20211015000737501](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211015000737501.png)

4. signature

    + 由三部分组成
    
    + 使用base64加密之后的header  + . + 使用base64加密之后的playload + 使用HS256算法加密，同时secret加盐处理
    
      secret是全局settings.py中的SECRET_KEY
    
      ![image-20211015001112034](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211015001112034.png)



#### JWT使用

1. 首先看下，rest_framwork中自带的token认证，不够安全，官方不建议使用。因此需要使用第三方模块jwt

![image-20211013223004045](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211013223004045.png)



2. 安装djangorestframwork-jwt

   ```python
   pip install djangorestframework-jwt
   ```

   

3. 配置settings.py

   restframwork中的认证配置，将其复制下来，放在全局settings.py中

   ![image-20211013223811524](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211013223811524.png)

   ```python
   REST_FRAMEWORK = {
   'DEFAULT_AUTHENTICATION_CLASSES': [	              'rest_framework_jwt.authentication.JSONWebTokenAuthentication',   #有优先级，默认使用JSONWebTokenAuthentication，没有则使用SessionAuthentication，如果会话认证未通过，则抛出异常
    'rest_framework.authentication.SessionAuthentication',
    'rest_framework.authentication.BasicAuthentication'
   ],
   }
   ```

   

4. 在子应用uses下创建urls.py，在里面设置登录的路由

   ```python
   from django.urls import path, include
   from rest_framework_jwt.views import obtain_jwt_token
   
   
   urlpatterns = [
       path('login/', obtain_jwt_token)
   ]
   ```

   实际上这里不用使用obtain_jwt_token.as_view()，因为rest_framework_jwt的views.py中已经使用了

   ![image-20211013224917472](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211013224917472.png)

5. 在主路由中配置子路由

   ```python
   from django.contrib import admin
   from django.urls import path, include, re_path
   from projects.views import *
   from rest_framework.documentation import include_docs_urls
   
   from drf_yasg import openapi
   from drf_yasg.views import get_schema_view
   
   # 声明schema_view
   schema_view = get_schema_view(
       openapi.Info(
           title="Lemon API接口文档平台", # 必传
           default_version='v1', # 必传
           description="这是一个美轮美奂的接口文档",
           terms_of_service="http://api.hello.site",
           contact=openapi.Contact(email="123456@qq.com"),
           license=openapi.License(name="BSD License"),
           ),
           public=True,
           #权限类
   )
   
   # 设置urlpatterns
   urlpatterns = [
       path('admin/', admin.site.urls),
       #方式一
       #path('index/', index),
       #方式二，用的方式比较多
       path('', include('projects.urls')),
       # path('api/', include('rest_framework.urls'))
       path('docs/', include_docs_urls(title='测试平台接口文档', description='这是一个接口文档平台')),
       re_path(r'^swagger(?P<format>\.json|\.yaml)$',
       schema_view.without_ui(cache_timeout=0), name='schema-json'),
       path('swagger/', schema_view.with_ui('swagger', cache_timeout=0), name='schema-swagger-ui'),
       path('redoc/', schema_view.with_ui('redoc', cache_timeout=0), name='schema- redoc'),
   
       path('api/', include('rest_framework.urls')),
       path('users/', include('users.urls'))
   ]
   ```

   6. 未登录调用 projects/1/接口，由于在views中通过permission_classes做了一个授权，因此未登录不能访问

      ```cmd
      http :8000/projects/1/
      ```

      ![image-20211013225609296](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211013225609296.png)

      ![image-20211013225637241](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211013225637241.png)

   调用登录接口，得到token

   ```cmd
   http :8000/users/login/ username=admin password=123456
   ```

   ![image-20211013225944540](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211013225944540.png)



携带token请求id为1的项目详情

使用Authorization: "JWT + 空格 +  <token>"的形式

```cmd
http :8000/projects/1/ Authorization:"JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImFkbWluIiwiZXhwIjoxNjM0MTM3NzcyLCJlbWFpbCI6IjEwNjk5NjY0NzZAcXEuY29tIn0.nLL99-VEYyZ09Uz3Qlb1A23sycU7XASbKHVIw1W0KrM"
```

![image-20211013230557401](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211013230557401.png)

通过加-v可以看到Authorization放在请求体中

![image-20211013230819703](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211013230819703.png)

7. rest_framwork_jwt的settings.py中的几个重要的配置信息

   ![image-20211013231621179](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211013231621179.png)

在全局settings.py中设置

```python
JWT_AUTH = {
    # 默认5分钟过期，可以使用JWT_EXPIRATION_DELTA来设置过期时间为1天
    'JWT_EXPIRATION_DELTA': datetime.timedelta(days=1),
    # 默认的前缀是JWT，可以使用JWT_AUTH_HEADER_PREFIX修改前缀为B
    'JWT_AUTH_HEADER_PREFIX': 'B',
}
```

可以看到使用JWT作为前缀，身份认证失败

![image-20211013232036081](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211013232036081.png)

使用B作为前缀，身份认证成功

![image-20211013232118214](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211013232118214.png)



8. 直接使用Authorization跟上token请求非常不方便，可以使用httpie-jwt-auth插件

   安装httpie-jwt-auth插件

   ```python
   pip install httpie-jwt-auth
   ```

   获取token

   ```cmd
   http :8000/users/login/ username=beck password=123456
   ```

   请求时，传递token（这个用的不多，可以暂时不用）

   ```cmd
   JWT_AUTH_PREFIX=JWT http --auth-type=jwt --auth="your token" :8000/projects/1/
   ```

   设置环境变量，简化token的传递

   ```cmd
   set JWT_AUTH_PREFIX="JWT"
   set JWT_AUTH_TOKEN="your token"
   ```

   --auth-type可以缩写为-A，实际的请求可以简化为

   ```cmd
   http -A jwt :8000/projects/1/ 
   ```

   ![image-20211013233211005](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211013233211005.png)
   
9. 如果不光要返回token，还要返回username

可以查看rest_framework_jwt的settings.py中，有对响应结果负载的操作

![image-20211014234510883](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211014234510883.png)

进入utils.py，发现jwt_response_playload_handler方法返回的是token

![image-20211014234838650](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211014234838650.png)

如果要加上返回username，则需要在utils目录下新建一个jwt_handler.py，在里面重写jwt_response_playload_handler方法

```python
def jwt_response_payload_handler(token, user=None, request=None):
    """
    Returns the response data for both the login and refresh views.
    Override to return a custom response such as including the
    serialized representation of the User.

    Example:

    def jwt_response_payload_handler(token, user=None, request=None):
        return {
            'token': token,
            'user': UserSerializer(user, context={'request': request}).data
        }

    """
    return {
        'token': token,
        'user_id': user.id,
        'username': user.username
    }
```

同时需要修改全局settings.py

```python
JWT_AUTH = {
    # 默认5分钟过期，可以使用JWT_EXPIRATION_DELTA来设置过期时间为1天
    'JWT_EXPIRATION_DELTA': datetime.timedelta(days=1),
    # 默认的前缀是JWT，可以使用JWT_AUTH_HEADER_PREFIX修改前缀为B
    'JWT_AUTH_HEADER_PREFIX': 'B',
    'JWT_RESPONSE_PAYLOAD_HANDLER': 'utils.jwt_handler.jwt_response_payload_handler',
}
```

重新请求登录接口，发现返回了我们想要的user_id和username

![image-20211014235621634](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211014235621634.png)



### 用户注册

#### User自带模型类

django自带User模型类，因此没有必要自己创建User模型

![image-20211017121622123](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211017121622123.png)

![image-20211017121831657](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211017121831657.png)

![image-20211017121907635](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211017121907635.png)

实际上django.contrib.auth是django系统的一个子应用

![image-20211017122410281](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211017122410281.png)

![image-20211017122216536](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211017122216536.png)

![image-20211017122624705](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211017122624705.png)



#### 注册功能需求

| 参数     | 输入/输出  | 校验/描述                    |
| -------- | ---------- | ---------------------------- |
| 用户名   | 输入、输出 | 6-20位，注册用户名不能重复   |
| 邮箱     | 输入       | 符合邮件格式                 |
| 密码     | 输入       | 6-20位，密码和确认密码要一致 |
| 确认密码 | 输入       | 6-20位，密码和确认密码要一致 |
| token    | 输出       | 注册成功之后，需要生成token  |

1. 用什么做校验？

   序列化器的反序列化过程进行校验

2. 创建序列化器，有两种情况：情况1，继承serializer，这些字段自己指定，情况2，继承ModelSerializer，没有必要自己指定字段

3. 在Users下定义serializer.py

   ```python
   from rest_framework import serializers
   from django.contrib.auth.models import User
   from rest_framework_jwt.settings import api_settings
   from rest_framework.validators import UniqueValidator
   
   
   
   # 检查用户名是否已注册
   # def is_unique_username(username):
   #     if User.objects.filter(username=username):
   #         raise serializers.ValidationError('用户名已注册')
   
   
   
   class RegisterSerializer(serializers.ModelSerializer):
       #password_confirm和token在User模型类中没有定义，没有定义的必须在这里定义
       #model没有指定的字段不能在extra_kwargs中定义，只能在下面指定
       password_confirm = serializers.CharField(label='确认密码',
                                                min_length=6,
                                                max_length=20,
                                                help_text='确认密码',
                                                write_only=True,
                                                error_messages={
                                                    'min_length': '仅允许6-20个字符的确认密码',
                                                    'max_length': '仅允许6-20个字符的确认密码',
                                                })
       token = serializers.CharField(label='生成token', read_only=True)
   
       class Meta:
           model = User
           fields = ('id', 'username', 'password', 'email', 'password_confirm', 'token')
           extra_kwargs = {
               #用户名没有必要指定read_only和write_only，默认能输入输出
               'username': {
                   'label': '用户名',
                   'help_text': '用户名',
                   'min_length': 6,
                   'max_length': 20,
                   'error_messages': {
                       'min_length': '仅允许6-20个字符的用户名',
                       'max_length': '仅允许6-20个字符的用户名'
                   },
                   #不用校验username重复，因为在原model中django/contrib/auth/models.py:321中已经定义username为unique
                   #'validators': [ is_unique_username ]
               },
               #email原模型类定义可以前端不传，blank=True，我们要求是必须要传，因为需要加required字段
               'email': {
                   'label': '邮箱',
                   'help_text': '邮箱',
                   'write_only': True,
                   'required': True,
                   #添加邮箱重复校验
                   'validators': [ UniqueValidator(queryset=User.objects.all(), message='邮箱已注册') ]
               },
               'password': {
                'label': '密码',
                   'help_text': '密码',
                'write_only': True,
                   'min_length': 6,
                   'max_length': 20,
                   'write_only': True,
                'error_messages': {
                       'min_length': '仅允许6-20个字符的用户名',
                       'max_length': '仅允许6-20个字符的用户名'
                   }
               }
           }
   
   
       # 检查密码和确认密码是否一致
       def validate(self, attrs):
           if attrs['password_confirm'] != attrs['password']:
               raise serializers.ValidationError('两次输入密码不正确')
           return attrs
   
   
       def create(self, validated_data):
           #del validated_data['password_confirm']
           validated_data.pop('password_confirm')
           #方法一:
           user = User.objects.create_user(**validated_data)
           #方法二：
           #user = super(RegisterSerializer, self).create(validated_data)
           #方法二必须使用set_password，否则密码是以明文保存的
           #user.set_password(validated_data['password'])
           #user.save()
   
           # 手动创建token
           jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
           jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER
   
           payload = jwt_payload_handler(user)
           token = jwt_encode_handler(payload)
   
           user.token = token
           return user
   ```
   
   4. 在Users/views.py中定义view
   
   ```python
   from django.shortcuts import render
   from rest_framework import generics
   from . import serializer
   
   # Create your views here.
   
   # 这里只需要一个post接口，不需要get接口
   # 如果继承APIView，需要自己定义post方法
   # generics模块下有一个CreateAPIView，可以直接使用
   # CreateAPIView继承了GenericAPIView，GenericAPIView使用的时候需要定义queryset和serializer_class
   # 但对于post请求，不需要queryset
   class RegisterView(generics.CreateAPIView):
       serializer_class = serializer.RegisterSerializer
       #这里不需要使用permission_classes指定权限
   ```
   
   5. 在users.urls中定义url路径
   
   ```python
   from django.urls import path, include
   from rest_framework_jwt.views import obtain_jwt_token
   from . import views
   
   
   urlpatterns = [
       path('login/', obtain_jwt_token),
       path('register/', views.RegisterView.as_view())
   ]
   ```
   
   6. 在全局study_django.urls中定义url路径
   
   ```python
   from django.contrib import admin
   from django.urls import path, include, re_path
   from projects.views import *
   from rest_framework.documentation import include_docs_urls
   
   from drf_yasg import openapi
   from drf_yasg.views import get_schema_view
   
   # 声明schema_view
   schema_view = get_schema_view(
       openapi.Info(
           title="Lemon API接口文档平台", # 必传
           default_version='v1', # 必传
           description="这是一个美轮美奂的接口文档",
           terms_of_service="http://api.hello.site",
           contact=openapi.Contact(email="123456@qq.com"),
           license=openapi.License(name="BSD License"),
           ),
           public=True,
           #权限类
   )
   
   # 设置urlpatterns
   urlpatterns = [
       path('admin/', admin.site.urls),
       #方式一
       #path('index/', index),
       #方式二，用的方式比较多
       path('', include('projects.urls')),
       # path('api/', include('rest_framework.urls'))
       path('docs/', include_docs_urls(title='测试平台接口文档', description='这是一个接口文档平台')),
       re_path(r'^swagger(?P<format>\.json|\.yaml)$',
       schema_view.without_ui(cache_timeout=0), name='schema-json'),
       path('swagger/', schema_view.with_ui('swagger', cache_timeout=0), name='schema-swagger-ui'),
       path('redoc/', schema_view.with_ui('redoc', cache_timeout=0), name='schema- redoc'),
   
       path('api/', include('rest_framework.urls')),
       path('users/', include('users.urls'))
   ]
   ```
   
   7. 请求



## 前端项目

### 项目结构分析

```python
lemon-test
	|
	|___public
    		|
        	|___index.html   主页（携带id=app)
    |
    |___src         前端源码目录
    		|
        	|___api
            		|
                	|___api.js    定义的所有接口的请求
            |
            |___assets      静态资源目录
            |
            |___axios       axios的一些配置
            |
            |___components  存放组件的目录
                    |
                    |___common   Header、Home、Sidebar、Tags等组件
                    |
                    |___page     自己写的组件
            |
            |___router    路由
            |
            |___App.vue   根组件
```



###  项目配置修改

#### 修改api.js

```javascript
let host = 'http://127.0.0.1:8000';   //将host修改为后端服务的地址，注意端口8000后不要加/
```



### 跨域请求

前后端联调的时候，在前端页面，点击登录，会阻止请求一个非本域的链接（ip地址或端口不一致都是跨域）

![image-20211020225439428](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211020225439428.png)



#### 后端解决方法：开启跨域

1. 安装django-cors-headers

   ```python
   pip install django-cors-headers
   ```

2. 将corsheaders添加到settings.py文件的INSTALLED_APPS中，尽量放在前面（放在子应用前面）

   ```python
   INSTALLED_APPS = [ 'corsheaders' ]
   ```

3. 添加中间件

   ```python
   MIDDLEWARE = [
       # 需要添加在CommonMiddleware中间件之前
       'corsheaders.middleware.CorsMiddleware',
       'django.middleware.common.CommonMiddleware',
   ]
   ```

4. 添加白名单

   ```python
   # CORS_ORIGIN_ALLOW_ALL为True，指定所有域名(ip)都可以访问后端接口，默认为False
   CORS_ORIGIN_ALLOW_ALL = True
   
   # CORS_ORIGIN_WHITELIST指定能够访问后端接口的ip或域名列表
   # CORS_ORIGIN_WHITELIST = [
   #	'http://127.0.0.1:8000',   # 后端
   #   'http://locahost:8080',    # 前端
   #	'http://192.168.1.63:8080',  #前端
   #	'http://localhost:8000',    # 后端
   # ]
   
   # 允许跨域时携带Cookie，默认为False
   CORS_ALLOW_CREDENTIALS = True
   ```

   

#### 前端解决方法

在public目录下的index.html中配置<meta http-equiv="Content-Security-Policy"



## 数据库设计



### 前序步骤

1. 在项目目录下创建apps目录

![image-20211116234603489](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211116234603489.png)

2. 在apps目录下使用```startapp appname```分别创建configures、debugtalks、envs、interfaces、projects、reports、testcases、testsuites、users等九大子应用

3. 配置setting.py

```python
import sys
import os, datetime

BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
sys.path.insert(0, os.path.join(BASE_DIR, 'apps'))


INSTALLED_APPS = [
    ...
    #注册子应用
    #子应用名.apps.子应用名首字母大写Config
    'projects.apps.ProjectsConfig',
    'interfaces.apps.InterfacesConfig',
    'users.apps.UsersConfig',
    'configures.apps.ConfiguresConfig',
    'debugtalks.apps.DebugtalksConfig',
    'envs.apps.EnvsConfig',
    'reports.apps.ReportsConfig',
    'testsuites.apps.TestsuitesConfig',
    'testcases.apps.TestcasesConfig',
    ...
```



### 基类模型

在utils目录下创建基类模型base_models.py

```python
from django.db import models

class BaseModel(models.Model):
    '''
    数据库表公共字段
    '''
    create_time = models.DateTimeField(auto_now_add=True, verbose_name="创建时间", help_text="创建时间")
    update_time = models.DateTimeField(auto_now=True, verbose_name="更新时间", help_text="更新时间")
    is_delete = models.BooleanField(default=False, verbose_name="逻辑删除", help_text="逻辑删除")

    class Meta:
        #为抽象模型类，用于其他模型来继承，数据库迁移时不会创建BaseModel表
        abstract = True
        verbose_name = "公共字段表"
        db_table = "BaseModel"
```



### 项目模型

apps/projects/models.py

```python
from django.db import models
from utils.base_models import BaseModel


class Projects(BaseModel):
    id = models.AutoField(verbose_name='id主键', primary_key=True, help_text='id主键')
    name = models.CharField('项目名称', max_length=200, unique=True, help_text='项目名称')
    leader = models.CharField('负责人', max_length=50, help_text='项目负责人')
    tester = models.CharField('测试人员', max_length=50, help_text='项目测试人员')
    programmer = models.CharField('开发人员', max_length=50, help_text='开发人员')
    publish_app = models.CharField('发布应用', max_length=100, help_text='发布应用')
    desc = models.CharField('简要描述', max_length=200, null=True, blank=True, default='描述信息',
                            help_text='简要描述')

    class Meta:
        db_table = 'tb_projects'
        verbose_name = '项目信息'
        verbose_name_plural = verbose_name


    def __str__(self):
        return self.name
```



### 接口模型

apps/interfaces/models.py

```python
from django.db import models
from utils.base_models import BaseModel


class Interfaces(BaseModel):
    id = models.AutoField(verbose_name='id主键', primary_key=True, help_text='id主键')
    name = models.CharField('接口名称', max_length=200, unique=True, help_text='接口名称')
    project = models.ForeignKey('projects.Projects', on_delete=models.CASCADE,
                                related_name='interfaces', help_text='所属项目')
    tester = models.CharField('测试人员', max_length=50, help_text='测试人员')
    desc = models.CharField('简要描述', max_length=200, null=True, blank=True, help_text='简要描述')


    class Meta:
        db_table = 'tb_interfaces'
        verbose_name = '接口信息'
        verbose_name_plural = verbose_name


    def __str__(self):
        return self.name
```



### 用例模型

apps/testcases/models.py

```python
from django.db import models
from utils.base_models import BaseModel


class TestCases(BaseModel):
    id = models.AutoField(verbose_name='id主键', primary_key=True, help_text='id主键')
    name = models.CharField('用例名称', max_length=50, unique=True, help_text='用例名称')
    interface = models.ForeignKey('interfaces.Interfaces', on_delete=models.CASCADE,
                                  help_text='所属接口')
    include = models.TextField('前置', null=True, help_text='用例执行前置顺序')
    author = models.CharField('编写人员', max_length=50, help_text='编写人员')
    request = models.TextField('请求信息', help_text='请求信息')


    class Meta:
        db_table = 'tb_testcases'
        verbose_name = '用例信息'
        verbose_name_plural = verbose_name


    def __str__(self):
        return self.name
```



### 配置模型

apps/configures/models.py

```python
from django.db import models
from utils.base_models import BaseModel


class Configures(BaseModel):
    id = models.AutoField(verbose_name='id主键', primary_key=True, help_text='id主键')
    name = models.CharField('配置名称', max_length=50, help_text='配置名称')
    interface = models.ForeignKey('interfaces.Interfaces', on_delete=models.CASCADE,
                                  related_name='configures', help_text='所属接口')
    author = models.CharField('编写人员', max_length=50, help_text='编写人员')
    request = models.TextField('请求信息', help_text='请求信息')

    class Meta:
        db_table = 'tb_configures'
        verbose_name = '配置信息'
        verbose_name_plural = verbose_name


    def __str__(self):
        return self.name
```



### 套件模型

apps/testsuites/models.py

```python
from django.db import models
from utils.base_models import BaseModel


class TestSuites(BaseModel):
    id = models.AutoField(verbose_name='id主键', primary_key=True, help_text='id主键')
    name = models.CharField('套件名称', max_length=200, unique=True, help_text='套件名称')
    project = models.ForeignKey('projects.Projects', on_delete=models.CASCADE,
                                related_name='testsuites', help_text='所属项目')
    include = models.TextField('包含的接口', null=False, help_text='包含的接口')

    class Meta:
        db_table = 'tb_testsuites'
        verbose_name = '套件信息'
        verbose_name_plural = verbose_name


    def __str__(self):
        return self.name
```



### 内置函数模型

apps/debugtalks/models.py

```python
from django.db import models
from utils.base_models import BaseModel


class DebugTalk(BaseModel):
    id = models.AutoField(verbose_name='id主键', primary_key=True, help_text='id主键')
    name = models.CharField('debugtalk文件名称', max_length=200, default='debugtalk.py',
                            help_text='debugtalk.py')
    debugtalk = models.TextField(null=True, default='#debugtalk.py', help_text='debugtalk.py文件')
    project = models.OneToOneField('projects.Projects', on_delete=models.CASCADE,
                                   related_name='debugtalks',  help_text='所属项目')


    class Meta:
        db_table = 'tb_debugtalks'
        verbose_name = 'debugtalk.py文件'
        verbose_name_plural = verbose_name


    def __str__(self):
        return self.name
```



###  环境变量模型

apps/envs/models.py

```python
from django.db import models
from utils.base_models import BaseModel


class Envs(BaseModel):
    id = models.AutoField(verbose_name='id主键', primary_key=True, help_text='id主键')
    name = models.CharField(verbose_name='环境名称', max_length=200, unique=True, help_text='环境名称')
    base_url = models.URLField(verbose_name='请求base url', max_length=200, help_text='请求base url')
    desc = models.CharField(verbose_name='简要描述', max_length=200, help_text='简要描述')

    class Meta:
        db_table = 'tb_envs'
        verbose_name = '环境信息'
        verbose_name_plural = verbose_name


    def __str__(self):
        return self.name
```



### 测试报告模型

apps/reports/models.py

```python
from django.db import models
from utils.base_models import BaseModel


class Reports(BaseModel):
    id = models.AutoField(verbose_name='id主键', primary_key=True, help_text='id主键')
    name = models.CharField('报告名称', max_length=200, unique=True, help_text='报告名称')
    result = models.BooleanField('执行结果', default=1, help_text='执行结果')   #1为成功
    count = models.IntegerField('用例总数', help_text='总用例数')
    success = models.IntegerField('成功总数', help_text='成功总数')
    html = models.TextField('报告HTML源码', help_text='报告HTML源码', null=True, blank=True, default='')
    summary = models.TextField('报告详情', help_text='报告详情', null=True, blank=True, default='')

    class Meta:
        db_table = 'tb_reports'
        verbose_name = '测试报告'
        verbose_name_plural = verbose_name


    def __str__(self):
        return self.name
```



## 项目模块



### 项目需求

| 参数                    | 输入/输出            | 校验/描述                     |
| ----------------------- | -------------------- | ----------------------------- |
| 项目名(name)            | 输入、输出           | 最大200个字符，项目名不能重复 |
| 项目负责人(leader)      | 输入、输出           | 最大50个字符                  |
| 测试人员(tester)        | 输入、输出           | 最大50个字符                  |
| 开发人员(programmer)    | 输入、输出           | 最大50个字符                  |
| 发布应用名(publish_app) | 输入、输出           | 最大100个字符                 |
| 简要描述(desc)          | 可输入也可输入、输出 | 最大200个字符                 |
| 创建时间(create_time)   | 输出                 |                               |
| 更新时间(update_time)   | 不输入、不输出       |                               |
| 逻辑删除(is_delete)     | 不输入、不输出       |                               |

+ 删除项目时，只进行逻辑删除(is_delete修改为True)
+ 获取项目列表信息时，要求能获取此项目下的接口总数、用例总数、配置总数、套件总数，同时要求输出创建时间，且需要将创建时间格式化为2019-06-24 00:36:55
+ 要求提供获取此项目下的所有项目名的接口
+ 要求提供获取次项目下的所有接口信息的接口



### 逻辑删除项目

apps/projects/views.py

```python
class ProjectsViewSet(ModelViewSet):
    '''
    list：返回项目（多个）列表数据
    create：创建项目
    retrieve：返回项目（单个）详情数据
    update：更新（全）项目
    partial_update：更新（部分）项目
    destory：删除项目
    names：返回所有项目ID和名称
    interfaces：返回某个项目的所有接口信息（ID和名称）
    '''
    queryset = Projects.objects.filter(is_delete=False)
    serializer_class = ProjectModelSerializer
    permission_classes = [permissions.IsAuthenticated]
    ordering_fields = ('id', 'name')


    def perform_destroy(self, instance: Projects):
        instance.is_delete = True
        instance.save()   #逻辑删除
```

### 获取所有的项目名

#### 应用场景

新增接口的时候

![image-20220619194707312](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220619194707312.png)

#### 代码实现

apps/projects/serializer.py

```python
class ProjectsNameSerializer(serializers.ModelSerializer):
    class Meta:
        model = Projects
        fields = ('id', 'name')
```

为什么需要项目id值？

因为apps/interfaces/models.py中有一个外键project，传入的是对象，保存的是project_id

```python
class Interfaces(BaseModel):
    id = models.AutoField(verbose_name='id主键', primary_key=True, help_text='id主键')
    name = models.CharField('接口名称', max_length=200, unique=True, help_text='接口名称')
    project = models.ForeignKey('projects.Projects', on_delete=models.CASCADE, related_name='interfaces', help_text='所属项目')
    tester = models.CharField('测试人员', max_length=50, help_text='测试人员')
    desc = models.CharField('简要描述', max_length=200, null=True, blank=True, help_text='简要描述')


    class Meta:
        db_table = 'tb_interfaces'
        verbose_name = '接口信息'
        verbose_name_plural = verbose_name
```

apps/projects/views.py

```python
@action(methods=['get'], detail=False)
def names(self, request, *args, **kwargs):
    queryset = self.get_queryset()
    serializer = ProjectsNameSerializer(instance=queryset, many=True)
    return Response(serializer.data)
```

### 获取所有的接口信息

#### 应用场景

新增用例的时候。先通过项目获取接口的信息，再选择接口新增用例

#### 方式一：使用序列化器

![image-20220619195713440](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220619195713440.png)

关于related_name

如果apps/interfaces/model.py中不定义related_name，那么在apps/projects/serializer.py中要使用interfaces_set，如果定义了related_name='interfaces'，那么interfaces_set要改成interfaces，到时候接口会展示一个字段：interfaces

apps/interfaces/model.py

```python
class Interfaces(BaseModel):
    id = models.AutoField(verbose_name='id主键', primary_key=True, help_text='id主键')
    name = models.CharField('接口名称', max_length=200, unique=True, help_text='接口名称')
    project = models.ForeignKey('projects.Projects', on_delete=models.CASCADE,
related_name='interfaces', help_text='所属项目')
    tester = models.CharField('测试人员', max_length=50, help_text='测试人员')
    desc = models.CharField('简要描述', max_length=200, null=True, blank=True, help_text='简要描述')
```

apps/projects/serializer.py

```python
class InterfaceNameSerializer(serializers.ModelSerializer):
    class Meta:
        model = Interfaces
        fields = ('id', 'name', 'tester')


class InterfacesByProjectIdSerializer(serializers.ModelSerializer):
    # interfaces_set = InterfaceNameSerializer(read_only=True, many=True)
    interfaces = InterfaceNameSerializer(read_only=True, many=True)


    class Meta:
        model = Projects
        # fields = ('id', 'interfaces_set')
        fields = ('id', 'interfaces')
```

apps/projects/views.py

```python
@action(methods=['get'], detail=True)
def interfaces(self, request, pk=None):
    serializer = InterfacesByProjectIdSerializer(self.get_object())
    return Response(data=serializer.data)
```

可以看到项目id=1的接口信息里的响应结果多了一个字段：interfaces

![image-20220619205257567](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220619205257567.png)

 

#### 方式二：手动定义

不使用序列化器定义，手动定义，可以避免这个字段

apps/projects/views.py

```python
@action(methods=['get'], detail=True)
def interfaces(self, request, pk=None):
    interface_objs = Interfaces.objects.filter(project_id=pk, is_delete=False)
    one_list = []
    for obj in interface_objs:
        one_list.append({
            'id': obj.id,
            'name': obj.name
        })
        # serializer = InterfacesByProjectIdSerializer(self.get_object())
        return Response(data=one_list)
    # return Response(data=serializer.data)
```

![image-20220619224315849](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220619224315849.png)



### 获取项目列表信息

#### 应用场景

获取项目列表功能，需要做的是：

1. 把时间格式化

2. 鼠标悬浮在项目名字上面会自动获取项目对应的接口数、套件数、用例数、配置数

![image-20220619224823918](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220619224823918.png)

![image-20220619225222126](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220619225222126.png)



#### 代码实现

对serializer.data进行修改，使得它能实现时间格式化，并自动获取项目对应的接口数、套件数、用例数、配置数。通过在list方法中，将serializer.data传入方法get_count_by_project来实现这一目的

apps/projects/views.py

```python
def list(self, request, *args, **kwargs):
    queryset = self.filter_queryset(self.get_queryset())

    page = self.paginate_queryset(queryset)
    if page is not None:
        serializer = self.get_serializer(page, many=True)
        datas = serializer.data
        datas = get_count_by_project(datas)
        return self.get_paginated_response(datas)

    serializer = self.get_serializer(queryset, many=True)
    datas = serializer.data
    datas = get_count_by_project(datas)
    return Response(datas)
```



#### 创建时间格式化

1. 存放在数据库的时间是UTC时间，如2021-11-21 03:07:58.947151，读取出来后会自动转化为东八区时间
2. 对取出来的时间进行格式化

apps/projects/utils.py

```python
def get_count_by_project(datas):
    datas_list = []
    for item in datas:
        match = re.search(r'(.*)T(.*)\..*?', item['create_time'])
        item['create_time'] = match.group(1) + ' ' +  match.group(2)
    	datas_list.append(item)
    return datas_list
```

#### 获取接口总数

apps/projects/utils.py

```python
def get_count_by_project(datas):
    datas_list = []
    for item in datas:
        match = re.search(r'(.*)T(.*)\..*?', item['create_time'])
        item['create_time'] = match.group(1) + ' ' +  match.group(2)

        project_id = item['id']
        #1.values('id')：根据interfaces中的id分组
        #2.annotate：对分组后的数据统计testcases中信息，testcases为设置别名
        #3.filter对分组后的信息进行过滤
        interfaces_testcases_objs = Interfaces.objects.values('id').annotate(testcases=Count('testcases')).filter(project_id=project_id, is_delete=False)
        #接口总数
        interfaces_count = interfaces_testcases_objs.count()
        datas_list.append(item)
    return datas_list
```

debug查看interfaces_testcases_objs返回的是一个查询集：

id：接口id为1

testcases: 接口id为1的用例数量为1

![image-20220619233542575](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220619233542575.png)

原生sql可以使用interfaces_testcases_objs.query查看

![image-20220619233716232](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220619233716232.png)

```mysql
SELECT `tb_interfaces`.`id`, COUNT(`tb_testcases`.`id`) AS `testcases` FROM `tb_interfaces` LEFT OUTER JOIN `tb_testcases` ON (`tb_interfaces`.`id` = `tb_testcases`.`interface_id`) WHERE (NOT `tb_interfaces`.`is_delete` AND `tb_interfaces`.`project_id` = 1) GROUP BY `tb_interfaces`.`id` ORDER BY NULL
```



#### 获取用例总数

apps/projects/utils.py

```python
def get_count_by_project(datas):
    datas_list = []
    for item in datas:
        match = re.search(r'(.*)T(.*)\..*?', item['create_time'])
        item['create_time'] = match.group(1) + ' ' +  match.group(2)

        project_id = item['id']
        #1.values('id')：根据interfaces中的id分组
        #2.annotate：对分组后的数据统计testcases中信息，testcases为设置别名
        #3.filter对分组后的信息进行过滤
        interfaces_testcases_objs = Interfaces.objects.values('id').annotate(testcases=Count('testcases')).\
            filter(project_id=project_id, is_delete=False)
        #接口总数
        interfaces_count = interfaces_testcases_objs.count()
        #用例总数
        testcases_count = 0
        for one_dict in interfaces_testcases_objs:
            testcases_count += one_dict['testcases']
            
        datas_list.append(item)
    return datas_list
```

#### 获取配置总数

apps/projects/utils.py

```python
def get_count_by_project(datas):
    datas_list = []
    for item in datas:
        match = re.search(r'(.*)T(.*)\..*?', item['create_time'])
        item['create_time'] = match.group(1) + ' ' +  match.group(2)

        project_id = item['id']
        #1.values('id')：根据interfaces中的id分组
        #2.annotate：对分组后的数据统计testcases中信息，testcases为设置别名
        #3.filter对分组后的信息进行过滤
        interfaces_testcases_objs = Interfaces.objects.values('id').annotate(testcases=Count('testcases')).\
            filter(project_id=project_id, is_delete=False)
        #接口总数
        interfaces_count = interfaces_testcases_objs.count()
        #用例总数
        testcases_count = 0
        for one_dict in interfaces_testcases_objs:
            testcases_count += one_dict['testcases']

        #interfaces_configures_objs是一个查询集，每个元素表示不同接口对应的配置数量
        interfaces_configures_objs = Interfaces.objects.values('id').annotate(configures=Count('configures')). \
            filter(project_id=project_id, is_delete=False)

        #配置总数
        #将不同接口对应的配置数量相加
        configures_count = 0
        for one_dict in interfaces_configures_objs:
            configures_count += one_dict['configures']
            
        datas_list.append(item)
    return datas_list
```

#### 套件总数

apps/projects/utils.py

```python
def get_count_by_project(datas):
    datas_list = []
    for item in datas:
        match = re.search(r'(.*)T(.*)\..*?', item['create_time'])
        item['create_time'] = match.group(1) + ' ' +  match.group(2)

        project_id = item['id']
        #1.values('id')：根据interfaces中的id分组
        #2.annotate：对分组后的数据统计testcases中信息，testcases为设置别名
        #3.filter对分组后的信息进行过滤
        interfaces_testcases_objs = Interfaces.objects.values('id').annotate(testcases=Count('testcases')).\
            filter(project_id=project_id, is_delete=False)
        #接口总数
        interfaces_count = interfaces_testcases_objs.count()
        #用例总数
        testcases_count = 0
        for one_dict in interfaces_testcases_objs:
            testcases_count += one_dict['testcases']

        interfaces_configures_objs = Interfaces.objects.values('id').annotate(configures=Count('configures')). \
            filter(project_id=project_id, is_delete=False)

        #配置总数
        configures_count = 0
        for one_dict in interfaces_configures_objs:
            configures_count += one_dict['configures']

        #套件总数
        testsuites_count = TestSuites.objects.filter(project_id=project_id, is_delete=False).count()

        #将数据添加进item字典
        item['iterfaces'] = interfaces_count
        item['testsuites'] = testsuites_count
        item['testcases'] = testcases_count
        item['configures'] = configures_count

        datas_list.append(item)
    return datas_list
```

#### 测试接口

获取项目列表接口里，已经有了接口总数、套件总数、用例总数、配置总数，并且创建时间已经格式化

![image-20220619235229915](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220619235229915.png)

### 分页

#### 自定义分页

utils/pagination.py

```python
from rest_framework import pagination

class PageNumberPaginationManual(pagination.PageNumberPagination):
    max_page_size = 50
    page_size_query_param = 'size'
    #方便在线接口文档查看描述信息
    page_query_description = '第几页'
    page_size_query_description = '每页几条'


    def get_paginated_response(self, data):
        response = super(PageNumberPaginationManual, self).get_paginated_response(data)
        response.data['total_pages'] = self.page.paginator.num_pages
        response.data['current.page_num'] = self.page.number
        return response
```

#### 测试接口

![image-20220620001623610](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220620001623610.png)





