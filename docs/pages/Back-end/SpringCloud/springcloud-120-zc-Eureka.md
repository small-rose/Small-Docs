---
title: SpringCloud Eureka
tags: SpringCloud
categories: SpringCloud
summary: 本文记录SpringCloud Eureka入门相关学习和整理。
keywords: >-
  Spring Cloud,Spring Cloud 微服务,Eureka,Eureka 原理,Eureka 工作原理,Eureka 配置,Eureka
  注册,Eureka 自我保护机制,Eureka 集群，Eureka 集群配置,Eureka 使用
author: Small-Rose / 张小菜
abbrlink: 648d6b26
date: 2020-09-15 16:00:00
---

## SpringCloud Eureka 服务注册与发现


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

`Eureka`是`Netflix`的一个子模块，也是核心模块之一。

`Eureka`是一个基于REST的服务，用于定位服务，以实现微服务中间层服务发现和故障转移。

服务注册与发现对于微服务架构来说是非常重要的，有了服务发现与注册，只需要使用服务的标识符，就可以访问到服务，而不需要修改服务调用的配置文件了。

功能类似于`dubbo`的注册中心，比如`Zookeeper`。

`Eureka` 原是` Spring Cloud` 体系中最核心、默认的注册中心组件。

`Spring Cloud Eureka` 是对 `Netflix` 公司的`Eureka`的二次封装，它实现了服务治理的功能，`Spring Cloud Eureka`提供服务端与客户端，服务端即是`Eureka`服务注册中心，客户端完成微服务向Eureka服务的注册与发现。服务端和客户端均采用Java语言编写。

##不过好像这个东西已经**停更维护**了，但是工作可能会遇到，有必要学习了解一下。

## 二、Eureka

Eureka 采用了 C-S 的设计架构。

Eureka Server 作为服务注册功能的服务器，它是服务注册中心。

而系统中的其他微服务，作为 Eureka 的客户端连接到 Eureka Server并维持心跳连接。这样系统的维护人员就可以通过 Eureka Server 来监控系统中各个微服务是否正常运行。

SpringCloud 的一些其他模块（比如Zuul）就可以通过 Eureka Server 来发现系统中的其他微服务，并执行相关的逻辑。

Eureka包含两个重要组件：`Eureka Server`和 `Eureka Client`
`Eureka Server`提供服务注册服务，各个节点启动后，会在`Eureka Server`中进行注册，这样`Eureka Server`中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观的展示。

`Eureka Client`是一个Java客户端，用于简化`Eureka Server`的交互，客户端同时也具备一个内置的、使用轮询(round-robin)负载算法的负载均衡器。在应用启动后，将会向`Eureka Server`发送心跳(默认周期为30秒)。如果`Eureka Server`在多个心跳周期内没有接收到某个节点的心跳，`Eureka Server`将会从服务注册表中把这个服务节点移除（默认90秒），这是Eureka的**Renew: 服务续约** 机制。



`Eureka Server`注册中心服务端主要对外提供了三个功能：

**服务注册**
 服务提供者启动时，会通过 Eureka Client 向 Eureka Server 注册信息，Eureka Server 会存储该服务的信息，Eureka Server 内部有二层缓存机制来维护整个注册表

**提供注册表**
 服务消费者在调用服务时，如果 Eureka Client 没有缓存注册表的话，会从 Eureka Server 获取最新的注册表

**同步状态**
 Eureka Client 通过注册、心跳机制和 Eureka Server 同步当前客户端的状态。

另外Eureka还有

**Eviction 服务剔除**
 当 Eureka Client 和 Eureka Server 不再有心跳时，Eureka Server 会将该服务实例从服务注册列表中删除，即服务剔除。

**Cancel: 服务下线**
 Eureka Client 在程序关闭时向 Eureka Server 发送取消请求。 发送请求后，该客户端实例信息将从 Eureka Server 的实例注册表中删除。

 Eureka 原理：

![Eureka 原理图](/images/springcloud/eureka-work.jpg)

 与dubbo类似：![Dubbo 原理图](/images/springcloud/dubbo-work.png)

从Eureka 工作原理图可以看出 Eureka 有三个重要的角色：

- Eureka Server  提供服务注册于发现；
- Eureka Provider 服务提供方将服务注册到Eureka Server ，从而使消费方使用；
- Eureka Comsumer 服务消费方从Eureka Server获取测试服务列表，消费服务；



## 三、Eureka使用

注意：更多用法参考官网：https://docs.spring.io/spring-cloud-netflix/docs/2.2.5.RELEASE/reference/html/

### 1、作为服务端

作为服务注册中心的Eureka Server 工程使用：

（1）POM中引入依赖

```xml
   <!--eureka-server服务端 -->
   <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-eureka-server</artifactId>
   </dependency>
```

（2）Config 配置

```yaml
server:
  port: 7001

eureka:
  instance:
    hostname: localhost #eureka服务端的实例名称
  client:
    register-with-eureka: false #false表示不向注册中心注册自己。
    fetch-registry: false #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/        #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址。
```

（3）开启 `Eureka Server` 服务

使用注解 `@EnableEurekaServer` 开启`EurekaServer`服务。

示例：

```java
package com.xiaocai.springcloud;
 
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;
 
@SpringBootApplication
@EnableEurekaServer//EurekaServer服务器端启动类,接受其它微服务注册进来
public class EurekaServer7001_App
{
  public static void main(String[] args)
  {
   	SpringApplication.run(EurekaServer7001_App.class, args);
  }
}
```



### 2、作为客户端

作为服务提供者的Eureka Client 工程使用：

（1）POM引入依赖

```xml
 <!-- 将微服务provider侧注册进eureka -->
   <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-eureka</artifactId>
   </dependency>
   <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-config</artifactId>
   </dependency>
```

（2）Config 配置

作为客服端需要补充Eureka客服端的配置：

```yaml
spring:
  application:
    name: ms-cloud-eureka-provider 

eureka:
  client: #客户端注册进eureka服务列表内
    service-url: 
      defaultZone: http://localhost:7001/eureka
  instance:
    instance-id: eureka-provider7001  # 可以自定义服务名称展示
    prefer-ip-address: true # 访问路径可以显示IP  

```

（3）开启 `Eureka Client `服务。

使用注解 `@EnableEurekaClient` 开启`EurekaClient`服务。

示例：

```java
package com.xiaocai.springcloud;
 
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
 
@SpringBootApplication
@EnableEurekaClient //本服务启动后会自动注册进eureka服务中
public class EurekaClientProvider8001_App
{
  public static void main(String[] args)
  {
   SpringApplication.run(EurekaClientProvider8001_App.class, args);
  }
}
```

### 3、测试

（1）先启动Eureka Server 工程

（2）访问：http://localhost:7001

（3）如果看到相应服务列表存在`ms-cloud-eureka-provider `则注册成功。



### 4、微服务info详细信息

（1）、客服端pom补充

```xml
	<!-- 健康监控 -->
	<dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
```

（2）、父工程补充

```xml
<build>
   <finalName>microservicecloud</finalName>
   <resources>
     <resource>
       <directory>src/main/resources</directory>
       <filtering>true</filtering>
     </resource>
   </resources>
   <plugins>
     <plugin>
       <groupId>org.apache.maven.plugins</groupId>
       <artifactId>maven-resources-plugin</artifactId>
       <configuration>
         <delimiters>
          <delimit>$</delimit>
         </delimiters>
       </configuration>
     </plugin>
   </plugins>
  </build>
```

（3）、客户端配置补充

```yaml
info:
  app.name: microservicecloud
  company.name: zhangxiaocai.cn
  build.artifactId: $project.artifactId$
  build.version: $project.version$
```



## 五、Eureka 自我保护

### 1、现象

关闭服务提供方的客户端，刷新测试时的访问页面，会出现红字。

![](/images/springcloud/eureka-protect.jpg)

**Eureka Server 触发自我保护机制后，页面会出现提示**。

### 2、保护机制

Eureka 自我保护机制，会出现：

（1）即某个微服务不可用了，eureka不会立刻进行服务剔除，依旧会对该微服务的信息进行保存。

（2）Eureka 仍然能够接受新服务的注册和查询请求，但是不会被同步到其它节点上。 

（3）当网络稳定或服务恢复后时，当前实例新的注册信息会被同步到其它节点中。

Eureka 自我保护机制是为了防止误杀服务而提供的一个机制。当个别客户端出现心跳失联时，则认为是客户端的问题，剔除掉客户端；当  Eureka 捕获到大量的心跳失败时，则认为可能是网络问题，进入自我保护机制；当客户端心跳恢复时，Eureka 会自动退出自我保护机制。

如果在保护期内刚好这个服务提供者非正常下线了，此时服务消费者就会拿到一个无效的服务实例，即会调用失败。对于这个问题需要服务消费者端要有一些容错机制，如重试，断路器等。

**通过在 Eureka Server 配置如下参数，开启或者关闭保护机制，生产环境建议打开：**

```yaml
eureka:
  server:
    enable-self-preservation: true
```

## 六、Eureka 集群

### 1、相关概念原理

Eureka 集群：由多台Eureka Server 组成的集群，Eureka 集群中的多台Eureka Server会相互注册，多台Eureka Server相互之间不分主从，各个节点地位平等。每个 Eureka Server 同时也是 Eureka Client 。 

Eureka Server通过相互注册来提高可用，每个节点需要添加一个或多个有效的`serice-url` 指向其他节点。

 如果某台 Eureka Server 宕机，Eureka Client 的请求会自动切换到新的 Eureka Server  节点。当宕机的服务器重新恢复后，Eureka  会再次将其纳入到服务器集群管理之中。当节点开始接受客户端请求时，所有的操作都会进行节点间复制，将请求复制到其它 Eureka Server  当前所知的所有节点中。 

Eureka 集群Server 同步基本原则：只要有一条边将节点连接，就可以进行信息传播与同步。所以，如果存在多个节点，只需要将节点之间两两连接起来形成通路，那么其它注册中心都可以共享信息。 多个 Eureka Server 之间通过 P2P 的方式完成服务注册表的同步。 

Eureka Server 集群之间的状态是采用异步方式同步的，所以不保证节点间的状态一定是一致的，不过基本能保证最终状态是一致的。 

Eureka 集群是为了保障注册中心的稳定性和高可用性。反应出Eureka 是满足CAP中的AP。

###  2、配置

（1）在Eureka 集群里，作为`ureka Server`册中心的同时也是作为`Eureka client` ，每个`Eureka Server` 需要相互注册。`service-url` 需要写多个注册地址，多个地址以逗号分隔`","`

示例：如作为`ureka Server`其中之一的`eureka7001.com`的配置，也要作为`Eureka Client`向其他几个注册中心注册。

```yaml
eureka: 
  instance:
    hostname: eureka7001.com #eureka服务端的实例名称
  client: 
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url: 
      defaultZone: http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
```

（2）在Eureka 集群里，作为服务提供者的`Eureka client `，每个`Eureka Client` 需要注册多个`ureka Server`。`service-url` 需要写多个注册中心的地址，以逗号`","`分隔。

示例：如作为`ureka Client`其中之一的`eureka7001.com`的配置，为保证高可用要向所有注册中心注册。

```yaml
spring:
  application:
    name: ms-cloud-eureka-provider 

eureka:
  client: #客户端注册进eureka服务列表内
    service-url: 
      defaultZone: http://eureka7001:7001/eureka,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
  instance:
    instance-id: eureka-provider7001  # 可以自定义服务名称展示
    prefer-ip-address: true # 访问路径可以显示IP  
```

## 七、其他

后续遇到的问题，后续再补充。

