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



