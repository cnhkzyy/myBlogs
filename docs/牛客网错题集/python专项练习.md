
## 2020年3月22日

1.如下程序会打印多少个数（）

```python
k = 1000
while k > 1;
	print k
	k = k / 2
```
A. 1000  
B. 10  
C. 11  
D. 9  

**答案**: D

**分析**

在python2中，/ 会做取整。因此会有9个  
k = 1000，k > 1，print 1000   
k = 500，k > 1， print 500  
k = 250，k > 1，print 250  
k = 125，k > 1，print 125  
k = 62，k > 1，print 62  
k = 31，k > 1，print 31  
k = 15，k > 1，print 15  
k = 7，k > 1，print 7  
k = 3，k > 1，print 3  
k = 1  

而在python3中，/ 不做取整，结果是浮点的，会有10个。如果想要得到9个，需要用地板除 // 取整  
k = 1000，k > 1，print 1000  
k = 500.000000，k > 1， print 500.000000  
k = 250.000000，k > 1，print 250.000000  
k = 125.000000，k > 1，print 125.000000  
k = 62.500000，k > 1，print 62.500000  
k = 31.250000，k > 1，print 31.250000  
k = 15.625000，k > 1，print 15.625000  
k = 7.812500，k > 1，print 7.812500  
k = 3.906250，k > 1，print 3.906250  
k = 1.953125， k > 1，print 1.953125  

因为题目中使用的是print k，为python2的语法，所以选D   


2.假设可以不考虑计算机运行资源（如内存）的限制，以下python3代码的预期运行结果是：（）

```python
import math
def sieve(size):
	sieve = [True] * size
	sieve[0] = False
	sieve[1] = False
	for i in range(2, int(math.sqrt(size) + 1)):
		k = i * 2
		while k < size:
			sieve[k] = False
			k += 1
	return sum(1 for x in sieve if x)
print(sieve(10000000000))
```

A. 455052510  
B. 455052511  
C. 455052512  
D. 455052513  

**答案**: B

**分析**

举个小点的例子，当size=10时：  
size = [True, True, True, True, True, True, True, True, True, True]，总共有10个元素是True  
size = [Flase, False, True, True, True, True, True, True, True, True]，第1个元素和第2个元素设置成了False  

```python
for i in range(2, int(3) + 1):
	外层for循环第一遍
	i = 2
	k = 4
        内层while循环第一遍
        4 < 10:
            sieve[4] = False
            k = 4 + 2 = 6
        内层while循环第二遍
        6 < 10:
        	sieve[6] = False
        	k = 6 + 2 = 8
        内层while循环第三遍
      	8 < 10:
      		sieve[8] = False
      		k = 8 + 2 = 10
      	跳出内层while循环
    外层for循环第二遍
    i = 3
    k = 6
    	内层while循环第一遍
    	6 < 10:
    	sieve[6] = False
        	k = 6 + 3 = 9
        内层while循环第二遍
      	9 < 10:
      		sieve[9] = False
      		k = 9 + 3 = 12
      	跳出内层while循环
    跳出外层for循环

因此最终的sieve = [Flase(0), False(1), True(2), True(3), False(4), True(5), False(6), True(7), False(8), False(9)]
```

0-9之间有4个为True的数，分别是：2, 3, 5, 7，这些都是质数，所以sum = 4   
至于0-10000000000之间有多少个质数，只能靠蒙了  


3.下列代码运行结果是（）
```python
a = 'a'
print a > 'b' or 'c
```

A. a  
B. b  
C. c  
D. d  
E. True  
F. False   

**答案**: C

**分析**

| 运算符 | 逻辑表达式 | 描述 | 实例 |
| -- | -- | -- | --|
| and | x and y | 布尔与，如果x为False，x and y 返回Flase，否则它返回y的计算值 | x为10，y为20，(x and y)返回20 |
| or | x or y | 布尔或，如果x为True，它返回x的值，否则返回y的计算值 | x为10，y为20，(x or y)返回10 |
| not | not x | 布尔非，如果x为True，返回False，如果x为False，返回True | not(a and b)返回False |


4.以下函数输出结果为（）
```python
import numpy as np
a = np.repeat(np.arange(5).reshape([1,-1]),10,axis = 0)+10.0 
b = np.random.randint(5, size= a.shape)
c = np.argmin(a*b, axis=1)
b = np.zeros(a.shape)
b[np.arange(b.shape[0]), c] = 1
print b
```

A. Hello World!    
B. 一个 shape = (5,10) 的随机整数矩阵  
C. 一个 shape = (5,10) 的 one-hot 矩阵  
D. 一个 shape = (10,5) 的 one-hot 矩阵  

**答案**: D

**分析**

```python
np.arange(5) -->  [0, 1, 2, 3, 4, 5]
reshape[1, -1]  -->  将原列表重组为1行，5/1列的新矩阵 1, 2, 3, 4, 5]]
axis=0，沿着x方向重复，增加10行  a    -->
[[0 1 2 3 4]
 [0 1 2 3 4]
 [0 1 2 3 4]
 [0 1 2 3 4]
 [0 1 2 3 4]
 [0 1 2 3 4]
 [0 1 2 3 4]
 [0 1 2 3 4]
 [0 1 2 3 4]
 [0 1 2 3 4]]

+10.0，给每个元素都 + 10.0  a  -->
[[10. 11. 12. 13. 14.]
 [10. 11. 12. 13. 14.]
 [10. 11. 12. 13. 14.]
 [10. 11. 12. 13. 14.]
 [10. 11. 12. 13. 14.]
 [10. 11. 12. 13. 14.]
 [10. 11. 12. 13. 14.]
 [10. 11. 12. 13. 14.]
 [10. 11. 12. 13. 14.]
 [10. 11. 12. 13. 14.]]
 
np.random.randint(5, size= a.shape)  随机生成a.shape大小的矩阵，矩阵的元素范围[0-5]之间的整数   b   -->

[[1 1 2 0 4]
 [3 2 4 0 4]
 [1 2 1 0 1]
 [2 2 1 2 4]
 [0 0 4 0 3]
 [2 4 0 4 2]
 [3 4 2 0 3]
 [1 3 4 4 4]
 [0 1 3 2 2]
 [2 0 0 0 2]]
 
np.argmin(a*b, axis=1)   矩阵a和b相乘   c   -->
[1 1 3 1 0 0 3 0 2 4]
 
np.zeros(a.shape)   生成和a矩阵大小相等的全0矩阵
[[0. 0. 0. 0. 0.]
 [0. 0. 0. 0. 0.]
 [0. 0. 0. 0. 0.]
 [0. 0. 0. 0. 0.]
 [0. 0. 0. 0. 0.]
 [0. 0. 0. 0. 0.]
 [0. 0. 0. 0. 0.]
 [0. 0. 0. 0. 0.]
 [0. 0. 0. 0. 0.]
 [0. 0. 0. 0. 0.]]


b[np.arange(10), c]=1表示np.arange(10)生成的数组中，所有c对应的位置全置为1
[[0. 1. 0. 0. 0.]
 [0. 1. 0. 0. 0.]
 [0. 0. 0. 1. 0.]
 [0. 1. 0. 0. 0.]
 [1. 0. 0. 0. 0.]
 [1. 0. 0. 0. 0.]
 [0. 0. 0. 1. 0.]
 [1. 0. 0. 0. 0.]
 [0. 0. 1. 0. 0.]
 [0. 0. 0. 0. 1.]]
```

因此是一个10行，5列的矩阵


5.下面的程序根据用户输入的三个边长a, b, c来计算三角形面积，请指出程序中的错误：（设用户输入合法面积公式无误）（） 
```python
import math
a, b, c = raw_input("Enter a, b, c: ")
s = a + b + c
s = s / 2.0
area = sqrt(s * (s - a) * (s - b) * (s - c))
print "The area is: ", area
```

A. 第一行  
B. 第二行  
C. 第五行  
D. 第六行  

**答案**: B和C

**分析**

第二行，在python3中，input输入的都是字符串，所以a, b, c = "12 13 14"，会报错ValueError: too many values to unpack (expected 3)。因为正常情况下，a = 1, b = 2, c = " "，剩下的值没有变量接收  
第五行，应该使用math.sqrt，否则会报NameError: name 'sqrt' is not defined  


## 2020年3月29日

1.关于Python中的复数，下列说法错误的是（）  
A.表示复数的语法是real + image j  
B.实部和虚部都是浮点型  
C.虚部必须后缀j，且必须小写  
D.方法conjugate返回复数的共轭复数  

**答案**：C

**分析** 

```把形如z = a + bj(a, b均为实数)的数称为复数，其中a称为实部，b称为虚部，j称为虚数单位,其中虚拟部分必须有后缀j或J,实数部分和虚部部分在Python中都是浮点数```   

```python
C:\Users\beck
λ python
Python 3.7.3 (v3.7.3:ef4ec6ed12, Mar 25 2019, 22:22:05) [MSC v.1916 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> a = 1 + 2j
>>> a.real
1.0
>>> type(a.real)
<class 'float'>
>>> a.imag
2.0
>>> type(a.imag)
<class 'float'>
>>>
```

2.已知print_func.py的代码如下： 

```python
print('Hello
World!')
print('__name__
value: ', __name__)
  
def main():
    
print('This message is from main function')
  
if __name__ ==
'__main__':
    
main()
  
print_module.py的代码如下：
import print_func
print("Done!")
```

运行print_module.py程序，结果是：  

```python
A. Hello World!  __name__ value: print_func  Done!    

B. Hello World!  __name__ value: print_module  Done!       

C. Hello World!  __name__ value: __main__  Done!     

D. Hello World!  __name__ value:  Done!  
```

**答案**:A

**分析**
一个python的文件有两种使用的方式，第一种直接作为脚本执行，第二种是import到其他的python模块中被调用时执行，在自己的模块中时，__name__的值等于'__main__‘，当导入到外部模块时，这个值是去除后缀.py的模块名  

![image-20200329213035328](https://i.loli.net/2020/03/29/6DzBwhEdkuGWe9L.png)  

![image-20200331223935745](https://i.loli.net/2020/03/31/UwcxjfzqJX5brKS.png)  

3.对于以下代码，描述正确的是：  
```python
list = ['1', '2', '3', '4', '5']
print list[10:]
```

A. 导致 IndexError  
B.输出['1', '2', '3', '4', '5']  
C.编译错误  
D.输出[ ]  

**答案**:D

**分析**

```list[10]会提示越界，但list[10:]输出为空列表```

4.python代码如下： 
```python
foo = [1,2]
foo1 = foo
foo.append(3)
```

A.foo值为[1,2]   
B.foo值为[1,2,3]  
C.foo1值为[1,2]  
D.foo1值为[1,2,3]  

**答案**：BD

**分析**

```列表在python中以"列表类"的形式存在，创建一个列表，即实例化一个类。python里的对象赋值实际上是对象的引用。foo = [1,2]， foo1 = foo，就是将列表对象赋予"foo1"，此时foo和foo1指向内存中同一个对象，而append()是列表类的方法，append()方法是在自身对象上进行操作，因为foo和foo1都是指向同一个列表对象，所以foo和foo1会得到同样的结果，另一个例子，foo = [1,2]，foo1 = foo，foo = [3]的结果是foo1 = [1, 2], foo = [3]，因为foo = [3]会实例化一个新的列表类```

5.__new__和__init__的区别，说法正确的是（）  
A.__new__是一个静态方法，而__init__是一个实例方法  
B.__new__方法会返回一个创建的实例，而__init__什么都不返回  
C.只有在__new__返回一个cls的实例时，后面的__init__才能被调用  
D.当创建一个新实例时调用__new__，初始化一个实例时调用__init__  

**答案**：A B C D

**分析**

```python3中的类都是type类的实例，__new__方法创建并返回了新的class对象，随后__init__方法初始化了新创建的对象，并不返回值。从源码中看__new__是一个静态方法，而__init__是实例方法```

```python
@staticmethod # known case of __new__
    def __new__(*args, **kwargs): # real signature unknown
        """ Create and return a new object.  See help(type) for accurate signature. """
        pass
        
    def __init__(cls, what, bases=None, dict=None): # known special case of type.__init__
        """
        type(object_or_name, bases, dict)
        type(object) -> the object's type
        type(name, bases, dict) -> a new type
        # (copied from class doc)
        """
        pass
```

![image-20200329224139496](https://i.loli.net/2020/03/29/Z5BRYzkhKJOpH6G.png)   

6.Python中单下划线\_foo与双下划线__foo与__foo__的成员，下列说法正确的是（）  

A. \_foo 不能直接用于’from module import \*'  
B. \__foo解析器用\_classname\__foo来代替这个名字，以区别和其他类相同的命名  
C. __foo__代表python里特殊方法专用的标识  
D. \__foo可以直接用于’from module import \*'  

**答案**：ABC

**分析**

```python
python中主要存在四种命名方式：
1. object      公共方法
2. _object     半保护，相当于"protect"，只有类对象和子类对象能访问到这些变量，在模块外或类外不可以使用，不能用from module import *导入
3.__object     全私有，全保护，私有成员"private"，只有类对象能访问，连子类对象都不能访问，更不能用from module import * 导入
__object为了避免与子类的方法名称冲突，对于该标识符描述的方法，父类的方法不能轻易被子类的方法覆盖，它们的名字实际上是_classname__methodname
4.__object__   内建方法，用户不要这样定义
```

![image-20200329225735454](https://i.loli.net/2020/03/29/1zmu7bZpF9y8Yr4.png)  

7.若 a = range(100)，以下哪些操作是合法的（）  

A. a[-3]  
B. a[2:13]  
C. [::3]  
D. a[2-3]  

**答案**: ABCD

**分析**

```python
a[: : 3]  start为0，end为99，step为3，依次是0 3 6 9...99
a[2 - 3]即a[-1]，等于99
```

8.下列关于python socket操作叙述正确的是（）  
A. 使用recvfrom()接收TCP数据  
B. 使用getsockname()获取连接套接字的远程地址  
C. 使用connect()初始化TCP服务器连接  
D. 服务端使用listen()开始TCP监听  

**答案**:CD

**分析**
recvfrom()接收UDP消息  
getsockname()返回当前套接字的地址  


## 2020年3月31日 

1.下列哪个语句在Python是非法的？（）  

A. x = y = z = 1  
B. x = (y = z + 1)  
C. x, y = y, x  
D. x += y  

**答案**: B

**分析**
```python
>>> x=y=z=1
>>> x
1
>>> y
1
>>> z
1
>>> x=(y=z+1)
  File "<stdin>", line 1
    x=(y=z+1)
        ^
SyntaxError: invalid syntax

y = z + 1 赋值结果不会返回值 再=x就会出错
```

2.下列代码输出为：  
```python
str = "Hello, Python"
suffix = "Python"
print(str.endwith(suffix, 2))
```

A. True  
B. False  
C. 语法错误  
D. P  

**答案**: A 

**分析** 
```python
查询str.endwith()方法的源码如下

def endswith(self, suffix, start=None, end=None): # real signature unknown; restored from __doc__
    """
    S.endswith(suffix[, start[, end]]) -> bool

    Return True if S ends with the specified suffix, False otherwise.
    With optional start, test S beginning at that position.
    With optional end, stop comparing S at that position.
    suffix can also be a tuple of strings to try.
    """
    return False
    
这个方法是判断字符串是否以指定后缀结尾，如果以指定后缀结尾返回True，否则返回False
可选参数"start"和"end"为检索字符串的开始于结束位置

举个例子：
"Hello,Python".endswith("Python", 2)   --> 检索范围为"llo, Python"，返回True
"Hello,Python".endswith("Python", 6)   --> 检索范围为"Python"，返回True
"Hello,Python".endswith("Python", 7)   --> 检索范围为"ython"，返回False
"Hello,Python".endswith("Python", 6, 11)   --> 检索范围为"Pytho"，返回False
"Hello,Python".endswith("Python", 6, 12)   --> 检索范围为"Python"，返回True
```

3.下面代码运行后，a、b、c、d四个变量的值，描述错误的是（）  

```python
import copy
a = [1, 2, 3, 4, ['a', 'b']]
b = a
c = copy.copy(a)
d = copy.deepcopy(a)
a.append(5)
a[4].append('c')
```
A. a ==  [1,2, 3, 4, ['a', 'b', 'c'], 5]  
B. b ==  [1,2, 3, 4, ['a', 'b', 'c'], 5]  
C. c ==  [1,2, 3, 4, ['a', 'b', 'c']]  
D. d ==  [1,2, 3, 4, ['a', 'b', ‘c’]]  

**答案**: D

**分析** 

+ 直接赋值：就是对象的引用（别名）
+ 浅拷贝（copy）：拷贝父对象，不拷贝对象内部的子对象
+ 深拷贝（deepcopy）：完全拷贝父对象及其子对象

```python
b = a：赋值引用，a和b都指向同一个对象
```

![img](https://images2018.cnblogs.com/blog/1186367/201804/1186367-20180414174652446-1871395674.png)  

```python
b = a.copy()：浅拷贝，a和b都是一个独立的对象，但它们的子对象是指向同一对象（是引用）
```

![img](https://images2018.cnblogs.com/blog/1186367/201804/1186367-20180414174922385-1333155603.png)  

```python
b = copy.deepcopy(a)：深拷贝，a和b完全拷贝了父对象和子对象，两者是完全独立的
```

![img](https://images2018.cnblogs.com/blog/1186367/201804/1186367-20180414175130913-861637706.png)  

```所以d=[1, 2, 3, 4, ['a', 'b']] ，a和d是完全独立的，a的变化不会引起d的变化```  


4.what gets printed?（）  

```python
kvps = { '1' : 1, '2' : 2 }
theCopy = kvps.copy()
kvps['1'] = 5
sum = kvps['1'] + theCopy['1']
print sum
```

A. 1  
B. 2  
C. 6  
D. 10  
E. An exception is thrown  

**答案**: C

**分析**

```copy是浅拷贝，因为kvps和theCopy的父对象是相互独立的，子对象是指向同一对象的，因此父对象kvps['1'] = 5, theCopy['1'] = 1。这里的子对象指的是父对象中的二级对象```  

5.下列表达式的值为True的是（）  

A. 5 + 4j > 2 - 3j  
B. 3 > 2 > 2  
C. (3, 2) < ('a', 'b')  
D. 'abc' > 'xyz'  

**答案**： C（在Python2中，Python3中没有答案）

**分析**

```python
A：无论是在Python2还是Python3，复数都不支持比较大小，Python3会抛出以下错误：
TypeError: '>' not supported between instances of 'complex' and 'complex'

B：3 > 2 > 2，Python2和Python3都支持连续比较，相当于 3 > 2 and 2 > 2，后一个判断式为False，所以整个表达式都为False

C：Python2中支持数字和字符串之间的比较，而Python3不支持，会报以下错误：
TypeError: '<' not supported between instances of 'int' and 'str'

tuple的比较是从两者的第一个元素的ASCII码开始，直到两个元素不相等为止，若前面元素都相等，则元素个数多的tuple较大
(1,9) < (2,3) # True
(8,9) < (1,2,3) # False
(1,2,3) == (1,2,3) # True
(3,2) < ('a','b') # True

D：字符串的比较与tuple类似，也是从第一个字符开始比较ASCII码，直到两个字符不相等为止，字母与数字的ASCII码大小范围是"a-z">"A-Z">"0-9"，D中"a"<"x"，因此为False
```

6.以下程序要求用户输入二进制数字0/1并显示之，请指出程序中代码第几行存在错误（）
```python
1	.bit = input("Enter a binary digit:")
2.	if bit = 0 or 1:
3.    print "your input is" ,bit
4.	else
5.    print "your input is invalid"
```

A. 4  
B. 5  
C. 3  
D. 2  

**答案**: A D

**分析**
```python
在Python2中
	input()只能接收"数字"的输入，在对待纯数字输入时具有自己的特性，它返回所输入的数字的类型(int，float)
	raw_input()将所有输入作为字符串看待，返回字符串类型
	
在Python3中
	只有input()，其接收任意输入，将所有输入默认为字符串处理，并返回字符串类型
	
	
本题中根据print来看是Python2

第2行应该是 if bit == 0 or bit == 1:
千万不能写成：if bit == 0 or 1: #相当于 if (bit == 0) or 1:这个语句不管bit为何值，都恒为真

第4行else少了: 
```






















```

```