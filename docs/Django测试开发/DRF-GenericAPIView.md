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

| 方法              | 作用                               |
| ----------------- | ---------------------------------- |
| get_object        | 返回模型类数据对象（单条详情数据） |
| get_queryset(qs)  | 返回查询集                         |
| get_serializer    | 返回序列化器对象                   |
| filter_queryset   | 对查询集进行过滤                   |
| paginate_queryset | 对查询集进行分页                   |

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