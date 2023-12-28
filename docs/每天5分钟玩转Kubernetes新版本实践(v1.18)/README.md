[toc]



# 每天5分钟玩转Kubernetes新版本实践(v1.18)

## K8S 中文文档

http://docs.kubernetes.org.cn/

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#run

https://geekhour.net/2023/12/23/kubernetes/

## 第4章 Kubernetes 架构

### 1. 参数 replicas 和 kubectl run

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



### 2. Lens 添加 Clusters

下载 Lens 桌面端后，打开 File-->Add Cluster，可以看到有一个 Add Clusters from kubeconfig

![image-20231228200841340](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312282008469.png)

在 master 终端中输入``` kubectl config view --minify --raw```查看 kubeconfig 信息

![image-20231228201017498](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312282010584.png)

将这些配置信息复制到 Add Clusters from kubeconfig 的空白区域，点击 Add clusters 按钮就可以添加cluster

![image-20231228201233157](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312282012240.png)

![image-20231228201340063](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312282013142.png)