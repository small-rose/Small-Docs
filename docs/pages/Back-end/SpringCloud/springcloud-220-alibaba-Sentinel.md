---
title: SpringCloud Sentinel
tags: SpringCloud
categories: SpringCloud
summary: 记录SpringCloud Sentinel相关学习和整理。
keywords: >-
  Spring Cloud,Spring Cloud 微服务,Spring Cloud Sentinel,Spring Cloud
  Sentinel原理,微服务学习教程,Spring Cloud Sentinel 熔断与限流,Spring Cloud Sentinel
  熔断,Sentinel 限流,Sentinel 配置
author: Small-Rose / 张小菜
abbrlink: 5122a811
date: 2020-09-29 16:00:00
---

## SpringCloud Alibaba Sentinel 实现熔断与限流


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

### 1、背景

随着服务在系统之间的分布越来越重要，可靠性变得越来越重要。Sentinel是一个功能强大的流控制组件，以“流”为突破口，涵盖流量控制、并发限制、电路中断、自适应系统保护等多个领域，保证微服务的可靠性。

- 2012 年，Sentinel 诞生，主要功能为入口流量控制。
- 2013-2017 年，Sentinel 在阿里巴巴集团内部迅速发展，成为基础技术模块，覆盖了所有的核心场景。Sentinel 也因此积累了大量的流量归整场景以及生产实践。
- 2018 年，Sentinel 开源，并持续演进。
- 2019 年，Sentinel 朝着多语言扩展的方向不断探索，推出 [C++ 原生版本](https://github.com/alibaba/sentinel-cpp)，同时针对 Service Mesh 场景也推出了 [Envoy 集群流量控制支持](https://github.com/alibaba/Sentinel/tree/master/sentinel-cluster/sentinel-cluster-server-envoy-rls)，以解决 Service Mesh 架构下多语言限流的问题。
- 2020 年，推出 [Sentinel Go 版本](https://github.com/alibaba/sentinel-golang)，继续朝着云原生方向演进。

### 2、官方地址

Sentinel 项目地址 ：https://github.com/alibaba/Sentinel

官网地址：https://sentinelguard.io/zh-cn/

官方介绍：https://sentinelguard.io/zh-cn/docs/introduction.html

**官方指南**：https://github.com/alibaba/Sentinel/wiki/%E6%96%B0%E6%89%8B%E6%8C%87%E5%8D%97

下载地址：https://github.com/alibaba/Sentinel/releases

**Spring Cloud 官方**：https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html#_spring_cloud_alibaba_sentinel

## 二、Sentinel 

### 1、Sentinel 基本概念

**资源**

资源是 Sentinel 的关键概念。它可以是 Java 应用程序中的任何内容，例如，由应用程序提供的服务，或由应用程序调用的其它应用提供的服务，甚至可以是一段代码。在接下来的文档中，我们都会用资源来描述代码块。

只要通过 Sentinel API 定义的代码，就是资源，能够被 Sentinel 保护起来。大部分情况下，可以使用方法签名，URL，甚至服务名称作为资源名来标示资源。

**规则**

围绕资源的实时状态设定的规则，可以包括流量控制规则、熔断降级规则以及系统保护规则。所有规则可以动态实时调整。

**Sentinel 使用**

Sentinel 的使用可以分为两个部分:

- 核心库（Java 客户端）：不依赖任何框架/库，能够运行于 Java 7 及以上的版本的运行时环境，同时对 Dubbo / Spring Cloud 等框架也有较好的支持（见 [主流框架适配](https://github.com/alibaba/Sentinel/wiki/%E4%B8%BB%E6%B5%81%E6%A1%86%E6%9E%B6%E7%9A%84%E9%80%82%E9%85%8D)）。
- 控制台（Dashboard）：控制台主要负责管理推送规则、监控、集群限流分配管理、机器发现等。

**主要特征**

![Sentinel](/images/springcloud/sentinel-module.jpg)

### 2、Sentinel 功能和设计理念

### 2.1 流量控制

流量控制在网络传输中是一个常用的概念，它用于调整网络包的发送数据。然而，从系统稳定性角度考虑，在处理请求的速度上，也有非常多的讲究。任意时间到来的请求往往是随机不可控的，而系统的处理能力是有限的。我们需要根据系统的处理能力对流量进行控制。Sentinel  作为一个调配器，可以根据需要把随机的请求调整成合适的形状，如下图所示：

![Sentinel 流量 ](/images/springcloud/sentinel-flow-overview.jpg)

流量控制有以下几个角度:

- 资源的调用关系，例如资源的调用链路，资源和资源之间的关系；
- 运行指标，例如 QPS、线程池、系统负载等；
- 控制的效果，例如直接限流、冷启动、排队等。

Sentinel 的设计理念是可以自由选择控制的角度，并进行灵活组合，从而达到想要的效果。

### 2.2 熔断降级

**什么是熔断降级**

除了流量控制以外，降低调用链路中的不稳定资源也是 Sentinel 的使命之一。由于调用关系的复杂性，如果调用链路中的某个资源出现了不稳定，最终会导致请求发生堆积。这个问题和 [Hystrix](https://github.com/Netflix/Hystrix/wiki#what-problem-does-hystrix-solve) 里面描述的问题是一样的。

![熔断降级](/images/springcloud/sentinel-work-hystrix.png)

Sentinel 和 Hystrix 的原则是一致的: 当调用链路中某个资源出现不稳定，例如，表现为 timeout，异常比例升高的时候，则对这个资源的调用进行限制，并让请求快速失败，避免影响到其它的资源，最终产生雪崩的效果。

**熔断降级设计理念**

在限制的手段上，Sentinel 和 Hystrix 采取了完全不一样的方法。

Hystrix 通过[线程池](https://github.com/Netflix/Hystrix/wiki/How-it-Works#benefits-of-thread-pools)的方式，来对依赖(在我们的概念中对应资源)进行了隔离。这样做的好处是资源和资源之间做到了最彻底的隔离。缺点是除了增加了线程切换的成本，还需要预先给各个资源做线程池大小的分配。

Sentinel 对这个问题采取了两种手段:

- 通过并发线程数进行限制

和资源池隔离的方法不同，Sentinel  通过限制资源并发线程的数量，来减少不稳定资源对其它资源的影响。这样不但没有线程切换的损耗，也不需要您预先分配线程池的大小。当某个资源出现不稳定的情况下，例如响应时间变长，对资源的直接影响就是会造成线程数的逐步堆积。当线程数在特定资源上堆积到一定的数量之后，对该资源的新请求就会被拒绝。堆积的线程完成任务后才开始继续接收请求。

- 通过响应时间对资源进行降级

除了对并发线程数进行控制以外，Sentinel 还可以通过响应时间来快速降级不稳定的资源。当依赖的资源出现响应时间过长后，所有对该资源的访问都会被直接拒绝，直到过了指定的时间窗口之后才重新恢复。

### 2.3 系统负载保护

Sentinel 同时提供[系统维度的自适应保护能力](https://sentinelguard.io/zh-cn/docs/system-adaptive-protection.html)。防止雪崩，是系统防护中重要的一环。当系统负载较高的时候，如果还持续让请求进入，可能会导致系统崩溃，无法响应。在集群环境下，网络负载均衡会把本应这台机器承载的流量转发到其它的机器上去。如果这个时候其它的机器也处在一个边缘状态的时候，这个增加的流量就会导致这台机器也崩溃，最后导致整个集群不可用。

针对这个情况，Sentinel 提供了对应的保护机制，让系统的入口流量和系统的负载达到一个平衡，保证系统在能力范围之内处理最多的请求。



## 三、Sentinel 控制台

下载地址：https://github.com/alibaba/Sentinel/releases

下载到本地 sentinel-dashboard-1.8.0.jar

**注意：**

（1）sentinel 控制台需要 java8环境

（2）sentinel 使用的是8080端口。

启动命令：

```bash
java -jar sentinel-dashboard-1.8.0.jar
```

启动之后访问sentinel 控制台地址：http://localhost:8080

登录账户/密码：sentinel/sentinel

## 四、Sentinel 案例

更多的例子可以参考: [Sentinel Demo 集锦](https://github.com/alibaba/Sentinel/tree/master/sentinel-demo) 

创建微服务sentinel-service-8401 。

### 1、引入POM依赖

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

    <artifactId>sentinel-service-8401</artifactId>

    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>

        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-datasource-nacos</artifactId>
        </dependency>

        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
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
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>4.6.3</version>
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

### 2、YML配置

```yaml
server:
  port: 8401

spring:
  application:
    name: sentinel-service
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
    sentinel:
      transport:
        dashboard: localhost:8080
        port: 8719  #默认8719，假如被占用了会自动从8719开始依次+1扫描。直至找到未被占用的端口

management:
  endpoints:
    web:
      exposure:
        include: '*'

```

### 3、主启动类

```java
package com.xiaocai.springcloud.alibaba;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;


@EnableDiscoveryClient
@SpringBootApplication
public class SentinelServiceApp8401
{
    public static void main(String[] args) {
        SpringApplication.run(SentinelServiceApp8401.class, args);
    }
}
```

### 4、业务类

```java
package com.xiaocai.springcloud.alibaba.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class FlowLimitController
{
    @GetMapping("/v1/order/all/list")
    public String all() {
        return "------order all";
    }

    @GetMapping("/v1/order/history/list")
    public String history() {

        return "------order history";
    }

}
```

### 5、测试

启动Nacos 服务，端口8848，http://localhost:8848/nacos/#/login

启动Sentinel 服务，端口8080；启动控制台：`java -jar sentinel-dashboard-1.7.2.jar`，控制台的访问地址：http://localhost:8080 用户名密码：sentinel/sentinel

启动微服务 sentinel-service-8401。

**Sentinel采用的懒加载机制，如果要在Sentinel 控制台查看，需要先访问一次服务。**

访问地址：http://localhost:8401/v1/order/all/list

![服务启动](/images/springcloud/sentinel-web-demo-01.jpg)



## 五、流控规则

### 1、流控元素

![添加流控规则](/images/springcloud/sentinel-liukong-demo-01.png)

资源名：唯一名称

针对来源：sentinel 可以针对调用进行限流，添加微服务名，默认default（不区分来源）

阈值类型/单机阈值：

- QPS 每秒的请求数据，当调用该资源API的QPS达到阈值的时候，进行限流
- 线程数，当调用该api的线程达到阈值的时候，进行限流
- 流控模式：
  - 直接：api达到限流条件时，直接限流
  - 关联：当关联的资源达到阈值时，就限流自己
  - 链路：只记录指定链路上的流量，指定资源从入口资源进入的流量如果达到阈值就进行限流。api级别的针对来源的限流。
- 是否集群：本次测试非集群。
- 流控效果：
  - 快速失败：直接失败、返回抛出的异常，系统默认设置。
  - Warm Up：预热模式，根据codeFactor（冷加载因子，默认是3）从阈值除以codeFactor，经过预热时长后，才达到设置的QPS阈值
  - 排队等待：匀速排队，让请求以匀速的速度通过，阈值类型必须设置为QPS，否则无效。

### 1、流量控制规则

| field           | 说明                                                         | 默认值                      |
| --------------- | ------------------------------------------------------------ | --------------------------- |
| resource        | 资源名，限流规则的作用对象如具体的url                        |                             |
| count           | 限流阀值，超过这个值就会触发                                 |                             |
| grade           | 阈值类型，QPS或者线程数                                      | QPS                         |
| limitApp        | 流控针对的调用来源                                           | default，代表不区分调用来源 |
| strategy        | 判断是根据资源本身还是根据其它关联资源（refResource）还是根据链路入口 | 根据资源本身                |
| controlBehavior | 流控效果（直接拒绝/排队等待/慢启动）                         | 直接拒绝                    |

同一个资源可以同时有多个限流规则。

### 2、流控模式

（1）直接模式

在微服务里新流控规则，资源名：`/v1/order/all/list`

针对来源默认，阈值类型QPS，单机阈值 2，流控模式选直接，流控效果快速失败

快速刷新访问：http://localhost:8401/v1/order/all/list ，查看效果。

```txt
Blocked by Sentinel (flow limiting)
```

（2）关联模式

当关联的资源达到阈值时，就限流自己。当资源A关联的资源B达到了阈值，资源A将被限流。

在微服务里新流控规则，资源名：`/v1/order/all/list`

针对来源默认，阈值类型QPS，单机阈值 2，

流控模式选关联，关联资源`/v1/order/history/list` ，流控效果快速失败。

保存设置。

快速刷新访问资源B：http://localhost:8401/v1/order/history/list  ，也可以使用postman模拟并发密集访问，在postman里新建多线程集合组，将访问地址添加进新的线程组，点击Run 执行，使得大批量线程高并发访问资源B。

然后访问资源A：http://localhost:8401/v1/order/all/list 

资源A 被流控：

```txt
Blocked by Sentinel (flow limiting)
```



（3）链路模式

**链路流控模式指的是，当从某个接口过来的资源达到限流条件时，开启限流；它的功能有点类似于针对 来源配置项，区别在于：针对来源是针对上级微服务，而链路流控是针对上级接口，也就是说它的粒度 更细。**

"流控规则"菜单，添加流控规则，填写资源名：`/v1/order/all/list`
 以QPS为例，填写单机阈值为1
 高级选项，流控模式选择“链路”
 入口资源处填写的应为`/order/all/list`的入口资源地址，即sentinel_web_servlet_context
 流控效果以“快速失败”为例
 点击新增。
测试发送请求：http://localhost8401/order/all/list 时，如果1秒内请求次数超过1次，就会自动触发限流。
此外，通过其他微服务模块请求资源 `/order/all/list `时，如果1秒内请求次数超过1次，同样会触发限流。

流控模式为“链路模式”下的配置就此完成。

实际上，链路的控制指的就是对一条链路的访问进行控制。

资源 A —> 资源B  —>  资源C

资源 A —> 资源D  —>  资源E

此时对资源A进行限流，实际上是对资源A之后的资源B、资源C、资源D 、资源E也一同进行限流。资源A对于后续资源（B/C/D/E）来说就入口资源。

### 3、流控效果

（1）直接模式

**直接拒绝**（`RuleConstant.CONTROL_BEHAVIOR_DEFAULT`）方式是默认的流量控制方式，当QPS超过任意规则的阈值后，新的请求就会被立即拒绝，拒绝方式为抛出**FlowException**。这种方式适用于对系统处理能力确切已知的情况下，比如通过压测确定了系统的准确水位时。

官方的Demo参见 [FlowQpsDemo](https://github.com/alibaba/Sentinel/blob/master/sentinel-demo/sentinel-demo-basic/src/main/java/com/alibaba/csp/sentinel/demo/flow/FlowQpsDemo.java)。

[官方文档关于排队等待](https://github.com/alibaba/Sentinel/wiki/%E6%B5%81%E9%87%8F%E6%8E%A7%E5%88%B6#%E7%9B%B4%E6%8E%A5%E6%8B%92%E7%BB%9D) 

（2）预热

系统初始化的默认阈值为10 / 3，即为3，也就是刚开始的时候阈值只有3，当经过5s后，阈值才慢慢提高到10；

Warm Up（`RuleConstant.CONTROL_BEHAVIOR_WARM_UP`）方式，即预热/冷启动方式。当系统长期处于低水位的情况下，当流量突然增加时，直接把系统拉升到高水位可能瞬间把系统压垮。通过"冷启动"，让通过的流量缓慢增加，在一定时间内逐渐增加到阈值上限，给冷系统一个预热的时间，避免冷系统被压垮。详细文档可以参考 [流量控制 - Warm Up 文档](https://github.com/alibaba/Sentinel/wiki/%E9%99%90%E6%B5%81---%E5%86%B7%E5%90%AF%E5%8A%A8)，具体的例子可以参见 [WarmUpFlowDemo](https://github.com/alibaba/Sentinel/blob/master/sentinel-demo/sentinel-demo-basic/src/main/java/com/alibaba/csp/sentinel/demo/flow/WarmUpFlowDemo.java)。

[官方文档关于预热Warm Up](https://github.com/alibaba/Sentinel/wiki/%E6%B5%81%E9%87%8F%E6%8E%A7%E5%88%B6#warm-up) 

根据`codeFactor`（冷加载因子，默认为3）的值，即请求 QPS 从 `threshold / 3` 开始，经预热时长逐渐升至设定的 QPS 阈值 ;

源码位置：**com.alibaba.csp.sentinel.slots.block.flow.controller.WarmUpController**

应用场景：秒杀系统的开启瞬间，会有很多流量上来，很可能会把系统打挂，预热方式就是为了保护系统，可以慢慢的把流量放进来，慢慢的把阈值增长到设定值；

![预热模式](/images/springcloud/sentinel-warmup-work.png)





（3）排队等待

匀速排队（`RuleConstant.CONTROL_BEHAVIOR_RATE_LIMITER`）方式会严格控制请求通过的间隔时间，也即是让请求以均匀的速度通过，对应的是漏桶算法。 

匀速排队，让请求以均匀的速度通过，阈值类型必须设置成`QPS`，否则无效。

"流控规则"菜单，添加流控规则，填写资源名：`/v1/order/all/list`
以QPS为例，填写单机阈值为1，高级选项，流控模式选择“直接”，流控效果以“排队等待”，点击新增。

设置的含义：`/v1/order/all/list`每秒1次请求，QPS大于1后，再有请求就排队，等待超时时间为20000毫秒；

这种方式主要用于处理间隔性突发的流量，例如消息队列。想象一下这样的场景，在某一秒有大量的请求到来，而接下来的几秒则处于空闲状态，我们希望系统能够在接下来的空闲期间逐渐处理这些请求，而不是在第一秒直接拒绝多余的请求。

![排队等待](/images/springcloud/sentinel-paidui-work.png)

这种方式主要用于处理间隔性突发的流量，例如消息队列。想象一下这样的场景，在某一秒有大量的请求到来，而接下来的几秒则处于空闲状态，我们希望系统能够在接下来的空闲期间逐渐处理这些请求，而不是在第一秒直接拒绝多余的请求。

> 注意：匀速排队模式暂时不支持 QPS > 1000 的场景。

源码位置：**com.alibaba.csp.sentinel.slots.block.flow.controller.RateLimiterController**

[官方文档关于排队等待](https://github.com/alibaba/Sentinel/wiki/%E6%B5%81%E9%87%8F%E6%8E%A7%E5%88%B6#%E5%8C%80%E9%80%9F%E6%8E%92%E9%98%9F) 



### 4、按资源名称限流

在sentinel-service-8401 的业务类添加

```java
	@GetMapping("/byResource")
    @SentinelResource(value = "byResource",blockHandler = "handleException")
    public String byResource()
    {
        return "按资源名称限流测试--正常返回";
    }
    public String handleException(BlockException exception)
    {
        return "按资源名称限流测试--自定义返回，服务降级";
    }
```

添加流控规则：

![添加流控规则](/images/springcloud/sentinel-byresource-demo-01.png)注意资源名没有`/` ，上述配置表示1秒钟内查询次数大于1，就会触发降级，进入`handleException`快速失败返回。

添加成功后

![添加流控规则](/images/springcloud/sentinel-byresource-demo-02.png)

测试步骤：

启动之后，访问测试地址：http://localhost:8401/byResource

每秒访问一次，正常返回。

快速刷新，或者使用jmeter并发测试，返回了自己定义的限流处理信息：

```txt
按资源名称限流测试--自定义返回，服务降级
```

### 5、按URL限流

在sentinel-service-8401 的业务类添加

```java
    @GetMapping("/rateLimit/byUrl")
    @SentinelResource(value = "byUrl")
    public String byUrl()
    {
        return "按url限流测试OK";
    }
```

![添加流控规则](/images/springcloud/sentinel-byUrl-demo-01.png)

注意：按URL限流前面有`/` 。

测试快速刷新：http://localhost:8401/rateLimit/byUrl

```txt
Blocked by Sentinel (flow limiting)
```



### 6、自定义限流

创建自定义限流处理类：`CustomerBlockHandler`类用于自定义限流处理逻辑。

CustomerBlockHandler 类：

```java
package com.xiaocai.springcloud.alibaba.myhandler;

import com.alibaba.csp.sentinel.slots.block.BlockException;

public class CustomerBlockHandler {

    public static String handleException(BlockException exception) {
        return "自定义限流处理信息....CustomerBlockHandler";

    }
}
```

业务代码块：

```java
@GetMapping("/rateLimit/customerBlockHandler")
@SentinelResource(value = "customerBlockHandler",
        blockHandlerClass = CustomerBlockHandler.class,
        blockHandler = "handlerException2")
public String customerBlockHandler()
{
    return  "按自定义限流正常返回" ;
}
```

启动服务，调用一次：http://localhost:8401/rateLimit/customerBlockHandler



### 7、@SentinelResource

`@SentinelResource` 

@SentinelResource 用于定义资源，并提供可选的异常处理和 fallback 配置项。 @SentinelResource 注解包含以下属性：

- value：资源名称，必需项（不能为空）

- entryType：entry 类型，可选项（默认为 EntryType.OUT）

- blockHandler / blockHandlerClass: blockHandler 对应处理 BlockException 的函数名称，可选项。blockHandler 函数访问范围需要是 public，返回类型需要与原方法相匹配，参数类型需要和原方法相匹配并且最后加一个额外的参数，类型为 BlockException。blockHandler 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 blockHandlerClass 为对应的类的 Class 对象，注意对应的函数必需为 static 函数，否则无法解析。

- fallback / fallbackClass：fallback 函数名称，可选项，用于在抛出异常的时候提供 fallback 处理逻辑。fallback 函数可以针对所有类型的异常（除了 exceptionsToIgnore 里面排除掉的异常类型）进行处理。fallback 函数签名和位置要求：

  - 返回值类型必须与原函数返回值类型一致；
  - 方法参数列表需要和原函数一致，或者可以额外多一个 Throwable 类型的参数用于接收对应的异常。

    fallback 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 fallbackClass 为对应的类的 Class 对象，注意对应的函数必需为 static 函数，否则无法解析。

- defaultFallback（since 1.6.0）：默认的 fallback 函数名称，可选项，通常用于通用的 fallback 逻辑（即可以用于很多服务或方法）。默认 fallback 函数可以针对所有类型的异常（除了 exceptionsToIgnore 里面排除掉的异常类型）进行处理。若同时配置了 fallback 和 defaultFallback，则只有 fallback 会生效。defaultFallback 函数签名要求：

  - 返回值类型必须与原函数返回值类型一致；

  - 方法参数列表需要为空，或者可以额外多一个 Throwable 类型的参数用于接收对应的异常。

    defaultFallback 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 fallbackClass 为对应的类的 Class 对象，注意对应的函数必需为 static 函数，否则无法解析。

- exceptionsToIgnore（since 1.6.0）：用于指定哪些异常被排除掉，不会计入异常统计中，也不会进入 fallback 逻辑中，而是会原样抛出。

    注：1.6.0 之前的版本 fallback 函数只针对降级异常（DegradeException）进行处理，不能针对业务异常进行处理。

特别地，若 blockHandler 和 fallback 都进行了配置，则被限流降级而抛出 BlockException 时只会进入 blockHandler 处理逻辑。若未配置 blockHandler、fallback 和 defaultFallback，则被限流降级时会将 BlockException 直接抛出（若方法本身未定义 throws BlockException 则会被 JVM 包装一层 UndeclaredThrowableException）。

更多说明：https://github.com/alibaba/Sentinel/wiki/%E6%B3%A8%E8%A7%A3%E6%94%AF%E6%8C%81



Sentinel主要有三个核心API

- SphU定义资源
- Tracer定义统计
- ContextUtil定义了上下文



## 六、熔断降级规则

Sentinel 熔断降级会在调用链路中某个资源出现不稳定状态（超时/异常比例升高）时，对该资源进行限流，让请求快速失败，避免影响到其他的资源而导致级联错误。当资源被降级后，在接下来的降级时间窗口之内，对该资源的调用都自动熔断（默认行为是抛出`DegradeException`）。

Sentinel 熔断降级和 Hystrix 熔断降级相比，Sentinel 熔断降级没有半开状态。



### 1、基本降级元素

特别说明：这里的使用的1.8的截图，所以熔断策略里出现了 **"慢调用比例"** 替换了旧版的 **"RT"**。

![降级规则元素](/images/springcloud/sentinel-jiangji-demo-01.png)

资源名：唯一名称

降级策略：

- RT - 平均响应时间；
- 异常比例，秒级；
- 异常数，分钟数；

### 2、熔断降级规则

熔断降级规则（DegradeRule）包含下面几个重要的属性：

| Field              | 说明                                                         | 默认值     |
| ------------------ | ------------------------------------------------------------ | ---------- |
| resource           | 资源名，即规则的作用对象                                     |            |
| grade              | 熔断策略，支持慢调用比例/异常比例/异常数策略                 | 慢调用比例 |
| count              | 慢调用比例模式下为慢调用临界 RT（超出该值计为慢调用）；异常比例/异常数模式下为对应的阈值 |            |
| timeWindow         | 熔断时长，单位为 s                                           |            |
| minRequestAmount   | 熔断触发的最小请求数，请求数小于该值时即使异常比率超出阈值也不会熔断（1.7.0 引入） | 5          |
| statIntervalMs     | 统计时长（单位为 ms），如 60*1000 代表分钟级（1.8.0 引入）   | 1000 ms    |
| slowRatioThreshold | 慢调用比例阈值，仅慢调用比例模式有效（1.8.0 引入）           |            |

#### （1）RT - 平均响应时间

RT - 平均响应时间（DEGRADE_DRADE_RT）：RT以秒为单位，平均响应时间超出阈值（（DegradeRule 中的count，以ms 为单位）且在实际窗口内的请求大于或等于5，两个条件同时满足后触发降级。窗口期结束后关闭断路器。RT 最大4900 更大的需要配置使用需要配置：

```properties
-Dcsp.sentinel.statistic.max.rt=XXX 
```

![RT 流程](/images/springcloud/sentinel-rt-01.png)

代码：

```java
    @GetMapping("/testD")
    public String rtTest()
    {
        try { 
            TimeUnit.SECONDS.sleep(1); 
        } catch (InterruptedException e) { 
            e.printStackTrace(); 
        }
        log.info("rtTest 测试 RT");

        return "------rtTest-----";
    }
```

模拟测试，可以使用jmeter进行压力测试。新建线程组，线程数手指成10，Ramp-Up Period 设置1，循环设置永远。

![RT -测试](/images/springcloud/sentinel-rt-02.png)

后续停止jemeter ，再次访问，资源恢复访问。

1.8之后策略更换为**慢调用比例**。

#### （2）异常比例

异常比例（DEGRADE_GRADE_EXCEPTION_RATIO）：秒级。 当资源每秒请求量QPS 大于或等于5 且 异常比例（每秒异常总数占通过量的比值）超过阈值（DegradeRule 中的count）时，触发降级，资源进入降级状态；时间窗口（DegradeRule 中的 timewindow，以s为单位）结束后，关闭降级。

异常比例的阈值范围是[0.0 , 1.0] ，代表 0% - 100%。

![异常比例 流程](/images/springcloud/sentinel-expcetion-01.jpg)

测试代码：

```java
@GetMapping("/testD")
    public String testD()
    {
        log.info("testD 测试 异常比例");
        int age = 10/0;
        return "------testD-----";
    }
```

降级规则设置：

![异常比例设置](/images/springcloud/sentinel-expcetion-02.jpg)

测试说明：

按上述配置，使用浏览器访问，因为写了 10/0 ，所以每次访问都会报错。

使用jmeter 测试

![jmeter 测试](/images/springcloud/sentinel-expcetion-03.jpg)

jmeter 测试，直接高并发请求，多次调用达到配置条件后，微服务不再报错，而是提示降级。



#### （3）异常数

异常数：分钟级，（分钟统计的）异常数超过阈值时触发降级；时间窗口结束后，关闭降级。

异常数（`DEGRADE_GRADE_EXCEPTION_COUNT`）：当资源近1分钟的异常数目超过阈值之后会进行熔断降级。注意由于统计时间是分钟基本，时间窗口要大于或等于60秒。

若 `timeWindow` 小于60秒，结束熔断之后仍有可能再进入熔断状态。

![异常数 流程](/images/springcloud/sentinel-expcetion-04.jpg)

测试代码：

```java
@GetMapping("/testE")
public String testE()
{
    log.info("testE 测试异常数");
    int age = 10/0;
    return "------testE 测试异常数 ------";
} 
```

降级规则设置

![降级规则设置](/images/springcloud/sentinel-expcetion-05.jpg)

### 3、自定义降级提示

返回的异常打到了前台用户界面看不到，不友好。

错误类`com.alibaba.csp.sentinel.slots.block.BlockException`

因为需要自定义。

自定义降级提示信息：使用`@SentinelResource` 注解。

示例

```java
@GetMapping("/testHotKey")
@SentinelResource(value = "testHotKey",blockHandler = "deal_testHotKey")
public String testHotKey(@RequestParam(value = "p1",required = false) String p1,
                         @RequestParam(value = "p2",required = false) String p2) {
    //int age = 10/0;
    return "------testHotKey";
}
 
public String deal_testHotKey (String p1, String p2, BlockException exception){
    return "------deal_testHotKey ----user words ";  
}
```

此时熔断降级时使用 `deal_testHotKey` 的方法返回。

需要注意的是：`@SentinelResource` 注解的降级只负责sentinel 控制台配置的规则的降级。针对异常，`@SentinelResource` 注解不负责。



### 4、服务降级+fallback 案例

sentinel + ribbon + openFeign + fallback

启动 nacos 和 sentinel 服务。

先加入Ribbon。

### 4.1 准备两个服务提供端

sentinel-provider-service-9003

sentinel-provider-service-9004

#### 4.1.1 引入pom依赖 

```xml
<dependencies>
    <!--SpringCloud ailibaba nacos -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>

    <!-- SpringBoot整合Web组件 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <!--日常通用jar包配置-->
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

#### 4.1.2 YML配置

YML配置：

```yaml
server:
  port: 9003

spring:
  application:
    name: nacos-provider-service
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

#### 4.1.3 启动类

```java
package com.xiaocai.springcloud.alibaba;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;


@SpringBootApplication
@EnableDiscoveryClient
public class ProviderServiceApp_9003
{
    public static void main(String[] args) {
        SpringApplication.run(ProviderServiceApp_9003.class, args);
    }
}
```

#### 4.1.4 业务类

```java
package com.xiaocai.springcloud.alibaba.controller;


import com.xiaocai.springcloud.alibaba.entities.CommonResult;
import com.xiaocai.springcloud.alibaba.entities.User;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import java.util.HashMap;


@RestController
public class UserProviderController
{
    @Value("${server.port}")
    private String serverPort;

    public static HashMap<Long, User> hashMap = new HashMap<>();
    static{
        hashMap.put(1L,new User(1L,"zhangxiaocai"));
        hashMap.put(2L,new User(2L,"zhangsan"));
        hashMap.put(3L,new User(3L,"lisi"));
    }

    @GetMapping(value = "/v1/getUser/{id}")
    public CommonResult<User> getUser(@PathVariable("id") Long id){
        User user = hashMap.get(id);
        CommonResult<User> result = new CommonResult(200,"from mysql,serverPort:  "+serverPort,user);
        return result;
    }
}
```

#### 4.1.5 测试

测试访问：http://localhost:9003/v1/getUser/1

属性sentinel 出现微服务。



#### 4.1.6 第二个工程

跟上述步骤一致，改下端口号。



### 4.2 准备一个消费端

 sentinel-consumer-9009，使用Ribbon

#### 4.2.1 引入pom依赖

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

    <artifactId>sentinel-consumer-9009</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
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

#### 4.2.2 相关配置

YML配置：

```yaml
server:
  port: 9009


spring:
  application:
    name: nacos-consumer-service
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
    sentinel:
      transport:
        dashboard: localhost:8080
        port: 8719

service-url:
  nacos-user-service: http://nacos-provider-service
 
```

java 配置类：

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
    @LoadBalanced
    public RestTemplate getRestTemplate()
    {
        return new RestTemplate();
    }
}
```

使用 Ribbon 负载均衡

#### 4.2.3 启动类

```java
package com.xiaocai.springcloud.alibaba;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;


@EnableDiscoveryClient
@SpringBootApplication
public class ConsumerNacosApp_9009
{
    public static void main(String[] args) {
        SpringApplication.run(ConsumerNacosApp_9009.class, args);
    }
}

```

#### 4.2.4 业务类

controller 类：

```java
package com.xiaocai.springcloud.alibaba.controller;

import com.alibaba.csp.sentinel.annotation.SentinelResource;
import com.alibaba.csp.sentinel.slots.block.BlockException;
import com.xiaocai.springcloud.alibaba.entities.CommonResult;
import com.xiaocai.springcloud.alibaba.entities.User;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;


@RestController
@Slf4j
public class CircleBreakerController {
   
    public static final String SERVICE_URL = "http://nacos-provider-service";

    @Resource
    private RestTemplate restTemplate;

    @RequestMapping("/consumer/fallback/{id}")
    //@SentinelResource(value = "fallback") //没有配置
    //@SentinelResource(value = "fallback",fallback = "handlerFallback") //fallback只负责业务异常
    //@SentinelResource(value = "fallback",blockHandler = "blockHandler") //blockHandler只负责sentinel控制台配置违规
    @SentinelResource(value = "fallback",fallback = "handlerFallback",blockHandler = "blockHandler",
            exceptionsToIgnore = {IllegalArgumentException.class})
    public CommonResult<User> fallback(@PathVariable Long id) {
        CommonResult<User> result = restTemplate.getForObject(SERVICE_URL + "/v1/getUser/"+id, CommonResult.class,id);

        if (id == 4) {
            throw new IllegalArgumentException ("IllegalArgumentException,非法参数异常....");
        }else if (result.getData() == null) {
            throw new NullPointerException ("NullPointerException,该ID没有对应记录,空指针异常");
        }

        return result;
    }
  
    //fallback
    public CommonResult handlerFallback(@PathVariable  Long id,Throwable e) {
        User user = new User(id,"null");
        return new CommonResult<>(444,"异常备选响应handlerFallback,exception内容  "+e.getMessage(),user);
    }
  
    //blockHandler
    public CommonResult blockHandler(@PathVariable  Long id,BlockException blockException) {
        User user = new User(id,"null");
        return new CommonResult<>(445,"blockHandler-sentinel 限流,无此流水: blockException  "+blockException.getMessage(),user);
    }
}
```

#### 4.2.5 测试

测试访问：http://localhost:9009/consumer/fallback/1

刷新sentinel 出现微服务。

可以分别使用以下四个配置进行测试，访问，并查看异常和负载均衡效果。

配置一：

```
@SentinelResource(value = "fallback") //没有配置
```

重启后测试访问一：

- http://localhost:9009/consumer/fallback/1
- http://localhost:9009/consumer/fallback/4
- http://localhost:9009/consumer/fallback/5



配置二：

```java
@SentinelResource(value = "fallback",fallback = "handlerFallback") //fallback只负责业务异常
```

重启后测试访问二：

- http://localhost:9009/consumer/fallback/5



配置三：

```java
@SentinelResource(value = "fallback",blockHandler = "blockHandler") //blockHandler只负责sentinel控制台配置违规    
```

重启后，进行简单的QPS限流测试，资源名：`9/consumer/fallback` ，限流阈值1等，方便模拟即可。

测试访问三：

- http://localhost:9009/consumer/fallback/1
- http://localhost:9009/consumer/fallback/4
- http://localhost:9009/consumer/fallback/5



配置四：

```java
@SentinelResource(value = "fallback",fallback = "handlerFallback",blockHandler = "blockHandler", exceptionsToIgnore = {IllegalArgumentException.class})
```

重启后测试访问四：

- http://localhost:9009/consumer/fallback/4
- http://localhost:9009/consumer/fallback/5



已经加入Ribbon，再加入Feign。

### 4.3 Feign的加入

在消费端工程添加 Feign 的支持。

#### 4.3.1 引入POM依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

#### 4.3.2 追加Feign的支持

```yaml
#对Feign的支持
feign:
  sentinel:
    enabled: true
```

#### 4.3.3 添加Feign封装接口

`ProviderUserService` 接口：

```java
package com.xiaocai.springcloud.alibaba.service;

import com.xiaocai.springcloud.alibaba.entities.CommonResult;
import com.xiaocai.springcloud.alibaba.entities.User;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;


@FeignClient(value = "nacos-provider-service",fallback = ProviderUserFallbackService.class)
public interface ProviderUserService
{
    @GetMapping(value = "/v1/getUser/{id}")
    public CommonResult<User> getUser(@PathVariable("id") Long id);
} 
```

fallback 对应的服务降级支持类`ProviderUserFallbackService`：

```java
package com.xiaocai.springcloud.alibaba.service;


import com.xiaocai.springcloud.alibaba.entities.CommonResult;
import com.xiaocai.springcloud.alibaba.entities.User;
import org.springframework.stereotype.Component;


@Component
public class ProviderUserFallbackService implements ProviderUserService
{
    @Override
    public CommonResult<Payment> getUser(Long id)
    {
        return new CommonResult<>(44444,"服务降级返回,---PaymentFallbackService",new Payment(id,"errorSerial"));
    }
}
```

#### 4.3.4 主启动类

主启动类追加 Feign 支持的注解`@EnableFeignClients` ：

```java
 package com.xiaocai.springcloud.alibaba;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;


@EnableDiscoveryClient
@SpringBootApplication
@EnableFeignClients
public class ConsumerNacosApp_9009
{
    public static void main(String[] args) {
        SpringApplication.run(ConsumerNacosApp_9009.class, args);
    }
}
```

#### 4.3.5 追加controller 测试入口

`CircleBreakerController`  追加测试feign的调用入口：

```java
// OpenFeign
@Resource
private ProviderUserService providerUserService;

@GetMapping(value = "/consumer/openfeign/{id}")
public CommonResult<User> getUser(@PathVariable("id") Long id) {
    return providerUserService.getUser(id);
}
```



#### 4.3.6 测试验证

重启服务访问。

Feign 的测试验证：http://lcoalhost:9009/consumer/openfeign/1

Feign 的测试验证：http://lcoalhost:9009/consumer/getUser/1

测试消费端9009调用服务端9003，此时故意关闭9003微服务提供者，看消费侧自动降级，不会被耗死。



### 5、熔断降级框架比较

| 比较项       | Sentinel                                                   | Hystrix               | resilience4j                      |
| ------------ | ---------------------------------------------------------- | --------------------- | --------------------------------- |
| 隔离策略     | 信号量隔离（并发线程数限流）                               | 线程池隔离/信号量隔离 | 信号量隔离                        |
| 熔断降级策略 | 基于响应时间、异常比例、异常数                             | 基于异常比例          | 基于异常比例、响应时间            |
| 实时统计实现 | 滑动窗口（LeapArray）                                      | 滑动窗口（RxJava）    | Ring Bit Buffer                   |
| 动态规则配置 | 支持多种数据源                                             | 支持多种数据源        | 有限支持                          |
| 扩展性       | 多个扩展点                                                 | 插件的形式            | 接口形式                          |
| 基于注解支持 | 支持                                                       | 支持                  | 支持                              |
| 限流         | 基于QPS/调用关系限流                                       | 有限支持              | Rate Limiter                      |
| 流量整形     | 支持预热模式、匀速器模式、预热排队模式                     | 不支持                | 简单的Rate Limiter 模式           |
| 系统自适应   | 支持                                                       | 不支持                | 不支持                            |
| 控制台       | 提供开销即用的控制台，可以配置规则、查看秒级监控、机器发现 | 简单的监控查看        | 不提供控制台，可 对接其他监控系统 |







## 七、热点限流

热点即经常访问的数据。很多时候我们希望统计某个热点数据中访问频次最高的 Top K 数据，并对其访问进行限制。比如：

- 商品 ID 为参数，统计一段时间内最常购买的商品 ID 并进行限制
- 用户 ID 为参数，针对一段时间内频繁访问的用户 ID 进行限制

热点参数限流会统计传入参数中的热点参数，并根据配置的限流阈值与模式，对包含热点参数的资源调用进行限流。热点参数限流可以看做是一种特殊的流量控制，仅对包含热点参数的资源调用生效。

 

Sentinel 利用 LRU 策略统计最近最常访问的热点参数，结合令牌桶算法来进行参数级别的流控。热点参数限流支持集群模式。

### 1、热点参数规则

热点参数规则（`ParamFlowRule`）类似于流量控制规则（`FlowRule`）：

| 属性              | 说明                                                         | 默认值   |
| ----------------- | ------------------------------------------------------------ | -------- |
| resource          | 资源名，必填                                                 |          |
| count             | 限流阈值，必填                                               |          |
| grade             | 限流模式                                                     | QPS 模式 |
| durationInSec     | 统计窗口时间长度（单位为秒），1.6.0 版本开始支持             | 1s       |
| controlBehavior   | 流控效果（支持快速失败和匀速排队模式），1.6.0 版本开始支持   | 快速失败 |
| maxQueueingTimeMs | 最大排队等待时长（仅在匀速排队模式生效），1.6.0 版本开始支持 | 0ms      |
| paramIdx          | 热点参数的索引，必填，对应 `SphU.entry(xxx, args)` 中的参数索引位置 |          |
| paramFlowItemList | 参数例外项，可以针对指定的参数值单独设置限流阈值，不受前面 `count` 阈值的限制。**仅支持基本类型和字符串类型** |          |
| clusterMode       | 是否是集群参数流控规则                                       | `false`  |
| clusterConfig     | 集群流控相关配置                                             |          |

主要元素

![热点规则设置](/images/springcloud/sentinel-redian-demo-01.png)

### 2、参数限流

资源请求中含有某个位置的参数时进行限流。

测试代码同上：

```java
@GetMapping("/testHotKey")
@SentinelResource(value = "testHotKey",blockHandler = "deal_testHotKey")
public String testHotKey(@RequestParam(value = "p1",required = false) String p1,
                         @RequestParam(value = "p2",required = false) String p2) {
    //int age = 10/0;
    return "------testHotKey";
}
 
public String deal_testHotKey (String p1, String p2, BlockException exception){
    return "------deal_testHotKey ----user words ";  
}
```

热点限流配置：

参数索引：0

流控模式：QPS

阈值：1

是否集群：否

是否例外项：无

演示：方法testHostKey里面第一个参数只要QPS超过每秒1次，马上降级处理。

测试快速刷新访问1：http://localhost:8401/testHotKey?p1=abc   限流

测试快速刷新访问2：http://localhost:8401/testHotKey?p1=abc&p2=33  限流

测试快速刷新访问3：http://localhost:8401/testHotKey?p2=33   不限流

### 3、参数例外项

p1参数当它是某个特殊值时，它的限流值和平时不一样。



![参数例外项设置](/images/springcloud/sentinel-params-liwai-01.jpg)

规则解释：

当p1等于5的时候，阈值变为200。http://localhost:8401/testHotKey?p1=5

当p1不等于5的时候，阈值就是平常的1。http://localhost:8401/testHotKey?p1=2



## 八、系统规则

系统保护规则是从应用级别的入口流量进行控制，从单台机器的 load、CPU 使用率、平均 RT、入口 QPS 和并发线程数等几个维度监控应用指标，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性。

系统保护规则是应用整体维度的，而不是资源维度的，并且**仅对入口流量生效**。入口流量指的是进入应用的流量（`EntryType.IN`），比如 Web 服务或 Dubbo 服务端接收的请求，都属于入口流量。

系统规则支持以下的模式：

-  **Load 自适应**（仅对 Linux/Unix-like 机器生效）：系统的 load1 作为启发指标，进行自适应系统保护。当系统 load1 超过设定的启发值，且系统当前的并发线程数超过估算的系统容量时才会触发系统保护（BBR 阶段）。系统容量由系统的 `maxQps * minRt` 估算得出。设定参考值一般是 `CPU cores * 2.5`。
-  **CPU usage**（1.5.0+ 版本）：当系统 CPU 使用率超过阈值即触发系统保护（取值范围 0.0-1.0），比较灵敏。
-  **平均 RT**：当单台机器上所有入口流量的平均 RT 达到阈值即触发系统保护，单位是毫秒。
-  **并发线程数**：当单台机器上所有入口流量的并发线程数达到阈值即触发系统保护。
-  **入口 QPS**：当单台机器上所有入口流量的 QPS 达到阈值即触发系统保护。

示例：

![系统规则设置](/images/springcloud/sentinel-system-rule-demo-01.png)

**原理**

先用经典图来镇楼:

![TCP-BBR-pipe](https://user-images.githubusercontent.com/9434884/50813887-bff10300-1352-11e9-9201-437afea60a5a.png)

我们把系统处理请求的过程想象为一个水管，到来的请求是往这个水管灌水，当系统处理顺畅的时候，请求不需要排队，直接从水管中穿过，这个请求的RT是最短的；反之，当请求堆积的时候，那么处理请求的时间则会变为：排队时间 + 最短处理时间。

- 推论一:  如果我们能够保证水管里的水量，能够让水顺畅的流动，则不会增加排队的请求；也就是说，这个时候的系统负载不会进一步恶化。

我们用 T 来表示(水管内部的水量)，用RT来表示请求的处理时间，用P来表示进来的请求数，那么一个请求从进入水管道到从水管出来，这个水管会存在 `P * RT`　个请求。换一句话来说，当 `T ≈ QPS * Avg(RT)` 的时候，我们可以认为系统的处理能力和允许进入的请求个数达到了平衡，系统的负载不会进一步恶化。

接下来的问题是，水管的水位是可以达到了一个平衡点，但是这个平衡点只能保证水管的水位不再继续增高，但是还面临一个问题，就是在达到平衡点之前，这个水管里已经堆积了多少水。如果之前水管的水已经在一个量级了，那么这个时候系统允许通过的水量可能只能缓慢通过，RT会大，之前堆积在水管里的水会滞留；反之，如果之前的水管水位偏低，那么又会浪费了系统的处理能力。

- 推论二:　当保持入口的流量是水管出来的流量的最大的值的时候，可以最大利用水管的处理能力。

然而，和 TCP BBR 的不一样的地方在于，还需要用一个系统负载的值（load1）来激发这套机制启动。

> 注：这种系统自适应算法对于低 load 的请求，它的效果是一个“兜底”的角色。**对于不是应用本身造成的 load 高的情况（如其它进程导致的不稳定的情况），效果不明显。**



## 九、规则持久化

第三节已经说明了sentinel 控制台的使用。

### 1、Sentinel 配置持久化

但是在测试过程中，微服务重启，之前配置的Sentinel规则就消失了，说明之前的规则配置都是临时的，但是在生产环境需要将配置规则进行持久化，不可能每次重启微服务都要去配置一次。

如何持久化？

将限流配置规则持久化进Nacos保存，只要刷新8401某个rest地址，sentinel控制台的流控规则就能看到，只要Nacos里面的配置不删除，针对8401上Sentinel上的流控规则持续有效。

### 2、案例

修改sentinel-service-8401

### 2.1 引入pom 依赖

```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```

### 2.2 YML配置添加数据源

追加数据源的配置

```
spring:
   cloud:
    sentinel:
    datasource:
     ds1:
      nacos:
        server-addr:localhost:8848
        dataid:${spring.application.name}
        groupid:DEFAULT_GROUP
        data-type:json
            rule-type:flow
```

完整的YML配置：

```yaml
server:
  port: 8401

spring:
  application:
    name: sentinel-service
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
    sentinel:
      transport:
        dashboard: localhost:8080
        port: 8719  #默认8719，假如被占用了会自动从8719开始依次+1扫描。直至找到未被占用的端口
        datasource:
          ds1:
            nacos:
              server-addr: localhost:8848
              dataId: sentinel-service
              groupId: DEFAULT_GROUP
              data-type: json
              rule-type: flow  

management:
  endpoints:
    web:
      exposure:
        include: '*'

feign:
  sentinel:
    enabled: true # 激活Sentinel对Feign的支持

```

###  2.3 Nacos 业务规则配置

添加配置：



内容解析：

```json
[
    {
         "resource": "/retaLimit/byUrl",
         "limitApp": "default",
         "grade":   1,
         "count":   1,
         "strategy": 0,
         "controlBehavior": 0,
         "clusterMode": false    
    }
]
```

### 2.4 测试

启动8401，访问地址：http://localhost:8401/rateLimit/byUrl

刷新sentinel发现业务规则出现。

快速刷新访问地址：http://localhost:8401/rateLimit/byUrl

```txt

```

停止端口为8401的服务，再看sentinel 控制台，规则没有了。

重新启动端口为8401的服务，再看sentinel控制台，规则可能不会立即出现，再次刷新访问地址：http://localhost:8401/rateLimit/byUrl

重新配置出现了，说明持久化验证通过。



## 十、新版Sentinel

**Sentinel在1.8.0版本对熔断降级做了大的调整，可以定义任意时长的熔断时间，引入了半开启恢复支持。**

### 1、熔断状态

 

熔断有三种状态，分别为OPEN、HALF_OPEN、CLOSED。

| 状态      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| OPEN      | 表示熔断开启，拒绝所有请求                                   |
| HALF_OPEN | 探测恢复状态，如果接下来的一个请求顺利通过则结束熔断，否则继续熔断 |
| CLOSED    | 表示熔断关闭，请求顺利通过                                   |

### 2、熔断策略

 

熔断降级支持慢调用比例、异常比例、异常数三种熔断策略。先明确下面两个概念：**慢调用**：指耗时大于阈值RT的请求称为慢调用，阈值RT由用户设置

**最小请求数**：允许通过的最小请求数量，在最小请求数量内不发生熔断，由用户设置

#### （1）慢调用比例

| **属性**   | **说明**                                     |
| ---------- | -------------------------------------------- |
| 最大RT     | 需要设置的阈值，超过该值则为慢应用           |
| 比例阈值   | 慢调用占所有的调用的比率，范围：[0~1]        |
| 熔断时长   | 在这段时间内发生熔断、拒绝所有请求           |
| 最小请求数 | 即允许通过的最小请求数，在该数量内不发生熔断 |

**执行逻辑**

熔断（OPEN）：请求数大于最小请求数并且慢调用的比率大于比例阈值则发生熔断，熔断时长为用户自定义设置。

探测（HALFOPEN）：当熔断过了定义的熔断时长，状态由熔断（OPEN）变为探测（HALFOPEN）。

- 如果接下来的一个请求小于最大RT，说明慢调用已经恢复，结束熔断，状态由探测（HALF_OPEN）变更为关闭（CLOSED）
- 如果接下来的一个请求大于最大RT，说明慢调用未恢复，继续熔断，熔断时长保持一致

#### （2）异常比例

 

通过计算异常比例与设置阈值对比的一种策略。

| **属性**     | **说明**                                          |
| ------------ | ------------------------------------------------- |
| 异常比例阈值 | 异常比例=发生异常的请求数÷请求总数取值范围：[0~1] |
| 熔断时长     | 在这段时间内发生熔断、拒绝所有请求                |
| 最小请求数   | 即允许通过的最小请求数，在该数量内不发生熔断      |

**执行逻辑**

熔断（OPEN）：当请求数大于最小请求并且异常比例大于设置的阈值时触发熔断，熔断时长由用户设置。

探测（HALFOPEN）：当超过熔断时长时，由熔断（OPEN）转为探测（HALFOPEN）

- 如果接下来的一个请求未发生错误，说明应用恢复，结束熔断，状态由探测（HALF_OPEN）变更为关闭（CLOSED）
- 如果接下来的一个请求继续发生错误，说明应用未恢复，继续熔断，熔断时长保持一致



#### （3）异常数 

通过计算发生异常的请求数与设置阈值对比的一种策略。

| **属性**   | **说明**                                     |
| ---------- | -------------------------------------------- |
| 异常数     | 请求发生异常的数量                           |
| 熔断时长   | 在这段时间内发生熔断、拒绝所有请求           |
| 最小请求数 | 即允许通过的最小请求数，在该数量内不发生熔断 |

**执行逻辑**

熔断（OPEN）：当请求数大于最小请求并且异常数量大于设置的阈值时触发熔断，熔断时长由用户设置。探测（HALFOPEN）：当超过熔断时长时，由熔断（OPEN）转为探测（HALFOPEN）

- 如果接下来的一个请求未发生错误，说明应用恢复，结束熔断，状态由探测（HALF_OPEN）变更为关闭（CLOSED）
- 如果接下来的一个请求继续发生错误，说明应用未恢复，继续熔断，熔断时长保持一致

### 4、规则参数说明

熔断降级DegradeRule中的属性进行说明

| **属性**           | **说明**                                                     |
| ------------------ | ------------------------------------------------------------ |
| resource           | 资源名称                                                     |
| grade              | 降级策略 0:慢调用比例 1:异常比例 2:异常数量                  |
| count              | 用户设置的阈值，根据不同的策略分别表示最大RT、异常比例阈值、异常数阈值 |
| timeWindow         | 熔断时长                                                     |
| minRequestAmount   | 最小请求数，默认为5                                          |
| slowRatioThreshold | 慢调用比率阈值，默认为1.0                                    |
| statIntervalMs     | 熔断时长，默认为1秒                                          |

 1.慢调用策略

```
[
    {
        "count": 3000,
        "grade": 0,
        "limitApp": "default",
		"minRequestAmount": 100,
        "resource": "/v1/test1",
        "slowRatioThreshold": 0.5,
        "statIntervalMs": 1000,
        "timeWindow": 5
    }
]
```

2.异常比例

```
{
    "count": 0.3,
    "grade": 1,
    "limitApp": "default",
    "minRequestAmount": 200,
    "resource": "degrade02",
    "slowRatioThreshold": 1,
    "statIntervalMs": 1000,
    "timeWindow": 5
}
```

3.异常数

```
{
    "count": 1000,
    "grade": 2,
    "limitApp": "default",
	"minRequestAmount": 300,
	"resource": "degrade03",
    "slowRatioThreshold": 1,
    "statIntervalMs": 1000,
    "timeWindow": 5
}
```

