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

![image-20231029133636644](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291444029.png)

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

### 1. What — 什么是容器

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

## 第3章 Docker 镜像

### 1. 镜像的内部结构

#### (1) hello-world — 最小的镜像

hello-world是Docker官方提供的一个镜像，通常用来验证Docker是否安装成功

```
docker pull hello-world  #从Docker Hub下载
docker images  #查看镜像信息
docker run hello-world  #运行
```

hello-world 的 Dockerfile

```
FROM scratch  #此镜像是从白手起家，从 0 开始构建
COPY hello /  #将文件“hello”复制到镜像的根目录。
CMD ["/hello"] #容器启动时，执行/hello
```

#### (2) base 镜像

所谓的 base 镜像：

(1) 不依赖其他镜像，从 scratch 构建

(2) 其他镜像可以之为基础进行扩展

以CentOS为例考察base镜像包含哪些内

下载镜像：

```shell
docker pull centos
```

查看镜像信息

![image-20231029145813722](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291458784.png)

Centos的镜像：标准的base镜像

```
FROM scratch
ADD centos-7-docker.tar.xz /
CMD ["/bin/bash"]
```



#### (3) 镜像为什么这么小（一个centos只有200M）

linux操作系统由**内核空间**和**用户空间**组成

内核空间是**kernel**，linux刚启动的时候回加载**bootfs**文件系统，之后bootfs会被卸载掉

用户空间是**rootfs**，包含熟悉的/dev、/bin目录等

对于base镜像来说，底层直接用 Host 的 kernel，自己只需要提供 rootfs 就行

![image-20231029150113896](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291501966.png)

 

#### (4) 为什么镜像可以运行在不同的 linux 发行版本上

不同Linux发行版的区别主要就是roots，Linux kernel差别不大

base镜像只是在用户控件与发行版一致，kernel版本与发行版是不同的。比如centos使用3.x.x的kernel，但是Docker Host是4.x.x的话，容器的kernel版本与Host一致

![image-20231029150453349](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291504406.png)

容器只能使用 Host 的 kernel，并且不能修改。所以如果容器对kernel版本有要求，则不建议用容器，虚拟机更合适

#### (5) 镜像的分层结构

Docker Hub 中 99% 的镜像都是通过在 base 镜像中安装和配置需要的软件构建出来的

新镜像是从 base 镜像一层一层叠加生成的。每安装一个软件，就在现有镜像的基础上增加一层

最大的一个好处就是 - 共享资源

(1) 有多个镜像都从相同的 base 镜像构建而来，那么 Docker Host 只需在磁盘上保存一份 base 镜像

(2) 同时内存中也只需加载一份 base 镜像，就可以为所有容器服务了

如果多个容器共享一份基础镜像，当某个容器修改了基础镜像的内容，比如 /etc 下的文件，其他容器下的/etc不会被修改

**可写的容器层**

[![img](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291342997.png)](https://img2018.cnblogs.com/i-beta/1381066/201912/1381066-20191216105221514-1065718843.png)

- 当容器启动时，一个新的可写层被加载到镜像的顶部
- 这一层通常被称作“容器层”，“容器层”之下的都叫“镜像层”
- 所有对容器的改动 - 无论添加、删除、还是修改文件都只会发生在容器层中
- 只有容器层是可写的，容器层下面的所有镜像层都是只读的

对于增删改查：

(1) 添加文件：在容器中创建文件时，新文件被添加到容器层中

(2) 读取文件：在容器中读取某个文件时，Docker 会从上往下依次在各镜像层中查找此文件。一旦找到，立即将其复制到容器层，然后打开并读入内存

(3) 修改文件：在容器中修改已存在的文件时，Docker 会从上往下依次在各镜像层中查找此文件。一旦找到，立即将其复制到容器层，然后修改之。 只有当需要**修改时才复制一份数据**，这种特性被称作 **Copy-on-Write**

(4) 删除文件：在容器中删除文件时，Docker 也是从上往下依次在镜像层中查找此文件。找到后，会在容器层中记录下此删除操作

容器层记录对镜像的修改，所有**镜像层都是只读的，不会被容器修改**，所以**镜像可以被多个容器共享**

### 2. 构建镜像

#### (1) docker commit

docker commit 命令是创建新镜像最直观的方法，其过程包含三个步骤：

1. 运行容器

![image-20231029151511092](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291515157.png)

2. 修改容器

![image-20231029151536565](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291515632.png)

3. 将容器保存为新的镜像

查看当前运行的容器：docker ps，发现自动分配了silly_goldberg名字

![image-20231029151715850](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291517927.png)

执行docker commit命令将容器保存为镜像，新的镜像命名为 ubuntu-with-vi

![image-20231029151759043](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291517096.png)

查看新镜像的属性，从size上看到镜像因为安装了软件而变大了

![image-20231029151908375](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291519445.png)

这种方式不建议使用，原因如下：

1. 手工，容易出错，效率低且可重复性弱
2. 使用者只能拿到一个镜像，不知道这个镜像这么创建的，无法对镜像进行审计，存在安全隐患

当然，Dockerfile底层也是docker commit一层层构建的



#### (2) Dockerfile

##### 第一个Dockerfile

Dockerfile就是普通文件，可用touch创建

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
镜像构建成功
```

通过 docker images 查看镜像信息

![image-20231029152438320](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291524394.png)

镜像 ID 为 35ca89798937，与构建时的输出一致我们要特别注意指令 RUN 的执行过程 ⑦、⑧、⑨。Docker 会在启动的临时容器中执行操作，并通过 commit 保存为新的镜像

##### 查看镜像分层结构

Ubuntu-with-vi-dockefile 是通过在base镜像的顶部添加一个新的镜像层而得到的

![image-20231029152519968](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291525091.png)

这个新镜像层的内容由RUN apt-get update && apt-get install -y vim 生成。这一点我们可以通过 docker history 命令验证

![image-20231029153324898](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291533036.png)

##### 镜像的缓存特性

Docker 会缓存已有镜像的镜像层，构建新镜像时，如果某镜像层已经存在，就直接使用，无需重新创建

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

​	   ① 确保 testfile 已存在

　　② 重点在这里：之前已经运行过相同的 RUN 指令，这次直接使用缓存中的镜像层 35ca89798937

​       ③ 执行 COPY 指令

其过程是启动临时容器，复制 testfile，提交新的镜像层 8d02784a78f4，删除临时容器ubuntu-with-vi-dockerfile-2

在 ubuntu-with-vi-dockefile 镜像上直接添加一层就得到了新的镜像 

![image-20231029153809711](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291538841.png)

如果我们希望在构建镜像时不使用缓存，可以在 docker build 命令中加上 --no-cache 参数 

Dockerfile 中每一个指令都会创建一个镜像层，上层是依赖于下层的。无论什么时候，只要某一层发生变化，其上面所有层的缓存都会失效。 也就是说，如果我们改变 Dockerfile 指令的执行顺序，或者修改或添加指令，都会使缓存失效

##### 调试Dockerfile

总结一下通过Dockerfile构建镜像的过程：

```
1.从 base 镜像运行一个容器。
2.执行一条指令，对容器做修改。
3.执行类似 docker commit 的操作，生成一个新的镜像层。
4.Docker 再基于刚刚提交的镜像运行一个新容器。
5.重复 2-4 步，直到 Dockerfile 中的所有指令执行完毕。
```

如果下一个指令有问题，可以通过运行上一条指令的镜像来调试

我们来看一个调试的例子，Dockerfile的内容如图所示

![image-20231029154129261](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291541317.png)

执行docker build，如图所示

![image-20231029154237993](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291542065.png)

Docker在执行第三步 RUN 指令时失败。我们可以利用第二步创建的镜像 22d31cc52b3e 进行调试，方法是通过 docker run -it 启动镜像的一个容器

![image-20231029154418234](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291544333.png)

手工执行 RUN 指令很容易定位失败的原因是 busybox 镜像中没有bash

##### Dockerfile 常用指令

下面列出了 Dockerfile 中常用的指令

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



### 3. RUN、CMD、ENTRYPOINT的区别

#### (1) Shell + Exec

我们可以使用两种方式指定RUN、CMD和ENTRYPOINT要运行的命令：Shell格式和Exec格式

```
Shell格式
    RUN apt-get install python3

Exec格式
    RUN ["apt-get","install","python3"]

指令执行时，shell格式底层会调用 /bin/sh -c [command]
    ENV name Cloud Man 
    ENTRYPOINT echo "Hello,$name"
    执行docker run [image]将输出：
    Hello，Cloud Man

指令执行时，Exec会直接调用 [command]，不会被shell解析
    ENV name Cloud Man 
    ENTRYPOINT ["/bin/echo","Hello,$name"]
    执行docker run [image]将输出：
    Hello，$name
如果希望使用环境变量，做如下修改：
    ENV name Cloud Man ENTRYPOINT ["/bin/sh","-c","echo Hello,$name"]
    将会输出
    Hello，Cloud Man
```

#### (2) RUN

```
RUN 指令通常用于安装应用和软件包
RUN 在当前镜像的顶部执行命令，并通过创建新的镜像层

RUN apt-get update && apt-get install -y \bzr\cvs\git\mercurial\subversion 

一定要这么写，不能如下写法
RUN apt-get update
RUN apt-get install -y \bzr\cvs\git\mercurial\subversion 
因为在第二步会创建镜像层，或者调用镜像层，由于镜像缓存，不能保证是最新的
```

#### (3) CMD

```
CMD 指令允许用户指定容器的默认执行的命令
此命令会在容器启动且 docker run 没有指定其他命令时运行
    1、如果 docker run 指定了其他命令，CMD 指定的默认命令将被忽略
    2、如果 Dockerfile 中有多个 CMD 指令，只有最后一个 CMD 有效

三种格式
    1、Exec格式(推荐)：CMD ["executable","paam1","parama2"]
    2、CMD ["param1","parama2"] 为ENTRYPOINT提供额外的参数，此时ENTRYPOINT必须使用Exec格式
    3、Shell格式：CMD command param1 param2

Docker片段：CMD echo "Hello World" ，运行容器 docker run -it [image]，输出：
Hello World

Docker片段：CMD echo "Hello World" ，运行容器 docker run -it [image] /bin/bash，此时：
CMD命令被忽略，bash将被执行
```

#### (4) ENTRYPOINT

可让容器以应用程序或者服务的形式运行。跟CMD不同的是，**不会被忽略**

```
ENTRYPOINT ["/bin/echo","Hello"] CMD ["world"]
运行容器 docker run -it [image]
Hello World
运行容器 docker run -it [image] CloudMan
Hello CloudMan

ENTRYPOINT echo "Hello" 的shell格式不会忽略任何CMD或docker run提供的参数
```

#### (5) 最佳实践

```
最佳实践 
    1、使用 RUN 指令安装应用和软件包，构建镜像
    2、如果 Docker 镜像的用途是运行应用程序或服务，比如运行一个 MySQL，应该优先使用 Exec 格式的 ENTRYPOINT 指令。CMD 可为 ENTRYPOINT 提供额外的默认参数，同时可利用 docker run 命令行替换默认参数
    3、如果想为容器设置默认的启动命令，可使用 CMD 指令。用户可在 docker run 命令行中替换此默认命令
```



### 4. 分发镜像

如何在多个Docker Host上使用镜像，这里有三种方式：

1. 用相同的Dockerfile在其他host构建镜像
2. 将镜像上传到公共Registry（比如Docker Hub），Host直接下载使用
3. 搭建私有的Registry供本地Host使用

#### (1) 为镜像命名

无论采用何种方式保存和分发镜像，首先都得给镜像命名

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

**1.9.1版本**

![image-20231029155843842](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291558999.png)

**1.9.2版本**

![image-20231029155910000](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291559193.png)

**2.0.0版本**

![image-20231029155951482](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291559628.png)

#### (2) 公共Registry

```
公共Docker Hub：
    1、先在Docker Hub中创建账号
    2、在Docker Host中登录：docker login -u 账号
    3、Docker Hub为了区分不同用户的同名镜像，镜像的registry中要包含用户名：[username]/xxx.tag。通过docker tag重命名镜像：docker tag hello-world dongye95/hello-world:test1 （注：Docker官方自己维护的镜像没有镜像名，比如httpd)
    4、比如要上传一份：docker push dongye95/hello-world:test1
    5、查看：docker images dongye95/hello-world
```

#### (3) 本地Registry

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

### 5. 小结

#### (1) 镜像常用的子命令

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

## 第4章 Docker 容器

### 1. 运行容器

三种方式指定容器启动时执行命令

```
    1、CMD指令
    2、ENTRYPOINT指令
    3、在docker run命令中指定
　　　　docker run ubuntu pwd
　　　　返回目录
```

例如下面的例子

![image-20231029160858742](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291608801.png)

容器启动时执行pwd，返回的 / 是容器中的当前目录。执行 docker ps 或 docker container ls 可以查看 Docker host 中 当前运行的容器

![image-20231029161057113](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291610172.png)

怎么没有容器？用 docker ps -a 或 docker container ls -a 看看

![image-20231029161155164](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291611230.png)

-a 会显示所有状态的容器，可以看到，之前的容器已经退出了，状态为Exited



### 2. 让容器长期运行

容器的生命周期依赖于启动时执行的命令，只要该命令不结束，容器也就不会退出

理解了这个原理，我们就可以通过一个长期运行的命令来保持容器的运行状态，例如执行下面的命令

```
docker run ubuntu /bin/bash -c "while true;do sleep 1;done"
这种方法会占用一个终端

不占用终端，后台执行
docker run -d ubuntu /bin/bash -c "while true;do sleep 1;done"
```

**容器的container id和name**

![image-20231029161615780](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291616837.png)

![image-20231029161706317](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291617366.png)

容器启动时有一个“长ID”，docker ps时container id字段会显示“长ID”的前12位。NAMES字段显示容器的名字，在启动容器的时候可以通过 --name参数显式的为容器命名，不命名则自动分配

![image-20231029161821455](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291618514.png)

这一次我们用 --name 指定了容器的名字。我们还可以看到容器运行的命令是 httpd-foreground，通过docker history 可知这个命令是通过 CMD 指定的

![image-20231029162044972](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291620036.png)



### 3. 两种进入容器的方法

我们经常需要进到容器内部去做一些工作，比如查看日志、调试、启动其他进程等。由两种方法进入容器：attach 和 exec

#### (1) docker attach

docker attach 可以attach到容器启动命令到终端，如图所示

![image-20231029162345022](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291623080.png)

通过“长ID”attach命令到了容器命令终端，之后看到echo每隔一秒打印到信息

注：可通过 Crtl + p，然后 Crtl + q 的组合退出attach终端



#### (2) docker exec

通过docker exec 进入相同的容器，如图所示

![image-20231029162846184](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291628264.png)

说明如下：

​       ① -it以交互模式打开 pseudo-TTY，执行bash，打开一个bash终端

　　② 进入到容器内，容器的hostname就是其“短ID”

​       ③ 可以像在linux里一样操作，ps -elf 显示了容器启动进程 while 以及当前的 bash 进程

​       ④ 执行 exit 退出容器，回退到 docker host



#### (3) attach VS exec

attach与exec的主要区别如下：

1. attach 直接进入容器启动命令的终端，不会启动新的进程
2. exec则是在容器中打开新的终端，并且可以启动新的进程
3. 如果想直接在终端中查看启动命令的输出，用attach，其他情况使用exec

当然，如果只是为了查看启动命令的输出，可以使用docker logs 命令，如图所示

![image-20231029163536023](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291635085.png)

### 4. 容器相关命令

```
启动/停止
docker stop <container>        实际上向容器进程发送一个SIGTERM
docker start <container>       会保留容器的第一次启动时的所有参数
docker kill <container>          快速结束容器，实际上是发送SIGKILL

自动重启
docker run -d --restart=always httpd
--restart=always        表示不管何种方式退出都重启----包括正常退出
--restart=on-failure:3，   意思是如果启动进程退出代码非0，则重启机器，最多3次

暂停/恢复
有时我们只是希望容器暂停工作一段时间，比如要对容器的文件系统打个快照，或者docker host 需要使用CPU，这时可以执行如下命令docker pause。处于暂停状态的容器不会占用CPU资源
docker pause <container>
docker unpause <container>

删除
docker rm <container>

批量删除所有已经退出的容器：
docker rm -v $(docker ps -aq -f status=exited)

创建
docker create <container>，只创建一个容器，处于created状态，未start
```



### 5. 状态机

下面这张状态机很好地总结了容器各种状态之间是如何转换的

![image-20231029164226411](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291642569.png)

由两点需要补充一下：

#### (1) 可以先创建容器，稍后再启动

![image-20231029164432639](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310291644751.png)

​       

​       ① docker create 创建的容器处于Created 状态

　　② docker start 将以后台方式启动容器。docker run 的命令实际上是 docker create 和 docker start 的组合

​    

#### (2) 只有当容器的启动进程退出时，--restart 才生效

退出包括正常退出或者非正常退出，这里举了两个例子：启动进程正常退出或发生OOM，此时Docker会根据--restart的策略判断是否需要重启容器。但如果容器是因为执行docker stop 或 docker kill 退出，则不会自动重启



### 6. 资源限制

#### (1) 内存限额

与操作系统类似，容器可使用的内存包括两部分：物理内存和swap

```
-m或者--memory：设置内存的使用限额
--memory-swap：设置内存+swap的使用限额

docker run -it -m 200M --memory-swap=300M ubuntu  #内存限额200M，swap限额100M
如果只有 -m 没有 --memory-swap，这后者默认为前者的两倍

使用progrium/stress镜像来测试，该镜像可用于对容器执行压力测试
docker run -it -m 200M --memory-swap=300M progrium/stress --vm 1 --vm-bytes 280M
--vm 1：启动1个内存工作线程
--vm-bytes 280M：每个线程分配280M内存
```



#### (2) CPU 限额

```
-c 或者 --cpu-shares，设定权重，不设定的话默认值为1024
docker run --name "container_A" -c 1024 ubuntu
docker run --name "container_B" -c 512 ubuntu
当争抢CPU时，则A能抢到B的两倍。不争抢都是正常跑满

docker run --name container_A -it -c 1024 progrium/stress --cpu 1
docker run --name container_B -it -c 512 progrium/stress --cpu 1
--cpu 1 用来设置工作线程数量，只有1个cpu的话，1个工作线程就能将CPU压满
此时，A分得66%的cpu资源，B只有33%。如果关掉A，则全部分给B。
```

![image-20231029170123069](/Users/beck/Library/Application Support/typora-user-images/image-20231029170123069.png)



#### (3) Block IO 带宽限额

Block IO 是另一种可以限制容器使用资源的资源。Block IO指的是磁盘的读写，docker可以通过设置权重、限制bps和iops的方式控制读写磁盘的带宽

##### block IO 权重

--block-weight 与 --cpu-shares 类似，设置的是相对权重值，默认为500。在下面的例子中，container A 读写磁盘的带宽 是container B的两倍

```
docker run -it --name container_A --blkio-weight 600 ubuntu docker run -it --name container_B --blkio-weight 300 ubuntu
```

##### 限制 bps 和 iops

bps 是 byte per second    每秒读写的数据量

iops 是 io per second     每秒IO的次数

可通过以下参数控制容器的 bps 和 iops

+ --device-read-bps：限制读某个设备的bps
+ --device-write-bps：限制写某个设备的bps
+ --device-read-iops：限制读某个设备的iops
+ --device-write-iops: 限制写某个设备的iops

下面这个例子限制容器写/dev/sda的速率为 30MB/s

```
docker run -it --device-write-bps /dev/sda:30MB ubuntu
```

我们来看试验效果，如图所示：

![image-20231029234456234](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310292344319.png)

通过dd测试在容器中写磁盘的速度。因为容器的文件系统是在host /dev/sda 上的，在容器中写文件相当于对host /dev/sda 进行写操作。另外，oflag=direct 指定用direct IO 方式写文件，这样 --device-write-bps 才能生效

结果表明，bps 25.6 MB/s 没有超过 30MB/s 的限速

作为测试对比，如果不限速，结果如图所示

![image-20231029234830008](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310292348132.png)



### 7. 实现容器的底层技术

cgroup 和 namespace 是最重要的两门技术。cgroup 实现资源限额，namespace 实现资源隔离

#### (1) cgroup

> Group 全称 Control Group。Linux系统通过 cgroup 可以设置进程使用CPU、内存和 IO 资源限额。前面的--cpu-shares、-m、--device-write-bps 实际上就是在配置 cgroup

cgroup长什么样子呢？我们可以在 /sys/fs/cgroup 中找到它。还是用例子说明，启动一个容器，设置 --cpu-shares=512，如图所示

![image-20231029235642620](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310292356750.png)

查看容器的ID，如图所示

![image-20231029235745842](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310292357967.png)

在 /sys/fs/cgroup/docker 目录中，Linux 会为每个人器创建一个 group 目录，以容器长 ID 命名，如图所示

![image-20231030000027177](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310300000311.png)

目录中包含所有与 cpu 相关的 cgroup 配置，文件 cpu.shares 保存的就是 --cpu-shares 的配置，值为512

同样的，/sys/fs/cgroup/memory/docker 和 /sys/fs/cgroup/blkio/docker 中保存的是内存以及Block IO的 group 配置

#### (2) namespace

> namespace管理着host中全局唯一的资源，并可以让每个容器都觉得只有自己在使用它。换句话说，namespace实现了容器间资源的隔离

Linux 使用了6种namespace，分别对应6种资源：Mount、UTS、IPC、PID、Network和User

##### Mount namespace

Mount namespace 让容器看上去拥有整个文件系统

容器有自己的 / 目录，可以执行mount和umount命令。当然我们知道这些操作只在当前容器中生效，不会影响到host和其他容器

##### UTS namespace

简单地说，UTS namespace 让容器有自己的 hostname。默认情况下，容器的hostname 是它的短ID，可以通过 -h 或 --hostname 参数设置

![image-20231030001005982](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310300010161.png)

##### IPC namespace

IPC namespace 让容器拥有自己的共享内存和信号量来实现进程间通信，而不会与 host 和其他容器的 IPC 混在一起

##### PID namespace

前面提到过，容器在host中以进程的形式运行。例如当前host中运行了两个容器，如图所示

![image-20231030001252490](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310300012630.png)

通过 ps axf 可以查看容器进程，如图所示

![image-20231030001419763](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310300014900.png)

所有容器的进程都挂在dockerd进程下，同时也可以看到容器自己的子进程，如果我们进入到某个容器，ps 就只能看到自己的进程了

![image-20231030001600794](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310300016921.png)

而且进程中的 PID 不同于 host 中对应进程的 PID，容器中 PID=1 的进程当然也不是host的init进程。**也就是说：容器拥有自己独立的一套PID，这就是 PID namespace 提供的功能**

##### Network namespace 

Network namespace 让容器拥有自己独立的网卡、IP、路由等资源

##### User namespace

User namespace 让容器能够管理自己的用户，host 不能看到容器中创建的用户，如图所示

![image-20231030001953903](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202310300019035.png)

在容器中创建了用户 cloudman，但host中并不会创建相应的用户



### 8. 小结

下面是容器常用操作命令

```
docker create 创建容器
docker run 运行容器
docker pause 暂停容器
docker unpause 取消暂停继续运行容器
docker stop 发送SIGTERM 停止容器
docker kill 发送SIGKILL 快速停止容器
docker start 启动容器
docker restart 重启容器
docker attach attach到容器启动进程的终端
docker exec 在容器中启动新进程，通常使用"-it"参数
docker logs 显示容器启动进程的控制台输出，用"-f"持续打印
docker rm 从磁盘中删除容器
```

### 9. 作业

**环境准备**

- 在本地安装docker
- 注册一个dockerhub账号

**作业**

(1). 从dockerhub获取一个centos 7作为基础镜像，在centos 7 中安装 vim，并使用 docker commit 打成新的镜像，删除本地镜像，将这个镜像push到dockerhub，然后下载该镜像，使用命令行的方式启动容器的时候，执行"vim -h"命令，使用一种方法查看容器启动时执行的命令

1. 拉取镜像![image-20231116203738587](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311162037904.png)

2. 检查系统版本，安装vim 

   ![image-20231116205047326](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311162050405.png)

3. 使用docker commit 打包成新的镜像，centos-vim

   ![image-20231116205522115](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311162055204.png)

4. push 到dockerhub

   ![image-20231116211713946](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311162117109.png)

   ![image-20231116213623853](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311162136957.png)

5. 删除本地的centos-vim 镜像

   ![image-20231116213841597](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311162138710.png)

6. 拉取dockerhub的镜像

   ![](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311162146774.png)

7. 启动容器时执行 "docker -h" 命令

   ![image-20231116214901769](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311162149883.png)

8. 使用一种方法查看容器启动时的命令

   ![image-20231116215757285](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311162157457.png)

(2). 使用dockerfile的方式打一个新的镜像：以centos7作为基础镜像，在镜像中安装vim，然后定义容器启动时的命令"vim -h"。然后直接使用本地镜像运行容器，使用一种方法查看容器启动时的命令

1. 定义dockerfile

   ![image-20231116220656987](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311162206268.png)

2. 打包新的镜像

   ![image-20231116221220614](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311162212732.png)

3. 运行容器，使用第一种方法查看容器启动时的命令

   ![image-20231116221604887](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311162216005.png)

4. 第二种查看启动命令的方法

   ![image-20231116221856969](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311162218084.png)

5. 第三种查看启动命令的方法

   ![image-20231116222340044](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311162223202.png)



### 10. 课堂问题

(1). docker 是否直接使用windows内核

<https://www.51cto.com/article/507951.html>

(2). dockerfile中使用ENV，是否可以直接设置容器的环境变量

![image-20231116224122214](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311162241319.png)

(3).如何保证docker一直在后台运行不停止

<https://www.jerrymei.cn/docker-container-run-not-stop-automatically/>



## 第5章 Docker 网络

### 1. Docker的原生网络

Docker安装时会自动在host上创建三个网络，我们可以用 docker network ls 命令查看，如图所示

![image-20231111233740089](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311112337297.png)



#### (1). none 网络

顾名思义，none网络就是什么都没有的网络。挂在这个网络下的容器除了 lo，没有其他任何网卡。容器创建时，可以通过 --network=none 指定使用none网络，如图所示

![image-20231111235216926](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311112352160.png)

这样一个封闭的网络有什么用？

封闭意味着隔离，一些对安全要求高并且不需要联网的应用可以使用none网络。比如某个容器的唯一用途是生成随机密码，就可以放到none网络中避免密码被窃取

#### (2). host 网络

连接到host网络的容器共享Docker host的网络栈，容器的网络配置与host完全一样，可以通过--network=host指定使用host网络，如图所示

![image-20231112100852984](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311121008078.png)

在容器中可以看到host的所有网卡，并且连hostname也是host的。host网络的使用场景又是什么呢？

直接使用Docker host的网络最大的好处就是性能，如果容器对网络传输效率有较高要求，则可以选择host网络。当然不便之处就是牺牲一些灵活性，比如要考虑端口冲突问题，Docker host上已经使用的端口就不能再用了

Docker host的另一个用途是让容器可以直接配置host网络，比如某些跨host的网络解决方案，其本身也是已容器方式运行，这些方案需要对网络进行配置，比如管理iptables

#### (3). bridge 网络

Docker 安装时会创建一个命名为 docker0 的Linux bridge。如果不指定--network，创建的容器默认都会挂到docker0上，如图所示

![image-20231112101808878](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311121018009.png)

当前docker0上没有任何其他网络设备，我们创建一个容器看看有什么变化，如图所示

![image-20231112102130966](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311121021087.png)

一个新的网络接口vetha85a84e被挂到了docker0上，vetha85a84e就是新创建容器的虚拟网卡

下面看一个容器的网络配置，如图所示（注：在使用 ip a命令前，需要在容器内先更新apt-get，然后安装iproute2，完整命令是：apt-get update & apt-get install -y iproute2）

![](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311121054998.png)

容器有一个网卡eth0@if97，为什么不是vetha85a84e呢？

实际上，eth0@if97 和 vetha85a84e 是一对 veth pair。veth pair 是一种成对出现的特殊网络设备，可以把它们想象成由一根虚拟网线连接起来的一对网卡，网卡的一头eth0@if97在容器中，另一头vetha85a84e挂在网桥docker0上，其效果就是将eth0@if97也挂在了docker0上

我们还可以看到eth0@if97已经配置了IP 172.17.0.2，为什么是这个网段呢？可以通过docker network inspect bridge 看一下bridge网络的配置信息，如图所示

![image-20231112110103033](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311121101165.png)

原来bridge网络配置的subnet就是172.17.0.0/16，而且网关是172.17.0.1。这个网关在哪儿呢？其实就是docker0，如图所示

![image-20231112110301488](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311121103637.png)

容器创建时，docker会自动从172.17.0.0/16中分配一个IP，这里16位的掩码保证有足够多的IP可以供容器使用

### 2. User-defined 网络

#### (1). user-defined 网络驱动

除了none、host、bridge 这三个自动创建的网络，用户也可以根据业务需要创建user-defined 网络

Docker 提供三种user-defined 网络驱动：bridge、overlay 和 macvlan。overlay 和 macvlan 用于创建跨主机的网络，这里暂不讨论，这里只讨论 bridge

#### (2). 自动分配网段

我们可通过 bridge 驱动创建类似前面默认的 bridge 网络

![image-20231112111011329](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311121110463.png)

查看当前host的网络结构变化，发现新增了一个网桥 br-2028aa34a67b。这里 br-2028aa34a67b 正好是新建 bridge 网络my_net的短id。执行docker network inspect 查看一下 my_net 的配置信息

![image-20231112111314123](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311121113262.png)

这里的172.18.0.0/16 是Docker自动分配的IP网段

#### (2). 自定义网段

在以上的场景中，自己也可以指定IP网段，只需要在创建网段时指定 --subnet 和 --gateway 参数，如图所示

![image-20231112223700320](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311122237272.png)

这样我们创建了新的 bridge 网络my_net2，网段位172.22.16.0/24，网关位172.22.16.1，与前面一样，网关在my_net2 对应的网桥 br-27705f85aca0 上，如图所示

![](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311122239667.png)

#### (3). 容器中使用自定义网络

容器中要使用新的网络，需要在启动时通过 --network 指定，如图所示

![image-20231112224159477](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311122241603.png)

容器分配到的IP为172.22.16.2

#### (4). 指定静态IP

还可以通过--ip指定指定一个静态IP，如图所示

![image-20231112224809142](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311122248295.png)

注：只有使用 --subnet 创建的网络才能指定静态IP，如果my_net 创建时没有指定--subnet，那么指定静态IP的时候会报错，如图所示

![image-20231112225229479](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311122252638.png)

#### (5). 网络隔离

我们来看看当前docker host的网络拓扑结构，如图所示

![image-20231113192753645](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311131927750.png)

两个busybox容器都挂在my_net2上，应该可以互通，我们验证一下，如图所示

![image-20231112234638133](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311122346288.png)

可见同一网络中的容器，网关之间是可以通信的

my_net2 与默认的 bridge 网络能通信吗？从拓扑图可知，两个网络属于不同的网桥，应该不能通信，我们可以验证一下，让busybox容器 ping httd 容器，如图所示

![image-20231112235015802](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311122350960.png)

确实ping不通，符合预期

如果不同的网络加上路由应该可以通信了吧，这是一个非常好的想法。确实，如果host对应每个网络都有一个路由，同时操作系统上打开了 ip forwarding，host就成了一个路由器，挂接在不同网桥上的网络就能够相互通信。下面我们来看看docker host是否满足这些条件呢？

ip r 查看 host 上的路由表：

![image-20231112235500064](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311122355227.png)

172.17.0.0/16 和 172.22.16.0/24 两个网络的路由都定义好了，再看看 ip forwarding：

![image-20231112235703107](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311122357262.png)

ip forwarding 也已经启动了。条件都满足，为什么不能通行呢？

我们还得看看 iptables：

![image-20231113000107803](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311130001957.png)

原因就在这里了：iptables DROP 掉了网桥 docker0 与 br-27705f85aca0 之间双向的流量。从规则的命名 DOCKER-ISOLATION 可知docker在设计上就是要隔离不同的network

那么接下来的问题是：怎样才能让busybox与httpd通信呢？

答案是：为httpd器添加一块my_net2的网卡。这个可以通过 docker network connect 命令实现，如图所示

![image-20231113000510020](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311130005179.png)

我们在httpd容器中查看一下网络配置，如图所示

![image-20231113000705945](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311130007099.png)

容器中增加了一个网卡 eth1, 分配了my_net2的IP 172.22.16.3。现在busybox应该都能够访问httpd了，验证一下，如图所示

![image-20231113001215118](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311130012288.png)

busybox能够ping到httpd，并且可以访问httpd的web服务。当前网络结构如图所示

![image-20231113201125118](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311132011213.png)



## 第6章 Docker 存储

### 1. 容器存储数据的资源

Docker为容器提供了两种存放数据的资源：

(1). 由 storage driver 管理的镜像层和容器层

(2). Data Volume

### 2. storage driver

#### (1). 分层结构

我们之前学习到Docker镜像的分层结构，如图所示

![image-20231113202044849](/Users/beck/Library/Application Support/typora-user-images/image-20231113202044849.png)

容器由最上面一个可写的容器层，以及若干只读的镜像层组成，容器的数据就存放在这些层中。这样的分层结构最大的特性是 Copy-on-Write

分层结构使镜像和容器的创建、共享以及分发变得非常高效，而这些都要归功于Docker storage driver。正是 storage driver 实现了分层数据的堆叠，并为用户提供了一个单一的合并之后的统一视图

#### (2). storage driver分类

Docker 支持多种 storage driver，由AUFS、Device Mapper、Btrfs、OverlayFS、VFS和ZFS等。它们都能实现分层的架构，同时又有各自的特性。Docker 安装时会根据当前系统的配置选择默认的driver。默认的driver具有最好的稳定性，Docker官方推荐优先使用Linux发行版默认的 storage driver

运行docker info可以查看Centos的默认driver，如图所示

![image-20231113202946224](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311132029372.png)

Centos 默认的driver用的是overlay2，底层文件系统是xfs，各层数据存放在/var/lib/docker/overlay2中，如图所示

![image-20231113203220728](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311132032876.png)

#### (3). 无状态应用

对于某些容器，直接将数据存放在由storage driver维护的层中是很好的选择，比如那些无状态的应用。无状态意味着容器没有需要持有化的数据，随时可以从镜像直接创建。比如busybox，它是一个工具箱，启动busybox是为了执行诸如wget、ping之类的命令，不需要保存数据供以后使用，使用完直接退出，容器删除时存放在容器层的工作数据也一起被删除，这没问题，下次再启动新容器即可

#### (4). 有状态应用

但对于另一类应用这种方式就不合适了，它们由持久化数据的需求，容器启动时需要加载已有的数据，容器销毁时希望保留产生的新数据，也就是说，这类容器是有状态的

这就要用到Docker的另一种存储机制：Data Volume

### 3. Data Volume

#### (1). Data Volume 的特点

Data Volume 本质上是Docker Host文件系统中的目录或文件，能够直接被mount到容器的文件系统中。Data Volume 由以下特点：

1. Data Volume 是目录或文件，而非没有格式化的磁盘（块设备）
2. 容器可以读写 volume 中的数据
3. volume 数据可以被永久地保存，即使使用它的容器已经销毁

#### (2). 使用场景

数据层（镜像层和容器层）和volume都可以用来存放数据，具体的使用场景如下：

1. Database 软件：数据层
2. Database 数据：Data Volume
3. Web 应用：数据层
4. 应用产生的日志：Data Volume
5. 数据分析软件：数据层
6. input/output的数据：Data Volume
7. Apache Server：数据层
8. 静态的HTML文件：Data Volume

#### (3). volume 的容量

如何设置volume的容量，因为volume实际上是docker host文件系统的一部分，所以volume的容量取决于文件系统当前未使用的空间，目前还没有方法设置volume的容量

#### (4). volume 的类型

docker提供了两种类型的volume：bind mount 和 docker managed volume

#### (5). bind mount

bind mount 是将 host 上已存在的目录或文件 mount 到容器

例如 docker host 上有目录/root/htdocs，在里面创建一个index.html文件，内容如图所示

![image-20231113210107805](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311132101955.png)

通过 -v 将其 mount 到 httpd 容器，如图所示

![image-20231113210540324](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311132105401.png)

-v 的格式为<host path>:<container path>。/usr/local/apache2/htdocs 就是Apache Server 存放静态文件的地方。由于/usr/local/apache2/htdocs 已经存在，原有数据会被隐藏起来，取而代之的是host /root/htdocs/中的数据

![image-20231113211400292](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311132114459.png)

curl 显示当前主页确实是/root/htdocs/index.html 中的内容。更新一下，看是否能生效，如图所示

![image-20231113214221020](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311132142193.png)

host 中的修改确实生效了，bind mount 可以让 host 与容器共享数据。这里管理上是非常方便的

下面我们将容器销毁，看看对 bind mount 有什么影响

![image-20231113220445293](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311132204452.png)

可见，即使容器没有了，bind mount 也还在。这也合理，bind mount 是 host 文件系统中的数据，只是借给容器用用，不能随便就给删了

另外，bind mount 时还可以指定数据的读写权限，默认是可读可写，可指定为只读，如图所示

![image-20231113221537119](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311132215283.png)

ro 设置了只读权限，在容器中是无法对 bind mount 数据进行修改的。只有 host 有权修改数据，提高了安全性

除了 bind mount 目录，还可以单独指定一个文件（注：使用单独文件 mount 的时候，要使用绝对路径），如图所示

![image-20231113223431506](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311132234686.png)

使用 bind mount 单个文件的场景是：只需要向容器添加文件，不希望覆盖整个目录。在上面的例子中，我们将 html 文件加到 apache 中，同时也保留了容器原有的数据

使用单一文件有一点要注意：host 中的源文件必须要存在，不然会被当作一个新目录 bind mount 给容器中，在 host 中修改代码就能看到应用的实时效果。再比如将 MySQL 容器的数据放在 bind mount 里，这样 host 可以方便地备份和迁移数据

bind mount 的使用直观高效，易于理解，但它也有不足的地方：bind mount 需要指定 host 文件系统的特定路径，这就限制了容器的可移植性，当需要将容器迁移到其他 host，而该 host 没有要 mount 的数据或者数据不在相同的路径时，操作会失败

移植性更好的方式是 docker managed volume

#### (6). docker managed volume

docker managed volume 与 bind mount 在使用上最大区别是不需要指定 mount 源，指明 mount point 就行了。还是以 httpd 容器为例，如图所示

![image-20231113225507233](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311132255396.png)

我们通过 -v 告诉 docker 需要一个 data volume，并将其 mount 到/usr/local/apache2/htdocs，那么这个 data volume 具体在哪儿呢？

实际上，在容器的配置信息里可以找到，执行 docker inspect 命令：

```
[root@k8scloude1 ~]# docker inspect beck_managed_httpd
"Mounts": [
            {
                "Type": "volume",
                "Name": "59a6104b0ef88f344a39e695526a995b1c1ce86487568cb17035ec42135c6219",
                "Source": "/var/lib/docker/volumes/59a6104b0ef88f344a39e695526a995b1c1ce86487568cb17035ec42135c6219/_data",
                "Destination": "/usr/local/apache2/htdocs",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
```

可以看到，Mounts 这部分会显示容器当前使用的所有 data volume，包括 bind mount 和 docker managed volume，Source 就是该 volume 在 host 上的目录

**原来，每当容器申请 mount docker managed volume 时，docker 都会在 /var/lib/docker/volumes 下生成一个目录（例子中是 /var/lib/docker/volumes/59a6104b0ef88f344a39e695526a995b1c1ce86487568cb17035ec42135c6219/_data），这个目录就是 mount 源**

下面继续研究这个 volume，看看里面有什么东西，如图所示

![image-20231113230406123](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311132304278.png)

volume 的内容与容器原有 /usr/local/apache2/htdocs 完全一样，是因为如果 mount point 指向的是已有目录，原有数据会被复制到 volume 中

但要明确一点：此时的 /usr/local/apache2/htdocs 已经不再是由 storage driver 管理的层数据了，它已经是一个 data volume。我们可以像 bind mount 一样对数据进行操作，例如更新数据，如图所示

![image-20231113230922203](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311132309360.png)

简单回顾一下 docker managed volume 的创建过程：

1. 容器启动时，简单地告诉 docker "我需要一个 volume 存放数据，帮我 mount 到目录 /abc"
2. docker 在 /var/lib/docker/volumes 中生成一个随机目录作为 mount 源
3. 如果 /abc 已经存在，则将数据复制到 mount 源
4. 将 volume mount 到 /abc

除了通过 docker inspect 查看 volume，我们还可以通过 docker volume 命令，如图所示

![image-20231113231501914](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311132315077.png)

目前，docker volume 只能查看 docker managed volume，还看不到 bind mount，同时也无法知道 volume 对应的容器，这些信息还得靠 docker inspect

我们已经学习了两种 data volume 的原理和基本使用方法，下面做个对比：

1. 相同点：两者都是 host 文件系统中的某个路径

2. 不同点，如表所示

   | 不同点                  | bind mount                   | docker managed volume        |
   | :---------------------- | ---------------------------- | ---------------------------- |
   | volume 位置             | 可在任意指定                 | /var/lib/docker/volumes/...  |
   | 对已有 mount point 影响 | 隐藏并替换为 volume          | 原有数据复制到 volume        |
   | 是否支持单个文件        | 支持                         | 不支持，只能是目录           |
   | 权限控制                | 可设置为只读，默认为读写权限 | 无控制，均为读写权限         |
   | 移植性                  | 移植性弱，与 host path 绑定  | 移植性强，无须指定 host 目录 |

   
