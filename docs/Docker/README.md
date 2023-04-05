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



