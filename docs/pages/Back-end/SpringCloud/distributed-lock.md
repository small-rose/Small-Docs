
# 分布式锁

## 一、什么是分布式锁？



在分布式环境中，多个JVM进程同时处理某些资源，但是这几个JVM进程资源对操作变化，不具有共享性和可见性，进程之间会产生干扰，使用分布式技术协调技术来管理多个JVM进程之间的调度的核心就是分布式锁。



## 二、分布式锁的实现有哪些

（1）**Memcached** 。利用 Memcached的 add 命令。add命令属于原子性操作，只有在 `key` 不存在的情况下，才能 `add` 成功，也就意味着线程得到了锁。

（2）**Redis**。利用 Redis 的 `setnx` 命令。此命令同样是原子性操作，只有在 `key` 不存在的情况下，才能 `set` 成功。封装框架有  `Redission` ，`Jedis`

（3）**Zookeeper**：利用 Zookeeper 的顺序临时节点，来实现分布式锁和等待队列。Zookeeper 设计的初衷，就是为了实现分布式锁服务的。

（4）**Chubby**。Google 的Chubby是一个分布式锁服务，Chubby底层一致性实现就是以Paxos算法为基础的。



## 三、分布式锁的实现

（1）Redis 简易版的分布式锁

```
try{
加锁---setnx

业务代码...

}catch(Exception e){
}finally{
	解锁---del
}
```



问题：如果执行业务时出错等原因导致宕机，就无法解锁，可以引入锁的有效时间，自动超时

（2）Redis 简易版的分布式锁，可以自动超时

```
try{
加锁---setnx(key, value)
超时---expire(key, 10S)

业务代码...

}catch(Exception e){
}finally{
	解锁---del
}
```

但是加锁和超时最好原子操作

```
try{
加超时锁---setnx(key, value, 10s)
 
业务代码...

}catch(Exception e){
}finally{
	解锁---del
}
```

（2）Redis 简易版的分布式锁





