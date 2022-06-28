---
title: 分布式事务 Seata方案实例
tags: 
 - SpringCloud
 - 分布式事务
categories: SpringCloud
summary: 记录SpringCloud Seata相关学习和整理。
keywords: >-
  Spring Cloud,Spring Cloud 分布式事务,Spring Cloud Seat,Spring Cloud Seata
  AT模式,Spring Cloud Seata 分布式事务,Spring Cloud Seata 分布式事务解决方案实例,Seata 配置实例,seata
  版本说明
author: Small-Rose / 张小菜
abbrlink: ea07db80
date: 2020-11-13 00:30:00
---

## SpringCloud 分布式事务 Seata 方案实例

### 一、前言

分布式事务学习了2PC，明白了2阶段提交，传统的XA方案依赖数据库对XA的支持，也需要在两阶段结束才能释放资源所。

Seata 方案对传统方案进行的改进，默认支持AT模式。

本次实例就是针对 Seata 的AT 模式进行尝试，嗯，官方也提供了示例，这个是我自己的写的实例，为什么不用官方的... 其实都可以

我主要是想自己动手试试，一路可以踩坑填坑，二来对容易犯错的地方做个记录，三来自然是不仅动脑还要动手...

我在写这个示例的过程中，真的有种感受，理论一听就很容易懂，代码一写环境一搭，就是没有那个效果...

所以说理解和应用是两个层次，应用过程中遇到问题，才能更好的理解。

### 二、版本选择

| 工具/服务组件                       | 版本          |
| ----------------------------------- | ------------- |
| JDK                                 | 1.8           |
| IDEA                                | 2020.1        |
| SpringBoot                          | 2.2.2.RELEASE |
| SpringCloud                         | Hoxton.SR8    |
| Spring-Cloud-alibaba                | 2.2.3.RELEASE |
| seata-server                        | 1.3.0         |
| 服务注册与服务发现 Eureka             |               |
| 服务调用openfeign + ribbon+ hystrix  |               |
| MySQL                               | 5.7.31        |

### 三、Seata 版本说明

关于Seata版本问题，内容来自官网，如果没有你需要的可以到文章尾部**注意事项**里找找，再没有就去官方[FAQ]( https://seata.io/zh-cn/docs/overview/faq.html)或者[官方ISSUE](https://github.com/seata/seata/issues) 找找。

关于Seata 怎么引入，我在搞的时候有点懵，怎么随便一查引入方式千奇百怪的，后来在官网找到了关于升级的说明，做一个简单整理。

官方原话：（不喜欢直接跳过）

> seata-all 默认不开启数据源自动代理。原 seata-all中 conf 文件配置项 client.support.spring.datasource.autoproxy 配置项失效，由注解 @EnableAutoDataSourceProxy 注解代替，注解参数可选择使用jdk代理或者cglib代理，当使用HikariDataSource 时推荐使用 cglib 代理模式。 seata-spring-boot-starter 默认开启数据源代理，对应数据源自动代理配置项与1.0.0 版本保持不变。
>
> 典型的早期引入方式，resource目录可以需要放那两个`.conf` 文件，如果XID不能全局传递需要自己去手工处理，比如用拦截器或者过滤器的方式将XID放到调用请求里。
>
> 使用spring cloud框架时需要使用[Spring Cloud Alibaba](https://github.com/alibaba/spring-cloud-alibaba)来进行seata 事务上下文的传递，与Spring Cloud Alibaba 版本集成依赖关系，参考 [版本说明](https://github.com/alibaba/spring-cloud-alibaba/wiki/版本说明)
>  spring-cloud-alibaba-seata 在 2.2.0.RELEASE 版本前 依赖的是seata-all 若继续使用低版本的  spring-cloud-alibaba-seata 可以使用高版本的 seata-all 取代内置的 seata-all 版本；
>  从spring-cloud-alibaba-seata 在 2.2.0.RELEASE  开始后（含）内部开始依赖seata-spring-boot-starter,2.2.0.RELEASE 内部集成  seata-spring-boot-starter 1.0.0 可以升级为 seata-spring-boot-starter  1.1.0，seata-spring-boot-starter 集成了seata-all，seata-spring-boot-starter  包装了对于properties或yml 配置的autoconfig 功能，在spring-cloud-alibaba-seata  2.2.0.RELEASE 前 autoconfig 功能由其本身支持，在其后去掉 spring-cloud-alibaba-seata 中关于 seata  本身的autoconfig 由seata-spring-boot-starter  支持，因此低版本spring-cloud-alibaba-seata 只能配合  seata-all使用，高版本spring-cloud-alibaba-seata 只能配合seata-spring-boot-starter  使用，以2.2.0.RELEASE为分界点。

简单整理一下大概意思：

**添加seata依赖（一般选择一种方式即可）**

- 依赖seata-all 。

> 1.1.0 之后，seata-all 默认不开启数据源自动代理。原 seata-all中 conf 文件配置项 client.support.spring.datasource.autoproxy 配置项失效，由注解 @EnableAutoDataSourceProxy 注解代替，注解参数可选择使用jdk代理或者cglib代理，当使用HikariDataSource 时推荐使用 cglib 代理模式。 seata-spring-boot-starter 默认开启数据源代理，对应数据源自动代理配置项与1.0.0 版本保持不变。
>
> 典型的早期引入方式，resource目录可以需要放那两个`.conf` 文件，如果XID不能全局传递需要自己去手工处理，比如用拦截器或者过滤器的方式将XID放到调用请求里。
>
> ```xml
> 	<dependency>
>         <groupId>io.seata</groupId>
>         <artifactId>seata-all</artifactId>
>         <version>0.x.x</version>
>     </dependency>
> ```

- 依赖seata-spring-boot-starter，支持yml、properties配置(.conf可删除)，内部已依赖seata-all，更换seata版本时需要自己手工排除内置的，再引入需要的版本。

> 如果使用 `seata-spring-boot-starter` 依赖，更换seata版本时使用，而且resource目录可以不用再放那两个`.conf` 文件了。XID传递暂时没有尝试，可能也不行吧，因为没有明确说**实现了xid传递**这种说明。
>
> 在我目前的环境里引入可以选择最高版本是 1.3.0
>
> ```xml
> 
>             <dependency>
>                  <groupId>io.seata</groupId>
>             	<artifactId>seata-spring-boot-starter</artifactId>
>                 <version>1.3.0</version>
>                 <exclusions>
>                     <exclusion>
>                         <groupId>io.seata</groupId>
>                         <artifactId>seata-all</artifactId>
>                     </exclusion>
>                 </exclusions>
>             </dependency>
>             <dependency>
>                 <groupId>io.seata</groupId>
>                 <artifactId>seata-all</artifactId>
>                 <version>x.x.x</version>
>             </dependency>
> ```
>
> 

- 依赖spring-cloud-alibaba-seata，内部集成了seata，并实现了xid传递。

> 在2.2.2.RELESE 之前引入时使用：
>
> ```xml
>             <dependency>
>                 <groupId>io.seata</groupId>
>                 <artifactId>seata-spring-boot-starter</artifactId>
>                 <version>最新版</version>
>             </dependency>
>             <dependency>
>                 <groupId>com.alibaba.cloud</groupId>
>                 <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
>                 <version>2.2.1.RELEASE</version>
>                 <exclusions>
>                     <exclusion>
>                         <groupId>io.seata</groupId>
>                         <artifactId>seata-spring-boot-starter</artifactId>
>                     </exclusion>
>                 </exclusions>
>             </dependency>
> ```
>
> 在2.2.2.RELESE （含）之后引入使用：
>
> ```xml
> 	   <dependency>
>             <groupId>com.alibaba.cloud</groupId>
>             <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
>             <version>2.2.2.RELEASE</version>
>        </dependency>
> ```



官方不同版本说明：

**注意事项1**：0.8 或 0.9 升级到 1.0时需要注意的问题：

> 1. （可选）1.0支持yml、properties，需用seata-spring-boot-starter替换掉 seata-all
> 2. （必选）TC端表lock_table字段branch_id增加普通索引
> 3. （可选）部分参数命名改动，[点击查看参数配置](https://seata.io/zh-cn/docs/user/configurations100.html)
> 4. （可选） client.report.success.enable可以置为false，提升性能

**注意事项2**：seata 升级到 1.1 时需要注意的问题：

>可以使用 seata-spring-boot-starter 的方式引入了。上面已经做了说明。
>
>TC端采用 db 存储模式时 branch_table 中增加 gmt_create，gmt_modified 字段的精度，用于精确确认回滚的顺序， [各数据库脚本参考](https://github.com/seata/seata/tree/1.1.0/script/server/db) 

**注意事项3**：seata 升级到 1.2 时需要注意的问题：

> nacos注册中心新增服务名的属性配置registry.nacos.application = "seata-server"，原固定名为serverAddr，现默认为seata-server，Server和Client端需保持一致。

**注意事项4**：seata 升级到 1.3 时需要注意的问题：

> 1. nacos注册中心新增group的属性配置seata.registry.nacos.group，如果无配置,则默认为DEFAULT_GROUP，Server和Client端需保持一致。
> 2. mysql undolog表去除id字段,与branch_table一并加强时间戳精度,防止undolog回滚时顺序错误导致出现脏数据无法回滚.(注:需要mysql5.6版本以上)

**注意事项5**：seata 目前最新版 1.4 不建议使用：

> 1.3与1.4的Redis数据无法兼容,因Redis模式重构数据存储结构为hash,1.3升级的用户需等待事务全部运行完毕后再做迭代.



### 四、服务规划

| 模式         | 工程名              | 端口 | 服务名         |
| ------------ | ------------------- | ---- | -------------- |
| Seata-Server | -                   | 8091 | seata-server   |
| 作为注册中心 | server-eureka-9900  | 9900 | -              |
| 模拟订单服务 | server-order-9901   | 9901 | server-order   |
| 模拟库存服务 | server-store-9902   | 9902 | server-store   |
| 模拟扣款服务 | server-account-9903 | 9903 | server-account |

主要目录结构

```
+-------+ distributed-tx-seata
|         
+-------+------ server-eureka-9900
|       |
|       +-------pom.xml
|
+-------+------ server-order-9901
|       |
|       +-------pom.xml
|
+-------+------ server-store-9902
|       |
|       +-------pom.xml
|
+-------+------ server-store-9903
|       |
|       +-------pom.xml
|
+-------pom.xml
```



### 五、测试目标：

（1） server-order 调用 server-store 。两层调用，正常调用和测试回滚。

（2） server-order 调用 server-store，server-store 再调用 server-account 。三层调用，正常调用和测试回滚。



### 六、代码结构

#### 1、父工程引入

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>


    <groupId>com.xiaocai.distran</groupId>
    <artifactId>distributed-transaction</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>distributed-transaction</name>
    <description>DistributedTransaction project for Spring Boot</description>

    <packaging>pom</packaging>

    <modules>
        <module>server-eureka-9900</module>
        <module>server-order-9901</module>
        <module>server-store-9902</module>
        <module>server-account-9903</module>
    </modules>

    <properties>
        <java.version>1.8</java.version>
        <spring-boot.version>2.2.2.RELEASE</spring-boot.version>
        <spring-cloud.version>Hoxton.SR8</spring-cloud.version>
        <spring-cloud-alibaba.version>2.2.3.RELEASE</spring-cloud-alibaba.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <junit.version>4.12</junit.version>
        <log4j.version>1.2.17</log4j.version>
        <lombok.version>1.18.12</lombok.version>
        <mysql.version>5.1.47</mysql.version>
        <druid.version>1.1.16</druid.version>
        <mybatis-springboot.version>2.1.3</mybatis-springboot.version>
        <seata.version>1.3.0</seata.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <!--spring boot 2.2.2-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!-- spring cloud Hoxton.SR8 -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--spring cloud alibaba 2.2.3.RELEASE-->
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring-cloud-alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>


            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>${mybatis-springboot.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-devtools</artifactId>
                <version>${spring-boot.version}</version>
                <scope>runtime</scope>
                <optional>true</optional>
            </dependency>
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>${mysql.version}</version>
                <scope>runtime</scope>
            </dependency>
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>${lombok.version}</version>
                <optional>true</optional>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
                <scope>test</scope>
                <exclusions>
                    <exclusion>
                        <groupId>org.junit.vintage</groupId>
                        <artifactId>junit-vintage-engine</artifactId>
                    </exclusion>
                </exclusions>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>${spring-boot.version}</version>
                <configuration>
                    <fork>true</fork>
                    <addResources>true</addResources>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```



#### 2、注册中心

这里只做一个注册中心配置，没有使用多注册中心。

```
server:
  port: 9900

eureka:
  instance:
    hostname: localhost #eureka服务端的实例名称
  client:
    register-with-eureka: false #false表示不向注册中心注册自己。
    fetch-registry: false #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/

```

#### 3、Seata-Server

##### 3.1 seata-server 下载

可以从[Github/seata](https://github.com/seata/seata/releases) 下载，可以复制下载链接到迅雷或者IDM之类的下载器，也可以使用代下载服务

- [http://gitd.cc/  点我直接去](http://gitd.cc/) 
- [http://github.b15.me/  点我直接去](http://github.b15.me/)

也可以从我的云盘下载：https://yun.zhangxiaocai.cn 如果没有请提醒我上传。

##### 3.2 关于Seata-Server 特别说明：

seata目录的conf文件夹中，有一个file.conf.example文件，这个是早期没有使用`.yml`、`.properties`进行配置的时候，作为客户端的file.conf。把它改名成file.conf，复制到项目resource下，修改其中的service，和store。

作为服务端配置来讲，

store 里配置的是seata作为TC角色时一些数据的存储方式，一般是`file`、`db`、`redis`三种，file只适合单机环境。对应的模式就改对应的块，没有就自己加下。

```
## transaction log store, only used in server side
store {
  ## store mode: file、db、redis
  mode = "db"
  
  db {
    ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp) etc.
    datasource = "druid"
    ## mysql/oracle/postgresql/h2/oceanbase etc.
    dbType = "mysql"
    driverClassName = "com.mysql.jdbc.Driver"
    url = "jdbc:mysql://127.0.0.1:3306/seata"
    user = "root"
    password = "123456"
    minConn = 5
    maxConn = 30
    globalTable = "global_table"
    branchTable = "branch_table"
    lockTable = "lock_table"
    queryLimit = 100
  }
}
```

service需要注意的就是`vgroup_mapping.xxx= “yyy”`，这里的xxx就是yml中配置的事务组，yyy就是最上边配置seata服务端时，注册到eureka上的服务名字。我刚刚填的是seata，这里也填seata。

`registry.conf` 是将seata-server 像注册中心注册的配置，如使用eureka作为注册中心：

```
registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  type = "eureka"
 
  eureka {
    serviceUrl = "http://localhost:9900/eureka"
    application = "seata_server"
    weight = "1"
  }
}
```

`application` 的值将会和`yml`里的 `vgroupMapping.my_test_tx_group` 的值对应，默认值是`default`，我这里统一使用了`seata_server`。

```yml
  cloud:
    alibaba:
      seata:
        application-id: ${spring.application.name}
        enabled: true
        tx-service-group: my_test_tx_group
        enable-auto-data-source-proxy: true
        service:
          disable-global-transaction: false
          vgroupMapping.my_test_tx_group: seata_server
          grouplist:
              default: 127.0.0.1:8091
```

关于 `vgroupMapping.my_test_tx_group` 早期不能在yml里配置的时候，需要在服务端的file.conf里进行配置，大概是这个样子吧，暂时没有去研究早期版本（如有不对欢迎指正）：

```
service {
  #transaction service group mapping
  vgroupMapping.my_test_tx_group = "seata_server"
  #only support when registry.type=file, please don't set multiple addresses
  seata_server.grouplist = "127.0.0.1:8091"
  #degrade, current not support
  enableDegrade = false
  #disable seata
  disableGlobalTransaction = false
}
```

而且 `vgroupMapping.my_test_tx_group` 是使用了下划线的方式 `vgroup_Mapping.my_test_tx_group` 。并且在使用的写法上两组值相关联：

```
vgroupMapping.my_test_tx_group = "seata_server"
  #only support when registry.type=file, please don't set multiple addresses
seata_server.grouplist = "127.0.0.1:8091"
```

`vgroupMapping` 你在使用的使用到底有没有下划线，可以试着找一下

再说说grouplist，注释已经说明了grouplist的只有在registry.type=file 的时候才能生效，意思就是多个seata-server的时候，这里可以配置一组值，来实现seata-server 集群管理，需要注意的是seata-server最后共享DB，否则事务注册信息会存储在不同的DB，肯定有问题。





##### 3.3 服务端 file.conf 的配置

服务端 file.conf 的配置，我使用的是db模式，就是事务协调器的数据放数据库里。

```
## transaction log store, only used in seata-server
store {
  ## store mode: file、db、redis
  mode = "db"

  ## file store property
  file {
    ## store location dir
    dir = "sessionStore"
    # branch session size , if exceeded first try compress lockkey, still exceeded throws exceptions
    maxBranchSessionSize = 16384
    # globe session size , if exceeded throws exceptions
    maxGlobalSessionSize = 512
    # file buffer size , if exceeded allocate new buffer
    fileWriteBufferCacheSize = 16384
    # when recover batch read size
    sessionReloadReadSize = 100
    # async, sync
    flushDiskMode = async
  }

  ## database store property
  db {
    ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp)/HikariDataSource(hikari) etc.
    datasource = "druid"
    ## mysql/oracle/postgresql/h2/oceanbase etc.
    dbType = "mysql"
    driverClassName = "com.mysql.jdbc.Driver"
    url = "jdbc:mysql://127.0.0.1:3306/seata"
    user = "root"
    password = "123456"
    minConn = 5
    maxConn = 30
    globalTable = "global_table"
    branchTable = "branch_table"
    lockTable = "lock_table"
    queryLimit = 100
    maxWait = 5000
  }
 
}

## server configuration, only used in server side
server {
  recovery {
    #schedule committing retry period in milliseconds
    committingRetryPeriod = 1000
    #schedule asyn committing retry period in milliseconds
    asynCommittingRetryPeriod = 1000
    #schedule rollbacking retry period in milliseconds
    rollbackingRetryPeriod = 1000
    #schedule timeout retry period in milliseconds
    timeoutRetryPeriod = 1000
  }
  undo {
    logSaveDays = 7
    #schedule delete expired undo_log in milliseconds
    logDeletePeriod = 86400000
  }
  #unit ms,s,m,h,d represents milliseconds, seconds, minutes, hours, days, default permanent
  maxCommitRetryTimeout = "-1"
  maxRollbackRetryTimeout = "-1"
  rollbackRetryTimeoutUnlockEnable = false
}

## metrics configuration, only used in server side
metrics {
  enabled = false
  registryType = "compact"
  # multi exporters use comma divided
  exporterList = "prometheus"
  exporterPrometheusPort = 9898
}
```

##### 3.4 服务端 registry.conf 的配置

```
registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  type = "eureka"


  eureka {
    serviceUrl = "http://localhost:9900/eureka"
    application = "seata_server"
    weight = "1"
  }
 
}

#配置中心相关
config {
  # file、nacos 、apollo、zk、consul、etcd3
  type = "file"

  nacos {
    serverAddr = "127.0.0.1:8848"
    namespace = ""
    group = "SEATA_GROUP"
    username = ""
    password = ""
  }
  consul {
    serverAddr = "127.0.0.1:8500"
  }
  apollo {
    appId = "seata-server"
    apolloMeta = "http://192.168.1.204:8801"
    namespace = "application"
  }
  zk {
    serverAddr = "127.0.0.1:2181"
    sessionTimeout = 6000
    connectTimeout = 2000
    username = ""
    password = ""
  }
  etcd3 {
    serverAddr = "http://localhost:2379"
  }
  file {
    name = "file.conf"
  }
}
```

##### 3.5 Seata-server 启动

```bash
# windows 
seata.bat -p 8091 -h 192.168.80.4 -m db

# linux 
seata.sh -p 8091 -h 192.168.80.4 -m db
```



#### 4、数据库环境

因为是模拟，可以创建四个数据库，也可以在一个数据库里然后使用四个数据源的方式。

我使用的创建4个数据库：

- seata
- server-order
- server-store
- server-account

有对应的4个SQL脚本，创建完成后执行一下。



#### 5、关键代码

**订单服务作为RM，也是TM角色**，官方的一些Demo会把TM独立出来。

`OrderService` 调用：

```java
package com.xiaocai.distran.serverorder.service.impl;

import com.xiaocai.distran.serverorder.bean.OrderBean;
import com.xiaocai.distran.serverorder.mapper.OrderMapper;
import com.xiaocai.distran.serverorder.openfeign.StoreClient;
import com.xiaocai.distran.serverorder.service.OrderService;
import io.seata.core.context.RootContext;
import io.seata.spring.annotation.GlobalTransactional;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述：
 * @author: 张小菜
 * @date: 2020/11/9 22:01
 * @version: v1.0
 */
@Service
@Slf4j
public class OrderServiceImpl implements OrderService {

    @Resource
    private OrderMapper orderMapper;

    @Resource
    private StoreClient storeClient;


    //@GlobalTransactional
    @GlobalTransactional(timeoutMills = 60000 , rollbackFor = Exception.class)
    @Override
    public boolean addOrder(OrderBean orderBean) {
        log.info("create order begin ... xid: " + RootContext.getXID());
        boolean flag = false;
        //  本地事务保存订单
        orderMapper.addOrder(orderBean);

        // 远程调用扣减库存
        if (storeClient.decreaseStore(orderBean.getProdId(), orderBean.getNumber(), orderBean.getUserId())){
            flag = true;
        }
        if (orderBean.getNumber() == 100){
            //throw new RuntimeException("故意制造异常测试回滚操作");
            int x = 10/0 ;
        }

        return flag;
    }
}
```

Feign的部分:

```java
package com.xiaocai.distran.serverorder.openfeign;

import com.xiaocai.distran.serverorder.openfeign.fallback.StoreFallBack;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述：
 * @author: 张小菜
 * @date: 2020/11/9 22:55
 * @version: v1.0
 */
@Component
@FeignClient(value = "server-store", fallback = StoreFallBack.class )
public interface StoreClient {

    /**
     *  调用扣减库存操作
     * @param prodId
     * @param number
     * @param userId
     * @return
     */
    @RequestMapping(value = "/v1/store/decrease" ,method = RequestMethod.POST)
    public boolean decreaseStore(@RequestParam("prodId") Integer prodId,
                                 @RequestParam("number") Integer number,
                                 @RequestParam("userId") Integer userId);

}
```

Seata 客户端配置，其他工程Seata部分都一样，如果你实在不放心也可以将两个`.conf` 文件放resource目录。

```
server:
  port: 9901
  servlet:
    application-display-name: server-order

#====================================datasource =============================================
spring:
  application:
    name: server-order
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.jdbc.Driver
    platform: mysql
    url: jdbc:mysql://127.0.0.1:3306/server_order?useUnicode=true&characterEncoding=utf8&autoReconnect=true&useSSL=false
    username: root
    password: 123456
    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
#    filters: stat,wall,log4j
    logSlowSql: true

  cloud:
    alibaba:
      seata:
        application-id: ${spring.application.name}
        enabled: true
        tx-service-group: my_test_tx_group
        enable-auto-data-source-proxy: true
        service:
          disable-global-transaction: false
          vgroupMapping.my_test_tx_group: seata_server
          grouplist:
              default: 127.0.0.1:8091
#
#    consul:
#      host: localhost
#      port: 8500
#      discovery:
#        service-name: ${spring.application.name}


#==================================== logging =============================================
logging:
  level:
    root: info
    com:
     xiaocai:
       distran:
         serverorder: info
    io:
      seata: info
    org:
      springframework:
        cloud:
          alibaba:
            seata:
              web: info

mybatis:
  # spring boot集成mybatis的方式打印sql
  configuration:
    mapUnderscoreToCamelCase: true
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
#==================================== eureka =============================================
eureka:
  client: #客户端注册进eureka服务列表内
    service-url:
      defaultZone: http://localhost:9900/eureka
    fetch-registry: true
  instance:
    instance-id: server-order  # 可以自定义服务名称展示
    prefer-ip-address: true # 访问路径可以显示IP

#==================================== feign hystrix =============================================
feign:
  hystrix:
    enabled: true #如果处理自身的容错就开启。

ribbon:
  ConnectTimeout: 600 # 设置连接超时时间 default 2000
  ReadTimeout: 6000    # 设置读取超时时间  default 5000
  OkToRetryOnAllOperations: true # 对所有操作请求都进行重试  default false
  MaxAutoRetriesNextServer: 2    # 切换实例的重试次数  default 1
  MaxAutoRetries: 1     # 对当前实例的重试次数 default 0

#==================================== endpoints =============================================
management:
  endpoints:
    web:
      exposure:
        include: refresh,health,info,env
```



**库存扣减服务作为RM角色**

`StoreService` 调用：

```java
package com.xiaocai.distran.serverstore.service.impl;

import com.xiaocai.distran.serverstore.bean.StorageBean;
import com.xiaocai.distran.serverstore.openfeign.AccountFeignClient;
import com.xiaocai.distran.serverstore.mapper.StoreMapper;
import com.xiaocai.distran.serverstore.service.StoreService;
import io.seata.core.context.RootContext;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述：
 * @author: 张小菜
 * @date: 2020/11/9 23:44
 * @version: v1.0
 */
@Service
@Slf4j
public class StoreServiceImpl implements StoreService {

    @Resource
    private StoreMapper storeMapper;

    @Autowired
    private AccountFeignClient accountFeignClient;

    @Override
    public boolean decreaseStore(Integer prodId, Integer number, Integer userId) {
        boolean flag = false ;
        log.info("decreaseStore begin ... xid: " + RootContext.getXID());
        log.info("----开始执行扣减库存操作-----");
        int i = storeMapper.updateStoreByProdId(prodId, number);


        log.info("----查询商品价格-----");
        StorageBean storageBean = storeMapper.getStorageBeanByProId(prodId);
        log.info("----商品价格是-----" + storageBean.getProdPrice());
        log.info("----开始调用扣减账户操作-----");
        if(i >0  && accountFeignClient.decreaseAccount(userId, storageBean.getProdPrice())){
            flag = true ;
        }
        //flag = i > 0 ?  true : false ;
        return flag;
    }
}
```

Feign的部分:

```java
package com.xiaocai.distran.serverstore.openfeign;

import com.xiaocai.distran.serverstore.openfeign.fallback.AccountFallBack;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述：
 * @author: 张小菜
 * @date: 2020/11/12 18:56
 * @version: v1.0
 */
@Component
@FeignClient(value = "server-account", fallback = AccountFallBack.class )
public interface AccountFeignClient {


    @RequestMapping(value = "/v1/account/decrease" ,method = RequestMethod.POST)
    public boolean decreaseAccount(@RequestParam("userId") int userId, @RequestParam("money") double money);

}
```



**账户扣款服务作为RM角色**

`AccountService` 调用：

```java
package com.xiaocai.distran.serveraccount.service.impl;

import com.xiaocai.distran.serveraccount.mapper.AccountMapper;
import com.xiaocai.distran.serveraccount.service.AccountService;
import io.seata.core.context.RootContext;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述：
 * @author: 张小菜
 * @date: 2020/11/12 18:53
 * @version: v1.0
 */
@Service
@Slf4j
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountMapper accountMapper;

    @Override
    public boolean decreaseAccount(int userId, double money) {
        log.info("----decreaseAccount  XID : "+ RootContext.getXID());
            /*
            try {
                Thread.sleep(50000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            */
        log.info("----执行扣减账户余额-----");
        boolean bool = accountMapper.decreaseAccount(userId, money);
        log.info("----扣减账户余额完成-----"+ bool);
        return bool;
    }
}
```



#### 6、测试验证

依次启动注册中心、Seata-Server、然后其他几个服务。

如果你是自己动手，有几个点关键词可以查看：

- `Global Transaction Clients are initialized. `  （作为TM角色启动时会有）

- `register RM success` 

Eureka注册中心查看服务注册情况：http://localhost:9900/

正常测试，看数据库初始数据，再访问`http://localhost:9901/v1/order/add/1001/1`

比较数据库数据变化。

回滚测试，看数据库初始数据，再访问`http://localhost:9901/v1/order/add/1001/100`

比较数据库数据变化。

打开超时代码块再次访问，验证回滚状态。

### 七、工程代码

下载地址：[boot-seata-at](https://github.com/small-rose/distributed-transaction/tree/main/boot-seata-at) 

也可以直接Git下载：

```bash
git clone git@github.com:small-rose/distributed-tx-seata.git
```

然后导入工程使用。

注意建表脚本在sql文件夹。

### 八、注意事项

[官方FAQ地址]( https://seata.io/zh-cn/docs/overview/faq.html) 

#### 1、Eureka做注册中心，TC高可用时

> **  Q: 8.Eureka做注册中心，TC高可用时，如何在TC端覆盖Eureka属性?** 
>
> **A：** 在seata\conf目录下新增eureka-client.properties文件，添加要覆盖的Eureka属性即可。
>  例如，要覆盖eureka.instance.lease-renewal-interval-in-seconds和eureka.instance.lease-expiration-duration-in-seconds，添加如下内容：
>
> ```
> eureka.lease.renewalInterval=1
> eureka.lease.duration=2
> ```
>
> 属性前缀为eureka，其后的属性名可以参考类com.netflix.appinfo.PropertyBasedInstanceConfigConstants，也可研究seata源码中的discovery模块的seata-discovery-eureka工程



#### 2、Seata 支持的 RPC 框架

> ** Q: 17.Seata 支持哪些 RPC 框架?** 
>
> ```
> 1. AT 模式支持Dubbo、Spring Cloud、Motan、gRPC 和 sofa-RPC。
> 2. TCC 模式支持Dubbo、Spring Cloud和sofa-RPC。
> ```



#### 3、AT模式注意事项

> **Q: 20. 使用 AT 模式需要的注意事项有哪些 ？**
>
> **A:**
>
> 1. 必须使用代理数据源，有 3 种形式可以代理数据源：
>
> - 依赖 seata-spring-boot-starter 时，自动代理数据源，无需额外处理。
> - 依赖 seata-all 时，使用 @EnableAutoDataSourceProxy (since 1.1.0) 注解，注解参数可选择 jdk 代理或者 cglib 代理。
> - 依赖 seata-all 时，也可以手动使用 DatasourceProxy 来包装 DataSource。
>
> 1. 配置 GlobalTransactionScanner，使用 seata-all 时需要手动配置，使用 seata-spring-boot-starter 时无需额外处理。
> 2. 业务表中必须包含单列主键，若存在复合主键，请参考问题 13 。
> 3. 每个业务库中必须包含 undo_log 表，若与分库分表组件联用，分库不分表。
> 4. 跨微服务链路的事务需要对相应 RPC 框架支持，目前 seata-all 中已经支持：Apache Dubbo、Alibaba  Dubbo、sofa-RPC、Motan、gRpc、httpClient，对于 Spring Cloud 的支持，请大家引用  spring-cloud-alibaba-seata。其他自研框架、异步模型、消息消费事务模型请结合 API 自行支持。
> 5. 目前AT模式支持的数据库有：MySQL、Oracle、PostgreSQL和 TiDB。
> 6. 使用注解开启分布式事务时，若默认服务 provider 端加入 consumer 端的事务，provider 可不标注注解。但是，provider 同样需要相应的依赖和配置，仅可省略注解。
> 7. 使用注解开启分布式事务时，若要求事务回滚，必须将异常抛出到事务的发起方，被事务发起方的 @GlobalTransactional 注解感知到。provide 直接抛出异常 或 定义错误码由 consumer 判断再抛出异常。

#### 4、无法注册分支事务到全局SESSSION XID

> **Q: 6.为什么分支事务注册时, 全局事务状态不是begin?**
>
> **A:**
>
> - 异常：Could not register branch into global session xid = status =  Rollbacked（还有Rollbacking、AsyncCommitting等等二阶段状态） while expecting Begin
> - 描述：分支事务注册时，全局事务状态需是一阶段状态begin，非begin不允许注册。属于seata框架层面正常的处理，用户可以从自身业务层面解决。
> - 出现场景（可继续补充）
>
> ```
>   1. 分支事务是异步，全局事务无法感知它的执行进度，全局事务已进入二阶段，该异步分支才来注册
>   2. 服务a rpc 服务b超时（dubbo、feign等默认1秒超时），a上抛异常给tm，tm通知tc回滚，但是b还是收到了请求（网络延迟或rpc框架重试），然后去tc注册时发现全局事务已在回滚
>   3. tc感知全局事务超时(@GlobalTransactional(timeoutMills = 默认60秒))，主动变更状态并通知各分支事务回滚，此时有新的分支事务来注册
> ```

#### 5、Seata 事务隔离性

> **Q: 4.怎么使用Seata框架，来保证事务的隔离性？** 
>
> **A:** 因seata一阶段本地事务已提交，为防止其他事务脏读脏写需要加强隔离。
>
> 1. 脏读 select语句加for update，代理方法增加@GlobalLock+@Transactional或@GlobalTransaction
> 2. 脏写 必须使用@GlobalTransaction
>     注：如果你查询的业务的接口没有GlobalTransactional 包裹，也就是这个方法上压根没有分布式事务的需求，这时你可以在方法上标注@GlobalLock+@Transactional 注解，并且在查询语句上加 for update。 如果你查询的接口在事务链路上外层有GlobalTransactional注解，那么你查询的语句只要加for update就行。设计这个注解的原因是在没有这个注解之前，需要查询分布式事务读已提交的数据，但业务本身不需要分布式事务。 若使用GlobalTransactional注解就会增加一些没用的额外的rpc开销比如begin 返回xid，提交事务等。GlobalLock简化了rpc过程，使其做到更高的性能。