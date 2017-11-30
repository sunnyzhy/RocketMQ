# 官网
http://rocketmq.apache.org/docs/quick-start/

# 安装
```
# cd /usr/local

# git clone -b develop https://github.com/apache/rocketmq.git

# cd rocketmq

# mvn -Prelease-all -DskipTests clean install -U

# cd distribution/target/apache-rocketmq
```

# 启动Name Server
```
# nohup sh bin/mqnamesrv &

# cat ~/logs/rocketmqlogs/namesrv.log
The Name Server boot success. serializeType=JSON
```

# JVM参数配置
```
# vim bin/runbroker.sh
JAVA_OPT="${JAVA_OPT} -server -Xms1g -Xmx1g -Xmn512m"
```
    根据实际的内存大小自定义

# 启动Broker
```
# nohup sh bin/mqbroker -n localhost:9876 &

# cat ~/logs/rocketmqlogs/broker.log
The broker[localhost.localdomain, 192.168.237.107:10911] boot success. serializeType=JSON and name server is localhost:9876
```

# 停止服务
```
# sh bin/mqshutdown broker
The mqbroker(7679) is running...
Send shutdown request to mqbroker(7679) OK

# sh bin/mqshutdown namesrv
The mqnamesrv(7638) is running...
Send shutdown request to mqnamesrv(7638) OK
```

# 开启端口
```
# firewall-cmd --zone=public --add-port=9876/tcp --permanent

# firewall-cmd --zone=public --add-port=10911/tcp --permanent

# firewall-cmd --zone=public --add-port=10909/tcp --permanent

# firewall-cmd --reload
```
