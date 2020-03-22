[TOC]





### 一种冗余的设计

比如我们有注册接口：http://localhost:8099/futureloan/mvc/api/member/register , 其请求方式支持GET/POST，其参数分别为：

| 变量 | 变量名 | 类型 | 说明 | 可否为空 |
| -- | -- | -- | -- | -- |
| 手机号 | mobilephone| String | 新会员的手机号 | 否 |
| 密码 | pwd | String | 6到18位（最少6位，最长18位） | 否 |
| 注册名 | regname | String | 相当于昵称 | 是 |

根据接口的字段要求可以设计如下的用例：

| 用例id | 描述 | 期望结果 | 实际结果 |
| -- | -- | -- | -- |
| register_001 | 正常注册  | 注册成功 | |
| register_002 | 手机号为空字符串 | 提示参数不能为空 | |
| register_003 | 手机号号段不对 | 提示手机号格式不正确 | |
| ... | ... | ... |  |

用代码实现，要写很多调用的方法，重复性高，维护麻烦

```python
import requests

def test_login(params):
    url = "http://localhost:8099/futureloan/mvc/api/member/register"
    req = requests.request(method="get", urr=url, params=params)
    print(req.status_code)
    print(req.text)



#用例1
test_login({"mobilePhone": "13523567801", "pwd": "123"})
#用例2
test_login({"mobilePhone": "", "pwd": "123"})
#用例3
test_login({"mobilePhone": "10086123456", "pwd": "123"})
...
```


### 存放测试数据

由于接口请求的参数比较整齐划一，一个接口需要url, method, params这些参数，因此可以考虑把数据放在Excel中，使用Excel的好处是：  
1.数据和代码分离，易于维护修改  
2.Excel比较直观，操作方便  

使用Excel的设计形式如下：  
| id | api_name | 用例说明 | url | method | request_data | expect_data |
| -- | -- | -- | -- | -- | --| -- |
| register_01 | register | 注册成功—无昵称 | url地址 | get |     {"mobilephone":"13723456712", "pwd":"test123"} |   {"status":1,"code":"10001","data":null,"msg":"注册成功"} |





