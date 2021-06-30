## 模型序列化器

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



## 指定全部字段校验

在终端中使用ipython实例化，发现会自动生成序列化器

![image-20210629000732119](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210629000732119.png)

## 指定部分字段校验

```python
        fields = ('id', 'name', 'leader', 'tester', 'programer')
```

重新在终端中导入

![image-20210629001202653](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210629001202653.png)



## 反向指定：排除指定的字段

```python
        exclude = ('publish_app', 'desc')
```

重新在终端中导入

![image-20210629222811881](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210629222811881.png)

注意模型序列化器生成的id属性始终为read_only=True



## 指定read_only = True

```python
        read_only_fields = ('leader', 'tester')
```

重新在终端中导入

![image-20210629223315801](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210629223315801.png)



## 自定义校验：覆盖原来的字段

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



## 自定义错误提示

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



## 自定义write_only = True

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



## 单字段和多字段校验

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



## create和update操作

在模型化序列化器中，create()和update()方法会自动生成，如果需要覆盖，重新定义即可



