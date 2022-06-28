---
title: SpringCloud OpenFeign
tags: SpringCloud
categories: SpringCloud
summary: 本文记录SpringCloud OpenFeign相关学习和整理。
keywords: >-
  Spring Cloud,Spring Cloud 微服务,OpenFeign,OpenFeign 原理,OpenFeign 工作原理,OpenFeign
  配置,OpenFeign 使用,OpenFeign Web Service,OpenFeign 服务调用，OpenFeign 接口调用,OpenFeign
  使用方法
author: Small-Rose / 张小菜
abbrlink: b27245de
date: 2020-09-16 14:00:00
---

## SpringCloud OpenFeign 服务接口调用


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

Spring Cloud OpenFeign是一种声明式的REST客户端，是一种面向接口编程的实现。是对HTTP调用一种封装和简化。

**主要特征**：

声明式REST客户机：Feign创建用JAX-RS或springmvc注释修饰的接口的动态实现。

**github**：https://github.com/spring-cloud/spring-cloud-openfeign 

官网：

https://spring.io/projects/spring-cloud-openfeign

文档地址：

https://docs.spring.io/spring-cloud-openfeign/docs/2.2.5.RELEASE/reference/html/

同类的功能

RestTemplate

HttpClient





## 二、OpenFeign 

Spring Cloud OpenFeign是一种声明式的REST客户端。

OpenFeign使用编写java http 客户端变得更容易方便。

在实际开发中，对服务依赖的调用可能不止一处，一个接口会被多出调用，所以可以针对每一个微服务自行封装一些客户端来包装一些依赖服务的调用。

所以Feign在此基础上做了进一步封装，让它来定义和实现依赖服务接口。在OpenFeign 的实现下，只需要创建一个接口并使用注解的方式来配置即可完成对服务提供方的接口绑定，简化了使用Ribbon 时自己封装服务调用客户端的开发。

OpenFeign  已集成了Ribbon。

Feign 和 OpenFeign 区别

| Feign                                                        | OpenFeign                                                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Feign 是 spring cloud 组件中一个轻量级Restful的HTTP服务客户端。<br>Feign内置了Ribbon，用来做客户端负载均衡，去调用服务注册中心的服务。Feign的使用方式是使用Feign的注解定义接口调用这个接口。 | OpenFeign是spring Cloud在Feign的基础上支持了SpringMVC的注解，如@RequestMapping等。<br><br />OpenFeign的@FeignClient可以解析SpringMVC的注解下的接口，并通过动态代理的方式产生实现类，实现类中做负载均衡并调用其他服务。 |
| artifact id <br>`spring-cloud-starter-feign`                 | artifact id <br>`spring-cloud-starter-openfeign`             |





## 三、OpenFeign 使用

**用在消费端**

作为消费端的组件必定是用在消费端/调用者工程中使用。

 

### 1、POM中引入依赖

```xml
		<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```

### 2、启用OpenFeign

在消费端的工程主启动类上添加 `@EnableFeignClients`注解启用OpenFeign。

示例：

```java
@SpringBootApplication
@EnableFeignClients
public class UserFeignApplication {

    public static void main(String[] args) {
        SpringApplication.run(UserFeignApplication.class, args);
    }

}
```



### 3、调用接口封装。

接口和平时写的service 接口类似，需要注意的是在调用者/消费端工程。

注意：

（1）写的是接口类，需要`@Component`注解；

（2）需要`@FeignClient`注解，声明要声明指定的微服务名称；

（3）调用方法的地址即服务提供方springMVC中的地址。

```java
package com.xiaocai.springcloud.service;

import com.xiaocai.springcloud.entities.CommonResult;
import com.xiaocai.springcloud.entities.User;
import feign.Param;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@Component
@FeignClient(value = "MS-CLOUD-USER-SERVICE")
public interface UserFeignService {

    @GetMapping(value = "/v1/user/get/{id}")
    public CommonResult getUserById(@PathVariable("id") Long id);
    
}
```

### 4、接口使用

在消费端的controller中，和平时调用service 一样，

```java
	@Resource
    private UserFeignService userFeignService;
```

消费端的controller示例：

```java
package com.xiaocai.springcloud.controller;

import com.xiaocai.springcloud.entities.CommonResult;
import com.xiaocai.springcloud.entities.User;
import com.xiaocai.springcloud.service.UserFeignService;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@RestController
public class UserFeignController {

    @Resource
    private UserFeignService userFeignService;

    @GetMapping(value = "/consumer/user/get/{id}")
    public CommonResult<User> getUserById(@PathVariable("id") Long id){
       return userFeignService.getUserById(id);
    }
}
```

### 5、启动测试

访问消费端工程路径，验证是否能正常访问。



## 四、OpenFeign超时控制

默认情况下，OpenFeign客户端只等待1秒，但是服务端处理可能需要超过1秒，导致OpenFeign客户端超时，直接返回报错。所以需要进行超时控制。

因为OpenFeign 集成了Ribbon，所以可以使用Ribbon的配置来进行控制：

```yaml
ribbon:
  ReadTimeout:  5000
  ConnectTimeout: 5000

```



## 五、OpenFeign日志

### 1、日志级别

OpenFeign 提供了日志打印给你，可以通过配置来调整日志级别，从而监控OpenFeign中Http的请求细节。

OpenFeign日志级别：

- NONE ：默认，不显示任何日志；
- BASIC ：仅记录请求方法、URL、响应状态码、执行时间；
- HEADERS ：除了BASIC定义的信息之外，还有请求头和响应头信息；
- FULL ：全部请求和响应信息，BASIC  + HEADERS  + BODY + DATA



### 2、日志配置

（1）OpenFeign的日志配置

```java
package com.xiaocai.springcloud.config;

import feign.Logger;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FeignConfig {

    @Bean
    Logger.Level feignLoggerLevel(){
        return Logger.Level.FULL;
    }
}

```

（2）YML文件需要开启日志的OpenFeign客户端

```yaml
logging:
  level:
    com.xiaocai.springcloud.service.UserFeignService: debug
 
```

添加以上配置后重新启动测试，在后头控制台查看输出日志



## 六、其他

后续遇到再补充。

