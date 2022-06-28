---
title: RocketMQ 入门与实操
tags:
  - RocketMQ
categories: MQ
summary: RocketMQ 入门与实操。
keywords: >-
  java,spring boot,spring boot RocketMQ , RocketMQ,SpringBoot
  RocketMQ,SpringBoot  RocketMQ 配置
author: Small-Rose / 张小菜
abbrlink: c6651ce9
date: 2020-11-30 22:00:00
---

## RocketMQ

## 一、基本特点

RocketMQ具有以下特点：

- RocketMQ是一个队列模型的消息中间件，具有高性能、高可靠、高实时、分布式特点。
- Producer、Consumer、队列都可以分布式。
- Producer向一些队列轮流发送消息，队列集合称为Topic，Consumer如果做广播消费，则一个consumer实例消费这个Topic对应的所有队列，如果做集群消费，则多个Consumer实例平均消费这个topic对应的队列集合。
- 能够保证严格的消息顺序
- 提供丰富的消息拉取模式
- 高效的订阅者水平扩展能力
- 实时的消息订阅机制
- 亿级消息堆积能力
- 依赖较少

## 二、四大核心组件

RocketMQ 核心的四大组件：Name Server、Broker、Producer、Consumer ，每个组件都可以部署成集群模式进行水平扩展。

### 1. Name Server

Name Server 是一个几乎无状态节点，可集群部署，节点之间无任何信息同步。

### 2. Broker

消息服务器（Broker）是消息存储中心，主要作用是接收来自 Producer 的消息并存储， Consumer 从这里取得消息。 它还存储与消息相关的元数据，包括用户组、消费进度偏移量、队列信息等。

从逻辑加个上看 Broker 有 Master 和 Slave 两种类型，Master 既可以写又可以读，Slave 不可以写只可以读。

从物理结构上看 Broker 的集群部署方式有四种：单 Master 、多 Master 、多 Master 多 Slave（同步刷盘）、多 Master多 Slave（异步刷盘）。

每个Broker与Name Server集群中的所有节点建立长连接，定时(每隔30s)注册Topic信息到所有Name Server。Name Server定时(每隔10s)扫描所有存活broker的连接，如果Name Server超过2分钟没有收到心跳，则Name Server断开与Broker的连接。

### 3. Producer

（1）生产者

生产者（Producer）负责产生消息，生产者向消息服务器发送由业务应用程序系统生成的消息。 

RocketMQ 提供了三种方式发送消息：同步、异步和单向。

- 同步发送

同步发送指消息发送方发出数据后会在收到接收方发回响应之后才发下一个数据包。一般用于重要通知消息，例如重要通知邮件、营销短信。

- 异步发送

异步发送指发送方发出数据后，不等接收方发回响应，接着发送下个数据包，一般用于可能链路耗时较长而对响应时间敏感的业务场景，例如用户视频上传后通知启动转码服务。

- 单向发送

单向发送是指只负责发送消息而不等待服务器回应且没有回调函数触发，适用于某些耗时非常短但对可靠性要求并不高的场景，例如日志收集。

（2）生产者组

生产者组（Producer Group）是某一类 Producer 的集合，这类 Producer 通常发送一类消息并且发送逻辑一致，所以将这些 Producer 分组在一起。 从部署结构上看生产者通过 Producer Group 的名字来标记自己是一个集群。

### 4. Consumer

（1）消费者

消费者（Consumer）负责消费消息，消费者从消息服务器拉取信息并将其输入用户应用程序。
站在用户应用的角度消费者有两种类型：拉取型消费者、推送型消费者。

- 拉取型消费者

拉取型消费者（Pull Consumer）主动从消息服务器拉取信息，只要批量拉取到消息，用户应用就会启动消费过程，所以 Pull 称为主动消费型。

- 推送型消费者

推送型消费者（Push Consumer）封装了消息的拉取、消费进度和其他的内部维护工作，将消息到达时执行的回调接口留给用户应用程序来实现。所以 Push 称为被动消费类型，但从实现上看还是从消息服务器中拉取消息，不同于 Pull 的是 Push 首先要注册消费监听器，当监听器处触发后才开始消费消息。

（2）消费者组

消费者组（Consumer Group）是某一类 Consumer 的集合名称，这类 Consumer 通常消费同一类消息并且消费逻辑一致，所以将这些 Consumer 分组在一起。 消费者组与生产者组类似，都是将相同角色的分组在一起并命名，分组是个很精妙的概念设计，RocketMQ 正是通过这种分组机制，实现了天然的消息负载均衡。 消费消息时通过 Consumer Group 实现了将消息分发到多个消费者服务器实例，比如某个 Topic 有18条消息，其中一个 Consumer Group 有3个实例（3个进程或3台机器），那么每个实例将均摊6条消息，这也意味着我们可以很方便的通过加机器来实现水平扩展。
消息服务器




## 三、RocketMQ 解压与安装


### Linux 解压

```bash
unzip rocketmq-all-4.7.1-source-release.zip
```

### windows 解压安装

windows 环境解压之后，启动可能会报错

调整运行参数，在`runbroker.cmd` 或 `runbroker.sh` 中根据实际内存调整：

```bash
set "JAVA_OPT=%JAVA_OPT% -server -Xms1g -Xmx2g -Xmn800m "
```


先配置环境变量：

变量名：`ROCKETMQ_HOME`
变量值：`E:\local\rocketmq-all-4.7.1-bin-release`

一来是启动脚本需要使用这个变量，二来配置一下环境变量，不用每次到bin目录执行。

如果还是启动报错，因为如果路径中有空格是不能识别。修改`runbroker.cmd` 文件。

修改有三处：

- %JAVA_HOME% 改成实际路径。
- mqlogs\mq_gc.log 路径补成实际路径
- "%CLASSPATH%" 加上双引号

```bash

if not exist "%JAVA_HOME%\bin\java.exe" echo Please set the JAVA_HOME variable in your environment, We need java(x64)! & EXIT /B 1
set "JAVA=C:\Program Files\Java\jdk1.8.0_212\bin\java.exe"

setlocal

set BASE_DIR=%~dp0
set BASE_DIR=%BASE_DIR:~0,-1%
for %%d in (%BASE_DIR%) do set BASE_DIR=%%~dpd

set CLASSPATH=.;%BASE_DIR%conf;%CLASSPATH%

rem ===========================================================================================
rem  JVM Configuration
rem ===========================================================================================
set "JAVA_OPT=%JAVA_OPT% -server -Xms1g -Xmx2g -Xmn800m "
set "JAVA_OPT=%JAVA_OPT% -XX:+UseG1GC -XX:G1HeapRegionSize=16m -XX:G1ReservePercent=25 -XX:InitiatingHeapOccupancyPercent=30 -XX:SoftRefLRUPolicyMSPerMB=0 -XX:SurvivorRatio=8"
set "JAVA_OPT=%JAVA_OPT% -verbose:gc -Xloggc:E:\local\rocketmq-all-4.7.1-bin-release\mqlogs\mq_gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCApplicationStoppedTime -XX:+PrintAdaptiveSizePolicy"
set "JAVA_OPT=%JAVA_OPT% -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=30m"
set "JAVA_OPT=%JAVA_OPT% -XX:-OmitStackTraceInFastThrow"
set "JAVA_OPT=%JAVA_OPT% -XX:+AlwaysPreTouch"
set "JAVA_OPT=%JAVA_OPT% -XX:MaxDirectMemorySize=15g"
set "JAVA_OPT=%JAVA_OPT% -XX:-UseLargePages -XX:-UseBiasedLocking"
set "JAVA_OPT=%JAVA_OPT% -Djava.ext.dirs=%BASE_DIR%lib"
set "JAVA_OPT=%JAVA_OPT% -cp "%CLASSPATH%" "

"%JAVA%" %JAVA_OPT% %*
```

## 四、RocketMQ 启动操作

### Windows 版：

启动Name Server

```bash
start mqnamesrv.cmd 
```

查看日志：

```
${user.home}/logs/rocketmqlogs/broker.log
```

启动broker：

```bash
start mqbroker.cmd -n 127.0.0.1:9876 autoCreateTopicEnable=true
```

查看 broker 是否注册成功


```bash
start mqadmin.cmd  clusterList -n 127.0.0.1:9876
```


### Linux 版：

启动Name Server

```bash
nohup ./bin/mqnamesrv &
```

查看日志：


```bash
tail -F ~/logs/rocketmqlogs/namesrv.log
```

启动broker


```bash
nohup sh bin/mqbroker -n localhost:9876 & 
nohup sh mqbroker -n 127.0.0.1:9876 autoCreateTopicEnable=true &
```

查看 broker 是否注册成功

```bash
sh mqadmin  clusterList -n 127.0.0.1:9876
```

也可以直接查看日志文件

```bash
tail -f ~/logs/rocketmqlogs/broker.log
```

## 五、RocketMQ web 控制台

可以搞个web控制台：

下载之后打包成jar

打包之前，可以根据需要修改相关信息，这样在启动的时候不需要带一些参数，如果不放心也可以不修改，在启动的时候设置参数。

常见的修改：

- `rocketmq.config.namesrvAddr`
- `server.port`

```properties
server.contextPath=mq
server.port=9999
#spring.application.index=true
spring.application.name=rocketmq-console
spring.http.encoding.charset=UTF-8
spring.http.encoding.enabled=true
spring.http.encoding.force=true
logging.config=classpath:logback.xml
#if this value is empty,use env value rocketmq.config.namesrvAddr  NAMESRV_ADDR | now, you can set it in ops page.default localhost:9876
rocketmq.config.namesrvAddr=127.0.0.1:9876
```

然后执行打包命令：

```bash
mvn clean package -Dmaven.test.skip=true
```

启动控制台:

```bash
java -jar rocketmq-console-ng-2.0.0.jar 
```

如果在本地机器运行，可以指定一下运行参数：

```bash
java -jar rocketmq-console-ng-2.0.0.jar -Server -Xms512m -Xmx512m
```

如果没有自己修改端口或者Nameserver地址的配置文件，也可以通过参数的方式设置：

```bash
java -jar rocketmq-console-ng-2.0.0.jar --server.port=9999 --rocketmq.config.namesrvAddr=127.0.0.1:9876
```

访问：

`http://127.0.0.1:9999/mq/`

可以在集群里看到broker，主题里免看到许多默认的主题。

官方文档或Demo：`http://rocketmq.apache.org/docs/quick-start/`

官方Git文档：https://github.com/apache/rocketmq/tree/master/docs/cn

## 五、工程使用实例

### 1. 引入依赖

生产者和消费者的引入依赖文件：

```xml
        <!-- rocketmq -->
        <dependency>
            <groupId>org.apache.rocketmq</groupId>
            <artifactId>rocketmq-client</artifactId>
            <version>4.5.2</version>
        </dependency>
```


### 2. 生产者端



（1）生产者添加配置：

```properties
spring.application.name=a
server.port=8000
 
# nacos配置地址
#nacos.config.server-addr=127.0.0.1:8848
# nacos注册地址
#nacos.discovery.server-addr=127.0.0.1:8848
 
spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
spring.jackson.time-zone=GMT+8
 
# 是否开启自动配置
rocketmq.producer.isOnOff=on
# 发送同一类消息设置为同一个group，保证唯一默认不需要设置，rocketmq会使用ip@pid（pid代表jvm名字）作为唯一标识
rocketmq.producer.groupName=${spring.application.name}
# mq的nameserver地址
rocketmq.producer.namesrvAddr=127.0.0.1:9876
# 消息最大长度 默认 1024 * 4 (4M)
rocketmq.producer.maxMessageSize = 4096
# 发送消息超时时间，默认 3000
rocketmq.producer.sendMsgTimeOut=3000
# 发送消息失败重试次数，默认2
rocketmq.producer.retryTimesWhenSendFailed=2
```
对应的Config:

```java
package com.xiaocai.rocketmq.producer.config;

import lombok.Data;
import lombok.extern.slf4j.Slf4j;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Data
@Slf4j
@Configuration
@ConfigurationProperties(prefix = "rocketmq.producer")
public class MQProducerConfig {

    // name server 组名
    private String groupName;
    // name server 地址
    private String namesrvAddr;
    // 消息最大值
    private Integer maxMessageSize;
    // 消息发送超时时间
    private Integer sendMsgTimeOut;
    // 失败重试次数
    private Integer retryTimesWhenSendFailed;
    /**
     * mq 生成者配置
     * @return
     * @throws MQClientException
     */
    @Bean
    @ConditionalOnProperty(prefix = "rocketmq.producer", value = "isOnOff", havingValue = "on")
    public DefaultMQProducer defaultProducer() throws MQClientException {
        log.info("defaultProducer creating ----------");
        DefaultMQProducer producer = new DefaultMQProducer(groupName);
        producer.setNamesrvAddr(namesrvAddr);
        producer.setVipChannelEnabled(false);
        producer.setMaxMessageSize(maxMessageSize);
        producer.setSendMsgTimeout(sendMsgTimeOut);
        producer.setRetryTimesWhenSendAsyncFailed(retryTimesWhenSendFailed);
        producer.start();
        log.info("rocketmq producer server init success  ----");
        return producer;
    }
}


```

（2）生产者发消息：

```java
package com.xiaocai.rocketmq.producer.controller;

import com.google.common.collect.Lists;
import com.xiaocai.rocketmq.producer.base.Result;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;
import org.apache.rocketmq.client.exception.MQBrokerException;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.exception.RemotingException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@Slf4j
@RestController
public class SendMsgController {

    @Autowired
    DefaultMQProducer defaultMQProducer;

    @RequestMapping("/send")
    @ResponseBody
    public Result send(String msg) throws InterruptedException, RemotingException, MQClientException, MQBrokerException {
        if (StringUtils.isEmpty(msg)) {
            return new Result().success();
        }
        log.info("发送MQ消息内容：" + msg);
        Message sendMsg = new Message("TestTopic", "TestTag", msg.getBytes());
        // 默认3秒超时
        SendResult sendResult = defaultMQProducer.send(sendMsg);
        log.info("消息发送响应：" + sendResult.toString());
        return new Result().success(sendResult);
    }

    @RequestMapping("/sends")
    @ResponseBody
    public Result sendMore() throws InterruptedException, RemotingException, MQClientException, MQBrokerException {
        List<SendResult> results = Lists.newArrayList();
        for (int i = 0; i < 6; i++) {

            String msg = "my message no " + i ;
            log.info("发送MQ消息内容：" +  msg);
            Message sendMsg = new Message("TestTopic", "TestTag", msg.getBytes());
            // 默认3秒超时
            SendResult sendResult = defaultMQProducer.send(sendMsg);
            log.info("消息发送响应：" + sendResult.toString());

            results.add(sendResult);
        }
        defaultMQProducer.shutdown();

        return new Result().success(results);
    }

    @RequestMapping("/send2")
    @ResponseBody
    public Result send2() throws InterruptedException, RemotingException, MQClientException, MQBrokerException {
        List<SendResult> results = Lists.newArrayList();
        for (int i = 0; i < 6; i++) {

            String msg = "my message no " + i ;
            log.info("发送MQ消息内容：" +  msg);

            Message sendMsg = new Message();
            sendMsg.setTopic("TestTopic");
            sendMsg.setTags("TestTag");
            sendMsg.setBody(msg.getBytes());
            // 默认3秒超时
            SendResult sendResult = defaultMQProducer.send(sendMsg);
            log.info("消息发送响应：" + sendResult.toString());

            results.add(sendResult);
        }
        defaultMQProducer.shutdown();

        return new Result().success(results);
    }


}
```


### 3. 消费者端

（1）消费者添加配置：

```properties
spring.application.name=mq-consumer
server.port=8802

# 是否开启自动配置
rocketmq.consumer.isOnOff=on
# 发送同一类消息设置为同一个group，保证唯一默认不需要设置，rocketmq会使用ip@pid（pid代表jvm名字）作为唯一标识
rocketmq.consumer.groupName=${spring.application.name}
# mq的nameserver地址
rocketmq.consumer.namesrvAddr=127.0.0.1:9876
# 消费者订阅的主题topic和tags（*标识订阅该主题下所有的tags），格式: topic~tag1||tag2||tags3;
rocketmq.consumer.topics=TestTopic~TestTag;

# 消费者线程数据量
rocketmq.consumer.consumeThreadMin=5
rocketmq.consumer.consumeThreadMax=32
# 设置一次消费信心的条数，默认1
rocketmq.consumer.consumeMessageBatchMaxSize=1

spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
spring.jackson.time-zone=GMT+8
```

对应的Config:

```java
package com.xiaocai.rocketmq.consumer.config;

import com.xiaocai.rocketmq.consumer.consumer.MQConsumerMsgListenerProcessor;
import lombok.Data;
import lombok.extern.slf4j.Slf4j;
import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.common.consumer.ConsumeFromWhere;
import org.apache.rocketmq.common.protocol.heartbeat.MessageModel;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;


@Slf4j
@Data
@Configuration
@ConfigurationProperties(prefix = "rocketmq.consumer")
public class MQConsumerConfig {


    private String groupName;
    private String namesrvAddr;
    private String topics;
    // 消费者线程数据量
    private Integer consumeThreadMin;
    private Integer consumeThreadMax;
    private Integer consumeMessageBatchMaxSize;

    @Autowired
    private MQConsumerMsgListenerProcessor consumeMsgListenerProcessor;
    /**
     * mq 消费者配置
     * @return
     * @throws MQClientException
     */
    @Bean
    @ConditionalOnProperty(prefix = "rocketmq.consumer", value = "isOnOff", havingValue = "on")
    public DefaultMQPushConsumer defaultConsumer() throws MQClientException {
        log.info("defaultConsumer creating -------------");
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer(groupName);
        consumer.setNamesrvAddr(namesrvAddr);
        consumer.setConsumeThreadMin(consumeThreadMin);
        consumer.setConsumeThreadMax(consumeThreadMax);
        consumer.setConsumeMessageBatchMaxSize(consumeMessageBatchMaxSize);
        // 设置监听
        consumer.registerMessageListener(consumeMsgListenerProcessor);

        /**
         * 设置consumer第一次启动是从队列头部开始还是队列尾部开始
         * 如果不是第一次启动，那么按照上次消费的位置继续消费
         */
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_LAST_OFFSET);
        /**
         * 设置消费模型，集群还是广播，默认为集群
         */
        consumer.setMessageModel(MessageModel.CLUSTERING);

        try {
            // 设置该消费者订阅的主题和tag，如果订阅该主题下的所有tag，则使用*,
            String[] topicArr = topics.split(";");
            for (String tag : topicArr) {
                String[] tagArr = tag.split("~");
                log.info("consumer 订阅Topic :{} , 订阅Tags :{}", tagArr[0], tagArr[1]);
                consumer.subscribe(tagArr[0], tagArr[1]);
            }
            consumer.start();
            log.info("consumer 创建成功 groupName={}, topics={}, namesrvAddr={}",groupName,topics,namesrvAddr);
        } catch (MQClientException e) {
            log.error("consumer 创建失败!");
        }
        return consumer;
    }
}

```


（2）消费者接收消息：

```java
package com.xiaocai.rocketmq.consumer.consumer;

import lombok.extern.slf4j.Slf4j;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.common.message.MessageExt;
import org.springframework.stereotype.Component;
import org.springframework.util.CollectionUtils;

import java.util.List;

@Slf4j
@Component
public class MQConsumerMsgListenerProcessor implements MessageListenerConcurrently {
    @Override
    public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
        if (CollectionUtils.isEmpty(list)) {
            log.info("MQ接收消息为空，直接返回成功");
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        }
        MessageExt messageExt = list.get(0);
        log.info("MQ接收到的消息为：" + messageExt.toString());
        try {
            String topic = messageExt.getTopic();
            String tags = messageExt.getTags();
            String body = new String(messageExt.getBody(), "utf-8");

            log.info("MQ消息topic={}, tags={}, 消息内容={}", topic,tags,body);
        } catch (Exception e) {
            log.error("获取MQ消息内容异常{}",e);
        }
        // TODO 处理业务逻辑
        return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
    }
}

```

