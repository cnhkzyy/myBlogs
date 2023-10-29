[toc]



# 每天5分钟玩转Docker容器技术



![img](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291248666.png)

## 第1章 容器生态系统

### 1. 容器核心技术

容器核心技术是指能够让Container在host上运行起来的那些技术

![image-20231029123724940](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291237597.png)

#### (1) 容器规范

> 容器不光是Docker，还有其他容器，比如CoreOS的rkt
>
> 保证容器的可移植性和互操作性：使不同组织不同厂商开发的容器能够在不同的runtime上运行

目前有两个规范：runtime spec 和 image format spec

![image-20231029131612707](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291316798.png)

#### (2) 容器runtime

> 容器真正运行的地方，为容器提供运行环境，类似Java中的JVM

三种主流：lxc、runc、rkt

![image-20231029131705306](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291317431.png)

lxc是Linux上老牌的容器runtime。Docker最初也是用lxc作为runtime

runc是Docker自己开发的容器runtime，符合oci规范，也是现在Docker的默认runtime

rkt是CoreOS开发的容器runtime

#### (3) 容器管理工具

lxd是lxc对应的管理工具

runc的管理工具是docker engine。docker engine包含后台deamon和cli两部分。我们通常提到的Docker，一般指的是docker engine

rkt的管理工具是rkt cli

![image-20231029131224684](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291312758.png)

#### (4) 容器定义工具

> 允许用户定义容器的内容和属性，这样容器就能够被保存、共享和重建

![image-20231029131532202](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291315251.png)

1. docker image是Docker容器的模板，runtime依据docker image创建容器
2. dockerfile是包含若干命令的文本文件，可以通过这些命令创建出docker image
3. ACI与docker image类似，只不过是有CoreOS开发的rkt容器的image格式

#### (5) Registry

> 统一存放image的仓库

![image-20231029132408803](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291324865.png)

1. 私有Registry
2. Docker Hub（https://hub.docker.com）是Docker为公众提供的托管仓库，上面有很多现成的image
3. Quay.io（https://quay.io）另一个公共托管仓

#### (6) 容器OS

> 专门运行容器的操作系统。与常规OS相比，容器OS通常体积更小，启动更快

CoreOS、Atomic、Ubuntu Core等 ![image-20231029132718300](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291327377.png)



### 2. 容器平台技术

> 容器核心技术使得容器能够在单个host上运行，而容器平台技术能够让容器作为集群在分布式环境中运行

![image-20231029133041166](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291330249.png)



#### (1) 容器编排引擎

基于容器的应用一般会采用微服务架构。在这种架构下，应用被划分为不同的组件，并以服务的形式运行在各自的容器中，通过 API 对外提供服务。为了保证应用的高可用，每个组件都可能会运行多个相同的容器。这些容器会组成集群，集群中的容器会根据业务需要被动态地创建、迁移和销毁

这样一个基于微服务架构的应用系统实际上是一个动态的可伸缩的系统。----容器编排引擎用来管理容器集群

所谓编排（orchestration），通常包括容器管理、调度、集群定义和服务发现等。通过容器编排引擎，容器被有机的组合成微服务应用，实现业务需求

1. docker swarm --- Docker 开发
2. kubernetes 是 Google 领导开发的开源容器编排引擎，同时支持 Docker 和 CoreOS 容器
3. mesos 是一个通用的集群资源调度平台，mesos 与 marathon 一起提供容器编排引擎功能

![image-20231029133306376](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291333500.png)

#### (2) 容器管理平台

> 通用的平台：通常支持多种编排引擎，为用户提供更方便的功能

Rancher、ContainerShip、Portainer

![image-20231029133636644](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291336713.png)

#### (3) 基于容器的PaaS

基于容器的 PaaS 为微服务应用开发人员和公司提供了开发、部署和管理应用的平台，使用户不必关心底层基础设施而专注于应用的开发

Deis、Flynn 和 Dokku

![image-20231029133741789](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291337881.png)

### 3. 容器支持技术

下面这些技术被用于支持基于容器的基础设施

![image-20231029133938037](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291339132.png)

#### (1) 容器网络

> 管理容器与容器，容器与其他实体之间的连通性和隔离性

docker network 是 Docker 原生的网络解决方案

第三方开源解决方案： flannel、weave 和 calico

![image-20231029134126031](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291341102.png)

#### (2) 服务发现

> 一种让 client 能够知道如何访问容器提供的服务的机制

动态变化是微服务应用的一大特点

当负载增加时，集群会自动创建新的容器；负载减小，多余的容器会被销毁。容器也会根据 host 的资源使用情况在不同 host 中迁移，容器的 IP 和端口也会随之发生变化

etcd、consul 和 zookeeper 是服务发现的典型解决方案

![image-20231029134410088](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291344169.png)



#### (3) 监控

docker ps/top/stats 是 Docker 原生的命令行监控工具。除了命令行，Docker 也提供了 stats API，用户可以通过 HTTP 请求获取容器的状态信息

sysdig、cAdvisor/Heapster 和 Weave Scope 是其他开源的容器监控方案

![](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291346546.png)



#### (4) 数据管理

保证持久化数据也能够动态迁移，是 Rex-Ray 这类数据管理工具提供的能力

![image-20231029134744812](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291347944.png)

#### (5) 日志管理

日志为问题排查和事件管理提供了重要依据，日志工具有两类

![image-20231029134905857](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291349996.png)

docker logs 是 Docker 原生的日志工具，而 logspout 对日志提供了路由功能，它可以收集不同容器的日志并转发给其他工具进行后处理

#### (6) 安全性

OpenSCAP 能够对容器镜像进行扫描，发现潜在的漏洞

![image-20231029135037046](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291350151.png)



### 4. 安装Docker

### 5. 运行第一个容器

环境就绪，马上运行第一个容器，执行命令：

```
docker run -d -p 80:80 httpd  端口映射，格式为：主机(宿主)端口:容器端口
```

结果如图所示

![image-20231029135800993](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291358096.png)

其过程可以简单地描述为：

(1) 从Docker Hub下载httpd镜像。镜像中已经安装好了 Apache HTTP Server

(2) 启动 httpd ，并将容器的80端口映射到 host 的 80 端口

[![img](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291342754.png)

 

## 第2章 容器核心知识

### 1 What — 什么是容器

> 容器是一种轻量级、可移植、自包含的软件打包技术，使应用程序可以在几乎任何地方以相同的方式运行。Container=集装箱，翻译成容器

容器和虚拟机的区别：

1. 容器：(1) 应用程序本身 (2) 依赖：比如应用程序需要的库或其他软件
2. 虚拟机：为了运行应用，除了部署应用本身及其依赖，还得安装整个操作系统
3. 另外，启动容器不需要启动整个操作系统，所以容器部署和启动速度更快，开销更小，也更容易迁移

如图所示，由于所有的容器共享同一个Host OS，这使得容器在体积上要比虚拟机小很多。另外，启动容器不需要启动整个操作系统，所以容器部署和启动速度更快，开销更小，也更容易迁移

![image-20231029140506344](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291405418.png)



### 2. Why — 为什么需要容器

#### (1) 容器解决的问题

1. 容器使软件具备了超强的可移植能力

2. 以前几乎所有的应用都采用三层架构（Presentation/Application/Data），系统部署到有限的几台物理服务器上（Web Server/Application Server/Database Server）。而今天，开发人员通常使用多种服务（比如 MQ，Cache，DB）构建和组装应用，而且应用很可能会部署到不同的环境，比如虚拟服务器，私有云和公有云

3. docker类似于集装箱，不管是香蕉还是榴莲，都放进集装箱里，不会相互影响。Container=集装箱，翻译成容器

![image-20231029141055160](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291410247.png)

#### (2) 容器的优势

1. 对于开发人员 - Build Once, Run Anywhere 。容器环境与所在的 Host 环境是隔离的

2. 对于运维人员 - Configure Once, Run Anything 



### 3. How — 容器是如何工作的

#### (1) Docker 架构

Docker的核心组件包括：

1. Docker客户端：Client
2. Docker服务器：Docker daemon
3. Docker镜像：Image
4. Registry
5. Docker容器：Container

![image-20231029141618920](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291416983.png)

Docker 采用的是 Client/Server 架构。客户端向服务器发送请求，服务器负责构建、运行和分发容器。客户端和服务器可以运行在同一个 Host 上，客户端也可以通过 socket 或 REST API 与远程的服务器通信

#### (2) Docker客户端

最常用的 Docker 客户端是 docker 命令。通过 docker 我们可以方便地在 Host 上构建和运行容器

docker支持很多子命令，后面会逐步用到

![image-20231029141744041](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291417138.png)

除了docker命令行工具，用户也可以通过 REST API 与服务器通信

#### (3) Docker 服务器

Docker daemon 是服务器组件，以 Linux 后台服务的方式运行

![image-20231029142024434](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291420575.png)

默认配置下，Docker daemon 只能响应来自本地 Host 的客户端请求。如果要允许远程客户端请求，需要在配置文件中打开 TCP 监听，步骤如下：

1. 编辑配置文件  /etc/systemd/system/multi-user.target.wants/docker.service ，在环境变量 ExecStart 后面添加  -H tcp://0.0.0.0 ，允许来自任意 IP 的客户端连接

![image-20231029142114681](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291421757.png)

2. 重启Docker daemon

```
systemctl daemon-reload
systemctl restart docker.service
```

3. 假如服务器ip为192.168.56.102，客户端在命令行里加 -H参数，即可与远程服务器通信

![image-20231029142307977](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291423043.png)

#### (4) Docker 镜像

可将 Docker 镜像看着只读模板，通过它可以创建 Docker 容器

镜像有多种生成方法：

1. 可以从无到有开始创建镜像
2. 也可以下载并使用别人创建好的现成的镜像
3. 还可以在现有镜像上创建新的镜像
4. 我们可以将镜像的内容和创建步骤描述在一个文本文件中，这个文件被称作 Dockerfile，通过执行 docker build <docker-file> 命令可以构建出 Docker 镜像

#### (5) Docker容器

Docker 容器就是 Docker 镜像的运行实例

用户可以通过 CLI（docker）或是 API 启动、停止、移动或删除容器

可以这么认为，对于应用软件，镜像是软件生命周期的构建和打包阶段，而容器则是启动和运行阶段

#### (6) Registry

Registry 是存放Docker镜像的仓库，Registry分为私有和公有两种

Docker Hub 是默认的Registry，由Docker公司维护，上面有数以万计的镜像，用户可以自由下载和使用

出于对速度和安全的考虑，用户也可以创建自己私有的Registry

docker pull 命令可以从 Registry 下载镜像

docker run 命令则是先下载镜像（如果本地没有），然后再启动容器



#### (7) 一个完整的例子

现在通过一个例子体会一下Docker的各个组件是如何协作的

容器启动过程如下：

![image-20231029143516913](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291435027.png)

1. Docker客户端执行docker run命令
2. Docker daemon 发现本地没有httpd镜像
3. daemon 从 Docker Hub 下载镜像
4. 下载完成后，镜像httpd被保存到本地
5. Docker daemon 启动容器

docker images 可以看到httpd已经下载到本地

![image-20231029143755738](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291437861.png)

docker ps 或者 docker container ls 显示容器正在运行

![image-20231029143954805](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291439904.png)

## 



```
docker pull hello-world从Docker Hub下载
docker images查看镜像信息
docker run hello-world运行
```

hello-world 的 Dockerfile



```
FROM scratch此镜像是从白手起家，从 0 开始构建
COPY hello /将文件“hello”复制到镜像的根目录。
CMD ["/hello"]
```

**3.1.2 base 镜像**

centos的镜像-标准的base镜像



```
FROM scratch
ADD centos-7-docker.tar.xz /
CMD ["/bin/bash"]
```

所谓的 base 镜像：

　　1、不依赖其他镜像，从 scratch 构建。

　　2、其他镜像可以之为基础进行扩展。

**3.1.3 镜像为什么这么小？（一个centos只有200M）**

linux操作系统由**内核空间**和**用户空间**组成

内核空间是**kernel**，linux刚启动的时候回加载**bootfs**文件系统，之后bootfs会被卸载掉

用户空间是**rootfs**，包含熟悉的/dev、/bin目录等

对于base镜像来说，底层直接用 Host 的 kernel，自己只需要提供 rootfs 就行

[![img](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291342954.png)](https://img2018.cnblogs.com/i-beta/1381066/201912/1381066-20191216104543130-1090738469.png)

 

 

**3.1.4 为什么镜像可以运行在不同的 linux 发行版本上**

不同Linux发行版的区别主要就是roots，Linux kernel差别不大。

base镜像只是在用户控件与发行版一致，kernel版本与发行版是不同的。比如centos使用3.x.x的kernel，但是Docker Host是4.x.x的话，容器的kernel版本与Host一致。

容器只能使用 Host 的 kernel，并且不能修改。所以如果容器对kernel版本有要求，则不建议用容器，虚拟机更合适。

**3.1.5 镜像的分层结构**

Docker Hub 中 99% 的镜像都是通过在 base 镜像中安装和配置需要的软件构建出来的。

新镜像是从 base 镜像一层一层叠加生成的。每安装一个软件，就在现有镜像的基础上增加一层。

最大的一个好处就是 - 共享资源

　　1、有多个镜像都从相同的 base 镜像构建而来，那么 Docker Host 只需在磁盘上保存一份 base 镜像
　　2、同时内存中也只需加载一份 base 镜像，就可以为所有容器服务了。

如果多个容器共享一份基础镜像，当某个容器修改了基础镜像的内容，比如 /etc 下的文件，其他容器下的/etc不会被修改

**3.1.6 可写的容器层**

[![img](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291342997.png)](https://img2018.cnblogs.com/i-beta/1381066/201912/1381066-20191216105221514-1065718843.png)

 

 

- 当容器启动时，一个新的可写层被加载到镜像的顶部。
- 这一层通常被称作“容器层”，“容器层”之下的都叫“镜像层”。 
- 所有对容器的改动 - 无论添加、删除、还是修改文件都只会发生在容器层中。 
- 只有容器层是可写的，容器层下面的所有镜像层都是只读的。 

对于增删改查：

1. 添加文件：在容器中创建文件时，新文件被添加到容器层中。
2. 读取文件：在容器中读取某个文件时，Docker 会从上往下依次在各镜像层中查找此文件。一旦找到，立即将其复制到容器层，然后打开并读入内存。 
3. 修改文件：在容器中修改已存在的文件时，Docker 会从上往下依次在各镜像层中查找此文件。一旦找到，立即将其复制到容器层，然后修改之。 只有当需要**修改时才复制一份数据**，这种特性被称作 **Copy-on-Write**。
4. 删除文件：在容器中删除文件时，Docker 也是从上往下依次在镜像层中查找此文件。找到后，会在容器层中记录下此删除操作。 

容器层记录对镜像的修改，所有**镜像层都是只读的，不会被容器修改**，所以**镜像可以被多个容器共享**。

***3\***|***2\*****3.2 构建镜像****3.2.1 docker commit**

1、运行容器



```
docker run -it ubuntu         -it参数的作用是以交互模式进入容器，并打开终端
```

2、修改容器



```
apt-get install -y vim
```

3、将容器保存为新的镜像



```
查看当前运行的容器：docker ps，发现自动分配了silly_goldberg名字
docker commit silly_goldberg ubuntu-with-vi
```

这种方式不建议使用，原因如下：

1. 手工，容易出错，效率低
2. 使用者只能拿到一个镜像，不知道这个镜像这么创建的，无法对镜像进行审计，存在安全隐患

当然，cockerfile底层也是docker commit一层层构建的

**3.2.2 Dockerfile**

　　Dockerfile就是普通文件，可用touch创建。
　　如果 /root 下有如下Dockerfile文件



```
FROM ubuntu
RUN apt-get upadte && apt-get install -y vim
```

　　如果运行的话



```
root@ubuntu:~# pwd         ①   
/root   
root@ubuntu:~# ls          ②    
Dockerfile    
root@ubuntu:~# docker build -t ubuntu-with-vi-dockerfile .        ③    
Sending build context to Docker daemon 32.26 kB           ④    
Step 1 : FROM ubuntu           ⑤    
 ---> f753707788c5    
Step 2 : RUN apt-get update && apt-get install -y vim           ⑥    
 ---> Running in 9f4d4166f7e3             ⑦    
......    
Setting up vim (2:7.4.1689-3ubuntu1.1) ...    
 ---> 35ca89798937           ⑧     
Removing intermediate container 9f4d4166f7e3          ⑨    
Successfully built 35ca89798937           ⑩    
root@ubuntu:~#    
```

　　各步骤详细解释



```
③
假设Dockerfile存在当前目录
docker build -t ubuntu-with-vi-dockerfile .  （会从 . 即当前目录查找dockerfile文件，并重命名为ubuntu-with-vi-dockerfile）
也可以通过 -f 参数指定 Dockerfile 的位置

④
从这步开始就是镜像真正的构建过程。 首先 Docker 将 build context 中的所有文件发送给 Docker daemon。build context 为镜像构建提供所需要的文件或目录。
Dockerfile 中的 ADD、COPY 等命令可以将 build context 中的文件添加到镜像。此例中，build context 为当前目录 /root，该目录下的所有文件和子目录都会被发送给 Docker daemon。 
所以，使用 build context 就得小心了，不要将多余文件放到 build context，特别不要把 /、/usr 作为 build context，否则构建过程会相当缓慢甚至失败。

⑤
执行FROM
将 ubuntu 作为 base 镜像。
ubuntu 镜像 ID 为 f753707788c5

⑥
执行 RUN，安装 vim

⑦
启动 ID 为 9f4d4166f7e3 的临时容器，在容器中通过 apt-get 安装 vim

⑧
安装成功后，将容器保存为镜像，其 ID 为 35ca89798937
这一步底层使用的是类似 docker commit 的命令 

⑨
删除临时容器 9f4d4166f7e3

⑩
镜像构建成功。 
通过 docker images 查看镜像信息。 
镜像 ID 为 35ca89798937，与构建时的输出一致我们要特别注意指令 RUN 的执行过程 ⑦、⑧、⑨。Docker 会在启动的临时容器中执行操作，并通过 commit 保存为新的镜像。 
```

　　查看镜像分层结构：



```
docker history ubuntu-with-vi-dockerfile
```

***3\***|***3\*****3.3 镜像的缓存特性**

　　Docker 会缓存已有镜像的镜像层，构建新镜像时，如果某镜像层已经存在，就直接使用，无需重新创建。

　　Dockerfile：



```
FROM ubuntu
RUN apt-get upadte && apt-get install -y vim
COPY testfile /
```

执行过程



```
root@ubuntu:~# ls           ① 
Dockerfile  testfile 
root@ubuntu:~# 
root@ubuntu:~# docker build -t ubuntu-with-vi-dockerfile-2 . 
Sending build context to Docker daemon 32.77 kB 
Step 1 : FROM ubuntu 
 ---> f753707788c5 
Step 2 : RUN apt-get update && apt-get install -y vim 
 ---> Using cache         ② 
 ---> 35ca89798937 
Step 3 : COPY testfile /          ③ 
 ---> 8d02784a78f4 
Removing intermediate container bf2b4040f4e9 
Successfully built 8d02784a78f4 
```

　　② 重点在这里：之前已经运行过相同的 RUN 指令，这次直接使用缓存中的镜像层 35ca89798937。 

　　如果我们希望在构建镜像时不使用缓存，可以在 docker build 命令中加上 --no-cache 参数。 

 

　　Dockerfile 中每一个指令都会创建一个镜像层，上层是依赖于下层的。无论什么时候，只要某一层发生变化，其上面所有层的缓存都会失效。 也就是说，如果我们改变 Dockerfile 指令的执行顺序，或者修改或添加指令，都会使缓存失效。 

***3\***|***4\*****3.4 调试Dockerfile**



```
1.从 base 镜像运行一个容器。
2.执行一条指令，对容器做修改。
3.执行类似 docker commit 的操作，生成一个新的镜像层。
4.Docker 再基于刚刚提交的镜像运行一个新容器。
5.重复 2-4 步，直到 Dockerfile 中的所有指令执行完毕。
```

　　如果下一个指令有问题，可以通过运行上一条指令的镜像来调试

***3\***|***5\*****3.5 Dockerfile 常用指令**



```
FROM 
    : 指定 base 镜像

MAINTAINER 
    : 设置镜像的作者，可以是任意字符串

COPY 
    : 将文件从 build context 复制到镜像。

COPY 支持两种形式 
    : 1. COPY src dest 
      2. COPY ["src", "dest"] 
      注意：src 只能指定 build context 中的文件或目录。

ADD
    : 与 COPY 类似，从 build context 复制文件到镜像。不同的是，如果 src 是归档文件（tar, zip, tgz, xz 等），文件会被自动解压到 dest

ENV
    : 设置环境变量，环境变量可被后面的指令使用
... 
ENV MY_VERSION 1.3 
RUN apt-get install -y mypackage=$MY_VERSION 
...

EXPOSE
    : 指定容器中的进程会监听某个端口，Docker 可以将该端口暴露出来

VOLUME
    : 将文件或目录声明为 volume

WORKDIR
    :为后面的 RUN, CMD, ENTRYPOINT, ADD 或 COPY 指令设置镜像中的当前工作目录

RUN
    :在容器中运行指定的命令

CMD
    : 容器启动时运行指定的命令。
    Dockerfile 中可以有多个 CMD 指令，但只有最后一个生效。CMD 可以被 docker run 之后的参数替换

ENTRYPOINT
    : 设置容器启动时运行的命令。
    Dockerfile 中可以有多个 ENTRYPOINT 指令，但只有最后一个生效。CMD 或 docker run 之后的参数会被当做参数传递给 ENTRYPOINT
```

**3.5.1 RUN、CMD、ENTRYPOINT的区别**

**Shell + Exec**



```
Shell格式
    RUN apt-get install python3

Exec格式
    RUN ["apt-get","install","python3"]

指令执行时，shell格式底层会调用 /bin/sh -c [command]
    ENV name Cloud Man ENTRYPOINT echo "Hello,$name"
    执行docker run [image]将输出：
    Hello，Cloud Man

指令执行时，Exec会直接调用 [command]，不会被shell解析
    ENV name Cloud Man ENTRYPOINT ["/bin/echo","Hello,$name"]
    执行docker run [image]将输出：
    Hello，$name
如果希望使用环境变量，做如下修改：
    ENV name Cloud Man ENTRYPOINT ["/bin/sh","-c","echo Hello,$name"]
    将会输出
    Hello，Cloud Man
```

**RUN**



```
RUN 指令通常用于安装应用和软件包。 
RUN 在当前镜像的顶部执行命令，并通过创建新的镜像层。

RUN apt-get update && apt-get install -y \bzr\cvs\git\mercurial\subversion 

一定要这么写，不能如下写法
RUN apt-get update
RUN apt-get install -y \bzr\cvs\git\mercurial\subversion 
因为在第二步会创建镜像层，或者调用镜像层，由于镜像缓存，不能保证是最新的
```

**CMD**



```
CMD 指令允许用户指定容器的默认执行的命令。 
此命令会在容器启动且 docker run 没有指定其他命令时运行。 
    1、如果 docker run 指定了其他命令，CMD 指定的默认命令将被忽略。 
    2、如果 Dockerfile 中有多个 CMD 指令，只有最后一个 CMD 有效。

三种格式
    1、Exec格式(推荐)：CMD ["executable","paam1","parama2"]
    2、CMD ["param1","parama2"] 为ENTRYPOINT提供额外的参数，此时ENTRYPOINT必须使用Exec格式
    3、Shell格式：CMD command param1 param2

Docker片段：CMD echo "Hello World" ，运行容器 docker run -it [image]，输出：
Hello World

Docker片段：CMD echo "Hello World" ，运行容器 docker run -it [image] /bin/bash，此时：
CMD命令被忽略，bash将被执行
```

**ENTRYPOINT**

　　可让容器以应用程序或者服务的形式运行。跟CMD不同的是，**不会被忽略**。



```
ENTRYPOINT ["/bin/echo","Hello"] CMD ["world"]
运行容器 docker run -it [image]
Hello World
运行容器 docker run -it [image] CloudMan
Hello CloudMan

ENTRYPOINT echo "Hello" 的shell格式不会忽略任何CMD或docker run提供的参数
```

**最佳实践**



```
最佳实践 
    1、使用 RUN 指令安装应用和软件包，构建镜像。 
    2、如果 Docker 镜像的用途是运行应用程序或服务，比如运行一个 MySQL，应该优先使用 Exec 格式的 ENTRYPOINT 指令。CMD 可为 ENTRYPOINT 提供额外的默认参数，同时可利用 docker run 命令行替换默认参数。 
    3、如果想为容器设置默认的启动命令，可使用 CMD 指令。用户可在 docker run 命令行中替换此默认命令。
```

***3\***|***6\*****3.6 分发镜像**

三种方式：

1. 用相同的Dockerfile在其他host构建镜像
2. 将镜像上传到公共Registry（比如Docker Hub），Host直接下载使用
3. 搭建私有的Registry供本地Host使用



```
docker build -t ubuntu-with-vi . ---就是镜像的名字

查看镜像 docker images 可以看到镜像的tag

实际上镜像的名字由两部分组成：
【image name】= 【repository】 : 【tag】
如果执行docker build时没有指定tag，默认为latest，相当于docker build -t ubuntu-with-vi : latest

假设我们现在发布了一个镜像 myimage，版本为 v1.9.1。那么我们可以给镜像打上四个 tag：1.9.1、1.9、1 和 latest:
    docker tag myimage-v1.9.1 myimage:1   （原来的myimage-v1.9.1:latest,新增一个myimage:1）
    docker tag myimage-v1.9.1 myimage:1.9
    docker tag myimage-v1.9.1 myimage:1.9.1
    docker tag myimage-v1.9.1 myimage:latest
过了一段时间，我们发布了 v1.9.2。这时可以打上 1.9.2 的 tag，并将 1 和1.9 和 latest 从 v1.9.1 移到 v1.9.2:
    docker tag myimage-v1.9.2 myimage:1
    docker tag myimage-v1.9.2 myimage:1.9
    docker tag myimage-v1.9.2 myimage:1.9.2
    docker tag myimage-v1.9.2 myimage:latest
之后，v2.0.0 发布了。这时可以打上 2.0.0、2.0 和 2 的 tag，并将 latest 移到 v2.0.0。

    docker tag myimage-v2.0.0 myimage:2
    docker tag myimage-v2.0.0 myimage:2.0
    docker tag myimage-v2.0.0 myimage:2.0.0
    docker tag myimage-v2.0.0 myimage:latest

这种 tag 方案使镜像的版本很直观，用户在选择非常灵活：
    myimage:1 始终指向 1 这个分支中最新的镜像。
    myimage:1.9 始终指向 1.9.x 中最新的镜像。
    myimage:latest 始终指向所有版本中最新的镜像。
如果想使用特定版本，可以选择 myimage:1.9.1、myimage:1.9.2 或 myimage:2.0.0。
```

***3\***|***7\*****3.7 Registry****3.7.1 公共Registry**



```
公共Docker Hub：
    1、先在Docker Hub中创建账号
    2、在Docker Host中登录：docker login -u 账号
    3、Docker Hub为了区分不同用户的同名镜像，镜像的registry中要包含用户名：[username]/xxx.tag。通过docker tag重命名镜像：docker tag hello-world dongye95/hello-world:test1
    4、比如要上传一份：docker push dongye95/hello-world:test1
    5、查看：docker images dongye95/hello-world
```

**3.7.2 本地Registry**



```
本地Registry：Registry本身也是一个镜像
    Docker已经将Registry开源，Docker Hub上也有官方的镜像registry
    1、docker pull registry(可以直接run，会自动去pull)
    2、docker run -d -p 5000:5000 -v /myregistry:/var/lib/registry registry:latest
        -d：后台启动容器
        -p：将容器的5000端口映射到Host的5000端口。5000是registry服务端口。前一个5000是本机端口，后一个是容器端口
        -v：将容器 /var/lib/registry目录映射到 Host的 /muregistry，用于存放镜像数据
    3、重命名镜像，使之与registry匹配
    docker tag dongye95/hello-world:test1 主机Host:5000/dongye95/hello-world:test1
    镜像名称由repository和tag两部分组成，而repository的完整格式为：[registry-host]:[port]/[username]/xxx，只有Docker Hub上的镜像可以省略 registry-host:[host]
    4、可以docker push 上传镜像了
    docker push 主机Host:5000/dongye/hello-world:test1
```

***3\***|***8\*****3.8 命令**



```
images：显示镜像列表
history：显示镜像构建历史
commit：从容器创建新镜像
build：从Dockerfile构建镜像
tag：给镜像打tag
pull：从registry下载镜像
push：将镜像上传到registry
rmi：删除Docker host中的镜像
    1、只能删除host上的镜像，不会删除registry的镜像
    2、一个镜像对应多个tag，只有当最后一个tag被删除时，镜像才被真正删除
search：搜索Docker Hub中的镜像
    1、搜索Docker Hub中的镜像
    2、docker search httpd
```

***4\***|***0\*****四、容器*****4\***|***1\*****4.1 运行容器**



```
三种方式指定容器启动时执行命令
    1、CMD指令
    2、ENTRYPOINT指令
    3、在docker run命令中指定
　　　　docker run ubuntu pwd
　　　　返回目录
```

查看Docker host中当前运行的容器



```
docker ps [-a]
docker container ls [-a]
[-a]包括已经exited的容器
```

**4.1.1 让容器长期运行**



```
docker run ubuntu /bin/bash -c "while true;do sleep 1;done"
这种方法会占用一个终端

不占用终端，后台执行
docker run -d ubuntu /bin/bash -c "while true;do sleep 1;done"
```

**4.1.2 容器的container id和name**



```
容器启动时有一个“长ID”，docker ps时container id字段会显示“长ID”的前12位。NAMES字段显示容器的名字，在启动容器的时候可以通过 --name参数显式的为容器命名，不命名则自动分配
docker run --name "my_http_server" -d httpd

对于容器的操作，需要通过“长ID”、“短ID”或者“名称”

docker run --name "mysql_http_server" -d -p 80:80 httpd
也可以通过rename重命名
```

**4.1.3 两种进入容器的方法**



```
docker attach <container>
退出：Ctrl+p，然后Ctrl +q 

docker exec -it <container> bash
    1、-it以交互模式打开 pseudo-TTY，执行bash，打开一个bash终端
    2、进入到容器内，容器的hostname就是其“短ID”
    3、可以想在linux里一样操作
    4、exit退出

docker exec -it <container> bash|sh

只查看日志的话可以用logs
docker logs -f <container>  -f类似于tail -f 能够持续输出
```

attach VS exec
\1. attach 直接进入容器启动命令的终端，不会启动新的进程
\2. exec则是在容器中打开新的终端，并且可以启动新的进程
\3. 如果想直接在终端中查看启动命令的输出，用attach，其他情况使用exec

attach 比较类似于logs命令

***4\***|***2\*****4.2 容器相关命令**



```
docker stop <container>        实际上向容器进程发送一个SIGTERM
docker start <container>       会保留容器的第一次启动时的所有参数
docker kill <container>          快速结束容器，实际上是发送SIGKILL

自动重启
docker run -d --restart=always httpd
--restart=always 表示不管何种方式退出都重启----包括正常退出
--restart=on-failure:3，意识是如果启动进程退出代码非0，则重启机器，最多3次

docker pause <container>
docker unpause <container>

docker rm <container>

批量删除所有已经退出的容器：
docker rm -v $(docker ps -aq -f status=exited)

docker create <container>，只创建一个容器，处于created状态，未start
```

***4\***|***3\*****4.3 资源限制****4.3.1 内存**

与操作系统类似，容器可使用的内存包括两部分：物理内存和swap



```
-m或者--memory：设置内存的使用限额
--memory-swap：设置内存+swap的使用限额

docker run -it -m 200M --memory-swap=300M ubuntu
如果只有 -m 没有 --memory-swap，这后者默认为前者的两倍

使用progrium/stress镜像来测试，该镜像可用于对容器执行压力测试
docker run -it -m 200M --memory-swap=300M progrium/stress --vm 1 --vm-bytes 280M
--vm 1：启动1个内存工作线程
--vm-bytes 280M：每个线程分配280M内存
```

**4.3.2 CPU限额**



```
-c 或者 --cpu-shares，设定权重，不设定的话默认值为1024
docker run --name "container_A" -c 1024 ubuntu
docker run --name "container_B" -c 512 ubuntu
当争抢CPU时，则A能抢到B的两倍。不争抢都是正常跑满

docker run --name container_A -it -c 1024 progrium/stress --cpu 1
docker run --name container_B -it -c 1024 progrium/stress --cpu 1
--cpu 1 用来设置工作线程数量，只有1个cpu的话，1个工作线程就能将CPU压满
此时，A分得66的cpu资源，B只有33。如果关掉A，则全部分给B。
```

