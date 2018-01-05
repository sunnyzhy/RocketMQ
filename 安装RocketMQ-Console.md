# 官网
https://github.com/apache/rocketmq-externals/tree/master/rocketmq-consol

# 安装
```
# cd /usr/local

# unzip rocketmq-console.zip

# cd rocketmq-console

# vim src/main/resources/application.properties
server.port=8090
rocketmq.config.namesrvAddr=localhost:9876

# mvn clean package -Dmaven.test.skip=true
```

# 启动
```
# cd target

# java -jar -Xms64M -Xmx128M -Xmn20M rocketmq-console-ng-1.0.0.jar
```

# 浏览器访问
http://localhost:8090/
