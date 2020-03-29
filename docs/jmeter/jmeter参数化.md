## 快速导航
- [```用户自定义的变量和用户参数的区别```](#用户自定义变量和用户参数的区别)```
- [```接口关联之正则提取和JSON提取```](#接口关联之正则提取和JSON提取)```
- [```跨线程组取参数值的两个方法```](#跨线程组取参数值的两个方法)```




## 用户自定义的变量和用户参数的区别

用户自定义的变量和用户参数，当都在做参数化的时候，同一个线程组，只有一个线程数，且运行一次时，区别不大。```主要的区别在于同一个线程组，多个线程数运行一次或者一个线程数运行多次时，用户自定义的变量值不变，而用户参数的值每次都在动态变化```。以注册接口为例，注册接口的请求参数如下：  

```javascript
{"mobile":"15800000001","password":"123456","code":"3367","platform":"w
indows","username":"test11","sex":1,"age":20,"email":"158000000011@test.com"}
```

注册接口要求mobile符合号段规则且不能重复，而其他参数像password、flatform、code有值就行，其他参数可传可不传，因此对于该接口，需要多次运行时，就要动态的生成mobile   

### 使用用户自定义的变量

在用户定义的变量里填入变量名mobile，变量值：${__Random(13700000000,13799999999,)}， 表示生成以137开头的长度为11位的手机号   

![image-20200328193643807](https://i.loli.net/2020/03/29/epITrW7oX2PhgBx.png)  

用${mobile}参数化请求数据中的mobile参数  

![image-20200328194012333](https://i.loli.net/2020/03/29/89ipQgXsPSFe21x.png)  

设置循环五次，查看调试取样器，发现每次的手机号都相同，所以响应结果除了第一次注册成功外，其他都提示用户已存在  

![image-20200329130557465](https://i.loli.net/2020/03/29/ZLWegNKib6jRUT8.png)  

![image-20200329130715773](https://i.loli.net/2020/03/29/Z1ltHNVfxqyAJ9M.png)  

![image-20200329130803063](https://i.loli.net/2020/03/29/QfNFzEJUqbsyWIj.png)  

![image-20200329130852443](https://i.loli.net/2020/03/29/JdgzYBRj1pw7sMn.png)  

### 使用用户参数

更改为用户参数，把mobile参数化，然后运行，发现值每次动态变化   

![image-20200329131111380](https://i.loli.net/2020/03/29/DoKRfeQ6vxElTcs.png)  

![image-20200329131151776](https://i.loli.net/2020/03/29/nC4GA3TiYoSUu2y.png)  

![image-20200329131225979](https://i.loli.net/2020/03/29/OlbkUmP4npe8jfc.png)  


## 接口关联之正则提取和JSON提取

用户不仅需要注册，注册后还需要进行登录操作，这时候根据接口文档，注册需要依赖登录返回的gqid，这时候就要用到关联了  

### 正则提取

我们要从注册的响应结果```{"code":0,"msg":"成功调用","data":{"id":272,"username":"test11","sex":1,"age":20,"mobile":"13741079532","email":"158000000011@test.com","gqid":"4000031","money":0.0,"pmoney":100.0,"createtime":1577035125244,"lasttime":1577035125244,"token":"6aOwpa5HXV+co8gCnPqKpKcPntjUYso9efAeeJnHTw2gCJ6a+JsrH1zdffO2OKtnxRU/jxaEwo/fQbjJJq9BrA==","identity":"fffa1e1412d9d9b0"}}```里提取出gqid，然后传递给登录接口，需要新建一个正则提取器，提取器的配置和表示的含义如下：  

![image-20200329135354652](https://i.loli.net/2020/03/29/h9BwAkSEgye7MCT.png)  

在登录接口中，使用${gqid}表示gqid的参数化，${mobile}表示mobile的参数化，所有注册接口的密码我们都设置为123456  

![image-20200329135610983](https://i.loli.net/2020/03/29/cbsXVh23IPiQg4Y.png)  

运行一下，发现注册登录都成功了，在调试取样器中可以看到参数gqid和mobile的值

![image-20200329135904128](https://i.loli.net/2020/03/29/5gskd2xawbRltTH.png)  

![image-20200329135944475](https://i.loli.net/2020/03/29/3LxAbOS65JlRfB7.png)  

![image-20200329140031196](https://i.loli.net/2020/03/29/Svm6FQgZMurjNk2.png)  


### JSON提取

JSON提取前在查看结果树里我们可以测试下，JSON提取的表达式正确与否。JSON提取表达式以$.开头  

![image-20200329140616703](https://i.loli.net/2020/03/29/uK91bYvEfLjqeRn.png)  

然后将提取表达式填入JSON提取器中，将登陆接口参数化，查看响应结果  

![image-20200329140839349](https://i.loli.net/2020/03/29/iBahe86nJplT4sZ.png)  

![image-20200329140929172](https://i.loli.net/2020/03/29/iSk3KQyLwD5RPzE.png)  

![image-20200329140956481](https://i.loli.net/2020/03/29/cHTev7CfDPjNZkO.png)  

```注意一个问题，同一个线程组下，如果注册接口和登陆接口使用同一个参数mobile的调用${mobile}，请使用用户定义的参数而非用户参数，因为用户定义的参数值运行过程中不变，能够保证注册接口和登录接口随机生成的是同一个手机号，而用户参数恰恰相反```  


## 跨线程组取参数值的两个方法

同样是上面的例子，如果注册接口和登录接口是两个线程组，跨线程组参数应该怎么传递呢？

### 定义属性法 

jmeter中属性是全局的，可以动态设置，而参数（变量）是单独属于每个线程的  

第一步：在jmeter中添加两个线程组，一个是注册，一个是登录  

第二步：在注册的线程组中，使用正则提取gqid，将其命名为gqid参数  

![image-20200329150837059](https://i.loli.net/2020/03/29/pyerq6C2dfDIZW1.png)  

第三步：在注册的线程组中，添加一个Beanshell后置处理器，然后在函数助手对话框中，```选择__setProperty()函数，函数的第一个值表示将要存放的属性名称，我们设置为gqidProperty，第二只输入第二步中定义的参数的调用${gqid}```   

![image-20200329150916906](https://i.loli.net/2020/03/29/FXuRJLNcxCHerG8.png)  

第四步：在登录的线程组中，将参数gqid参数化表示  

![image-20200329151047400](https://i.loli.net/2020/03/29/eGlTwE9u7PxI418.png)  

运行后，会发现一个问题，就是gqid并没有真正被替换，这个问题困扰了我很久，才发现是线程组执行顺序的问题  

![image-20200329151228975](https://i.loli.net/2020/03/29/HW7RwS189GAMZna.png)  

从日志里分析，```这两个线程组并不是按照注册先执行，登录后执行的顺序，而是注册先启动，接着登录启动，登录执行完了，注册才结束，也就是存在一种可能，${gqid}还没拿到值就去调用${gqidProperty}，所以参数才没有被替换```   

![image-20200329151547196](https://i.loli.net/2020/03/29/xwOInVbEUm3dl1g.png)  

我们需要更改的地方是测试计划，勾上独立的运行每一个线程组   

![image-20200329151714008](https://i.loli.net/2020/03/29/FY7vb2xmTDQpA9e.png)  

再次运行，结果正常，可以看到gqid参数被真正替换了   

![image-20200329151842156](https://i.loli.net/2020/03/29/wVPUfb8vyXAKOeF.png)  

### 文件转接法

将注册线程组的运行结果，存储到文件，登录线程组通过csv读取文件，然后使用正则提取文件中需要的值，作为参数输入  

第一步：在jmeter中，添加两个线程组，一个注册，一个登录  
第二步：在注册线程组中，添加监视器—>保存响应到文件，设置文件名称前缀  

![image-20200329155258302](https://i.loli.net/2020/03/29/PiOu5MbKwAxEBFk.png)

