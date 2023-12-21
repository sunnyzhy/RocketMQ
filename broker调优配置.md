# broker 调优配置

## RockerMQ 原理

Producer 把消息写入 Broker，Consumer 从 Broker 拉取消息。如下图：

![rocketmq-03.png](./images/rocketmq-03.png 'rocketmq-03.png')

RockerMQ 默认采用异步刷盘策略，Producer 把消息发送到 Broker 后，Broker 会先把消息写入 Page Cache，刷盘线程定时地把数据从 Page Cache 刷到磁盘上。如下图：

![rocketmq-04.png](./images/rocketmq-04.png 'rocketmq-04.png')

在开启 transientStorePoolEnable 的情况下，写入消息时会先写入堆外内存（DirectByteBuffer），然后刷入 Page Cache，最后刷入磁盘。而读取消息是从 Page Cache，这样可以实现读写分离，避免读写都在 Page Cache 带来的问题。如下图：

![rocketmq-05.png](./images/rocketmq-05.png 'rocketmq-05.png')

***除非是对消息要求极为严谨的业务服务，否则可以配置 ```transientStorePoolEnable = true```，这样可以有效缓解 ```[TIMEOUT_CLEAN_QUEUE]broker busy```，但也可能造成消息丢失。***

## 消息重试

Producer 只有收到以下响应码的时候才会进行重试（消息重发）：

```java
//DefaultMQProducer类
private final Set<Integer> retryResponseCodes = new CopyOnWriteArraySet<Integer>(Arrays.asList(
    ResponseCode.TOPIC_NOT_EXIST,
    ResponseCode.SERVICE_NOT_AVAILABLE,
    ResponseCode.SYSTEM_ERROR,
    ResponseCode.NO_PERMISSION,
    ResponseCode.NO_BUYER_ID,
    ResponseCode.NOT_IN_CURRENT_UNIT
));
```

## Broker 端

### broker busy

主要是由于 Broker 在追加消息时持有的锁时间超过了阈值（默认为 1s），Broker 为了自我保护，会实现 ```快速失败 -> 抛出错误```，Producer 会选择其他 Broker 服务器进行重试。

#### Page Cache 繁忙

清理过期请求之前首先会判断 Page Cache 是否繁忙，获取 CommitLog 写入锁超过 1s ，就会给 Producer 返回一个系统繁忙的响应：```CODE: 2  DESC: [PCBUSY_CLEAN_QUEUE]broker busy, start flow control for a while, period in queue: %sms, size of queue: %d```

#### 清理过期请求

清理过期请求时，如果请求线程的创建时间到当前系统时间间隔大于 waitTimeMillsInSendQueue（默认 200ms，可以配置）就会清理这个请求，然后给 Producer 返回一个系统繁忙的响应：```CODE: 2  DESC: [TIMEOUT_CLEAN_QUEUE]broker busy, start flow control for a while, period in queue: %sms, size of queue: %d```

### system busy

#### 拒绝请求

如果 NettyRequestProcessor 拒绝了请求，就会给 Producer 返回一个系统繁忙的响应：```CODE: 2  DESC: [REJECTREQUEST]system busy, start flow control for a while```

请求被拒绝的情况有两种情况：

- Page Cache 繁忙
- TransientStorePoolDeficient

#### 线程池拒绝

Broker 收到请求后，会把处理逻辑封装成到 Runnable 中，由线程池来提交执行，如果线程池满了就会拒绝请求（这里线程池中队列的大小默认是 10000，可以通过参数 sendThreadPoolQueueCapacity 进行配置），就会给 Producer 返回一个系统繁忙的响应：```CODE: 2  DESC: [OVERLOAD]system busy, start flow control for a while```

## Consumer 端

### 缓存消息数量超过阈值

ProcessQueue 保存的消息数量超过阈值，默认 1000，可以配置 pullThresholdForQueue

### 缓存消息大小超过阈值

ProcessQueue 保存的消息大小超过阈值，默认 100M，可以配置 pullThresholdSizeForQueue

### 缓存消息跨度超过阈值

对于非顺序消费的场景，ProcessQueue 中保存的最后一条和第一条消息偏移量之差超过阈值，默认 2000，可以配置 consumeConcurrentlyMaxSpan

### 获取锁失败

对于顺序消费的情况，ProcessQueue 加锁失败，也会延迟拉取。

## broker 调优配置

broker 调优配置 ```broker.conf``` 或 ```broker-x.properties```:

```conf
# 集群名称
brokerClusterName=rocketmq-cluster

# broker 名称
brokerName=broker-a

# broker 角色
# 0: Master
# > 0: Slave
brokerId=0

# nameServer 地址，用分号分割
namesrvAddr=192.168.0.10:9876;192.168.0.20:9876

# 服务器上存在多网卡或配置了 VIP(虚拟IP) 时，需要指定 broker 具体的 IP 地址
brokerIP1=192.168.0.10

# 删除消息的时间点，默认凌晨 4点
deleteWhen=04

# 消息保留的时长，默认 48 小时
fileReservedTime=48

# 消息持久化存储的根路径
storePathRootDir=/usr/local/data/store

# commitLog 持久化存储的路径
storePathCommitLog=/usr/local/data/store/commitlog

# broker 具体的角色
# ASYNC_MASTER: 异步复制 Master
# SYNC_MASTER: 同步复制 Master
# SLAVE: Slave
brokerRole=ASYNC_MASTER

# 持久化存储的方式
# ASYNC_FLUSH: 异步
# SYNC_FLUSH: 同步
flushDiskType=ASYNC_FLUSH

#线上关闭自动创建topic
autoCreateTopicEnable=false

# 消息发送队列等待时间，默认200
waitTimeMillsInSendQueue=10000

# 发送消息的最大线程数，默认1，因为服务器配置不同，具体多少需要压测后取最优值
sendMessageThreadPoolNums=16

# 发送消息是否使用可重入锁（4.1版本以上才有）
useReentrantLockWhenPutMessage=true

#开启临时存储池
transientStorePoolEnable=true

#开启Slave读权限（分担master 压力）
slaveReadEnable=true
 
#关闭堆内存数据传输
transferMsgByHeap=false
 
#开启文件预热
warmMapedFileEnable=true
```
