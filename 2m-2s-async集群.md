# RocketMQ 集群部署 2m-2s-async

## 集群结构

<table>
  <tr>
    <td>IP</td>
    <td>BrokerRole</td>
    <td>BrokerName</td>
  <tr>
    <td>192.168.0.10</td>
    <td>Master</td>
    <td>broker-a</td></tr>
  <tr>
    <td>192.168.0.11</td>
    <td>Slave</td>
    <td>broker-a-s</td></tr>
  <tr>
    <td>192.168.0.20</td>
    <td>Master</td>
    <td>broker-b</td></tr>
  <tr>
    <td>192.168.0.21</td>
    <td>Slave</td>
    <td>broker-b-s</td></tr>
</table>

## 修改集群配置

### 修改 192.168.0.10 的 broker-a.properties
```bash
# cd /usr/local/rocketmq-all-4.7.1-bin-release/conf/2m-2s-async

# vim broker-a.properties
brokerClusterName=rocketmq-cluster
brokerName=broker-a
brokerId=0
namesrvAddr=192.168.0.10:9876;192.168.0.20:9876
deleteWhen=04
fileReservedTime=48
storePathRootDir=/usr/local/data/store
storePathCommitLog=/usr/local/data/store/commitlog
brokerRole=ASYNC_MASTER
flushDiskType=ASYNC_FLUSH
```

### 修改 192.168.0.11 的 broker-a-s.properties
```bash
# cd /usr/local/rocketmq-all-4.7.1-bin-release/conf/2m-2s-async

# vim broker-a-s.properties
brokerClusterName=rocketmq-cluster
brokerName=broker-a
brokerId=1
namesrvAddr=192.168.0.10:9876;192.168.0.20:9876
deleteWhen=04
fileReservedTime=48
storePathRootDir=/usr/local/data/store
storePathCommitLog=/usr/local/data/store/commitlog
brokerRole=SLAVE
flushDiskType=ASYNC_FLUSH
```

### 修改 192.168.0.20 的 broker-b.properties
```bash
# cd /usr/local/rocketmq-all-4.7.1-bin-release/conf/2m-2s-async

# vim broker-b.properties
brokerClusterName=rocketmq-cluster
brokerName=broker-b
brokerId=0
namesrvAddr=192.168.0.10:9876;192.168.0.20:9876
deleteWhen=04
fileReservedTime=48
storePathRootDir=/usr/local/data/store
storePathCommitLog=/usr/local/data/store/commitlog
brokerRole=ASYNC_MASTER
flushDiskType=ASYNC_FLUSH
```

### 修改 192.168.0.21 的 broker-b-s.properties
```bash
# cd /usr/local/rocketmq-all-4.7.1-bin-release/conf/2m-2s-async

# vim broker-b-s.properties
brokerClusterName=rocketmq-cluster
brokerName=broker-b
brokerId=1
namesrvAddr=192.168.0.10:9876;192.168.0.20:9876
deleteWhen=04
fileReservedTime=48
storePathRootDir=/usr/local/data/store
storePathCommitLog=/usr/local/data/store/commitlog
brokerRole=SLAVE
flushDiskType=ASYNC_FLUSH
```

## 启动集群服务

***一定要先启动 nameserver，再启动 broker。***

### 启动 192.168.0.10 的 nameserver
```bash
# nohup sh /usr/local/rocketmq-all-4.7.1-bin-release/bin/mqnamesrv > /dev/null &
```

### 启动 192.168.0.20 的 nameserver
```bash
# nohup sh /usr/local/rocketmq-all-4.7.1-bin-release/bin/mqnamesrv > /dev/null &
```

### 启动 192.168.0.10 的 broker
```bash
# nohup sh /usr/local/rocketmq-all-4.7.1-bin-release/bin/mqbroker -c /usr/local/rocketmq-all-4.7.1-bin-release/conf/2m-2s-async/broker-a.properties > /dev/null &
```

### 启动 192.168.0.11 的 broker
```bash
# nohup sh /usr/local/rocketmq-all-4.7.1-bin-release/bin/mqbroker -c /usr/local/rocketmq-all-4.7.1-bin-release/conf/2m-2s-async/broker-a-s.properties > /dev/null &
```

### 启动 192.168.0.20 的 broker
```bash
# nohup sh /usr/local/rocketmq-all-4.7.1-bin-release/bin/mqbroker -c /usr/local/rocketmq-all-4.7.1-bin-release/conf/2m-2s-async/broker-b.properties > /dev/null &
```

### 启动 192.168.0.21 的 broker
```bash
# nohup sh /usr/local/rocketmq-all-4.7.1-bin-release/bin/mqbroker -c /usr/local/rocketmq-all-4.7.1-bin-release/conf/2m-2s-async/broker-b-s.properties > /dev/null &
```

### 在 192.168.0.10 上启动 rocketmq-console
```bash
# vim /usr/local/rocketmq-console-ng-2.0.0.jar
/application
rocketmq.config.namesrvAddr=192.168.0.10:9876;192.168.0.20:9876

# nohup java -jar -Xmn128m -Xmx256m -Xms256m /usr/local/rocketmq-console-ng-2.0.0.jar --server.port=8080 > /dev/null &
```
