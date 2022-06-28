---
title: SpringCloud Alibaba
tags: SpringCloud
categories: SpringCloud
summary: 记录SpringCloud Alibaba相关学习和整理。
keywords: >-
  Spring Cloud,Spring Cloud 微服务,Spring Cloud Alibaba ,Spring Cloud
  Alibaba原理,微服务学习教程,Spring Cloud Alibaba 使用,Alibaba 相关组件
author: Small-Rose / 张小菜
abbrlink: 461589aa
date: 2020-09-28 13:00:00
---

## SpringCloud Alibaba


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


## 一、背景

Spring Cloud Netflix项目进入维护模式。[查看官方说明](https://spring.io/blog/2018/12/12/spring-cloud-greenwich-rc1-available-now)

SpringCloud alibaba 出现了。

官方介绍：https://github.com/alibaba/spring-cloud-alibaba/blob/master/README-zh.md

Spring Cloud Alibaba 致力于提供微服务开发的一站式解决方案。此项目包含开发分布式应用微服务的必需组件，方便开发者通过 Spring Cloud 编程模型轻松使用这些组件来开发分布式应用服务。 

相关文档：https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html 

## 二、SpringCloud Alibaba

### 1、主要功能

- **服务限流降级**：默认支持 WebServlet、WebFlux,  OpenFeign、RestTemplate、Spring Cloud Gateway, Zuul, Dubbo 和 RocketMQ  限流降级功能的接入，可以在运行时通过控制台实时修改限流降级规则，还支持查看限流降级 Metrics 监控。
- **服务注册与发现**：适配 Spring Cloud 服务注册与发现标准，默认集成了 Ribbon 的支持。
- **分布式配置管理**：支持分布式系统中的外部化配置，配置更改时自动刷新。
- **消息驱动能力**：基于 Spring Cloud Stream 为微服务应用构建消息驱动能力。
- **分布式事务**：使用 @GlobalTransactional 注解， 高效并且对业务零侵入地解决分布式事务问题。。
- **阿里云对象存储**：阿里云提供的海量、安全、低成本、高可靠的云存储服务。支持在任何应用、任何时间、任何地点存储和访问任意类型的数据。
- **分布式任务调度**：提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。同时提供分布式的任务执行模型，如网格任务。网格任务支持海量子任务均匀分配到所有 Worker（schedulerx-client）上执行。
- **阿里云短信服务**：覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建客户触达通道。



### 2、相关组件

**Sentinel**：把流量作为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

**Nacos**：一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。

**RocketMQ**：一款开源的分布式消息系统，基于高可用分布式集群技术，提供低延时的、高可靠的消息发布与订阅服务。

**Dubbo**：Apache Dubbo™ 是一款高性能 Java RPC 框架。

**Seata**：阿里巴巴开源产品，一个易于使用的高性能微服务分布式事务解决方案。

**Alibaba Cloud OSS**: 阿里云对象存储服务（Object Storage Service，简称 OSS），是阿里云提供的海量、安全、低成本、高可靠的云存储服务。您可以在任何应用、任何时间、任何地点存储和访问任意类型的数据。

**Alibaba Cloud SchedulerX**: 阿里中间件团队开发的一款分布式任务调度产品，提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。

**Alibaba Cloud SMS**: 覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建客户触达通道。



### 3、引入依赖

如果需要使用已发布的版本，在 `dependencyManagement` 中添加如下配置。

```
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2.2.3.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

然后在 `dependencies` 中添加自己所需使用的依赖即可使用。



### 4、版本适配

项目的版本号格式为 `x.x.x` 的形式，其中 x 的数值类型为数字，从 0 开始取值，且不限于 `0 ~ 9 `这个范围。项目处于孵化器阶段时，第一位版本号固定使用 0，即版本号为 `0.x.x` 的格式。

由于 `Spring Boot 1.x.x` 和 `Spring Boot 2.x.x `在 Actuator 模块的接口和注解有很大的变更，且  spring-cloud-commons 从 `1.x.x` 版本升级到 `2.0.0` 版本也有较大的变更，因此我们采取跟 SpringBoot  版本号一致的版本:

- 1.5.x 版本适用于 Spring Boot 1.5.x
- 2.0.x 版本适用于 Spring Boot 2.0.x
- 2.1.x 版本适用于 Spring Boot 2.1.x
- 2.2.x 版本适用于 Spring Boot 2.2.x

