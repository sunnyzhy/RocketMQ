# FAQ

## broker busy 和 system busy 解决方案

```bash
# vim broker.conf
# broker busy 异常，可适当增大 waitTimeMillsInSendQueue 的值
waitTimeMillsInSendQueue = 6000

# system busy 异常，可适当增大 osPageCacheBusyTimeOutMills 的值
osPageCacheBusyTimeOutMills = 6000

# 发送消息的线程池大小，如果值大于 1，就需要将 useReentrantLockWhenPutMessage 设置为 true; 如果值等于 1，就不需要配置 useReentrantLockWhenPutMessage
sendMessageThreadPoolNums = 8
# 发送消息是否使用可重入锁
useReentrantLockWhenPutMessage = true
```

## RocketMq broker注册IP

RocketMQ broker启动时使用了默认的配置文件，其中 brokerIP1 的值默认是系统自动识别的第一个IP地址，所以在多网卡的服务器上需要手动配置IP地址。

解决方法：

```bash
# cd /usr/local/rocketmq/distribution/target/apache-rocketmq

# echo "brokerIP1=192.168.x.x" > conf/broker.properties

# nohup sh bin/mqnamesrv &

# nohup sh bin/mqbroker -n localhost:9876 -c conf/broker.properties &
```

## ```the broker's disk is full [CL:  0.91 CQ:  0.91 INDEX:  0.91]```

原因：

- 检查磁盘空间，已使用空间是 91%
- ```~/logs/rocketmqlogs/rocketmq_client.log``` 所在的磁盘空间占用超过 90%，[如何查看日志文件的配置路径](./自定义日志的存储路径.md '如何查看日志文件的配置路径')
- rocket 是根据 ratio 来决定磁盘是否满的，默认是 90%，每 60 秒扫描一次磁盘，如果大于 ratio 就会对所有的 producer 发送过来的请求返回磁盘满的错

解决方法：

- 清理 ```~/logs/rocketmqlogs/rocketmq_client.log``` 所在的磁盘空间，即 ```/root``` 的磁盘空间
