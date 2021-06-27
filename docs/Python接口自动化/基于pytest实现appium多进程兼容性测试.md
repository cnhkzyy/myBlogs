# 前言
在实际工作中，如果要用appium实现多设备的兼容性测试，大家想到的也许是“多线程”，但由于python中GIL的影响，多线程并不能做到"多机并行"，这时候可以考虑使用多进程的方式

# 为什么基于pytest
我们知道，pytest中的conftest.py可以定义不同的fixture，测试用例方法可以调用这些fixture，来实现数据共享。以前的框架的思路是：Common目录下的base_driver.py定义生成driver的方法-->conftest.py中调用前者生成driver-->TestCases下的测试用例调用fixture，来实现driver共享 。但是现在不同了，我们有多个设备，这些设备的信息如果只是单纯的写在yml中，我们并行去取的时候似乎也不方便，那可以写在哪里？conftest.py似乎也不是写设备信息的好地方，最后只剩下了main.py，而且将main.py作为多进程的入口再合适不过了
但问题又来了，如果我们想启动多个appium服务，需要考虑以下几点：
1. appium通过什么方式启动？
2. 设备信息如何传递给base_driver方法来生成driver

第一点很明确，客户端启动appium server的方式似乎有点不合时宜了，如果你要同时测5个手机，难道要一个个启动客户端吗？最好的方式是启动命令行，因为命令行启动更方便更快
再说第二点前，先整理一下思路：main.py定义多个设备信息-->base_driver方法调用，生成多个driver-->TestCases下的测试用例调用fixture，但是设备信息怎么传递给base_driver方法呢？这时候pytest中的pytestconfig就派上用场了

# 使用pytestconfig
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

# 具体实现
## 定义main.py
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

## 定义Caps下的caps.yml
这里基本上定义的是多设备相同的desired_caps的公共部分
```yml
platformName: Android
appPackage: com.xxzb.fenwoo
appActivity: com.xxzb.fenwoo.activity.addition.WelcomeActivity
newCommonTimeout: 500
noReset: False
```
## 定义Common下的base_driver.py
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

## 定义conftest.py
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

# 多进程运行
运行main.py，展示多进程运行的截图

http://www.lemfix.com/uploads/photo/2019/25e4f18d-1681-4db5-ac98-d8dc

# 遗留问题
多进程兼容性测试也会带来一些问题：

+ 测试报告如何更好的区分多台设备
+ 对于分辨率不同的机型，要保证一些操作方法的健壮性和稳定性。如A手机屏幕大，确定按钮就在屏幕可见位置，B手机屏幕小，需要多次滑动才能看到按钮，这就要求定义方法时足够健壮
+ 业务逻辑问题。如果并行的去操作（调用同一个接口），会不会有业务逻辑上的限制，比如要抢一个免单券，一天同一个ip，同一个设备只能抢一件，这时候应该只会有一个成功，另一个无疑会失败。这就需要要么调整限制，要么调整方法



