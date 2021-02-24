# 知识点
## 一、消息的消费状态
1. NOT_ONLINE: 订阅端不在线

2. CONSUMED: 消息已经被消费

   订阅端返回 SUCCESS、ReconsumerLater、NULL，或者抛出异常，消息都会走重试流程，消息的消费状态都是 CONSUMED。

3. CONSUMED_BUT_FILTERED: 消息已经被消费，但是被忽略掉了

   ![rocketmq-01.png](./images/rocketmq-01.png 'rocketmq-01.png')
   
   如上图所示，在集群模式下，使用不同的 Consumer （消费者组的名称都是 group1）消费相同 Topic 但不同 Tag 的消息， 最终，Consumer1 却只消费了一部分消息。查看消息的消费状态，发现未被 Consumer1 消费的消息状态都为 CONSUMED_BUT_FILTERED，为什么会出现这种情况？
   
   原因：消息的分配是由 Broker 决定的，而不是 Consumer；在集群模式消费中，消费者组的名称又是相同的情况下，Broker 会负载均衡地把消息分配到各个节点去消费，所以只有一部分消息被分配给了 Consumer1，而另一部分消息则被分配到给了 Consumer2，但是 Consumer2 订阅的是 tag2，所以不会有任何输出，相当于被 Consumer2 忽略掉了。
   
   **解决这个问题的关键就是：针对不同的 Tag ，集群里的 Consumer 要配置不同的 Group。**
   
   ![rocketmq-02.png](./images/rocketmq-02.png 'rocketmq-02.png')

4. NOT_CONSUME_YET: 消息未被消费

   有可能消息发生了堆积，还未被消费；也有可能消费线程 hang 住了，导致消费线程迟迟没有返回。
