[toc]





# DRF

## 创建接口的任务

### 序列化和反序列化

- 校验用户数据
- 将请求的数据（如json格式）转换为模型类对象

```
反序列化
1. 将其他格式(json、xml等)转换为程序中的数据类型
2. 将json格式的字符串转换为django中的模型类的对象
```

- 操作数据库
- 将模型类对象转换为响应的数据（如json格式）

```
序列化
1.将程序中的数据类型转换为其他格式（json、xml等）
2.例如将django中模型类对象转换为json字符串
```



### 数据增删改查流程



```增```

+ 校验请求参数-->反序列化-->保存数据-->将保存的对象序列化并返回



```删```

+ 判断要删除的数据是否存在-->执行数据库删除

  

```改```

+ 判读要修改的数据是否存在-->校验请求参数-->反序列化-->保存数据-->将保存的对象序列化输出

  

```查```

+ 查询数据库-->将数据序列化并返回



## 序列化器

### 简介

+ 在Django框架基础之上，进行二次开发
+ 用于构建Restful API
+ 简称为DRF框架或REST framework框架



### 特性

- 提供了强大的Serializer序列化器，可以高效地进行序列化和反序列化操作

- 提供了极为丰富的类视图、Mixin扩展类、ViewSet视图集

- 提供了直观的Web API界面

- 多种身份认证和权限认证

- 强大的排序、过滤、分页、搜索、限流等功能

- 可扩展性，插件丰富

  

### 安装

```python
pip install djangorestframework
pip install markdown
```

### 配置

settings.py

```python
INSTALLED_APPS = [
    'rest_framework'
]

```

### 序列化


#### 定义序列化器projects/serializer.py

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

#### ProjectsDetail.get()  序列化返回一条数据

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

#### ProjectsList.get()  序列化多条数据

```python
def get(self, request):
   #1.从数据库中获取所有的项目信息
   project_qs = Projects.objects.all()

   #序列化
   #如果返回的是列表数据(多条数据)时，需要添加many=True这个参数
   serializer = ProjectSerializer(instance=project_qs, many=True)
   return JsonResponse(serializer.data, safe=False)
```

### 反序列化

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

### 请求数据校验

#### 方法一

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

#### 方法二

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



#### 字段参数设置

##### 选项参数

| 参数名称        | 作用                                         |
| --------------- | -------------------------------------------- |
| max_length      | 最大长度                                     |
| min_length      | 最小长度                                     |
| allow_blank     | 是否允许为空（字段不传）                     |
| trim_whitespace | 是否截断空白字符（前后空格，不包括中间空格） |
| max_value       | 最大值                                       |
| min_value       | 最小值                                       |

##### 通用参数

| 参数名称       | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| required       | 默认为None，指定前端必须传该字段，如果为False，则参数可以为空（与read_only不可同时设置） |
| read_only      | 仅用于序列化输出，默认为False，若为True，则该字段在反序列化输入时可不用传，若传入字段也不做校验 |
| write_only     | 仅用于反序列化输入，默认为False，若为True，则该字段必传，但不会输出（与read_only不可同时设置） |
| default        | 前端不传参，则使用默认值                                     |
| allow_null     | 指定传参时参数可以为空值，默认为False                        |
| validators     | 字段指定校验器                                               |
| error_messages | 设置错误信息，key为校验的参数名，value为校验失败后的返回信息，如：error_messages={"required": "该字段必填"} |
| label          | 与模型类中的verbose_name作用相同，HTML展示API页面时显示的字段名称 |
| help_text      | 与模型类中的help_text作用相同，HTML展示API页面时显示的字段帮助提示信息 |


## 字段校验

### 项目名不能重复

#### 修改序列化器projects/serializer.py

```python
from rest_framework import serializers
from rest_framework.validators import UniqueValidator
from projects.models import Projects


#1.继承Serializer类或者子类
class ProjectSerializer(serializers.Serializer):
    '''
    创建项目序列化器类
    '''
    #1.序列化器中定义的类属性往往与模型类字段一一对应
    #2.label选项相当于verbose_name，help_text在后台显示，是一个别名
    #3.#定义的序列化器字段，默认既可以进行序列化输出，也可以进行反序列化输入
    #4.read_only=True，指定该字段只能进行序列化输出
    #5.write_only=True,指定该字段只进行反序列化输入，但不进行序列化输出
    #6.需要输出哪些字段，那么在序列化器中就定义哪些字段。不需要输入和输出的可以直接不定义
    id = serializers.IntegerField(label='ID', read_only=True)
    name = serializers.CharField(label='项目名称', max_length=200,
                                 help_text='项目名称111', write_only=True, validators = [UniqueValidator(queryset=Projects.objects.all(), message='项目名不能重复')])
    leader = serializers.CharField(label='负责人', max_length=50, help_text='负责人')
    tester = serializers.CharField(label='测试人员', max_length=50, help_text='测试人员')
    programer = serializers.CharField(label='开发人员', max_length=50, help_text='开发人员')
    publish_app = serializers.CharField(label='发布应用', max_length=50, help_text='发布应用')
    #allow_null相当于模型类中的null，allow_blank相当于模型类中的blank
    desc = serializers.CharField(label='简要描述', allow_null=True, allow_blank=True, help_text='简要描述', default='')
```

name已重复，在vscode中使用httpie发送请求：```http -v :8000/index/ < create_project.json```, 失败的提示是定义的message

![image-20210628212235267](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210628212235267.png)

### 自定义校验器项目名必须包含项目

```python
#创建自定义校验器
#第一个参数为字段的值
def is_unique_project_name(name):
    #要求项目的名字必须包含项目
    if '项目' not in name:
        raise serializers.ValidationError('项目名称中必须包含"项目"')
        
        
#1.继承Serializer类或者子类
class ProjectSerializer(serializers.Serializer):
    '''
    创建项目序列化器类
    '''
    #1.序列化器中定义的类属性往往与模型类字段一一对应
    #2.label选项相当于verbose_name，help_text在后台显示，是一个别名
    #3.#定义的序列化器字段，默认既可以进行序列化输出，也可以进行反序列化输入
    #4.read_only=True，指定该字段只能进行序列化输出
    #5.write_only=True,指定该字段只进行反序列化输入，但不进行序列化输出
    #6.需要输出哪些字段，那么在序列化器中就定义哪些字段。不需要输入和输出的可以直接不定义
    id = serializers.IntegerField(label='ID', read_only=True)
    name = serializers.CharField(label='项目名称', max_length=200,
                                 help_text='项目名称111', write_only=True,
                                 validators=[UniqueValidator(queryset=Projects.objects.all(), message='项目名不能重复'), is_unique_project_name])
    leader = serializers.CharField(label='负责人', max_length=50, help_text='负责人')
    tester = serializers.CharField(label='测试人员', max_length=50, help_text='测试人员')
    programer = serializers.CharField(label='开发人员', max_length=50, help_text='开发人员')
    publish_app = serializers.CharField(label='发布应用', max_length=50, help_text='发布应用')
    #allow_null相当于模型类中的null，allow_blank相当于模型类中的blank
    desc = serializers.CharField(label='简要描述', allow_null=True, allow_blank=True, help_text='简要描述
```

name不包含项目，在vscode中使用httpie发送请求：```http -v :8000/index/ < create_project.json```, 失败的提示为抛出的异常提示

![image-20210628213441609](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210628213441609.png)



### 单字段的校验

以validate_开头，能够自动识别，不需要添加到validators中

```python
#单字段的校验
    def validate_name(self, value):
        if not value.endswith('项目'):
            raise serializers.ValidationError('项目名称必须以"项目"结尾')
        #校验成功之后，一定要返回value
        return value
```

name部署以项目结尾

![image-20210628214935766](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210628214935766.png)



### 多字段的校验

```python
    def validate(self, attrs):
        '''
        多字段联合检验
        :param attrs:
        :return:
        '''
        if 'Jeff' not in attrs['tester'] and 'Jeff' not in attrs['leader']:
            raise serializers.ValidationError('Jeff必须是项目负责人或者在项目测试人员')
        return attrs
```

Jeff 没有在项目负责人或者在项目测试人员

![image-20210628220802148](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210628220802148.png)

### 校验顺序

```python
字段定义时的限制（包含validators列表条目从左到右进行校验）-->单字段的校验（validate_字段名）-->多字段联合校验（validate）
```

## 创建更新

### 创建

如果在创建序列化器对象的时候，只给data传参，那么调用save方法，实际调用的是序列化对象的create()方法

serializer.py中的ProjectSerializer下定义

```python
    def create(self, validated_data):
        return Projects.objects.create(**validated_data)
```

在views.py中的ProjectsList的post方法中使用

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

        #1.如果在创建序列化器对象的时候，只给data传参，那么调用save方法，实际调用的是序列化对象的create()方法
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
        #project = Projects.objects.create(**serializer.validated_data)
        serializer.save()

        #3.将模型类对象转化为字典返回
        #序列化
        return JsonResponse(serializer.data, status=201)
```



### 更新

在创建序列化器对象是，如果同时给instance和data传参，那么调用save()方法会自动调用序列化器对象的update()方法

serializer.py中的ProjectSerializer下定义

```python
    def update(self, instance, validated_data):
        instance.name = validated_data['name']
        instance.leader = validated_data['leader']
        instance.tester = validated_data['tester']
        instance.programer = validated_data['programer']
        instance.publish_app = validated_data['publish_app']
        instance.desc = validated_data['desc']
        instance.save()
        return instance
```

在views.py中的ProjectsDetail的put方法中使用

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
        serializer = ProjectSerializer(instance=project, data=python_data)

        try:
            serializer.is_valid(raise_exception=True)
        except Exception as e:
            return JsonResponse(serializer.errors)

        #4.更新项目
        # 在创建序列化器对象是，如果同时给instance和data传参，那么调用save()方法会自动调用序列化器对象的update()方法
        serializer.save()

        #5.将模型类对象转化为字典
        #序列化
        return JsonResponse(serializer.data, status=201)
```



## 模型序列化器

### 定义

1. 目的：为了简化序列化器类的定义

2. 功能：

   基于模型类自动生成一系列字段

   基于模型类自动为Serializer生成validators，如：unique_together

   包含默认的create()和update()的实现

3. ModelSerializer定义的字段若与模型类字段同名，会覆盖原有的模型类字段

4. 在serializers.py中定义一个模型序列化器

```python
class ProjectModelSerializer(serializers.ModelSerializer):

    class Meta:
        #1.指定参考哪一个模型类来创建
        model = Projects
        #2.指定为模型类的哪些字段来生成序列化器
        fields = '__all__'
```



### 指定全部字段校验

在终端中使用ipython实例化，发现会自动生成序列化器

![image-20210629000732119](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210629000732119.png)

### 指定部分字段校验

```python
        fields = ('id', 'name', 'leader', 'tester', 'programer')
```

重新在终端中导入

![image-20210629001202653](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210629001202653.png)



### 反向指定：排除指定的字段

```python
        exclude = ('publish_app', 'desc')
```

重新在终端中导入

![image-20210629222811881](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210629222811881.png)

注意模型序列化器生成的id属性始终为read_only=True



### 指定read_only = True

```python
        read_only_fields = ('leader', 'tester')
```

重新在终端中导入

![image-20210629223315801](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210629223315801.png)



### 自定义校验：覆盖原来的字段

如这里重新定义name，使其调用了自定义的校验方法

```python
class ProjectModelSerializer(serializers.ModelSerializer):

    name = serializers.CharField(label='项目名称', max_length=200,
                                 help_text='项目名称111', write_only=True,
                                 validators=[UniqueValidator(queryset=Projects.objects.all(), message='项目名不能重复'),
                                             is_unique_project_name])

    class Meta:
        #1.指定参考哪一个模型类来创建
        model = Projects
        #2.指定为模型类的哪些字段来生成序列化器
        #fields = '__all__'
        #fields = ('id', 'name', 'leader', 'tester', 'programer')
        exclude = ('publish_app', 'desc')
        read_only_fields = ('leader', 'tester')
```



### 自定义错误提示

先回顾下之前定义的ProjectSerializer类，如果要自定义比较友好的错误提示，可以使用error_messages，error_messages为字典形式，key为校验项，value为校验项报错时的提示信息

```python
 name = serializers.CharField(label='项目名称', max_length=200,
                                 help_text='项目名称111', write_only=True,
                                 validators=[UniqueValidator(queryset=Projects.objects.all(), message='项目名不能重复'),
                                             is_unique_project_name],
                                 error_messages={
                                     'max_length': '长度不能超过200个字节'
                                 }
                                 )
    leader = serializers.CharField(label='负责人', max_length=50, min_length=6, help_text='负责人',
                                   error_messages={ 'max_length': '长度不能超过50个字节',
                                                    'min_length': '长度不能少于6个字节'})
```

同样，如果在模型序列化器中自定义错误提示，可以使用extra_kwargs。比如要定义leader字段对应的错误提示

```python
    class Meta:
        #1.指定参考哪一个模型类来创建
        model = Projects
        #2.指定为模型类的哪些字段来生成序列化器
        #fields = '__all__'
        #fields = ('id', 'name', 'leader', 'tester', 'programer')
        exclude = ('publish_app', 'desc')
        #read_only_fields = ('leader', 'tester')
        extra_kwargs = {
            'leader': {
                'error_messages': {
                    'max_length': '长度不能超过50个字节',
                    'min_length': '长度不能少于6个字节'}
            }
        }
```



### 自定义write_only = True

同样是在extra_kwargs中定义write_only  = True

```python
    class Meta:
        #1.指定参考哪一个模型类来创建
        model = Projects
        #2.指定为模型类的哪些字段来生成序列化器
        #fields = '__all__'
        #fields = ('id', 'name', 'leader', 'tester', 'programer')
        exclude = ('publish_app', 'desc')
        #read_only_fields = ('leader', 'tester')
        extra_kwargs = {
            'leader': {
                'write_only': True,
                'error_messages': {
                    'max_length': '长度不能超过50个字节',
                    'min_length': '长度不能少于6个字节'}
            }
        }
```



### 单字段和多字段校验

将单字段和多字段校验方法放在ProjectModelSerializer类下面，Meta类外面

```python
class ProjectModelSerializer(serializers.ModelSerializer):

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


    name = serializers.CharField(label='项目名称', max_length=200,
                                 help_text='项目名称111', write_only=True,
                                 validators=[UniqueValidator(queryset=Projects.objects.all(), message='项目名不能重复'),
                                             is_unique_project_name])

    class Meta:
        ...
```



### create和update操作

在模型化序列化器中，create()和update()方法会自动生成，如果需要覆盖，重新定义即可


## APIView

### 特性

 继承了Django中的View，进行了拓展，是可浏览的API页面

 具备认证、授权、限流、不同请求数据的解析



### APIView与View的不同

1）传入到视图方法中的是Request对象，而不是Django中的HttpRequest对象

 2）视图方法可以返回Response对象，会为响应数据处理（render）为符合前端要求的格式

 3）任何APIException异常都会被捕获到，并且处理成合适的响应信息

 4）在进行dispatch()分发前，会对请求进行身份认证、权限检查、流量控制



### 常用类属性

 authentication_classes 列表或元组，身份认证类

 permission_classes 列表或元组，权限检查类

 throttle_classes 列表或元组，流量控制类



### Request

1）对Django中的HttpRequest进行了拓展

接收到请求后会根据请求头中的Content-Type，自动进行解析，解析为字典形式保存到Request对象

无论前端发送哪种格式的数据，都可以以统一的方式读取

 2）.data —— 获取json格式的参数、form表单的参数、files（类似于.body 、.POST、.FILES ）

利用了REST framework的parsers解析器，不仅支持表单类型数据，也支持JSON数据

可以对POST、PUT、PATCH的请求体参数进行解析

Render类 Parse解析类

 3）.query_params —— 获取查询字符串参数（类似于.GET ）

 4）支持Django HttpRequest中所有的对象和方法



### Response

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

## GenericAPIView

### 特性

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

### 使用GenericAPIView类

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



### 模型序列化器的说明

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



### 示例

#### 获取项目列表

![image-20210701004119039](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210701004119039.png)



#### 新建项目

![image-20210701004325676](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210701004325676.png)



#### 获取id为1的项目详情

![image-20210701005717362](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210701005717362.png)



#### 更新id为1的项目

![image-20210701005906470](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210701005906470.png)



### 排序

#### 单个视图指定排序

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



#### 全局指定排序

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



### 过滤

#### 单个视图指定过滤

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



#### 全局指定过滤

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



### 分页

#### 全局指定分页-使用默认分页引擎

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

 

#### 全局指定分页-使用自定义分页引擎

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

#### 单个视图指定分页-使用自定义分页引擎

views.py的ProjectsList类的get方法

```python
from utils.pagination import PageNumberPaginationManual

 	#7.在某个视图中指定分页类
    pageination_class = PageNumberPaginationManual
```

## Mixin

### 拓展类

+ RetrieveModelMixin

  提供 retrieve(request, *args, **kwargs) 方法，获取详情数据(一条数据)

  获取成功，返回 200 OK，否则返回 404 Not Found

- ListModelMixin

  提供 list(request, *args, **kwargs) 方法，获取列表数据(获取多条数据)

  获取成功，返回 200 OK，否则返回 404 Not Found

- CreateModelMixin

  提供 create(request, *args, **kwargs) 方法，创建数据

  获取成功，返回 201 OK；请求参数有误，返回 400 Bad Request

- UpdateModelMixin

  提供 update(request, *args, **kwargs) 方法，全更新

  提供 partial_update(request, *args, **kwargs) 方法，部分更新，支持PATCH方法

  更新成功(更新一条记录)，返回 200 OK

  请求参数有误，返回 400 Bad Request；不存在，返回 404 Not Found

- DestoryModelMixin

  提供 destory(request, *args, **kwargs) 方法，删除数据

  删除成功，返回 204 No Content，不存在，返回 404 Not Found

### 示例

```python
from rest_framework import mixins

#1.首先继承mixins，然后再继承GenericAPIView
class ProjectsList(mixins.ListModelMixin,
                   mixins.CreateModelMixin,
                   GenericAPIView):


    queryset = Projects.objects.all()

    serializer_class = ProjectModelSerializer

    ordering_fields = ['name', 'leader']

    filter_fields = ['name', 'leader', 'tester']

    #7.在某个视图中指定分页类
    pageination_class = PageNumberPaginationManual


    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)


    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)


class ProjectsDetail(mixins.RetrieveModelMixin,
                     mixins.UpdateModelMixin,
                     mixins.DestroyModelMixin,
                     GenericAPIView):
    queryset = Projects.objects.all()
    serializer_class = ProjectModelSerializer



    def get(self, request, *args, **kwargs):
        self.retrieve(request, *args, **kwargs)


    def put(self, request, *args, **kwargs):
        self.update(request, *args, **kwargs)


    def detele(self, request, *args, **kwargs):
        self.destroy(request, *args, **kwargs)
```

## Concreate_Generic_Views

### 拓展类

- RetrieveApIView

  提供：get方法

  继承：RetrieveModelMixin、GenericAPIView

- UpdateApIView

  提供：put、patch方法

  继承：UpdateModelMixin、GenericAPIView

- DestoryApIView

  提供：delete方法

  继承：DestoryModelMixin、GenericAPIView

- ListAPIView

  提供：get方法

  继承：ListModelMixin、GenericAPIView

- CreateAPIView

  提供：post方法

  继承：CreateModelMixin、GenericAPIView

- ListCreateApIView

  提供：post、get方法

  继承：ListModelMixin、CreateModelMixin、GenericAPIView

- RetrieveUpdateApIView

  提供：get、put、patch方法

  继承：ListModelMixin、CreateModelMixin、GenericAPIView

- ListCreateApIView

  提供：ge、createt方法

  继承：ListModelMixin、CreateModelMixin、GenericAPIView

- RetrieveUpdateApIView

  提供get、put、patch方法

  继承：RetrieveModelMixin、UpdateModelMixin、GenericAPIView

- RetrieveDestoryApIView

  提供get、delete方法

  继承：RetrieveModelMixin、DestoryModelMixin、GenericAPIView

- RetrieveUpdateDestoryApIView

  提供get、put、patch、delete方法

  继承：RetrieveModelMixin、UpdateModelMixin、DestoryModelMixin、GenericAPIView

### 示例

views.py

```python
from rest_framework import generics

class ProjectsList(generics.ListCreateAPIView):
    queryset = Projects.objects.all()
    serializer_class = ProjectModelSerializer

    ordering_fields = ['name', 'leader']
    filter_fields = ['name', 'leader', 'tester']



class ProjectsDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Projects.objects.all()
    serializer_class = ProjectModelSerializer
```



### 痛点

列表视图和详情视图无法合并

两个类视图中，有相同的get方法，会冲突

两个类视图所对应的url地址不一致



## ViewSet



### 介绍

继承ViewSetMixin和Views.APIView

ViewSet不再支持get、post、put、delete等请求方法，只支持action动作

未提供get_object()、get_serializer()、queryset、serializer_class等

所以需要继承GenericViewSet



### 支持的动作

| 请求方法 | 动作           | 描述                 |
| -------- | -------------- | -------------------- |
| GET      | retrieve       | 获取详情数据（单条） |
| GET      | list           | 获取列表数据（多条） |
| POST     | create         | 创建数据             |
| PUT      | update         | 更新数据             |
| PATCH    | partial_update | 部分更新             |
| DELTE    | destory        | 删除数据             |





## GenericViewSet

### 介绍

继承ViewSetMixin和generics.GenericAPIView

提供get_object()、get_serializer()、queryset、serializer_class等

在定义路由时，需将请求方法与action动作进行绑定

使用Mixins类简化程序



### ViewSet与GenericViewSet的区别

| ViewSet                                                      | GenericViewSet                                               |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 继承：(ViewSetMixin, Views.APIView)                          | 继承：(ViewSetMixin, generics.GenericAPIView)                |
| 未提供通用View的方法集：get_object()、get_serializer()、queryset、serializer_class等 | 包含基本的通用View的方法集：get_object()、get_serializer()、queryset、serializer_class等 |
| 支持action动作                                               | 在定义路由时，需将请求方法与action动作进行映射               |



### 示例

views.py

```python
from rest_framework import viewsets

#APIView
#GenericAPIView
#ViewSet不再支持get、post、put、delete等请求方法，只支持action动作
#但是ViewSet这个类中未提供get_object()、get_serializer()等方法
#所以需要继承GenericViewSet
class ProjectsViewSet(viewsets.GenericViewSet):
    queryset = Projects.objects.all()
    serializer_class = ProjectModelSerializer

    ordering_fields = ['name', 'leader']
    filter_fields = ['name', 'leader', 'tester']


    def list(self, request, *args, **kwargs):
        queryset = self.filter_queryset(self.get_queryset())

        page = self.paginate_queryset(queryset)
        if page is not None:
            serializer = self.get_serializer(page, many=True)
            return self.get_paginated_response(serializer.data)

        serializer = self.get_serializer(queryset, many=True)
        return Response(serializer.data)



    def retrieve(self, request, *args, **kwargs):
        instance = self.get_object()
        serializer = self.get_serializer(instance)
        return Response(serializer.data)



    def update(self, request, *args, **kwargs):
        instance = self.get_object()
        serializer = self.get_serializer(instance, data=request.data, partial=partial)
        serializer.is_valid(raise_exception=True)
        serializer.save()
        return Response(serializer.data)



    def destroy(self, request, *args, **kwargs):
        instance = self.get_object()
        instance.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)



    def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        serializer.save()
        return Response(serializer.data, status=status.HTTP_201_CREATED)
```

projects/urls.py

其中name为给路由起的别名

```python
 path('', views.ProjectsViewSet.as_view({
        'get': 'list',
        'post': 'create',
    }), name='projects-list'),
    path('<int:pk>/', views.ProjectsViewSet.as_view({
        'get': 'retrieve',
        'put': 'update',
        'delete': 'destroy'
    }))
```



## 使用Mixin和GenericViewSet

### 原因

由于Mixin中封装了list、retrieve、update、destroy、create等方法，因此可以直接继承Mixin

GenericViewSet有get_object()、get_serializer()、queryset、serializer_class等方法



### 示例

views.py

```python
from rest_framework import mixins
from rest_framework import viewsets

class ProjectsViewSet(mixins.ListModelMixin,
                      mixins.UpdateModelMixin,
                      mixins.RetrieveModelMixin,
                      mixins.DestroyModelMixin,
                      mixins.CreateModelMixin,
                      viewsets.GenericViewSet):
    queryset = Projects.objects.all()
    serializer_class = ProjectModelSerializer
    ordering_fields = ['name', 'leader']
    filter_fields = ['name', 'leader', 'tester']
```



### 痛点

有没有一个类能继承这些类呢？



## ModelViewSet

### 介绍

继承自GenericAPIVIew，同时继承了ListModelMixin、RetrieveModelMixin、CreateModelMixin、UpdateModelMixin、DestoryModelMixin

支持的Action

列表视图：list、create

详情视图： retrieve、 update、destory



### 示例

views.py

```python
from rest_framework import viewsets

class ProjectsViewSet(viewsets.ModelViewSet):
    queryset = Projects.objects.all()
    serializer_class = ProjectModelSerializer
    ordering_fields = ['name', 'leader']
    filter_fields = ['name', 'leader', 'tester']
```



## Action

### 介绍

可以使用action装饰器来声明自定义的动作

action装饰器：@action(methods, detail)

methods：支持的请求方式

默认值为['get']，可定义多个请求方式['get', 'post', 'put']

detail：声明要处理的是否是详情资源对象（通过url路径获取主键，url是否需要传递pk键值）

True 表示使用通过URL获取的主键对应的数据对象

False 表示不使用URL获取主键



### 示例

1.获取所有的项目名

serializer.py

```python
class ProjectsNameSerializer(serializers.ModelSerializer):
    class Meta:
        model = Projects
        fields = ('id', 'name')
```

views.py

```python
from projects.serializer import ProjectsNameSerializer

class ProjectsViewSet(viewsets.ModelViewSet):
    queryset = Projects.objects.all()
    serializer_class = ProjectModelSerializer
    ordering_fields = ['name', 'leader']
    filter_fields = ['name', 'leader', 'tester']


    #需求：获取所有的项目名
    #1.可以使用action装饰器来声明自定义的动作
    #默认情况下，实例方法名就是动作名
    #methods参数用于指定该动作支持的请求方法，默认为get
    #detail参数用于指定该动作要处理的是否为详情资源对象(url是否需要传递pk键值)
    @action(methods=['get'], detail=False)
    def names(self, request, *args, **kwargs):
        queryset = self.get_queryset()
        serializer = ProjectsNameSerializer(instance=queryset, many=True)
        return Response(serializer.data)
```

projects/urls.py

```python
urlpatterns = [
	path('names/', views.ProjectsViewSet.as_view({
            'get': 'names',
        }), name='projects-names')
]
```

2.获取某个项目id下的所有接口

serializer.py

```python
from interfaces.models import Interfaces


class InterfaceNameSerializer(serializers.ModelSerializer):
    class Meta:
        model = Interfaces
        fields = ('id', 'name', 'tester')



class InterfacesByProjectIdSerializer(serializers.ModelSerializer):
    interfaces_set = InterfaceNameSerializer(read_only=True, many=True)

    class Meta:
        model = Projects
        fields = ('id', 'interfaces_set')
```

views.py

```python
    from projects.serializer import InterfacesByProjectIdSerializer
    
    #获取某个项目id下的所有接口
    @action(detail=False)
    def interfaces(self, request, *args, **kwargs):
        instance = self.get_object()
        serializer = InterfacesByProjectIdSerializer(instance=instance)
        return Response(serializer.data)
```

projects/urls.py

```python
urlpatterns = [
    path('<int:pk>/interfaces/', views.ProjectsViewSet.as_view({
            'get': 'interfaces',
        }))
]
```



## Router

### 介绍

为了简化路由定义，需要使用注册路由



### 步骤

1.创建SimpleRouter路由对象

2.注册路由

3.将自动生成的路由，添加到urlpatterns中



### SimpleRouter示例

这里为了演示起见，做一些修改

study_django/urls.py

```python
urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('projects.urls')),
]
```

projects/urls.py

方法一：

```python
from django.urls import path, include
from rest_framework import routers
from projects import views

#1.创建SimpleRouter路由对象
router = routers.SimpleRouter()
#2.注册路由
#第一个参数prefix为路由前缀，一般添加应用名即可
#第二个参数viewset为视图集，不要加as_view()
router.register(r'projects', views.ProjectsViewSet)

urlpatterns = [
    #3.将自动生成的路由，添加到urlpatterns中
    #path('', include(router.urls))
]
```

方法二：

第3步可修改为

```python
urlpatterns += router.urls
```

在浏览器中运行

![image-20210706002917489](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210706002917489.png)

之前我们定义的路由

![image-20210706002623508](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210706002623508.png)



### @action中url_path与url_name的作用

在names的@action中定义url_path为'nm'，url_name为'url_name'

![image-20210706003100797](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210706003100797.png)

可以看到，如果有定义url_path，则路径为定义的url_path，而不是默认的action name

如果有定义url_name，则别名为项目名-定义的url_name，而不是默认的项目名-默认的action name

![image-20210706003315887](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210706003315887.png)



### DefaultRouter示例

将router改为DefaultRouter()

```python
router = routers.DefaultRouter()
```

运行后发现定义了一个根路由

![image-20210706003821240](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210706003821240.png)

