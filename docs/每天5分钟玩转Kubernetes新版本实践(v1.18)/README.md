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

### jobs.batch "hello-1703776080-rxs79" not found

为什么会出现报错： 

```she
[root@k8s-master kubernetes-demo]# kubectl describe job/hello-1703776080-rxs79
Error from server (NotFound): jobs.batch "hello-1703776080-rxs79" not found
```

![image-20231228230928932](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312282309064.png)

![image-20231228231144763](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312282311862.png)