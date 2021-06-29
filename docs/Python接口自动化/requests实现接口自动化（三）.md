
## 添加日志

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


## 生成测试报告

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

## 路径配置化

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