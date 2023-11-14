[toc]



# Docker Compose



## 一. 为什么要用 Docker Compose

当我们有N多台容器或者应用需要启动的时候，如果手动去操作，是非常耗费时间且容易出错的，这种情况下推荐使用 Docker Compose，只需要一个配置文件就可以帮我们搞定

但是 Docker Compose 只能管理当前主机上的 Docker，不能去管理其他服务器上的服务，只能作用于单机环境

## 二. 什么是 Docker Compose 

Docker Compose 是基于docker的开源项目，托管于github上，由python实现，调用 docker服务的API负责实现对docker容器集群的快速编排，即通过一个单独的yaml文件，来定义一组相关的容器来为一个项目服务

所以，Docker Compose 默认的管理对象是项目，通过子命令的方式对项目中的一组容器进行生命周期的管理

## 三. 怎么使用 Docker Compose

### 1. Docker Compose的安装

本文简单介绍两种安装 Docker Compose 的方式，第一种方式相对简单，但是由于网络问题，常常安装不上，并且经常会断开，第二种方式略微麻烦，但是安装过程比较稳定。

#### (1). github下载安装

```
curl -L https://github.com/docker/compose/releases/download/1.24.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose 

chmod +x /usr/local/bin/docker-compose

docker-compose --version
```

#### (2). python pip 安装

```
yum -y install epel-release python3-pip gcc python-devel -y  
pip3 install docker-compose

docker-compose version
```

### 2. Docker Compose 的YAML文件

#### (1). 简介

Docker Compose 允许用户通过一个docker-compose.yml模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目

Docker Compose 模板文件是一个定义服务、网络和卷的YAML文件。模板文件默认路径是当前目录下的docker-compose.yml，可以使用.yml或.yaml作为文件扩展名

Docker Compose 标准模板文件应该包含version、services、networks 三大部分，最关键的是services和networks两个部分

```
version: '2'

services:

  web:

    image: dockercloud/hello-world

    ports:

      - 8080

    networks:

      - front-tier

      - back-tier

 

  redis:

    image: redis

    links:

      - web

    networks:

      - back-tier

 

  lb:

    image: dockercloud/haproxy

    ports:

      - 80:80

    links:

      - web

    networks:

      - front-tier

      - back-tier

    volumes:

      - /var/run/docker.sock:/var/run/docker.sock 

 

networks:

  front-tier:

    driver: bridge

  back-tier:

    driver: bridge
```



#### (2). version

Compose目前有三个版本分别为Version 1，Version 2，Version 3，Compose区分Version 1和Version 2（Compose 1.6.0+，Docker Engine 1.10.0+）。Version 2支持更多的指令。Version 1将来会被弃用

```
version: '2'
```



#### (3). image

image是指定服务的镜像名称或镜像ID。如果镜像在本地不存在，Docker Compose 将会尝试拉取镜像

```
services: 

    web: 

        image: hello-world
```



#### (4). build

服务除了可以基于指定的镜像，还可以基于一份Dockerfile，在使用up启动时执行构建任务，构建标签是build，可以指定Dockerfile所在文件夹的路径。Docker Compose 将会利用Dockerfile自动构建镜像，然后使用镜像启动服务容器

context 为上下文路径

```
build:

  context: .

  dockerfile: path/of/Dockerfile
```



#### (5). container_name

Docker Compose 的容器名称格式是：<项目名称><服务名称><序号>

可以自定义项目名称、服务名称，但如果想完全控制容器的命名，可以使用标签指定：

```
container_name: api_auto_testing
```



#### (6). port

ports用于映射端口的标签

使用 HOST:CONTAINER 格式或者只是指定容器的端口，宿主机会随机映射端口

```
services:
  web:
    build: .
    ports:
     - "5000"
     - "5000:5000"
```



#### (7). volumes

挂载一个目录或者一个已存在的数据卷容器，可以直接使用 HOST:CONTAINER 格式，或者使用 HOST:CONTAINER:ro 格式，后者对于容器来说，数据卷是只读的，可以有效保护宿主机的文件系统

Compose 的数据卷指定路径可以是相对路径，使用 . 或者 .. 来指定相对目录

数据卷的格式可以是下面多种形式

```
volumes:

  // 只是指定一个路径，Docker 会自动在创建一个数据卷（这个路径是容器内部的）

  - /var/lib/mysql

  // 使用绝对路径挂载数据卷

  - /opt/data:/var/lib/mysql

  // 以 Compose 配置文件为中心的相对路径作为数据卷挂载到容器

  - ./cache:/tmp/cache

  // 使用用户的相对路径（~/ 表示的目录是 /home/<用户目录>/ 或者 /root/）

  - ~/configs:/etc/configs/:ro

  // 已经存在的命名的数据卷

  - datavolume:/var/lib/mysql
```



#### (8). env_file

在docker-compose.yml中可以定义一个专门存放变量的文件

如果通过docker-compose -f FILE指定配置文件，则env_file中路径会使用配置文件路径

如果有变量名称与environment指令冲突，则以后者为准。格式如下：env_file: .env

或者根据docker-compose.yml设置多个(如果在配置文件中有build操作，变量并不会进入构建过程中)

```
env_file:

  - ./common.env

  - ./apps/web.env

  - /opt/secrets.env
```



#### (9). expose

暴露端口，但不映射到宿主机，只允许能被连接的服务访问。仅可以指定内部端口为参数，如下所示：

```
expose:

    - "8000"
```



#### (10). command

使用command可以覆盖容器启动后默认执行的命令

```
command: ls -l
```



#### (11). entrypoint 

容器启动时的命令，覆盖Dockerfile中的定义

```
entrypoint: /code/entrypoint.sh
```



#### (12). depends_on

在使用 Compose 时，最大的好处就是少打启动命令，但一般项目容器启动的顺序是有要求的，如果直接从上到下启动容器，必然会因为容器依赖问题而启动失败。例如在没启动数据库容器的时候启动应用容器，应用容器会因为找不到数据库而退出。depends_on标签用于解决容器的依赖、启动先后的问题

```
version: '2'

services:

  web:

    build: .

    depends_on:

      - db

      - redis

  redis:

    image: redis

  db:

    image: postgres
```

上述YAML文件定义的容器会先启动redis和db两个服务，最后才启动web 服务



### 3. Docker Compose 常用命令

#### (1). 数据准备

##### 创建工程目录

创建一个测试目录：

```
mkdir compose_test
cd compose_test
```

##### 创建app.py

在测试目录中创建一个名为 app.py 的文件，并复制粘贴以下内容：

```
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)


def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)


@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```

在此示例中，redis 是应用程序网络上的 redis 容器的主机名，该主机使用的端口为 6379

##### 创建requirements.txt

在 composetest 目录中创建另一个名为 **requirements.txt** 的文件，内容如下：

```
flask
redis
```

##### 创建Dockerfile

在 compose_test 目录中，创建一个名为 **Dockerfile** 的文件，内容如下：

```
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP app.py
ENV FLASK_RUN_HOST 0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
COPY . .
CMD ["flask", "run"]
```

- **FROM python:3.7-alpine**: 从 Python 3.7 映像开始构建镜像

- **WORKDIR /code**: 将工作目录设置为 /code

- ```
  ENV FLASK_APP app.py
  ENV FLASK_RUN_HOST 0.0.0.0
  ```

  设置 flask 命令使用的环境变量

- **RUN apk add --no-cache gcc musl-dev linux-headers**: 安装 gcc，以便诸如 MarkupSafe 和 SQLAlchemy 之类的 Python 包可以编译加速

- ```
  COPY requirements.txt requirements.txt
  RUN pip install -r requirements.txt
  ```

  复制 requirements.txt 并安装 Python 依赖项

- **COPY . .**: 将 . 项目中的当前目录复制到 . 镜像中的工作目录

- **CMD ["flask", "run"]**: 容器提供默认的执行命令为：flask run

##### 创建docker-compose.yml

```
# yaml 配置
version: '3'
services:
  web:
    build: .
    ports:
     - "5000:5000"
  redis:
    image: "redis:alpine"
```

该 Compose 文件定义了两个服务：web 和 redis

- **web**：该 web 服务使用从 Dockerfile 当前目录中构建的镜像。然后，它将容器和主机绑定到暴露的端口 5000。此示例服务使用 Flask Web 服务器的默认端口 5000 
- **redis**：该 redis 服务使用 Docker Hub 的公共 Redis 映像

#### 项目整体结构

![image-20231114222840313](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311142228385.png)

![image-20231114222924484](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311142229568.png)



#### (2). docker-compose 命令格式

```
docker-compose [-f <arg>...] [options] [COMMAND] [ARGS...] 

命令选项如下:
-f --file FILE指定Compose模板文件，默认为docker-compose.yml
-p --project-name NAME 指定项目名称，默认使用当前所在目录为项目名
--verbose  输出更多调试信息
-v，-version 打印版本并退出
--log-level LEVEL 定义日志等级(DEBUG, INFO, WARNING, ERROR, CRITICAL)
```



#### (3). docker-compose up

> 启动容器

```
docker-compose up [options] [--scale SERVICE=NUM...] [SERVICE...]

选项包括：
-d 在后台运行服务容器
–build 在启动容器前构建服务镜像
-t, –timeout TIMEOUT 停止容器时候的超时（默认为10秒）
–-scale SERVICE=NUM 设置服务运行容器的个数，将覆盖在compose中通过scale指定的参数
```



##### 例1: 启动所有服务

![image-20231114230848649](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311142308755.png)






docker-compose up -d

在后台所有启动服务

-f 指定使用的Compose模板文件，默认为docker-compose.yml，可以多次指定。

docker-compose -f docker-compose.yml up -d

#### (3). docker-compose ps

> 列出项目中目前的所有容器

```
docker-compose ps [options] [SERVICE...]
```





#### (4). docker-compose port

> 显示某个容器端口所映射的公共端口

```
docker-compose port [options] SERVICE PRIVATE_PORT

选项包括：

–protocol=proto，指定端口协议，TCP（默认值）或者UDP
–index=index，如果同意服务存在多个容器，指定命令对象容器的序号（默认为1）
```



#### (5). docker-compose down

> 停止和删除容器、网络、卷、镜像

```
docker-compose down [options]

选项包括：

–rmi type，删除镜像，类型必须是：all，删除compose文件中定义的所有镜像；local，删除镜像名为空的镜像
-v, –volumes，删除已经在compose文件中定义的和匿名的附在容器上的数据卷
–remove-orphans，删除服务中没有在compose中定义的容器
```







#### (6). docker-compose logs

> 查看服务容器的输出。默认情况下，docker-compose将对不同的服务输出使用不同的颜色来区分。可以通过–no-color来关闭颜色

```
docker-compose logs [options] [SERVICE...]
```





#### (7). docker-compose build

> 构建（重新构建）项目中的服务容器

```
docker-compose build [options] [--build-arg key=val...] [SERVICE...]

选项包括：

–compress 通过gzip压缩构建上下环境
–force-rm 删除构建过程中的临时容器
–no-cache 构建镜像过程中不使用缓存
–pull 始终尝试通过拉取操作来获取更新版本的镜像
-m, –memory MEM为构建的容器设置内存大小
–build-arg key=val为服务设置build-time变量

服务容器一旦构建后，将会带上一个标记名。可以随时在项目目录下运行docker-compose build来重新构建服务
```



#### (8). docker-compose pull

> 拉取服务依赖的镜像

```
docker-compose pull [options] [SERVICE...]

选项包括：

–ignore-pull-failures，忽略拉取镜像过程中的错误

–parallel，多个镜像同时拉取

–quiet，拉取镜像过程中不打印进度信息
```



#### (9). docker-compose push

> 推送服务依的镜像

```
docker-compose push [options] [SERVICE...]

选项包括：

-ignore-push-failures 忽略推送镜像过程中的错误
```



#### (10). docker-compose create

> 为服务创建容器

```
docker-compose create [options] [SERVICE...]

选项包括：

–force-recreate：重新创建容器，即使配置和镜像没有改变，不兼容–no-recreate参数
–no-recreate：如果容器已经存在，不需要重新创建，不兼容–force-recreate参数
–no-build：不创建镜像，即使缺失
–build：创建容器前，生成镜像
```



#### (11). docker-compose start

> 启动已经存在的服务容器

```
docker-compose start [SERVICE...]
```





#### (12). docker-compose restart

> 重启项目中的服务

```
docker-compose restart [options] [SERVICE...]

选项包括：

-t, –timeout TIMEOUT，指定重启前停止容器的超时（默认为10秒）
```





#### (13). docker-compose run

> 在指定服务上执行一个命令

```
docker-compose run [options] [-v VOLUME...] [-p PORT...] [-e KEY=VAL...] SERVICE [COMMAND] [ARGS...]
```





#### (14). docker-compose exec

> 在正在运行的容器中执行命令

```
docker-compose exec [options] SERVICE COMMAND [ARGS...]

选项包括：

-d 分离模式，后台运行命令
–privileged 获取特权
–user USER 指定运行的用户
-T 禁用分配TTY，默认docker-compose exec分配TTY
–index=index，当一个服务拥有多个容器时，可通过该参数登陆到该服务下的任何服务，例如：docker-compose exec –index=1 web /bin/bash ，web服务中包含多个容器
```





#### (15). docker-compose pause

> 暂停一个服务容器

```
docker-compose pause [SERVICE...]
```





#### (16). docker-compose unpause

> 恢复处于暂停状态中的服务

```
docker-compose unpause [SERVICE...]
```





#### (17). docker-compose stop

> 停止正在运行的容器，可以通过docker-compose start 再次启动

```
docker-compose stop [options] [SERVICE...]

选项包括：
-t, –timeout TIMEOUT 停止容器时候的超时（默认为10秒）
```





#### (18). docker-compose rm

> 删除所有（停止状态的）服务容器。推荐先执行docker-compose stop命令来停止容器

```
docker-compose rm [options] [SERVICE...]

选项包括：
–f, –force，强制直接删除，包括非停止状态的容器
-v，删除容器所挂载的数据卷
```



#### (19). docker-compose version

> 打印版本信息

![image-20231114230440367](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311142304181.png)



#### (20). docker-compose -h

> 查看帮助

![image-20231114230611768](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311142306868.png)



