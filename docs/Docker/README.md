[toc]

# 第一本Docker书

## Dockerfile

### 1.CMD

CMD指令：用于指定一个容器启动时要运行的命令

RUN指令：指定镜像被构建时要运行的命令

```linux
docker run -i -t beck/static_web /bin/true
```

上述的代码与```CMD ["/bin/true"]```是等价的

另外需要注意，docker run命令可以覆盖CMD指令。如Dockerfile中使用CMD指令

```linux
CMD ["/bin/bash"]
```

在命令行运行docker run，看到的结果是列出正在进行的进程列表

![image-20230405131526420](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202304051315633.png)

### 2.ENTRYPOINT

ENTRYPOINT指令提供的命令不会在启动容器时被覆盖

```linux
方式一：不带参数
ENTRYPOINT ["/usr/sbin/nginx"]
方式二：带参数
ENTRYPOINT ["/usr/sbin/nginx", "-g", "daemon off;"]
```

将Dockerfile设置为方式一，重新构建镜像

```linux
docker build -t="beck01/static_web" .
```

使用docker run命令启动包含ENTRYPOINT指令的容器，这里指定了-g "daemon off;"参数，这个参数会传递给用ENTRYPOINT指定的命令，在这里该命令为```/usr/sbin/nginx -g "daemon off;"```，该命令会以前台运行的方式启动Nginx守护进程，此时这个容器就会作为一台web服务器来运行

```linux
docker run -t -i beck01/static_web -g "daemon off;"
```

### 3.WORKDIR

可以通过-w标志在运行时覆盖工作目录，比如Dockerfile中设置的工作目录为/opt/webapp/db，这时候使用docker run -w将容器内的工作目录设置为/var/log

```linux
WORKDOR /opt/webapp/db
RUN bundle install
```

```linux
docker run -ti -w /var/log ubuntu pwd
```

### 4.ADD

在ADD文件时，Docker通过目的地址参数末尾的字符来判断文件源是目录还是文件，如果目标地址以/结尾，那么Docker就认为源位置指向的是一个目录。如果目的地址以/结尾，那么Docker就认为源位置指向的是目录。如果目的地址不是以/结尾，那么Docker就认为源位置指向的是文件

```linux
ADD software.lic /opt/application/software.lic
```

最后值得一提的是，ADD在处理本地归档文件(tar archive)时还有一些小魔法。如果将一个归档文件(合法的归档文件包括gzip、bzip2、xz)指定为源文件，Docker会自动将归档文件解开(unpack)

```linux
ADD latest.tar.gz /var/www/wordpress/
```

这条命令会将归档文件latest.tar.gz解开到/var/www/wordpress/目录下。最后，如果目的位置不存在的话，Docker将会为我们创建这个全路径



### 5.COPY

COPY指令只关心在构建上下文复制本地文件，而不会去做文件提取和解压的工作



## 在测试中使用Docker

### 1.使用Docker测试静态网站

![image-20230406221654351](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202304062217137.png)

代码清单：global.conf

```linux
server {
        listen          0.0.0.0:80;
        server_name     _;

        root            /var/www/html/website;
        index           index.html index.htm;

        access_log      /var/log/nginx/default_access.log;
        error_log       /var/log/nginx/default_error.log;
}
```

代码清单：nginx.conf

在这个配置文件里，daemon off; 选项阻止Nginx进入后台，强制其在前台运行。这是因为想要保持Docker容器的活跃状态，需要其中运行的进程不能中断。默认情况下，Nginx会以守护进程的方式启动，这会导致容器只是暂时运行，在守护进程被fork启动后，发起守护进程的原始进程就会退出，这时容器就停止运行了

这个文件通过ADD命令复制到/etc/nginx/nginx.conf

读者应该注意到了两个ADD指令的目标有细微的差别。第一个指令以目录/etc/nginx/conf.d/结束，而第二个指令指定了文件/etc/nginx/nginx.conf。将文件复制到Docker镜像时，这两种风格都是可以用的

```linux
user www-data;
worker_processes 4;
pid /run/nginx.pid;
daemon off;

events {  }

http {
  sendfile on;
  tcp_nopush on;
  tcp_nodelay on;
  keepalive_timeout 65;
  types_hash_max_size 2048;
  include /etc/nginx/mime.types;
  default_type application/octet-stream;
  access_log /var/log/nginx/access.log;
  error_log /var/log/nginx/error.log;
  gzip on;
  gzip_disable "msie6";
  include /etc/nginx/conf.d/*.conf;
}
```

代码清单：Dockerfile

```linux
FROM ubuntu:14.04
LABEL name="beck" email="1069966476@qq.com"
ENV PEFRESHED_AT 2023-04
RUN apt-get -yqq update && apt-get -yqq install nginx
RUN mkdir -p /var/www/html/website
ADD nginx/global.conf /etc/nginx/conf.d/
ADD nginx/nginx.conf /etc/nginx/nginx.conf 
EXPOSE 80
```

代码清单：index.html

```linux
<head>

    <title>Test website</title>
    
    </head>
    
    <body>
    
    <h1>This is a test website</h1>
    
</body>
```

构建新的Nginx镜像

```linux
docker build -t beck01/nginx .
```

构建第一个Nginx测试容器

```linux
docker run -d -p 80 --name website -v $PWD/website:/var/www/html/website beck01/nginx nginx
```

可以看到，在执行docker run时传入了nginx作为容器的启动命令。比较好奇这个$PWD

-v允许我们将宿主机的目录作为卷，挂载到容器里。卷在Docker里非常重要，也很有用。卷是在一个或者多个容器内被选定的目录，可以绕过分层的联合文件系统，为Docker提供持久数据或共享数据。**这意味着对卷的修改会直接生效，并绕过镜像。当提交或者创建镜像时，卷不被包含在镜像里**。

![image-20230406222938433](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202304062229493.png)

也可以通过在目录后面加上rw或者ro来指定容器内目录的读写状态，如代码清单所示

代码清单：控制卷的写状态

```linux
docker run -d -p 80 --name website -v $PWD/website:/var/www/html/website:ro beck01/nginx nginx
```

