## 快速导航
- [```实现测试请求类的两种方式```](#实现测试请求类的两种方式)
- [```实现参数化的两种方式```](#实现参数化的两种方式)
- [```实现接口关联的两种方式```](#实现接口关联的两种方式)
- [```优化：断言正则化```](#优化：断言正则化)
- [```优化：接口关联之补充```](#优化：接口关联之补充)





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



## 实现参数化的两种方式

想象一种情况，如果我们的手机号是写死的，那么手机号正常注册后，下次再运行，注册接口会报错：手机号已注册，如何实现手机号码的动态参数化，是摆在我们面前的一个问题。目前有两种解决办法，一是清库，二是随机生成，在实际项目中，一个账号涉及到的数据是比较多的，不仅有数据库，还有缓存，因此清库这种办法比较麻烦，第二种随机生成是可以做到的。随机生成的手机号又分为两种方式，一种是使用Excel，一种是使用Excel + 时间戳  


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


### Excel + 时间戳的方式实现参数化

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


## 实现接口关联的两种方式

在接口关联中，我们要处理两种情况，一种是B接口的请求参数需要依赖A接口的返回结果中的某个值，这种常见的例子是获取验证码和登录，登录接口的请求参数中的验证码是从获取验证码接口的返回结果中拿到的；另一种是B接口的请求参数需要依赖A接口的请求参数中的某个值，这种常见的例子是重复注册和注册后登陆  
使用我们旧的参数化方法，也就是Excel init_data做初始化，不存在第二种情况，只要保证注册接口和重复注册接口的变量符号一致(都是${phone2})就行，${phone2}在init_data字典中对应的值是init_data["phone2"]，但是新的参数化方法就不一样了，直接忽略掉了后缀的数字(2)，然后拼接成random_phone，每次都通过使用eval()方法动态的调用random_phone()方法，因此重复注册接口获取到的手机号也是动态生成的，所以需要针对这一问题做相应的处理  

### 设置全局变量  

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


### 引入反射

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
        if params.find("${") != -1:
            replace_string = eval("RandomGenerate().random_{0}()".format(re.sub("\d+", "", re.findall("\${(\\w+)}", params)[0])))
            params = re.sub("\${\\w+}", replace_string, params)
        elif params.find("{{") != -1:
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


## 优化：断言正则化

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


## 优化：接口关联之补充

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

