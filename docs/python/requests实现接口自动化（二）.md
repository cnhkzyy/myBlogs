## 快速导航
- [```实现测试请求类的两种方式```](#实现测试请求类的两种方式)
- [```实现参数化的两种方式```](#实现参数化的两种方式)





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



## 实现参数化的两种方式

想象一种情况，如果我们的手机号是写死的，那么手机号正常注册后，下次再运行，注册接口会报错：手机号已注册，如何实现手机号码的动态参数化，是摆在我们面前的一个问题。目前有两种解决办法，一是清库，二是随机生成，在实际项目中，一个账号涉及到的数据是比较多的，不仅有数据库，还有缓存，因此清库这种办法比较麻烦，第二种随机生成是可以做到的。随机生成的手机号又分为两种方式，一种是使用Excel，一种是使用Excel + 反射  


### Excel的方式实现参数化  
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
            result = send_request(case_data["method"], case_data["url"], params=json.loads(case_data["request_data"]))
        elif method.lower() == "post":
            result = send_request(case_data["method"], case_data["url"], data=json.loads(case_data["request_data"]))
        print(result)
        self.assertEqual(case_data["expect_data"], result)
```

这样做看起来似乎不错，但基本上都能找到问题，比如```read_init_data()方法中的for i in range(1, 4)，这个4是我们根据${phonex}的个数+1得到的，但这个值你能控制吗？```  

![image-20200323203929698](https://i.loli.net/2020/03/23/p4Vqc9ZPeJAh7iL.png)  

```同样与之关联的是update_init_data()方法中的5```   

![image-20200323204112493](https://i.loli.net/2020/03/23/XOqAWpG9K3fvEFm.png)  


### Excel + 反射的方式实现参数化

