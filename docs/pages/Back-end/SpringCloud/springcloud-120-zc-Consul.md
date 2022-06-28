---
title: SpringCloud Consul
tags: SpringCloud
categories: SpringCloud
summary: 本文记录服务注册Consul入门相关学习和整理。
keywords: >-
  Spring Cloud,Spring Consul 微服务,Consul,Consul 原理,Consul 工作原理,Consul 配置,Consul
  服务注册,Consul 服务注册与发现,Consul 集群，Consul 集群配置,Consul 使用
author: Small-Rose / 张小菜
abbrlink: 3010739f
date: 2020-09-13 16:00:00
---

## SpringCloud Consul 服务注册与发现


相关链接：

- [SpringCloud 微服务学习](bad02d94.html)
- [RESTful 接口设计规范](eeda118.html)
- [SpringCloud Eureka 服务注册与发现](648d6b26.html)
- [SpringCloud Consul 服务注册与发现](3010739f.html)
- [SpringCloud Ribbon 客户端负载均衡与服务调用](7ecbba09.html)
- [SpringCloud OpenFeign 服务接口调用](b27245de.html)
- [SpringCloud Hystrix 熔断器](464bb413.html)
- [SpringCloud Gateway 服务网关](bc16d093html)
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

### 1、为什么了解Consul

Eureka 停更了。https://github.com/Netflix/eureka/wiki



### 2、什么是Consul？

Consul 是由HashCorp公司用Go语言开发的一套开源的分布式服务发现和配置管理系统。

Consul需要一个数据平面，并支持代理和本机集成模型。

Consul提供了一个简单的内置代理，可以开箱即用。

官网：https://www.consul.io

官方文档：https://www.consul.io/docs/intro

下载：https://www.consul.io/downloads.html

spring cloud 中使用：https://www.springcloud.cc/spring-cloud-consul.html



### 3、主要特征：

（1）**服务发现**。Consul的客户端可以注册一个服务，比如api或者mysql，其他客户端可以使用Consul来发现某个服务的提供者，进行消费使用。使用DNS或HTTP两种发现方式。

（2）**健康检查**。Consul 提供任意数量的健康检查。支持多种协议HTTP/TCP/DOCKER/SHELL脚本等。

（3）**KV存储**。可以利用CONSUR的分层key/value存储进行任何目的，包括动态配置、功能标记、协调、领导人选举等。

（4）**安全服务通信**。Consul可以为服务生成和分发TLS证书，以建立相互TLS连接。可以用来定义意图的服务。服务分段可以很容易地管理，意图可以实时更改，而不是使用复杂的网络拓扑和静态防火墙规则。

（5）**多数据中心**。支持开箱即用的多个数据中心。

（6）**友好的Web界面**。



## 二、安装运行

### 1、windows安装

官方安装说明：https://learn.hashicorp.com/consul/getting-started/install.html

当前版本（2020.9）Consul 1.8.4。

下载完成后只有一个 `consul.exe`文件，文件路径下双击运行即可。

查看版本信息 windows 和 linux 是一样的。

````bash
consul --version
````

以开发模式启动：

```bash
consul agent -dev
```

Consul 的DNS协议代理TCP/UDP端口：8600

Consul 的HTTPS协议代理TCP端口：8500

Consul 的gRPC协议代理TCP端口：8502

访问Consul的web控制台：`http://localhost:8500`

停止Consul代理服务：

```bash
consul leave
```

### 2、Centos/RHEL 安装

其他系统参考官网：https://learn.hashicorp.com/tutorials/consul/get-started-install

（1）方式一

安装`yum-config-manager`管理镜像源：

```bash
yum install -y yum-utils
```

添加HashCorp 的linux 镜像源

```bash
yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
```

直接安装即可：

```bash
yum -y install consul
```

验证安装：

```bash
consul
```

如果有相关的命令提示表示安装成功。

（2）方式二

也可以使用zip下载解压的方式：

```bash
wget https://releases.hashicorp.com/consul/1.8.4/consul_1.8.4_linux_amd64.zip
unzip consul_1.8.4_linux_amd64.zip
```

然后验证一想安装。再将`consul`所在目录`/opt/consul`配置到`path` 。

```bash
vim ~/.bash_profile
```

添加配置：

```
PATH=$PATH:$HOME/bin：/opt/consul
```

使配置生效

```bash
source ~/.bash_profile
```

如果需要web控制台，则需要单独下载：

```bash
wget https://releases.hashicorp.com/consul/1.8.4/consul_1.8.4_web_ui.zip
```

Consul 常见参数：

- `server` 表示启动的为consul server ，构建一个consul cluster 一般建议使用3或者5个consul server
- `bootstrap-expect 1` 表示期望的服务节点数目为1
- `-data-dir` 数据目录，如果该文件夹不存在则手工创建,如果在consul发生错误后，建议先清理该目录文件
- `advertise` 设置广播地址,ip可以设置为公网ip
- `client` 设置client访问的地址
- `ui-dir` web控制台目录位置

执行命令启动：

```bash
consul -advertise 192.168.100.180 -client 192.168.100.181 -ui-dir /opt/consul/consul-ui
```

## 三、Consul 使用

因为Consul 作为服务注册中心，不需要再创建注册中心的工程。

那么其他微服务（提供者或消费者）相关配置基本相似。

### 1、引入pom依赖

```xml
<!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-consul-discovery -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>
 		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>



```

### 2、Config 配置

```yaml
server:
  port: 8006


spring:
  application:
    name: consul-provider-payment
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        service-name: ${spring.application.name}
```

### 3、开启服务发现

使用注解 `@EnableDiscoveryClient` 开启服务发现。

示例：

```java
package com.xiaocai.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class ConsulClientProvider8006 {
    public static void main(String[] args) {
        SpringApplication.run(ConsulClientProvider8006.class,args);
    }
}
```

### 4、针对消费端

如果服务提供方是多台，可以在调用方使用 Ribbon + RestTemplate 当时进行客户端负载均衡。

关于Ribbon 参考[《SpringCloud Ribbon》]()

补充配置：

```java
package com.xiaocai.springcloud.config;

import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class ApplicationContextConfig {

    @LoadBalanced //Ribbon 客户端负载均衡
    @Bean
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }

}
```





### 5、验证测试

打开Consul 的web控制台，查看服务是否正常注册进注册中心。

访问消费端地址，验证是否正常调用服务端返，浏览器查看是否正常返回数据。



## 四、总结

Consul 作为服务发现数据中心，是开箱即用的。

作为关注数据粒度的CAP理论：

- C ：Consistency  强一致性
- A ：Availability 可用性
- P ：Partition tolerance 分区容错

Consul 是符合CP 原则。

Zookeeper 作为服务注册中心也是CP原则。

