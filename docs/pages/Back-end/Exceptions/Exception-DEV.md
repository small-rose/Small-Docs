

# 开发异常

## Mybatis SQL打印问题
----------------
**现象**： mybatis 无法打印SQL日志
**环境**：SpringBoot2.2.2.RELEASE、Mybatis-Plus、Logback

配置：
```yaml
logging:
  level:
    root: INFO
    com.fenet.insurance.mm.mapper: debug   # 这里可以关闭SQL打印
    org:
      springframework:
        transaction: DEBUG
        orm: DEBUG
      apache:
         ibatis: DEBUG
    druid:
      sql:
        DataSource: DEBUG
        Connection: DEBUG
        Statement: DEBUG

 mybatis-plus:
   mapper-locations: classpath:/mapper/${system-conf.dao}/*.xml
   configuration:
     log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
     local-cache-scope: session
     map-underscore-to-camel-case: false
```

**思路**：

使用  logging.level 配置,可以设置console、file的SQL打印

```yaml
logging:
  level:
    root: INFO
    com.fenet.insurance.mm.mapper: debug   # 这里可以关闭SQL打印
    org:
      springframework:
        transaction: DEBUG
        orm: DEBUG
      apache:
         ibatis: DEBUG
    druid:
      sql:
        DataSource: DEBUG
        Connection: DEBUG
        Statement: DEBUG

mybatis-plus:
  mapper-locations: classpath:/mapper/${system-conf.dao}/*.xml
  configuration:
    #log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
    local-cache-scope: session
    map-underscore-to-camel-case: false
 
```

## XML 相关
----------------
**异常信息**
```
the processing instruction target matching "[xX][mM][lL]" is not allowed
```
**思路**：
 
（1）一般是XML解析抛出的异常，多是配置文件XML格式的问题，多见于`<?xml version="1.0" encoding="utf-8"?>` 的前面出现空行或者空格情况。
```xml
<?xml version="1.0" encoding="utf-8"?>
```
（2）在webservice等常见接口定义XML报文也会出现类似问题，将XML报文数据包作为接口的值进行传递时，如下：

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ws="http://com/fenet/insurance/paystation/consume/ws">
   <soapenv:Header/>
   <soapenv:Body>
      <ws:takeApplication >
         <ws:msg><![CDATA[<?xml version="1.0" encoding="utf-8"?>
<PACKET TYPE="REQUEST"> 
  <HEAD> 
    ... 
  </HEAD>  
  <BODY> 
    ...
  </BODY> 
</PACKET>]]>
         </ws:msg>
      </ws:takeApplication>
   </soapenv:Body>
</soapenv:Envelope>
```
正确写法：
 ```xml
 <ws:msg><![CDATA[<?xml version="1.0" encoding="utf-8"?>
<PACKET TYPE="REQUEST"> 
  <HEAD> ...
  </HEAD>  
  <BODY>  ... 
  </BODY> 
</PACKET>]]></ws:msg>
```
如果出现换行或者空格也会出在解析数据报文时出现错误。

空格示例：
```xml
 <ws:msg><![CDATA[ <?xml version="1.0" encoding="utf-8"?>
<PACKET TYPE="REQUEST"> 
  <HEAD>
  </HEAD>  
  <BODY>  
  </BODY> 
</PACKET>]]></ws:msg> 
```
换行示例：
```xml
 <ws:msg><![CDATA[ 
 	<?xml version="1.0" encoding="utf-8"?> 
<PACKET TYPE="REQUEST"> 
  <HEAD>
  </HEAD>  
  <BODY>  
  </BODY> 
</PACKET>]]>
</ws:msg>
```



##  Jenkins 相关
---------------------------
**问题描述**

　　jenkins编译的时候报错：ERROR: Maven JVM terminated unexpectedly with exit code 137

**可能原因**: maven设置的内存不足

**解决参考**：

　　1、找到linux中maven安装目录,然后编辑mvn文件

```SH
[root@]# whereis mvn
[root@]# vim mvn
```

　　2、在mvn配置文件中添加

```TXT
MAVEN_OPTS="$MAVEN_OPTS -Xms256m -Xmx512m -XX:MaxPermSize=128m -XX:ReservedCodeCacheSize=64m"
```
　　

