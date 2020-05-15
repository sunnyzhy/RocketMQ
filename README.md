# windows（单机）
## 修改 store 路径
### 一、复制 broker.properties
```
>cd conf

>copy 2m-noslave\broker-a.properties broker.properties
```

### 二、在 broker.properties 里添加以下配置信息
```
diskMaxUsedSpaceRatio=90
storePathRootDir=F:\\rocketmq\\store
storePathCommitLog=F:\\rocketmq\\store\\commitLog
```

## 三、启动 broker
```
>cd bin

>start mqbroker.exe -n 127.0.0.1:9876  -c ../conf/broker.properties
```

## 修改 logs 路径
修改 logback_broker.xml、logback_filtersrv.xml、logback_namesrv.xml、logback_tools.xml 的方法都相同。

### 一、打开 conf 目录
```
>cd conf
```

### 二、在 xml 配置文件里添加 property 节点
```xml
<configuration>
	<property name="LogPath" value="F:\\rocketmq"></property>
  <appender ...
```

### 三、把 ${user.home} 替换为 ${LogPath}

### 四、启动namesrv
```
>start mqnamesrv.exe
```

### 五、启动 broker
```
>start mqbroker.exe -n 127.0.0.1:9876  -c ../conf/broker.properties
```
