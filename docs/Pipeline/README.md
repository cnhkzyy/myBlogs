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

## 第2章 Pipeline 插件

### 1. 安装 Jenkins

建议使用jenkins最新的docker映像jenkins/jenkins:2.430-jdk21，否则很多插件因为版本太低将会不能安装

注意：jenkins 要预留比较大的内存(1G以上），否则在容器启动后，输入初始密码点击下一步就会出现容器自动退出

```shell
docker run \
-u root \
-d \
-p 8080:8080 \
-p 50000:50000 \
-v ~/jenkins-data:/var/jenkins_home \
-v /var/run/docker.sock:/var/run/docker.sock \
--name jenkins \
jenkins/jenkins:2.430-jdk21
```



### 2. 配置镜像源

```shell
cd ~/jenkins-data
vim hudson.model.UpdateCenter.xml

<?xml version='1.1' encoding='UTF-8'?>
<sites>
  <site>
    <id>default</id>
    <url>https://mirrors.aliyun.com/jenkins/updates/update-center.json</url>
  </site>
</sites>%    

docker restart jenkins
```



### 3. 安装 Pipeline

![image-20231205234643172](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312052346228.png)



## 第3章 Pipeline 语法

这里介绍的 Pipeline 语法是声明式，如果有兴趣，可以看下脚本式

### 1. agent

> 节点是一个机器，可以是Jenkins的master节点也可以是slave节点。通过agent指定当前job运行的机器
>
> 参数：
>
> - any 在任何可用的节点上执行 pipeline
> - none 没有指定agent的时候默认是none
> - label 在指定标签上的节点上运行 pipeline

```groovy
pipeline{
  	agent {
    		label 'node-1'
  	} 
		stages{
   			//    
		}
}
```

### 2. Stages

> 包含一系列一个或多个 stage 指令, 建议 stages 至少包含一个 stage 指令用于连续交付过程的每个离散部分,比如构建, 测试, 和部署

```groovy
pipeline {
    agent any
    stages { 
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

### 3. stage

> stage定义了在整个流水线的执行任务的概念性的不同的阶段。例如： GetCode、Build、Test、Deploy、CodeScan每个阶段

```groovy
pipeline{
		agent any
		stages{
    		stage("GetCode"){
        		//steps  
    		}
        
    		stage("build"){
       			//step
    		}
		}
}
```

### 4. steps

> step是每个阶段中要执行的每个步骤。例如： 在执行GetCode的时候需要判断用户提供的参数srcType的值是Git还是svn

```groovy
pipeline{
		agent any
		stages{
        stage("GetCode"){
            	steps{ 
                	sh "ls "    //step
            	}
        }    
    }
}
```



### 5. parameters

> parameters指令提供用户在触发Pipeline时的参数列表。这些参数值通过该params对象可用于Pipeline步骤
> 目前只支持 booleanParam, choice, credentials, file, text, password, run, string这几种参数类型，其他高级参数化类型还需等待社区支持

![image-20231217094618595](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312170946801.png)

#### 例1: string

```groovy
pipeline {
    agent any
    parameters {
        string(name: 'user', defaultValue: 'John', description: 'A user that triggers the pipeline')
    }
    stages {
        stage('Trigger pipeline') {
            steps {
                echo "Pipeline triggered by ${params.USER}"
            }
        }
    }
}
```

#### 例2: choice

```groovy 
pipeline {
    agent any


    parameters {
        choice(name:'PerformMavenRelease',choices:'False\nTrue',description:'desc')
    }

    
    stages {
        stage('get parameters') {
            steps {
                echo "${params.PerformMavenRelease}"
            }
        }        
    }
}
```



### 6. environment

> nvironment指令指定一系列键值对，这些键值对将被定义为所有step或stage-specific step的环境变量，具体取决于environment指令在Pipeline中的位置
>
> 该指令支持一种特殊的方法credentials()，可以通过其在Jenkins环境中的标识符来访问预定义的凭据。对于类型为“Secret Text”的凭据，该 credentials()方法将确保指定的环境变量包含Secret Text内容；对于“标准用户名和密码”类型的凭证，指定的环境变量将被设置为username:password

#### 例1: pipeline 级别

```groovy
//在“pipeline”级别：
pipeline {
    agent any
    
    environment {
        JENKINS_SERVER = 'http://192.168.1.126:8080'
    }
    
    stages {
        stage('Example') {
            steps {
                echo "${JENKINS_SERVER}"
            }
        }
    }
}
```



#### 例2: stage 级别

```groovy
//在“stage”级别：
pipeline {
    agent any
    stages {
        stage ('build') {
            environment {
                OUTPUT_PATH = './outputs/'
            }
          
          	steps {
              	echo "${OUTPUT_PATH}"
          	}
        }
    }
}
```





### 7. script

> 此步骤用于将脚本化流水线语句添加到声明式流水线中，从而提供更多功能。此步骤必须包括在“stage”级别
>
> 脚本块可以多次用于不同的项目。这些块使您可以扩展Jenkins功能，并可以实现为共享库

```groovy
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'

                script {
                    def browsers = ['chrome', 'firefox']
                    for (int i = 0; i < browsers.size(); ++i) {
                        echo "Testing the ${browsers[i]} browser"
                    }
                }
            }
        }
    }
}
```



### 8. when

> when 指令允许流水线根据给定的条件决定是否应该执行阶段。 when 指令必须包含至少一个条件。 如果`when` 指令包含多个条件, 所有的子条件必须返回True，阶段才能执行。 这与子条件在 allOf 条件下嵌套的情况相同
>
> 内置条件
>
> - branch 当正在构建的分支与模式给定的分支匹配时，执行这个阶段,这只适用于多分支流水线例如:
>
>   ```groovy
>   when { branch 'master' }
>   ```
>
> - environment 当指定的环境变量是给定的值时，执行这个步骤,例如:
>
>   ```groovy
>   when { environment name: 'DEPLOY_TO', value: 'production' }
>   ```
>
> - expression 当指定的Groovy表达式评估为true时，执行这个阶段, 例如:
>
>   ```groovy
>   when { expression { return params.DEBUG_BUILD } }
>   ```
>
> - not 当嵌套条件是错误时，执行这个阶段,必须包含一个条件，例如:
>
>   ```groovy
>   when { not { branch 'master' } }
>   ```
>
> - allOf 当所有的嵌套条件都正确时，执行这个阶段,必须包含至少一个条件，例如:
>
>   ```groovy
>   when { allOf { branch 'master'; environment name: 'DEPLOY_TO', value: 'production' } }
>   ```
>
> - anyOf 当至少有一个嵌套条件为真时，执行这个阶段,必须包含至少一个条件，例如:
>
>   ```groovy
>   when { anyOf { branch 'master'; branch 'staging' } }
>   ```

#### 例1: branch

```groovy
pipeline {
    agent any
    stages {
        stage('Example Build') {
            steps {
                echo 'Hello World'
            }
        }
        stage('Example Deploy') {
            when {
                branch 'production'
            }
            steps {
                echo 'Deploying'
            }
        }
    }
}
```

#### 例2: environment

```groovy
pipeline {
    agent any
    stages {
        stage('Example Build') {
            steps {
                echo 'Hello World'
            }
        }
        stage('Example Deploy') {
            when {
                branch 'production'
                environment name: 'DEPLOY_TO', value: 'production'
            }
            steps {
                echo 'Deploying'
            }
        }
    }
}
```

#### 例3: expression

```groo
pipeline {
    agent any
    stages {
        stage('Example Build') {
            steps {
                echo 'Hello World'
            }
        }
        stage('Example Deploy') {
            when {
                expression { BRANCH_NAME ==~ /(production|staging)/ }
                anyOf {
                    environment name: 'DEPLOY_TO', value: 'production'
                    environment name: 'DEPLOY_TO', value: 'staging'
                }
            }
            steps {
                echo 'Deploying'
            }
        }
    }
}
```



#### 例4: allOf

```groovy
pipeline {
    agent any
    stages {
        stage('Example Build') {
            steps {
                echo 'Hello World'
            }
        }
        stage('Example Deploy') {
            when {
                allOf {
                    branch 'production'
                    environment name: 'DEPLOY_TO', value: 'production'
                }
            }
            steps {
                echo 'Deploying'
            }
        }
    }
}
```



#### 例5: anyOf

```groovy
pipeline {
    agent any
    stages {
        stage('Example Build') {
            steps {
                echo 'Hello World'
            }
        }
        stage('Example Deploy') {
            when {
                branch 'production'
                anyOf {
                    environment name: 'DEPLOY_TO', value: 'production'
                    environment name: 'DEPLOY_TO', value: 'staging'
                }
            }
            steps {
                echo 'Deploying'
            }
        }
    }
}
```



### 9. post

> 定义一个或多个steps ，这些阶段根据流水线或阶段的完成情况而 运行(取决于流水线中`post`部分的位置). post 支持以下 post-condition 块中的其中之一: always, changed, failure, success, unstable, 和 aborted。这些条件块允许在 post 部分的步骤的执行取决于流水线或阶段的完成状态
>
> - always 无论流水线或者阶段的完成状态
> - changed 只有当流水线或者阶段完成状态与之前不同时
> - failure 只有当流水线或者阶段状态为”failure”运行
> - success 只有当流水线或者阶段状态为”success”运行
> - unstable 只有当流水线或者阶段状态为”unstable”运行。例如：测试失败
> - aborted 只有当流水线或者阶段状态为”aborted “运行。例如：手动取消

```groovy
pipeline {
		agent any
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
  
    post { 
        always { 
            echo 'I will always say Hello again!'
        }
    }
}
```



### 10. trigger

> 构建触发器
>
> - cron 计划任务定期执行构建。
>
>   ```groovy
>   triggers { cron('H */4 * * 1-5') }
>   ```
>
> - pollSCM 与cron定义类似，但是由jenkins定期检测源码变化。
>
>   ```groovy
>   triggers { pollSCM('H */4 * * 1-5') }
>   ```
>
> - upstream 接受逗号分隔的工作字符串和阈值。 当字符串中的任何作业以最小阈值结束时，流水线被重新触发。
>
>   ```groovy
>   triggers { upstream(upstreamProjects: 'job1,job2', threshold: hudson.model.Result.SUCCESS) }
>   ```

```groovy
pipeline {
    agent any
    triggers {
      	cron('H */4 * * 1-5')
    }
    stages {
        stage('Example') {
            steps {
              echo 'Hello World'
            }
        }
    }
}
```



## 第4章 Blue Ocean



### 1. Blue Ocean 简介

到现在为止，我们pipeline就定制好了，但是我们看到，pipeline的界面好像不是太好看，其实还有一个插件专门用于更好的查看流水线效果图，这个插件叫做Blue Ocean

Blue Ocean为开发人员提供了更具乐趣的Jenkins使用方式，它是从基础开始构建的，实现了一种全新的、现代风格的用户界面，有助于任何规模的团队实现持续交付

github地址：https://github.com/jenkinsci/blueocean-plugin

### 2. Blue Ocean 特点

| 特点         | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| 流水线编辑器 | 用于创建贯穿始终的持续交付流水线，是一种直观并可视化的流水线编辑器 |
| 流水线可视化 | 对流水线的可视化表示，提高了全企业范围内持续交付过程的清晰度 |
| 流水线诊断   | 即刻定位自动化问题，无需持续扫描日志或关注多个屏幕           |
| 个性化仪表盘 | 用户可以自定义仪表盘，只显示与自身相关的流水线               |
| 平台集成     | 针对所有特性分支和Pull请求运行流水线，以报告的形式反馈状态给Github，使整个团队可以掌握是否需要执行更改，是否一切保存正常 |



### 3. Blue Ocean 插件

安装 Blue Ocean 插件

![image-20231205234710676](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312052347785.png)



### 4. 安装 Bibucket Server（可跳过）

#### 安装 MySQL

```mysql
# mkdir -p /opt/docker_v/mysql/conf
# cd docker_v/mysql/conf
# vim my.cnf
[mysqld]

skip-grant-tables
# docker run -p 3306:3306 --name mysql -v /opt/docker_v/mysql/conf:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
4ec4f56455ea2d6d7251a05b7f308e314051fdad2c26bf3d0f27a9b0c0a71414
```

在mysql中创建bitbucket数据库（注意一定要加上 SET utf8 COLLATE utf8_bin）

```mysql
[root@k8scloude2 ~]# mysql -h 192.168.1.126
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 27
Server version: 5.7.36 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
mysql>CREATE DATABASE bitbucket CHARACTER SET utf8 COLLATE utf8_bin;
Query OK, 1 row affected (0.12 sec)

mysql> select default_character_set_name from information_schema.SCHEMATA S where schema_name='bitbucket';
+----------------------------+
| default_character_set_name |
+----------------------------+
| utf8                       |
+----------------------------+
1 row in set (0.25 sec)
```

#### 安装 Bitbucket Server

拷贝MySQL驱动，重启 bitbucket 容器

```shell
docker run -d -v /tmp/bitbucket:/var/atlassian/application-data/bitbucket -p 7990:7990 -- name bitbucket clockq/atlassian:bitbucket-6.5.2

cp /opt/mysql-connector-java-5.1.49-bin.jar /tmp/bitbucket/lib
cd /tmp/bitbucket/lib
chmod +x mysql-connector-java-5.1.49-bin.jar

docker restart bitbucket
```

![image-20231216144935235](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312161449333.png)

![image-20231216162114830](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312161621070.png)



### 5. 安装 Gitlab（可跳过）

#### 安装 Gitlab

```shell
$ docker run -d  -p 443:443 -p 80:80 -p 23:22 --name gitlab --restart always -v /data/gitlab/config:/etc/gitlab -v /data/gitlab/logs:/var/log/gitlab -v /data/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce
```

#### 配置

按上面的方式，gitlab容器运行没问题，但在gitlab上创建项目的时候，生成项目的URL访问地址是按容器的hostname来生成的，也就是容器的id。作为gitlab服务器，我们需要一个固定的URL访问地址，于是需要配置gitlab.rb

```shell
docker exec -it gitlab /bin/bash

vim /etc/gitlab/gitlab.rb
external_url '192.168.1.126'

gitlab-ctl reconfigure
```

#### 初始密码

gitlab 启动时间较长，占用内存较大，因此建议预留内存 2GB 以上，且不要关闭交换分区，启动过程中，如果访问页面出现 502，只需要耐心等待。成功加载页面后，初始密码在 /data/gitlab/config/initial_root_password文件中

```shell
[root@k8scloude2 ~]# cd /data/gitlab/config
[root@k8scloude2 config]# ls
gitlab.rb            initial_root_password  ssh_host_ecdsa_key.pub  ssh_host_ed25519_key.pub  ssh_host_rsa_key.pub
gitlab-secrets.json  ssh_host_ecdsa_key     ssh_host_ed25519_key    ssh_host_rsa_key          trusted-certs
[root@k8scloude2 config]# cat initial_root_password 
# WARNING: This value is valid only in the following conditions
#          1. If provided manually (either via `GITLAB_ROOT_PASSWORD` environment variable or via `gitlab_rails['initial_root_password']` setting in `gitlab.rb`, it was provided before database was seeded for the first time (usually, the first reconfigure run).
#          2. Password hasn't been changed manually, either via UI or via command line.
#
#          If the password shown here doesn't work, you must reset the admin password following https://docs.gitlab.com/ee/security/reset_user_password.html#reset-your-root-password.

Password: tAw0UJHidcdpLT9eozwZlHIYAC0+bfXs/x5Nf78QlfE=

# NOTE: This file will be automatically deleted in the first reconfigure run after 24 hours.
```

![image-20231216185820514](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312161858617.png)

![image-20231216190344311](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312161903387.png)



### 6. 使用 Blue Ocean 创建 Pipeline

#### 项目准备

如上节课所示，由一些 flask web应用，但没有Jenkinsfile，因此我们现将其 push 到 Github

![image-20231217154517986](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312171545109.png)

![image-20231217154539267](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312171545588.png)



#### 创建 Pipeline

点击打开 Blue Ocean，点击创建新的流水线

![image-20231217132643017](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312171326147.png)

![image-20231217132741398](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312171327533.png)

代码仓库选择 GitHub，选择Create an access token here，然后去 Github 创建一个Personal access tokens，复制在这里

![image-20231217154902759](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312171549870.png)

在确认了 Github的账号后，可以选择一个 repo 创建流水线

![image-20231217155124609](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312171551698.png)

创建成功后，Jenkins上会出现一个多分支流水线的 Job

![image-20231217133547866](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312171335032.png)

接着进入流水线设置页，先设置 agent 为any

![image-20231217155404133](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312171554217.png)

设置好后，点 + 按钮，可以设置 stage

![image-20231217155551089](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312171555168.png)

在 stage 下点击添加步骤设置 step

![image-20231217155616025](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312171556139.png)

选择 Shell Script，输入 'docker-compose up -d'，还可以添加第二个步骤，'curl 127.0.0.1:5000'

![image-20231217174312109](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312171743293.png)

![image-20231217174436942](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312171744067.png)

![image-20231217174505927](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312171745046.png)

一切准备就绪，点击右上角的保存，弹出一个提示，点击 Save & Run，会将设计好的 Pipeline 放在Jenkinsfile文件里，提交给master分支

![image-20231217174602080](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312171746216.png)

![image-20231217161835074](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312171618426.png)

然后开始执行 Pipeline

![image-20231217180121257](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312171801450.png)

![image-20231217180140575](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312171801776.png)

![image-20231217180209272](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312171802449.png)

![image-20231217180230691](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312171802913.png)

其实切回 Jenkins 后，可以看到执行了一次job，且输出的内容和Blue Ocean看到的一模一样

![image-20231217180327081](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312171803260.png)

![image-20231217180412009](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312171804189.png)

![image-20231217180818056](https://becktuchuang.oss-cn-beijing.aliyuncs.com/img/202312171808177.png)



## 第5章 总结

Pipeline声明式 在易用性上要比 Blue Ocean 好用，Blue Ocean 写起来不是很方便，且只支持默认的Jenkinsfile文件名，不支持自定义。但对于复杂的Pipeline, 在梳理 Pipeline 结构的时候，Blue Ocean 可视化的界面又相对比较清晰，因此建议：

+ 使用 Pipeline 声明式 编写 Pipeline
+ 使用 Blue Ocean 作为辅助查看 Pipeline 的结构





