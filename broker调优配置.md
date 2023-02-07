# broker 调优配置

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

# 消息发送队列等待时间，默认200
waitTimeMillsInSendQueue=10000

# 发送消息的最大线程数，默认1，因为服务器配置不同，具体多少需要压测后取最优值
sendMessageThreadPoolNums=16

# 发送消息是否使用可重入锁（4.1版本以上才有）
useReentrantLockWhenPutMessage=true
```
