---
title: SpringCloud Ribbon
tags: SpringCloud
categories: SpringCloud
summary: 本文记录SpringCloud Ribbon入门相关学习和整理。
keywords: >-
  Spring Cloud,Spring Cloud 微服务,Ribbon,Ribbon 原理,Ribbon工作原理,Ribbon 配置,Ribbon
  使用,Ribbon 负载均衡,Ribbon 负载均衡算法，Ribbon 自定义算法,Ribbon 使用方法
author: Small-Rose / 张小菜
abbrlink: 7ecbba09
date: 2020-09-15 13:00:00
---

## SpringCloud Ribbon 客户端负载均衡与服务调用


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

Spring Cloud Ribbon 是基于Netflix Ribbon 实现的一套客户端负载均衡工具。 

**Project Status: On Maintenance**

https://github.com/Netflix/ribbon

可能的替换方案：Spring Cloud Starter Loadbalancer

## 二、Ribbon

### 1、基本认识

Ribbon 是Netflix 发布的开源项目，主要是提供**客户端负载均衡算法和服务调用**。

负载均衡 Load Balance ：

将请求平摊分配到多个服务单元，从而达到系统服务的高可用。

常见负载均衡软件：Nginx、LVS、硬件F5等

Nginx 是服务器端负载均衡，客户端所有请求都会交给Nginx服务器，然后由Ninx 服务器根据分配算法实现请求分配转发。

**工作过程：**

Ribbon 是客户端负载均衡，**在调用微服务接口时，会在服务注册中心获取注册的服务列表后缓存到客户端的JVM，从而实现在客户端实现RPC进行远程服务调用技术**。



### 2、负载均衡算法

Ribbon核心组件IRule相关接口及均在均衡算法：

![Ribbon 接口](/images/springcloud/ribbon-work.jpg)



`IRule`接口会根据特定算法从服务列表中选取一个要访问的服务。

Ribbon自带的负载均衡算法：

（1）**轮询算法**。默认的负载均衡算法，源码`com.netflix.loadbalancer.RoundRobinRule`。

（2）**随机算法**。源码`com.netflix.loadbalancer.RandomRule`。

（3）**重试算法**。先按照`RoundRobinRule`的策略轮询获取服务，如果获取服务失败则在指定时间内会进行重试。源码`com.netflix.loadbalancer.RetryRule`。

（4）**响应权重算法**。对`RoundRobinRule`的扩展，响应速度越快的实例选择权重越大，越容易被选择。源码`com.netflix.loadbalancer.WeightedResponseTimeRule `。

（5）**最可用算法**。Ribbon会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务。源码`com.netflix.loadbalancer.BestAvailableRule `。

（6）**可用过滤算法**。先过滤掉故障实例，再选择并发较小的实例。源码`com.netflix.loadbalancer.AvailabilityFilteringRule  `。

（7）**区域可用算法**。默认规则，复合判断server所在区域的性能和server的可用性选择服务器。源码`com.netflix.loadbalancer.ZoneAvoidanceRule  `。



### 3、负载均衡算法原理

轮询算法原理：

**restful接口第几次请求数 % 服务器集群总数量 = 实际调用的服务器位置下标。每次服务重启之后restful接口计数从1开始。**

轮询算法核心代码：

```java
public class RoundRobinRule extends AbstractLoadBalancerRule {

    // 原子整型类
    private AtomicInteger nextServerCyclicCounter;
    private static final boolean AVAILABLE_ONLY_SERVERS = true;
    private static final boolean ALL_SERVERS = false;

    private static Logger log = LoggerFactory.getLogger(RoundRobinRule.class);

    public RoundRobinRule() {
        nextServerCyclicCounter = new AtomicInteger(0);
    }

    public RoundRobinRule(ILoadBalancer lb) {
        this();
        setLoadBalancer(lb);
    }

    public Server choose(ILoadBalancer lb, Object key) {
        if (lb == null) {
            log.warn("no load balancer");
            return null;
        }

        Server server = null;
        int count = 0;
        while (server == null && count++ < 10) {
            List<Server> reachableServers = lb.getReachableServers();
            List<Server> allServers = lb.getAllServers();
            int upCount = reachableServers.size();
            int serverCount = allServers.size();

            if ((upCount == 0) || (serverCount == 0)) {
                log.warn("No up servers available from load balancer: " + lb);
                return null;
            }

            int nextServerIndex = incrementAndGetModulo(serverCount);
            server = allServers.get(nextServerIndex);

            if (server == null) {
                /* Transient. */
                Thread.yield();
                continue;
            }

            if (server.isAlive() && (server.isReadyToServe())) {
                return (server);
            }

            // Next.
            server = null;
        }

        if (count >= 10) {
            log.warn("No available alive servers after 10 tries from load balancer: "
                    + lb);
        }
        return server;
    }

    /**
     * Inspired by the implementation of {@link AtomicInteger#incrementAndGet()}.
     *
     * @param modulo The modulo to bound the value of the counter.
     * @return The next value.
     */
    private int incrementAndGetModulo(int modulo) {
        for (;;) {
            int current = nextServerCyclicCounter.get();
            int next = (current + 1) % modulo;
            if (nextServerCyclicCounter.compareAndSet(current, next))
                return next;
        }
    }

    @Override
    public Server choose(Object key) {
        return choose(getLoadBalancer(), key);
    }
}
```

轮询负载均衡算法执行步骤：

（1）设置server 为null，获取所有可达的服务数量`upCount`，获取全部的服务列表`serverCount`。

（2）接着将当前调用次数取模`serverCount`得到实际调用的服务器下标，再利用比较并交换（自旋锁）的方式获取最终下一个调用服务的下标，

（3）在全部的服务列表中根据下标获取相应Server，`list.get(index)`。

（4）验证服务是否为null，验证服务是否存活，如果存活就返回对应的Server，如果不可用就重复上述步骤。

（5）如果重复10次依旧无法定位可达服务，则打印没有可用服务的警告。



## 三、Ribbon使用

**用在客户端**

作为客户端的组件必定是用在客户端/调用者工程中使用。

默认情况下，Eureka 客户端已经封装了Ribbon 组件。官方说明https://spring.io/projects/spring-cloud-netflix

如果在没有封装Ribbon 的环境中使用，可以试试下面的，如果版本不行，就自己找一下maven的依赖引入即可。

### 1、POM中引入依赖

```xml
	<dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-ribbon</artifactId>
   </dependency>
```



### 2、Config 配置。

千万注册是调用者/消费端工程。

```java
package com.xiaocai.springcloud.config;

import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class ApplicationContextConfig {

    @LoadBalanced
    @Bean
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```

### 3、RestTemplate调用

Ribbon和Eureka或者Consul整合后消费端可以直接调用服务而不用再关心地址和端口号。

消费端的controller示例：

```java
package com.xiaocai.springcloud.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;

@RestController
@Slf4j
public class UserConsulController {

    public static final String HTTP_MS_URL = "http://MS-CLOUD-USER-SERVICE";

    @Resource 
    private RestTemplate restTemplate;

    @GetMapping("/consumer/user/consul")
    public String user(){
      String result = restTemplate.getForObject(HTTP_MS_URL+"/v1/user/consul",String.class);
      return result;
    }

}
```

服务端user_01的controller示例：

```java
package com.xiaocai.springcloud.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;

@RestController
public class UserConsulController {
    
    @GetMapping("/v1/user/consul")
    public String user(){
       
      return "我是服务端user_01， 我的端口号是 8001";
    }

}
```

服务端user_02的controller示例：

```java
package com.xiaocai.springcloud.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;

@RestController
public class UserConsulController {

    @GetMapping("/v1/user/consul")
    public String user(){
       
      return "我是服务端user_02， 我的端口号是 8002";
    }

}
```

### 4、测试

访问消费端：http://localhost:6001/consumer/user/consul

默认情况下是轮询，反复访问地址，在浏览器上会交替出现下

```txt
我是服务端user_01， 我的端口号是 8001
```

和

```txt
我是服务端user_02， 我的端口号是 8002
```



## 四、自带负载均衡规则替换



### 1、添加自定义规则类

需要注意的细节是，这个自定义的配置类一定要和springboot 启动类所在位置隔离，不能在springboot启动类的同一层级或子级目录。否则配置会被所有的Ribbon客户端共享，达不到特殊化定制目的。

```java
package com.xiaocai.myrule;

import com.netflix.loadbalancer.IRule;
import com.netflix.loadbalancer.RandomRule;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MySelfRule {

    @Bean
    public IRule myRule(){
        return new RandomRule();//定义为随机
        //如果想自定义算法，可以在此处new  MyRule().
    }
}
```

### 2、在启动类声明

使用 `@RibbonClient ` 注解在主启动类中，声明自定义规则

```java
@RibbonClient(name = "MS-CLOUD-USER-SERVICE",configuration = MySelfRule.class)
```

示例：

```java
package com.xiaocai.springcloud;

import com.xiaocai.myrule.MySelfRule;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.netflix.ribbon.RibbonClient;

@EnableDiscoveryClient //此处注册consul
@SpringBootApplication
@RibbonClient(name = "MS-CLOUD-USER-SERVICE",configuration = MySelfRule.class)
public class UserConsumerMain6001 {
    public static void main(String[] args) {
        SpringApplication.run(UserConsumerMain6001.class,args);
    }

}
```

`@EnableDiscoveryClient`表示此处注册`consul`服务。如果使用Eureka 使用 `@EnableEurekaClient`注解。

### 3、测试

启动测试，检查设置的随机算法`RandomRule`是否生效。



## 五、自定义负载均衡

需要注意的细节同上，这个自定义的类/接口一定要和springboot 启动类所在位置隔离，不能在springboot启动类的同一层级或子级目录。

### 1、取消默认

去掉 `@LoadBalanced `注解，取消默认的负载均衡方式。

### 2、添加负载均衡接口

```java
package com.xiaocai.springcloud.lb;

import org.springframework.cloud.client.ServiceInstance;

import java.util.List;

public interface LoadBalancer {
     //收集服务器总共有多少台能够提供服务的机器，并放到list里面
    ServiceInstance instances(List<ServiceInstance> serviceInstances);

}
```

### 3、添加负载均衡算法实现

重新写轮询算法。不能遗漏`@Component` 注解。

```java
package com.xiaocai.springcloud.lb;

import org.springframework.cloud.client.ServiceInstance;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.concurrent.atomic.AtomicInteger;
//自定义写的轮询
@Component
public class UserLoadBalance implements LoadBalancer {

    private AtomicInteger atomicInteger = new AtomicInteger(0);

    //坐标
    private final int getAndIncrement(){
        int current;
        int next;
        do {
            current = this.atomicInteger.get();
            next = current >= Integer.MAX_VALUE ? 0 : current + 1;
        }while (!this.atomicInteger.compareAndSet(current,next));  //第一个参数是期望值，第二个参数是修改值
        System.out.println("*******第几次访问，次数next: "+next);
        return next;
    }

    @Override
    public ServiceInstance instances(List<ServiceInstance> serviceInstances) {  //得到机器的列表
       int index = getAndIncrement() % serviceInstances.size(); //得到服务器的下标位置
        return serviceInstances.get(index);
    }
}
```



## 六、其他

后续遇到再补充。

