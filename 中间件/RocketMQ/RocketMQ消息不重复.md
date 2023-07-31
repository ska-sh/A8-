影响消息正常发送和消费的重要原因是网络的不确定性。

引起重复消费的原因

- ACK
正常情况下在consumer真正消费完消息后应该发送ack，通知broker该消息已正常消费，从queue中剔除

当ack因为网络原因无法发送到broker，broker会认为词条消息没有被消费，此后会开启消息重投机制把消息再次投递到consumer

- 消费模式
在CLUSTERING模式下，消息在broker中会保证相同group的consumer消费一次，但是针对不同group的consumer会推送多次

**解决方案**

- 数据库表
处理消息前，使用消息主键在表中带有约束的字段中insert

- Map
单机时可以使用map ConcurrentHashMap -> putIfAbsent guava cache

- Redis
分布式锁搞起来。