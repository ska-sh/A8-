Redis有以下五种基本类型：
- String(字符串)
- Hash(哈希)
- List(列表)
- Set(集合)
- zset(有序集合)
还有三种特殊的数据结构类型
- Geospatial
- Hyperlog
- Bitmap

## Redis的五种基本数据类型
![[Pasted image 20230731165556.png]]
### String(字符串)
- 简介：String是Redis最基础的数据结构类型，他是二进制安全的，可以存储图片或者序列化的对象，最大存储512M
- 简单实用举例：set key value, get key等
- 应用场景：共享session、分布式锁、计数器、限流
- 内部编码有三种，int(8字节长整型)/embstre(小于等于39字节)/raw(大于39字节)
### Hash(哈希)
- 简介：在Redis中，哈希类型是指v（值）本身又是一个键值对（k-v）结构
- 简单使用举例：hset key field value 、hget key field
- 内部编码：ziplist（压缩列表） 、hashtable（哈希表）
- 应用场景：缓存用户信息等。
- **注意点**：如果开发使用hgetall，哈希元素比较多的话，可能导致Redis阻塞，可以使用hscan。而如果只是获取部分field，建议使用hmget。

字符串和哈希类型对比如下图：
![[Pasted image 20230731170428.png]]

### List（列表）
- 简介：列表（list）类型是用来存储多个有序的字符串，一个列表最多可以存储2^32-1个元素。
- 简单实用举例：lpush key value [value ...] 、lrange key start end
- 内部编码：ziplist（压缩列表）、linkedlist（链表）
- 应用场景：消息队列，文章列表,

一图看懂list类型的插入与弹出：
![[Pasted image 20230731170804.png]]

list应用场景参考一下：
- lpush+lpop=Stack（栈）
- lpush+rpop=Queue（队列）
- lpsh+ltrim=Capped Collection（有限集合）
- lpush+brpop=Message Queue（消息队列）

### Set（集合）
![[Pasted image 20230731171029.png]]

- 简介：集合（set）类型也是用来保存多个的字符串元素，但是不允许重复元素
- 简单使用举例：sadd key element [element ...]、smembers key
- 内部编码：intset（整数集合）、hashtable（哈希表）
- 注意点：smembers和lrange、hgetall都属于比较重的命令，如果元素过多存在阻塞Redis的可能性，可以使用sscan来完成。
- 应用场景：用户标签,生成随机数抽奖、社交需求。
### 有序集合（zset）

- 简介：已排序的字符串集合，同时元素不能重复
- 简单格式举例：zadd key score member [score member ...]，zrank key member
- 底层内部编码：ziplist（压缩列表）、skiplist（跳跃表）
- 应用场景：排行榜，社交需求（如用户点赞）。

## Redis 的三种特殊数据类型
### Geo
Redis3.2推出的，地理位置定位，用于存储地理位置信息，并对存储的信息进行操作。
### HyperLogLog
用来做基数统计算法的数据结构，如统计网站的UV。
### Bitmaps 
用一个比特位来映射某个元素的状态，在Redis中，它的底层是基于字符串类型实现的，可以把bitmaps成作一个以比特位为单位的数组