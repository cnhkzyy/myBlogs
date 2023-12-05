[toc]

# Pipeline

## 第1章 认识 Pipeline

### 1. 什么是 Pipeline

- Pipeline是Jenkins的核心功能，提供一组可扩展的工具
- 通过Pipeline 的DSL语法可以完成从简单到复杂的交付流水线实现
- jenkins的Pipeline是通过Jenkinsfile（文本文件）来实现的
- 这个文件可以定义Jenkins的执行步骤，例如检出代码

### 2. 为什么使用 Pipeline

本质上，jenkins是一个自动化引擎，它支持许多自动模式。流水线向Jenkins添加了一组强大的工具，支持用例、简单的持续集成到全面的持续交付流水线。 通过对一系列的发布任务建立标准的模板，用户可以利用更多流水线的特性，比如：

- 代码化: 流水线是在代码中实现的，通常会存放到源代码控制，使团队具有编辑、审查和更新他们项目的交付流水线的能力
- 耐用性：流水线可以从Jenkins的master节点重启后继续运行
- 可暂停的：流水线可以由人功输入或批准继续执行流水线
-  解决复杂发布： 支持复杂的交付流程。例如循环、并行执行
- 可扩展性： 支持扩展DSL和其他插件集成

### 3. Jenkinsfile

- Jenkinsfile使用两种语法进行编写，分别是声明式和脚本式
- 声明式和脚本式的流水线从根本上是不同的
- 声明式是jenkins流水线更友好的特性
- 脚本式的流水线语法，提供更丰富的语法特性
- 声明式流水线使编写和读取流水线代码更容易设计

## 第2章 安装 Pipeline 插件

### 1. 插件管理

安装 Pipeline

![image-20231205234643172](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312052346228.png)

安装 Blue Ocean

![image-20231205234710676](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312052347785.png)

### 2. jenkinsci/blueocean 镜像

建议使用的Docker映像是[`jenkinsci/blueocean` image](https://hub.docker.com/r/jenkinsci/blueocean/)(来自 the [Docker Hub repository](https://hub.docker.com/))。 该镜像包含当前的[长期支持 (LTS) 的Jenkins版本](https://www.jenkins.io/download) （可以投入使用） ，捆绑了所有Blue Ocean插件和功能。这意味着你不需要单独安装Blue Ocean插件

```shell
docker run \
  -u root \
  --rm \
  -d \
  -p 8080:8080 \
  -p 50000:50000 \
  -v ~/jenkins-data:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkinsci/blueocean
```





