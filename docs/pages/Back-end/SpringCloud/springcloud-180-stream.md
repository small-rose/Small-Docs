---
title: SpringCloud Stream
tags: SpringCloud
categories: SpringCloud
summary: 记录SpringCloud Stream相关学习和整理。
keywords: >-
  Spring Cloud,Spring Cloud 微服务,Spring Cloud Stream ,Spring Cloud Stream原理,bus
  教程,Spring Cloud Stream 使用
author: Small-Rose / 张小菜
abbrlink: 7f3b07b1
date: 2020-09-24 10:00:00
---

## SpringCloud Stream 消息驱动

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

**Spring Cloud Stream**  是一个构建消息驱动微服务应用的框架。它基于  **Spring Boot**  构建独立的、生产级的  **Spring**  应用，并使用  **Spring Integration**  为消息代理提供链接。 

通俗理解是屏蔽底层消息中间件的插件，降低切换版本，统一消息的编程模型。

Spring Cloud Stream 官网：https://spring.io/projects/spring-cloud-stream#overview

官放文档：https://cloud.spring.io/spring-cloud-static/spring-cloud-stream/3.0.1.RELEASE/reference/html/

**Spring Cloud Stream中文指导手册**：https://m.wang1314.com/doc/webapp/topic/20971999.html



## 二、Stream 

### 1、设计思想

标准MQ模型

![标准MQ模型](/images/springcloud/stream-mq-general.jpg)

>MQ（Message Queue）消息队列，是基础数据结构中“先进先出”的一种数据结构。一般用来解决应用解耦，异步消息，流量削锋等问题，实现高性能，高可用，可伸缩和最终一致性架构。 

**MQ 作用：**

>**解耦**：一个业务需要多个模块共同实现，或者一条消息有多个系统需要对应处理，只需要主业务完成以后，发送一条MQ，其余模块消费MQ消息，即可实现业务，降低模块之间的耦合。
>
>**异步**：主业务执行结束后从属业务通过MQ，异步执行，减低业务的响应时间，提高用户体验。
>
>**削峰**：高并发情况下，业务异步处理，提供高峰期业务处理能力，避免系统瘫痪。

**MQ的缺点**  

> 1、系统可用性降低。依赖服务也多，服务越容易挂掉。需要考虑MQ瘫痪的情况
>
> 2、系统复杂性提高。需要考虑消息丢失、消息重复消费、消息传递的顺序性
>
> 3、业务一致性。主业务和从属业务一致性的处理

MQ 是生产者和消费者之间靠消息媒介传递信息内容的中间件。

MQ 的消息必须走特定的消息通道MessageChannel。

MQ 里的消息通道MessageChannel的子接口SubscribableChannel，由MessageHandler消息处理器订阅，完成消息的收发处理。

常见的MQ 中间件：

- **RabbitMQ**   http://rabbitmq.mr-ping.com/ 
- **Kafka** 
- **RocketMQ** 
- **ActiveMQ** 



为什么用Spring  Cloud Stream 

如果同时使用了 RabbitMQ 和 Kafka，由于两个消息中间件的架构不同，像Rabbit 有exchange，kafka有Topic和Partitions分区，这些中间件的差异会导致，项目中如果要进行消息列队迁移是很麻烦了，因为和系统耦合紧密这时Spring  Cloud Stream 的出现就提供了一种解耦合的方式。

![Stream 设计目的](/images/springcloud/stream-destination.png)

### 2、Stream 原理

在没有绑定器的概念的时，springboot 应用要直接与消息中间件进行信息交互时，由于各个消息中间件构建的初衷不同，在实现细节上会有较大差异，使用时会有不便。

**Spring  Cloud Stream** **通过定义绑定器Binder作为中间层，完美地实现了应用程序与消息中间件细节之间的隔离。**通过向应用程序暴露统一的channel 通道，使得应用程序不需要再考虑各种不同消息中间件的实现。

![Stream 模型](/images/springcloud/stream-work.jpg)

Spring  Cloud Stream 对消息中间件的进一步封装，做到代码层面对中间件的无感知，甚至于动态切换中间件（RqbbitMQ 切换为 Kafka），使微服务开发高度解耦，每个服务可以更好的关注自己的业务实现。



Stream中的消息通信方式遵循了**发布-订阅模式**。使用Topic 主题进行广播，类似RabbitMQ里的exchange，Kafka中的Topic。

![Stream 模型2](/images/springcloud/stream-work-02.png)

![Stream 流程示意图](/images/springcloud/stream-work-04.png)

找了一张实际使用时的流程示意图：

![Stream 流程示意图](/images/springcloud/stream-work-03.jpg)

### 3、Stream 组件

从图可以看到Stream 几个重要组件：

- Binder 绑定器组件，用于连接中间件，屏蔽差异；

- Channel 消息通道，是队列Queue的一种抽象，在消息通讯系统中就是实现存储和转发的媒介，通过对Channel对队列进行配置。

- Source 消息生产口，其实就是消息输入端Output，stream组件自身；

- Sink 消息消费口，其实就是消息输出端Input，stream组件自身；

  

### 4、编码API和常用注解

| 组成              | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| Middleware        | 中间件，目前只支持RabbitMQ和Kafka                            |
| Binder            | Binder是应用于消息中间件之间的封装，目前实行了Kafka和RabbitMQ的Binder，通过Binder可以很方便的连接中间件，可以动态的改变消息类型（对应于Kafka的topic，RabbitMQ的exchange），这些都可以通过配置文件来实现 |
| `@Input`          | 注解标识输入通道，通过该输入通道接收到的消息进入应用程序     |
| `@Output`         | 注解标识输出通道，发布的消息将通过该通道离开应用程序         |
| `@StreamListener` | 监听队列，用于消费者的队列的消息接收                         |
| `@EnableBinding`  | 指信道channel和exchange绑定在一起                            |

### 5、Stream 应用场景

1、异步处理

比如用户在电商网站下单，下单完成后会给用户推送短信或邮件，发短信和邮件的过程就可以异步完成。因为下单付款是核心业务，发邮件和短信并不属于核心功能，并且可能耗时较长，所以针对这种业务场景可以选择先放到消息队列中，有其他服务来异步处理。

2、应用解耦：

假设公司有几个不同的系统，各系统在某些业务有联动关系，比如 A 系统完成了某些操作，需要触发 B 系统及 C 系统。如果 A  系统完成操作，主动调用 B 系统的接口或 C 系统的接口，可以完成功能，但是各个系统之间就产生了耦合。用消息中间件就可以完成解耦，当 A  系统完成操作将数据放进消息队列，B 和 C 系统去订阅消息就可以了。这样各系统只要约定好消息的格式就好了。

3、流量削峰

比如秒杀活动，一下子进来好多请求，有的服务可能承受不住瞬时高并发而崩溃，所以针对这种瞬时高并发的场景，在中间加一层消息队列，把请求先入队列，然后再把队列中的请求平滑的推送给服务，或者让服务去队列拉取。

4、日志处理

kafka 最开始就是专门为了处理日志产生的。

当碰到上面的几种情况的时候，就要考虑用消息队列了。如果你碰巧使用的是 RabbitMQ 或者 kafka ，而且同样也是在使用 Spring Cloud ，那可以考虑下用 Spring Cloud Stream。



## 三、使用案例

### 1、RabbitMQ 安装

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

### 2、三个微服务

作为生产者发送消息服务：stream-rabbitmq-provider-8001

作为消费者接收消息服务：stream-rabbitmq-consumer-8002

作为消费者接收消息服务：stream-rabbitmq-provider-8003

#### 2.1 消息生产/发送端

##### （1）引入pom依赖

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

    <artifactId>stream-rabbitmq-provider8001</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-eureka-server -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

     
        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-devtools -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-test -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>
```

##### （2）YML配置

```yaml
server:
  port: 8001

spring:
  application:
    name: stream-provider-8001
  cloud:
    stream:
      binders: 		# 在此处配置要绑定的rabbitmq的服务信息；
        defaultRabbit: 		# 表示定义的名称，用于于binding整合
          type: rabbit 		# 消息组件类型
          environment: 		# 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                username: guest
                password: guest
      bindings: 		# 服务的整合处理
        output: 		# 这个名字是一个通道的名称
          destination: studyExchange 	# 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
          binder: defaultRabbit  # 设置要绑定的消息服务的具体设置

eureka:
  client: # 客户端进行Eureka注册的配置
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
    instance-id: send-8001.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址
 
```

##### （3）主启动类

```java
package com.xiaocai.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@EnableEurekaClient
public class StreamMQApp_8001 {
    public static void main(String[] args) {
        SpringApplication.run(StreamMQApp_8001.class, args);
    }
}
```

##### （4）发送消息功能

消息发送接口：

```java
package com.xiaocai.springcloud.service;

public interface IMessageProvider
{
    public String send();
}
```

消息发送实现类：

```java
package com.xiaocai.springcloud.service.impl;

import com.xiaocai.springcloud.service.IMessageProvider;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.integration.support.MessageBuilderFactory;
import org.springframework.messaging.MessageChannel;
import org.springframework.integration.support.MessageBuilder;
import javax.annotation.Resource;
import org.springframework.cloud.stream.messaging.Source;

import javax.annotation.Resource;
import java.util.UUID;


@EnableBinding(Source.class) //定义消息的推送管道
public class MessageProviderImpl implements IMessageProvider
{
    @Resource
    private MessageChannel output; // 消息发送管道

    @Override
    public String send()
    {
        String serial = UUID.randomUUID().toString();
        output.send(MessageBuilder.withPayload(serial).build());
        System.out.println(" 生成 UUID 序列号: "+serial);
        return "success";
    }
}
```

controller 类

```java
package com.xiaocai.springcloud.controller;

import com.xiaocai.springcloud.service.IMessageProvider;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;


@RestController
public class SendMessageController
{
    @Resource
    private IMessageProvider messageProvider;

    @GetMapping(value = "/sendMessage")
    public String sendMessage()
    {
        return messageProvider.send();
    }

}
```

##### （5）测试

启动Eureka 服务注册中心，

启动RabbitMQ 服务，`rabbitmq-plugins enable rabbitmq_management` 访问地址：http://localhost:15672/

启动`stream-rabbitmq-provider-8001`，访问：http://localhost:8001/sendMessage





#### 2.2  消息消费/接收端

##### （1）引入pom依赖

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

    <artifactId>stream-rabbitmq-consumer-8002</artifactId>

    <dependencies>
 
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-eureka-server -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
 
        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>


        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-devtools -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-test -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

##### （2）YML配置

```yaml
server:
  port: 8002

spring:
  application:
    name: stream-consumer-8002
  cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitmq的服务信息；
        defaultRabbit: # 表示定义的名称，用于于binding整合
          type: rabbit # 消息组件类型
          environment: # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
        input: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
          binder: defaultRabbit  # 设置要绑定的消息服务的具体设置

eureka:
  client: # 客户端进行Eureka注册的配置
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
    instance-id: receive-8002.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址
 
```

注意端口是8002。

##### （3）主启动类

```java
package com.xiaocai.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@EnableEurekaClient
public class StreamMQApp_8002 {
    public static void main(String[] args) {
        SpringApplication.run(StreamMQApp_8002.class, args);
    }
}
```



##### （4）接收消息

```java
package com.xiaocai.springcloud.controller;

import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.messaging.Message;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.messaging.Sink;
import org.springframework.stereotype.Component;

@Component
@EnableBinding(Sink.class) //定义消息的接收管道
public class ReceiveMessageListenerController {
    
    @Value("${server.port}")
    private String serverPort;

    @StreamListener(Sink.INPUT)
    public void input(Message<String> message) {
        System.out.println("第一个消费者，接受消息："+message.getPayload()+"\t port:"+serverPort);
    }

}
```

##### （5）测试接收消息

启动8001，启动8002，

访问测试发生消息：http://localhost:8001/sendMessage



此时如果一切顺利，8002的控制台会打印相关消费的消息。

#### 2.3  消息消费/接收端

此时建第二个消费端。

##### （1）引入pom依赖

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

    <artifactId>stream-rabbitmq-consumer-8002</artifactId>

    <dependencies>
 
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-eureka-server -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
 
        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>


        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-devtools -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-test -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

##### （2）YML配置

```yaml
server:
  port: 8003

spring:
  application:
    name: stream-consumer-8003
  cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitmq的服务信息；
        defaultRabbit: # 表示定义的名称，用于于binding整合
          type: rabbit # 消息组件类型
          environment: # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
        input: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
          binder: defaultRabbit  # 设置要绑定的消息服务的具体设置
          

eureka:
  client: # 客户端进行Eureka注册的配置
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
    instance-id: receive-8003.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址
 
```

注意端口是8002。

##### （3）主启动类

```java
package com.xiaocai.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@EnableEurekaClient
public class StreamMQApp_8003 {
    public static void main(String[] args) {
        SpringApplication.run(StreamMQApp_8003.class, args);
    }
}
```



##### （4）接收消息

```java
package com.xiaocai.springcloud.controller;

import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.messaging.Message;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.messaging.Sink;
import org.springframework.stereotype.Component;

@Component
@EnableBinding(Sink.class) //定义消息的接收管道
public class ReceiveMessageListenerController {
    
    @Value("${server.port}")
    private String serverPort;

    @StreamListener(Sink.INPUT)
    public void input(Message<String> message) {
        System.out.println("第二个消费者，接受消息："+message.getPayload()+"\t port:"+serverPort);
    }

}
```

##### （5）测试接收消息

启动RabbitMQ，启动Eureka 7001注册中心，启动8001，启动8002，启动8003，

访问测试发送消息：http://localhost:8001/sendMessage （访问一次发送一次消息）

应该会发现8002和8003的控制台都收到了相同的消息，此时出现重复消费问题。



#### 2.4 分组消费

微服务应用放置于同一个消息group中，就能够保证消息只会被其中一个应用消费一次。不同的组是可以消费相同的消息，同一个组内会发生竞争关系，只有其中一个应用可以消费，即不允许多个应用共同消费一个消息。

此时将8002和8003 都添加分组，并且要让两个消费端在一个相同的组内，实现**分组消费**。

同时修改8002和8003 的YML配置，添加 `group` 分组设置。

此为8002的部分配置：

```yaml
·spring:
  application:
    name: stream-consumer-8002
  cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitmq的服务信息；
        defaultRabbit: # 表示定义的名称，用于于binding整合
          type: rabbit # 消息组件类型
          environment: # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
        input: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
          binder: defaultRabbit  # 设置要绑定的消息服务的具体设置
          group: xiaocai_01 # 将8002分组到xiaocai_01
```

8003的配置修改同理，group的组名也是`xiaocai_01`

注意：此时如果 group 的组名不一致，因为不同的组是可以消费相同的消息，组名不一致时是允许重复消费。

结论：如果要防止微服务重复消费，必须将多个消费端放在同一个group组内，组名必须完全一致。



#### 2.5 消息持久化

消息持久化也是通过group 来实现。

停止8002和8003两个消费端，将其中一个的group 属性删除，另一个属性保留，比如：8002的group 属性删除，8003的group 属性保留；

访问发送消息地址：http://localhost:8001/sendMessage （刷新几次）

只启动8802，无分组属性配置，后台不会打印出消费的消息；

启动8803，有分组属性配置，后台打出来了MQ上的消息；在8003服务没有启动的时候发送了消息，8003服务启动之后依旧可以消费之前发送的消息。



### 四、其他

后续学习再补充。