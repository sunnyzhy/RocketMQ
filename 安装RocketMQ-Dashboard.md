# 安装 RocketMQ-Dashboard

## 官网

```
https://rocketmq.apache.org/download#rocketmq-dashboard
```

## 编译

```bash
# unzip rocketmq-dashboard-1.0.0-source-release.zip

# cd rocketmq-dashboard-1.0.0

# vim src/main/resources/application.properties
server.port=8090
rocketmq.config.namesrvAddr=localhost:9876
rocketmq.config.loginRequired=true

# vim src/main/resources/users.properties
admin=[YourPassword],1

# mvn clean package -Dmaven.test.skip=true
```

## 启动

```bash
# cd target

# nohup java -jar -Xmn128m -Xmx256m -Xms256m /usr/local/rocketmq-dashboard-1.0.0.jar &
```

## 浏览器访问

```
http://localhost:8090/
```
