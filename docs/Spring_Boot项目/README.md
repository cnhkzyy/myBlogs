[toc]





# Spring Boot项目



## 01-项目介绍



### 前置知识说明

1. Java基础
2. Mysql JDBC
3. JavaWEB
4. SSM
5. Java高级：Redis、Nigix、Idea、Maven、Git、Spring Boot
6. 在线教育项目



### 项目第一天

1. 介绍项目背景
2. 介绍项目采用的商业模式
3. 介绍项目实现功能模块
4. 介绍项目使用的技术
5. 学习技术-MyBatisPlus



## 02-项目背景介绍

略



## 03-项目商业模式介绍

略



## 04-项目功能模块介绍

B2C模式：管理员、普通用户

系统后台：管理员使用

1、讲师管理模块

2、课程分类管理模块

3、课程管理模块

(1)、视频

4、统计分析模块

5、订单管理

6、banner管理

7、权限管理

系统前台：普通用户使用

1、首页数据显示

2、讲师列表和详情

3、课程列表和课程详情

(1)、视频在线播放

4、登录和注册功能

5、微信扫码登录

6、微信扫码支付



## 05-项目技术点介绍

项目采用前后端分离开发

### 后端技术

SpringBoot

SpringCloud

MybatisPlus

Spring Security

Redis

Maven

easyExcel

JWT

OAuth2



### 前端技术

Vue 

ElementUI

Axios

NodeJS



### 其他技术

阿里云OSS

阿里云视频点播服务

阿里云短信服务

微信支付和微信登录

Docker

Git

Jenkins



## 06-MyBatisPlus简介



### 一. 简介

官网：http://mp.baomidou.com/

参考文档：http://mp.baomidou.com/guide/

[MyBatis-Plus](https://github.com/baomidou/mybatis-plus)（简称 MP）是一个 [MyBatis](https://www.mybatis.org/mybatis-3/)的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生



### 二. 特性

- **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑
- **损耗小**：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作
- **强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
- **支持 Lambda 形式调用**：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错
- **支持主键自动生成**：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题
- **支持 ActiveRecord 模式**：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大的 CRUD 操作
- **支持自定义全局通用操作**：支持全局通用方法注入（ Write once, use anywhere ）
- **内置代码生成器**：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用
- **内置分页插件**：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询
- **分页插件支持多种数据库**：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer 等多种数据库
- **内置性能分析插件**：可输出 SQL 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询
- **内置全局拦截插件**：提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作



### 三. 创建并初始化数据库

#### 1.  创建数据库

mybatis_plus



#### 2. 创建User表

其表结构如下：

| id   | name   | age  | email              |
| ---- | ------ | ---- | ------------------ |
| 1    | Jone   | 18   | test1@baomidou.com |
| 2    | Jack   | 20   | test2@baomidou.com |
| 3    | Tom    | 28   | test3@baomidou.com |
| 4    | Sandy  | 21   | test4@baomidou.com |
| 5    | Billie | 24   | test5@baomidou.com |

其对应的数据库 Schema 脚本如下：

```mysql
DROP TABLE IF EXISTS user;

CREATE TABLE user
(
    id BIGINT(20) NOT NULL COMMENT '主键ID',
    name VARCHAR(30) NULL DEFAULT NULL COMMENT '姓名',
    age INT(11) NULL DEFAULT NULL COMMENT '年龄',
    email VARCHAR(50) NULL DEFAULT NULL COMMENT '邮箱',
    PRIMARY KEY (id)
);
```

其对应的数据库 Data 脚本如下：

```python
DELETE FROM user;

INSERT INTO user (id, name, age, email) VALUES
(1, 'Jone', 18, 'test1@baomidou.com'),
(2, 'Jack', 20, 'test2@baomidou.com'),
(3, 'Tom', 28, 'test3@baomidou.com'),
(4, 'Sandy', 21, 'test4@baomidou.com'),
(5, 'Billie', 24, 'test5@baomidou.com');
```





