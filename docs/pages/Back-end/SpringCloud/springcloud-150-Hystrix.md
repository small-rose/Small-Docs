---
title: SpringCloud Hystrix
tags: SpringCloud
categories: SpringCloud
summary: 本文记录SpringCloud Hystrix相关学习和整理。
keywords: >-
  Spring Cloud,Spring Cloud 微服务,Hystrix,Hystrix 原理,Hystrix 工作原理,Hystrix
  配置,Hystrix 使用,Hystrix 断路器,Hystrix 服务降级，Hystrix 服务熔断,Hystrix 服务限流
author: Small-Rose / 张小菜
abbrlink: 464bb413
date: 2020-09-17 15:00:00
---

## SpringCloud Hystrix熔断器


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

分布式系统环境下，服务间类似依赖非常常见，一个业务调用通常依赖多个基础服务。 

### 1、服务雪崩

多个微服务调用时，服务A调用服务B和服务C，服务B和C又调用其他微服务...这就是**扇出**。

扇出链路上某个或某些微服务的调用响应时间过长或不可用，对服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃，即发生”雪崩效应“。

**雪崩效应常见场景**

- 硬件故障：如服务器宕机，机房断电，光纤损坏等。
- 流量激增：如异常流量，重试加大流量等。
- 缓存穿透：一般发生在应用重启，所有缓存失效时，以及短时间内大量缓存失效时。大量的缓存不命中，使请求直击后端服务，造成服务提供者超负荷运行，引起服务不可用。
- 程序BUG：如程序逻辑导致内存泄漏，JVM长时间FullGC等。
- 同步等待：服务间采用同步调用模式，同步等待造成的资源耗尽。

**雪崩效应应对策略**

针对造成雪崩效应的不同场景，可以使用不同的应对策略，没有一种通用所有场景的策略，参考如下：

- 硬件故障：多机房容灾、异地多活等。
- 流量激增：服务自动扩容、流量控制（限流、关闭重试）等。
- 缓存穿透：缓存预加载、缓存异步加载等。
- 程序BUG：修改程序bug、及时释放资源等。
- 同步等待：资源隔离、MQ解耦、不可用服务调用快速失败等。资源隔离通常指不同服务调用采用不同的线程池；不可用服务调用快速失败一般通过熔断器模式结合超时机制实现。

一个应用不能对来自依赖的故障进行隔离，那该应用本身就处在被拖垮的风险中。 

雪崩效应常见场景及对策参考文章：https://blog.csdn.net/loushuiyifan/article/details/82702522

因此Hystrix出现了。

![官方图标](/images/springcloud/hystrix-00.png)

Hystrix [hɪst'rɪks]，中文含义是豪猪，因其背上长满棘刺，从而拥有了自我保护的能力。



## 二、Hystrix

Hystrix是Netflix开源的一款容错框架。

Hystrix是一个用于处理分布式系统的延迟和容错的开源库，Hystrix能够保证在一个依赖出问题的情况下，不会导致整体服务是不，避免级联故障，以提高分布式系统的弹性。

### 1、Hystrix目标

Hystrix被设计的目标是：

1. 对通过第三方客户端库访问的依赖项（通常是通过网络）的延迟和故障进行保护和控制。
2. 在复杂的分布式系统中阻止级联故障。
3. 快速失败，快速恢复。
4. 回退，尽可能优雅地降级。
5. 启用近实时监控、警报和操作控制。

### 2、Hystrix解决了什么问题

复杂分布式体系结构中的应用程序有许多依赖项，每个依赖项在某些时候都不可避免地会失败。如果主机应用程序没有与这些外部故障隔离，那么它有可能被他们拖垮。

例如，对于一个依赖于30个服务的应用程序，每个服务都有99.99%的正常运行时间，你可以期望如下：

99.9930  =  99.7% 可用

也就是说一亿个请求的0.03% = 3000000 会失败

如果一切正常，那么每个月有2个小时服务是不可用的

现实通常是更糟糕。 

正常情况下：

![](/images/springcloud/hystrix-01.png)

当其中有一个系统有延迟时，它可能阻塞整个用户请求： 

![](/images/springcloud/hystrix-02.png)

在高流量的情况下，一个后端依赖项的延迟可能导致所有服务器上的所有资源在数秒内饱和：

![](/images/springcloud/hystrix-03.png)

使用Hystrix来包装每个依赖项时，上图中所示的架构会发生变化，如下图所示：
每个依赖项相互隔离，当延迟发生时，它会被限制在资源中，并包含回退逻辑，该逻辑决定在依赖项中发生任何类型的故障时应作出何种响应：

![](/images/springcloud/hystrix-04.png)


### 3、Hystrix设计原则

Hystrix 涉及原则

- 防止任何单个依赖项耗尽所有容器（如Tomcat）用户线程。
- 甩掉包袱，快速失败而不是排队。
- 在任何可行的地方提供回退，以保护用户不受失败的影响。
- 使用隔离技术（如隔离板、泳道和断路器模式）来限制任何一个依赖项的影响。
- 通过近实时的度量、监视和警报来优化发现时间。
- 通过配置的低延迟传播来优化恢复时间。
- 支持对Hystrix的大多数方面的动态属性更改，允许使用低延迟反馈循环进行实时操作修改。
- 避免在整个依赖客户端执行中出现故障，而不仅仅是在网络流量中。

### 4、Hystrix 工作流程

Hystrix 熔断工作流程图

![](/images/springcloud/hystrix-work.png)

Hystrix整个工作流如下：

1. 构造一个 HystrixCommand或HystrixObservableCommand对象，用于封装请求，并在构造方法配置请求被执行需要的参数；
2. 执行命令，Hystrix提供了4种执行命令的方法，后面详述；
3. 判断是否使用缓存响应请求，若启用了缓存，且缓存可用，直接使用缓存响应请求。Hystrix支持请求缓存，但需要用户自定义启动；
4. 判断熔断器是否打开，如果打开，跳到第8步；
5. 判断线程池/队列/信号量是否已满，已满则跳到第8步；
6. 执行HystrixObservableCommand.construct()或HystrixCommand.run()，如果执行失败或者超时，跳到第8步；否则，跳到第9步；
7. 统计熔断器监控指标；
8. 走Fallback备用逻辑
9. 返回请求响应



Hystrix 隔离

Hystrix隔离方式采用线程/信号的方式，通过隔离限制依赖的并发量和阻塞扩散 

（1）线程隔离

Hystrix在用户请求和服务之间加入了线程池。

Hystrix为每个依赖调用分配一个小的线程池，如果线程池已满调用将被立即拒绝，默认不采用排队.加速失败判定时间。线程数是可以被设定的。

原理：用户的请求将不再直接访问服务，而是通过线程池中的空闲线程来访问服务，如果线程池已满，则会进行降级处理，用户的请求不会被阻塞，至少可以看到一个执行结果（例如返回友好的提示信息），而不是无休止的等待或者看到系统崩溃。

（2）信号隔离

​         信号隔离也可以用于限制并发访问，防止阻塞扩散,  与线程隔离最大不同在于执行依赖代码的线程依然是请求线程（该线程需要通过信号申请, 如果客户端是可信的且可以快速返回，可以使用信号隔离替换线程隔离,降低开销。信号量的大小可以动态调整,  线程池大小不可以。



## 三、Hystrix使用

Hystrix 比较重要的几个概念/功能。

- 服务降级
- 服务熔断
- 服务限流
- 服务监控

### 1、服务降级

服务降级：在请求服务时，服务器因某些原因（异常/超时等原因）繁忙无法立即响应，Hystrix会有一个fallback 的备选响应。

服务降级引发原因：

- 程序运行异常
- 超时，执行开始冒烟在允许的时间内完成
- 服务熔断触发服务降级
- 线程池/信号量拒绝，不尝试执行



超时快速响应，异常快速失败。

**服务降级测试**：

进行服务降级测试的时候，可以让线程Sleep模拟超时，手工制造异常模拟运行异常，停止服务模拟宕机进行访问。



#### （1）针对特定方法的服务降级

在服务提供方，针对某个可能会频繁调用的核心业务方法进行服务降级。

A、在业务类中启用，使用`@HystrixCommand`注解开启。

B、在主启动类启用熔断器，使用`@EnableCircuitBreaker`注解。

示例：

业务类方法：

```java
package com.xiaocai.springcloud.service;

import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixProperty;
import org.springframework.stereotype.Service;

import java.util.concurrent.TimeUnit;

@Service
public class UserHystrixService {

    //成功
    public String usertInfo_OK(Integer id){
        return "线程池："+Thread.currentThread().getName()+"   usertInfo_OK,id：  "+id+"\t"+"哈哈哈"  ;
    }

    //失败
    @HystrixCommand(fallbackMethod = "usertInfo_TimeOutHandler",commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "3000")  //表示在3秒钟以内响应就是正常的业务逻辑
    })
    public String usertInfo_TimeOut(Integer id){
       	//这里写业务逻辑代码
        int timeNumber = 5;
        int age = 1/0;
        // try { TimeUnit.SECONDS.sleep(timeNumber); }catch (Exception e) {e.printStackTrace();}
        return "线程池："+Thread.currentThread().getName()+"   usertInfo_TimeOut,id：  "+id+"\t"+" 耗时(秒)"+timeNumber;
    }

    //兜底方法
    public String usertInfo_TimeOutHandler(Integer id){
        return "线程池："+Thread.currentThread().getName()+"   系统繁忙, 请稍候再试  ,id：  "+id+"\t"+" 服务降级";
    }

    //成功
    public String getUserById(Integer id){
        return   ;
    }
}
```

主启动类`@EnableCircuitBreaker`示例：

```java

@SpringBootApplication
@EnableEurekaClient 
@EnableCircuitBreaker
public class UserInfoApplication {

    public static void main(String[] args) {
        SpringApplication.run(UserInfoApplication.class, args);
    }

}
```



在服务消费方，针对前端或浏览器调用的方法进行服务降级。

（A）开启配置

```yaml
feign:
  hystrix:
    enabled: true #如果处理自身的容错就开启。
 
```

（B）主启动类，添加注解`@EnableHystrix`

（C）消费方调用业务类的controller

```java
package com.xiaocai.springcloud.service;

import com.xiaocai.springcloud.entities.CommonResult;
import com.xiaocai.springcloud.entities.User;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixProperty;
import org.springframework.stereotype.Service;

import java.util.concurrent.TimeUnit;

@RestController
public class UserController {
    
    @Autowired
    private UserFeignService userFeignService ;
    
    @GetMapping("/consumer/user/hystrix/timeout/{id}")
    @HystrixCommand(fallbackMethod = "userTimeOutFallbackMethod",commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "1500")  //1.5秒钟以内就是正常的业务逻辑
    })
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id){
        String result = userFeignService.usertInfo_TimeOut(id);
        return result;
    }

    //fallback方法
    public String userTimeOutFallbackMethod(@PathVariable("id") Integer id){
        return "我是消费端，系统繁忙请10秒钟后再试或者自己运行出错请检查自己,(┬＿┬)";
    }
}

```

对`@HystrixCommand`内属性的修改需要重启微服务。



#### （2）针对所有方法全局服务降级

实际开发中，可能有很多方法，为每个方法写一个专属的服务降级方法，导致方法特别多，代码膨胀。只有几个特点的核心方法、调用频繁的方进行特定fallback降级服务。其他的可以使用全局公共的fallback进行服务降级。

在类的头部声明默认的fallback。

注意，这里的`usertInfo_TimeOut` 方法的特定服务降级`@HystrixCommand`是在userHystrixService中写的。

```java
package com.xiaocai.springcloud.service;

import com.xiaocai.springcloud.entities.CommonResult;
import com.xiaocai.springcloud.entities.User;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixProperty;
import org.springframework.stereotype.Service;

import java.util.concurrent.TimeUnit;

@RestController
@DefaultProperties(defaultFallback = "default_Global_Fallback")
public class UserProviderController {
    @Autowired
 	private UserHystrixService userHystrixService;
    
    @GetMapping("/v1/user/hystrix/timeout/{id}")
    public String usertInfo_TimeOut(@PathVariable("id") Integer id){
        String result = userHystrixService.usertInfo_TimeOut(id);
        return result;
    }
    
  	@GetMapping(value = "/v1/user/get/{id}")
    public CommonResult getUserById(@PathVariable("id") Long id){
        User user = userHystrixService.getUserById(id);
        CommonResult result = new CommonResult(200,"调用成功"，user);
    	return result;
    }
    
    @GetMapping(value = "/v1/user/ok/{id}")
    public CommonResult usertInfo_OK(@PathVariable("id") Long id){
    	return userHystrixService.usertInfo_OK(id);
    }
    
    public String default_Global_Fallback(){
    	return "Global Fallback  系统繁忙，请10秒之后再试！";
    }
}
```

#### （3）配合Feign的使用

既然是使用Feign，那么肯定是在调用端/客户端工程。

配合Feign的使用的时候可以更好的解耦，将服务降级分离出正常的业务逻辑代码中。

（A）配置

```yaml
feign:
  hystrix:
    enabled: true #如果处理自身的容错就开启。
 
```



（B）存在一个对服务调用的Feign的Service接口封装，并指明fallback的类是`UserFallbackService`

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
@FeignClient(value = "MS-CLOUD-USER-SERVICE"，fallback=UserFallbackService.class)
public interface UserFeignService {

    @GetMapping(value = "/v1/user/get/{id}")
    public CommonResult getUserById(@PathVariable("id") Long id);
    
    
    @GetMapping(value = "/v1/user/ok/{id}")
    public CommonResult usertInfo_OK(@PathVariable("id") Long id);
    
    @GetMapping(value = "/v1/user/hystrix/timeout/{id}")
    public CommonResult usertInfo_TimeOut(@PathVariable("id") Long id);
    
}
```

（C）新建一个降级使用的类`UserFallbackService`，并实现Feign的Service 接口

```java
package com.xiaocai.springcloud.service;
...
    
@Component
public class UserFallbackService implements UserFeignService{

    @Override
    public String getUserById(){
    	return "我是 getUserById 的服务降级方法，来自 UserFallbackService "；
    }
	
    @Override
    public String userInfo_OK(){
    	return "我是 userInfo_OK 的服务降级方法，来自 UserFallbackService "；
    }

    @Override
    public String usertInfo_TimeOut(){
    	return "我是 usertInfo_TimeOut 的服务降级方法，来自 UserFallbackService "；
    }
}
```

该类中对应方法的服务降级fallback响应。这样服务降级就和业务进行分离。



#### （3）fallback方法抛出异常

fallback方法在什么情况下会抛出异常

| 名称 | 描述          | 是否抛异常  |
| ----------------- | ------------------------------ | ---- |
| FALLBACK_FAILURE  | Fallback执行抛出出错           | YES  |
| FALLBACK_REJECTED | Fallback信号量拒绝，不尝试执行 | YES  |
| FALLBACK_MISSING  | 没有Fallback实例               | YES  |



### 2、服务熔断

服务熔断：由于某些原因使得服务出现了过载现象，为防止造成整个系统故障，从而采用的一种保护措施，所以很多地方把熔断亦称为过载保护。 

类似保险开关功能，服务器接近最大服务访问后，拉闸限电，直接拒绝访问，然后调用服务降级的方法并返回友好提示。

**Hystrix 熔断机制**：

如果某个目标服务调用慢或者有大量超时，此时，熔断该服务的调用，对于后续调用请求，不在继续调用目标服务，直接返回，快速释放资源。如果目标服务情况好转则恢复调用。 

（服务降级——> 服务熔断——>恢复调用链路）

熔断相关论述https://martinfowler.com/bliki/CircuitBreaker.html

**熔断器:Circuit Breaker**

​      熔断器是位于线程池之前的组件。用户请求某一服务之后，Hystrix会先经过熔断器，此时如果熔断器的状态是打开（跳起），则说明已经熔断，这时将直接进行降级处理，不会继续将请求发到线程池**。**熔断器相当于在线程池之前的一层屏障。每个熔断器默认维护10个bucket ，每秒创建一个bucket ，每个blucket记录成功,失败,超时,拒绝的次数。当有新的bucket被创建时，最旧的bucket会被抛弃。

​       

​           ![熔断器的状态](/images/springcloud/hystrix-circuit-break.jpg) 

#### （1）熔断使用

熔断使用依旧是注解 `@HystrixCommand`

```java
@HystrixCommand(fallbackMethod = "userCircuitBreaker_fallback",commandProperties = {
     @HystrixProperty(name = "circuitBreaker.enabled",value = "true"),  //是否开启断路器
     @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"),   //请求次数
     @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "10000"),  //时间范围
     @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "60"), //失败率达到多少后跳闸
})
```



熔断主要是针对服务提供者。

Service 示例：

```java
package com.xiaocai.springcloud.service;

import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixProperty;
import org.springframework.stereotype.Service;

import java.util.concurrent.TimeUnit;

@Service
public class UserHystrixService {

    //成功
    public String usertInfo_OK(Integer id){
        return "线程池："+Thread.currentThread().getName()+"   usertInfo_OK,id：  "+id+"\t"+"哈哈哈"  ;
    }

    //失败-服务降级
    @HystrixCommand(fallbackMethod = "usertInfo_TimeOutHandler",commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "3000")  //表示在3秒钟以内响应就是正常的业务逻辑
    })
    public String usertInfo_TimeOut(Integer id){
       	//这里写业务逻辑代码
        int timeNumber = 5;
        int age = 1/0;
        // try { TimeUnit.SECONDS.sleep(timeNumber); }catch (Exception e) {e.printStackTrace();}
        return "线程池："+Thread.currentThread().getName()+"   usertInfo_TimeOut,id：  "+id+"\t"+" 耗时(秒)"+timeNumber;
    }

    //兜底方法
    public String usertInfo_TimeOutHandler(Integer id){
        return "线程池："+Thread.currentThread().getName()+"   系统繁忙, 请稍候再试  ,id：  "+id+"\t"+" 服务降级";
    }

    //成功
    public String getUserById(Integer id){
        return   ;
    }
    
    
    //服务熔断
    @HystrixCommand(fallbackMethod = "userCircuitBreaker_fallback",commandProperties = {
            @HystrixProperty(name = "circuitBreaker.enabled",value = "true"),  //是否开启断路器
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"),   //请求次数
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "10000"),  //时间范围
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "60"), //失败率达到多少后跳闸
    })
    public String userCircuitBreaker(@PathVariable("id") Integer id){
        if (id < 0){
            throw new RuntimeException("*****id 不能负数");
        }
        String serialNumber = IdUtil.simpleUUID();

        return Thread.currentThread().getName()+"\t"+"调用成功,流水号："+serialNumber;
    }
    public String userCircuitBreaker_fallback(@PathVariable("id") Integer id){
        return "id 不能负数，请稍候再试,(┬＿┬)/~~     id: " +id;
    }


}
```

在controller中的调用，就平时service调用方法一致。

```java
package com.xiaocai.springcloud.service;

import com.xiaocai.springcloud.entities.CommonResult;
import com.xiaocai.springcloud.entities.User;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixProperty;
import org.springframework.stereotype.Service;

import java.util.concurrent.TimeUnit;

@RestController
@DefaultProperties(defaultFallback = "default_Global_Fallback")
public class UserProviderController {
    @Autowired
 	private UserHystrixService userHystrixService;
    
    @GetMapping("/v1/user/hystrix/timeout/{id}")
    public String usertInfo_TimeOut(@PathVariable("id") Integer id){
        String result = userHystrixService.usertInfo_TimeOut(id);
        return result;
    }
    
  	@GetMapping(value = "/v1/user/get/{id}")
    public CommonResult getUserById(@PathVariable("id") Long id){
        User user = userHystrixService.getUserById(id);
        CommonResult result = new CommonResult(200,"调用成功"，user);
    	return result;
    }
    
    @GetMapping(value = "/v1/user/ok/{id}")
    public CommonResult usertInfo_OK(@PathVariable("id") Long id){
    	return userHystrixService.usertInfo_OK(id);
    }
    
    //服务熔断测试
    @GetMapping(value = "/v1/user/circuit/{id}")
    public CommonResult usertInfo_OK(@PathVariable("id") Long id){
    	return userHystrixService.userCircuitBreaker(id);
    }
    
    public String default_Global_Fallback(){
    	return "Global Fallback  系统繁忙，请10秒之后再试！";
    }
}
```

测试：

测试访问地址1：http://localhost:6002/v1/user/circuit/10

测试访问地址2：http://localhost:6002/v1/user/circuit/-10

一次正确一次错误。

频繁刷新地址2之后，再频繁访问地址1，多次错误,然后慢慢正确，发现刚开始不满足条件，就算是正确的访问地址也不能进行访问，需要慢慢的恢复链路。



#### （2）熔断器参数

**熔断器/断路器的三个重要参数**：快照时间窗、请求总数阈值、错误百分比阈值。

（A）**快照时间窗**：熔断器确定是否打开需要统计一些请求和错误数据，而统计的世界范围就是快照时间窗，默认为最近10秒。

```java
@HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "10000")
```

（B）**请求总数阈值**：在快照时间窗内，必须满足请求总数阈值才有资格熔断。默认是20，即在10秒内，如果Hystrix命令的调用次数不足20次，即使所有请求都超时或其他原因失败，熔断器都不会打开。

```java
@HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "20"),
@HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "10000"),
```

（C）**错误百分比阈值**：当请求总数在快照时间窗内超过了阈值，比如在10秒内发生40次调用，如果在40次调用中有20次调用发生了超时或异常，即超过50%的错误百分比，在默认设定50%阈值的情况下，这时熔断器打开。

```java
@HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "10000"),
@HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "60"), 
```

熔断器打开的条件：

- 当满足一定阈值的时候（默认是10秒内超过20次）
- 当失败率达到一定的时候（默认是10秒内超过50%）

熔断器打开的时候，会引发服务降级，直接进入fallback的备选响应。

熔断器打开对主逻辑熔断之后，Hystrix会进入一个休眠时间窗，该期间，降级逻辑会成为主逻辑进行响应。

一段时间后（休眠时间窗结束，默认是5秒），熔断器自动进入半开状态，熔断器会让其中一次（少量）请求放行，进入正常的业务主逻辑，如果成功，断路器会关闭，放行后续请求，恢复正常的业务逻辑线；若请求失败，熔断器继续开启，依旧进行服务降级，进入fallback的备选响应。等待一段时间（休眠时间窗）后，再次进入半开状态，重复该步骤。

**常用参数：**

**## 超时时间 ##**

（1）hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds

在调用方配置，被该调用方的所有方法的超时时间都是该值，优先级低于下边的指定配置

（2）hystrix.command.HystrixCommandKey.execution.isolation.thread.timeoutInMilliseconds

在调用方配置，被该调用方的指定方法（HystrixCommandKey方法名）的超时时间是该值

线程池核心线程数

hystrix.threadpool.default.coreSize（默认为10）

**## Queue ##**

（1）hystrix.threadpool.default.maxQueueSize（最大排队长度。默认-1，使用SynchronousQueue。其他值则使用 LinkedBlockingQueue。如果要从-1换成其他值则需重启，即该值不能动态调整，若要动态调整，需要使用到下边这个配置）

（2）hystrix.threadpool.default.queueSizeRejectionThreshold（排队线程数量阈值，默认为5，达到时拒绝，如果配置了该选项，队列的大小是该队列）

注意：如果maxQueueSize=-1的话，则该选项不起作用

**## 断路器 ##**

（1）`hystrix.command.default.circuitBreaker.requestVolumeThreshold`（当在配置时间窗口内达到此数量的失败后，进行熔断路。默认20个）

简言之，10s内请求失败数量达到20个，断路器开。

（2）`hystrix.command.default.circuitBreaker.sleepWindowInMilliseconds`（短路多久以后开始尝试是否恢复，默认5s）

（3）`hystrix.command.default.circuitBreaker.errorThresholdPercentage`（出错百分比阈值，当达到此阈值后，开始短路。默认50%）

**## fallback ##**  

hystrix.command.default.fallback.isolation.semaphore.maxConcurrentRequests（调用线程允许请求HystrixCommand.GetFallback()的最大数量，默认10。超出时将会有异常抛出，注意：该项配置对于THREAD隔离模式也起作用）

**熔断器全部参数：**

官方文档配置说明：https://github.com/Netflix/Hystrix/wiki/Configuration

```java
@HystrixCommand(fallbackMethod = "user_fallbackMethod",
                groupKey = "user_GroupCommand",
                commandKey = "user_Command", 
                threadPoolKey = "user_ThreadPool",
                commandProperties = {
     //设置隔离策略，THREAD-表示线程池隔离；SEMAPHORE-信号池隔离
     @HystrixProperty(name = "execution.isolation.strategy",value = "THREAD"),
     //放隔离策略选择信号池隔离时，用来设置信号池的大小（最大并发量）
     @HystrixProperty(name = "execution.isolation.semaphore.maxConcurrentRequests",value = "10"),
     //是否启动超时时间
     @HystrixProperty(name = "execution.timeout.enabled",value = "true"),   
     //配置命令执行的超时时间
     @HystrixProperty(name = "execution.isolation.thread.timeoutinMilliseconds",value = "10"), 
     //执行超时的时候是否中断
     @HystrixProperty(name = "execution.isolation.thread.interruptOnTimeout",value = "true"), 
     //执行被取消的时候是否中断
     @HystrixProperty(name = "execution.isolation.thread.interruptOnCancel",value = "true"),
                    
     //允许回调方法执行的最大并发数
     @HystrixProperty(name = "fallback.isolation.semaphore.maxConcurrentRequests",value = "10"),
     //服务降级是否启用，是否执行回调函数
     @HystrixProperty(name = "fallback.enabled",value = "true"),
     //-------------------------------熔断------------------------      
     //是否开启断路器/熔断器
     @HystrixProperty(name = "circuitBreaker.enabled",value = "true"), 
     //该属性用来设置在滚动时间窗中，熔断器熔断的最小请求数，例如，默认该值为20的时候，如果滚动时间窗（默认10秒）内收到19个请求，即使19个请求都失败了，熔断器也不会打开。
     @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "20"),   	   
     // 该属性用来设置在滚动时间窗中，请求数量超过circuitBreaker.requestVolumeThreshold的情况下，如果错误请求百分比超过50%，就把熔断器设置为打开状态，否则设置为关闭状态。即失败率达到多少后跳闸
                    @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "50"),
     // 该属性用来设置当熔断器打开之后的休眠时间窗，休眠时间窗结束之后，会将熔断器置为"半开"状态，尝试放行熔断的请求命令，如果放行请求依然失败就将熔断器继续设置为"打开"状态，如果成功就设置为"关闭"状态.     
     @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "5000"), 
     // 熔断器强制打开               
     @HystrixProperty(name = "circuitBreaker.forceOpen",value = "false"), 
     // 熔断器强制关闭             
     @HystrixProperty(name = "circuitBreaker.forceClosed",value = "false"),

     // -------------------------
     // 滚动时间窗设置，该时间用于熔断器判断健康度时需要手机信息的持续时间             
     @HystrixProperty(name = "metrics.rollingStats.timeinMilliseconds",value = "10000"),

     // 该属性用来设置在滚动时间窗内统计指标信息时划分"桶"的数量。熔断器在手机指标信息的时候会根据设置的时间窗长度拆分成多个"桶" 来累计各度量值，每个"桶"记录了一段时间内的采集指标。比如在10秒 内拆分成10个“桶”收集样本，所以 timeinMilliseconds 必须能被 numBuckets 整除。否则会抛出异常。      
     @HystrixProperty(name = "metrics.rollingStats.numBuckets",value = "10"),

     //该属性用来设置对命令执行的延迟是否使用百分位来跟踪和计算。如果设置为false,那么所有的概要统计都将返回-1
     @HystrixProperty(name = "metrics.rollingPercentile.enabled",value = "false"),
                    
	 //该属性用来设置 百分位统计的滚动窗口的持续时间。单位为毫秒
     @HystrixProperty(name = "metrics.rollingPercentile.timeinMilliseconds",value = "60000"), 
                    
	 //该属性用来设置 百分位统计的滚动窗口中使用 "桶" 的数量
     @HystrixProperty(name = "metrics.rollingPercentile.numBuckets",value = "60000"), 
	 //该属性用来设置 在执行过程中，每个 "桶" 中保留的最大执行次数，如果在滚动时间窗内发生超过该设定值的执行次数，就从最初的位置重新开始写。例如，将该组值设置为100，滚动时间为10秒，若在10秒内一个 "桶" 中发生了500次执行，那么"桶"中值保留最后的100次执行的统计。另外，增加该值的大小将会增加内存的消耗，并增加排序百分位数所需的计算时间。
     @HystrixProperty(name = "metrics.rollingPercentile.bucketSize",value = "100"),                    
	 //该属性用来设置 采集影响熔断器状态的健康快照（请求的成功、错误百分比）的间隔风带时间
     @HystrixProperty(name = "metrics.healthSnapshot.intervalinMilliseconds",value = "500"), 
	 
	 //该属性用来设置 是否开启请求缓存
     @HystrixProperty(name = "requestCache.enabled",value = "true"),         	  // HystrixCommand 的执行和事件是否打印日志到 HystrixRequestLog 中
     @HystrixProperty(name = "requestLog.enabled",value = "true") 
     			},
     			threadPoolProperties = {
	 //该属性用来设置 执行命令线程池的核心线程数，该值是命令执行的最大并发量，默认10
     @HystrixProperty(name = "coreSize",value = "10"), 
	 //该属性用来设置 线程池的最大队列大小，当设置为-1时，线程池将使用 SynchronousQueue 实现的队列(阻塞队列)，否则将使用 LinkedBlockingQueue 实现的队列
     @HystrixProperty(name = "maxQueueSize",value = "-1"), 
	//该参数用来 为队列设置 拒绝阈值，通过该参数，即使队列没有达到最大值也能拒绝请求，改参数主要是对 LinkedBlockingQueue 队列的补充，因为 LinkedBlockingQueue 队列不能动态修改它的对象大小，而通过该属性就可以调整拒绝请求的队列大小了。
     @HystrixProperty(name = "queueSizeRejectionThreshold",value = "5")                     
         
})
```



#### （2）参数配置

本小节参考《Hystrix使用说明，配置参数说明》一文，原文链接：http://blog.csdn.net/tongtong_use/article/details/78611225*

（一）**Command Properties**

以下属性控制HystrixCommand行为：

1、**Execution**

以下属性控制`HystrixCommand.run()`如何执行。

| **参数**                                            | **描述**                                                     | **默认值**                                                   |
| --------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| execution.isolation.strategy                        | 隔离策略，有THREAD和SEMAPHORE THREAD - 它在单独的线程上执行，并发请求受线程池中的线程数量的限制  SEMAPHORE - 它在调用线程上执行，并发请求受到信号量计数的限制 | 默认使用THREAD模式，以下几种场景可以使用SEMAPHORE模式： 只想控制并发度 外部的方法已经做了线程隔离 调用的是本地方法或者可靠度非常高、耗时特别小的方法（如medis） |
| execution.isolation.thread.timeoutInMilliseconds    | 超时时间                                                     | 默认值：1000 在THREAD模式下，达到超时时间，可以中断 在SEMAPHORE模式下，会等待执行完成后，再去判断是否超时 设置标准： 有retry，99meantime+avg meantime 没有retry，99.5meantime |
| execution.timeout.enabled                           | HystrixCommand.run（）执行是否应该有超时。                   | 默认值：true                                                 |
| execution.isolation.thread.interruptOnTimeout       | 在发生超时时是否应中断HystrixCommand.run（）执行。           | 默认值：true THREAD模式有效                                  |
| execution.isolation.thread.interruptOnCancel        | 当发生取消时，执行是否应该中断。                             | 默认值为false THREAD模式有效                                 |
| execution.isolation.semaphore.maxConcurrentRequests | 设置在使用时允许到HystrixCommand.run（）方法的最大请求数。   | 默认值：10 SEMAPHORE模式有效                                 |

 

2、**Fallback**

以下属性控制`HystrixCommand.getFallback()`如何执行。这些属性适用于`ExecutionIsolationStrategy.THREAD`和`ExecutionIsolationStrategy.SEMAPHORE`。

| **参数**                                           | **描述**                                                     | **默认值**                   |
| -------------------------------------------------- | ------------------------------------------------------------ | ---------------------------- |
| fallback.isolation.semaphore.maxConcurrentRequests | 设置从调用线程允许HystrixCommand.getFallback（）方法的最大请求数。 | SEMAPHORE模式有效 默认值：10 |
| fallback.enabled                                   | 确定在发生失败或拒绝时是否尝试调用HystrixCommand.getFallback（）。 | 默认值为true                 |

 

3、**Circuit Breaker**

断路器属性控制`HystrixCircuitBreaker`的行为。

| **参数**                                 | **描述**                                                     | **默认值**                                                   |
| ---------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| circuitBreaker.enabled                   | 确定断路器是否用于跟踪运行状况和短路请求（如果跳闸）。       | 默认值为true                                                 |
| circuitBreaker.requestVolumeThreshold    | 熔断触发的最小个数/10s                                       | 默认值：20                                                   |
| circuitBreaker.sleepWindowInMilliseconds | 熔断多少秒后去尝试请求                                       | 默认值：5000                                                 |
| circuitBreaker.errorThresholdPercentage  | 失败率达到多少百分比后熔断                                   | 默认值：50 主要根据依赖重要性进行调整                        |
| circuitBreaker.forceOpen                 | 属性如果为真，强制断路器进入打开（跳闸）状态，其中它将拒绝所有请求。 | 默认值为false 此属性优先于circuitBreaker.forceClosed         |
| circuitBreaker.forceClosed               | 该属性如果为真，则迫使断路器进入闭合状态，其中它将允许请求，而不考虑误差百分比。 | 默认值为false 如果是强依赖，应该设置为true circuitBreaker.forceOpen属性优先，因此如果forceOpen设置为true，此属性不执行任何操作。 |

 

4、**Metrics**

以下属性与从`HystrixCommand`和`HystrixObservableCommand`执行捕获指标有关。

| **参数**                                      | **描述**                                                     | **默认值**    |
| --------------------------------------------- | ------------------------------------------------------------ | ------------- |
| metrics.rollingStats.timeInMilliseconds       | 此属性设置统计滚动窗口的持续时间（以毫秒为单位）。对于断路器的使用和发布Hystrix保持多长时间的指标。 | 默认值：10000 |
| metrics.rollingStats.numBuckets               | 此属性设置rollingstatistical窗口划分的桶数。 以下必须为true - “metrics.rollingStats.timeInMilliseconds%metrics.rollingStats.numBuckets == 0” -否则将抛出异常。 | 默认值：10    |
| metrics.rollingPercentile.enabled             | 此属性指示是否应以百分位数跟踪和计算执行延迟。 如果禁用它们，则所有摘要统计信息（平均值，百分位数）都将返回-1。 | 默认值为true  |
| metrics.rollingPercentile.timeInMilliseconds  | 此属性设置滚动窗口的持续时间，其中保留执行时间以允许百分位数计算，以毫秒为单位。 | 默认值：60000 |
| metrics.rollingPercentile.numBuckets          | 此属性设置rollingPercentile窗口将划分的桶的数量。 以下内容必须为true - “metrics.rollingPercentile.timeInMilliseconds%metrics.rollingPercentile.numBuckets == 0” -否则将抛出异常。 | 默认值：6     |
| metrics.rollingPercentile.bucketSize          | 此属性设置每个存储桶保留的最大执行次数。如果在这段时间内发生更多的执行，它们将绕回并开始在桶的开始处重写。 | 默认值：100   |
| metrics.healthSnapshot.intervalInMilliseconds | 此属性设置在允许计算成功和错误百分比并影响断路器状态的健康快照之间等待的时间（以毫秒为单位）。 | 默认值：500   |

 

5、**Request Context**

这些属性涉及HystrixCommand使用的HystrixRequestContext功能。

| **参数**             | **描述**                                                     | **默认值**   |
| -------------------- | ------------------------------------------------------------ | ------------ |
| requestCache.enabled | HystrixCommand.getCacheKey（）是否应与HystrixRequestCache一起使用，以通过请求范围的缓存提供重复数据删除功能。 | 默认值为true |
| requestLog.enabled   | HystrixCommand执行和事件是否应记录到HystrixRequestLog。      | 默认值为true |

 

（二）**Collapser Properties**

下列属性控制HystrixCollapser行为。

| **参数**                 | **描述**                                                     | **默认值**        |
| ------------------------ | ------------------------------------------------------------ | ----------------- |
| maxRequestsInBatch       | 此属性设置在触发批处理执行之前批处理中允许的最大请求数。     | Integer.MAX_VALUE |
| timerDelayInMilliseconds | 此属性设置创建批处理后触发其执行的毫秒数。                   | 默认值：10        |
| requestCache.enabled     | 此属性指示是否为HystrixCollapser.execute（）和HystrixCollapser.queue（）调用启用请求高速缓存。 | 默认值：true      |

 （三）**ThreadPool Properties**

以下属性控制Hystrix命令在其上执行的线程池的行为。

大多数时候，默认值为10的线程会很好（通常可以做得更小）。

| **参数**                                | **描述**                                                     | **默认值**                                                   |
| --------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| coreSize                                | 线程池coreSize                                               | 默认值：10 设置标准：qps*99meantime+breathing room           |
| maximumSize                             | 此属性设置最大线程池大小。 这是在不开始拒绝HystrixCommands的情况下可以支持的最大并发数。 请注意，此设置仅在您还设置allowMaximumSizeToDivergeFromCoreSize时才会生效。 | 默认值：10                                                   |
| maxQueueSize                            | 请求等待队列                                                 | 默认值：-1 如果使用正数，队列将从SynchronizeQueue改为LinkedBlockingQueue |
| queueSizeRejectionThreshold             | 此属性设置队列大小拒绝阈值 - 即使未达到maxQueueSize也将发生拒绝的人为最大队列大小。 此属性存在，因为BlockingQueue的maxQueueSize不能动态更改，我们希望允许您动态更改影响拒绝的队列大小。 | 默认值：5 注意：如果maxQueueSize == -1，则此属性不适用。     |
| keepAliveTimeMinutes                    | 此属性设置保持活动时间，以分钟为单位。                       | 默认值：1                                                    |
| allowMaximumSizeToDivergeFromCoreSize   | 此属性允许maximumSize的配置生效。 那么该值可以等于或高于coreSize。 设置coreSize <maximumSize会创建一个线程池，该线程池可以支持maximumSize并发，但在相对不活动期间将向系统返回线程。 （以keepAliveTimeInMinutes为准） | 默认值：false                                                |
| metrics.rollingStats.timeInMilliseconds | 此属性设置statistical rolling窗口的持续时间（以毫秒为单位）。 这是为线程池保留多长时间。 | 默认值：10000                                                |
| metrics.rollingStats.numBuckets         | 此属性设置滚动统计窗口划分的桶数。  注意：以下必须为true - “metrics.rollingStats.timeInMilliseconds%metrics.rollingStats.numBuckets == 0” -否则将引发异常。 | 默认值：10                                                   |



（四）**其他**

| **参数**   | **描述**                             | **默认值**                          |
| ---------- | ------------------------------------ | ----------------------------------- |
| groupKey   | 表示所属的group，一个group共用线程池 | 默认值：getClass().getSimpleName(); |
| commandKey |                                      | 默认值：当前执行方法名              |

#### （3）熔断状态

  熔断器的状态机：

![熔断器的状态](/images/springcloud/hystrix-circuit-break.jpg)

![熔断器的状态及机制](/images/springcloud/hystrix-circuit-status.png)

- Closed：熔断器关闭状态，熔断器关闭不会对服务进行熔断操作。因调用失败次数积累，到了阈值（或一定比例）则启动熔断机制，熔断器进入打开状态；
- Open：熔断器打开状态。此时对下游的调用都内部直接返回错误，不走网络，但设计了一个时钟选项，默认的时钟达到了一定时间（这个时间一般设置成平均故障处理时间，也就是MTTR），到了这个时间，进入半熔断状态； 
- Half-Open：半熔断状态，允许部分/定量的服务请求，如果调用都成功（或一定比例）则认为恢复了，关闭熔断器，否则认为还没好，又回到熔断器打开状态； 

（3）熔断器流程图

![熔断器流程图](/images/springcloud/hystrix-circuit-break-work.jpg)

### 3、服务限流

服务限流：服务器面对秒杀等高并发操作，防止瞬时流量过大使服务和数据崩溃而进行的一种限制请求正常通行的机制。

 Hystrix 的限流 我的理解只是作为一种防御高并发冲击，防止扇出链路超时异常引发级联故障，完全不同于Sentinel 限流里的流控机制。Hystrix的限流就是通过熔断和服务降级来实现的。是一种防御式操作，降级的快速失败，紧紧是将微服务进行保护式隔离，本质上并不能真正解决高流量请求问题。

如果理解有误或不同观点，欢迎指正，后续再补充。



### 4、服务监控

除了隔离服务、Hystrix 还可以进行准实时监控Hystrix Dashboard 。

Hystrix 会持续地记录所有通过Hystrix发起的请求的执行信息，并以统计报表和图形的形式展示，包括每秒执行多少请求、多少成功、多少失败、等等。

Netflix 通过 hystrix-mertics-event-stream 项目实现了对以上指标的监控。

Spring Cloud 也提供了 Hystrix DashBoard 的整合，将监控内容转换为可视化web界面。

#### 4.1 服务监控使用

作为独立的监控程序，需要创建新的微服务工程`MS-Hystrix-Dashboard`。

（1）POM引入依赖

```java
        <!--新增hystrix dashboard-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

（2）配置端口和服务名

```yam
server:
  port: 9999
  
spring:
  application:
    name: MS-Hystrix-Dashboard  
```

（3）启用HystrixDashboard

使用 `@EnableHystrixDashboard` 注解开启即可。

```java
package com.xiaocai.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.hystrix.dashboard.EnableHystrixDashboard;

@SpringBootApplication
@EnableHystrixDashboard
public class HystrixDashboardMain9001 {
    public static void main(String[] args) {
        SpringApplication.run(HystrixDashboardMain9001.class,args);
    }
}
```

（4）所以被监控微服务需要引入监控模块

```xml
 <!-- springboot 监控模块 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

（5）注意事宜

如果打开报错，尝试新版本Hystrix需要在主启动类MainAppHystrix8001中指定监控路径

```java
@Bean
public ServletRegistrationBean getServlet(){
    HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
    ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
    registrationBean.setLoadOnStartup(1);
    registrationBean.addUrlMappings("/hystrix.stream");
    registrationBean.setName("HystrixMetricsStreamServlet");
    return registrationBean;
}
```

（6）开始监控

打开Hystrix 主页面：http://localhost:9999/hystrix

输入想要监控的路径：http://localhost:6001/hystrix.stream

进行服务访问测试。查看监控页面变化。

#### 4.2 服务监控阅读

（1）7色。7中不同颜色的线均代表各自含义。

（2）1圆。实心圆，有两种含义。

它的颜色变化代表微服务实例的健康程度。绿色 < 黄色 < 橙色 < 红色

它的大小变化会根据实例的请求流量发生变化，流量越大，圆越大。

所以，可以根据颜色和大小快速发现故障实例和高流量实例。

（3）1线。曲线，用来记录2分钟内的流量相对变化。

可以通过曲线来观察流量的上升和下降趋势。

示例图：

![Hystrix Dashboard 监控示例图1](/images/springcloud/hystrix-dashboard-01.jpg)

![Hystrix Dashboard 监控示例图2](/images/springcloud/hystrix-dashboard-02.jpg)
完整示例图：

![Hystrix Dashboard 监控示例图2](/images/springcloud/hystrix-dashboard-03.jpg)

