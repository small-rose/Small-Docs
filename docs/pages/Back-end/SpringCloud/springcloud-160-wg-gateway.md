---
title: SpringCloud Gateway
tags: SpringCloud
categories: SpringCloud
summary: 本文记录SpringCloud Gateway相关学习和整理。
keywords: >-
  Spring Cloud,Spring Cloud 微服务,Gateway,Gateway 原理,Gateway 工作原理,Gateway
  配置,Gateway 使用,Gateway 网关配置,Gateway 服务网关
author: Small-Rose / 张小菜
abbrlink: bc16d093
date: 2020-09-18 22:00:00
---

## SpringCloud Gateway 服务网关


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

https://github.com/Netflix/zuul/wiki

zuul 2 难产了....

Spring Cloud 就整了个SpringCloud Gateway

官方的使用说明文档：

https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/

### 1、基本介绍

Gateway 是在 spring 的生态系统圈之上构建的API 网关服务组件。

SpringCloud Gateway 是基于Spring5，SpringBoot 2 和 Project Reactor等技术，可以提供一种简单而有效的方式来对API进行路由管理，还提供了一些强大的过滤功能。

如：安全、监控、熔断、限流、重试等

相比较而言，Zuul 1.x 是非Reactor 模式的老版本。为了提升网关性能，Gateway  基于WebFlux 框架实现，而WebFlux框架底层使用了高性能的Reactor模式通信框架Netty。

Gateway   目标提供统一的路由方式，且基于Filter链的方式提供网关的基本功能。

SpringCloud Gateway 可以完成代理、鉴权、流量控制、熔断、日志监控等。

### 2、架构角色介绍

![Gateway在架构中的角色](/images/springcloud/gateway-01.jpg)

## 二、SpringCloud Gateway

### 1、主要特点

（1）基于Spring5，SpringBoot 2 和 Project Reactor等技术构建。

（2）动态路由；能够匹配任何请求属性。

（3）对路由进行断言（Predicate）和过滤（Filter），编写方便；

（4）集成Hystrix的断路器功能；

（5）集成Spring Cloud 服务发现功能；

（6）对请求进行限流；

（7）支持路径重写；



### 2、Gateway与 Zuul

在Spring Cloud Finchley 正式版之前官方推荐网关是Netflix 提供的Zuul，现在的最新版是 Spring Cloud Hoxton 。

| Zuul                                                         | Gateway                                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Zuul 1.x 是基于一个阻塞I/O 的API网关                         | 基于Netty的非阻塞IO的API网关。                               |
| Zuul 1.x 是基于Servlet2.5 使用阻塞架构，不支持长连接（如WebSocket）<br>Zuul 的设计模式和Nginx 较像，每次I/O操作都是从工作线程中选择一个执行，请求线程被阻塞到工作线程完成，但是差别是Ngixn是C++实现，Zuul 是用Java实现的，JVM本身就有第一次加载较慢的情况，使Zuul的性能相对较差。 | 基于Spring5，SpringBoot 2 和 Project Reactor等技术构建。Gateway  基于WebFlux 框架实现，底层是非阻塞的Netty，webFux 是典型的非阻塞异步响应式框架，支持非阻塞+函数式编程。支持长连接，支持websocket。 |
| Zuul 2.x 理念更先进，计划基于Netty 非阻塞支持长连接，但Spring Cloud 没有整合。Zuul 2比Zuul1 有较大提升。 | Gateway 属于spring 生态圈，与spring 相关组件紧密集成，更好的组合适配，更好的开发体验。 |

### 3、Gateway组件

Gateway 三个核心的概念：

- Route 路由。路由是构建网关的基本模块，它由ID，目标URI，一系列的断言和过滤器组成，如果断言为true则匹配该路由。
- Predicate 断言。参考的是java8的`java.util.function.Predicate`开发人员可以匹配HTTP请求中的所有内容（例如请求头或请求参数），如果请求与断言相匹配则进行路由。
- Filter 过滤。指的是Spring框架中GatewayFilter的实例，使用过滤器，可以在请求被路由前或者之后对请求进行修改。

## 三、Gateway-路由使用

新建微服务ms-cloud-gateway，端口2244

### 1、引入POM依赖

```xml
 <!--新增gateway-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
```

### 2、添加配置

```yaml
server:
  port: 2244
  
spring:
  application:
    name: ms-cloud-gateway

eureka:
  instance:
    hostname: ms-cloud-gateway-service
  client:
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka

```

### 3、启动类

其实就是之前的Eureka 服务注册服务发现。

```java
package com.xiaocai.springcloud;


import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
public class GateWayMain2244 {
    public static void main(String[] args) {
            SpringApplication.run( GateWayMain2244.class,args);
        }
}

```

### 4、网关路由映射

如果不想暴露微服务的6001端口，希望在6001外面套一层2244。

关于网关路由映射的配置方式，有两种方式，这里先使用YML操作方式、Java的方式参见6。

```yaml
server:
  port: 2244
  
spring:
  application:
    name: ms-cloud-gateway
  cloud:
    gateway:
      routes:
        - id: user_routh #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:6001   #匹配后提供服务的路由地址
          predicates:
            - Path=/v1/user/get/**   #断言,路径相匹配的进行路由

        - id: user_routh2
          uri: http://localhost:6001
          predicates:
            - Path=/v1/user/lb/**   #断言,路径相匹配的进行路由
eureka:
  instance:
    hostname: ms-cloud-gateway-service
  client:
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka            
```

### 5、测试

启动Eureka 注册中心，启动服务提供方，启动Gateway网关工程。

直接访问（不使用网关）原有地址：http://localhost:6001/v1/user/get/10

通过网关访问地址：http://localhost:2244/v1/user/get/10

### 6、网关路由映射-Java版

只需在代码中注入 `RouteLocator`的Bean即可。

不在YML中写，创建配置类`GateWayConfig`

```java
package com.xiaocai.springcloud.config;

import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class GateWayConfig {

    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder routeLocatorBuilder) {
        RouteLocatorBuilder.Builder routes = routeLocatorBuilder.routes();
        routes.route("path_rote_xiaocai", r -> r.path("/xiaocai").uri("http://zhangxiaocai.com/xiaocai")).build();
        return routes.build();
    }
}
```

### 7、微服务名动态路由

默认情况下Gateway会根据注册中心的服务列表，以注册中心上微服务名为路径创建动态路由进行转发，从而实现动态路由的功能。

修改YML配置：

```yaml
server:
  port: 2244
  
spring:
  application:
    name: ms-cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true  #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: user_routh #路由的ID，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001   #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service # 使用lb协议，表示启用Gateway的负载均衡功能
          predicates:
            - Path=/v1/user/get/**   #断言,路径相匹配的进行路由

        - id: user_routh2
          #uri: http://localhost:8001   #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service
          predicates:
            - Path=/v1/user/lb/**   #断言,路径相匹配的进行路由


eureka:
  instance:
    hostname: cloud-gateway-service
  client:
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
 
```

`lb://serviceName`是`spring cloud gateway`在微服务中自动为我们创建的负载均衡uri。

**动态路由测试**

启动Eureka 服务注册中心，启动两个服务提供者，端口分别是6001,6002，启动Gateway网关。

http://localhost:2244/v1/user/lb

`/v1/user/lb` 方法返回各自的端口号，负载均衡默认使用轮询算法，如果两个端口交替出现，说明Gateway的负载均衡起作用了。



## 四、Gateway-断言

启动Gateway2244工程，可以看到控制台打印信息中有以下字样：

```txt
Loaded RoutePredicateFactory [ xxx]
```

SpringCloud Gateway 将路由匹配作为 Spring WebFlux HandlerMapping基础架构的一部分。

SpringCloud Gateway 包括许多内置的Route Predicate Factory ( 路由断言工厂)。所有这些Predicate 都与HTTP请求的不同属性匹配。多个Route Predicate 工厂可以通过逻辑and进行组合。

SpringCloud Gateway 创建Route 对象时，使用`RoutePredicateFactory` 创建 Predicate 对象，Predicate对象可以赋值给Route。

**常用Route Predicate**：

![常用的Predicate](/images/springcloud/gateway-route-predicate-factories.jpg)

（1）After Route Predicate 在某个时间点之后断言为true，即在时间点之后访问地址开始生效；

时间格式为java8里的带时区的时间：

```java
ZonedDateTime zonedDateTime = ZonedDateTime.now();
System.out.println(zonedDateTime);
```

对应的YML配置：

```yaml

spring:
  application:
    name: ms-cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true  #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: user_routh #路由的ID，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001   #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service # 使用lb协议，表示启用Gateway的负载均衡功能
          predicates:
            - Path=/v1/user/get/**   #断言,路径相匹配的进行路由
            - After=2020-09-28T10:59:34.102+08:00[Asia/Shanghai]
 
```



（2）Before Route Predicate 在某个时间点之前断言为true，即在时间点之后访问地址开始失效；

配置示例：

```yaml
spring:
  application:
    name: ms-cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true  #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: user_routh2
          uri: lb://cloud-payment-service
          predicates:
            - Path=/v1/user/lb/**   #断言,路径相匹配的进行路由
            - Before=2020-09-28T10:59:34.102+08:00[Asia/Shanghai]
```



（3）Between Route Predicate 在某个时间点之后断言为true，即在时间区间内访问地址生效；

配置示例：

```yaml
spring:
  application:
    name: ms-cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true  #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: user_routh2
          uri: lb://cloud-payment-service
          predicates:
            - Path=/v1/user/lb/**   #断言,路径相匹配的进行路由
            - Between=2020-09-28T11:59:34.102+08:00[Asia/Shanghai] ,  2020-09-28T12:03:34.102+08:00[Asia/Shanghai]
```



（4）Cookie Route Predicate 

```yaml
spring:
  application:
    name: ms-cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true  #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: user_routh2
          uri: lb://cloud-payment-service
          predicates:
            - Path=/v1/user/lb/**   #断言,路径相匹配的进行路由
            - Cookie=username,atguigu    #并且Cookie是username=zhangshuai才能访问
```

测试地址：

（5）Header Route Predicate

```yaml
spring:
  application:
    name: ms-cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true  #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: user_routh2
          uri: lb://cloud-payment-service
          predicates:
            - Path=/v1/user/lb/**   #断言,路径相匹配的进行路由
            - Cookie=username,atguigu    #并且Cookie是username=zhangshuai才能访问
            - Header=X-Request-Id, \d+   #请求头中要有X-Request-Id属性并且值为整数的正则表达式
```

curl  http://localhost:2244/v1/user/lb  -H "x=Request-id:!23"

（6）Host Route Predicate

```yaml
spring:
  application:
    name: ms-cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true  #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: user_routh2
          uri: lb://cloud-payment-service
          predicates:
            - Path=/v1/user/lb/**   #断言,路径相匹配的进行路由
            - Cookie=username,atguigu    #并且Cookie是username=zhangshuai才能访问
            - Header=X-Request-Id, \d+   #请求头中要有X-Request-Id属性并且值为整数的正则表达式
            - Host=**.xiaocai.com
```

（7）Method Route Predicate

```yaml
spring:
  application:
    name: ms-cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true  #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: user_routh2
          uri: lb://cloud-payment-service
          predicates:
            - Path=/v1/user/lb/**   #断言,路径相匹配的进行路由
            - Cookie=username,atguigu    #并且Cookie是username=zhangshuai才能访问
            - Header=X-Request-Id, \d+   #请求头中要有X-Request-Id属性并且值为整数的正则表达式
            - Host=**.xiaocai.com
            - Method=GET
```

（8）Path Route Predicate

使用最多的一个。

```yaml
- Path=/v1/user/lb/**   #断言,路径相匹配的进行路由
```



（9）Query Route Predicate

```yaml
spring:
  application:
    name: ms-cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true  #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: user_routh2
          uri: lb://cloud-payment-service
          predicates:
            - Path=/v1/user/lb/**   #断言,路径相匹配的进行路由
            - Cookie=username,atguigu    #并且Cookie是username=zhangshuai才能访问
            - Header=X-Request-Id, \d+   #请求头中要有X-Request-Id属性并且值为整数的正则表达式
            - Host=**.xiaocai.com
            - Method=GET
            - Query=username, \d+ #要有参数名称并且是正整数才能路由
```

Predicate就是为了实现一组匹配规则，使得请求过来找到对应的Route进行处理。



## 五、Gateway-过滤

### 1、Filter简介

路由过滤器可用于修改进入的HTTP请求和返回的HTTP响应，路由过滤器只能指定路由进行使用。

SpringCloud Gateway 内置了多种路由过滤器，都是有GatewayFilter的工厂来产生。

**Filter的生命周期**

- pre 在业务逻辑之前
- post 在业务逻辑之后

**Filter的类型**

- 单一的-GatewayFilter ，官方列举31种
- 全局的-GlobalFilter ，官方列举10种
  - `AddRequestHeader` 
  - `AddRequestParameter` 
  - `AddResponseHeader`  
  - `DedupeResponseHeader` 
  - `Hystrix `
  - `CircuitBreaker`
  - `FallbackHeaders` 
  - `MapRequestHeader` 
  - `PrefixPath` 
  - `PreserveHostHeader` 

官方文档说明：https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#gatewayfilter-factories

后续在实际工作中遇到再补充，大部分基本上都可以见名知意。

### 2、自定义Filter

自定义GlobalFilter可以进行全局的日志记录，统一的网关鉴权等。

若要自定义全局的GlobalFilter，需要类实现两个接口`GlobalFilter`和`Ordered`

**自定义全局的GlobalFilter**

示例：

```java
package com.xiaocai.springcloud.filter;

import lombok.extern.slf4j.Slf4j;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

import java.util.Date;

@Component
@Slf4j
public class MyLogGateWayFilter implements GlobalFilter,Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {

        log.info("*********come in MyLogGateWayFilter: "+new Date());
        String uname = exchange.getRequest().getQueryParams().getFirst("username");
        if(StringUtils.isEmpty(username)){
            log.info("*****用户名为Null 非法用户,(┬＿┬)");
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);//给人家一个回应
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return 0;
    }
}
```

**测试**

启动Eureka 注册中心，启动两个服务提供者，启动Gateway网关。

测试访问地址：http://localhost:2244/v1/user/lb?uname=gtway

再测试路径1：http://localhost:2244/v1/user/lb

再测试路径2：http://localhost:2244/v1/user/lb?uname



