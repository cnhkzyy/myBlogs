## 快速导航
- [```实现测试请求类的两种方式```](#实现测试请求类的两种方式)





## 实现测试请求类的两种方式

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
                result = send_request(case_data["method"], case_data["url"], params=json.loads(case_data["request_data"]))
            elif method.lower() == "post":
                result = send_request(case_data["method"], case_data["url"], data=json.loads(case_data["request_data"]))
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
            result = send_request(case_data["method"], case_data["url"], params=json.loads(case_data["request_data"]))
        elif method.lower() == "post":
            result = send_request(case_data["method"], case_data["url"], data=json.loads(case_data["request_data"]))
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




