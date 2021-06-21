# 创建 topic

## topic 结构体
```json
{
	"writeQueueNums": 16,
	"readQueueNums": 16,
	"perm": 6,
	"order": false,
	"topicName": "topic-test",
	"brokerNameList": ["broker-a"],
	"clusterNameList": ["DefaultCluster"]
}
```

- writeQueueNums: 写队列的个数
- readQueueNums: 读队列的个数
- perm: 读写模式, 2:禁读; 4:禁写; 6:同时支持读写
- order: 是否有序
- topicName: 主题
- brokerNameList: broker名称
- clusterNameList: 集群名称

writeQueueNums 和 readQueueNums 介绍:
1. writeQueueNums 和 readQueueNums 是 rocketmq 的读写队列
2. 读写队列是在路由信息时使用。发送消息的时候，使用写队列的个数返回路由信息；消费消息的时候，使用读队列的个数返回路由信息
3. 在物理文件层面，只有写队列才会创建文件。比如，配置写队列的个数是 8，读队列的个数是 4，那么会创建 8 个文件夹（0 1 2 3 4 5 6 7），但在消息消费时，路由信息只返回 4，在具体拉取消息时，就只会消费 0 1 2 3 这 4 个队列中的消息，4 5 6 7压根就没有被消费。反过来，如果写队列的个数是 4，读队列的个数是 8，在生产消息时只会往 0 1 2 3 中生产消息，消费消息时则会从 0 1 2 3 4 5 6 7 所有的队列中消费，当然 4 5 6 7 中压根就没有消息 ，假设消费 group 有两个消费者，事实上只有第一个消费者在真正的消费消息(0 1 2 3)，第二个消费者压根就消费不到消息
4. 只有 readQueueNums >= writeQueueNums,程序才能正常进行。**最佳实践是 readQueueNums = writeQueueNums**
5. **读写分离不同于读写队列**。读写分离使用主从同步机制，将一个节点的数据同步到另外一个节点，主节点多用于写，从节点只用于读，从而减轻主节点的压力。

## add/update topic
- 添加/修改单个主题
   ```bash
   curl -X POST -H 'Content-type':'application/json' http://127.0.0.1:8080/topic/createOrUpdate.do -d '{"writeQueueNums":16,"readQueueNums":16,"perm":6,"order":false,"topicName":"topic-test","brokerNameList":["broker-a"],"clusterNameList":["DefaultCluster"]}'
   ```

- 添加/修改多个主题
   ```bash
   # touch rocketmq_topic.sh
   
   # vim rocketmq_topic.sh
   #/bin/sh
   #set -x
   
   curl -X POST -H 'Content-type':'application/json' http://127.0.0.1:8080/topic/createOrUpdate.do -d '{"writeQueueNums":16,"readQueueNums":16,"perm":6,"order":false,"topicName":"topic-test-1","brokerNameList":["broker-a"],"clusterNameList":["DefaultCluster"]}'
   
   curl -X POST -H 'Content-type':'application/json' http://127.0.0.1:8080/topic/createOrUpdate.do -d '{"writeQueueNums":16,"readQueueNums":16,"perm":6,"order":false,"topicName":"topic-test-2","brokerNameList":["broker-a"],"clusterNameList":["DefaultCluster"]}'
   
   curl -X POST -H 'Content-type':'application/json' http://127.0.0.1:8080/topic/createOrUpdate.do -d '{"writeQueueNums":16,"readQueueNums":16,"perm":6,"order":false,"topicName":"topic-test-3","brokerNameList":["broker-a"],"clusterNameList":["DefaultCluster"]}'

   # chmod 777 rocketmq_topic.sh
   
   # ./rocketmq_topic.sh
   ```
