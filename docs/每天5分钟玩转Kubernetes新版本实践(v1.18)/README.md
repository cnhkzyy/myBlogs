[toc]



# 每天5分钟玩转Kubernetes新版本实践(v1.18)

## K8S 中文文档

http://docs.kubernetes.org.cn/

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#run

https://geekhour.net/2023/12/23/kubernetes/



## Lens 添加 Clusters

Lens 下载地址：https://k8slens.dev/

Lens 官方文档：https://docs.k8slens.dev/getting-started/add-cluster/

下载 Lens 桌面端后，打开 File-->Add Cluster，可以看到有一个 Add Clusters from kubeconfig

![image-20231228200841340](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312282008469.png)

在 master 终端中输入``` kubectl config view --minify --raw```查看 kubeconfig 信息

![image-20231228201017498](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312282010584.png)

将这些配置信息复制到 Add Clusters from kubeconfig 的空白区域，点击 Add clusters 按钮就可以添加cluster

![image-20231228201233157](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312282012240.png)

![image-20231228201340063](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312282013142.png)



## 第4章 Kubernetes 架构

### 参数 replicas 和 kubectl run

在P23页的例子中，使用--replicas=2可以创建2个副本pod

![image-20231228172225652](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312281722768.png)

事实上，在v1.18版本中，--replicas=2 并没有创建2个副本，而是给出了一段提示："Flag --replicas has been deprecated, has no effect and will be removed in the future."

如果细心留意还会发现，**kubectl run 并没有创建deployment**， 这个问题有人提过issue：https://github.com/caicloud/kube-ladder/issues/35

![image-20231228172346760](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312281723816.png)

官方文档对 kubectl run 的解释是：

> <font size=5>run</font>
>
> Create and run a particular image in a pod.
>
> <font size=5>Usage</font>
>
> ```shell
> $ kubectl run NAME --image=image [--env="key=value"] [--port=port] [--dry-run=server|client] [--overrides=inline-json] [--command] -- [COMMAND] [args...]
> ```
>
> <font size=5>Flags</font>
>
> Flags 里并无 replicas
>
> <font size=5>Examples</font>
>
> **Start a nginx pod**
>
> ```shell
> kubectl run nginx --image=nginx
> ```
>
> **Start a hazelcast pod and let the container expose port 5701**
>
> ```shell
> kubectl run hazelcast --image=hazelcast/hazelcast --port=5701
> ```
>
> **Start a hazelcast pod and set environment variables "DNS_DOMAIN=cluster" and "POD_NAMESPACE=default" in the container**
>
> ```shell
> kubectl run hazelcast --image=hazelcast/hazelcast --env="DNS_DOMAIN=cluster" --env="POD_NAMESPACE=default"
> ```

即使使用按照官方文档所说，使用```kubectl create deployment nginx-deployment --image=nginx:1.7.9 --replicas=2``` 也是不能创建副本的，提示没有--replicas这个参数

![image-20231228182600722](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312281826800.png)

![image-20231228182331421](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312281823552.png)

如果去掉 --replicas 参数，可以在default namespace 下成功创建 deployment-->replicaset-->pod

![image-20231228182820040](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312281828122.png)

**如果需要扩展deployment的副本数，可以使用```kubectl scale```命令**

![image-20231228184913332](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312281849407.png)

![image-20231228185002726](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312281850804.png)

还可以使用```kubectl edit deployment nginx-deployment```编辑yaml文件，这时候将 replicas 改为 2， 修改之后，K8S会自动将副本数量修改为2

![image-20231228185501978](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312281855074.png)

![image-20231228185907687](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312281859761.png)



## 第5章 运行应用

### apiVersion: extensions/v1beta1

在第P30-P31页，当使用```kubectl apply -f nginx.yaml```时，会出现报错：error: unable to recognize "nginx.yaml": no matches for kind "Deployment" in version "extensions/v1beta1"

![image-20231228204236782](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312282042882.png)

![image-20231228204534054](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312282045150.png)

实际上，我在本地运行的结果报错了

![image-20231228204613721](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312282046804.png)

这个错误通常是由Kubernetes API版本不匹配引起的。在不同的Kubernetes版本中，API可能会有所不同，包括支持的资源种类和对象类型。因此，如果我们在较旧的Kubernetes版本中使用了较新的API对象类型，就会出现无法识别对象的错误。例如，如果我们在Kubernetes 1.15版本中尝试创建一个Deployment对象，但是Deployment对象是在1.16版本中引入的，那么Kubernetes就无法识别这个对象，从而导致错误的发生

此时，可以更新 K8S 版本或者 apiVersion 

![image-20231228205804658](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312282058772.png)

### deployment: selector

更新后再次部署应用，发现还会报错：error: error validating "nginx.yaml": error validating data: ValidationError(Deployment.spec): missing required field "selector" in io.k8s.api.apps.v1.DeploymentSpec; if you choose to ignore these errors, turn validation off with --validate=false

![image-20231228205901481](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312282059570.png)

增加一个 selector，可以看到成功了

![image-20231228210610514](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312282106600.png)

![image-20231228210746567](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312282107665.png)

### node 故障后的恢复时间

约为6分钟，参考这篇文章：https://www.cnblogs.com/gaoyuechen/p/16529774.html

>在默认配置下，k8s节点故障时node notready，工作负载的调度周期约为6分钟，
>
>参数概念：
>
>node-monitor-period
>节点控制器(node controller) 检查每个节点的间隔，默认5秒。
>
>node-monitor-grace-period
>节点控制器判断节点故障的时间窗口, 默认40秒。即40 秒没有收到节点消息则判断节点为故障。
>
>pod-eviction-timeout
>当节点故障时，kubelet允许pod在此故障节点的保留时间，默认300秒。即当节点故障5分钟后，kubelet开始在其他可用节点重建pod。
>
>5+40+300 ≈ 6分钟

![image-20231228212751142](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312282127273.png)

一段时间后，状态为 Terminating 的 Pod 会消失

![image-20231228213533447](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312282135601.png)

### Job 类型的资源执行完任务后

在第P41页说，因为Pod执行完毕后容器已经退出，需要用--show-all才能查看Completed状态的Pod。事实上，在v1.18版本上直接可以使用 kubectl get pod 查看

![image-20231228223311479](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312282233592.png)

![image-20231228223339854](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312282233926.png)

### enable CronJob 功能

第P46页有提到，在部署CronJob资源的时候，会因为K8S默认配置没有开启CronJob功能而导致的报错，在v1.18中并没有遇到，且配置文件/etc/kubernetes/manifests/kube-apiserver.yaml 中并未找到相关的配置项

![image-20231228225435480](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312282254606.png)

![image-20231228225501315](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312282255413.png)



## 第6章 通过 Service 访问 Pod

### selector: run: httpd

P50页在为httpd创建Service的时候，配置文件如图所示，采用了```run: httpd```的这种写法

![image-20231229221617284](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312292216379.png)

在新版本v1.18上操作后，发现 curl cluster ip: port 显示连接拒绝，查看service详情发现Endpoints中并没有显示三个Pod的IP和端口

![image-20231229221755326](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312292217455.png)

发现当selector改成```app: httpd```后，curl命令的请求结果和Endpoints都正常了

![image-20231229222448863](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312292224934.png)



### Cluster IP 底层实现

使用```iptables-save```命令查看当前节点的 iptables 规则

```shell
-A KUBE-SERVICES ! -s 10.244.0.0/16 -d 10.103.38.177/32 -p tcp -m comment --comment "default/httpd-svc: cluster IP" -m tcp --dport 8080 -j KUBE-MARK-MASQ
-A KUBE-SERVICES -d 10.103.38.177/32 -p tcp -m comment --comment "default/httpd-svc: cluster IP" -m tcp --dport 8080 -j KUBE-SVC-RL3JAE4GN7VOGDGP

#1/3概率转发到 KUBE-SEP-TEXJ5FRT6GSZ3TBS
-A KUBE-SVC-RL3JAE4GN7VOGDGP -m comment --comment "default/httpd-svc:" -m statistic --mode random --probability 0.33333333349 -j KUBE-SEP-TEXJ5FRT6GSZ3TBS
#1/3概率转发到 KUBE-SEP-MKVKRXXYH7GSXE6O
-A KUBE-SVC-RL3JAE4GN7VOGDGP -m comment --comment "default/httpd-svc:" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-MKVKRXXYH7GSXE6O
#1/3概率转发到 KUBE-SEP-ZIZBJ3RYTU5VVQLX
-A KUBE-SVC-RL3JAE4GN7VOGDGP -m comment --comment "default/httpd-svc:" -j KUBE-SEP-ZIZBJ3RYTU5VVQLX

-A KUBE-SEP-TEXJ5FRT6GSZ3TBS -s 10.244.2.137/32 -m comment --comment "default/httpd-svc:" -j KUBE-MARK-MASQ
-A KUBE-SEP-TEXJ5FRT6GSZ3TBS -p tcp -m comment --comment "default/httpd-svc:" -m tcp -j DNAT --to-destination 10.244.2.137:80
-A KUBE-SEP-MKVKRXXYH7GSXE6O -s 10.244.2.138/32 -m comment --comment "default/httpd-svc:" -j KUBE-MARK-MASQ
-A KUBE-SEP-MKVKRXXYH7GSXE6O -p tcp -m comment --comment "default/httpd-svc:" -m tcp -j DNAT --to-destination 10.244.2.138:80
-A KUBE-SEP-ZIZBJ3RYTU5VVQLX -s 10.244.2.139/32 -m comment --comment "default/httpd-svc:" -j KUBE-MARK-MASQ
-A KUBE-SEP-ZIZBJ3RYTU5VVQLX -p tcp -m comment --comment "default/httpd-svc:" -m tcp -j DNAT --to-destination 10.244.2.139:80
```



### 跨namespace 访问 httpd2-svc 失败

httpd2.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name:  httpd2
  namespace: kube-public
spec:
  selector:
    matchLabels:
      app: httpd2
  replicas: 3
  template:
    metadata:
      labels:
        app: httpd2
    spec:
      containers:
      - name:  httpd2
        image:  httpd      
        ports:
        - containerPort:  80

---

apiVersion: v1
kind: Service
metadata:
  name: httpd2-svc
  namespace: kube-public
spec:
  selector:
    app: httpd2
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 80
```

第一次试验的时候发现在busybox容器中使用```wget httpd2-svc.kube-public:8080```访问报错```wget: bad address 'httpd-svc.kube-public:8080'```

后面试验成功了，怀疑是当时Pod中的容器还没有启动。现在成功了，可以看到三个Pod中的容器都已经启动

![image-20240103224154127](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202401032242200.png)

![image-20240103224233015](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202401032242109.png)

### EXTERNAL-IP

P56 页上说 EXTERNAL-IP 为 nodes，但在v1.18版本上发现这个值为none

![image-20240103230206035](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202401032302177.png)

![image-20240103230339579](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202401032303661.png)

### NodePort 底层实现

```shell
-A KUBE-NODEPORTS -p tcp -m comment --comment "default/httpd-svc:" -m tcp --dport 31544 -j KUBE-MARK-MASQ
-A KUBE-NODEPORTS -p tcp -m comment --comment "default/httpd-svc:" -m tcp --dport 31544 -j KUBE-SVC-RL3JAE4GN7VOGDGP

-A KUBE-SVC-RL3JAE4GN7VOGDGP -m comment --comment "default/httpd-svc:" -m statistic --mode random --probability 0.33333333349 -j KUBE-SEP-NGX2GKBZ734ZVF5H
-A KUBE-SVC-RL3JAE4GN7VOGDGP -m comment --comment "default/httpd-svc:" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-Z7LSLPUABQUNUITA
-A KUBE-SVC-RL3JAE4GN7VOGDGP -m comment --comment "default/httpd-svc:" -j KUBE-SEP-WUZQJLKL3EGKLRUO

-A KUBE-SEP-NGX2GKBZ734ZVF5H -s 10.244.2.151/32 -m comment --comment "default/httpd-svc:" -j KUBE-MARK-MASQ
-A KUBE-SEP-NGX2GKBZ734ZVF5H -p tcp -m comment --comment "default/httpd-svc:" -m tcp -j DNAT --to-destination 10.244.2.151:80
-A KUBE-SEP-Z7LSLPUABQUNUITA -s 10.244.2.152/32 -m comment --comment "default/httpd-svc:" -j KUBE-MARK-MASQ
-A KUBE-SEP-Z7LSLPUABQUNUITA -p tcp -m comment --comment "default/httpd-svc:" -m tcp -j DNAT --to-destination 10.244.2.152:80
-A KUBE-SEP-WUZQJLKL3EGKLRUO -s 10.244.3.34/32 -m comment --comment "default/httpd-svc:" -j KUBE-MARK-MASQ
-A KUBE-SEP-WUZQJLKL3EGKLRUO -p tcp -m comment --comment "default/httpd-svc:" -m tcp -j DNAT --to-destination 10.244.3.34:80

```



## 第8章 Health Check

### Health Check 在 Scale Up 中的应用

在 P70 页举例的时候，并未说明应用的判断逻辑中Check Database 这部分应该怎么部署，因此这个例子很难再本地调的通

![image-20240107225323049](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202401072253110.png)

![image-20240107225435289](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202401072254397.png)

可以自己先部署2个副本的redis，然后通过flask访问redis，falsk打包成新的镜像beck123/myweb，在k8s中部署，目录结构是：

![image-20240107225720298](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202401072257372.png)

**app.py**

```python
import time
import random
import redis
from flask import Flask

app = Flask(__name__)




@app.route('/healthy')
def check_redis():
    try:
        hosts= ["192.168.1.125", "192.168.1.155"]
        host = random.choice(hosts)
        r = redis.Redis(host=host, port=30001)
        result = r.ping()
        print(f"host: {host}, result: {result}")

        if result == True:
            return "成功连接到redis", 200
        else:
            return "无法连接到Redis", 400
    except Exception as e:
        return f"连接错误: {str(e)}", 503


app.run()
```

**docker-compose.yaml**

```yaml
# yaml 配置
version: '3'
services:
  web:
    build: .
    ports:
     - "5000:5000"
    image: beck123/myweb
```

**Dockerfile**

```shell
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP app.py
ENV FLASK_RUN_HOST 0.0.0.0
RUN apk add gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
COPY . .
CMD ["flask", "run"]
```

**myweb.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name:  myweb
spec:
  selector:
    matchLabels:
      app: myweb
  replicas: 2
  template:
    metadata:
      labels:
        app: myweb
    spec:
      containers:
      - name:  myweb
        image:  beck123/myweb  
        readinessProbe:
          httpGet:
            scheme: HTTP 
            path: /healthy 
            port: 5000
          initialDelaySeconds: 10
          periodSeconds: 5 

---
apiVersion: v1
kind: Service
metadata:
  name: myweb-svc
spec:
  selector:
    app: myweb
  ports:
  - protocol: TCP
    port: 5000
    targetPort: 5000
```

**redis.yaml**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: redis-conf
data:
  redis.conf: |
        bind 0.0.0.0
        port 6379
        #requirepass 111111 #设置认证密码
        appendonly yes
        cluster-config-file nodes-6379.conf
        pidfile /redis/log/redis-6379.pid
        cluster-config-file /redis/conf/redis.conf
        dir /redis/data/
        logfile /redis/log/redis-6379.log
        cluster-node-timeout 5000
        protected-mode no


--- 
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
spec:
  replicas: 2
  serviceName: redis
  selector:
    matchLabels:
      name: redis
  template:
    metadata:
      labels:
        name: redis
    spec:
      initContainers:
      - name: init-redis
        image: busybox #初始化容器镜像
        command: ['sh', '-c', 'mkdir -p /redis/log/;mkdir -p /redis/conf/;mkdir -p /redis/data/']
        volumeMounts:
        - name: data
          mountPath: /redis/
      containers:
      - name: redis
        image: redis
        imagePullPolicy: IfNotPresent
        command:
        - sh
        - -c
        - "exec redis-server /redis/conf/redis.conf"
        ports:
        - containerPort: 6379
          name: redis
          protocol: TCP
        volumeMounts:
        - name: redis-config
          mountPath: /redis/conf/
        - name: data
          mountPath: /redis/
      volumes:
      - name: redis-config
        configMap:
          name: redis-conf
      - name: data
        hostPath:
          path: /app/ #挂载宿主机目录
      

--- 
kind: Service
apiVersion: v1
metadata:
  labels:
    name: redis
  name: redis
spec:
  type: NodePort
  ports:
  - name: redis
    port: 6379
    targetPort: 6379
    nodePort: 30001
  selector:
    name: redis
```

**requirements.txt**

```shell
flask
redis
```

部署成功之后，可以从2个myweb的Pod 日志中查看到 Readiness 探测的执行情况

![image-20240107230731581](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202401072307680.png)

