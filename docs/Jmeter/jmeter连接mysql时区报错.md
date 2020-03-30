## jmeter连接mysql报错：Cannot create PoolableConnectionFactory (The server time zone value...

![image-20200330171428583](https://i.loli.net/2020/03/30/u4T7xs2UiMabEgX.png)  

服务器时区设置有问题，在JDBC Connection Configuration下面的Database URL后面加上```&serverTimezone=UTC```   

```mysql
jdbc:mysql://localhost:3306/futureloan?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC
```