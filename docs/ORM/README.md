[toc]

# ORM

## 定义

- 将类和数据表进行映射（可以支持多种数据库）
- 通过类和对象就能操作它所对应表格中的数据（CRUD）
- ORM中可以创建模型类，通过模型类，自动创建数据库表



## 安装步骤

### 1.安装mysql客户端，配置数据库连接信息
+ 新建数据库连接. 数据库密码要设置的和DATABASES中设置的一致
+ 新建数据库 字符集选择utf8 排序规则选择utf8_general_ci

### 2.安装mysqllicent

```python
pip install mysqlclient
```

### 3.在settings中配置数据库信息

settings.py

```python
DATABASES = {
    'default': {
        #指定引擎
        'ENGINE': 'django.db.backends.mysql',
        #指定数据库名称
        'NAME': 'learn_django',
        #数据库用户名
        'USER': 'root',
        #数据库用户密码
        'PASSWORD': '123456',
        #HOST
        'HOST': 'localhost',
        'PORT': 3306,
    }
}
```



## 定义数据模型

+ 在子应用的models.py中，定义数据模型
+ 一个数据模型类对应一个数据表
+ 数据模型类，需要继承Model父类或者Model子类
+ 在数据模型类中，添加的类属性（Field对象）来对应数据表中的字段
+ 会自动创建字段名为id的类属性，自增 主键 非空，若某一个字段中primary_key=True，那么django不会自动创建id字段，会使用自己创建的
+ 在执行迁移脚本后，生成的数据表名默认为子应用名_模型类名小写
+ 如果想指定表名则在模型类下定义子类，子类名称固定为Meta，可使用db_table属性，指定表名

```python
from django.db import models

# Create your models here.
#1.每一个应用下的数据库模型类，需要在当前应用下的models.py文件中定义
#2.一个数据库模型类，相当于一个数据表（Table）
#3.一个数据库模型类需要继承Model或者Model的子类
class Person(models.Model):
    '''
    创建Person类
    '''
    #4.定义的一个类属性，就相当于数据库表中的一个字段
    #5.默认会创建一个自动递增的id主键
    #6.默认创建的数据表名称为：应用名小写_数据库模型类小写，比如projects_person
    #id = models.AutoField(primary_key=True)
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)
    
    class Meta:
        db_table = 'tb_person'
        verbose_name = '人类'
        verbose_name_plural = '人类'


class Projects(models.Model):
    '''
    创建Projects模型类
    '''
    #7.max_length为字段的最大长度
    #8.unique参数用于设置当前字段是否唯一，默认为unique=False
    #9.verbose_name用于设置更人性化的字段名
    #10.help_text用于api文档中的一个中文名称
    name = models.CharField(verbose_name='项目名称', max_length=200, unique=True, help_text='项目名称')
    leader = models.CharField(verbose_name='负责人', max_length=50, help_text='负责人')
    tester = models.CharField(verbose_name='测试人员', max_length=50, help_text='测试人员')
    programer = models.CharField(verbose_name='开发人员', max_length=50, help_text='开发人员')
    publish_app = models.CharField(verbose_name='发布应用', max_length=100, help_text='发布应用')
    #11.null设置数据库中此字段允许为空，blank用于设置前端可以不传递，default设置默认值
    desc = models.TextField(verbose_name='简要描述', help_text='简要描述', blank=True, default='', null=True)


    #12.定义子类Meta，用于设置当前数据模型的元数据信息
    class Meta:
        db_table = 'tb_projects'
        #会在admin站点中，显示一个更人性化的表名
        verbose_name = '项目'
        #数据库模型类的复数 apple -》apples 一般和verbose_name保持一致 因为英语单词有单数和复数两种形式，这个属性是模型对象的复数名。中文则跟 verbose_name 值一致。如果不指定该选项，那么默认的复数名字是 verbose_name 加上 ‘s’ 。
        verbose_name_plural = '项目'

```

## 表与表之间的关系

+ 一对一：models.OneToOneField

+ 一对多：models.ForeignKey，“一”：父表；“多”：子表、从表。 如：项目表、接口表，一个接口属于一个项目，一个项目拥有多个接口，因此，项目表（一） -- > 接口表（多）
+ 多对多：models.ManyToManyField

使用```python manage.py startapp Interfaces```创建一个Iterfaces的应用，在settings.py的INSTALLED_APPS中加入```interfaces.apps.InterfacesConfig```

settings.py

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'projects.apps.ProjectsConfig',
    'interfaces.apps.InterfacesConfig',
]
```

Interfaces/models.py

```python
from django.db import models

# Create your models here.

#一个项目中有多个接口
#那么需要在"多"的一侧创建外键
#项目表为父表("一")，接口表("多")为子表
class Interfaces(models.Model):
    name = models.CharField(verbose_name='接口名称', max_length=200, unique=True, help_text='接口名称')
    tester = models.CharField(verbose_name='测试人员', max_length=50, help_text='测试人员')
    desc = models.TextField(verbose_name='简要描述', help_text='简要描述', blank=True, default='', null=True)
    #第一个参数为关联的模型路径(应用名.模型类)或者模型类(如果使用from projects.models import Projects已经导入的话，第一个参数可以直接写Projects)
    #第二个参数设置的是，当父表删除之后，该字段的处理方式
    #CASCADE-->子表也会被删除
    #SET_NULL-->当前外键值会被设置为None，需要设置null=True
    #PROJECT-->会报错
    #SET_DEFAULT-->设置默认值，同时需要指定默认值，null=True
    #外键字段名称一般为父表模型类名小写。如果创建的外键字段是project，那么在数据表中生成的字段是project_id
    project = models.ForeignKey('projects.Projects', on_delete=models.CASCADE,
                                verbose_name='所属项目', help_text='所属项目')


    #定义子类Meta，用于设置当前数据模型的元数据信息
    class Meta:
        db_table = 'tb_interfaces'
        verbose_name = '接口'
        verbose_name_plural = '接口'
```

## 创建完数据库模型类后，需要迁移才能生成数据表

1.执行python manage.py makemigrations (子应用名) 生成迁移脚本，放在projects/migrations目录中（如果不加子应用名，则会将所有子应用进行迁移）

 2.执行迁移脚本：python manage.py migrate

## 使用admin站点添加数据

### 1.注册

projects/admin.py

```python
from .models import Projects, Person
# Register your models here.

admin.site.register(Projects)
admin.site.register(Person)
```

### 2.创建后台用户

使用```python manage.py createsuperuser```创建后台超级用户

```python
(learn_django) E:\virtual_workshop\learn_django>python manage.py createsuperuser
用户名 (leave blank to use 'beck'): beck
电子邮件地址: 123456@qq.com
Password:
Password (again):
密码跟 电子邮件地址 太相似了。
密码长度太短。密码必须包含至少 8 个字符。
这个密码太常见了。
密码只包含数字。
Bypass password validation and create user anyway? [y/N]: y
Superuser created successfully.
```

而创建的用户在auth_user这张表中

![image-20210718234805163](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210718234805163.png)

### 3.登录后台查看

![image-20210718235201922](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210718235201922.png)

### 4.新增项目

![image-20210718235355839](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210718235355839.png)

### 5.优化项目名字

新增后的项目名称

![image-20210719225027666](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210719225027666.png)

使用\_\_str\_\_优化项目名称。projects/models.py

```python
class Projects(models.Model):
    '''
    创建Projects模型类
    '''
    #7.max_length为字段的最大长度
    #8.unique参数用于设置当前字段是否唯一，默认为unique=False
    #9.verbose_name用于设置更人性化的字段名
    #10.help_text用于api文档中的一个中文名称
    name = models.CharField(verbose_name='项目名称', max_length=200, unique=True, help_text='项目名称')
    leader = models.CharField(verbose_name='负责人', max_length=50, help_text='负责人')
    tester = models.CharField(verbose_name='测试人员', max_length=50, help_text='测试人员')
    programer = models.CharField(verbose_name='开发人员', max_length=50, help_text='开发人员')
    publish_app = models.CharField(verbose_name='发布应用', max_length=100, help_text='发布应用')
    #11.null设置数据库中此字段允许为空，blank用于设置前端可以不传递，default设置默认值
    desc = models.TextField(verbose_name='简要描述', help_text='简要描述', blank=True, default='', null=True)


    #12.定义子类Meta，用于设置当前数据模型的元数据信息
    class Meta:
        db_table = 'tb_projects'
        #会在admin站点中，显示一个更人性化的表名
        verbose_name = '项目'
        #数据库模型类的复数 apple -》apples 一般和verbose_name保持一致 因为英语单词有单数和复数两种形式，这个属性是模型对象的复数名。中文则跟 verbose_name 值一致。如果不指定该选项，那么默认的复数名字是 verbose_name 加上 ‘s’ 。
        verbose_name_plural = '项目'


    def __str__(self):
        return self.name
```

![image-20210719225248966](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210719225248966.png)

### 6.指定要显示的字段

指定在新增项目时需要显示的字段使用fields，指定在修改项目时要显示的字段使用list_display

```python
from django.contrib import admin
from .models import Projects, Person
# Register your models here.


class ProjectsAdmin(admin.ModelAdmin):
    '''
    定制后台管理站点类
    '''
    #指定在新增项目时需要显示的字段
    fields = ('name', 'leader', 'tester')

    #指定在修改项目时要显示的字段
    list_display = ['id', 'name', 'leader', 'tester']

admin.site.register(Projects, ProjectsAdmin)
admin.site.register(Person)
```

![image-20210719231701894](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210719231701894.png)

![image-20210719231738559](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210719231738559.png)

![image-20210719231820576](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210719231820576.png)



## CRUD

### 1.C-创建create

方法一：使用模型类对象创建

①创建一个projects模型类对象

②调用模型对象的save()，保存

创建模型类对象，还未执行sql语句，调用save方法保存，才去数据库中执行sql

```python
one_obj = Projects(name='这是一个神奇的项目', leader='John', programer='Alan', publish_app='这是一个神奇的应用', desc='五描述')
one_obj.save()
```

方法二：使用查询集的create方法

objects是manager对象，用于对数据进行操作

使用模型类.objects.create()方法，无需再save

```python
Projects.objects.create(name='这是一个神奇的项目666', leader='John', programer='Alan', publish_app='这是一个神奇的应用666', desc='五描述')
```

### 2.R-查询retrieve

**获取表中的所有记录：all()**

返回QuertSet查询集对象，类似列表，支持列表中的某些操作

+ 支持数字索引取值（负索引不支持，返回模型类对象，一条记录）、切片（返回查询集对象）

+ 支持for循环迭代，每次迭代取出一个模型类对象

+ QuertSet查询集对象.first()获取第一个记录、last()获取最后一条记录

+ count()获取查询集中的记录条数

+ 惰性查询，只有真正使用数据时，才会去数据库中执行sql语句，为了性能要求

+ 链式调用

```python
from projects.models import Projects

qs = Projects.objects.all()
for i in qs:
    print(f"{type(i)}")
    print(f"{i.name}")
```

**获取一条记录：get()**

get方法只能获取一条记录，如果获取多条记录或者查询的记录不存在，则会抛出异常。get方法的参数一般只能使用主键或唯一键(unique)

返回的模型类对象，会自动提交

```python
from projects.models import Projects

#获取id为1的模型类对象
one_project = Projects.objects.get(id=1)

#获取id为1的模型类对象的name值
Projects.objects.get(id=1).name
```

**获取某一些记录：filter()**

fildter方法支持多个过滤表达式，字段名__过滤表达式，如果查询结果为空，则返回 一个空查询集

返回QuertSet查询集对象

```python
Projects.objects.filter()   # 等同于all()
qs = Projects.objects.filter(leader='John')
qs = Projects.objects.filter(leader__contains='John')
qs = Projects.objects.filter(leader__startswith='J')
qs = Projects.objects.filter(leader__in=['Jim','John'])
```

在ORM中有一个内置的变量pk，为数据库模型类的主键别名，不仅仅是id，可以指向任何主键

| 字段名                        |           释义            |
| ----------------------------- | :-----------------------: |
| __ startswith=                |   过滤以xxx开头的字符串   |
| __ isstartswith（忽略大小写） |   过滤以xxx开头的字符串   |
| __ endswith=                  |   过滤以xxx结尾的字符串   |
| __ iendswith（忽略大小写）    |   过滤以xxx结尾的字符串   |
| __ in=[]                      | 过滤出在条件列表中的数据  |
| __ gt=                        |           大于            |
| __ gte=                       |         大于等于          |
| __ lt=                        |           小于            |
| __ lte=                       |         小于等于          |
| __ exact=                     | 等于（等同于 字段名=值 ） |
| __ isnull=Tru                 | 查询字段为空/不为空的数据 |
| __ contains=                  |     过滤包含xxx的数据     |
| __ icontains=（忽略大小写）   |     过滤包含xxx的数据     |
| __ regex=                     |         正则过滤          |



**反向查询：exclude()**

使用方法等同于filter()，查询结果相反

```python
qs = Projects.objects.exclude(id=1)
```



**关联查询**

通过从表的信息获取父表的记录，从表模型类名(小写)__ 从表字段名 __查询表达式

返回查询集对象

比如从表为tb_interfaces，父表为tb_projects，查询接口名字为登录接口对应的父表信息

```python
qs = Projects.objects.filter(interfaces__name='登录接口')
```

![image-20210721235017986](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210721235017986.png)

![image-20210721235134333](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20210721235134333.png)