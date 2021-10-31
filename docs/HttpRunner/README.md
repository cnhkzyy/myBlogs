[toc]



# HttpRunner



## 作用

+ 接口自动化测试（unittest等）
+ 接口进行压测（locust等）
+ 持续集成
+ 日志器
+ 轻轻松松进行接口测试平台化
+ UI自动化测试（较弱）



## 安装

```python
pip install httprunner==2.3.0
```



## 创建项目

```python
hrun --startproject 项目名
```



## 工程目录结构(2.x)

![image-20211031115047447](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211031115047447.png)

- api目录
  - 主要存放接口的最小执行单元（正向用例）
- reports目录
  - 主要存放测试报告
- testcases目录
  - 接口复杂的执行逻辑（处理接口依赖）
- testsuites目录
  - 批量执行测试（参数化处理）
- env目录
  - 存放全局环境变量 比如管理员账号密码
- debugtalk文件
  - 主要用于处理动态参数（接口参数的动态化，测试数据的动态化）



## yaml文件用例编写规则

- a.yaml配置文件（json文件类似），主要用于实现测试数据存储或者测试的执行逻辑
- b.强缩进的格式，默认缩进4个空格
- c.冒号前面为key,冒号后面为value，如key: value
- d.缩进一致的且没有'-'，往往为嵌套的字典，如果有'-',那么为列表
- e.yaml中所有的key不可以加引号，value如果加单引号或者双引号，那么一定为字符串类型
- f.如果值为数字或小数，那么为int或float类型，如果值中有字母符号，那么一定为字符串类型



```python
# name固定 指定当前接口名称

name: 项目的列表信息接口

base_url: http://api.keyou.site:8000

# 指定当前用例需要用到的变量

variables:

    # 定义两个变量 变量名可以随便定义
    # 变量名：变量值
    # 定义变量后可以在定义语句的下方任意地方调用 使用$变量名

    uname: "lemon3333"
    passwd: "123456"
    token: "$token"

# 指定接口的请求信息

request:
    # 指定请求的url路径
    url: /projects/

    # 指定请求方式 大写小写都可以 get post put
    method: GET

    # 指定请求头信息
    headers:
        User-Agent: "${get_random_user_agent()}"
        Authorization: "JWT $token"
    
    # json区域来指定json格式请求参数
    # data区域来指定form表单请求参数
    # params区域来指定查询字符串参数
    # files指定文件请求参数
    # 可以查看request库传参 一模一样
    params:
        page: 1
        size: 3


# 指定断言方式
validate:

      # eq指定做相等断言 [实际值,期望值]
      # 默认能使用的属性有：status_code, cookies, elapsed, headers, content, text, json, encoding, ok, reason, url
      # headers为响应头、content 字节,text 字符串, json 都为响应体数据（如果响应数据为json格式的话，那么会自动转化为字典）
      # 字典获取值：content或text或json.字典key
      # 列表获取值：content或text或json.数字索引值.字典key  content.0.username

     - eq: ["status_code", 200]
     - eq: ["content.username", "keyou1"]
     - contains: ["content.username", "1"]
     - {"check": "status_code", "comparator": "eq", "expect": 200}
```

在api下编写login.yml

```python
name: 登录接口
variables:
    var1: value1
    var2: value2
request:
    url:  http://127.0.0.1:8000/users/login/
    method: POST
    headers:
        Content-Type: "application/json"
    json:
        username: admin
        password: 123456
validate:
    - eq: ["status_code", 200]
```



## 运行用例

### 方式一

命令行运行在terminal控制台中执行hrun命令

```python 
hrun yaml文件的绝对路径或相对路径
hrun yaml文件 --log-level debug  设置日志等级为debug，默认为info
```

hrun方法支持如下参数：yaml文件、字典

测试报告如下

![image-20211022220915083](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211022220915083.png)

总结：

优先：适合jenkins持续集成，缺点：不适合平台化



### 方式二

使用py程序运行命令，可以作为平台运行用例的底层逻辑

创建codes/run_test.py

```python
from httprunner.api import HttpRunner


#1.创建HttpRunner对象
#failfast当用例执行失败之后，会自动暂停，设为False，当用例失败后，会继续执行
runner = HttpRunner(failfast=False)

#2.运行用例
#run方法支持如下参数
#yaml用例文件的路径
#字典（用例的信息）
runner.run(r'E:\virtual_workshop\httprunner-learn\testcases\login.yml')

#3.查看运行汇总结果
print(runner.summary)
```



## 全局环境变量

.env文件下定义全局环境变量，可以在任何地方调用

.env

```python
USERNAME=admin
PASSWORD=123456
```

在login.yaml中引用

```python

name: 登录接口
variables:
    var1: value1
    var2: value2
request:
    url:  http://127.0.0.1:8000/users/login/
    method: POST
    headers:
        Content-Type: "application/json"
    json:
        username: ${ENV(USERNAME)}
        password: ${ENV(PASSWORD)}
validate:
    - eq: ["status_code", 200]
```



## 全局变量

variables表示当前登录接口的全局变量，可以在variables区域下定义变量，导入环境变量作为值，然后可以使用$变量名，来获取variables区域下的变量

```python

name: 登录接口
#当前登录接口的全局变量
variables:
    #4.可以在variables区域下定义变量，导入环境变量作为值
    username: ${ENV(USERNAME)}
    password: ${ENV(PASSWORD)}
request:
    url:  http://127.0.0.1:8000/users/login/
    method: POST
    headers:
        Content-Type: "application/json"
    json:
        #1.在项目根目录下的.env中定义环境变量
        #2.调用环境变量，使用${ENV(变量名)}
        #3.可以使用$变量名，来获取variables区域下的变量
        #username: ${ENV(USERNAME)}
        #password: ${ENV(PASSWORD)}
        username: $username
        password: $password
validate:
    - eq: ["status_code", 200]
```



## 函数

在debugtalk.py中创建随机获取user_agent的函数get_user_agent

```python
import random
import time

def sleep(n_secs):
    time.sleep(n_secs)


def get_user_agent():
    user_agent = ['Mozilla/5.0', 'Mozilla/6.0', 'Mozilla/7.0']
    return random.choice(user_agent)
```

在login.yml中可以调用项目根目录下debugtalk.py文件中定义的Python函数

```python

name: 登录接口
#当前登录接口的全局变量
variables:
    #4.可以在variables区域下定义变量，导入环境变量作为值
    username: ${ENV(USERNAME)}
    password: ${ENV(PASSWORD)}
request:
    url:  http://127.0.0.1:8000/users/login/
    method: POST
    headers:
        Content-Type: "application/json"
        #5.可以调用项目根目录下debugtalk.py文件中定义的Python函数
        #使用${函数名(参数)}
        User-Agent: ${get_user_agent()}
    json:
        #1.在项目根目录下的.env中定义环境变量
        #2.调用环境变量，使用${ENV(变量名)}
        #3.可以使用$变量名，来获取variables区域下的变量
        #username: ${ENV(USERNAME)}
        #password: ${ENV(PASSWORD)}
        username: $username
        password: $password
validate:
    - eq: ["status_code", 200]
```

## 公共参数

可以将公共的host提取出来作为base_url

```python

name: 登录接口
base_url: http://127.0.0.1:8000
#当前登录接口的全局变量
variables:
    #4.可以在variables区域下定义变量，导入环境变量作为值
    username: ${ENV(USERNAME)}
    password: ${ENV(PASSWORD)}
request:
    #6.可以将url固定部分，提取出来，在base_url中定义
    url:  /users/login/
    method: POST
    headers:
        Content-Type: "application/json"
        #5.可以调用项目根目录下debugtalk.py文件中定义的Python函数
        #使用${函数名(参数)}
        User-Agent: ${get_user_agent()}
    json:
        #1.在项目根目录下的.env中定义环境变量
        #2.调用环境变量，使用${ENV(变量名)}
        #3.可以使用$变量名，来获取variables区域下的变量
        #username: ${ENV(USERNAME)}
        #password: ${ENV(PASSWORD)}
        username: $username
        password: $password
validate:
    - eq: ["status_code", 200]
```

## 断言

### 断言方式一

可用的响应属性有：status_code、cookies、elapsed、headers、content、text、json、encoding

```python
 - eq: ["status_code", 200]
 - eq: ["headers.Content-Type", "application/json"]
```

### 断言方式二

也可以写成check的形式（上面是缩写，最完整的形式是check的形式）

```python
#8.check指定断言哪一个字段（实际值）
#comparator指定断言的操作（eq、lt、lte、gt、gte、str_eq、len_eq、len_gt、contains、
- {check: "headers.Content-Type", comparator: "eq", expect: "application/json"}
#判断username中包含"ad"字符串
- {check: "json.username", comparator: "contains", expect: "ad"}
#json.username等价于content.username
- {check: "content.username", comparator: "contains", expect: "ad"}

```



## 继承



### testcases继承api

1. testcases下的login.yml继承api目录下的登录接口，会与本区域定义的variables、validate合并覆盖(如本区域内再定义，就会优先覆盖，没有定义，就会继承)

2. 最基础的断言，如status_code，放在api下的login.yml，响应体中的断言放在testcases下的login.yml中
3. testcases中多用于接口依赖场景的创建



testcases/login.yml

```python
config:
    name: "登录接口测试"
    variables:
        device_sn: "ABC"
        username: ${ENV(USERNAME)}
        password: ${ENV(PASSWORD)}
    base_url: "http://127.0.0.1:8000"

teststeps:
-
    name: 登录
    #继承api目录下的登录接口，会与本区域定义的variables、validate合并覆盖
    #如本区域内再定义，就会优先覆盖，没有定义，就会继承
    #最基础的断言，如status_code，放在api下的login.yml，响应体中的断言放在testcases下的login.yml中
    api: api/login.yml
    validate:
        - {check: "content", comparator: "contains", expect: "username"}
```



### testsuites继承testcases

继承testcases下的login.yml的testsuite也遵循合并覆盖的原则，即testsuite下的变量优先级最高

testsuites/api_testsuite.yml

```python
config:
    name: "接口测试套件"
    variables:
        device_sn: "XYZ"
    base_url: "http://127.0.0.1:5000"

testcases:
-
    name: "登录接口"
    testcase: testcases/login.yml
    #可以在parameters下实现数据驱动
    #也实现了合并覆盖
    parameters:
        #方式一
        #2.多个参数具有关联性的参数，需要将其定义在一起，采用短横线进行连接
        - title-username-password-status_code-contain_msg:
              - ["正常登录", "admin", "123456", 200, "token"]
              - ["密码错误", "admin", "1234567", 400, "non_field_errors"]
              - ["账号错误", "admin123", "123456", 400, "non_field_errors"]
              - ["用户名为空", "", "123456", 400, "username"]
              - ["密码为空", "admin", "", 400, "password"]
```



## 数据驱动



### 方式一

使用参数化，多个参数具有关联性的参数，需要将其定义在一起，采用短横线进行连接

testsuites/api_testsuite.yml

```python

config:
    name: "接口测试套件"
    variables:
        device_sn: "XYZ"
    base_url: "http://127.0.0.1:5000"

testcases:
-
    name: "登录接口"
    testcase: testcases/login.yml
    #可以在parameters下实现数据驱动
    #也实现了合并覆盖
    parameters:
        #方式一
        #2.多个参数具有关联性的参数，需要将其定义在一起，采用短横线进行连接
        - title-username-password-status_code-contain_msg:
              - ["正常登录", "admin", "123456", 200, "token"]
              - ["密码错误", "admin", "1234567", 400, "non_field_errors"]
              - ["账号错误", "admin123", "123456", 400, "non_field_errors"]
              - ["用户名为空", "", "123456", 400, "username"]
              - ["密码为空", "admin", "", 400, "password"]

```



在testcases/login.yml中可以使用$变量名的形式引用，但是为了让testcases/login.yml能够单独运行，一般会在其上方的variables下定义变量值

testcases/login.yml

```python

config:
    name: "登录接口测试"
    #这里加变量为了让testcases中这条用例能单独执行
    variables:
        title: "登录"
        status_code: 200
        contain_msg: "token"
    base_url: "http://127.0.0.1:8000"

teststeps:
-
    #这里的title会被api_testsuites中的title变量覆盖
    name: $title
    #继承api目录下的登录接口，会与本区域定义的variables、validate合并覆盖
    #如本区域内再定义，就会优先覆盖，没有定义，就会继承
    #最基础的断言，如status_code，放在api下的login.yml，响应体中的断言放在testcases下的login.yml中
    api: api/login.yml
    validate:
        - {check: "status_code", comparator: "eq", expect: $status_code}
        - {check: "content", comparator: "contains", expect: $contain_msg}
```



### 方式二

当数据驱动的数据特别多的时候，直接放在parameters不方便，可以使用csv文件保存测试数据

使用${P(datas/accounts.csv)}来使用数据，datas/accounts.csv为csv文件路径

创建datas/accounts.csv文件

```python
title,username,password,status_code,contain_msg
正常登录,admin,123456,200,token
密码错误,admin,1234567,400,non_field_errors
账号错误,admin123,123456,400,non_field_errors
用户名为空,,123456,400,username
密码为空,admin,,400,password
```

testsuites/api_testsuite.yml

```python

config:
    name: "接口测试套件"
    variables:
        device_sn: "XYZ"
    base_url: "http://127.0.0.1:5000"

testcases:
-
    name: "登录接口"
    testcase: testcases/login.yml
    #可以在parameters下实现数据驱动
    #也实现了合并覆盖
    parameters:
        #方式二
        #使用csv文件保存测试数据
        #但是读取csv中的200，会默认为str类型，从而造成类型不匹配。可以考虑一些前置后置钩子等处理
        - title-username-password-status_code-contain_msg: ${P(datas/accounts.csv)}

```



### 方式三

使用函数动态生成参数，函数需要返回嵌套字典的列表

debug.talk

```python
def get_accounts():
    accounts = [
        {"title": "正常登录", "username": "admin", "password": "123456", "status_code": 200, "contain_msg": "token"},
        {"title": "密码错误", "username": "admin", "password": "1234567", "status_code": 400, "contain_msg": "non_field_errors"},
        {"title": "账号错误", "username": "admin123", "password": "123456", "status_code": 400, "contain_msg": "non_field_errors"},
        {"title": "用户名为空", "username": "", "password": "123456", "status_code": 400, "contain_msg": "username"},
        {"title": "密码为空", "username": "admin", "password": "", "status_code": 400, "contain_msg": "password"},
    ]
    return accounts
```



testsuites/api_testsuite.py

```python

config:
    name: "接口测试套件"
    variables:
        device_sn: "XYZ"
    base_url: "http://127.0.0.1:5000"

testcases:
-
    name: "登录接口"
    testcase: testcases/login.yml
    #可以在parameters下实现数据驱动
    #也实现了合并覆盖
    parameters:
        #方式三
        #使用函数动态生成参数
        #函数需要返回嵌套字典的列表
        - title-username-password-status_code-contain_msg: ${get_accounts()}
```



