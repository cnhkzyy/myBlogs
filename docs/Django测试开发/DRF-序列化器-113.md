## 简介

+ 在Django框架基础之上，进行二次开发
+ 用于构建Restful API
+ 简称为DRF框架或REST framework框架



## 特性

- 提供了强大的Serializer序列化器，可以高效地进行序列化和反序列化操作
- 提供了极为丰富的类视图、Mixin扩展类、ViewSet视图集
- 提供了直观的Web API界面
- 多种身份认证和权限认证
- 强大的排序、过滤、分页、搜索、限流等功能
- 可扩展性，插件丰富

  

## 安装

```python
pip install djangorestframework
pip install markdown
```

## 配置

settings.py

```python
INSTALLED_APPS = [
    'rest_framework'
]

```

## 序列化


### 定义序列化器projects/serializer.py

```python
from rest_framework import serializers

class ProjectSerializer(serializers.Serializer):
    '''
    创建项目序列化器类
    '''
    #1.序列化器中定义的类属性往往与模型类字段一一对应
    #2.label选项相当于verbose_name，help_text在后台显示，是一个别名
    #3.定义的序列化器字段，默认既可以进行序列化输出，也可以进行反序列化输入
    #4.read_only=True，指定该字段只能进行序列化输出
    #5.write_only=True,指定该字段只进行反序列化输入，但不进行序列化输出
    #6.需要输出哪些字段，那么在序列化器中就定义哪些字段。不需要输入和输出的可以直接不定义
    id = serializers.IntegerField(label='ID', read_only=True)
    name = serializers.CharField(label='项目名称', max_length=200, help_text='项目名称111', write_only=True)
    leader = serializers.CharField(label='负责人', max_length=50, help_text='负责人')
    tester = serializers.CharField(label='测试人员', max_length=50, help_text='测试人员')
    programer = serializers.CharField(label='开发人员', max_length=50, help_text='开发人员')
    publish_app = serializers.CharField(label='发布应用', max_length=50, help_text='发布应用')
    #allow_null相当于模型类中的null，allow_blank相当于模型类中的blank
    desc = serializers.CharField(label='简要描述', allow_null=True, allow_blank=True, help_text='简要描述', default='')
```

### ProjectsDetail.get()  序列化返回一条数据

```python
from projects.serializer import ProjectSerializer

def get(self, request, pk):
    #1.校验前端传递的pk(项目id)值，类型是否正确(正整数)，在数据中是否存在
    #省略
    #2.获取指定pk值的项目
    project = self.get_object(pk)

    #序列化
    #1.通过模型类对象(或者查询集)，传给instance，可进行序列化操作
    #2.通过序列化器ProjectSerializer对象的data属性，就可以获取转化之后的字典
    serializer = ProjectSerializer(instance=project)

    return JsonResponse(serializer.data)
```

### ProjectsList.get()  序列化多条数据

```python
def get(self, request):
   #1.从数据库中获取所有的项目信息
   project_qs = Projects.objects.all()

   #序列化
   #如果返回的是列表数据(多条数据)时，需要添加many=True这个参数
   serializer = ProjectSerializer(instance=project_qs, many=True)
   return JsonResponse(serializer.data, safe=False)
```

## 反序列化

ProjectsList.post()  

```python
def post(self, request):
        '''
        新增项目
        :param request:
        :return:
        '''
        #1.从前端获取json格式的数据，转化为Python中的类型
        #为了严谨性，这里需要做各种复杂的校验
        #比如：是否为json，传递的项目数据是否符合要求，有些必传参数是否携带等
        #反序列化
        json_data = request.body.decode('utf-8')
        python_data = json.loads(json_data, encoding='utf-8')

        serializer = ProjectSerializer(data=python_data)
        #校验前端输入的数据
        #1.调用序列化器对象的is_valid()方法，开始校验前端参数
        #2.如果校验成功，则返回True，校验失败返回False
        #3.raise_exception=True，那么校验失败之后，会抛出异常
        #4.当调用is_valid方法之后，才可以调用errors属性，获取校验的错误提示(字典)
        try:
            serializer.is_valid(raise_exception=True)
        except Exception as e:
            return JsonResponse(serializer.errors)
        #校验成功之后的数据，可以使用validated_data属性来获取
        #2.向数据库中新增项目
        project = Projects.objects.create(**serializer.validated_data)

        #3.将模型类对象转化为字典返回
        #序列化

        serializer = ProjectSerializer(project)

        return JsonResponse(serializer.data, status=201)
```

## 请求数据校验

### 方法一

ProjectsDetail.get()  ProjectsDetail.put()  

```python
class ProjectsDetail(View):


    def get_object(self, pk):
        try:
            return Projects.objects.get(id=pk)
        except Projects.DoesNotExist:
            raise Http404


    def get(self, request, pk):
        #1.校验前端传递的pk(项目id)值，类型是否正确(正整数)，在数据中是否存在
        #省略
        #2.获取指定pk值的项目
        ----------------------方法一--------------------------
        project = self.get_object(pk)
        -----------------------------------------------------

        #序列化
        #1.通过模型类对象(或者查询集)，传给instance，可进行序列化操作
        #2.通过序列化器ProjectSerializer对象的data属性，就可以获取转化之后的字典
        serializer = ProjectSerializer(instance=project)

        return JsonResponse(serializer.data)


    #全部更新，patch为部分更新
    def put(self, request, pk):
        #1.校验前端传递的pk(项目id)值，类型是否正确(正整数)，在数据中是否存在
        #2.获取指定id为pk值的项目
        project = self.get_object(pk)

        #3.从前端获取json格式的数据
        #反序列化
        json_data = request.body.decode('utf-8')
        python_data = json.loads(json_data, encoding='utf-8')
        serializer = ProjectSerializer(data=python_data)

        try:
            serializer.is_valid(raise_exception=True)
        except Exception as e:
            return JsonResponse(serializer.errors)

        #4.更新项目
        project.name = serializer.validated_data['name']
        project.leader = serializer.validated_data['leader']
        project.tester = serializer.validated_data['tester']
        project.programer = serializer.validated_data['programer']
        project.publish_app = serializer.validated_data['publish_app']
        project.desc = serializer.validated_data['desc']
        project.save()

        #5.将模型类对象转化为字典
        #序列化
        serializer = ProjectSerializer(instance=project)
        return JsonResponse(serializer.data, status=201)
```

### 方法二

ProjectsDetail.put()  

```python
 #全部更新，patch为部分更新
    def put(self, request, pk):
        #1.校验前端传递的pk(项目id)值，类型是否正确(正整数)，在数据中是否存在
        #2.获取指定id为pk值的项目
        project = self.get_object(pk)

        #3.从前端获取json格式的数据
        #反序列化
        json_data = request.body.decode('utf-8')
        python_data = json.loads(json_data, encoding='utf-8')
        serializer = ProjectSerializer(data=python_data)

        ----------------------方法二--------------------------
        try:
            serializer.is_valid(raise_exception=True)
        except Exception as e:
            return JsonResponse(serializer.errors)
        ----------------------------------------------------- 

        #4.更新项目
        -------------------手动实现更新，待优化------------------
        project.name = serializer.validated_data['name']
        project.leader = serializer.validated_data['leader']
        project.tester = serializer.validated_data['tester']
        project.programer = serializer.validated_data['programer']
        project.publish_app = serializer.validated_data['publish_app']
        project.desc = serializer.validated_data['desc']
        project.save()
        -------------------------------------------------------

        #5.将模型类对象转化为字典
        #序列化
        serializer = ProjectSerializer(instance=project)
        return JsonResponse(serializer.data, status=201)
```



### 字段参数设置

#### 选项参数

| 参数名称   | 作用     |
| ---------- | -------- |
| max_length | 最大长度 |
|  min_length	| 最小长度 |
| allow_blank	| 是否允许为空（字段不传）|
| trim_whitespace | 是否截断空白字符（前后空格，不包括中间空格） |
| max_value	| 最大值 |
| min_value	 | 最小值 |

#### 通用参数

| 参数名称 | 说明 |
| -------- | ---- |
|   required       |   默认为None，指定前端必须传该字段，如果为False，则参数可以为空（与read_only不可同时设置）   |
| read_only | 仅用于序列化输出，默认为False，若为True，则该字段在反序列化输入时可不用传，若传入字段也不做校验 |
| write_only | 仅用于反序列化输入，默认为False，若为True，则该字段必传，但不会输出（与read_only不可同时设置）|
| default | 前端不传参，则使用默认值 |
| allow_null	| 指定传参时参数可以为空值，默认为False |
| validators | 字段指定校验器 |
| error_messages | 设置错误信息，key为校验的参数名，value为校验失败后的返回信息，如：error_messages={"required": "该字段必填"} |
| label | 与模型类中的verbose_name作用相同，HTML展示API页面时显示的字段名称 |
| help_text | 与模型类中的help_text作用相同，HTML展示API页面时显示的字段帮助提示信息 |






