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





