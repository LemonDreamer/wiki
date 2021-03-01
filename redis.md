## 1.redis定义

redis全称remote dictionary server，一种基于键值对(key-value)的NoSQL数据库。Redis中的值可以是由string、hash、list、set、zset、Bitmaps、HyperLogLog、GEO等多种数据结构和算法组成。

思考：NoSQL数据库有哪些？其与关系型数据库相比的优缺点是什么？

## 2.redis8个重要特性

2.1）速度快：redis所有数据存放在内存中，是其速度快的最主要原因；其次，redis是用c语言编写的，接近操作系统的语言；还有，redis是单线程的结构，不存在多线程可能产生的资源竞争问题；以及，redis代码本身的精炼与优雅；

2.2）数据结构的多样性：基于键值对的数据结构服务器

2.3）功能丰富性：键过期可用以实现缓存；发布订阅可用以实现消息系统；可利用luas脚本创造新redis命令；简单的事务功能在一定程度上保证事务特性；流水线功能可一次性批量redis命令，减少网络连接的开销。

2.4）简单稳定：源码量少；单线程模型使服务端处理模型简单及简单化客户端开发；实现自己的事务处理的相关功能，不依赖于操作系统类库；使用的稳定性；

2.5）客户端语言多

2.6）可持久化：使用RDB&AOF两种策略将内存数据保存至硬盘中；

2.7）主从复制

2.8）高可用和分布式：Redis从2. 8 版本正式提供了高可用实现 Redis Sentinel， 它能够保证 Redis节点的故障发现和故障自动转移。 Redis从3.0版本正式提供了分布式实现 Redis Cluster， 它是Redis真正的分布式实现，提供了高可用、 读写和容量的扩展性。

## 3.应用场景

缓存、排行榜系统、计数器应用、社交网络、消息队列系统；

## 4.redis全局命令

```
#查看所有键，keys * 遍历所有键,因此其时间复杂度是o(n)
keys *

#键总数,dbsize是直接获取redis内置的键总数变量，不会遍历所有键，因此其时间复杂度是o(1)
dbsize

#检查键是否存在,存在返回1,不存在返回0
exists key

#删除键,返回结果为成功删除键的个数，假设删除一个不存在的键，就会返回0,支持删除多个键
del key [key]

#键过期
expire key seconds

#查询键过期时间,返回键的剩余过期时间；如果返回值是大于等于0的整数则表示键的剩余过期时间,返回值为-1表示没有设置过期时间,值为-2表示键不存在;
ttl key

#键的数据结构类型，如果键不存在，则返回none
type key

#查询内部编码
object encoding key

127.0.0.1:6379> keys *
1) "csharp"
2) "python"
3) "hobbies"
4) "go"
5) "java"

127.0.0.1:6379> dbsize
(integer) 5

127.0.0.1:6379> exists csharp
(integer) 1

127.0.0.1:6379> expire csharp 60
(integer) 1

127.0.0.1:6379> ttl java
(integer) -1

127.0.0.1:6379> type java
string

127.0.0.1:6379> object encoding hobbies
"quicklist"
```

## 5.各类型数据结构命令

### 5.1.字符串

```
#设置值
#EX seconds:为键设置秒级过期时间
#PX milliseconds:为键设置毫秒级过期时间;
#NX:键必须不存在，才可以设置成功,用于添加
#XX:键必须存在，才可以设置成功，用于更新
set key value [EX seconds|PX milliseconds] [NX|XX]

setex key seconds value 
setnx key value(实现redis分布式锁的一种方案)

#获取值
get key

#批量设置值
mset key value [key value ...]

#批量获取值,n次get时间=1次网络时间+n次命令时间
mget key [key ...]

#计数,用于对值自增操作
#值不是整数，返回错误(error) ERR value is not an integer or out of range
#值是整数，返回自增后的结果
#键不存在，按照值为0自增，返回结果为1
incr key

#值自减
decr key 

#按指定数字自增
incrby key increment 

#按指定数字自减
decrby key decrement 

#自增浮点数
incrbyfloat key increment

#追加值,向字符串尾部加值
append key value

#字符串长度
strlen key

#设置并返回原值
getset key value

#设置指定位置的字符
setrange key offset value

#获取部分字符串
getrange key start end

```

### 5.2.Hash

```
#设置值
hset key field value [field value ...]

#获取值
hget key field

#删除field
hdel key field [field ...]

#计算field个数
hlen key

#批量设置或获取field-value
hmget key field [field ...]
hmset key field value [field value ...]

#判断field是否存在
hexists key field

#获取所有field
hkeys key

#获取所有value
hvals key

#获取所有key-value
hgetall key

#field自增
hincrby key field 
hincrbyfloat key field

#计算value的字符串长度(需要Redis3.2以上)
hstrlen key field
```

### 5.3.列表

```
#添加:从右边插入元素
rpush key value [value ...]

#添加:从左边插入元素
lpush key value [value ...]

#添加向某个元素前或者后插入元素
linsert key BEFORE|AFTER pivot element

#查找:获取指定范围内的元素列表
#索引下标从左到右分别是0到N-1但是,从右到左分别是-1到-N
#lrange中的end选项包含了自身
lrange key start end

#查找:获取列表指定索引下标的元素
lindex key index

#查找:获取列表长度
llen key

#删除:从列表左侧弹出元素
lpop key

#删除:从列表右侧弹出元素
rpop key

#删除:指定元素
#lrem命令会从列表中找到等于element的元素进行删除,根据count的不同分为三种情况： 
#count>0,从左到右,删除最多count个元素
#count<0,从右到左,删除最多count绝对值个元素
#count=0,删除所有
lrem key count element

#删除:按照索引范围修剪列表
ltrim key start stop

#修改:修改指定索引下标的元素
lset key index newValue

#阻塞操作
```

### 5.4.集合

#### 5.4.1.集合内操作

```
#添加元素
sadd key member [member ...]

#删除元素
srem key member [member ...]

#计算元素个数
scard key

#判断元素是否在集合内，如果给定元素element在集合内返回1，反之返回0。
sismember key member

#随机从集合返回指定个数元素
srandmember key count

#从集合中随机弹出元素
spop key

srandmember和spop都是随机从集合选出元素，两者不同的是spop命令执行后，元素会从集合中删除，而srandmember 不会。

#获取所有元素
smembers key

```

#### 5.4.2.集合间操作

```
#求多个集合的交集
sinter key [key ...]

#求多个集合的并集
suinon key [key ...]

#求多个集合的差集
sdiff key [key ...]

#将交集并集差集进行保存
sinterstore destination key [key ...] 
suionstore destination key [key ...] 
sdiffstore destination key [key ...]

将集合间交集、并集、差集的结果保存在destination key中。
```

### 5.5.有序集合

​        有序集合不重复、可排序，给每一个元素设置一个分数作为排序依据。

####         5.5.集合内

```
#添加元素
zadd key [NX|XX] [CH] [INCR] score member [score member ...]
·ch:返回此次操作后，有序集合元素和分数发生变化的个数

#计算成员个数
zcard key

#计算某个成员的分数
zscore key member

#计算成员排名(排名从0开始)
zrank key member(从低到高)
zrevrank key member(从高到低)

#删除成员
zrem key member

#增加成员分数
zincrby key increment member

#返回指定排名范围的成员
zrange key start end [withscores] 
zrevrange key start end [withscores]

#返回指定分数范围的成员
zrangebyscore key min max [WITHSCORES] [LIMIT offset count]
zrevrangebyscore key max min [WITHSCORES] [LIMIT offset count]
min和max还支持开区间(小括号)和闭区间(中 括号)，-inf和+inf分别代表无限小和无限大。

#返回指定分数范围的成员个数
zcount key min max

#删除指定排名内的升序元素
zremrangebyrank key start stop

#删除指定分数范围的成员
zremrangebyscore key min max
```

## 6.键管理

### 6.1.单个键

```
#重命名键
rename key newkey
如果在rename之前，键newkey已经存在，那么它的值也将被覆盖。

renamenx key newkey

#随机返回一个键
randomkey

#键过期(后期深入)
·expireat key timestamp：键在秒级时间戳timestamp后过期

#迁移键
move key db
```

### 6.2.遍历建

```
#全量遍历
key pattern

#渐进式遍历
scan cursor [MATCH pattern] [COUNT count] [TYPE type]
hscan key cursor [MATCH pattern] [COUNT count]
sscan key cursor [MATCH pattern] [COUNT count]
zscan key cursor [MATCH pattern] [COUNT count]

·cursor是必需参数，实际上cursor是一个游标，第一次遍历从0开始，每次scan遍历完都会返回当前游标的值， 直到 游标值为0， 表示遍历结束。 
·match pattern是可选参数，它的作用的是做模式的匹配，这点和keys的模式匹配很像。 
·count number是可选参数，它的作用是表明每次要遍历的键个数，默认值是10，此参数可以适当增大。
```

### 6.3.数据库

```
#切换数据库
select dbIndex

#键总数
dbsize

#清除数据库
flushdb只清除当前数据库
flushall清除所有数据库
```

