---
title: SpringCloud Seata
tags: SpringCloud
categories: SpringCloud
summary: 记录SpringCloud Seata相关学习和整理。
keywords: >-
  Spring Cloud,Spring Cloud 微服务,Spring Cloud Seat,Spring Cloud Seata
  原理,微服务学习教程,Spring Cloud Seata 分布式事务,Spring Cloud Seata 分布式事务原理,Sentinel
  学习,Seata 配置
author: Small-Rose / 张小菜
abbrlink: fb7c74c4
date: 2020-09-30 16:00:00
---

## SpringCloud Alibaba Seata 分布式事务


相关链接：

- [SpringCloud 微服务学习](bad02d94.html)
- [RESTful 接口设计规范](eeda118.html)
- [SpringCloud Eureka 服务注册与发现](648d6b26.html)
- [SpringCloud Consul 服务注册与发现](3010739f.html)
- [SpringCloud Ribbon 客户端负载均衡与服务调用](7ecbba09.html)
- [SpringCloud OpenFeign 服务接口调用](b27245de.html)
- [SpringCloud Hystrix 熔断器](464bb413.html)
- [SpringCloud Gateway 服务网关](bc16d093.html)
- [SpringCloud Zuul 服务网关](fbbe72e1.html)
- [Spring Cloud Config 分布式配置中心](5b5896d1.html)
- [Spring Cloud Bus 消息总线](9e02a3e8.html)
- [SpringCloud Stream 消息驱动](7f3b07b1.html)
- [SpringCloud Sleuth 分布式请求链路追踪](b99c053f.html)
- [SpringCloud Alibaba](461589aa.html)
- [SpringCloud Alibaba Nacos服务注册和配置中心](a46e46f8.html)
- [SpringCloud Alibaba Sentinel 实现熔断与限流](5122a811.html)
- [SpringCloud Alibaba Seata 分布式事务](fb7c74c4.html)
- [SpringCloud 分布式事务 Seata 方案实例](ea07db80.html)
- [SpringCloud-Exception-libs](1618e871.html)

## 一、概述

单体应用拆分成多个业务单元的微服务，每个微服务内部的数据一致性由微服务自身的本地事务来保证，但是整个系统的全局的数据一致性问题随之产生。

比如：一次业务操作需要跨多个数据源或需要跨多个系统进行远程调用，就会产生分布式事务问题。

Seata 是一款开源的分布式事务解决方案。

![架构图](/images/springcloud/seata-work-01.jpg)

## 二、Seata

### 1、主要内容

**官网**

Seata 官网：http://seata.io/zh-cn/

发布说明或下载地址：https://github.com/seata/seata/releases

参数配置：https://seata.io/zh-cn/docs/user/configurations.html

**主要内容**

Seata是一款开源的分布式事务解决方案，致力于在微服务架构下提供高性能和简单易用的分布式事务服务。

Seata 将为用户提供了 AT、TCC、SAGA 和 XA 事务模式，为用户打造一站式的分布式解决方案。 

**AT 模式** 默认的事务模式。

**前提**

- 基于支持本地 ACID 事务的关系型数据库。
- Java 应用，通过 JDBC 访问数据库。

**整体机制**

两阶段提交协议的演变：

- 一阶段：业务数据和回滚日志记录在同一个本地事务中提交，释放本地锁和连接资源。
- 二阶段：
  - 提交异步化，非常快速地完成。
  - 回滚通过一阶段的回滚日志进行反向补偿。

**写隔离**

- 一阶段本地事务提交前，需要确保先拿到 **全局锁** 。
- 拿不到 **全局锁** ，不能提交本地事务。
- 拿 **全局锁** 的尝试被限制在一定范围内，超出范围将放弃，并回滚本地事务，释放本地锁。

示例：https://seata.io/zh-cn/docs/overview/what-is-seata.html

**使用 AT 模式需要的注意事项**

- 每个业务库中必须包含 undo_log 表，若与分库分表组件联用，分库不分表。 
- AT模式支持的数据库有：MySQL、Oracle、PostgreSQL和 TiDB。 
- 注解开启分布式事务时，若默认服务 provider 端加入 consumer 端的事务，provider 可不标注注解。但是，provider 同样需要相应的依赖和配置，仅可省略注解。 
- 使用注解开启分布式事务时，若要求事务回滚，必须将异常抛出到事务的发起方，被事务发起方的 @GlobalTransactional 注解感知到。provide 直接抛出异常 或 定义错误码由 consumer 判断再抛出异常。 

更多参考：[官方FAQ](https://seata.io/zh-cn/docs/overview/faq.html) 

Seata 分布式事务有全局事务和分支事务的划分

![全局事务和分支事务](/images/springcloud/seata-work-02.png)



### 2、Seata 事务模型

分布式事务处理模型：**1个ID +3个组件**

1个ID： 

- 全局事务Transaction ID ：XID

3个组件：

- **TC (Transaction Condinator) - 事务协调者：** 维护全局和分支事务的状态，驱动全局事务提交或回滚。
- **TM (Transaction Manager) - 事务管理器：** 定义全局事务的范围：负责开启一个全局事务，最终发起提交或回滚全局事务。
- **RM (Resource Manager) - 资源管理器：** 管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。



### 3、Seata 事务过程

分布式事务处理过程：

（1）TM事务管理器 向 TC 事务协调器 申请开启一个全局事务，全局事务创建成功并生成一个全局唯一的XID；

（2）XID 在微服务调用链路的上下文中传播；

（3）RM资源管理器 向 TC事务协调器注册分支事务，将其纳入XID对应全局事务的管辖；

（4）TM事务管理器 向 TC事务协调器 发起针对XID的全局提交或回滚的决议；

（5）TC事务协调器 调度XID 下管辖的全部分支事务完成提交或回滚请求。

![分布式事务处理过程](/images/springcloud/seata-work-03.png)



### 4、使用

Spring 本地事务使用的是 `@Transactional` 。

seata 全局事务使用的是注解 `@GlobalTransactional` 。

![分布式事务处理过程](/images/springcloud/seata-work-04.jpg)



## 三、Seata Server安装

Seata-Server安装步骤：

### 1、下载

下载地址：http://seata.io/zh-cn/blog/download.html

下载地址：https://github.com/seata/seata/releases

其实都是从github的release 那里下载的。

 github下载较慢的话，可以使用下载器或者代下载来提速。下载完成后，将`nacos-server-1.3.0.zip`解压到指定目录。

### 2、修改配置

**（1）file.conf**

修改`conf`目录下的`file.conf`配置文件。

主要修改：

- 自定义事务组名称；
- 事务日志存储模式为db；
- 数据库连接信息。

file.conf 原始文件：

server 模块，新的版本可能没有这个模块了，如果没有先忽略。这里主要是修改事务组的名字。

```txt
 vgroup_mapping.my_test_tx_group = "xiaocai_tx_group"
```

store模块：

```txt
mode = "db"
```

db 模块中修改数据库连接：

```j
url = "jdbc:mysql://127.0.0.1:3306/seata"
user = "root"
password = "自己的密码"
```

**（2）registry.conf**

修改`seata-server-1.3.0/seata/conf`目录下的`registry.conf`配置文件，主要是为了指明注册中心，测试为nacos，及修改nacos连接信息。

registry 注册类型改为：

```txt
type = "nacos"
```

nacos 模块：

```txt
  nacos {
    application = "seata-server"
    serverAddr = "127.0.0.1:8848"
    group = "SEATA_GROUP"
    namespace = ""
    cluster = "default"
    username = ""
    password = ""
  }
```

### 3、数据库初始化

mysql57 新建数据库 seata ，然后执行初始化建表脚本。

建表脚本`db_store.sql`在`/seata-server-1.3.0/seata/conf`目录里面。

内容如下：

```SQL
-- the table to store GlobalSession data
drop table if exists `global_table`;
create table `global_table` (
  `xid` varchar(128)  not null,
  `transaction_id` bigint,
  `status` tinyint not null,
  `application_id` varchar(32),
  `transaction_service_group` varchar(32),
  `transaction_name` varchar(128),
  `timeout` int,
  `begin_time` bigint,
  `application_data` varchar(2000),
  `gmt_create` datetime,
  `gmt_modified` datetime,
  primary key (`xid`),
  key `idx_gmt_modified_status` (`gmt_modified`, `status`),
  key `idx_transaction_id` (`transaction_id`)
);

-- the table to store BranchSession data
drop table if exists `branch_table`;
create table `branch_table` (
  `branch_id` bigint not null,
  `xid` varchar(128) not null,
  `transaction_id` bigint ,
  `resource_group_id` varchar(32),
  `resource_id` varchar(256) ,
  `lock_key` varchar(128) ,
  `branch_type` varchar(8) ,
  `status` tinyint,
  `client_id` varchar(64),
  `application_data` varchar(2000),
  `gmt_create` datetime,
  `gmt_modified` datetime,
  primary key (`branch_id`),
  key `idx_xid` (`xid`)
);

-- the table to store lock data
drop table if exists `lock_table`;
create table `lock_table` (
  `row_key` varchar(128) not null,
  `xid` varchar(96),
  `transaction_id` long ,
  `branch_id` long,
  `resource_id` varchar(256) ,
  `table_name` varchar(32) ,
  `pk` varchar(36) ,
  `gmt_create` datetime ,
  `gmt_modified` datetime,
  primary key(`row_key`)
);

```

事务参与者需要持有 undo_log 的表，用来存储的SQL执行时的前置镜像和后置镜像，如果发生回滚就可以根据镜像进行SQL反向补偿操作。

```SQL
-- the table to store seata xid data
-- 0.7.0+ add context
-- you must to init this sql for you business databese. the seata server not need it.
-- 此脚本必须初始化在你当前的业务数据库中，用于AT 模式XID记录。与server端无关（注：业务数据库）
-- 注意此处0.3.0+ 增加唯一索引 ux_undo_log
drop table `undo_log`;
CREATE TABLE `undo_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) NOT NULL,
  `context` varchar(128) NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  `ext` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

### 4、启动服务

先启动Nacos，再启动Seata 服务。


关于Seata启动
```BASH
seata-server.sh -p 8091 -h 127.0.0.1 -m file
```
`-m`  ： Seata启动之后的镜像存储模式，如file、db、redis
`-p`  ： 启动端口




## 四、Seata 实战

实战单独放了一篇[《SpringCloud 分布式事务 Seata 方案实例》](https://zhangxiaocai.cn/posts/ea07db80.html)


