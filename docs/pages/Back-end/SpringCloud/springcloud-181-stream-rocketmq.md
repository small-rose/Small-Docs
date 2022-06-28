---
title: SpringCloud Stream RocketMQ
tags:
  - RocketMQ
categories: SpringCloud
summary: RocketMQ 入门实操。
keywords: >-
  java,spring boot,spring boot Stream RocketMQ ,SpringCloud RocketMQ,SpringBoot
  RocketMQ,SpringBoot SpringCloud stream RocketMQ 配置
author: Small-Rose / 张小菜
abbrlink: 6df661fe
date: 2020-12-04 22:00:00
---

## Spring Cloud Stream - RocketMQ



## RocketMQ的Stream

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rocketmq</artifactId>
            <version>0.2.1.RELEASE</version>
        </dependency>
```
或者

```xml
	<properties>
		<spring-boot.version>2.2.7.RELEASE</spring-boot.version>
        <spring-cloud.version>Hoxton.SR8</spring-cloud.version>
        <stream-rocketmq.version>0.2.1.RELEASE</stream-rocketmq.version>
        <!-- other version -->
	</properties>

		<!-- other dependency -->

		<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-stream</artifactId>
            <version>${spring-cloud-stream.version}</version>
            <!--
            <scope>test</scope>
            <classifier>test-binder</classifier>
            <type>test-jar</type>
            -->
        </dependency>
```

使用：

通过在配置类上使用`@EnableBinding`指定需要使用的Binding，它指定的是一个接口，在对应的接口中会定义一些标注了`@Input` 或 `@Output` 的方法，它们就对应一个Binding。`@Output`对应的是 `org.springframework.messaging.MessageChannel`，`@Input`对应的是 `org.springframework.messaging.SubscribableChannel` 。

Spring Cloud内置的 `@EnableBinding` 可使用的接口有 `org.springframework.cloud.stream.messaging.Source`、`org.springframework.cloud.stream.messaging.Sink` 和 `org.springframework.cloud.stream.messaging.Processor`。

Spring Cloud Stream没有对这些接口进行任何特殊处理，它们仅仅被提供开箱即用。

Source的定义如下，它定义了一个OUTPUT类型的Binding，名称为output，当不通过`@Output`指定Binding的名称时，默认会使用方法名作为Binding的名称。 

```java
public interface Source {

	String OUTPUT = "output";
	
	@Output(Source.OUTPUT)
	MessageChannel output();

}
```

Sink的定义如下，它定义了一个INPUT类型的Binding，名称为input，当不通过@Input指定Binding的名称时，默认会使用方法名作为Binding的名称。

```java
public interface Sink {

	String INPUT = "message-all-in-one-input";

	@Input(Sink.INPUT)
	SubscribableChannel input();

}
```
在一个接口中可以同时定义多个Binding，只需要定义多个@Input或@Output标注的方法。Processor接口同时继承了Source和Sink接口，所以当@EnableBinding指定了Processor接口时相当于同时应用了两个Binding。 Processor 其实就是继承了Source 和 Sink 。
```java
package org.springframework.cloud.stream.messaging;

public interface Processor extends Source, Sink {

}
```

也可以自定义通道名：

```java
package com.xiaocai.rocketmq.producer.stream;

import org.springframework.cloud.stream.annotation.Input;
import org.springframework.cloud.stream.annotation.Output;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.SubscribableChannel;

public interface MyResource {

    String MESSAGE_FOR_CONSUMER = "test-msg-consumer";
    String MESSAGE_FOR_PRODUCER = "test-msg-producer";

    @Output(MyResource.MESSAGE_FOR_PRODUCER)
    MessageChannel outputForProducer();

    @Input(MyResource.MESSAGE_FOR_CONSUMER)
    SubscribableChannel inputForConsumer();
}

```
然后在配置类中使用`@EnableBinding`注解连接到消息代理，进行激活使用即可。

```java
package com.xiaocai.rocketmq.producer.config;

import com.xiaocai.rocketmq.producer.stream.MyResource;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableBinding({MyResource.class})
public class GlobalConfig {


}
```

下面代码中我们指定了`@EnableBinding`接口为`MyResource`接口 和 `MyResource2`接口，即启用了名称为`test-msg-producer`的OUTPUT类型的`Binding` 和名称为`test-msg-consumer`的 Input 类型的Binding 。

Spring Cloud会自动实现该Binding的实现，也会提供Binding接口的实现，并注册到bean容器中。

即可以在程序中自动注入 MyResource 类型的bean，也可以注入MessageChannel类型和 SubscribableChannel 的bean。

**注入Bean的方式使用**
```java
package com.xiaocai.rocketmq.producer.stream;

import org.springframework.beans.factory.annotation.Autowired; 
import org.springframework.messaging.support.MessageBuilder;
import org.springframework.stereotype.Component;

@Component
public class SendServiceBean {

        private MyResource resource;

        @Autowired
        public SendServiceBean(MyResource mySource) {
            this.resource = mySource;
        }

        public void sayHello(String name) {
            resource.outputForProducer().send(MessageBuilder.withPayload(name).build());
        }
}

```
OUTPUT类型的Binding是用来发消息的，Spring Cloud会自动提供`@EnableBinding`指定接口的实现，所以在需要发送消息的时候我们可以直接注入 MyResource 类型的bean，然后通过MyResource的 outputForProducer()获取MessageChannel实例，通过它的send()方法进行消息发送。

**注入 MessageChannel 的方式使用**

```java
package com.xiaocai.rocketmq.producer.stream;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.support.MessageBuilder;
import org.springframework.stereotype.Component;

@Component
public class SendServiceChannel {

    private MessageChannel output;

    @Autowired
    //因为这里使用了自定义名称的资源通道，所以使用自己的通道需要指定自己名字
    public SendServiceChannel(@Qualifier(MyResource.MESSAGE_FOR_PRODUCER) MessageChannel output) {
        this.output = output;
    }

    public boolean justSend(String msg) {
        return output.send(MessageBuilder.withPayload(msg).build());
    }
}

```

那消费方应该也同理：

**注入Bean的方式使用**

```java
package com.xiaocai.rocketmq.producer.stream;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.messaging.Message;
import org.springframework.messaging.MessageHandler;
import org.springframework.messaging.MessageHeaders;
import org.springframework.messaging.MessagingException;
import org.springframework.messaging.support.MessageBuilder;
import org.springframework.stereotype.Component;

@Component
public class AcceptServiceBean {

    private MyResource resource;

    @Autowired
    public AcceptServiceBean(MyResource mySource) {
        this.resource = mySource;
    }

    public void accept() {
        resource.inputForConsumer().subscribe(new MessageHandler() {
            @Override
            public void handleMessage(Message<?> message) throws MessagingException {
                MessageHeaders mhs = message.getHeaders();
                String msgs = (String)message.getPayload();
                // do consumer
            }
        });
    }
}
```

**注入 MessageChannel 的方式使用**

```java
package com.xiaocai.rocketmq.producer.stream;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.messaging.*;
import org.springframework.stereotype.Component;

@Component
public class AcceptServiceChannel {

    private SubscribableChannel output;

    @Autowired
    //因为这里使用了自定义名称的资源通道，所以使用自己的通道需要指定自己名字
    public AcceptServiceChannel(@Qualifier(MyResource.MESSAGE_FOR_CONSUMER) SubscribableChannel output) {
        this.output = output;
    }

    public boolean accept(String msg) {
        return output.subscribe(new MessageHandler() {
            @Override
            public void handleMessage(Message<?> message) throws MessagingException {
                MessageHeaders mhs = message.getHeaders();
                String msgs = (String)message.getPayload();
                // do consumer
            }
        });
    }
}
```
不过一般不会这么搞，总不能每次都要调方法才去消费，可以定义一个基础的模板

```java
package com.xiaocai.rocketmq.producer.mq;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.messaging.Message;
import org.springframework.messaging.MessageHeaders;

import java.text.SimpleDateFormat;

public abstract class BaseConsumer<T> {
    private static ObjectMapper objectMapper;

    public BaseConsumer() {
        objectMapper = new ObjectMapper();
        SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        objectMapper.setDateFormat(df);
    }

    protected void dealWithMessage(Message<T> message) {
        this.processHeader(message.getHeaders());
        this.processPayload(message.getPayload());
    }

    protected void processHeader(MessageHeaders headers) {
        // do some common header ....
    }

    protected abstract void processPayload(T payload);
}
```
ObjectMapper 对象序列化配置：

```java
    @Bean
    public ObjectMapper serializingObjectMapper() {
        JavaTimeModule module = new JavaTimeModule();
        LocalDateTimeDeserializer localDateTimeDeserializer = new LocalDateTimeDeserializer(LOCAL_DATE_TIME_FORMATTER);
        module.addDeserializer(LocalDateTime.class, localDateTimeDeserializer);
        ObjectMapper objectMapper = Jackson2ObjectMapperBuilder.json()
                .modules(module)
                .featuresToDisable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
                .build();
        return objectMapper;
    }
 ```   
然后自己的消费操作继承一个标准，使用 `@StreamListener` 注解监听对应消费通道：


```java
package com.xiaocai.rocketmq.producer.stream;

import com.xiaocai.rocketmq.producer.TestInfo;
import com.xiaocai.rocketmq.producer.mq.BaseConsumer;
import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.messaging.Message;
import org.springframework.stereotype.Component;

@Component
public class ConsumerService extends BaseConsumer<TestInfo> {

    @StreamListener(MyResource.MESSAGE_FOR_CONSUMER)
    public void processSomething(Message<TestInfo> message) {
        dealWithMessage(message);
    }

    @Override
    protected void processPayload(TestInfo payload) {
        // do some business
    }
}
```

> 注意：stream 的后续版本中，几个注解 `@StreamMessageConverter`、`@Output`、`@Input`、`@StreamListener`会过期。


如果消息内容有对应的POJO， `@StreamListener`将会对消息和对应的POJO完成自动转换。

关于SpringCloud-Stream 官方 Demo [spring-cloud-stream-samples](https://github.com/spring-cloud/spring-cloud-stream-samples)

## 关于内容转换

为了传播有关生成消息的内容类型的信息，Spring Cloud Stream默认将contentType头附加到流出的消息。 对于不直接支持头部的中间件，Spring Cloud Stream 提供了将流出消息自动封装在自己的包中的机制。 对于支持头的中间件，Spring Cloud Stream 应用可以从非Spring Cloud Stream应用程序接收具有给定内容类型的消息。

Spring Cloud Stream 2.0已经重新设计了内容类型解析流程。

Spring Cloud Stream允许使用`spring.cloud.stream.bindings.<channelName> .content-type` 属性来声明式地配置输入和输出的类型转换。注意通用类型转换也可以通过在应用中使用转换来轻松完成。

对于输入和输出通道，如果contentType消息头不存在，则通过属性或注释设置contentType只会触发default转换器。 这对于只发送POJO而不发送任何头信息，或者消费没有contentType头的消息的情况非常有用。 框架将始终使用消息头中的值覆盖任何默认设置。
虽然contentType成为必需的属性，但框架将为所有输入/输出通道设置application / json默认值（如果用户没有设置）。

contentType值被解析为媒体类型，例如application / json或text / plain; charset = UTF-8。
  MIME类型对于指示如何转换为String或byte []内容特别有用。 Spring Cloud Stream还使用MIME类型格式来表示Java类型：使用具有type参数的常规类型application / x-java-object。 例如，可以将application / x-java-object; type = java.util.Map或application / x-java-object; type = com.bar.Foo设置为输入绑定的content-type属性。 另外，Spring Cloud Stream提供了自定义的MIME类型，值得注意的是，application / x-spring-tuple指定了一个Tuple（元组）

**通道contentType和消息头**

  可以使用`spring.cloud.stream.bindings.<channelName>.content-type`属性或使用`@Input`和`@Output`注解来配置消息通道的内容类型。 通过这样做，即使您发送的POJO没有contentType信息，框架也会将MessageHeader的contentType设置为为该通道设置的指定值。
  但是，如果发送Message <T>并手动设置contentType，则优先于配置的属性值。 这对于输入和输出通道都是有效的。MessageHeader将始终优先于通道的默认配置的contentType。


**输出通道的ContentType处理**

从2.0版本开始，框架将不再尝试根据Message<T>的负载T来推断contentType。 它将使用contentType头（或由框架提供的默认值）来配置正确的MessageConverter以将负载序列化为byte []。

设置的contentType是示意激活相应的MessageConverter。 然后，转换器可以修改contentType以增加信息，例如Kryo和Avro转换器的情况。
对于流出消息，如果您的负载是 `byte[]` 类型，则框架将跳过转换逻辑，并将这些字节写入连线。在这种情况下，如果消息的contentType不存在，它会将指定的默认值设置为channel。

**输入通道的ContentType处理**

对于输入通道，Spring Cloud Stream使用@StreamListener和@ServiceActivator内容处理来支持转换。检查通道的content-type是否已经通过@Input（contentType =“text / plain”）注解或者`spring.cloud.stream.bindings.<channel>.contentType`属性设置了，或者是否存在contentType头，以此来支持内容类型处理。

框架将检查为Message设置的contentType，选择合适的MessageConverter并应用传递参数作为目标类型的转换。如果转换器不支持目标类型，它将返回null，如果所有配置的转换器都返回null，则抛出MessageConversionException。 就像输出通道一样，如果你的方法负载参数的类型是`Message<byte []>`、 `byte[]` 或者 `Message <？>`，就会跳过转换，可以直接拿到来自连线的原始字节以及相应的头部。


**自定义消息转换**

除了支持开箱即用的转换外，Spring Cloud Stream还支持注册你自己的消息转换实现。 这允许您以各种自定义格式（包括二进制）发送和接收数据，并将它们与特定的contentType关联。Spring Cloud Stream 将所有使用 `@StreamConverter`注释限定的的`org.springframework.messaging.converter.MessageConverter`类型的自定义消息转换器以及开箱即用的消息转换器注册为bean。

框架需要@StreamConverter限定符注释，以避免获取到ApplicationContext上可能存在的其他转换器，并可能与默认的转换器重叠。如果你的消息转换器需要使用特定的 `content-type`和目标类（对于输入和输出），则消息转换器需要扩展`org.springframework.messaging.converter.AbstractMessageConverter`。若使用`@StreamListener`进行转换，一个实现`org.springframework.messaging.converter.MessageConverter`的消息转换器就足够了。


```java
package com.xiaocai.rocketmq.producer.config;


import com.xiaocai.rocketmq.producer.stream.MyCustomMessageConverter;
import com.xiaocai.rocketmq.producer.stream.MyResource;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.annotation.StreamMessageConverter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.converter.MessageConverter;

@Configuration
@EnableBinding({MyResource.class})
public class GlobalConfig {

    @Bean
    @StreamMessageConverter
    public MessageConverter customMessageConverter() {
        return new MyCustomMessageConverter();
    }
}

```
转换器：

```java
package com.xiaocai.rocketmq.producer.stream;

import com.xiaocai.rocketmq.producer.TestInfo;
import org.springframework.messaging.Message;
import org.springframework.messaging.converter.AbstractMessageConverter;
import org.springframework.util.MimeType;

public class MyCustomMessageConverter extends AbstractMessageConverter {
    @Override
    protected boolean supports(Class<?> aClass) {
        return false;
    }

    public MyCustomMessageConverter() {
        super(new MimeType("application", "bar"));
    }

    @Override
    protected Object convertFromInternal(Message<?> message, Class<?> targetClass, Object conversionHint) {
        Object payload = message.getPayload();
        return (payload instanceof TestInfo ? payload : new TestInfo((byte[]) payload));
    }
}
```

