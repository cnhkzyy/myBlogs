### 2020年3月22日
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
