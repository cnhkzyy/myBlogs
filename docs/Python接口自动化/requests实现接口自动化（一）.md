## 一种冗余的设计

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


## 存放测试数据

由于接口请求的参数比较整齐划一，一个接口需要url, method, params这些参数，因此可以考虑把数据放在Excel中，使用Excel的好处是：  
1.数据和代码分离，易于维护修改  
2.Excel比较直观，操作方便  

使用Excel的设计形式如下：  

| id | api_name | 用例说明 | url | method | request_data | expect_data |
| -- | -- | -- | -- | -- | --| -- |
| register_01 | register | 注册成功—无昵称 | url地址 | get|{"mobilephone":"13723456712", "pwd":"test123"}|{"status":1,"code":"10001","data":null,"msg":"注册成功"} |



## 读取测试数据的两种方式
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


## 分层设计思想

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



## 封装请求方法的两种方式
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






