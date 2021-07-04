[toc]



# 接口自动化

## request实现接口自动化(一)

### 一种冗余的设计

比如我们有注册接口：http://localhost:8099/futureloan/mvc/api/member/register , 其请求方式支持GET/POST，其参数分别为：

| 变量   | 变量名      | 类型   | 说明                         | 可否为空 |
| ------ | ----------- | ------ | ---------------------------- | -------- |
| 手机号 | mobilephone | String | 新会员的手机号               | 否       |
| 密码   | pwd         | String | 6到18位（最少6位，最长18位） | 否       |
| 注册名 | regname     | String | 相当于昵称                   | 是       |

根据接口的字段要求可以设计如下的用例：

| 用例id       | 描述             | 期望结果             | 实际结果 |
| ------------ | ---------------- | -------------------- | -------- |
| register_001 | 正常注册         | 注册成功             |          |
| register_002 | 手机号为空字符串 | 提示参数不能为空     |          |
| register_003 | 手机号号段不对   | 提示手机号格式不正确 |          |
| ...          | ...              | ...                  |          |

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

| id          | api_name | 用例说明        | url     | method | request_data                                   | expect_data                                              |
| ----------- | -------- | --------------- | ------- | ------ | ---------------------------------------------- | -------------------------------------------------------- |
| register_01 | register | 注册成功—无昵称 | url地址 | get    | {"mobilephone":"13723456712", "pwd":"test123"} | {"status":1,"code":"10001","data":null,"msg":"注册成功"} |



### 读取测试数据的两种方式

使用python提供的openpyxl库可以方便的读取测试数据，但是测试数据需要以什么样的方式存储，这个值得考虑。上面说过接口的形式都是比较整齐的，我们后面将这些数据传递给测试函数的时候，如果写很多调用来实现，这样工作量无疑是巨大的。怎么样才能写一次调用，实现多次参数传递，参考for循环的实现，for循环可以对列表、元组、字符串、字典等进行循环，元组限制较多不合适，字符串也不合适，剩下的列表和字典，正是我们想要的。考虑到我们数据存放在Excel中，Excel的行： 列有点类似key: value，因此内层数据可以使用字典，外层数据统一用列表包装即可

```python
以前的方式
def test_login():
	...
	
#用例1
test_login_normal():
	test_login()
	
#用例2
test_login_phoneIsEmpty():
	test_login()

...

```

```python
现在的方式
def test_login():
	...
	
	
all_case_data = [{}, {}, {} ...]

for case_data in all_case_data:
	...
	#只调用一次测试函数
	test_login()  
	...
```

定义好了数据的存储格式，即```列表[字典]```的形式，下一步就要读取Excel了，我先写出第一种方式，看看有什么优缺点   

```python
from openpyxl import load_workbook

class DoExcel:

    def __init__(self, filePath, sheetName):
        self.wb = load_workbook(filePath)
        self.sh = self.wb[sheetName]


    #读取所有的测试数据
    def read_allCaseData(self):
        all_case_data = []
        for row in range(2, self.sh.max_row + 1):
            case_data = {}
            case_data["id"] = self.sh.cell(row, 1).value
            case_data["url"] = self.sh.cell(row, 4).value
            case_data["method"] = self.sh.cell(row, 5).value
            case_data["request_data"] = self.sh.cell(row, 6).value
            case_data["expect_data"] = self.sh.cell(row, 7).value
            all_case_data.append(case_data)
        print(all_case_data)
        return all_case_data


if __name__ == '__main__':
    DoExcel(r"E:\python_workshop\python_API\TestDatas\api_info.xlsx", "case_datas").read_allCaseData()
    
    
    
#运行结果
[{'id': 'register_01', 'url': 'http://localhost:8099/futureloan/mvc/api/member/register', 'method': 'get', 'request_data': '{"mobilephone":"13723456712", "pwd":"test123"}', 'expect_data': '{"status":1,"code":"10001","data":null,"msg":"注册成功"}'}, {'id': 'register_02', 'url': 'http://localhost:8099/futureloan/mvc/api/member/register', 'method': 'get', 'request_data': '{"mobilephone":"13723456713", "pwd":"test123", "regname":"xiaozhai"}', 'expect_data': '{"status":1,"code":"10001","data":null,"msg":"注册成功"}'}, {'id': 'register_03', 'url': 'http://localhost:8099/futureloan/mvc/api/member/register', 'method': 'post', 'request_data': '{"mobilephone":"", "pwd":"test123", "regname":"xiaozhai"}', 'expect_data': '{"status":0,"code":"20103","data":null,"msg":"手机号不能为空"}'}, {'id': 'register_04', 'url': 'http://localhost:8099/futureloan/mvc/api/member/register', 'method': 'get', 'request_data': '{"mobilephone":"13723456715", "pwd":"", "regname":"xiaozhai"}', 'expect_data': '{"status":0,"code":"20103","data":null,"msg":"密码不能为空"}'}, {'id': 'register_05', 'url': 'http://localhost:8099/futureloan/mvc/api/member/register', 'method': 'post', 'request_data': '{"mobilephone":"1372345671", "pwd":"test123", "regname":"xiaozhai"}', 'expect_data': '{"status":0,"code":"20109","data":null,"msg":"手机号码格式不正确"}'}, {'id': 'register_06', 'url': 'http://localhost:8099/futureloan/mvc/api/member/register', 'method': 'get', 'request_data': '{"mobilephone":"137234567156", "pwd":"test123", "regname":"xiaozhai"}', 'expect_data': '{"status":0,"code":"20109","data":null,"msg":"手机号码格式不正确"}'}, {'id': 'register_07', 'url': 'http://localhost:8099/futureloan/mvc/api/member/register', 'method': 'get', 'request_data': '{"mobilephone":"13723456715", "pwd":"test1", "regname":"xiaozhai"}', 'expect_data': '{"status":0,"code":"20108","data":null,"msg":"密码长度必须为6~18"}'}, {'id': 'register_08', 'url': 'http://localhost:8099/futureloan/mvc/api/member/register', 'method': 'get', 'request_data': '{"mobilephone":"13723456715", "pwd":"test123456789012345", "regname":"xiaozhai"}', 'expect_data': '{"status":0,"code":"20108","data":null,"msg":"密码长度必须为6~18"}'}, {'id': 'register_09', 'url': 'http://localhost:8099/futureloan/mvc/api/member/register', 'method': 'get', 'request_data': '{"mobilephone":"13723456713", "pwd":"test123"}', 'expect_data': '{"status":0,"code":"20110","data":null,"msg":"手机号码已被注册"}'}]
```

这种方式最大的好处是代码比较清晰，我们看到for循环从第2行开始循环读取测试数据，然后把列名作为key，指定行号和列号的单元格的值作为value，存储到字典中，每行循环完了，把该行对应的字典存储到列表中，下一行开始的时候，重新生成一个空字典。但是缺点也很明显，主要集中在这里：  

```python
 case_data["id"] = self.sh.cell(row, 1).value
 case_data["url"] = self.sh.cell(row, 4).value
 case_data["method"] = self.sh.cell(row, 5).value
 case_data["request_data"] = self.sh.cell(row, 6).value
 case_data["expect_data"] = self.sh.cell(row, 7).value
```

把列号写死，万一要在id和url之间加一列，url后面的列号都得变了，这样维护起来比较麻烦。刚好之前学过zip可以把两个可迭代对象打包成一个对象，然后使用dict()将其变成字典，再追加到列表中。```第一个对象是第一行的列表组成的列表，第二个对象是每一行的单元格的值组成的列表（这里是全部读取）```  
有了这样的思路，第二种方式实现起来也比较简单：  


```python
from openpyxl import load_workbook

class DoExcel:

    def __init__(self, filePath, sheetName):
        self.wb = load_workbook(filePath)
        self.sh = self.wb[sheetName]




    #读取所有测试数据
    def read_allCaseData(self):
        max_row = self.sh.max_row
        max_column = self.sh.max_column
        all_case_data = []
        title = [self.sh.cell(1, column).value for column in range(1, max_column + 1)]
        for row in range(2, max_row + 1):
            case_data = [self.sh.cell(row, column).value for column in range(1, self.sh.max_column+1)]
            all_case_data.append(dict(zip(title, case_data)))
        print(all_case_data)
        return all_case_data
        


if __name__ == '__main__':
    DoExcel(r"E:\python_workshop\python_API\TestDatas\api_info.xlsx", "case_datas").read_allCaseData()
    
    
#运行结果
D:\program\Python37\python.exe E:/python_workshop/python_API/Common/DoExcel.py
[{'id': 'register_01', 'api_name': 'register', '用例说明': '注册成功—无昵称', 'url': 'http://localhost:8099/futureloan/mvc/api/member/register', 'method': 'get', 'request_data': '{"mobilephone":"13723456712", "pwd":"test123"}', 'expect_data': '{"status":1,"code":"10001","data":null,"msg":"注册成功"}'}, {'id': 'register_02', 'api_name': 'register', '用例说明': '注册成功—有昵称', 'url': 'http://localhost:8099/futureloan/mvc/api/member/register', 'method': 'get', 'request_data': '{"mobilephone":"13723456713", "pwd":"test123", "regname":"xiaozhai"}', 'expect_data': '{"status":1,"code":"10001","data":null,"msg":"注册成功"}'}, {'id': 'register_03', 'api_name': 'register', '用例说明': '手机号为空', 'url': 'http://localhost:8099/futureloan/mvc/api/member/register', 'method': 'post', 'request_data': '{"mobilephone":"", "pwd":"test123", "regname":"xiaozhai"}', 'expect_data': '{"status":0,"code":"20103","data":null,"msg":"手机号不能为空"}'}, {'id': 'register_04', 'api_name': 'register', '用例说明': '密码为空', 'url': 'http://localhost:8099/futureloan/mvc/api/member/register', 'method': 'get', 'request_data': '{"mobilephone":"13723456715", "pwd":"", "regname":"xiaozhai"}', 'expect_data': '{"status":0,"code":"20103","data":null,"msg":"密码不能为空"}'}, {'id': 'register_05', 'api_name': 'register', '用例说明': '手机号长度少于11位', 'url': 'http://localhost:8099/futureloan/mvc/api/member/register', 'method': 'post', 'request_data': '{"mobilephone":"1372345671", "pwd":"test123", "regname":"xiaozhai"}', 'expect_data': '{"status":0,"code":"20109","data":null,"msg":"手机号码格式不正确"}'}, {'id': 'register_06', 'api_name': 'register', '用例说明': '手机号长度多于11位', 'url': 'http://localhost:8099/futureloan/mvc/api/member/register', 'method': 'get', 'request_data': '{"mobilephone":"137234567156", "pwd":"test123", "regname":"xiaozhai"}', 'expect_data': '{"status":0,"code":"20109","data":null,"msg":"手机号码格式不正确"}'}, {'id': 'register_07', 'api_name': 'register', '用例说明': '密码长度少于8位', 'url': 'http://localhost:8099/futureloan/mvc/api/member/register', 'method': 'get', 'request_data': '{"mobilephone":"13723456715", "pwd":"test1", "regname":"xiaozhai"}', 'expect_data': '{"status":0,"code":"20108","data":null,"msg":"密码长度必须为6~18"}'}, {'id': 'register_08', 'api_name': 'register', '用例说明': '密码长度多于18位', 'url': 'http://localhost:8099/futureloan/mvc/api/member/register', 'method': 'get', 'request_data': '{"mobilephone":"13723456715", "pwd":"test123456789012345", "regname":"xiaozhai"}', 'expect_data': '{"status":0,"code":"20108","data":null,"msg":"密码长度必须为6~18"}'}, {'id': 'register_09', 'api_name': 'register', '用例说明': '重复注册', 'url': 'http://localhost:8099/futureloan/mvc/api/member/register', 'method': 'get', 'request_data': '{"mobilephone":"13723456713", "pwd":"test123"}', 'expect_data': '{"status":0,"code":"20110","data":null,"msg":"手机号码已被注册"}'}]

Process finished with exit code 0
```

这种方式有效的避免了第一种方式的缺点，可以任意向Excel中增加列、删除列，读取的将是全部的测试数据   


### 分层设计思想

我们定义了Excel，读取Excel的类，后面还要定义接口请求类、日志类等等，如果将这些东西全部放在一个目录下，看起来杂乱无章。实现数据和代码分离的功能就是为了分层，所以我们的目录最好体现分层思想。新的目录结构设计：

```python
	Common(公共目录)
		--> 处理Excel.py
		--> 封装日志.py
		--> 发送请求.py
		--> 读取配置.py
		...
	Conf(配置目录)
		--> 数据库配置.cfg
		--> url配置.cfg
		...
	TestCases(测试方法目录)
		--> 测试发送请求.py
	TestDatas(测试数据目录)
		--> 测试数据.xlsx
	Logs(日志目录)
		--> 输出日志.log
		...
	HTMLReports(测试报告目录)
		--> 测试报告.log
		...
	main.py(框架入口方法)
```



### 封装请求方法的两种方式

先来说第一种，由于request方法会根据请求method区分请求数据的参数是params还是data，因此需要对method做判断  

```python
import requests


def send_request(method, url, request_data):
    if method.lower() == "get":
        result = requests.request(method, url, params=request_data).text
    elif method.lower() == "post":
        result = requests.request(method, url, data=request_data).text
    print(result)
    return result



if __name__ == '__main__':
    method = "post"
    url = "http://localhost:8099/futureloan/mvc/api/member/register"
    request_data = {"mobilephone":"13623456952", "pwd":"test123"}
    send_request(method, url, request_data)
    
    
#运行结果
D:\program\Python37\python.exe E:/python_workshop/python_API/Common/MyRequest.py
{"status":1,"code":"10001","data":null,"msg":"注册成功"}

Process finished with exit code 0
```

这种方法缺点是扩展性差，如果我要请求方式需要加上cookie，或者header，需要在方法中加上对应的字段  

```python
def send_request(method, url, request_data, cookie=None, header=None):
    if method.lower() == "get":
        result = requests.request(method, url, params=request_data).text
    elif method.lower() == "post":
        result = requests.request(method, url, data=request_data).text
    print(result)
    return result
```

也就是说，不同接口需要的请求信息是变化的，这就要求我们的代码要做到足够的灵活。采用关键字参数\*\*kwargs可以解决这个问题，先来看一组demo             

只有请求参数的情况：  

```python
#只有一个request_data
import requests


def send_request(method, url, **kwargs):
    print(kwargs)



if __name__ == '__main__':
    method = "post"
    url = "http://localhost:8099/futureloan/mvc/api/member/register"
    request_data = {"mobilephone":"13623456952", "pwd":"test123"}
    send_request(method, url, request_data=request_data)
    
    
#运行结果
D:\program\Python37\python.exe E:/python_workshop/python_API/Common/MyRequest.py
{'request_data': {'mobilephone': '13623456952', 'pwd': 'test123'}}

Process finished with exit code 0
```


除了请求参数还有cookie的情况：    

```python
#除了request_data，还有cookie
import requests


def send_request(method, url, **kwargs):
    print(kwargs)



if __name__ == '__main__':
    method = "post"
    url = "http://localhost:8099/futureloan/mvc/api/member/register"
    request_data = {"mobilephone":"13623456952", "pwd":"test123"}
    send_request(method, url, request_data=request_data, cookie="JSLKAJ27643538202")
    

#运行结果
D:\program\Python37\python.exe E:/python_workshop/python_API/Common/MyRequest.py
{'request_data': {'mobilephone': '13623456952', 'pwd': 'test123'}, 'cookie': 'JSLKAJ27643538202'}

Process finished with exit code 0
```

看到这里就明白\*\*kwargs的意义了，关键字可以有效的包装封装的请求方法始终不变，只要改变不同的传参就可以实现不同的定制，接口需要什么传什么，只要遵循key = value的形式就行  

按照关键字参数的思路，我们来重新设计下封装请求方法。先来看这种写法对不对  

```python
import requests


def send_request(method, url, **kwargs):
    print(type(kwargs))
    if method.lower() == "get":
        result = requests.request(method, url, params=kwargs).text
    elif method.lower() == "post":
        result = requests.request(method, url, data=kwargs).text
    print(result)
    return result



if __name__ == '__main__':
    method = "post"
    url = "http://localhost:8099/futureloan/mvc/api/member/register"
    request_data = {"mobilephone":"13623456962", "pwd":"test123"}
    send_request(method, url, request_data=request_data)
```

第一眼感觉没问题，但是怎么运行，它都只会得到一种结果  

```python
{"status":0,"code":"20103","data":null,"msg":"手机号不能为空"}
```

debug一下，就发现问题之所在了，  data=kwargs中的kwargs = {'request_data': {'mobilephone': '13623456962', 'pwd': 'test123'}}，而将data = kwargs传递给 requests.request的\*\*kwargs，会重新组合为data = {'request_data': {'mobilephone': '13623456962', 'pwd': 'test123'}}，也就是```请求参数多了一个request_data的key```，所以接口就找不到手机号了  

![image-20200323124704961](https://i.loli.net/2020/03/23/EeQ2Rr1XpbMCsdl.png)   

这相当于 key = value传递，我们在此基础上使得 key = { key: value }，唯一的办法是少定义一个 key = value，那么就要把请求方法中控制get还是post的判断去掉，让params=request_data或者data=request_data再传递参数时判断即可。这样整体的代码也比较简洁了  

```python
import requests


def send_request(method, url, **kwargs):
    result = requests.request(method, url, **kwargs).text
    print(result)
    return result



if __name__ == '__main__':
    method = "post"
    url = "http://localhost:8099/futureloan/mvc/api/member/register"
    request_data = {"mobilephone":"13623456960", "pwd":"test123"}
    send_request(method, url, data=request_data)
 

#运行结果
D:\program\Python37\python.exe E:/python_workshop/python_API/Common/MyRequest.py
{"status":1,"code":"10001","data":null,"msg":"注册成功"}

Process finished with exit code 0
```

## request实现接口自动化(二)
### 实现测试请求类的两种方式

在TestCases目录下创建测试请求类TestMyRequest，然后获取列表[字典]形式的测试数据，最后使用for循环遍历每一组测试数据  

```python
import unittest
import json
from Common.DoExcel import DoExcel
from Common.MyRequest import *

class TestMyRequest(unittest.TestCase):

    all_case_data = DoExcel(r"E:\python_workshop\python_API\TestDatas\api_info.xlsx", "case_datas").read_allCaseData()


    def test_my_request(self):
        for case_data in self.all_case_data:
            method = case_data["method"]
            if method.lower() == "get":
                result = send_request(method, case_data["url"], params=json.loads(case_data["request_data"]))
            elif method.lower() == "post":
                result = send_request(method, case_data["url"], data=json.loads(case_data["request_data"]))
            print(result)
            self.assertEqual(case_data["expect_data"], result)
```

使用unittest运行后，可以发现多条测试用例被合并成了一条测试用例，这显然是不符合我们期望的，我们期望Excel中的每一行的数据都应该对应着一条测试用例   

![image-20200323174903997](https://i.loli.net/2020/03/23/aPGScodI6M1jh2k.png)


因此，有必要使用ddt对其进行改造，改造后的代码如下  

```python
import unittest
import ddt
import json
from Common.DoExcel import DoExcel
from Common.MyRequest import *


@ddt.ddt
class TestMyRequest(unittest.TestCase):

    all_case_data = DoExcel(r"E:\python_workshop\python_API\TestDatas\api_info.xlsx", "case_datas").read_allCaseData()


    @ddt.data(*all_case_data)
    def test_my_request(self, case_data):
        method = case_data["method"]
        if method.lower() == "get":
            result = send_request(method, case_data["url"], params=json.loads(case_data["request_data"]))
        elif method.lower() == "post":
            result = send_request(method, case_data["url"], data=json.loads(case_data["request_data"]))
        print(result)
        self.assertEqual(case_data["expect_data"], result)
```

再次运行发现报错了，这是ddt的mk_test_name被我们改造过，必须要求Excel第一列的列名是case_id而非id，改过来后发现可以运行成功  

![image-20200323175927151](https://i.loli.net/2020/03/23/Fn3rPoex9kWmzQU.png)

总共9条用例，7条成功，2条失败，我们的目的实现了  

![image-20200323180407121](https://i.loli.net/2020/03/23/2EeJpQhMDdYvnqx.png)

这里需要注意几个地方：  

1. request方法需要的params或data都是字典形式，而Excel读出来的是json字符串，因此需要使用```json.loads()```将其转为Python的字典  
2. 使用\*all_case_data，引用数据变量  
3. ddt运行时的错误排查，如有必要，去源码里可以看看。这里由于修改源码，导致出现了一些问题，如果源码未修改，不会出现这个问题   



### 实现参数化的两种方式

想象一种情况，如果我们的手机号是写死的，那么手机号正常注册后，下次再运行，注册接口会报错：手机号已注册，如何实现手机号码的动态参数化，是摆在我们面前的一个问题。目前有两种解决办法，一是清库，二是随机生成，在实际项目中，一个账号涉及到的数据是比较多的，不仅有数据库，还有缓存，因此清库这种办法比较麻烦，第二种随机生成是可以做到的。随机生成的手机号又分为两种方式，一种是使用Excel，一种是使用Excel + 时间戳  


#### Excel的方式实现参数化  

先看看Excel的方式，下表是case_datas这个工作表的数据，可以用```${phonex}```将变量括起来，这个变量表示要做动态替换  

![image-20200323193925131](https://i.loli.net/2020/03/23/7xBSZQXbHig3yKo.png)  

在另一个工作表init_data中，将mobilephone和value作为标题，${phone1}和具体的手机号作为初始化值  

![image-20200323194255832](https://i.loli.net/2020/03/23/bSZMXzwTUftY2DP.png)  

```python
1. 我们的做法是先获取init_data工作表的值，将其作为一个init_data的字典保存起来，其中字典的key=${phone1}，value=具体的手机号，然后根据case_datas工作表中有几个${phonex}，来决定生成多少个字典元素
2. 在生成all_case_data之前，对每一行的request_data做判断，如果request_data字符串中能找到init_data中的任何一个key，则使用init_data的value来替代request_data中的key-->即${phonex}
3. 最后对init_data工作表中的初始值做一个动态变化
```


整个代码我们做了一些微调，DoExel类实例化时去掉了sheetName这个参数，将filePath设置为全局属性，还有最大的一个变化是在```all_case_data```生成之后，再去做的替换。因为我们之前的读取所有测试数据的写法是按照zip将两个列表合并，而非逐行逐列读取，逐行逐列的缺点上面也提到了，column写死在代码里，新增一行扩展起来不太方便，亦或是通过双层for循环读取行和列，但这样也得到的是行号和列号，要知道哪一列的列名是request_data，可能还需要再做封装。索性在读取之后替换，拿到每一行的数据case_data，去找key = request_data对应的值即可  

```python
from openpyxl import load_workbook

class DoExcel:

    def __init__(self, filePath):
        self.filePath = filePath
        self.wb = load_workbook(self.filePath)
        self.sh = self.wb["case_datas"]
        self.init_sh = self.wb["init_data"]



    #读取所有测试数据
    def read_allCaseData(self):
        init_data = self.read_init_data()
        max_row = self.sh.max_row
        max_column = self.sh.max_column
        all_case_data = []
        title = [self.sh.cell(1, column).value for column in range(1, max_column + 1)]
        for row in range(2, max_row + 1):
            case_data = [self.sh.cell(row, column).value for column in range(1, self.sh.max_column+1)]
            all_case_data.append(dict(zip(title, case_data)))
        for case_data in all_case_data:
            for key, value in init_data.items():
                if case_data["request_data"].find(key) != -1:
                    case_data["request_data"] = case_data["request_data"].replace(key, value)
        print(all_case_data)
        return all_case_data



    #读取init_data
    def read_init_data(self):
        init_data = {}
        for row in range(2, self.init_sh.max_row + 1):
            key = self.init_sh.cell(row, 1).value
            print(key)
            value = int(self.init_sh.cell(row, 2).value)
            print(type(value))
            init_data[key] = str(value)

        for i in range(1, 4):
            new_key = key.replace("1", str(i + 1))
            new_value = str(value + i)

            init_data[new_key] = new_value

        print(init_data)
        return init_data



    #更新init_data的初始值
    def update_init_data(self):
        self.init_sh.cell(2, 2).value = str(int(self.init_sh.cell(2, 2).value) + 5)



    #保存
    def save_excel(self):
        self.wb.save(self.filePath)


    #关闭
    def close_excel(self):
        self.wb.close()



if __name__ == '__main__':
    DoExcel(r"E:\python_workshop\python_API\TestDatas\api_info.xlsx").read_allCaseData()
```

在测试请求类TestMyRequests中，增加了setUpClass和tearDownClass方法，后者在测试结束时做个收尾工作，将Excel中init_data工作表中的初始值更新并保存  

```python
import unittest
import ddt
import json
from Common.DoExcel import DoExcel
from Common.MyRequest import *


@ddt.ddt
class TestMyRequest(unittest.TestCase):
    do_excel = DoExcel(r"E:\python_workshop\python_API\TestDatas\api_info.xlsx")

    all_case_data = do_excel.read_allCaseData()

    @classmethod
    def setUpClass(cls):
        print("接口测试开始啦!")


    @classmethod
    def tearDownClass(cls):
        cls.do_excel.update_init_data()
        cls.do_excel.save_excel()
        cls.do_excel.close_excel()




    @ddt.data(*all_case_data)
    def test_my_request(self, case_data):
        method = case_data["method"]
        if method.lower() == "get":
            result = send_request(method, case_data["url"], params=json.loads(case_data["request_data"]))
        elif method.lower() == "post":
            result = send_request(method, case_data["url"], data=json.loads(case_data["request_data"]))
        print(result)
        self.assertEqual(case_data["expect_data"], result)
```

这样做看起来似乎不错，但基本上都能找到问题，比如```read_init_data()方法中的for i in range(1, 4)，这个4是我们根据${phonex}的个数+1得到的，但这个值你能控制吗？```  

![image-20200323203929698](https://i.loli.net/2020/03/23/p4Vqc9ZPeJAh7iL.png)  

```同样与之关联的是update_init_data()方法中的5```   

![image-20200323204112493](https://i.loli.net/2020/03/23/XOqAWpG9K3fvEFm.png)  


#### Excel + 时间戳的方式实现参数化

这种方式采取截取时间戳和号段一起拼接成动态的手机号，然后替换掉Excel中的${phonex}，那么问题来了  

```python
1. 如何拼接时间戳，这个比较好解决，定义一个随机方法即可
2. 如何识别${phonex}，这个也比较好解决，使用字符串.find()即可
3. 最大的问题是，怎么将${phonex}和随机方法联系起来并替换
```

要解决第3点，我们首先要考虑，${phonex}和随机方法的关联，是不是可以设置随机方法为random_phone()来返回一个动态的手机号，```也就是"${phonex}"使用正则表达式去掉${}和x后得到了"phone"，这个"phone"和"random“拼接成了"random_phone```，我们怎么调用random_phone()方法呢？```python中的eval()可以实现这一功能，eval("random_phone()")执行的就是random_phone()这个方法```，基于这一点，我们完善了一些代码   

在Common目录下定义一个RandomGenerate类，这个类是用来随机生成手机号的  

```python
import time


class RandomGenerate:

    def __init__(self):
        self.timestamp = str(round(time.time_ns()))


    #随机生成手机号
    def random_phone(self):
        random_phone = "131" + self.timestamp[-8:]
        return random_phone



if __name__ == '__main__':
    print(RandomGenerate().random_phone())
```

在Common目录下，再定义一个ReplaceVariable类，这个类是用来替换请求参数的```

```python
import re
from Common.RandomGenerate import *

class ReplaceVariable:


    def replace_varibale(params):
        if params.find("${") != -1:
            replace_string = eval("RandomGenerate().random_{0}()".format(re.sub("\d+", "", re.findall("\${(\\w+)}", params)[0])))
            params = re.sub("\${\\w+}", replace_string, params)
        return params
```

修改DoExcel，修改的目的是去掉一些无用的方法

```python
from openpyxl import load_workbook
from Common.ReplaceVariable import *
import time

class DoExcel:

    def __init__(self, filePath):
        self.filePath = filePath
        self.wb = load_workbook(self.filePath)
        self.sh = self.wb["case_datas"]
        self.init_sh = self.wb["init_data"]



    #读取所有测试数据
    def read_allCaseData(self):
        max_row = self.sh.max_row
        max_column = self.sh.max_column
        all_case_data = []
        title = [self.sh.cell(1, column).value for column in range(1, max_column + 1)]
        for row in range(2, max_row + 1):
            case_data = [self.sh.cell(row, column).value for column in range(1, self.sh.max_column+1)]
            all_case_data.append(dict(zip(title, case_data)))
        return all_case_data




if __name__ == '__main__':
    DoExcel(r"E:\python_workshop\python_API\TestDatas\api_info.xlsx").read_allCaseData()
```

最后再修改测试请求类TestMyRequests，在里面调用替换方法，替换动态的手机号，运行并查看测试结果   

```python
import unittest
import ddt
import json, time
from Common.DoExcel import DoExcel
from Common.MyRequest import *
from Common.ReplaceVariable import ReplaceVariable


@ddt.ddt
class TestMyRequest(unittest.TestCase):
    do_excel = DoExcel(r"E:\python_workshop\python_API\TestDatas\api_info.xlsx")

    all_case_data = do_excel.read_allCaseData()



    @ddt.data(*all_case_data)
    def test_my_request(self, case_data):

        #替换随机手机号
        case_data["request_data"] = ReplaceVariable.replace_varibale(case_data["request_data"])

        method = case_data["method"]
        if method.lower() == "get":
            result = send_request(method, case_data["url"], params=json.loads(case_data["request_data"]))
        elif method.lower() == "post":
            result = send_request(method, case_data["url"], data=json.loads(case_data["request_data"]))
        self.assertEqual(case_data["expect_data"], result)
```

![image-20200323235835199](https://i.loli.net/2020/03/24/CAnyWXNQzx6JD3U.png)  

这样做的一些好处我们总结下：  

1. 数据生成更灵活，不依赖于Excel初始数据，因为不用写Excel，所以Excel打开的情况下也能读取并执行 
2. 不用统计${phonex}有多少个，再决定生成多少个动态的手机号  


### 实现接口关联的两种方式

在接口关联中，我们要处理两种情况，一种是B接口的请求参数需要依赖A接口的返回结果中的某个值，这种常见的例子是获取验证码和登录，登录接口的请求参数中的验证码是从获取验证码接口的返回结果中拿到的；另一种是B接口的请求参数需要依赖A接口的请求参数中的某个值，这种常见的例子是重复注册和注册后登陆  
使用我们旧的参数化方法，也就是Excel init_data做初始化，不存在第二种情况，只要保证注册接口和重复注册接口的变量符号一致(都是${phone2})就行，${phone2}在init_data字典中对应的值是init_data["phone2"]，但是新的参数化方法就不一样了，直接忽略掉了后缀的数字(2)，然后拼接成random_phone，每次都通过使用eval()方法动态的调用random_phone()方法，因此重复注册接口获取到的手机号也是动态生成的，所以需要针对这一问题做相应的处理  

#### 设置全局变量  

具体的思路：  

```python
1. 在Excel中加一列expression表示从返回结果提取，该列的值为空表示不需要提取，不为空表示需要提取
2. 在提取的单元格中写上提取表达式，比如${user_id}="id":(\d+),在需要被替换的接口的请求参数中写上表达式{"memberId":"${user_id}"}
3. 设置一个全局变量var={}在存储提取值
4. 在断言之前判断case_id["expression"]是否为空，如果非空，则使用字符串分割的方式将${user_id}="id":(\d+)分割为${user_id}和‘"id":(\d+)’，其中全局变量的key = ${user_id}，全局变量的value = re.findall('"id":(\d+)', result)[0]，将key和value添加到全局变量var中
5. 在接口请求前判断var的长度是否为空，并且case_data["request_data"]是否为空，如果均为非空，则判断使用for key, value in var.items()来遍历全局变量，接着判断case["request_data"].find(key) != -1，如果判断成立，使用字符串的replace方法将value来取代key
```

Excel中的设计也不难   

 ![image-20200324143640718](https://i.loli.net/2020/03/24/6PgN1AX3W2jyCFK.png)  

代码中需要对测试请求类TestMyRequest做一些调整，一个是断言前提取并存入全局变量，一个是请求前判断并做替换  

```python
...
global var = {}
@ddt.data(*all_case_data)
def test_my_request(self, case_data):
        global global_var
        if len(global_var) > 0 and case_data["request_data"] is not None:
            for key, value in global_var.items():
                if case_data["request_data"].find(key) != -1:
                    case_data["request_data"] = case_data["request_data"].replace(key, value)
                    
         method = case_data["method"]
        if method.lower() == "get":
            result = send_request(method, case_data["url"], params=json.loads(case_data["request_data"]))
        elif method.lower() == "post":
            result = send_request(method, case_data["url"], data=json.loads(case_data["request_data"]))
        
        
        if "expression" in case_data.keys():
            #分割case_data["expression"]，=前面的是key,=后面的是提取表达式
            temp= case_data["expression"].split("=")
            print(temp)
            key = temp[0]
            reg = temp[1]
            #从响应结果中提取出用户id，将其作为值存储在global_var中
            value = re.findall(reg, res)[0]
            global_var[key] = value
            print(global_var)
            
            
        self.assertEqual(case_data["expect_data"], result)        
```


#### 引入反射

反射是个动态的概念，指的是脚本运行过程中自动的给反射类添加属性，可以理解成一个类似字典一样的容器，只不过我们的对象是类，通过调用```类.属性名```的方式获取到依赖的数据，拿到依赖的数据再做一次替换即可。根据这个原理，我们可以将思路整理下  

```python
1. Excel中需要添加request_context和reponse_context的列
2. 如果注册接口需要提取请求数据，则在request_context中加入对应的提取表达式，比如phone2=(\d(11))，response_context的情况依然
3. 如果重复注册接口需要替换请求参数，则使用{{phone2}}的形式，将替换和提取标识关联起来，比如{"mobilephone":"{{phone2}}", "pwd":"test123"}
4. 在Common下创建一个反射类Context
5. 修改ReplaceVariable类，判断如果请求参数里有"{{"，则使用eval调用Context.phone的形式将{{phone2}}替换为Context.phone的属性值
6. 修改测试请求类TestMyRequest，将请求数据反射到反射类
```

Excel中需要加上一些反射的列和反射特有的标识{{参数名}}和提取表达式   

![image-20200324152133326](https://i.loli.net/2020/03/24/Ux9XsS1epAkhzcC.png)

在Common下创建一个反射类Context   

```python
class Context:
    pass
```

修改参数替换类ReplaceVariable类，主要加入对```{{参数}}}的判断和替换```   

```python
import re
from Common.RandomGenerate import *
from Common.Context import Context

class ReplaceVariable:


    def replace_varibale(params):
        if params.find("\$\{") != -1:
            replace_string = eval("RandomGenerate().random_{0}()".format(re.sub("\d+", "", re.findall("\${(\\w+)}", params)[0])))
            params = re.sub("\${\\w+}", replace_string, params)
        elif params.find("\{\{") != -1:
            replace_string = eval("Context.{0}".format(re.findall("{{(\\w+)}}", params)[0]))
            params = re.sub("{{\\w+}}", replace_string, params)
        return params
```

最后修改测试请求类TestMyRequest类，运行并得到返回结果   

```python
import unittest
import ddt
import re
import json, time
from Common.DoExcel import DoExcel
from Common.MyRequest import *
from Common.ReplaceVariable import ReplaceVariable
from Common.Context import Context



@ddt.ddt
class TestMyRequest(unittest.TestCase):
    do_excel = DoExcel(r"E:\python_workshop\python_API\TestDatas\api_info.xlsx")
    all_case_data = do_excel.read_allCaseData()



    @ddt.data(*all_case_data)
    def test_my_request(self, case_data):

        #替换随机参数
        case_data["request_data"] = ReplaceVariable.replace_varibale(case_data["request_data"])

        #请求数据反射
        if case_data["request_context"] != None:
            key, pattern = case_data["request_context"].split("=")
            value = re.findall(pattern, case_data["request_data"])[0]
            setattr(Context, key, value)

        method = case_data["method"]
        if method.lower() == "get":
            result = send_request(method, case_data["url"], params=json.loads(case_data["request_data"]))
        elif method.lower() == "post":
            result = send_request(method, case_data["url"], data=json.loads(case_data["request_data"]))

        self.assertEqual(case_data["expect_data"], result)
```

![image-20200324142009707](https://i.loli.net/2020/03/24/vuAwUC6fJTMi3aH.png)


### 优化：断言正则化

请看一种场景，投资接口返回的测试数据是动态变化的，类似下面这种形式：  

```python
result: {"status":1,"code":"10001","data":{"id":128,"regname":"xiaozhai","pwd":"CC03E747A6AFBBCBF8BE7668ACFEBEE5","mobilephone":"13179150300","leaveamount":"200.00","type":"1","regtime":"2020-03-24 14:46:31.0"},"msg":"充值成功"}
```

在这里返回结果中，如果使用self.assertEqual()显然不合适，对于这个接口，```我们应该只关心字段中的status、code、mobilephone、leaveamount以及msg```，因为就要做部分匹配了，部分匹配我们首先想到的是正则表达式，实际上也确实是这样。对于校验字段，其他的还好，```只有mobilephone需要给case_data["expect_data"]做替换```，那么我们把Excel中的expect_data处理一下  

![image-20200324150628623](https://i.loli.net/2020/03/24/lMzxO8ZNswRiBoj.png)  

然后测试请求类TestMyRequest中对case_data["expect_data"]做替换  

```python
#替换期望结果
case_data["expect_data"] = ReplaceVariable.replace_varibale(case_data["expect_data"])
```

最后使用self.assertIsNotNone和re.search()结合做断言   

```python
self.assertIsNotNone(re.search(case_data["expect_data"], result))
```


### 优化：接口关联之补充

在接口关联那一部分，我们只看了一种情况，就是重复注册接口依赖注册接口的请求手机号，但是依赖返回结果的情况没有说，这里再来看一个场景，获取投资列表接口的请求参数memberId来自于投资接口返回结果中的id，这样先从设计Excel开始，可以把投资接口的response_context设置为响应结果的提取表达式user_id="id":(\d+)，再把获取投资记录里的请求数据中的memberId的值参数化{{user_id}}   

![image-20200324152815664](https://i.loli.net/2020/03/24/LD6Eria8wF7Oh4n.png)  

剩下的我们就去测试请求类TestMyRequest中对提取到的响应结果做反射  

```python
import unittest
import ddt
import re
import json, time
from Common.DoExcel import DoExcel
from Common.MyRequest import *
from Common.ReplaceVariable import ReplaceVariable
from Common.Context import Context



@ddt.ddt
class TestMyRequest(unittest.TestCase):
    do_excel = DoExcel(r"E:\python_workshop\python_API\TestDatas\api_info.xlsx")
    all_case_data = do_excel.read_allCaseData()



    @ddt.data(*all_case_data)
    def test_my_request(self, case_data):

        #替换请求参数
        case_data["request_data"] = ReplaceVariable.replace_varibale(case_data["request_data"])


        #请求数据反射
        if case_data["request_context"] != None:
            key, pattern = case_data["request_context"].split("=")
            value = re.findall(pattern, case_data["request_data"])[0]
            setattr(Context, key, value)


        method = case_data["method"]
        if method.lower() == "get":
            result = send_request(method, case_data["url"], params=json.loads(case_data["request_data"]))
        elif method.lower() == "post":
            result = send_request(method, case_data["url"], data=json.loads(case_data["request_data"]))


        #响应数据反射
        if case_data["response_context"] != None:
            key, pattern = case_data["response_context"].split("=")
            value = re.findall(pattern, result)[0]
            setattr(Context, key, value)


        #替换期望结果
        case_data["expect_data"] = ReplaceVariable.replace_varibale(case_data["expect_data"])
        self.assertIsNotNone(re.search(case_data["expect_data"], result))
```

## request实现接口自动化(三)

### 添加日志

添加日志同样是在Common目录下，创建一个存放日志的目录Logs和日志类MyLogger，实现的方式并不复杂，不再赘述，只是添加下源码  

```python
#MyLogger类
import logging
import time, os
from logging.handlers import RotatingFileHandler


#设置输出的日志内容格式
fmt = '%(asctime)s  %(filename)s  %(funcName)s [line:%(lineno)d] %(levelname)s %(message)s'
datefmt = '%a, %d %b %Y %H:%M:%S'

#设置当前时间
curTime = time.strftime("%Y-%m-%d %H%M", time.localtime())

#设置输出渠道——输出到控制台
hd_1 = logging.StreamHandler()

#设置输出渠道——输出到文件
hd_2 = RotatingFileHandler("E:/python_workshop/python_API/Logs/API_autoTest_log_{0}.log".format(curTime), backupCount=20, encoding="utf-8")

#设置root logger，由于basicConfig()的参数是**kwargs，所以参数要以key=value的形式传入
logging.basicConfig(format=fmt, datefmt=datefmt, level=logging.INFO, handlers=[hd_1, hd_2])

#调用输出方法
#logging.info("hehehe")
```

修改测试请求类TestMyRequest，加入日志并执行，可以看到一些效果  

```python
import unittest
import ddt
import re
import json, time
from Common.DoExcel import DoExcel
from Common.MyRequest import *
from Common.ReplaceVariable import ReplaceVariable
from Common.Context import Context
from Common.MyLogger import *
import logging



@ddt.ddt
class TestMyRequest(unittest.TestCase):
    do_excel = DoExcel(r"E:\python_workshop\python_API\TestDatas\api_info.xlsx")
    all_case_data = do_excel.read_allCaseData()



    @ddt.data(*all_case_data)
    def test_my_request(self, case_data):

        logging.info("{0}--接口请求开始啦!".format(case_data["用例说明"]))

        #替换请求参数
        case_data["request_data"] = ReplaceVariable.replace_varibale(case_data["request_data"])
        logging.info("替换后的请求参数是: " + case_data["request_data"])


        #请求数据反射
        if case_data["request_context"] != None:
            key, pattern = case_data["request_context"].split("=")
            value = re.findall(pattern, case_data["request_data"])[0]
            setattr(Context, key, value)


        method = case_data["method"]
        if method.lower() == "get":
            result = send_request(method, case_data["url"], params=json.loads(case_data["request_data"]))
        elif method.lower() == "post":
            result = send_request(method, case_data["url"], data=json.loads(case_data["request_data"]))


        #响应数据反射
        if case_data["response_context"] != None:
            key, pattern = case_data["response_context"].split("=")
            value = re.findall(pattern, result)[0]
            setattr(Context, key, value)


        #替换期望结果
        case_data["expect_data"] = ReplaceVariable.replace_varibale(case_data["expect_data"])
        logging.info("替换后的期望结果是: " + case_data["expect_data"])
        logging.info("接口的响应结果是: " + result)

        try:
            self.assertIsNotNone(re.search(case_data["expect_data"], result))
            logging.info("断言成功\n\n")
        except Exception as e:
            logging.error("断言失败\n\n")
```

![image-20200324160506468](https://i.loli.net/2020/03/24/qgWEryhz9Ujfivc.png)  


### 生成测试报告

在项目目录下新建一个存放报告的目录HTMLReposts，再创建一个main.py类作为整个脚本执行的入口，其代码如下  

```python
import unittest
from HTMLTestRunnerNew import HTMLTestRunner
import time


#实例化TestSuite
suite = unittest.TestSuite()
#实例化TestLoader
loader = unittest.TestLoader()
#采用loader的discover方法收集测试用例，并将其加载到测试套件中
suite.addTests(loader.discover("E:/python_workshop/python_API/TestCases"))


#设置当前时间
curTime = time.strftime("%Y-%m-%d_%H-%M-%S")
#创建一个html文件
fs = open("E:/python_workshop/python_API/HTMLReports/API_TestReports_{0}.html".format(curTime), "wb")

#实例化HTMLTestRunner
runner = HTMLTestRunner(stream=fs, title="接口自动化测试报告", tester="小翟")
#运行测试用例
runner.run(suite)
```

运行后在HTMLReports目录下，生成对应的html测试报告  

![image-20200324155907365](https://i.loli.net/2020/03/24/jSWaHxQyklevtMB.png)  

### 路径配置化

可以发现，在我们的脚本里充斥着大量的绝对路径，诸如这样的  

![image-20200324160238301](https://i.loli.net/2020/03/24/WdcLDrM1okRu6zp.png)  

一旦工程目录发生变化，给脚本的运行和维护带来了诸多不便。鉴于这种情况，把绝对路径配置化就显得很有必要。首先在Common目录下新建一个ConfDir类，类中的内容为  

```python
import os

cur_dir = os.path.split(os.path.abspath(__file__))[0]

htmlreports_dir = cur_dir.replace("Common", "HTMLReports")

testcases_dir = cur_dir.replace("Common", "TestCases")

testdatas_dir = cur_dir.replace("Common", "TestDatas")

logs_dir = cur_dir.replace("Common", "Logs")
```

接着修改MyLogger.py  

```python
from Common.ConfDir import *

#设置输出渠道——输出到文件
hd_2 = RotatingFileHandler("{0}/API_autoTest_log_{1}.log".format(logs_dir, curTime), backupCount=20, encoding="utf-8")
```

修改TestMyRequest.py  

```python
from Common.ConfDir import *

do_excel = DoExcel(testdatas_dir + "/api_info.xlsx")
```

最后修改main.py  

```python
from Common.ConfDir import *


#采用loader的discover方法收集测试用例，并将其加载到测试套件中
suite.addTests(loader.discover(testcases_dir))
...
#创建一个html文件
fs = open("{0}/API_TestReports_{1}.html".format(htmlreports_dir, curTime), "wb")
```



## 基于pytest实现appium多进程兼容性测试

### 前言

在实际工作中，如果要用appium实现多设备的兼容性测试，大家想到的也许是“多线程”，但由于python中GIL的影响，多线程并不能做到"多机并行"，这时候可以考虑使用多进程的方式

### 为什么基于pytest

我们知道，pytest中的conftest.py可以定义不同的fixture，测试用例方法可以调用这些fixture，来实现数据共享。以前的框架的思路是：Common目录下的base_driver.py定义生成driver的方法-->conftest.py中调用前者生成driver-->TestCases下的测试用例调用fixture，来实现driver共享 。但是现在不同了，我们有多个设备，这些设备的信息如果只是单纯的写在yml中，我们并行去取的时候似乎也不方便，那可以写在哪里？conftest.py似乎也不是写设备信息的好地方，最后只剩下了main.py，而且将main.py作为多进程的入口再合适不过了
但问题又来了，如果我们想启动多个appium服务，需要考虑以下几点：

1. appium通过什么方式启动？
2. 设备信息如何传递给base_driver方法来生成driver

第一点很明确，客户端启动appium server的方式似乎有点不合时宜了，如果你要同时测5个手机，难道要一个个启动客户端吗？最好的方式是启动命令行，因为命令行启动更方便更快
再说第二点前，先整理一下思路：main.py定义多个设备信息-->base_driver方法调用，生成多个driver-->TestCases下的测试用例调用fixture，但是设备信息怎么传递给base_driver方法呢？这时候pytest中的pytestconfig就派上用场了

### 使用pytestconfig

内置的pytestconfig可以通过命令行参数、选项、配置文件、插件、运行目录等方式来控制pytest。pytestconfig是request.config的快捷方式，它在pytest文档里有时候被称为"pytest配置对象"
要理解pytestconfig是如何工作的，可以查看如何添加一个自定义的命令行选项，然后在测试用例中读取该选项。你可以直接从pytestconfig里读取自定义的命令行选项，但是，为了让pytest能够解析它，还需要使用hook函数pytest_addoption
下面使用pytest的hook函数pytest_addoption添加几个命令行选项

```python
pytestconfig/conftest.py
def pytest_addoption(parser):
	parser.addoption("--myopt", action="store_true", help="some boolean option")
	parser.addoption("--foo", action="store", default="bar", help="foo: bar or baz")
```

接下来就可以在测试用例中使用这些选项了

```python
pytest/test_config.py
import pytest

def test_option(pytestconfig):
	print("'foo' set to:", pytestconfig.getoption('foo'))
	print("'myopt' set to:", pytestconfig.getoption('myopt'))
```

让我们看看它是如何工作的

```python
E:\virtual_workshop\pytest-demo\test_demo7\pytestconfig>pytest -s -q test_config.py::test_config
'foo' set to: bar
'myopt' set to: False
.
1 passed in 0.02s

E:\virtual_workshop\pytest-demo\test_demo7\pytestconfig>pytest -s -q --myopt test_config.py::test_config
'foo' set to: bar
'myopt' set to: True
.
1 passed in 0.01s

E:\virtual_workshop\pytest-demo\test_demo7\pytestconfig>pytest -s -q --myopt --foo baz test_config.py::test_config
'foo' set to: baz
'myopt' set to: True
.
1 passed in 0.01s
```

因为pytestconfig是一个fixture，所以它也可以被其他的fixture使用。如果你喜欢，也可以为这些选项创建fixture

```python
@pytest.fixture()
def foo(pytestconfig):
	return pytestconfig.option.foo
	
@pytest.fixture()
def myopt(pytestconfig):
	return pytestconfig.option.myopt
	
def test_fixtures_for_options(foo, myopt):
	print("'foo' set to: ", foo)
	print("'myopt' set to: ", myopt)
```

### 具体实现

#### 定义main.py

既然可以使用pytest命令行参数了，那只需要在pytest.main中加上参数--cmdopt即可，main.py类似这样：

```python
import pytest, os
from multiprocessing import Pool


device_infos = [{"platform_version": "5.1.1", "server_port": 4723, "device_port": 62001, "system_port": 8200},
                {"platform_version": "7.1.2", "server_port": 4725, "device_port": 62025, "system_port": 8201}]



def run_parallel(device_info):
    pytest.main([f"--cmdopt={device_info}",
                 "--alluredir", "Reports"])
    os.system("allure generate Reports -o Reports/html --clean")




if __name__ == "__main__":
    with Pool(2) as pool:
        pool.map(run_parallel, device_infos)
        pool.close()
        pool.join()
```

为什么设备信息我只写了四个？platform_version、server_port、device_port、system_port。其他的类似于appPackage、appActivity、platformName等去哪了？当然你也可以写在这儿，其他的应该都是多个设备相同的，我写在yml的配置信息中了

+ 值得注意的是，这里的server_port多个设备不能重复，这是appium server启动的端口号，如果多个设备server_port都重复，那只能启动一个服务了，所以要不同
+ system_port又是什么？这个是为了防止"互争互抢"现象的发生。多进程多设备并行时，如果多个设备同时使用同一个appium remote port（如8200）。对多个设备而言，它们并不知道相互使用同一port，因此就会出现多个设备发出的Request和接收的Action衔接不上而造成的测试混乱，可能会出现"Original error：Could not proxy command to remote server"的报错

#### 定义Caps下的caps.yml

这里基本上定义的是多设备相同的desired_caps的公共部分

```yml
platformName: Android
appPackage: com.xxzb.fenwoo
appActivity: com.xxzb.fenwoo.activity.addition.WelcomeActivity
newCommonTimeout: 500
noReset: False
```

#### 定义Common下的base_driver.py

这里有几点需要注意下：

+ 多进程在调用BaseDriver类的base_driver方法时，实例化时应该先通过命令行的方式启动appium server，设想一下，如果启动appium server放在get_base_driver中，会出现什么样的场景？conftest中每调用一次get_base_driver方法，就会打开一个cmd窗口，试图去启动appium server
+ yaml.load方法注意新的写法，加上参数 Loader=yaml.FullLoader，这样据说更安全

```python
from appium import webdriver
from .conf_dir import caps_dir
import yaml
import os


class BaseDriver:

    def __init__(self, device_info):
        self.device_info = device_info
        cmd = "start appium -p {0} -bp {1} -U 127.0.0.1:{2}".format(self.device_info["server_port"], self.device_info["server_port"] + 1, self.device_info["device_port"])
        os.system(cmd)



    def base_driver(self, automationName="appium"):
        fs = open(f"{caps_dir}//caps.yml")
        #平台名称、包名、Activity名称、超时时间、是否重置、server_ip、
        desired_caps = yaml.load(fs, Loader=yaml.FullLoader)
        #版本信息
        desired_caps["platform_version"] = self.device_info["platform_version"]
        #设备名称
        desired_caps["deviceName"] = f"127.0.0.1:{self.device_info['device_port']}"
        #系统端口号
        desired_caps["systemPort"] = self.device_info["system_port"]

        if automationName != "appium":
            desired_caps["automationName"] = automationName

        driver = webdriver.Remote(f"http://127.0.0.1:{self.device_info['server_port']}/wd/hub", desired_capabilities=desired_caps)
        return driver
```

#### 定义conftest.py

关键点是pytest_addoption和request.config.getoption这两个函数的使用，一个添加命令行，一个解析命令行，但仍有需要注意的：

+ eval(cmdopt)：之所以使用eval将cmdopt转为字典，是因为cmdopt本身是字符串，类似这样的："{'platform_version': '7.1.2', 'server_port': 4725, 'device_port': 62025, 'system_port': 8201}"，这样取值多不方便。
+ 此外，还需要解决一个问题，如果有多个fixture，必须保证第一个测试用例用到的fixture实现BaseDriver的实例化，并且将这一实例化的结果base_driver作为全局变量，供所有的fixture共用，否则就会出现启动多个cmd窗口，启动多个appium server的问题

```python
from common.base_driver import BaseDriver
import pytest

driver = None


def pytest_addoption(parser):
    parser.addoption("--cmdopt", action="store", default="device_info", help=None)


@pytest.fixture
def cmdopt(pytestconfig):
    #两种写法
    return pytestconfig.getoption("--cmdopt")
    #return pytestconfig.option.cmdopt



#定义公共的fixture
@pytest.fixture
def common_driver(cmdopt):
    global driver
    base_driver = BaseDriver(eval(cmdopt))
    driver = base_driver.base_driver()
    yield driver
    driver.close_app()
    driver.quit()
```

因为pytestconfig是request.config的快捷方式，所以cmdopt也可以写作

```python
@pytest.fixture
def cmdopt(request):
    return request.config.getoption("--cmdopt")
```

### 多进程运行

运行main.py，展示多进程运行的截图

http://www.lemfix.com/uploads/photo/2019/25e4f18d-1681-4db5-ac98-d8dc

### 遗留问题

多进程兼容性测试也会带来一些问题：

+ 测试报告如何更好的区分多台设备
+ 对于分辨率不同的机型，要保证一些操作方法的健壮性和稳定性。如A手机屏幕大，确定按钮就在屏幕可见位置，B手机屏幕小，需要多次滑动才能看到按钮，这就要求定义方法时足够健壮
+ 业务逻辑问题。如果并行的去操作（调用同一个接口），会不会有业务逻辑上的限制，比如要抢一个免单券，一天同一个ip，同一个设备只能抢一件，这时候应该只会有一个成功，另一个无疑会失败。这就需要要么调整限制，要么调整方法



