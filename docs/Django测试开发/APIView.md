## 特性

 继承了Django中的View，进行了拓展，是可浏览的API页面

 具备认证、授权、限流、不同请求数据的解析



## APIView与View的不同

1）传入到视图方法中的是Request对象，而不是Django中的HttpRequest对象

 2）视图方法可以返回Response对象，会为响应数据处理（render）为符合前端要求的格式

 3）任何APIException异常都会被捕获到，并且处理成合适的响应信息

 4）在进行dispatch()分发前，会对请求进行身份认证、权限检查、流量控制

#### 

## 常用类属性

 authentication_classes 列表或元组，身份认证类

 permission_classes 列表或元组，权限检查类

 throttle_classes 列表或元组，流量控制类



## Request

1）对Django中的HttpRequest进行了拓展

接收到请求后会根据请求头中的Content-Type，自动进行解析，解析为字典形式保存到Request对象

无论前端发送哪种格式的数据，都可以以统一的方式读取

 2）.data —— 获取json格式的参数、form表单的参数、files（类似于.body 、.POST、.FILES ）

利用了REST framework的parsers解析器，不仅支持表单类型数据，也支持JSON数据

可以对POST、PUT、PATCH的请求体参数进行解析

Render类 Parse解析类

 3）.query_params —— 获取查询字符串参数（类似于.GET ）

 4）支持Django HttpRequest中所有的对象和方法



## Response

1）对Django中的HTTPResponse进行了拓展

 2）请求头中的Accept默认为：application/json，浏览器访问时自动设置为：text/html，返回html页面

 3）指定响应默认渲染类

```python
# 在全局settings中的REST_FRAMEWORK修改DRF的配置
# DRF框架所有的全局配置都放在REST_FRAMEWORK字典中

REST_FRAMEWORK = {
    'DEFAULT_RENDERER_CLASSES': [
        # 按列表中的元素顺序排优先级
        'rest_framework.renderers.JSONRenderer',
        'rest_framework.renderers.BrowsableAPIRenderer',
    ],
}
```

 比如浏览器访问时，会以html页面展示

![image-20210701000422874](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210701000422874.png)

如果把rest_framework.renderers.BrowsableAPIRenderer注释掉，则浏览器访问会出现

![image-20210701000534158](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210701000534158.png)

4）Response参数

```python
Response(data, status=None, template_name=None, headers=None, content_type=None)
```

- data

   序列化处理后的数据

   一般为serializer.data（python基本数据类型、字典、嵌套字典的列表）

- status

   状态码，默认为200

- template_name

   模板名称，使用HTMLRenderer渲染时需指明

- headers

   请求头

- content_type

   响应头中的content_type

   通常该参数无需设置，会自动根据前端所需类型数据来设置参数