---
title: SpringCloud Config
tags: SpringCloud
categories: SpringCloud
summary: 本文记录SpringCloud Config入门相关学习和整理。
keywords: 'Spring Cloud,Spring Cloud 微服务,Config github整合,Config 原理,Config 配置中心,Config配置'
author: Small-Rose / 张小菜
abbrlink: 5b5896d1
date: 2020-09-20 19:00:00
---

## Spring Cloud Config 分布式配置中心


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

微服务将原本应用中的业务拆分成一个个子服务，每个服务粒度相对较小，因此系统中会出现大量服务。每个微服务都有自己的配置信息才能运行，当微服务较多的时候，有一套集中式的、动态的配置管理组件是适宜的。

Spring Cloud 提供了 ConfigServer 来解决这个问题。



## 二、Config

![Config](/images/springcloud/spcl-config-work.png)

### 1、基本信息

官方：https://cloud.spring.io/spring-cloud-static/spring-cloud-config/2.2.1.RELEASE/reference/html/

SpringCloud Config 为微服务架构中的各个微服务提供集中式的外部配置支持，配置服务器为各个不同微服务应用的所有环境提供了一个**中心化的外部配置**。

 SpringCloud Config  分为**服务端**和**客户端**两个部分。

服务端也叫分布式配置中心，它是一个独立的微服务应用，用来连接配置服务器并为客户端获取配置信息，加密解密信息等访问接口。

客户端则是通过指定的配置中心来管理应用资源、以及业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息。

配置服务器默认采用git来存储配置信息，这样有助于对环境配进行版本管理，且可以通过git客户端工具来方便的管理和访问配置内容。由于SpringCloud Config默认使用Git来存储配置文件，同时也有其它方式，比如支持svn和本地文件，但官方最推荐的还是Git，而且使用的是http/https访问的形式。

服务端负责将git（svn）中存储的配置文件发布成REST接口，客户端可以从服务端REST接口获取配置。但客户端并不能主动感知到配置的变化，从而主动去获取新的配置。客户端如何去主动获取新的配置信息呢，springcloud已经给我们提供了解决方案，每个客户端通过POST方法触发各自的刷新。 

### 2、主要特点

（1）针对不同环境不同配置，可以动态化配置更新，如多环境部署dev/test/prod/beta/release.

（2）运行期间动态调整配置，可以不需要在每个服务部署机器上编写配置文件，服务会向配置中心统一拉取自己的配置信息。

（3）当配置发生变动时，服务无须重启即可感知配置变化并应有新的配置。

（4）Config将配置信息以REST接口形式暴露

（5）默认使用git 存储，与github整合。



## 三、Config 使用

### 1、Config 配置-Git

### 1.1 服务端配置

#### （1）准备工作

登录github账号，在Github上新建一个名为sprincloud-config的新Repository 。

在本地硬盘上新建git仓库并clone刚刚的仓库，地址：`D:\springcloud2020`。

```bash
git clone git@github.com/username/sprincloud-config.git
```

在`D:\springcloud2020\sprincloud-config`目录新建多环境的配置文件，注意文件格式为**UTF-8**

- config-dev.yml

- config-prod.yml

- config-test.yml

config-dev.yml 的内容：
```yaml
config:
  info: this is master config-dev from github ---
```
config-test.yml 的内容：
```yaml
config:
  info: this is master config-test from github ---
```

config-prod.yml 的内容：
```yaml
config:
  info: this is master config-prod from github ---
```

如果需要修改可以模拟运维人员修改配置文件，进行提交。

```bash
git add .
git commit -m "init yml"
git push origin master
```

新建子工程cloud-config-2255，作为cloud的配置中心模块。

#### （2）引入pom依赖

```yaml
 		<!-- config server -->
 		<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
 
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

#### （3）YML配置

```yaml
server:
  port: 2255
spring:
  application:
    name: cloud-config-center
  cloud:
    config:
      server:
        git:
          uri:  填写你自己的github路径
          search-paths:
            - springcloud-config
      label: master
eureka:
  client:
    service-url:
      defaultZone:  http://localhost:7001/eureka
 

```

#### （4）启用Config服务

在主启动类上使用注解`@EnableConfigServer`

```java
package com.xiaocai.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer
public class ConfigCenterMain2255 {
    public static void main(String[] args) {
            SpringApplication.run(ConfigCenterMain2255 .class,args);
        }
}
```

写个访问的controller

```java
package com.xiaocai.springcloud.Controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ConfigClientController {

    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/configInfo")
    public String getConfigInfo(){
        return configInfo;
    }
}
```





#### （5）测试

注意：host 添加域名

```txt
127.0.0.1 config-2255.com
```

启动服务，访问测试：http://config-2255.com:2255/master/config-dev.yml

浏览器是否打印出`config-dev.yml`的内容。



#### （6）配置读取规则

官方列举：

http服务支持的格式

```txt
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```

推荐使用格式：

`/{label}/{application}-{profile}.yml`

`/{label}/{application}-{profile}.properties`

label 只的github 仓库里的分支

master分支：

http://config-2255.com:2255/master/config-dev.yml

http://config-2255.com:2255/master/config-test.yml

http://config-2255.com:2255/master/config-prod.yml

dev分支：

http://config-2255.com:2255/dev/config-dev.yml

http://config-2255.com:2255/dev/config-test.yml

http://config-2255.com:2255/dev/config-prod.yml

小结：

`/{label}/{application}-{profile}.yml`

- label ：分支
- application ：服务名
- profiles： 环境（dev/test/prod）



### 1.2 客户端配置

客户端工程是除Config Server 之外的其他微服务。 

新建子工程 cloud-config-client-2266

#### （1）引入pom依赖

```xml
		<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
```

#### （2）新建bootstrap.yml配置

`application.yml` 是用户级的资源配置项；

`bootstrap.yml `是系统级的资源配置项，优先级更高。

Spring Cloud 会创建一个 Bootstrap Context ，作为spring 应用的 Application Context 的父上下文。初始化时，Bootstrap Context 负责从外部源加载配置属性并解析配置。这两个上下文共享一个从外部获取的 Envitonment 。

`bootstrap ` 属性有更高的优先级，默认情况下不会被本地配置覆盖。`Bootstrap Context`  和 `Application Context` 有着不同的约定，所以新增了一个`bootstrap.yml `文件，保证 `Bootstrap Context` 和 `Application Context `配置的分离。

```yaml
server:
  port: 2266

spring:
  application:
    name: config-client
  cloud:
    config:
      label: master  # /{label}/{name}-{profile}.yml 
      name: config   # /{label}/{name}-{profile}.yml
      profile: dev   # /{label}/{name}-{profile}.yml
      uri: http://localhost:2255 # 配置中心地址
eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka
 
```

`spring.cloud.config.label` 、`spring.cloud.config.name` 、`spring.cloud.config.profile` 三个值会按照`/{label}/{name}-{profile}.yml `的格式组合成要读取的配置文件名。

#### （3）启用类和访问地址

启用类

```java
package com.xiaocai.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer
public class ConfigClient_2266 {
    public static void main(String[] args) {
            SpringApplication.run(ConfigClient_2266.class,args);
        }
}
```

访问地址

```java
package com.xiaocai.springcloud.Controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ConfigClientController {

    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/configInfo")
    public String getConfigInfo(){
        return configInfo;
    }
}
```



#### （4）测试

（A）修改本地仓库配置文件

修改本地硬盘上git仓库里的文件： `config-dev.yml` 配置并提交到GitHub中，比如加个变量xiaocai 或者版本号version=1.2

config-dev.yml 的内容：

```yaml
config:
  info: this is master config-dev from github --- version=1.2 --- xiaocai test
  
```
（B）启动两个工程测试

启动config server 进行自测

访问1：http://config-2255.com:2255/master/config-dev.yml

访问2：http://config-2255.com:2255/master/config-test.yml

启动confi client ，测试访问：http://localhost:2266/configInfo 看是否能获取修改后的最新配置。



**工程不要停止**，再次修改config-dev.yml 的内容：

```yaml
config:
  info: this is master config-dev from github --- version=1.3 --- xiaocai dev
  
```

提交到github远程仓库。

重新访问Config Server：http://config-2255.com:2255/master/config-dev.yml 会发现配置中心的配置立即更新了。

重新访问Config Client：http://localhost:2266/configInfo  会发现配置没有更新。

重启Config Client 之后，再次访问 http://localhost:2266/configInfo  配置才会更新。

#### （4）Config Client 动态刷新

在客户端工程 cloud-config-client-2266 引入监控模块：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

修改YML配置：

```yaml
server:
  port: 2266

spring:
  application:
    name: config-client
  cloud:
    config:
      label: master  # /{label}/{name}-{profile}.yml 
      name: config   # /{label}/{name}-{profile}.yml 
      profile: dev   # /{label}/{name}-{profile}.yml 
      uri: http://localhost:2255  # 配置中心地址
eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka

## 暴露健康端点，此处使用 *
management:
  endpoints:
    web:
      exposure:
        include: "*"

```

添加注解`@RefreshScope` ：

```java
package com.xiaocai.springcloud.Controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RefreshScope
@RestController
public class ConfigClientController {

    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/configInfo")
    public String getConfigInfo(){
        return configInfo;
    }
}
```

测试操作：

启动config server 进行自测

访问1：http://config-2255.com:2255/master/config-dev.yml 

启动config client ，测试访问：http://localhost:2266/configInfo 查看

**工程不要停止**，再次修改config-dev.yml 的内容：

```yaml
config:
  info: this is master config-dev from github --- version=1.4 --- xiaocai dev22
  
```

提交到github远程仓库。

重新访问Config Server：http://config-2255.com:2255/master/config-dev.yml 会发现配置中心的配置立即更新了。

此时手工发送post请求刷新端口为2266的config client工程，执行命令：

```bash
curl -X POST "http://localhost:2266/actuator/refresh"
```

（ windows 需要自己安装curl 命令）

重新访问Config Client：http://localhost:2266/configInfo  会发现配置更新。

**工程不要停止**，再次修改config-dev.yml 的内容反复以上操作进行测试验证。



### 2、Config 配置-SVN

### 2.1 服务端

#### （1）准备工作

电脑安装一下下 SVN Server，官方下载地址：https://www.visualsvn.com/server/download/

在SVN Server 中新建一个仓库 `sprincloud-config`

创建一个 config 目录。

在本地硬盘上创建一个SVN检出的文件夹 `D:\springcloud-svn`

在目录进行SVN检出操作，SVN地址:https://DESKTOP-77P2ES4/svn/sprincloud-config/config

创建三个配置文件：

- config-dev.properties
- config-prod.properties
- config-test.properties

特别说明：properties 格式或 yml格式都可以，最好和工程里的配置文件格式一致。

config-dev.properties的内容：

```properties
config.info=this is trunk config-dev from svn server ---
```

config-test.properties的内容：

```properties
config.info=this is trunk config-test from svn server ---
```

config-prod.properties 的内容：

```properties
config.info=this is trunk config-prod from svn server ---
```



并提交到SVN仓库。

 创建子工程 Config-SVN-2277 

#### （2）引入依赖

```xml
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
<!-- https://mvnrepository.com/artifact/org.tmatesoft.svnkit/svnkit -->
        <dependency>
            <groupId>org.tmatesoft.svnkit</groupId>
            <artifactId>svnkit</artifactId>
            <version>1.9.3</version>
        </dependency>
```

注意：这里引入了 svnkit 的依赖。

#### （3）配置文件

```properties
server.port=8003
spring.application.name=config-svn-2277
spring.cloud.config.server.svn.uri=https://DESKTOP-77P2ES4/svn/sprincloud-config/config
spring.cloud.config.server.svn.search-paths=
spring.cloud.config.server.svn.username=xiaocai
spring.cloud.config.server.svn.password=123456
spring.profiles.active=subversion
spring.cloud.config.server.default-label=
```

注意：如果svn地址中有主分支标记 `trunk`，那么default-label=trunk，和git的相似。

#### （4）启用config 服务

使用`@EnableConfigServer `注解激活即可。

```java
package com.xiaocai.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer
public class ConfigSVNApp_2277 {
    public static void main(String[] args) {
            SpringApplication.run(ConfigSVNApp_2277 .class,args);
        }
}
```

#### （5）测试

启动工程，访问：<http://localhost:2277/config-dev.properties> 



### 2.2 客户端

新建子工程 cloud-config-client-2288

#### （1）引入pom依赖

```xml
		<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

#### （2）2个配置文件

需要配置两个配置文件，application.properties和bootstrap.properties

application.properties如下：

```properties
spring.application.name=spring-cloud-config-client
server.port=2288
```

bootstrap.properties如下：

```properties
spring.cloud.config.name=config
spring.cloud.config.profile=dev
#配置中心地址
spring.cloud.config.uri=http://localhost:2277/   
spring.cloud.config.label=master

# 开启端点暴露
management.endpoints.web.exposure.include=*
```

- spring.application.name：对应{application}部分
- spring.cloud.config.profile：对应{profile}部分
- spring.cloud.config.label：对应git的分支。如果配置中心使用的是本地存储，则该参数无用
- spring.cloud.config.uri：配置中心的具体地址
- spring.cloud.config.discovery.service-id：指定配置中心的service-id，便于扩展为高可用配置集群。

> 特别注意：上面这些与spring-cloud相关的属性必须配置在bootstrap.properties中，config部分内容才能被正确加载。因为config的相关配置会先于application.properties，而bootstrap.properties的加载也是先于application.properties。 bootstrap.properties和bootstrap.yml 相同，只是文件格式不一样。

#### （3）controller

`@RefreshScope`注解开启自动刷新，在客户端执行/refresh的时候就会更新此类下面的变量值 ：

```java
package com.xiaocai.springcloud.Controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RefreshScope
@RestController
public class ConfigSVNClientController {

    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/configInfo")
    public String getConfigInfo(){
        return configInfo;
    }
}
```

#### （4）测试

启动config svn 配置中心进行自测

访问1：http://config-2277.com:2277/config-dev.properties （如果这种不行就写个controller访问）

启动config client ，测试访问：http://localhost:2288/configInfo 查看

**工程不要停止**，修改config-dev.properties 的内容：

```properties
config.info=this is master config-dev from github --- version=S1.2 --- 
```

提交到SVN仓库。

重新访问Config Server：http://config-2277.com:2277/config-dev.properties 会发现配置中心的配置立即更新了。

此时手工发送post请求刷新端口为2288的config client工程，执行命令：

```bash
curl -X POST "http://localhost:2288/actuator/refresh"
```

（ windows 需要自己安装curl 命令）

重新访问Config Client：http://localhost:2277/config-dev.properties会发现配置更新。

**工程不要停止**，再次修改config-dev.properties的内容反复以上操作进行测试验证。

以post请求的方式来访问http://localhost:2288/configInfo  就会更新修改后的配置文件。 



## 四、Config 其他

后续学习再补充。