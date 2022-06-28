---
title: SpringCloud 微服务学习
tags: SpringCloud
categories: SpringCloud
summary: 本文记录SpringCloud入门相关学习和整理。
keywords: 'SpringCloud,SpringCloud 微服务'
author: Small-Rose / 张小菜
abbrlink: bad02d94
top: true
date: 2020-09-12 16:00:00
---

## SpringCloud 微服务学习


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

## 一、关于微服务

### 1、马丁福勒

微服务提出人，马丁.福勒（Martin Fowler） 这样描述微服务：
论文网址： https://martinfowler.com/articles/microservices.html



> 微服务架构是⼀种架构模式，它提倡将单⼀应⽤程序划分成⼀组⼩的服务，服务之间互相协调、互相配合，为⽤户提供最终价值。每个服务运⾏在其独⽴的进程中，服务与服务间采⽤轻量级的通信机制互相协作（通常是基于HTTP协议的RESTful API）。每个服务都围绕着具体业务进⾏构建，并且能够被独⽴的部署到⽣产环境、类⽣产环境等。另外，应当尽量避免统⼀的、集中式的服务管理机制，对具体的⼀个服务⽽⾔，应根据业务上下⽂，选择合适的语⾔、⼯具对其进⾏构建。

### 2、定义与思想

强调的是服务的大小，它关注的是某一个点，是具体解决某一个问题的实际服务请求对应服务的一个服务应用，可以看作Eclipse里面的一个个微服务工程/或者Module。



微服务化的核心就是将传统的一站式应用，根据业务拆分成一个一个的服务，彻底地去耦合,每一个微服务提供单个业务功能的服务，一个服务做一件事，从技术角度看就是一种小而独立的处理过程，类似进程概念，能够自行单独启动或销毁，拥有自己独立的数据库。



微服务核心思路就是分而治之。

对于微服务中的服务可以这么理解：服务是一个可以独立运行、提供范围有限的功能（可以是业务功能，也有可能是非业务功能）的组件。

### 3、优缺点

微服务架构的优缺点

| 优点                           | 缺点                                             |
| ------------------------------ | ------------------------------------------------ |
| 松耦合（一系列的小服务集合）   | 可用性降低（远程调用可能会不稳定，善处链路雪崩） |
| 抽象（调用某服务才能修改数据） | 分布式事务棘手                                   |
| 独立（独立编译、打包、部署）   | 全能对象阻碍业务拆分（某些对象参与较多业务模块） |
| 更高可用性和弹性（各自上下线） | 学习难度曲线加大                                 |
|                                | 组织架构变更                                     |

### 4、交互原则

微服务交互主流基本原则：

（1）使用REST协议，并使用HTTP作为服务调用协议，使用标准动词（GET/PUT/POST/DELETE）

（2）使用URI表达出要解决的问题、提供的方法、资源与资源的关系

（3）使用JSON数据格式（轻量级数据格式、自带序列化和反序列化）

（4）使用HTTP标准状态码



### 5、微服务架构

![微服务架构](/images/springcloud/micro-services-work.jpg)

### 6、主要技术栈

微服务的主要技术栈：

| 微服务条目                             | 主流技术                                                     |
| -------------------------------------- | ------------------------------------------------------------ |
| 服务开发                               | Springboot、Spring、SpringMVC                                |
| 服务配置与管理                         | Netflix公司的Archaius、阿里的Diamond等                       |
| 服务注册与发现                         | Eureka、Consul、Zookeeper、Nacos等                           |
| 服务调用                               | Rest、RPC、gRPC                                              |
| 服务熔断器                             | Hystrix、sentinel、Envoy等                                   |
| 负载均衡                               | Ribbon、Nginx等                                              |
| 服务接口调用(客户端调用服务的简化工具) | Feign等                                                      |
| 消息队列                               | Kafka、RabbitMQ、ActiveMQ等                                  |
| 服务配置中心管理                       | SpringCloudConfig、Nacos、Chef等                             |
| 服务路由（API网关）                    | Zuul、Geteway等                                              |
| 服务监控                               | Zabbix、Nagios、Metrics、Spectator等                         |
| 全链路追踪                             | Zipkin，Brave、Dapper等                                      |
| 服务部署                               | Docker、OpenStack、Kubernetes等                              |
| 数据流操作开发包                       | SpringCloud Stream（封装与Redis,Rabbit、Kafka等发送接收消息） |
| 事件消息总线                           | Spring Cloud Bus、Nacos等                                    |
| ......                                 |                                                              |

## 二、关于SpringCloud

### 1、SpringCloud干什么的

SpringCloud，是基于SpringBoot提供了一套微服务解决方案，包括服务注册与发现，配置中心，全链路监控，服务网关，负载均衡，熔断器等组件，除了基于NetFlix的开源组件做高度抽象封装之外，还有一些选型中立的开源组件。

SpringCloud利用SpringBoot的开发便利性巧妙地简化了分布式系统基础设施的开发，SpringCloud为开发人员提供了快速构建分布式系统的一些工具，包括配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞选、分布式会话等等,它们都可以用SpringBoot的开发风格做到一键启动和部署。

SpringCloud 即分布式微服务架构下的一站式解决方案，是各个微服务架构落地技术的集合体，俗称微服务全家桶。

### 2、与spring boot

**SpringCloud与SpringBoot**

SpringBoot专注于快速方便的开发单个个体微服务。

SpringCloud是关注全局的微服务协调整理治理框架，它将SpringBoot开发的一个个单体微服务整合并管理起来，
为各个微服务之间提供，配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞选、分布式会话等等集成服务

SpringBoot可以离开SpringCloud独立使用开发项目，但是SpringCloud离不开SpringBoot，属于依赖的关系.

SpringBoot专注于快速、方便的开发单个微服务个体，SpringCloud关注全局的服务治理框架。

### 3、与Dubbo相比

**Dubbo与SpringCloud**

Dubbo是高性能 RPC 的分布式服务，SpringCloud采用的是基于HTTP的REST方式。

SpringCloud提供了一整套的解决方案，Dubbo需要自己进行组装。

因为Dubbo还不够熟悉，对后续再补充。



## 三、SpringCloud 

### 1、主要网站

**SpringCloud 官网** https://spring.io/projects/spring-cloud

**SpringCloud 中文网** http://springcloud.cc/

**SpringCloud 中文社区** http://springcloud.cn/



### 2、SpringCloud 版本选择

SpringCloud 版本不是数字表示，是以字母命名，Axxx、Bxxx、Cxxx.....

当前的最新版本是：Spring Cloud Hoxton.SR8 

搭配spring Boot的版本是2.2或2.3

以下推荐搭配表：

<table class="tableblock frame-all grid-all stretch">
<caption class="title">SpringCloud与SpringBoot 搭配</caption>
<colgroup>
<col style="width: 60%;">
<col style="width: 40%;">
</colgroup>
<thead>
<tr>
<th class="tableblock halign-left valign-top">Release Train</th>
<th class="tableblock halign-left valign-top">Boot Version</th>
</tr>
</thead>
<tbody>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><a href="https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Hoxton-Release-Notes">Hoxton</a></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock">2.2.x, 2.3.x (Starting with SR5)</p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><a href="https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Greenwich-Release-Notes">Greenwich</a></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock">2.1.x</p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><a href="https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Finchley-Release-Notes">Finchley</a></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock">2.0.x</p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><a href="https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Edgware-Release-Notes">Edgware</a></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock">1.5.x</p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><a href="https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Dalston-Release-Notes">Dalston</a></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock">1.5.x</p></td>
</tr>
</tbody>
</table>

### 3、代码结构

有多个微服务，会有SpringBoot工程，springboot 较多的maven 依赖都是一致的，防止子工程引入依赖版本不一致问题，使用统一的父工程进行管理。



父工程 `pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>

  <groupId>com.xiaocai.springcloud</groupId>
  <artifactId>cloud2020</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>pom</packaging>


  <!-- 统一管理jar包版本 -->
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <junit.version>4.12</junit.version>
    <log4j.version>1.2.17</log4j.version>
    <lombok.version>1.16.18</lombok.version>
    <mysql.version>5.1.47</mysql.version>
    <druid.version>1.1.16</druid.version>
    <mybatis.spring.boot.version>1.3.0</mybatis.spring.boot.version>
  </properties>

  <!-- 子模块继承之后，提供作用：锁定版本+子modlue不用写groupId和version  -->
  <dependencyManagement>
    <dependencies>
      <!--spring boot 2.2.2-->
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-dependencies</artifactId>
        <version>2.2.2.RELEASE</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
      <!--spring cloud Hoxton.SR1-->
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dependencies</artifactId>
        <version>Hoxton.SR8</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
      <!--spring cloud alibaba 2.2.0.RELEASE-->
      <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-alibaba-dependencies</artifactId>
        <version>2.2.0.RELEASE</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>

      <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>${mysql.version}</version>
      </dependency>
      <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>${druid.version}</version>
      </dependency>
      <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>${mybatis.spring.boot.version}</version>
      </dependency>
      <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>${junit.version}</version>
      </dependency>
      <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>${log4j.version}</version>
      </dependency>
      <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>${lombok.version}</version>
        <optional>true</optional>
      </dependency>
    </dependencies>
  </dependencyManagement>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
          <fork>true</fork>
          <addResources>true</addResources>
        </configuration>
      </plugin>
    </plugins>
  </build>

</project>

```

父工程创建完成执行`mvn:install`将父工程发布到仓库方便子工程继承。

子工程的 `pom.xml`根据需要引入依赖即可，可以省略版本号。



