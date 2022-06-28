---
title: Spring-Cloud-Exception-libs
tags:
  - SpringCloud-Exeptions
categories: SpringCloud
summary: Spring-Cloud-Exception-libs。
keywords: >-
  java, SpringCloud Exception,ClientException,FeignException$NotFound:
  [404],Method has too many Body parameters,
author: Small-Rose /张小菜
abbrlink: '1618e871'
date: 2020-11-13 13:00:00
---

## Spring-Cloud-Exception-libs

会持续更新....

最后更新日期：2020-11-13

**说明**

作为一个`Spring-Cloud-Exception`异常库，目前操作的都是以`SpringBoot2.2.2`和 `Spring cloud Hoxton.SR8` 版本为基准。如果不是该版本，会特别说明。

```
<spring-boot.version>2.2.2.RELEASE</spring-boot.version>
<spring-cloud.version>Hoxton.SR8</spring-cloud.version>
```

### 1、 ClientException异常：

com.netflix.client.ClientException: Load balancer does not have available server for client xxxx 

解决思路：

1、检查配置，该异常一般涉及组件远程调用组件Feign、注册中心组件。

2、开启ribbon配合Eureka使用

```
ribbon.eureka.enable: true
```

3、添加 fetch-registry = true 的选项，允许客户端获取注册信息，我可以点进入看默认构造方法里其实已经是true了（我的版本是），尽管我这样设置之后确实没有该异常了，但是问题并没有真正解决，先记录该问题。后续学习深层次的原理再去探究。

```
eureka:
  client: #客户端注册进eureka服务列表内
    service-url:
      defaultZone: http://localhost:9900/eureka
    fetch-registry: true
```



### 2、 FeignException$NotFound: [404]

```
feign.FeignException$NotFound: [404] during [GET] to [http://server-store/v1/store/update/1001/] [StoreClient#updateStore(int,Integer)]: [{"timestamp":"2020-11-11T14:04:33.075+0000","status":404,"error":"Not Found","message":"No message available","path":"/v1/store/update/1001/"}]
	at feign.FeignException.clientErrorStatus(FeignException.java:201) ~[feign-core-10.10.1.jar:na]
```

如果你出现了Feign始终无法调用下游服务一直进你的`fallback`，或你的调用出现该异常，可以参考以下经验，如果以下方式不能解决你的问题，欢迎邮件交流 <a href="http://mail.qq.com/cgi-bin/qm_share?t=qm_mailme&email=ssHf097en8Ddwdfyw8Oc0d3f"> 给我写信 </a>

该异常是 feign-core 的包抛出，显然是和feign组件相关。

一般不会出现该问题，因为你有对应的`xxxfallBack`操作，无法展示出此操作，只是一直无法调用下游服务。如果想重现该异常，请先去掉 `fallback = StoreFallBack.class ` 进行测试，如果出现，尝试将 `@GetMapping` 换成 `@RequestMapping`再尝试。早起版好像是支持`@FeignClient` 和`@GetMapping`

的组合。

```
@Component
@FeignClient(value = "SERVER-STORE", fallback = StoreFallBack.class )
public interface StoreClient {

    @RequestMapping(value = "/v1/store/update/{prodId}/{number}" ,method = RequestMethod.GET)
    public boolean updateStore(@PathVariable("prodId") Integer prodId, @PathVariable("number") Integer number);

}
```

### 3、 IllegalStateException: Method has too many Body parameters

```
Caused by: java.lang.IllegalStateException: Method has too many Body parameters: public  xxx 
```

异常原因：当使用Feign时，如果发送的是get请求，那么需要在请求参数前加上@RequestParam注解修饰，Controller里面可以不加该注解修饰。

正确写法：

```java
    @RequestMapping(value = "/v1/account/decrease" ,method = RequestMethod.POST)
    public boolean decreaseAccount(@RequestParam("userId") int userId, @RequestParam("money") double money);

	@RequestMapping
	public int dosomethings(@RequestBody final OrderBean p,@RequestParam("userId") String userId,@RequestParam("money") Double money);

```

### 4、 Could not register branch into global session xid = status = Rollback

- 异常：Could not register branch into global session xid = status =  Rollbacked（还有Rollbacking、AsyncCommitting等等二阶段状态） while expecting Begin
- 描述：分支事务注册时，全局事务状态需是一阶段状态begin，非begin不允许注册。属于seata框架层面正常的处理，用户可以从自身业务层面解决。
- 出现场景（可继续补充）

```
  1. 分支事务是异步，全局事务无法感知它的执行进度，全局事务已进入二阶段，该异步分支才来注册
  2. 服务a rpc 服务b超时（dubbo、feign等默认1秒超时），a上抛异常给tm，tm通知tc回滚，但是b还是收到了请求（网络延迟或rpc框架重试），然后去tc注册时发现全局事务已在回滚
  3. tc感知全局事务超时(@GlobalTransactional(timeoutMills = 默认60秒))，主动变更状态并通知各分支事务回滚，此时有新的分支事务来注册
```


### 5、 Maven......xml message：前言中不允许有内容

出现这个异常很有可能是因你修改了字符集导致的。

找到对应的xml文件，使用可以修改字符集编码的软件打开，比如UE， Notepad++等，打开之后修改字符集编码，设置成和工程一致的编码即可。


### 6、 Caused by: java.sql.SQLException: No timezone mapping entry for 'GMT+'

 将 serverTimezone=GMT+8 改为serverTimeZone=UGMT+8


### 7、 MongoSocketOpenException: Exception opening socket 

springboot 自动装配； MongDb连接，如果使用请配置相关的数据源连接信息，如果不使用需要排除之：

```java
@SpringBootApplication(exclude = MongoAutoConfiguration.class)
@EnableDiscoveryClient
@EnableFeignClients
public class HmilyOrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(HmilyOrderApplication.class, args);
    }

}
```

### 8、 .kryo.KryoException: Unable to find class: xxx

异常现象：

```
00:00  WARN: [kryo] Unable to load class com.xiaocai.distran.hmilyorder.service.OrderService with kryo's ClassLoader. Retrying with current..
2020-11-14 20:49:47.984 ERROR 26396 --- [self-recovery-1] .s.HmilyTransactionSelfRecoveryScheduled : hmily scheduled transaction log is error:

com.esotericsoftware.kryo.KryoException: Unable to find class: com.xiaocai.distran.hmilyorder.service.OrderService
Serialization trace:
targetClass (org.dromara.hmily.repository.spi.entity.HmilyInvocation)
  at com.esotericsoftware.kryo.util.DefaultClassResolver.readName(DefaultClassResolver.java:160) ~[kryo-shaded-4.0.0.jar:na]
  at com.esotericsoftware.kryo.util.DefaultClassResolver.readClass(DefaultClassResolver.java:133) ~[kryo-shaded-4.0.0.jar:na]
  at com.esotericsoftware.kryo.Kryo.readClass(Kryo.java:693) ~[kryo-shaded-4.0.0.jar:na]
  at com.esotericsoftware.kryo.serializers.DefaultSerializers$ClassSerializer.read(DefaultSerializers.java:329) ~[kryo-shaded-4.0.0.jar:na]
  at com.esotericsoftware.kryo.serializers.DefaultSerializers$ClassSerializer.read(DefaultSerializers.java:317) ~[kryo-shaded-4.0.0.jar:na]
  at com.esotericsoftware.kryo.Kryo.readObjectOrNull(Kryo.java:782) ~[kryo-shaded-4.0.0.jar:na]
  at com.esotericsoftware.kryo.serializers.ObjectField.read(ObjectField.java:132) ~[kryo-shaded-4.0.0.jar:na]
  at com.esotericsoftware.kryo.serializers.FieldSerializer.read(FieldSerializer.java:540) ~[kryo-shaded-4.0.0.jar:na]
  at com.esotericsoftware.kryo.Kryo.readObject(Kryo.java:709) ~[kryo-shaded-4.0.0.jar:na]
  at org.dromara.hmily.serializer.kryo.KryoSerializer.deSerialize(KryoSerializer.java:63) ~[hmily-serializer-kryo-2.1.1.jar:na]
  at org.dromara.hmily.repository.database.manager.AbstractHmilyDatabase.buildHmilyParticipantByResultMap(AbstractHmilyDatabase.java:543) ~[hmily-repository-database-manager-2.1.1.jar:na]
  at java.util.stream.ReferencePipeline$3$1.accept(ReferencePipeline.java:193) ~[na:1.8.0_191]
  at java.util.stream.ReferencePipeline$2$1.accept(ReferencePipeline.java:175) ~[na:1.8.0_191]
  at java.util.ArrayList$ArrayListSpliterator.forEachRemaining(ArrayList.java:1382) ~[na:1.8.0_191]
  at java.util.stream.AbstractPipeline.copyInto(AbstractPipeline.java:481) ~[na:1.8.0_191]
  at java.util.stream.AbstractPipeline.wrapAndCopyInto(AbstractPipeline.java:471) ~[na:1.8.0_191]
  at java.util.stream.ReduceOps$ReduceOp.evaluateSequential(ReduceOps.java:708) ~[na:1.8.0_191]
  at java.util.stream.AbstractPipeline.evaluate(AbstractPipeline.java:234) ~[na:1.8.0_191]
  at java.util.stream.ReferencePipeline.collect(ReferencePipeline.java:499) ~[na:1.8.0_191]
  at org.dromara.hmily.repository.database.manager.AbstractHmilyDatabase.listHmilyParticipant(AbstractHmilyDatabase.java:332) ~[hmily-repository-database-manager-2.1.1.jar:na]
  at org.dromara.hmily.core.schedule.HmilyTransactionSelfRecoveryScheduled.lambda$selfTccRecovery$2(HmilyTransactionSelfRecoveryScheduled.java:101) ~[hmily-core-2.1.1.jar:na]
  at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511) ~[na:1.8.0_191]
  at java.util.concurrent.FutureTask.runAndReset(FutureTask.java:308) ~[na:1.8.0_191]
  at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$301(ScheduledThreadPoolExecutor.java:180) ~[na:1.8.0_191]
  at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:294) ~[na:1.8.0_191]
  at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) ~[na:1.8.0_191]
  at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) ~[na:1.8.0_191]
  at java.lang.Thread.run(Thread.java:748) ~[na:1.8.0_191]
Caused by: java.lang.ClassNotFoundException: com.xiaocai.distran.hmilyorder.service.OrderService
  at java.net.URLClassLoader.findClass(URLClassLoader.java:382) ~[na:1.8.0_191]
  at java.lang.ClassLoader.loadClass(ClassLoader.java:424) ~[na:1.8.0_191]
  at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:349) ~[na:1.8.0_191]
  at java.lang.ClassLoader.loadClass(ClassLoader.java:357) ~[na:1.8.0_191]
  at java.lang.Class.forName0(Native Method) ~[na:1.8.0_191]
  at java.lang.Class.forName(Class.java:348) ~[na:1.8.0_191]
  at com.esotericsoftware.kryo.util.DefaultClassResolver.readName(DefaultClassResolver.java:154) ~[kryo-shaded-4.0.0.jar:na]
  ... 27 common frames omitted
```
