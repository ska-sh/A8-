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