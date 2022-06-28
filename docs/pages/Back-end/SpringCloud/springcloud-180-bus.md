---
title: SpringCloud Bus
tags: SpringCloud
categories: SpringCloud
summary: 本文记录SpringCloud Bus入门相关学习和整理。
keywords: >-
  Spring Cloud,Spring Cloud 微服务,Spring Cloud Bus ,Spring Cloud bus原理,bus 教程,bus
  使用
author: Small-Rose / 张小菜
abbrlink: 9e02a3e8
date: 2020-09-21 21:00:00
---

## Spring Cloud Bus 消息总线


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

因为Spring Cloud config 配置修改后需要手动重启或者访问一次才能生效，即可进行动态刷新。如果微服务较多时，需要一个一个去访问刷新才能生效......

Spring Cloud Bus消息总线配合Spring Cloud Config使用可以实现配置的动态刷新，可以实现一次访问，处处生效。



## 二、Bus

bus 是一种消息总线。

**关于消息总线**

在微服务架构系统中，通常使用轻量级的消息代理来构建一个共用的消息主题，并让系统中所有微服务实例都连上来。由于该主题产生的消息会被所有实例监听和消费，所以称为消息总线。在总线的各个实例，都可以方便地广播一些需要让其他链接在该主题上的实例都知道的消息。

**基本原理**

Config Client 实例都监听MQ中同一个Topic （默认是Spring Cloud Bus），当一个服务刷新数据的时候，它会把这个信息放入到Topic中，这样其他监听同一个Topic的服务就能得到通知，然后去更新自身的配置。

Spring Cloud Bus通过轻量消息代理连接各个分布的节点，会用在广播状态的变化（例如配置变化）或者其他的消息指令。Spring  Bus的一个核心思想是通过分布式的启动器对spring  boot应用进行扩展，也可以用来建立一个多个应用之间的通信频道。目前唯一实现的方式是用AMQP消息代理作为通 。

Spring Cloud Bus 用来将分布式系统的节点与轻量级消息连接起来，整合Java的事件处理机制和消息中间件的功能。用于管理和传播所有分布式系统间的消息，本质是利用了MQ的广播机制在分布式的系统中传播消息，可用于广播状态更改、事件推送等，也可以作为微服务间的通信通道。

支持消息代理：**Kafka**和**RabbitMQ**。

![消息总线Bus](/images/springcloud/springcloud-bus.jpg)

Spring Cloud Bus做配置更新的步骤:

1. 提交代码触发post给客户端A发送bus/refresh
2. 客户端A接收到请求从Server端更新配置并且发送给Spring Cloud Bus
3. Spring Cloud bus接到消息并通知给其它客户端
4. 其它客户端接收到通知，请求Server端获取最新配置
5. 全部客户端均获取到最新的配置



##  三、Bus 使用

### 1、Bus 设计思想

#### 1、触发客户端进行广播

利用消息总线触发一个客户端/bus/refresh,而刷新所有客户端的配置。

![触发客户端进行广播示图](/images/springcloud/bus-refresh-01.jpg)



#### 2、触发服务端进行广播

利用消息总线触发一个服务端ConfigServer的/bus/refresh端点,而刷新所有客户端的配置。

![触发服务端进行广播示图](/images/springcloud/bus-refresh-02.jpg)



#### 3、小结

刷新客户端和刷新服务端，推荐刷新服务端。

因为刷新客户端打破了微服务的职责单一性，破坏了微服务各节点的对等性。微服务本身是业务模块，它本不应该承担配置刷新职责。如果微服务进行迁移，网络地址会发生变化，需要进行不少的修改。



### 2、RabbitMQ环境配置

#### （1）RabbitMQ 安装

RabbitMQ 的环境依赖 Erlang：

下载地址：http://erlang.org/download/

RabbitMQ 下载地址：https://dl.bintray.com/rabbitmq/all/rabbitmq-server/

安装完成后进入RabbitMQ安装目录下的sbin目录，打开CMD窗口，运行

```bash
rabbitmq-plugins enable rabbitmq_management
```

该命令是为RabbitMQ 添加可视化插件。

启动RabbitMQ，可以在浏览器访问：http://172.0.0.1:15672/

登录账户和密码：guest/guest

### 3、Bus 动态刷新全局广播

#### 3.1 工程准备

作为配置中心服务端：config-server-3344  

作为配置客户端：config-client-3355  

作为配置客户端：config-client-3366 

#### 3.2.1 配置中心服务端

##### （一）相关pom依赖



引入Bus模块依赖`spring-cloud-starter-bus-amqp`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>com.xiaocai.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>config-center-3344</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
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
        </dependency>
    </dependencies>
</project>
```

##### （二）相关配置

添加mq相关配置

```yaml
server:
  port: 3344
  
spring:
  application:
    name: config-center
  cloud:
    config:
      server:
        git:
          uri:  https://github.com/small-rose/springcloud-config.git  		#git@github.com:small-rose/springcloud-config.git
          search-paths:
            - springcloud-config
      label: master

# 添加mq的配置
rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest

eureka:
  client:
    service-url:
      defaultZone:  http://localhost:7001/eureka

# bus 刷新端点暴露
management:
  endpoints:
    web:
      exposure:
        include: 'bus-refresh'
```

##### （三）主启动类

```java
package com.xiaocai.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer
public class ConfigCenter_3344 {
    public static void main(String[] args) {
            SpringApplication.run(ConfigCenter_3344.class,args);
        }
}
```

配置中心的服务基本完成。

#### 3.2.2 配置客户端3355

##### （一）相关pom依赖

引入mq 的依赖

```xml
		<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
		<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
		...
```

##### （二）添加mq的配置

```yaml
server:
  port: 3355

spring:
  application:
    name: config-client-3355
  cloud:
    config:
      label: master
      name: config
      profile: dev
      uri: http://localhost:3344

rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest

eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka
      
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

##### （三）主启动类

```java
package com.xiaocai.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer
@EnableEurekaClient// 
public class ConfigClient_3355 {
    public static void main(String[] args) {
            SpringApplication.run(ConfigClient_3355.class,args);
        }
}
```

注意：有些注解可能遗漏了，请自行补充。



#### 3.2.3 配置客户端3366

按照3355的步骤，再来一次，只是端口改成3366换了。



#### 3.2.4 动态刷新测试

启动config server 进行测试验证：

访问1：http://config-3344.com:3344/master/config-dev.yml 

启动config-client-3355 ，测试访问：http://localhost:3355/configInfo 验证是否能获取配置

启动config-client-3366 ，测试访问：http://localhost:3366/configInfo 验证是否能获取配置

如果一切正常表示config 使用环境OK，否则需排查问题，验证config 组件环境使用正常。

**工程不要停止**，再次修改config-dev.yml 的内容：

```yaml
config:
  info: this is master config-dev from github --- version=1.5 --- config bus
  
```

提交到github远程仓库。

此时手工发送post请求刷新端口为3344的config server工程，执行命令：

```bash
curl -X POST "http://localhost:3344/actuator/bus-refresh"
```

 此时，

测试访问：http://localhost:3355/configInfo 验证是否获取到更新的配置；

测试访问：http://localhost:3366/configInfo 验证是否获取到更新的配置；

**工程不要停止**，再次修改config-dev.yml 的内容可以进行反复测试验证。

### 4、Bus 动态刷新定点广播

定点广播，是指只让某一个实例生效，其他实例不进行广播。

刷新的URL：**http://域名或IP:配置中心的端口号/actuator/bus-refresh/{destination}**

此时/bus/refresh请求不再发送到具体的服务实例上，而是发给config server并通过destination参数类指定需要更新配置的服务或实例。

之前的可以继续测试，只是将post请求刷新的命令改为：

```bash
curl -X POST "http://localhost:3344/actuator/bus-refresh/config-client:3355"
```

### 5、广播通知流程

![广播流程示图](/images/springcloud/bus-refresh-work.jpg)

## 四、Bus 其他

后续学习再补充。



