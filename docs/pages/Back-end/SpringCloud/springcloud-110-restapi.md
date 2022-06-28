---
title: RESTful 接口设计规范
tags: RESTful
categories: SpringCloud
summary: 本文记录RESTful接口设计规范相关学习和整理。
keywords: 'Spring Cloud,Spring Cloud 微服务,RESTful ,RESTful 接口,RESTful 接口设计规范'
author: Small-Rose / 张小菜
abbrlink: eeda118
date: 2020-09-12 20:00:00
---



## RESTful 接口设计规范

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

本文学习 RESTful API 设计相关的一些原则和规范。

少量的API在使用时不知不觉，但是当后续API不断增加了，需求变动也会导致API版本的变化。主要是为了可以进行规范化开发，并非是只能使用该规范，其他做法亦可，只是作为前人大佬已经在实践中进行总结整理。学习好的编程设计规范总是便捷有利的。



## 二、常见规范

### 1、协议和域名

API通讯使用http协议，如果能使用https尽量使用https。

尽量使用独立域名，

https://api.zhangxiaocai.cn

http://api.zhangxiaocai.cn

如果不能使用可以以api 打头区分访问

https://zhangxiaocai.cn/api



### 2、版本变化

常见的三种版本方式：

1. 在uri中放版本信息：`GET /v1/users/1`
2. Accept Header：`Accept: application/json+v1`
3. 自定义 Header：`X-Api-Version: 1`

推荐用第一种，虽然没有那么优雅，最明显最方便。

### 3、URI资源

#### （1）URI概念

URI 表示资源，资源一般对应服务器端领域模型中的实体类。
URI规范

- 不用大写;
- 用中杠`-`而不用下杠`_`;
- 参数列表要encode;
- URI中的名词表示资源集合，使用复数形式;
- 避免层级过深
- 带上版本号



#### （2）资源路径

**资源集合：** 

所有动物园

```
/zoos/　　　　
```

id为1的动物园内的所有动物

```txt
/zoos/1/animals
```



**单个资源：**

比如：id为1的动物园

```txt
/zoos/1      
```

id为1,2,3的动物园

```txt
/zoos/1;2;3   
```

**层级过深资源：**

过深的导航容易导致url膨胀，不易维护，如 `GET /zoos/1/areas/3/animals/4`，尽量使用查询参数代替路径中的实体导航，如`GET /animals?zoo=1&area=3`; 

### 4、操作类型

对于资源的具体操作类型，由HTTP动词表示。

常用的HTTP动词有下面五个。

| HTTP动作 | 含义                                             | 对应的SQL操作 |
| -------- | ------------------------------------------------ | ------------- |
| GET      | 从服务器取出资源（一项或多项）                   | SELECT        |
| POST     | 在服务器新建一个资源。                           | CREATE        |
| PUT      | 在服务器更新资源（客户端提供改变后的完整资源）   | UPDATE        |
| PATCH    | 在服务器更新资源（客户端提供改变的属性）         | UPDATE        |
| DELETE   | 从服务器删除资源。                               | DELETE        |
| HEAD     | 获取资源的元数据。                               |               |
| OPTIONS  | 获取信息，关于资源的哪些属性是客户端可以改变的。 |               |

示例：

> - GET /zoos：列出所有动物园
> - POST /zoos：新建一个动物园
> - GET /zoos/ID：获取某个指定动物园的信息
> - PUT /zoos/ID：更新某个指定动物园的信息（提供该动物园的全部信息）
> - PATCH /zoos/ID：更新某个指定动物园的信息（提供该动物园的部分信息）
> - DELETE /zoos/ID：删除某个动物园
> - GET /zoos/ID/animals：列出某个指定动物园的所有动物
> - DELETE /zoos/ID/animals/ID：删除某个指定动物园的指定动物

### 5、条件过滤

记录数量很多，不能全返回，需要对数据进行过滤。API应该提供参数，过滤返回结果。 

下面是一些常见的参数（分页条件、查询条件）。

> - ?limit=10：指定返回记录的数量
> - ?offset=10：指定返回记录的开始位置。
> - ?page=2&per_page=100：指定第几页，以及每页的记录数。
> - ?sortby=name&order=asc：指定返回结果按照哪个属性排序，以及排序顺序。
> - ?animal_type_id=1：指定筛选条件

参数的设计允许存在冗余，即允许API路径和URL参数偶尔有重复。比如，GET /zoo/ID/animals 与 GET /animals?zoo_id=ID 的含义是相同的。 

### 6、状态码

#### （1）常规状态码

常规的状态主要是参考 HTTP 状态码。

> - 200 OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。
> - 201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功。
> - 202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）
> - 204 NO CONTENT - [DELETE]：用户删除数据成功。
> - 400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
> - 401 Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）。
> - 403 Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的。
> - 404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
> - 406 Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
> - 410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的。
> - 422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误。
> - 500 INTERNAL SERVER ERROR - [\*]：服务器发生错误，用户将无法判断发出的请求是否成功。


[HTTP状态码](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) 

#### （2）业务类状态码

系统较多的时候，将系统码也带上，使用数字的定义也可以参考HTTP进行设计。

`业务系统码+四位数字`

具体的可以根据需求来。

#### （3）错误与异常

发生错误或异常时：

1. 不响应2xx开头的状态码，客户端可能会缓存成功的http请求；

2. 正确设置http状态码，遵循HTTP协议规范；

3. Response body 需要提供的信息

   ​	1) 错误的代码，方便定位日志，排查问题；

   ​	2) 直接正面的描述错误的文本。

   

如果状态码是4xx，就应该向用户返回出错信息。一般来说，返回的信息中将error作为键名，出错信息作为键值即可。 

| 状态码 | 场景                 |
| ------ | -------------------- |
| 400    | 参数校验失败         |
| 401    | 未验证的用户，未登录 |
| 403    | 无权限               |
| 404    | 资源不存在           |
| 500    | 非业务类异常         |

业务类异常，一般自定义Exception，见词知义。

常规异常：
```json
{
    "status":"failed",
    "code":400，
    "error": "参数xxx校验失败"
}
```
业务类异常：系统名为ABCD
```json
{
    "status":"failed",
    "code":ABCD2001，
    "error": "XXX数据已经过期。"

}
```

#### （4）正常返回

正常返回，减少数据层级。

判断操作成功失败的标记 和操作的单个数据

```json
{
    "success": true,
    "code": 200，
    "info": "操作成功"，
    "data": {"id":"1","name":"zhangxiaocai"}
}
```
info 可选。

分页查询

```json
{
    "paging":{"limit":10,"offset":0,"total":119},
    "data":[{},{},{}...]
}
```

操作与响应

| HTTP操作  | 响应格式                |
| --------- | ----------------------- |
| GET       | 状态标记+集合、单个对象 |
| POST      | 状态标记+新增成功的对象 |
| PUT/PATCH | 状态标记+更新成功的对象 |
| DELETE    | 状态标记                |

#### （5）异步任务

对耗时的异步任务，服务器端接受客户端传递的参数后，应返回创建成功的任务资源，其中包含了任务的执行状态。客户端可以轮训该任务获得最新的执行进度。 

比如常见信息：

任务ID，任务执行状态，发起人。
请求：
```txt
GET /task/3    
```
返回：

```json
{"taskId":3,"createBy":"Anonymous","status":"success"}
    
{"taskId":3,"createBy":"Anonymous","status":"running"}
```
批量请求：
```txt
POST /batchTasks/1;2;3;
```
批量返回：
```json
[{"taskId":3,"createBy":"Anonymous","status":"success"},{},{}...]

[{"from":0,"to":1,"info":"Runing 50 %"},{},{}...]
```

如果任务的执行状态包括较多信息，可以把“执行状态”抽象成组合资源，客户端查询该状态资源了解任务的执行情况。 

提交：
```
GET /task/3/status
```
返回：    
```    
{"progress":"50%","total":18,"success":8,"fail":1}
```



### 7、Hypermedia API

RESTful API最好做到Hypermedia，即返回结果中提供链接，连向其他API方法，使得用户不查文档，也知道下一步应该做什么。

比如，当用户向api.doname.com的根目录发出请求，会得到这样一个文档。

> ```
> {"link": {
>   "rel":   "collection https://www.example.com/zoos",
>   "href":  "https://api.example.com/zoos",
>   "title": "List of zoos",
>   "type":  "application/vnd.yourformat+json"
> }}
> ```

上面代码表示，文档中有一个link属性，用户读取这个属性就知道下一步该调用什么API了。rel表示这个API与当前网址的关系（collection关系，并给出该collection的网址），href表示API的路径，title表示API的标题，type表示返回类型。

Hypermedia API的设计被称为[HATEOAS](http://en.wikipedia.org/wiki/HATEOAS)。Github的API就是这种设计，访问[api.github.com](https://api.github.com/)会得到一个所有可用API的网址列表。

> ```
> {
>   "current_user_url": "https://api.github.com/user",
>   "authorizations_url": "https://api.github.com/authorizations",
>   // ...
> }
> ```

从上面可以看到，如果想获取当前用户的信息，应该去访问[api.github.com/user](https://api.github.com/user)，然后就得到了下面结果。

> ```
> {
>   "message": "Requires authentication",
>   "documentation_url": "https://developer.github.com/v3"
> }
> ```

上面代码表示，服务器给出了提示信息，以及文档的网址。

### 8、数据格式

JSON 或 XML 按要求使用即可。