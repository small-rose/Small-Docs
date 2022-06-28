---
title: SpringCloud Sleuth
tags: SpringCloud
categories: SpringCloud
summary: 本文记录SpringCloud Sleuth相关学习和整理。
keywords: >-
  Spring Cloud,Spring Cloud 微服务,Spring Cloud Sleuth ,Spring Cloud Sleuth
  原理,Sleuth 教程,Spring Cloud Sleuth 使用,Sleuth 链路监控
author: Small-Rose / 张小菜
abbrlink: b99c053f
date: 2020-09-26 12:00:00
---

## SpringCloud Sleuth 分布式请求链路追踪


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

在微服务架构中，一个有客户端发起的请求在后端系统可能会经过多个不同发服务器节点来调用协同产生出最后的请求结果。调用示例如图：

![服务调用示例](/images/springcloud/sleuth-demo-01.jpg)

每个请求都会形成一条服务的分布式服务调用链路，链路中的任何一环出现高延迟或错都会引起整个请求最后的失败。如何快速定位失败是一个问题。

于是，Spring Cloud Sleuth出现了，它提供了一套完整的服务链路跟踪的解决方案。

官方：https://github.com/spring-cloud/spring-cloud-sleuth

## 二、Sleuth

### 1、基本概念

Spring Cloud Sleuth提供了一套完整的服务链路跟踪的解决方案。它在整个分布式系统中能跟踪一个用户请求的过程(包括数据采集，数据传输，数据存储，数据分析，数据可视化)，捕获这些跟踪数据，就能构建微服务的整个调用链的视图，这是调试和监控微服务的关键工具。 

在分布式系统中提供追踪解决方案并且兼容支持了zipkin。

Spring Cloud Sleuth是对Zipkin的一个封装，对于Span、Trace等信息的生成、接入HTTP Request，以及向Zipkin Server发送采集信息等全部自动完成。

Spring Cloud Sleuth的调用链路图 ：

![Spring Cloud Sleuth](/images/springcloud/sleuth-work-01.png)

表示请求链路，一条链路通过Trace Id 唯一标识，Span 标识发起的请求信息，各Span 通过parent id 关联起来。

Trace：类似于树结构的Span集合，表示一条调用链路，存在唯一标识；

span：表示调用链路来源，通俗的理解span就是一次请求信息；

### 2、特点

Spring Cloud Sleuth有4个特点

| 特点              | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| 提供链路追踪      | 通过sleuth可以很清楚的看出一个请求经过了哪些服务， 可以方便的理清服务局的调用关系 |
| 性能分析          | 通过sleuth可以很方便的看出每个采集请求的耗时， 分析出哪些服务调用比较耗时，当服务调用的耗时 随着请求量的增大而增大时，也可以对服务的扩容提 供一定的提醒作用 |
| 数据分析 优化链路 | 对于频繁地调用一个服务，或者并行地调用等， 可以针对业务做一些优化措施 |
| 可视化            | 对于程序未捕获的异常，可以在zipkpin界面上看到                |

### 3、Zipkin

Spring Cloud 从F版开始不需要自己构建Zipkin Server，调用jar包即可。

Zipkin 地址：https://dl.bintray.com/openzipkin/maven/io/zipkin/java/zipkin-server/

Zipkin版本：zipkin-server-xx.xx.x.exec.jar

启动 Zipkin 服务：

```bash
java -jar zipkin-server-2.12.9-exec.jar
```

访问Web：http://localhost:9411/zipkin/



## 三、案例

### 1、服务提供端

Eureka Server 使用之前的。

新建子工程：sleuth-provider-6060

#### （1）引入pom依赖

```xml
		<!--包含了sleuth+zipkin-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
        </dependency>
```



#### （2）YML 配置

```yaml
server:
  port: 6060


spring:
  application:
    name: provider-service-6060
  zipkin:
    base-url: http://localhost:9411
  sleuth:
    sampler:
    probability: 1
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: org.gjt.mm.mysql.Driver
    url: 
    username: root
    password: 

mybatis:
  mapperLocations: classpath:mapper/*.xml
  type-aliases-package: com.xiaocai.springcloud.entities


eureka:
  client:
    register-with-eureka: true
    fetchRegistry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka  #集群版
  instance:
    instance-id: provider-6060
    prefer-ip-address: true
 
```

#### （3）提供方controller

```java
package com.xiaocai.springcloud.controller;
 
import com.xiaocai.springcloud.entities.CommonResult;
import com.xiaocai.springcloud.entities.User;
import com.xiaocai.springcloud.service.UserService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.web.bind.annotation.*;
import org.springframework.cloud.client.discovery.DiscoveryClient;
 
import javax.annotation.Resource;
import java.util.List;
import java.util.concurrent.TimeUnit;
 
 
@RestController
@Slf4j
public class UserProviderController
{
    @Resource
    private UserService userService;
 
    @Value("${server.port}")
    private String serverPort;
 
    @Resource
    private DiscoveryClient discoveryClient;
 
    @PostMapping(value = "/v1/user/create")
    public CommonResult create(@RequestBody User user)
    {
        int i = userService.create(user);
        log.info("*****插入结果："+result);
 
        if(result > 0)
        {
            return new CommonResult(200,"插入数据库成功,serverPort: "+serverPort,i);
        }else{
            return new CommonResult(444,"插入数据库失败",null);
        }
    }
 
    @GetMapping(value = "/v1/user/get/{id}")
    public CommonResult<User> getUserById(@PathVariable("id") Long id)
    {
        User user = userService.getUserById(id);
 
        if(user != null)
        {
            return new CommonResult(200,"查询成功,serverPort:  "+serverPort,user);
        }else{
            return new CommonResult(444,"没有对应记录,查询ID: "+id,null);
        }
    }

    @GetMapping("/v1/user/zipkin")
    public String userZipkin()
    {
        return " i'am userzipkin server fall back，welcome to xiaocai ~";
    }
}
 
```



### 2、服务消费端

新建子工程：sleuth-consumer-6061

#### （1）引入pom依赖

```xml
		<!--包含了sleuth+zipkin-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
        </dependency>
```



#### （2）YML 配置

```yaml
server:
  port: 6061


spring:
  application:
    name: consumer-service-6061
  zipkin:
    base-url: http://localhost:9411
  sleuth:
    sampler:
    probability: 1
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: org.gjt.mm.mysql.Driver
    url: #mysql 链接url
    username: root
    password: #mysql密码

mybatis:
  mapperLocations: classpath:mapper/*.xml
  type-aliases-package: com.xiaocai.springcloud.entities


eureka:
  client:
    register-with-eureka: true
    fetchRegistry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka  #集群版
  instance:
    instance-id: consumer-6061
    prefer-ip-address: true
 
```

#### （3）调用方controller

```java
package com.xiaocai.springcloud.controller;
 
import com.xiaocai.springcloud.entities.CommonResult;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.web.bind.annotation.*;
import org.springframework.cloud.client.discovery.DiscoveryClient;
 
import javax.annotation.Resource;
import java.util.List;
import java.util.concurrent.TimeUnit;
 
 
@RestController
@Slf4j
public class UserConsumerController
{
    @Resource
 	private RestTemplate restTemplate ；
        
    @GetMapping("/consumer/user/zipkin")
    public String paymentZipkin()
    {
        String result = restTemplate.getForObject("http://localhost:6060"+"/v1/user/zipkin/", String.class);
        return result;
    }
  
}
```

### 3、启动测试

启动eureka7001，启动sleuth-provider-6060，启动sleuth-consumer-6061;

访问地址调用服务：http://localhost:6061/consumer/user/zipkin

使用浏览器访问：http://localhost:9411/zipkin/

选择服务，查看依赖关系，就可以展示调用链路过程。