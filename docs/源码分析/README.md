[toc]





# 源码分析



## 分支：v4.0_创建增删改查五个接口(111-112)

### 代码

projects/urls.py

```python
from django.urls import path
from projects import views


urlpatterns = [
    path('', views.ProjectsList.as_view()),
    path('<int:pk>/', views.ProjectsDetail.as_view())
]

```

projects/views.py

```python
from django.http import HttpResponse, JsonResponse
from django.views import View
from projects.models import Projects
import json

class ProjectsList(View):

    def get(self, request):
        #1.从数据库中获取所有的项目信息
        project_qs = Projects.objects.all()

        #2.将数据模型实例转化为字典类型(嵌套字典的列表)
        project_list = []
        for project in project_qs:
            # one_dict = {
            #     'name': project.name,
            #     'leader': project.leader,
            #     'tester': project.tester,
            #     'publish_app': project.publish_app,
            #     'desc': project.desc
            # }
            # project_list.append(one_dict)

            project_list.append({
                'name': project.name,
                'leader': project.leader,
                'tester': project.tester,
                'publish_app': project.publish_app,
                'desc': project.desc
            })

        #JsonResponse第一个参数默认只能为dict字典，如果要设置为其他类型，需要将safe=False
        #序列化
        return JsonResponse(project_list, safe=False)


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

        #2.向数据库中新增项目
        # project = Projects.objects.create(name=python_dict['name'],
        #                         leader=python_dict['leader'],
        #                         tester=python_dict['tester'],
        #                         programer=python_dict['programer'],
        #                         publish_app=python_dict['publish_app'],
        #                         desc=python_dict['desc'])
        project = Projects.objects.create(**python_data)

        #3.将模型类对象转化为字典返回
        #序列化
        one_dict = {
            'name': project.name,
            'leader': project.leader,
            'tester': project.tester,
            'publish_app': project.publish_app,
            'desc': project.desc
        }

        return JsonResponse(one_dict, status=201)



class ProjectsDetail(View):

    def get(self, request, pk):
        #1.校验前端传递的pk(项目id)值，类型是否正确(正整数)，在数据中是否存在
        #省略
        #2.获取指定pk值的项目
        project = Projects.objects.get(id=pk)

        #3.将模型类对象转化为字典
        #序列化
        one_dict = {
            'name': project.name,
            'leader': project.leader,
            'tester': project.tester,
            'publish_app': project.publish_app,
            'desc': project.desc
        }

        return JsonResponse(one_dict)


    #全部更新，patch为部分更新
    def put(self, request, pk):
        #1.校验前端传递的pk(项目id)值，类型是否正确(正整数)，在数据中是否存在
        #2.获取指定id为pk值的项目
        project = Projects.objects.get(id=pk)

        #3.从前端获取json格式的数据
        #反序列化
        json_data = request.body.decode('utf-8')
        python_data = json.loads(json_data, encoding='utf-8')


        #4.更新项目
        project.name = python_data['name']
        project.leader = python_data['leader']
        project.tester = python_data['tester']
        project.programer = python_data['programer']
        project.publish_app = python_data['publish_app']
        project.desc = python_data['desc']
        project.save()

        #5.将模型类对象转化为字典
        #序列化
        one_dict = {
            'name': project.name,
            'leader': project.leader,
            'tester': project.tester,
            'publish_app': project.publish_app,
            'desc': project.desc
        }

        return JsonResponse(one_dict, status=201)



    def detele(self, request, pk):
        #1.校验前端传递的pk(项目id)值，类型是否正确(正整数)，在数据中是否存在
        #2.获取指定id为pk值的项目
        project = Projects.objects.get(id=pk)
        project.delete()

        return JsonResponse(None, safe=False, status=204)
```

### 源码分析

[django类视图as_view()方法解析](https://www.cnblogs.com/olivertian/p/11072528.html)



## 分支：v4.2_序列化和反序列化-1(113)

### CharField类应用

projects/serializer.py

```python
from rest_framework import serializers


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
    name = serializers.CharField(label='项目名称', max_length=200, help_text='项目名称111', write_only=True)
    leader = serializers.CharField(label='负责人', max_length=50, help_text='负责人')
    tester = serializers.CharField(label='测试人员', max_length=50, help_text='测试人员')
    programer = serializers.CharField(label='开发人员', max_length=50, help_text='开发人员')
    publish_app = serializers.CharField(label='发布应用', max_length=50, help_text='发布应用')
    #allow_null相当于模型类中的null，allow_blank相当于模型类中的blank
    desc = serializers.CharField(label='简要描述', allow_null=True, allow_blank=True, help_text='简要描述', default='')
```



### CharField类源码分析

1. serializers是一个模块文件，实际上在serializers.py中首先从rest_framework.fields导入了各种字段类型对应的类

![image-20211129225321224](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211129225321224.png)



2. 以最常见的CharField为例，看看它的源码的逻辑

   rest_framework/field.py

```python
class CharField(Field):
    default_error_messages = {
        'invalid': _('Not a valid string.'),
        'blank': _('This field may not be blank.'),
        'max_length': _('Ensure this field has no more than {max_length} characters.'),
        'min_length': _('Ensure this field has at least {min_length} characters.'),
    }
    initial = ''

    def __init__(self, **kwargs):
        self.allow_blank = kwargs.pop('allow_blank', False)
        self.trim_whitespace = kwargs.pop('trim_whitespace', True)
        self.max_length = kwargs.pop('max_length', None)
        self.min_length = kwargs.pop('min_length', None)
        super().__init__(**kwargs)
        if self.max_length is not None:
            message = lazy_format(self.error_messages['max_length'], max_length=self.max_length)
            self.validators.append(
                MaxLengthValidator(self.max_length, message=message))
        if self.min_length is not None:
            message = lazy_format(self.error_messages['min_length'], min_length=self.min_length)
            self.validators.append(
                MinLengthValidator(self.min_length, message=message))

        # ProhibitNullCharactersValidator is None on Django < 2.0
        if ProhibitNullCharactersValidator is not None:
            self.validators.append(ProhibitNullCharactersValidator())

    def run_validation(self, data=empty):
        # Test for the empty string here so that it does not get validated,
        # and so that subclasses do not need to handle it explicitly
        # inside the `to_internal_value()` method.
        if data == '' or (self.trim_whitespace and str(data).strip() == ''):
            if not self.allow_blank:
                self.fail('blank')
            return ''
        return super().run_validation(data)

    def to_internal_value(self, data):
        # We're lenient with allowing basic numerics to be coerced into strings,
        # but other types should fail. Eg. unclear if booleans should represent as `true` or `True`,
        # and composites such as lists are likely user error.
        if isinstance(data, bool) or not isinstance(data, (str, int, float,)):
            self.fail('invalid')
        value = str(data)
        return value.strip() if self.trim_whitespace else value

    def to_representation(self, value):
        return str(value)
```

在类创建的时候，有两个属性，一个default_error_messages为字典，另一个initial为空字符串

在实例化的时候，有一些关键字参数：allow_blank、trim_whitespace、max_length、min_length，kwargs.pop的作用是找到参数中最末尾的allow_blank的值，并将它传给实例属性self.allow_blank，如果没有传allow_blank的值，则self.allow_blank=False，以下几个参数的作用皆同

super().__init__(**kwargs)表示它还继承了父类Field中的关键字参数，Field实例化的时候，对应的关键字参数有：

rest_framework/field.py

```python
class Field:
    ...
     def __init__(self, read_only=False, write_only=False,
              required=None, default=empty, initial=empty, 		                 source=None, label=None, help_text=None, style=None,                 error_messages=None, validators=None, allow_null=False):
    ...
```

接下来是个判断，如果self.max_length非空，也就是有最大值，它会直接调用lazy_format这个类，lazy_format从一开始就在field.py中导入了

```python
from rest_framework.utils.formatting import lazy_format
```

同样，self.max_length非空，也就是有最大值，它会直接调用lazy_format这个类

3. 以最大值为例，假设我在serializer.py中定义了这样一条字段

```python
	name = serializers.CharField(label='项目名称', min_length=5, max_length=7, help_text='项目名称111', write_only=True)
```

​		通过debug，可以看到message有一个固定值

![image-20211130234726861](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211130234726861.png)

4. 进入lazy_format类

   rest_framework/utils/formatting.py

   ```python
   class lazy_format:
       """
       Delay formatting until it's actually needed.
   
       Useful when the format string or one of the arguments is lazy.
   
       Not using Django's lazy because it is too slow.
       """
       __slots__ = ('format_string', 'args', 'kwargs', 'result')
   
       def __init__(self, format_string, *args, **kwargs):
           self.result = None
           self.format_string = format_string
           self.args = args
           self.kwargs = kwargs
   
       def __str__(self):
           if self.result is None:
               self.result = self.format_string.format(*self.args, **self.kwargs)
               self.format_string, self.args, self.kwargs = None, None, None
           return self.result
   
       def __mod__(self, value):
           return str(self) % value
   ```
   
   关于\_\_slots__：
   
   默认情况下，访问一个实例的属性是通过访问该实例的\_\_dict__来实现的，如访问a.x就相当于访问a.\_\_dict\_\_['x']，而python内置的字典本质是一个哈希表，它是一种用空间换时间的数据结构，为了解决冲突的问题，当字典使用量超过2/3时，python会根据情况进行2-4倍的扩容，因此使用\_\_slots\_\_可以大幅减少实例的空间消耗。\_\_slots\_\_相当于定义的一个属性名称集合，只有在这个集合里的名称才可以绑定

```python
class Test(object):
    __slots__ = ('x', 'y')
    
    def __init__(self):
        self.x = 10
        self.y = 20
    
t = Test()
print(t.x)  #10
print(t.y)  #20
```

  然后看\_\_str__这个魔术方法，它会返回一个对象的描述信息。如果是实例第一次创  建，那么result=None，这时候它会格式化format_string对应的字符串，这个  format_string以及它对应的动态参数args、关键字参数kwargs，都是实例化lazy_format时带入的参数，再返回到CharField类，看看message这一行

```python
      message = lazy_format(self.error_messages['max_length'], max_length=self.max_length)
```

​	这时lazy_format实例化时的参数对应起来就是，   format_string=self.error_messages['max_length']，*args=max_length=self.max_length

5. 先看self.error_messages，发现error_messages是子类CharField从父类Field继承过来的属性

   rest_framework/fields.py

   ```python
   class Field:
       _creation_counter = 0
   
       default_error_messages = {
           'required': _('This field is required.'),
           'null': _('This field may not be null.')
       }
       ...
       
        def __init__(...):
               ...
           # Collect default error message from self and parent classes
           messages = {}
           for cls in reversed(self.__class__.__mro__):
               messages.update(getattr(cls, 'default_error_messages', {}))
           messages.update(error_messages or {})
           self.error_messages = messages
   ```

   注解说这一段代码是收集实例自身和父类的error message，父类的容易理解，就是通过使用getattr拿到类属性default_error_messages，怎么收集自身的error message？看下self.\_\_class__.\_\_mro\_\_的含义，它表示拿到该类以及所有父类，而reversed是按照继承的优先级排序，基类object第一位，最后是该类本身

   ![image-20211201003040545](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211201003040545.png)

   刚好CharField在实例化的时候，就继承了父类的构造方法，也就是说CharField实例化的时候，会先从父类Field中取出default_error_messages对应的字典，作为键值对添加到空字典messages中，如果没有类属性default_error_messages，则不会新增，然后从该类CharField中取出default_error_messages对应的字典，作为键值对更新到字典messages中

   ```python
   class CharField(Field):
        default_error_messages = {
           'invalid': _('Not a valid string.'),
           'blank': _('This field may not be blank.'),
           'max_length': _('Ensure this field has no more than {max_length} characters.'),
           'min_length': _('Ensure this field has at least {min_length} characters.'),
       }
           ...
           
        def __init__(self, **kwargs):
           self.allow_blank = kwargs.pop('allow_blank', False)
           self.trim_whitespace = kwargs.pop('trim_whitespace', True)
           self.max_length = kwargs.pop('max_length', None)
           self.min_length = kwargs.pop('min_length', None)
   		super().__init__(**kwargs)
           if self.max_length is not None:
               message = lazy_format(self.error_messages['max_length'], max_length=self.max_length)
               self.validators.append(
                   MaxLengthValidator(self.max_length, message=message))
           ...
   ```

   for循环结束后，messages还会将定义序列化器字段时自定义的error_messages添加进来，如果error_messages没有自定义，则不会新增

   ```python
    for cls in reversed(self.__class__.__mro__):
               messages.update(getattr(cls, 'default_error_messages', {}))
   messages.update(error_messages or {})
   self.error_messages = messages
   ```

   因为CharField类继承的Field类，它的\__\__init\_\_方法中定义了有error_messages位置参数

   ```python
   class Field:
       ...
   
       def __init__(self, read_only=False, write_only=False,
    		required=None, default=empty, initial=empty, source=None,
   		label=None, help_text=None, style=None, 		   	               error_messages=None, validators=None, allow_null=False):
   ```

   因此可以在定义序列化器的时候就可以自定义error_messages

   ![image-20211201005259840](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211201005259840.png)

   通过debug，最终看到的error_messages是这样的

   ![image-20211201005743037](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211201005743037.png)

6. 问题来了，我们明明定义的default_error_messages是英文，怎么会显示简体中文？

   ![image-20211201005919279](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211201005919279.png)

   需要注意到，default_error_messages每个键对应的值外面都有一个_()，我们点击\_查看源码，这个实际上是django自带的翻译器

   django/utils/translation/\_\_init\_\_.py

   ```python
   gettext_lazy = lazy(gettext, str)
   ```

   至于为什么跳转到了gettext_lazy而不是\_，因为在fields.py文件一开始导入的时候就给gettext_lazy起了个别名\_

   ![image-20211201010607137](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211201010607137.png)

   7. 这时可以理解当self.max_length有定义时，message为什么是“请确保这个字段不能超过7个字符”。因为代码先会从error_messages字典中取出键max_length对应的值，将它翻译成简体中文：请确保这个字段不能超过 {max_length} 个字符，然后通过lazy_format函数将我们定义的max_length传给它

      ![image-20211201011106849](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211201011106849.png)

      ![image-20211201010856118](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211201010856118.png)

      8. 再来看下下一行中，是往self.validators这个校验器列表中添加数据

         ```python
         self.validators.append(MaxLengthValidator(self.max_length, message=message))
         ```

         self.validators是继承Field类的属性

         ```python
         class Field:
             ...
             default_validators = []
             ...
         
              def __init__(self, read_only=False, write_only=False,
                          required=None, default=empty, initial=empty, source=None,
                          label=None, help_text=None, style=None,
                          error_messages=None, validators=None, allow_null=False):
             ...
             	if validators is not None:
                 	self.validators = list(validators)
                     
             ...
              @property
              def validators(self):
                 if not hasattr(self, '_validators'):
                     self._validators = self.get_validators()
                 return self._validators
             
              @validators.setter
              def validators(self, validators):
                  self._validators = validators
         
              def get_validators(self):
                  return list(self.default_validators)
             ...
             
         ```

         @property负责把一个getter方法变为属性，因为如果使用self.属性=属性值的方法，容易把属性暴露出去，导致属性值可以任意修改，而使用self.validators()直接调用getter方法又略显负责，因此可以给getter方法加上@property装饰器，把它作为一个属性调用，相当于

         ```python
         self = Field()
         
         改变之前：
         validators = self.validators()
         
         改变之后：
         validators = self.validators
         ```

         如果要给变量赋值，使用setter方法self.validators('值')略显麻烦，因此使用@validators.setter装饰器把对应的setter方法变成属性赋值，相当于

         ```python
         self = Field()
         
         改变之前：
         self.validators(['xxx'])
         
         改变之后：
         self.validators = ['xxx']
         ```

         分别两种情况：

         情况一：如果Field类实例化的时候validators有定义，也就是我们在序列化器中定义字段的时候有自定义validators值，如

         ```python
         #创建自定义校验器
         #第一个参数为字段的值
         def is_unique_project_name(name):
             #要求项目的名字必须包含项目
             if '项目' not in name:
                 raise serializers.ValidationError('项目名称中必须包含"项目"')
                 
                 
         #1.继承Serializer类或者子类
         class ProjectSerializer(serializers.Serializer):
             ...
                 
                 name = serializers.CharField(label='项目名称', max_length=200,
                                          help_text='项目名称111', write_only=True,
                                          validators=[UniqueValidator(queryset=Projects.objects.all(), message='项目名不能重复'),
                                                      is_unique_project_name])
              ...
         ```

         那么在实例化的时候，会判断validators是否为空，如果不为空，则直接通过调用self.validators调用setter方法给属性赋值，这个值就是将位置参数validators的值转为列表

         ```python
         if validators is not None:
                self.validators = list(validators)
         ```

         情况二：如果Field类实例化的时候validators没有定义，那么子类CharField中调用self.validators实际上调用的是Field类的getter方法，当self没有_validators属性的时候，它会将调用self.get_validators()，这个方法返回的是空列表

         ```python
         @property
         def validators(self):
             if not hasattr(self, '_validators'):
                 self._validators = self.get_validators()
             return self._validators
             
         @validators.setter
         def validators(self, validators):
             self._validators = validators
         
         def get_validators(self):
             return list(self.default_validators)
         ```
         
         ![image-20211207212850349](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211207212850349.png)
         
         
         
         9. 因此self.validators.append就是将校验器添加到一个校验器列表中，那么MaxLengthValidator类是从哪里来的？
         
            ```python
             self.validators.append(MaxLengthValidator(self.max_length, message=message))
            ```
         
            查看源码，发现MaxLengthValidator是django.core.validators.py中的一个类，这个类继承了父类BaseValidator及其构造方法，又重写了父类的compare和clean方法
         
            因此MaxLengthValidator实例化的时候，可以传两个参数，第一个是位置参数limit_value，第二个是关键字参数message，实际调用中我们传的第一个是self.max_length，也就是序列化器类中字段的最大值max_length，第二个参数传的是message，是经过lazy_format格式化之后的message“请确保这个字段不能超过7个字符”
         
            然后实例化的时候，会给MaxLengthValidator增加属性limit_value，如果message存在的时候，会给它增加属性message
         
            这个\_\_call\_\_方法的意义是，将一个实例作为一个方法来调用，如上所述，self.validators这个校验器列表中放的元素都是一个个校验器实例对象，比如MaxLengthValidator()、MinLengthValidator()等，真正要用的时候，将这些对象再调用一次，就用到了\_\_call\_\_方法
         
            这些实例作为方法调用，要给它们传一个参数value，这个value就是请求传的参数，比如在这个例子中就是调用post请求接口时，给name字段传的值。先调用子类的clean(value)方法，将name字段的值的长度返回赋值给变量cleaned。接下来是一个判断，如果self.limit_value是可调用对象(实现了\_\_call\_\_方法的都是可调用对象，有可能self.limit_value是一个方法名，该方法返回的是一个值)，那么通过self.limit_value拿到可调用对象的值赋值给limit_value，如果不是可调用对象，直接赋值给limit_value。将limit_value（期望字符长度）、cleaned（实际字符长度）、value（实际值）构造一个字典，命名为params，接下来再调用子类的compare方法，将实际字符长度和期望字符长度做个比较，如果实际字符长度>期望字符长度，则返回True，执行raise语句，否则为False，不会执行raise语句，抛出异常
         
            这里的校验逻辑，在后面说到is_valid方法的时候还会提及
            
            django/core/validators.py
            
            ```python
            @deconstructible
            class BaseValidator:
                message = _('Ensure this value is %(limit_value)s (it is %(show_value)s).')
                code = 'limit_value'
            
                def __init__(self, limit_value, message=None):
                    self.limit_value = limit_value
                    if message:
                        self.message = message
            
                def __call__(self, value):
                    cleaned = self.clean(value)
                    limit_value = self.limit_value() if callable(self.limit_value) else self.limit_value
                    params = {'limit_value': limit_value, 'show_value': cleaned, 'value': value}
                    if self.compare(cleaned, limit_value):
                        raise ValidationError(self.message, code=self.code, params=params)
            
                def __eq__(self, other):
                    if not isinstance(other, self.__class__):
                        return NotImplemented
                    return (
                        self.limit_value == other.limit_value and
                        self.message == other.message and
                        self.code == other.code
                    )
            
                def compare(self, a, b):
                    return a is not b
            
                def clean(self, x):
                    return x
            
            
            @deconstructible
            class MaxLengthValidator(BaseValidator):
                message = ngettext_lazy(
                    'Ensure this value has at most %(limit_value)d character (it has %(show_value)d).',
                    'Ensure this value has at most %(limit_value)d characters (it has %(show_value)d).',
                    'limit_value')
                code = 'max_length'
            
                def compare(self, a, b):
                    return a > b
         
                def clean(self, x):
                 return len(x)
            ```
            
            10. 所以无论是序列化器中的IntegerField类也好，还是CharField类，只要继承了Field类，只是向self.validators这个列表中添加校验器，如果在定义字段的时候，本身就传了validators值，那么自定义的校验器会放在列表的前面
            
            