---
title: SpringCloud Alibaba Nacos
tags: SpringCloud
categories: SpringCloud
summary: 本文记录SpringCloud Nacos相关学习和整理。
keywords: >-
  Spring Cloud,Spring Cloud 微服务,Spring Cloud Nacos,Spring Cloud Nacos
  原理,微服务学习教程,Spring Cloud Nacos 服务注册与发行,Nacos 使用
author: Small-Rose / 张小菜
abbrlink: a46e46f8
date: 2020-09-28 13:40:00
---

## SpringCloud Alibaba Nacos服务注册和配置中心


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


## 一、概述

Nacos ：Nacos：Dynamic Naming and Configuration Service，服务注册与配置中心。

Na : Naming 缩写，

Co ：Configuration 缩写，

S ：Service 缩写

**相关网站**

gitHub项目地址：https://github.com/alibaba/Nacos

Nacos 官网：https://nacos.io/zh-cn/index.html

Github 地址：https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html#_spring_cloud_alibaba_nacos_discovery

## 二、Nacos

### 1、基本概念

Nacos是用构建微服务应用的动态服务发现，配置管理和服务管理中心。是服务注册中心和服务配置中心的组合。

Nacos = Eureka + Config +Bus .

Nacos 内置Ribbon，可以用于消费端的负载均衡。

### 2、主要功能：

（1）替代 Eureka 成为服务注册中心。

（2）替代 Config 成为服务配置中心。

（3）动态DNS服务

（4）服务及其元数据管理

### 3、服务注册中心比较

简单比较

注意：参考时间（2019-2020）

| 服务注册与发现 | CAP模型 | Web 界面 | 社区活跃度 |
| -------------- | ------- | -------- | ---------- |
| Eureka         | AP      | 支持     | 低         |
| Zookeeper      | CP      | 不支持   | 中         |
| Consul         | CP      | 支持     | 高         |
| Nacos          | AP      | 支持     | 高         |

特性比较：

![Nacos 比较](/images/springcloud/nacos-compare-01.jpg)



服务发现比较：

![Nacos 比较](/images/springcloud/nacos-compare-02.jpg)

Nacos 支持AP模式和CP模式切换。

一般而言，不需要存储服务级别的信息且服务实例是通过Nacos-client 注册，并且能保持心跳进行上报，就可以选择AP模式，即高可用-分区容错模式。AP模式为了服务高可用，减弱了一致性，因为AP模式只支持注册临时实例。当前主流服务Spring Cloud 和 Dubbo 服务都适用于AP模式。

如果要存储服务级别的信息或存储配置信息，那么CP是必须的，K8S服务和DNS服务适用于CP模式，即强一致-分区容错模式。CP模式为了数据一致，降低了服务的高可用性，CP模式支持注册持久化实例，此时以Raft 协议为集群运行模式，该模式下注册实例前必须先注册服务，如果服务不存在则返回错误。

Nacos 切换模式命令为：

```bash
curl -X PUT '$NACOS_SERVER:8848/nacos/v1/ns/operator/switches?entry=serverMode&value=CP'
```



## 三、Nacos 安装

### 1、准备条件

安装 JDK1.8 和 Maven 3 



### 2、下载

gitHub 下载地址：https://github.com/alibaba/nacos/releases

下载之后解压安装包即可，Windows 直接运行bin目录的startup.cmd，Linux 直接运行bin目录的startup.sh。

启动成功后访问web界面：http://localhost:8848/nacos

登录账户/密码：nacos/nacos

登录之后进入系统：

![Nacos Web 控制台](/images/springcloud/nacos-web-demo-01.jpg)

## 四、简单实例

官方实例：https://nacos.io/zh-cn/docs/quick-start-spring-cloud.html

### 1、作为服务注册中心

创建3个工程，2个作为服务提供端，1个作为服务消费端。

服务端：nacos-provider-6070，nacos-provider-6071

客户端：nacos-consumer-6080

### 1.1、服务提供端

#### （1）引入POM依赖

父工程：

```xml
<!--spring cloud alibaba 2.1.0.RELEASE-->
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-alibaba-dependencies</artifactId>
  <version>2.1.0.RELEASE</version>
  <type>pom</type>
  <scope>import</scope>
</dependency>
```

服务端工程POM依赖：

```xml
<dependencies>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency><dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.62</version>
</dependency>
 </dependencies>
```

#### （2）YML配置

```yaml
server:
  port: 6070

spring:
  application:
    name: nacos-user-provider
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #配置Nacos地址

management:
  endpoints:
    web:
      exposure:
        include: '*'
```

#### （3）主启动类

```java
package com.xiaocai.springcloud.alibaba;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@EnableDiscoveryClient
@SpringBootApplication
public class NacosProvider_6070 { 
    public static void main(String[] args) {
        SpringApplication.run(NacosProvider_6070.class,args);
    }
}
```

controller 业务类：

```java
package com.xiaocai.springcloud.alibaba.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;


@RestController
public class UserProviderController
{
    @Value("${server.port}")
    private String serverPort;

    @GetMapping(value = "/v1/user/nacos/{id}")
    public String getUser(@PathVariable("id") Integer id)
    {
        return "nacos registry, serverPort: "+ serverPort+"\t id"+id;
    }
}
```

#### （4）第二个服务提供端

为了模拟消费端的负载均衡效果，按上述步骤，再建一个服务提供端，端口为6071。

#### （5）测试服务注册

启动工程，进行访问地址：http://lcoalhost:6070/v1/provider/nacos/1

查看Nacos控制台，点击“服务列表”，如果看到微服务名`nacos-user-provider`说明服务注册成功。

### 1.2、服务消费端

#### （1）引入POM依赖

父工程依赖已经引入。

服务端工程POM依赖：

```xml
<dependencies>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency><dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.62</version>
</dependency>
 </dependencies>
```

#### （2）配置相关

YML 配置：

```yaml
server:
  port: 6080


spring:
  application:
    name: nacos-consumer-6080
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848


service-url:
  nacos-user-service: http://nacos-user-provider
        
```

Java 配置：

```java
package com.xiaocai.springcloud.alibaba.config;

import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;


@Configuration
public class ApplicationContextConfig
{
    @Bean
    @LoadBalanced //Ribbon 负载均衡
    public RestTemplate getRestTemplate()
    {
        return new RestTemplate();
    }
}
```



#### （3）主启动类

```java
package com.xiaocai.springcloud.alibaba;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@EnableDiscoveryClient
@SpringBootApplication
public class NacosConsumer_6080 { 
    public static void main(String[] args) {
        SpringApplication.run(NacosConsumer_6080.class,args);
    }
}
```

controller 业务类：

```java
package com.xiaocai.springcloud.alibaba.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;


@RestController
@Slf4j
public class UserNacosConsumerController
{
    @Resource
    private RestTemplate restTemplate;

    @Value("${service-url.nacos-user-service}")
    private String serverURL;

    @GetMapping(value = "/consumer/user/nacos/{id}")
    public String paymentInfo(@PathVariable("id") Long id)
    {
        return restTemplate.getForObject(serverURL+"/v1/user/nacos/"+id,String.class);
    }

}
```



#### （4）测试服务调用

启动工程后，查看Nacos控制台，点击“服务列表”，如果看到微服务名`nacos-consumer-6080`说明服务注册成功。

服务注册成功后，进行访问地址：http://lcoalhost:6080/consumer/user/nacos/11 模拟服务消费端调用服务，多此点击查看，验证是否符合Ribbon的负载均衡效果。



### 2、作为配置中心

将Nacos 服务作为配置中心 Nacos Server，那么需要创建Nacos Client 工程，从配置中心拉取自己的配置。创建2个nacos 客户端：nacos-client-6090，nacos-client-6091

### 2.1 创建客户端

#### （1）引入POM依赖

父工程已经引入。

服务端工程POM依赖，需要引入config模块。

```xml
<dependencies>
    <!--nacos-config-->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    </dependency>
    <!--nacos-discovery-->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    <!--web + actuator-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <!--一般基础配置-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

#### （2）YML配置

和Config Client 的原理一样，写两个配置，application.yml 和 bootstrap.yml，bootstrap.yml 文件加载的优先级要高于 application.yml 文件。

（A）bootstrap.yml 文件内容

```yaml
server:
  port: 6090

spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #服务注册中心地址
      config:
        server-addr: localhost:8848 #配置中心地址
        file-extension: yaml #指定yaml格式的配置

```

（B）application.yml 文件内容

```java
spring:
  profiles:
    active: dev
```



#### （3）主启动类

```java
package com.xiaocai.springcloud.alibaba;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;


@EnableDiscoveryClient
@SpringBootApplication
public class NacosConfigClient_6090
{
    public static void main(String[] args) {
        SpringApplication.run(NacosConfigClient_6090.class, args);
    }
}
```

controller 业务类：

```java
package com.xiaocai.springcloud.alibaba.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;


@RestController
@RefreshScope
public class ConfigClientController
{
    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/config/info")
    public String getConfigInfo() {
        return configInfo;
    }
} 
```

通过 Spring Cloud 原生注解 `@RefreshScope` 实现配置自动更新。

#### （4）在Nacos中添加配置

Nacos 作为配置中心，Nacos Client 会从配置中心拉取配置，所以需要在Nacos 的配置中心添加配置。

**Nacos 配置匹配规则**：

之前在`bootstrap.yml`配置了 `spring.application.name` ，是因为它是构成 Nacos 配置管理 `dataId`字段的一部分。

在 Nacos Spring Cloud 中，`dataId` 的完整格式如下：

```txt
${prefix}-${spring.profiles.active}.${file-extension}
```

- `prefix` 默认为 `spring.application.name` 的值，也可以通过配置项 `spring.cloud.nacos.config.prefix`来配置。

- `spring.profiles.active` 即为当前环境对应的 profile，详情可以参考 [Spring Boot文档](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html#boot-features-profiles)。 **注意：当 spring.profiles.active 为空时，对应的连接符 - 也将不存在，dataId 的拼接格式变成**

- ````
  ${prefix}.${file-extension}
  ````

- `file-exetension` 为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension` 来配置。目前只支持 `properties` 和 `yaml` 类型。

**常用配置文件格式**：

```txt
${spring.application.name}-${spring.profiles.active}.${file-extension}
```

打开Nacos控制台：

点击"服务列表"—打开表格右上角的加号"+"，出现添加配置页面：

![](/images/springcloud/nacos-config-demo-01.jpg)

设置DataId，参考公式

```txt
${spring.application.name}-${spring.profiles.active}.${file-extension}
```

从配置文件中可以找到：

- spring.application.name 的值是 nacos-config-client
- spring.profiles.active 的值是 dev
- file-extension 的值是 yaml

所以DataId 的值是：`nacos-config-client-dev.yaml`



#### （5）测试拉取配置

注意，需要先在Nacos 控制台-"配置管理"-"配置管理"的栏目下有没有对应的yaml配置文件，如果没有请先添加。

启动工程nacos-config-client:6090，进行访问地址：http://lcoalhost:6090//config/info

修改下Nacos中的yaml配置文件内容，再次调用刚才的访问地址：http://lcoalhost:6090//config/info，会发现配置已经刷新。



### 3、Nacos配置中心分类配置

在实际开发中，会有多个项目、多个环境问题。

dev开发环境/test测试环境/prod生产环境

Nacos 可以保证指定环境启动时，服务能正确读取到Nacos 配置中心里相应的环境配置文件。

Nacos 设计：

**Namespace + Group + Data Id**

Namespace 是用于区分部署环境，Group 和 Data Id 逻辑上区别两个目标对象。

![Nacos 分组配置模型](/images/springcloud/nacos-config-data-model.jpg)

默认情况下：

**Namespace=public，Group=DEFAULT_GROUP，Cluster=DEFAULT**

Namespace 主要是用来实现隔离。

如有三个环境：开发环境、测试环境、生产环境。此时可以创建三个Namespace，不同的Namespace之间是隔离的。

Group 默认是DEFAULT_GROUP，Group 可以把不同的服务划分到同一个组。

Service 是微服务，一个Service 可以包含多个Cluster（集群），Nacos 默认Cluster是DEFAULT，Cluster是对指定微服务的一个虚拟划分（逻辑划分）。比如为了容灾，将Service微服务分别部署在北京机房和上海机房，北京机房的Service微服务取个集群名称BJ，上海机房的Service微服务取个集群名称SH。

Instance 微服务实例。

使用三个进行多环境配置方案：

### 3.1 Data Id 方案

Spring Boot 通过`spring.profile.active`属性就能进行多环境下配置文件的读取。因此可以指定`spring.profile.active`和配置文件的DataID来使不同环境下读取不同的配置。

配置方案：**默认空间+默认分组+不同环境的DataID**

![DataId 方案示例](/images/springcloud/nacos-config-demo-dataId.jpg)

### 3.2 Group 方案

通过Group实现环境区分。

在Nacos控制台新建两个不同的Group，分别为TEST_GROUP和DEV_GROUP。

然后新建配置文件DataID，此时Data Id 格式为：`${spring.application.name}.${file-extension}`

在`boostrap.yml` 的config下增加一条group的配置，值可配置为`DEV_GROUP`或`TEST_GROUP`

```YAML
server:
  port: 6090

spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #服务注册中心地址
      config:
        server-addr: localhost:8848 #配置中心地址
        file-extension: yaml #指定yaml格式的配置
        group: DEV_GROUP
```



### 3.3 Namespace 方案

Nacos 控制台-"命名空间"-新建两个不同的命名空间dev和test。

进入服务管理-服务列表查看，列举了命名空间如图

![Namespace 示例](/images/springcloud/nacos-config-demo-namespace.jpg)

在对应的命名空间内添加配置，可以在不同的组创建配置。

在`boostrap.yml` 的config下增加一条namespace的配置，值对应命名空间的ID值。

```yaml
server:
  port: 6090

spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #服务注册中心地址
      config:
        server-addr: localhost:8848 #配置中心地址
        file-extension: yaml #指定yaml格式的配置
        namespace: 3c5e51ab-9904-4485-884a-5bfbc1285910
        group: DEV_GROUP
```

application.yml 文件：

```yaml
spring:
  profiles:
    active: dev
```

那么此时，服务启动之后，拉取的配置是namespace 的id 为 `3c5e51ab-9904-4485-884a-5bfbc1285910` 即dev 的命名空间，Group 为 DEV_GROUP 下的，Data id 为 dev 环境的配置文件。

此时使用了三级分组。



## 五、Nacos 集群和配置持久化

官方说明：https://nacos.io/zh-cn/docs/cluster-mode-quick-start.html

### 1、集群部署架构图

因此开源的时候推荐用户把所有服务列表放到一个vip下面，然后挂到一个域名下面

<http://ip1>:port/openAPI  直连ip模式，机器挂则需要修改ip才可以使用。

<http://VIP>:port/openAPI  挂载VIP模式，直连vip即可，下面挂server真实ip，可读性不好。

<http://nacos.com>:port/openAPI  域名 + VIP模式，可读性好，而且换ip方便，推荐模式

![Nacos 架构图](/images/springcloud/nacos-config-cluster.jpg)

在百度找了一张图：

![Nacos 架构图](/images/springcloud/nacos-config-cluster-demo.jpg)

### 2、部署方式

官方地址：https://nacos.io/zh-cn/docs/deployment.html

三种部署模式：

- 单机模式 - 用于测试和单机试用。
- 集群模式 - 用于生产环境，确保高可用。
- 多集群模式 - 用于多数据中心场景。

### 2.1 **单机模式下运行Nacos**

**Linux/Unix/Mac**

- Standalone 表示是单机模式： `sh startup.sh  -m standalone`

**Windows**

- 执行`cmd startup.cmd` 或者双击 startup.cmd 文件。

**单机模式支持mysql**

在0.7版本之前，在单机模式时nacos默认使用自带嵌入式数据库（derby）实现数据的存储，不方便观察数据存储的基本情况。0.7版本增加了支持mysql数据源能力，从derby 切换使用mysql具体的操作步骤：

- 1.安装mysql数据库，版本要求：5.6.5+
- 2.初始化mysql数据库，数据库初始化文件：nacos-mysql.sql，可以在`/nacos/conf`目录下找到sql脚本
- 3.修改`conf/application.properties`文件，增加支持mysql数据源配置（目前只支持mysql），文件末尾添加mysql数据源的url、用户名和密码。

```properties
spring.datasource.platform=mysql

db.num=1
db.url.0=jdbc:mysql://192.168.100.125:3306/nacos_devtest?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=123456
```



### 2.2 集群模式下运行Nacos

### 2.2.1. 预备环境准备

官网说明，请确保是在环境中安装使用:

1. 64 bit OS  Linux/Unix/Mac，推荐使用Linux系统。
2. 64 bit JDK 1.8+；[下载](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html).[配置](https://docs.oracle.com/cd/E19182-01/820-7851/inst_cli_jdk_javahome_t/)。
3. Maven 3.2.x+；[下载](https://maven.apache.org/download.cgi).[配置](https://maven.apache.org/settings.html)。
4. 3个或3个以上Nacos节点才能构成集群。

### 2.2.2.下载安装包

两种方式来获取 Nacos。

（1）从 Github 上下载源码方式

下载地址：https://github.com/alibaba/nacos/releases/download/

选择一个版本下载后进行编译。

```
unzip nacos-source.zip
cd nacos/
mvn -Prelease-nacos clean install -U  
cd nacos/distribution/target/nacos-server-1.3.0/nacos/bin
```

（2）下载编译后压缩包方式

下载编译后压缩包方式：https://github.com/alibaba/nacos/releases/download/

- windows： [zip包](https://github.com/alibaba/nacos/releases/download/1.3.0/nacos-server-1.3.0.zip) 
- Linux：[tar.gz包](https://github.com/alibaba/nacos/releases/download/1.3.0/nacos-server-1.3.0.tar.gz) 

下载后解压：

```
unzip nacos-server-1.3.0.zip 或者 tar -xvf nacos-server-1.3.0.tar.gz
cd nacos/bin
```

### 2.2.3.集群环境搭建

（1）环境准备

Linux Centos 

Nacos 集群环境需要3个或3个以上Nacos节点才能构成集群。

下载地址：https://github.com/alibaba/nacos/releases/tag/1.1.4

解压后安装。

安装并配置Mysql 57 ，执行`nacos-mysql.sql`，Linux 如果不想使用命令，也可以使用mysql客户端工具连接后执行。执行之后会有Nacos 的配置表。

![脚本执行完成的表](/images/springcloud/nacos-mysql-config-table.jpg)

（2）修改配置

修改`nacos/conf/application.properties`配置：

```properties
spring.datasource.platform=mysql
 
db.num=1
db.url.0=jdbc:mysql://192.168.100.125:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=12345

```

mysql  授权远程访问

```bash
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
flush privileges;
```

（3）集群配置cluster.conf

3台nacos机器的不同服务端口号，比如：8801,8802,8803

备份集群配置：

```bash
cp cluster.conf   cluster.conf.bk
```

配置集群：

```bash
vim cluster.conf 
```

文件内容：

```txt
192.168.100.125:8801
192.168.100.125:8802
192.168.100.125:8803
```

注意：IP不能写`127.0.0.1`,必须是Linux命令`hostname -i`能够识别的IP

（4）修改启动脚本start.sh

编辑Nacos的启动脚本`startup.sh`，使它能够接受不同的启动端口。

找到`nacos/bin`目录下`startup.sh`文件

![修改对比图1](/images/springcloud/nacos-cluster-update-start-sh-01.jpg)

![修改对比图2](/images/springcloud/nacos-cluster-update-start-sh-02.jpg)

（4）启动集群

进入 `nacos/bin `目录，带上端口分别启动：

```bash
./startup.sh -p 8801
./startup.sh -p 8802
./startup.sh -p 8803
```

（5）nginx配置

Nginx 作为负载均衡器，修改nginx的配置文件：

```
...

upstream cluster{                                                        
    server 127.0.0.1:3333;
    server 127.0.0.1:4444;
    server 127.0.0.1:5555;
}

http{
	...
	
	server{
                          
    	listen 8888;
    	server_name localhost;
    	location /{
         	proxy_pass http://cluster;
        }                                            
    }
}
```

检查并重新加载nginx 配置

```bash
nginx -t
nginx -s reload
```

（6）测试

通过Nginx 访问Nacos ：https://192.168.100.125:8888/nacos/#/login

在控制台新建配置后发布，然后连接mysql是否写入新增加的配置，如果写入成功说明配置持久化成功。



## 六、其他

后续学习再补充。

