## 特性

1）GenericAPIView继承了APIView，实现了过滤、排序、分页的功能

2）增加了对于列表视图和详情视图可能用到的通用支持方法， 每个具体通用视图都是一个GenericAPIView搭配一个或多个Mixin扩展类

3）支持的属性

| 属性             | 作用                                                         |
| ---------------- | ------------------------------------------------------------ |
| queryset         | 指定列表视图的查询集                                         |
| serializer_class | 指定视图使用的序列化器                                       |
| pagination_class | 指定分页引擎                                                 |
| filter_backends  | 指定过滤引擎                                                 |
| lookup_field     | 查询单一数据库对象时使用的条件字段，默认为'pk'，可以指定为'id' |
| lookup_url_kwarg | 查询单一数据时URL中的参数关键字名称，默认与look_field相同    |



 4）提供的方法

| 方法                 | 作用                                               |
| -------------------- | -------------------------------------------------- |
| get_object           | 返回模型类数据对象（单条详情数据）                 |
| get_queryset(qs)     | 返回查询集                                         |
| get_serializer       | 返回序列化器对象                                   |
| get_serializer_class | 返回序列化器类，默认返回serializer_class，可以重写 |
| filter_queryset      | 对查询集进行过滤                                   |
| paginate_queryset    | 对查询集进行分页                                   |

## 使用GenericAPIView类

get_object()：获取url中的id，返回模型类对象

get_serializer()：返回序列化器类

```python
from rest_framework.views import APIView
from rest_framework import filters
from rest_framework.generics import GenericAPIView
from rest_framework import status
from rest_framework.response import Response

from projects.serializer import ProjectModelSerializer, ProjectModelSerializer
from projects.models import Projects

class ProjectsList(GenericAPIView):

    queryset = Projects.objects.all()
    serializer_class = ProjectModelSerializer


    def get(self, request):
        #1.从数据库中获取所有的项目信息
        queryset = Projects.objects.all()
        #如果返回的是列表数据(多条数据)时，需要添加many=True这个参数
        serializer = self.get_serializer(instance=queryset, many=True)
        return Response(serializer.data, status=status.HTTP_200_OK)


    def post(self, request):

        serializer = self.get_serializer(data=request.data)
        try:
            serializer.is_valid(raise_exception=True)
        except Exception as e:
            return Response(serializer.errors)
        serializer.save()
        return Response(serializer.data, status=status.HTTP_200_OK)


#1.需要继承GenericAPIView基类
class ProjectsDetail(GenericAPIView):
    #2.必须指定queryset和serializer_class
    #queryset用于指定需要使用的查询集
    queryset = Projects.objects.all()
    #serializer_class指定需要使用到的序列化器类
    serializer_class = ProjectModelSerializer

    #使用lookup_field属性，可以修改组件路由名称
    #lookup_field = id


    #使用GenericAPIView后不需要定义get_object方法，这个类本身有定义
    # def get_object(self, pk):
    #     try:
    #         return Projects.objects.get(id=pk)
    #     except Projects.DoesNotExist:
    #         raise Http404


    def get(self, request, pk):
        #project = self.get_object(pk)
        #3.无需自定义get_object方法
        #使用get_object方法，返回详情视图所需的模型类对象
        project = self.get_object()

        #serializer = ProjectModelSerializer(instance=project)
        #使用get_serializer获取序列化器类
        serializer = self.get_serializer(instance=project)
        #如果前端请求头中未指定Accept，那么默认返回Json格式的数据
        return Response(serializer.data, status=status.HTTP_200_OK)


    #全部更新，patch为部分更新
    def put(self, request, pk):
        project = self.get_object()
        serializer = self.get_serializer(instance=project, data=request.data)

        try:
            serializer.is_valid(raise_exception=True)
        except Exception as e:
            return (serializer.errors)
        serializer.save()
        return Response(serializer.data, status=status.HTTP_201_CREATED)



    def detele(self, request, pk):
        project = self.get_object()
        project.delete()
        return Response(None, status=status.HTTP_204_NO_CONTENT)
```



## 模型序列化器的说明

```python
min_length=5,class ProjectModelSerializer(serializers.ModelSerializer):

    def validate_name(self, value):
        if not value.endswith('项目'):
            raise serializers.ValidationError('项目名称必须以"项目"结尾')
        # 校验成功之后，一定要返回value
        return value

    def validate(self, attrs):
        '''
        多字段联合检验
        :param attrs:
        :return:
        '''
        if 'Jeff' not in attrs['tester'] and 'Jeff' not in attrs['leader']:
            raise serializers.ValidationError('Jeff必须是项目负责人或者在项目测试人员')
        return attrs


    name = serializers.CharField(label='项目名称', max_length=200, min_length=5,
                                 help_text='项目名称111',
                                 validators=[UniqueValidator(queryset=Projects.objects.all(), message='项目名不能重复'),
                                             is_unique_project_name])

    class Meta:
        #1.指定参考哪一个模型类来创建
        model = Projects
        #2.指定为模型类的哪些字段来生成序列化器
        #fields = '__all__'
        fields = ('id', 'name', 'leader', 'tester', 'programer')
        #exclude = ('publish_app', 'desc')
        read_only_fields = ('id', )
        extra_kwargs = {
            'leader': {
                'write_only': True,
                'error_messages': {
                    'max_length': '长度不能超过50个字节'
                }
            }
        }

```

1.校验name时，validators中从左到右校验，如果不满足，则不会校验单字段和多字段联合校验

![image-20210701003210592](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210701003210592.png)

![image-20210701003322221](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210701003322221.png)

2.如果validators校验通过，则校验单字段，单字段不通过，不会校验多字段联合校验

![image-20210701003527440](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210701003527440.png)

3.如果validators和单字段校验通过，会进行多字段联合校验

![image-20210701003638195](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210701003638195.png)

2.未指定read_only和write_only的字段默认既能read，又能write。像下面这种，反序列化输入(write)的字段有name、leader、tester、programer，序列化输出(read)的字段有：id、name、tester、programer

```python
  fields = ('id', 'name', 'leader', 'tester', 'programer')
        #exclude = ('publish_app', 'desc')
        read_only_fields = ('id', )
        extra_kwargs = {
            'leader': {
                'write_only': True,
                'error_messages': {
                    'max_length': '长度不能超过50个字节'
                }
            }
        }
```



## 测试请求

### 获取项目列表

![image-20210701004119039](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210701004119039.png)



### 新建项目

![image-20210701004325676](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210701004325676.png)



### 获取id为1的项目详情

![image-20210701005717362](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210701005717362.png)



### 更新id为1的项目

![image-20210701005906470](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210701005906470.png)



## 排序

### 单个视图指定排序

```python
from rest_framework.filters import OrderingFilter

class ProjectsList(GenericAPIView):

    #1.指定查询集
    queryset = Projects.objects.all()
    #2.指定序列化器类
    serializer_class = ProjectModelSerializer

    #3.在视图类中指定过滤引擎，这里使用的是排序过滤
    filter_backends = [OrderingFilter]
    #4.指定需要排序的字段
    ordering_fields = ['name', 'leader']


    def get(self, request):
        #5.使用get_queryset获取查询集
        project_qs = self.get_queryset()
        #6.使用filter_queryset方法过滤查询集
        project_qs = self.filter_queryset(project_qs)
        #如果返回的是列表数据(多条数据)时，需要添加many=True这个参数
        serializer = self.get_serializer(instance=project_qs, many=True)
        return Response(serializer.data, status=status.HTTP_200_OK)
```

在命令行运行

```python
#根据name升序排列
http -v :8000/index/?ordering=name
    
#根据name倒序排列
http -v :8000/index/?ordering=-name
```



### 全局指定排序

settings.py

```python
# 在全局settings.py中指定排序引擎
# 配置DRF
REST_FRAMEWORK = {
    'DEFAULT_FILTER_BACKENDS': [
        'rest_framework.filters.OrderingFilter'
    ],
}
```

views.py

```python
    #3.在视图类中指定过滤引擎，这里使用的是排序过滤。在全局指定后不需要在这里指定filter_backends
    #filter_backends = [filters.OrderingFilter]
    #4.指定需要排序的字段
    ordering_fields = ['name', 'leader']
```



## 过滤

### 单个视图指定过滤

安装过滤模块

```python
pip install django-filter
```

settings.py

```python
INSTALLED_APPS = [
    'django_filters'
]
```

views.py

```python
from django_filters.rest_framework.backends import DjangoFilterBackend

class ProjectsList(GenericAPIView):
	#1.指定查询集
    queryset = Projects.objects.all()
    #2.指定序列化器类
    serializer_class = ProjectModelSerializer
    #3.在类视图中指定过滤引擎
    filter_backends = [DjangoFilterBackend]
    #4.指定需要过滤的字段
    filterset_fields = ['name', 'leader', 'tester']
```

在命令行运行

```python
#查询name为beck的项目
http -v :8000/index/ leader==beck
```



### 全局指定过滤

settings.py

```python
# 在全局settings.py中指定过滤引擎
# 添加应用
INSTALLED_APPS = [
    'django_filters',
]
# 配置DRF
REST_FRAMEWORK = {
    'DEFAULT_FILTER_BACKENDS': [
   'django_filters.rest_framework.backends.DjangoFilterBackend'
    ],
}
```

views.py

```python
	#5.在类视图中指定过滤引擎
    #filter_backends = [DjangoFilterBackend]

    #6.指定需要过滤的字段
    filterset_fields = ['name', 'leader', 'tester']
```



## 分页

### 全局指定分页-使用默认分页引擎

指定分页引擎：pagination_class

settings.py

```python
#在全局settings.py中指定分页引擎
#配置DRF
REST_FRAMEWORK = {
    # 使用默认的分页引擎
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    # 指定每页展示数据的条数
    'PAGE_SIZE': 3,
}
```

先过滤后分页

get_queryset()：获取查询集

filter_queryset()：对查询集进行过滤，并返回一个查询集

paginate_queryset()：对查询集进行分页

```python
class ProjectsList(GenericAPIView):

    #1.指定查询集
    queryset = Projects.objects.all()
    #2.指定序列化器类
    serializer_class = ProjectModelSerializer

    #3.在视图类中指定过滤引擎，这里使用的是排序过滤。在全局指定后不需要在这里指定filter_backends
    #filter_backends = [filters.OrderingFilter]
    #4.指定需要排序的字段
    ordering_fields = ['name', 'leader']

    #5.在类视图中指定过滤引擎
    #filter_backends = [DjangoFilterBackend]

    #6.指定需要过滤的字段
    filter_fields = ['name', 'leader', 'tester']


    def get(self, request):
        #5.使用get_queryset获取查询集
        project_qs = self.get_queryset()
        #6.使用filter_queryset方法过滤查询集
        project_qs = self.filter_queryset(project_qs)

        #使用paginate_queryset来进行分页，然后返回分页之后的查询集
        page = self.paginate_queryset(project_qs)

        #存在settings中不使用引擎的情况
        if page is not None:
            serializer = self.get_serializer(instance=page, many=True)
            #可以使用get_paginated_response方法返回
            return self.get_paginated_response(serializer.data)
        serializer = self.get_serializer(instance=project_qs, many=True)
        return Response(serializer.data)
```

在命令行运行，查询任意一个项目列表

```python
http -v :8000/index/ leader==beck
```

![image-20210702233425889](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210702233425889.png)

指定page，获取第二页的数据

```python
http -v :8000/index/ leader==beck page==2
```

![image-20210702233628627](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210702233628627.png)

 

### 全局指定分页-使用自定义分页引擎

创建一个新目录utils，用于存放自定义模块，继承PageNumberPagination，重新定义分页引擎

当自定义分页引擎与settings.py中分页引擎同时存在时，自定义的优先级更高

utils/pagination.py

```python
from rest_framework.pagination import PageNumberPagination

class PageNumberPaginationManual(PageNumberPagination):
    page_query_param = 'p'
    #默认情况下，每一页显示的条数为2
    page_size = 2
    # 设置每页显示的数据条数的查询字符串key名称，不设置则无法指定每页显示的数据量
    page_size_query_param = 's'
    #指定前端能分页的最大的page_size
    max_page_size = 50
```

settings.py

```python

REST_FRAMEWORK = {
    #使用自定义的分页引擎
    'DEFAULT_PAGINATION_CLASS': 'utils.pagination.PageNumberPaginationManual',
}
```

获取第5页的数据，每页展示2条

```python
http -v :8000/index/?p=5
```

![image-20210703132241274](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210703132241274.png)

获取第5页的数据，每页展示3条

```python
http://localhost:8000/index/?p=4&s=3
```

![image-20210703132401361](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210703132401361.png)

### 单个视图指定分页-使用自定义分页引擎

views.py的ProjectsList类的get方法

```python
from utils.pagination import PageNumberPaginationManual

 	#7.在某个视图中指定分页类
    pageination_class = PageNumberPaginationManual
```

