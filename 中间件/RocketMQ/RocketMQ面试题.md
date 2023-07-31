## 1、RocketMQ 如何保证消息不丢失
首先在如下三个部分都有可能丢失消息的情况：
1. Producer端
2. Broker端
3. Consumer端
### 1.1、Producer端如何保证消息不丢失
- 采取send()同步发送消息，发送结果是同步感知的
- 发送失败后可以重试，设置重试次数。默认3次
```java
producer.setRetryTimesWhenSendFailed(10);
```
- 集群部署，比如发送失败了的原因可能是当前Broker宕机了，重试的时候会发送到其他Broker上
### 1.2、Broker端如何保证消息不丢失
- 修改刷盘策略为同步刷盘。默认情况下是异步刷盘
```
flushDiskType = SYNC_FLUSH
```
- 集群部署，主从模式，高可用
### 1.3、Consumer端如何保证消息不丢失
完全消费正常后在进行手动ack确认。

## 2、RocketMQ消费模式有几种
消费模型由Consumer决定，消费维度为Topic.
- 集群消费
```
1. 一条消息只会被同Group中的一个Consumer消费
2. 多个Group同时消费一个Topic时，每个Group都会有一个Consumer消费到数据
```

- 广播消费
```
消息将对一个Consumer Group下的所有Consumer实例都消费一遍
```

## 3、消费消息是push还是pull?
RocketMQ没有真正意思的push，都是pull，虽然有类push类，但实际底层实现采用的是<font color="#dd0000">长轮询机制</font>,即拉去方式
```
broker端属性longPollingEnable标记是否开启长轮询。默认开启
```

源码如下：
```java
// {@link org.apache.rocketmq.client.impl.consumer.DefaultMQPushConsumerImpl#pullMessage()}
// 看到没，这是一只披着羊皮的狼，名字叫PushConsumerImpl，实际干的确是pull的活。

// 拉取消息，结果放到pullCallback里
this.pullAPIWrapper.pullKernelImpl(pullCallback);
```
**追问：为什么要主动拉去消息而不是使用事件监听方式？**
事件驱动方式是建立好长连接，由事件（发送数据）的方式来实时推送。

如果broker主动推送消息的话有可能push速度快，消费速度慢的情况，那么就会造成消息在consumer端堆积过多，同时又不能被其他consumer消费的情况。而pull的方式可以根据当前自身情况来pull，不会造成过多的压力而造成瓶颈。所以采取了pull的方式。