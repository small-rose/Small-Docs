---
title: SpringCloud Zuul
tags: SpringCloud
categories: SpringCloud
summary: 本文记录SpringCloud Zuul入门相关学习和整理。
keywords: >-
  Spring Cloud,Spring Cloud 微服务,Zuul,Zuul 原理,Zuul 工作原理,Zuul  配置,Zuul 使用,Zuul
  网关配置,Zuul 服务网关
author: Small-Rose / 张小菜
abbrlink: fbbe72e1
date: 2020-09-19 18:00:00
---

## SpringCloud Zuul 服务网关


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

Zuul包含主要的功能：

- 请求路由
- 请求过滤
- 代理

其中路由功能负责将外部请求转发到具体的微服务实例上，是实现外部访问统一入口的基础而过滤器功能则负责对请求的处理过程进行干预，是实现请求校验、服务聚合等功能的基础。

Zuul和Eureka进行整合之后，将Zuul自身作为Eureka Client注册进Eureka Server服务治理下的应用，同时从Eureka Server中获得其他微服务的消息，也即以后的访问微服务都是通过Zuul跳转后获得。

https://github.com/Netflix/zuul/wiki/Getting-Started

## 二 、Zuul 使用

创建微服务 ms-zuul-gateway

### 1、引入POM依赖

```xml
     <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-eureka</artifactId>
   </dependency>
   <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-zuul</artifactId>
   </dependency>
```

### 2、修改配置

```yaml
server: 
  port: 2233
 
spring: 
  application:
    name: ms-zuul-gateway
 
eureka: 
  client: 
    service-url: 
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka  
  instance:
    instance-id: Zuul-2233.com
    prefer-ip-address: true 
 
 
info:
  app.name: xiaocai-cloud
  company.name: www.zhangxiaocai.com
  build.artifactId: $project.artifactId$
  build.version: $project.version$


```

注意添加hosts 映射：

```txt
127.0.0.1  testzuul.com
```

### 3、启用Zuul

在工程住启动类添加注解`@EnableZuulProxy`

```java
package com.xiaocai.springcloud;
 
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;
 
@SpringBootApplication
@EnableZuulProxy
public class ZuulGateWayApp
{
  public static void main(String[] args)
  {
   SpringApplication.run(ZuulGateWayApp.class, args);
  }
}
```

### 4、启动测试

启动Eureka 服务中心，启动服务提供者MS-CLOUD-USER-SERVICE

启动网关ms-zuul-gateway

测试直接访问服务提供端：

http://localhost:6001/v1/user/get/2

测试通过网关访问服务提供端：

http://testzuul:2233/ms-cloud-user-service/v1/user/get/2



## 三 、Zuul路由访问映射规则

### 1、添加代理名称

修改配置YML文件

```yaml
zuul: 
  routes: 
    mydept.serviceId: ms-cloud-user-service
    mydept.path: /useropt/**
```

此时可以通过网关访问服务：

不仅可以通过服务名：`http://testzuul:2233/ms-cloud-user-service/v1/user/get/2`

来访问，还可以使用代理名称来访问服务：

`http://testzuul:2233/useropt/v1/user/get/2`



### 2、忽略真实服务名

修改配置YML文件

```yaml
zuul: 
  ignored-services: ms-cloud-user-service 
  routes: 
    mydept.serviceId: ms-cloud-user-service
    mydept.path: /useropt/**
```

`ignored-services`：表示忽略原有的微服务`ms-cloud-user-service`，不允许直接访问了。

如果是多个可以使用`"*"`

```yaml
zuul: 
  ignored-services: *
  routes: 
    mydept.serviceId: ms-cloud-user-service
    mydept.path: /useropt/**
```

### 3、设置统一的公共前缀

修改YML配置：

```yaml

zuul: 
  prefix: /xiaocai
  ignored-services: ms-cloud-user-service 
  routes: 
    mydept.serviceId: ms-cloud-user-service
    mydept.path: /useropt/**
```

此时访问服务可以带上前缀使用：

`http://testzuul:2233/xiaocai/useropt/v1/user/get/2`



## 四、其他

后续遇到有机会再补充。