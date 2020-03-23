## 快速导航
- [一种冗余的设计](#一种冗余的设计）)
- [存放测试数据](#存放测试数据)





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

定义好了数据的存储格式，即```列表[字典]```的形式，下一步就要读取Excel了，我先写出第一种的方式，看看有什么优缺点   
```python
from openpyxl import load_workbook

class DoExcel:

    def __init__(self, filePath, sheetName):
        self.wb = load_workbook(filePath)
        self.sh = self.wb[sheetName]


    #读取所有的测试数据
    def read_allCaseData(self):
        all_case_data = []
        for row in range(2, self.sh.max_row):
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
D:\program\Python37\python.exe E:/python_workshop/python_API/Common/DoExcel.py
10
[{'id': 'register_01', 'url': 'http://localhost:8099/futureloan/mvc/api/member/register', 'method': 'get', 'request_data': '{"mobilephone":"13723456712", "pwd":"test123"}', 'expect_data': '{"status":1,"code":"10001","data":null,"msg":"注册成功"}'}, {'id': 'register_02', 'url': 'http://localhost:8099/futureloan/mvc/api/member/register', 'method': 'get', 'request_data': '{"mobilephone":"13723456713", "pwd":"test123", "regname":"xiaozhai"}', 'expect_data': '{"status":1,"code":"10001","data":null,"msg":"注册成功"}'}, {'id': 'register_03', 'url': 'http://localhost:8099/futureloan/mvc/api/member/register', 'method': 'post', 'request_data': '{"mobilephone":"", "pwd":"test123", "regname":"xiaozhai"}', 'expect_data': '{"status":0,"code":"20103","data":null,"msg":"手机号不能为空"}'}, {'id': 'register_04', 'url': 'http://localhost:8099/futureloan/mvc/api/member/register', 'method': 'get', 'request_data': '{"mobilephone":"13723456715", "pwd":"", "regname":"xiaozhai"}', 'expect_data': '{"status":0,"code":"20103","data":null,"msg":"密码不能为空"}'}, {'id': 'register_05', 'url': 'http://localhost:8099/futureloan/mvc/api/member/register', 'method': 'post', 'request_data': '{"mobilephone":"1372345671", "pwd":"test123", "regname":"xiaozhai"}', 'expect_data': '{"status":0,"code":"20109","data":null,"msg":"手机号码格式不正确"}'}, {'id': 'register_06', 'url': 'http://localhost:8099/futureloan/mvc/api/member/register', 'method': 'get', 'request_data': '{"mobilephone":"137234567156", "pwd":"test123", "regname":"xiaozhai"}', 'expect_data': '{"status":0,"code":"20109","data":null,"msg":"手机号码格式不正确"}'}, {'id': 'register_07', 'url': 'http://localhost:8099/futureloan/mvc/api/member/register', 'method': 'get', 'request_data': '{"mobilephone":"13723456715", "pwd":"test1", "regname":"xiaozhai"}', 'expect_data': '{"status":0,"code":"20108","data":null,"msg":"密码长度必须为6~18"}'}, {'id': 'register_08', 'url': 'http://localhost:8099/futureloan/mvc/api/member/register', 'method': 'get', 'request_data': '{"mobilephone":"13723456715", "pwd":"test123456789012345", "regname":"xiaozhai"}', 'expect_data': '{"status":0,"code":"20108","data":null,"msg":"密码长度必须为6~18"}'}]

Process finished with exit code 0
```

这种方式最大的好处是代码比较清晰，我们看到for循环从第2行开始循环读取测试数据，然后把列名作为key，指定行号和列号的单元格的值作为value，存储到字典中，每行循环完了，把该行对应的字典存储到列表中，下一行开始的时候，重新生成一个空字典。但是缺点也很明显，主要集中在这里：  

```python
 case_data["id"] = self.sh.cell(row, 1).value
 case_data["url"] = self.sh.cell(row, 4).value
 case_data["method"] = self.sh.cell(row, 5).value
 case_data["request_data"] = self.sh.cell(row, 6).value
 case_data["expect_data"] = self.sh.cell(row, 7).value
```
把列号写死，万一要在id和url之间加一列，url后面的列号都得变了，这样维护起来比较麻烦  
