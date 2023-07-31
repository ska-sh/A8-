通过Topic在多Broker中分布式存储实现。

## producer端
发送端指定message queue 发送消息到相应的broker，来达到写入时的负载均衡
- 提升写入吞吐量，当多个producer同时向一个broker写入数据时，性能不会下降
- 消息分布在多broker中，为负载消费做准备

### 默认策略是随机选择：
- producer维护一个index
- 每次取节点会自增
- index向所有broker个数取余
- 自带容错策略

### 其他实现
- SelectMessageQueueByHash
	- hash的是传入的args
- SelectMessageQueueByRandom
- SelectMessageQueueByMachineRoom 没有实现
也可以自定义实现MessageQueueSelector接口中的select方法
```java
MessageQueue select(final List<MessageQueue> mqs, final Message msg, final Object arg);
```

## consumer端
采用的是平均分配算法来进行负载均衡。

### 其他负载均衡算法
- 平均分配策略(默认)(AllocateMessageQueueAveragely)
- 环形分配策略(AllocateMessageQueueAveragelyByCircle)
- 手动配置分配策略(AllocateMessageQueueByConfig)
- 机房分配策略(AllocateMessageQueueByMachineRoom)
- 一致性哈希分配策略(AllocateMessageQueueConsistentHash)
- 靠近机房策略(AllocateMachineRoomNearby)

## 追问：当消费负载均衡consumer和queue不对等的时候会发生什么？

Consumer和queue会优先平均分配，如果Consumer少于queue的个数，则会存在部分Consumer消费多个queue的情况，如果Consumer等于queue的个数，那就是一个Consumer消费一个queue，如果Consumer个数大于queue的个数，那么会有部分Consumer空余出来，白白的浪费了。