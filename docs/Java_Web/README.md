[toc]



# Java Web



## Tomcat-新建项目-部署-运行-访问



### Tomcat的安装与配置



1. 解压：不要有中文，不要有空格

2. 目录结构说明：

   bin  可执行文件目录

   conf  配置文件目录

   lib  存放lib的目录

   logs  日志文件目录

   webapps   项目部署的目录

   work  工作目录

   temp  临时目录

   

3. 配置环境变量，让tomcat能够运行

   因为tomcat也是用java和c来写的，因此需要JRE，所以需要配置JAVA_HOME

   在catalina.bat最前面设置JAVA_HOME和JRE_HOME

   ```java
   set JAVA_HOME=D:\program\Java\jdk1.8.0_171
   set JRE_HOME=D:\program\Java\jdk1.8.0_171\jre
   ```

   

4. 启动tomcat，然后访问主页

   ```java
   localhost:8080
   ```

### 新建web项目，并在Tomcat中部署

在webapps目录下新建项目目录baidu（对应的是context root)，在项目目录下新建WEB-INF(名字不能改)，然后在WEB-INFO同级目录下新建一个test.html

![image-20220219113022031](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220219113022031.png)

在浏览器访问（这里修改了端口号为8081）

![image-20220219113107423](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220219113107423.png)



## 在IDEA下新建javaweb项目-部署-运行



### 新建项目

![image-20220219114116813](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220219114116813.png)



### 部署配置

![image-20220219114548408](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220219114548408.png)

![image-20220219114729962](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220219114729962.png)

![image-20220219114821881](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220219114821881.png)

加入改为/pro07

![image-20220219114905950](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220219114905950.png)

![image-20220219115035887](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220219115035887.png)

IDEA实际上部署的目录是```E:\java_workshop\pro07-javaweb-begin\out\artifacts\pro07_javaweb_begin_war_exploded```，而不是tomcat的webapp目录，这点比较特殊

![image-20220219115739713](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220219115739713.png)

### 运行

![image-20220219115553502](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220219115553502.png)

![image-20220219115616498](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220219115616498.png)



## Servlet入门—获取参数

### 交互逻辑

1. 用户发请求，action=add   
2. 项目中，web.xml中找到url-pattern=/add      ->第13行
3. 找第12行的servlet-name=AddServlet 
4. 找和servlet-mapping中servlet-name一致的servlet   ->第8行
5. 找第9行的servlet-class  ->com.atguigu.servlets.AddServlet
6. 用户发送的是post请求(method=post)，因此tomcat会执行AddServlet中的doPost方法





### AddServlet的作用

1. 获取用户（客户端）发给服务器的数据
2. 调用DAO中的方法完成添加功能
3. 在控制台打印添加成功





### 编写AddServlet

1. 在src下创建com.atguigu.servlets.AddServlet.java类
2. 注意HttpServlet是Tomcat下的类，因此需要在IDEA中导入Tomcat依赖

```java
package com.atguigu.servlets;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * Created by Beck on 2022/2/19.
 */
public class AddServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String fname = request.getParameter("fname");
        //从request对象获取到的参数只能是字符串类型
        String priceStr = request.getParameter("price");
        Integer price = Integer.parseInt(priceStr);
        String fcountStr = request.getParameter("fcount");
        Integer fcount = Integer.parseInt(fcountStr);
        String remark = request.getParameter("remark");

        System.out.println("fname = " + fname);
        System.out.println("price = " + price);
        System.out.println("fcount = " + fcount);
        System.out.println("remark = " + remark);
    }
}
```

![image-20220219142921856](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220219142921856.png)

![image-20220219142947650](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220219142947650.png)

![image-20220219143016325](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220219143016325.png)

![image-20220219143049826](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220219143049826.png)

com.atguigu.servlets.AddServlet.java

```java
package com.atguigu.servlets;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * Created by Beck on 2022/2/19.
 */
public class AddServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String fname = request.getParameter("fname");
        //从request对象获取到的参数只能是字符串类型
        String priceStr = request.getParameter("price");
        Integer price = Integer.parseInt(priceStr);
        String fcountStr = request.getParameter("fcount");
        Integer fcount = Integer.parseInt(fcountStr);
        String remark = request.getParameter("remark");

        System.out.println("fname = " + fname);
        System.out.println("price = " + price);
        System.out.println("fcount = " + fcount);
        System.out.println("remark = " + remark);
    }
}
```



### 配置add与AddServlet的对应关系

在WEB-INF下的web.xml中配置

```java
    <servlet>
        <servlet-name>AddServlet</servlet-name>
        <servlet-class>com.atguigu.servlets.AddServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>AddServlet</servlet-name>
        <url-pattern>/add</url-pattern>
    </servlet-mapping>
```

![image-20220219144223061](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220219144223061.png)

1. 用户发请求，action=add   
2. 项目中，web.xml中找到url-pattern=/add      ->第13行
3. 找第12行的servlet-name=AddServlet 
4. 找和servlet-mapping中servlet-name一致的servlet   ->第8行
5. 找第9行的servlet-class  ->com.atguigu.servlets.AddServlet
6. 用户发送的是post请求(method=post)，因此tomcat会执行AddServlet中的doPost方法



### 运行

![image-20220219170433700](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220219170433700.png)

![image-20220219170501676](http://becktuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220219170501676.png)