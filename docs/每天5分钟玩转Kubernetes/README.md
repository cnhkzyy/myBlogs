

[toc]



# 每天5分钟玩转Kubernetes



## 第1章 重要概念

### 1. Kubernetes

> Kubernetes(K8s) 是 Google 在2014年发布的一个开源项目，是容器编排引擎工具

据说 Google 的数据中心里运行着20多亿个容器，而且 Google 十年前就开始使用容器技术

从2014年第一个版本发布以来，Kubernetes 迅速获得开源社区的追捧，包括 Red Hat、VMware、Canonical 在内的很多有影响力的公司加入到开发和推广的阵营。目前 Kubernetes 已经成为发展最快、市场占用率最高的容器编排引擎产品

Kubernets 一直在快速地开发和迭代。本书我们将以v1.7和v1.8为基础学习 Kubernetes。我们会讨论 Kubernetes 重要的概念和架构，学习 Kubernetes 如何编排容器，**包括优化资源利用、高可用、滚动更新、网络插件、服务发现、监控、数据管理、日志管理等**

### 2. Cluster

> Cluster 是计算、存储和网络资源的集合，Kubernetes 利用这些资源运行各种基于容器的应用

### 3. Master

> Master 是 Cluster 的大脑，它的主要职责是调度，即决定将应用放在哪里运行

Master 运行 Linux 操作系统，可以是物理机或者虚拟机，为了实现高可用，可以运行多个Master

### 4. Node

> Node 的职责是运行容器应用。Node 由 Master 管理，Node 负责监控并汇报容器的状态，同时根据 Master 的要求管理容器的生命周期

Node 运行在 Linux 操作系统上，可以是物理机或者是虚拟机，如图所示，我们可以使用 kubectl get nodes 获取所有的 node 信息

![image-20231119131809779](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311191318656.png)

### 5. Pod

> Pod 是 Kubernetes 的最小工作单元，每个 Pod 包含一个或者多个容器，Pod 中的容器会作为一个整体被 Master 调度到另一个 Node 上运行

#### (1) 引入 Pod 的目的

Kubernetes 引入 Pod 主要基于下面两个目的：

##### 可管理性

有些容器天生是需要紧密联系，一起工作。Pod 提供了比容器更层次的抽象，将它们封装到一个部署单元中。Kubernetes 以 Pod 为最小单元进行调度、扩展、共享资源、管理生命周期

##### 通信和资源共享

Pod 中所有容器使用同一个网络 namespace，即相同的IP地址和Port空间。它们可以直接用localhost通信。同样的，这些容器可以共享存储，当 Kubernetes 挂载volume到Pod，本质上是将volume挂载到Pod中的每一个容器

#### (2) Pod 的使用方式

Pod 有两种使用方式：

##### 运行单一容器

one-container-per-Pod 是 Kubernetes 最常见的模型，这种情况下，只是将单个容器简单封装成 Pod。即便是只有一个容器，Kubernetes 管理的也是 Pod 而不是直接管理容器

##### 运行多个容器

问题在于：那些容器应该放在一个 Pod 中？

答案是：这些容器联系必须非常紧密，而且需要直接共享资源

举个例子，如图所示，这个 Pod 包含两个容器：一个是 File Puller，一个是 Web Server

![image-20231119132908507](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311191329632.png)

File Puller 会定期从外部的 Content Manager 中拉取最新的文件，将其存放在共享的 volume 中。Web Server 从 volume 读取文件，响应 Consumer 的请求

这两个容器是紧密协作的，它们一起为 Consumer 提供最新的数据；同时它们也通过 volume 共享数据，所以放在一个 Pod 是合适的

再来看一个反例：是否需要将 Tomcat 和 MySQL 放到一个 Pod 中

Tomcat 从 MySQL 读取数据，它们之间需要协作，但还不至于需要放到一个 Pod 中一起部署、一起启动、一起停止。同时它们之间是通过 JDBC 交换数据，并不是直接共享存储，所以放到各自的 Pod 中更合适

### 6. Controller

> Kubernetes 通常不会直接创建 Pod，而是通过 Controller 来管理 Pod 的。Controller 中定义了Pod 的部署特性，比如有几个副本、在什么样的 Node 上运行等

为了满足不同的业务场景，Kubernetes 提供了多种 Controller，包括 Deployment、Replicaset、DaemonSet、StatefuleSet、Job 等，我们逐一讨论

#### (1) Deployment

Deployment 是最常用的 Controller，Deployment 可以管理 Pod 的多个副本，并确保 Pod 按照期望的状态运行

#### (2) ReplicaSet

ReplicaSet 实现了 Pod 的多副本管理。**使用 Deployment 时会自动创建 ReplicaSet，也就是说 Deployment 是通过 ReplicaSet 来管理 Pod 的多个副本的，我们通常不需要直接使用 ReplicaSet**

#### (3) DaemonSet

DaemonSet 用于每个 Node 最多只运行一个 Pod 副本的场景。**DaemonSet 通常用于运行 daemon**

#### (4) StatefuleSet

StatefuleSet 能够保证 Pod 的每个副本在整个生命周期中名称是不变的，而其他 Controller 不提供这个功能。当某个 Pod 发生故障需要删除并重新启动时，Pod 的名称会发生变化，同时 StatefuleSet 会保证副本按照固定的顺序启动、更新或者删除

#### (5) Job

Job 用于运行结束就删除的应用，而其他 Controller 中的 Pod 通常是长期持续运行

### 7. Service

> Kubernetes Service 定义了外界访问一组特定 Pod 的方式。Service 有自己的IP和端口，Service 为 Pod 提供了负载均衡

Kubernetes 运行容器 (Pod) 与访问容器 (Pod) 这两项任务分别由 Controller 和 Service 执行

### 8. Namespace

如果有多个用户或者项目组使用同一个Kubernetes Cluster，如何将他们创建的 Controller、Pod 等资源分开呢？

答案是 Namespace

> Namespace 可以将一个物理的 Cluster 逻辑上划分成多个虚拟 Cluster，每个 Cluster 就是一个 Namespace。不同 Namespace 里的资源是完全隔离的

Kubernetes 默认创建了四个 Namespace，如图所示

![image-20231119163656747](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311191636948.png)

- default：创建资源时如果不指定，将会放到这个 Namespace 中
- kube-node-lease：各个节点的心跳
- kube-public：可以公开访问的数据，如存放了集群信息的 ConfigMap。比如我们使用命令 kubectl cluster-info 拿到集群信息，就是来自这个 Namespace 下的数据
- kube-system：Kubernetes 自己创建的系统资源将放到这个 Namespace 中

## 第2章 部署 Kubernetes Cluster

部署 Kubernetes Cluster 有 kubeadm 和 二进制部署两种方式，前者比较简单，本章将以介绍前者

### 1. kubeadm

kubeadm是官方社区推出的一个用于快速部署kubernetes集群的工具。

这个工具能通过两条指令完成一个kubernetes集群的部署：

```
# 创建一个 Master 节点
$ kubeadm init

# 将一个 Node 节点加入到当前集群中
$ kubeadm join <Master节点的IP和端口 >
```

### 2. 安装要求

在开始之前，部署Kubernetes集群机器需要满足以下几个条件：

- 一台或多台机器，操作系统 CentOS7.x-86_x64
- 硬件配置：2GB或更多RAM，2个CPU或更多CPU，硬盘30GB或更多
- 可以访问外网，需要拉取镜像，如果服务器不能上网，需要提前下载镜像并导入节点
- 禁止swap分区

### 3. 准备环境

| 角色   | IP           |
| ------ | ------------ |
| master | 192.168.1.11 |
| node1  | 192.168.1.12 |
| node2  | 192.168.1.13 |

```
# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

# 关闭selinux
sed -i 's/enforcing/disabled/' /etc/selinux/config  # 永久
setenforce 0  # 临时

# 关闭swap
swapoff -a  # 临时
sed -ri 's/.*swap.*/#&/' /etc/fstab    # 永久

# 根据规划设置主机名
hostnamectl set-hostname <hostname>

# 在master添加hosts
cat >> /etc/hosts << EOF
192.168.44.146 k8smaster
192.168.44.145 k8snode1
192.168.44.144 k8snode2
EOF


cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=www server-csr.json | cfssljson -bare server


# 将桥接的IPv4流量传递到iptables的链
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system  # 生效

# 时间同步
yum install ntpdate -y
ntpdate time.windows.com

https://www.cnblogs.com/lijiale/p/17255089.html
```

### 4. 所有节点安装Docker/kubeadm/kubelet

Kubernetes默认CRI（容器运行时）为Docker，因此先安装Docker。

#### (1) 安装Docker

```
$ wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
$ yum -y install docker-ce-18.06.1.ce-3.el7
$ systemctl enable docker && systemctl start docker
$ docker --version
Docker version 18.06.1-ce, build e68fc7a
```

```
$ cat > /etc/docker/daemon.json << EOF
{
  "registry-mirrors": ["https://sh2j2unb.mirror.aliyuncs.com"]
}
EOF
```

#### (2) 添加阿里云yum软件源

```
$ cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

#### (3) 安装kubeadm，kubelet和kubectl

由于版本更新频繁，这里指定版本号部署：

```
$ yum install -y kubelet-1.18.0 kubeadm-1.18.0 kubectl-1.18.0
$ systemctl enable kubelet
```

### 5. 部署Kubernetes Master

在192.168.31.61（Master）执行。

```
$ kubeadm init \
  --apiserver-advertise-address=192.168.44.146 \
  --image-repository registry.aliyuncs.com/google_containers \
  --kubernetes-version v1.18.0 \
  --service-cidr=10.96.0.0/12 \
  --pod-network-cidr=10.244.0.0/16
```

由于默认拉取镜像地址k8s.gcr.io国内无法访问，这里指定阿里云镜像仓库地址。

使用kubectl工具：

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
$ kubectl get nodes
```

### 6. 加入Kubernetes Node

在192.168.1.12/13（Node）执行。

向集群添加新节点，执行在kubeadm init输出的kubeadm join命令：

```
$ kubeadm join 192.168.1.11:6443 --token esce21.q6hetwm8si29qxwn \
    --discovery-token-ca-cert-hash sha256:00603a05805807501d7181c3d60b478788408cfe6cedefedb1f97569708be9c5
```

默认token有效期为24小时，当过期之后，该token就不可用了。这时就需要重新创建token，操作如下：

```
kubeadm token create --print-join-command
```

### 7. 部署CNI网络插件

```
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

默认镜像地址无法访问，sed命令修改为docker hub镜像仓库。

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

[https://blog.csdn.net/weixin_43784564/article/details/124721760]

kubectl get pods -n kube-system
NAME                          READY   STATUS    RESTARTS   AGE
kube-flannel-ds-amd64-2pc95   1/1     Running   0          72s
```

### 8. 测试kubernetes集群

在Kubernetes集群中创建一个pod，验证是否正常运行：

```
$ kubectl create deployment nginx --image=nginx
$ kubectl expose deployment nginx --port=80 --type=NodePort
$ kubectl get pod,svc
```

访问地址：http://NodeIP:Port  



## 第3章 Kubernetes 架构

Kubernetes Cluster 由 Master 和 Node 组成，节点上运行着若干 Kubernetes 服务

### 1. Master 节点

Master 是 Kubernetes Cluster 的大脑，运行着 Daemon 服务包括 kube-apiserver、kube-scheduler、kube-controller-manager、etcd 和 Pod 网络（例如flannel），如图所示

![image-20231119171006158](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202311191710300.png)

#### (1) API Server (kube-apiserver)

API Server 提供 HTTP/HTTPS RESTful API，即 Kubernetes API。API Server 是 Kubernetes Cluster 的前端接口，各种客户端工具(CLI或UI) 以及 Kubernetes 其他组件可以通过它管理 Cluster 的各种资源

#### (2) Scheduler (kube-scheduler)

Scheduler 负责决定将 Pod 放在哪个 Node 上运行。Scheduler 在调度时会充分考虑 Cluster 的拓扑结构，当前各个节点的负载，以及应用对高可用、性能、数据亲和性的需求

#### (3) Controller Manager (kube-controller-manager)

Controller Manager 负责管理 Cluster 各种资源，保证资源处于预期的状态。Controller Manager 由多种 controller 组成，包括 replication controller、endpoint controller、namespace controller、serviceaccounts controller 等

不同的 controller 管理不同的资源，例如，replication controller 管理 Deployment、Statefulset、DaemonSet 的生命周期，namespace controller 管理 Namespace 资源

#### (4) etcd

etcd 负责保存 Kubernetes Cluster 的配置信息和各种资源的状态信息。当数据发生变化时，etcd 会快速地通知 Kubernetes 相关组件

#### (5) Pod 网络

Pod 要能够相互通信，Kubernetes Cluster 必须部署 Pod 网络，flannel 是其中一个可选方案









