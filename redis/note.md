## Redis五大数据类型及其常用操作
### 字符串
set key value ——设置属性值  
get key ——获取值  
getset key value   ——先获取值再重新设置值  
del key ——删除    
incr key  ——自增，如果key不存在，则会创建一个值为0的变量  
decr key ——自减  
incrby key number ——变量+number  
decrby key number ——变量-number  
append key string ——追加字符串  
getrange key start end ——获取[start,end]之间的字符串  
setrange key offset value ——将从offset开始的子串设置为给定值  
bitcount key [ start end ] ——统计二进制位串指定范围内的1的二进制位的数量  


### Hash
hset key k v  ——设置某个key下的键值对  
hmset key k v k v... ——设置某个key下的几个键值对  
hegt key k ——获取某个key下的键值  
hmget key k k...   ——获取某个key下的几个键值  
hgetall key ——获取某个key下的所有键值对  
hdel key k k ... ——删除某个key下的多个键值  
hincrby key k number   ——给某个key下的某个键值+number  
hexists key k  ——判断某个键是否存在 （1——存在，0——不存在）  
hlen key ——获取key的属性个数  
hkeys key ——获取key的所有键名称  
hvals key ——获取key的所有值  

### list
lpush key v v v   ——从左侧插入链表  
rpush key v v v ——从右侧插入链表  
lrange key start end  —— 返回[start,end]之间的元素  
lpop key ——左端弹出  
rpop key ——右端弹出  
llen key ——获取链表元素个数  
lpushx key  v ——如果key存在则插入到头部  
rpushx key v ——如果key存在则插入到尾部  
lrem key  number v —— 删除key中number个v（number为负表示从后往前，0表示所有）  
lset key index v ——在key的index的位置处设置v值  
linsert key before v1 v2 ——在第一个v1之前插入v2  
linsert key after v1 v2 ——在第一个v1之后插入v2  
rpoplpush key1 key2——将key1的值弹出压入key   
lindex key index  ——获取列表中给定位置上的单个元素（从0开始）  
ltrim key start end ——只保留[start,end]之间的元素  
blpop key timeout ——弹出最左端的元素，如果元素不存在将阻塞timeout秒并等待可弹出的元素出现  
brpop key timeout ——弹出最右端的元素，如果元素不存在将阻塞timeout秒并等待可弹出的元素出现  
rpoplpush source-key dest-key ——将source-key中最右端的元素弹出，压入到dest-key的最左端  
brpoplpush source-key dest-key timeout ——将source-key中最右端的元素弹出，压入到dest-key的最左端，如果source-key中元素不存在，将阻塞timeout秒并等待可弹出的元素出现

### set
sadd key v v v...   ——往集合里面添加元素  
srem key v v  v ...——删除  
smembers key ——查看  
sisimember key v ——判断是否存在  
sdiff key1 key2 ——找出key1中在key2不存在的元素  
sinter key1 key2  ——交集  
sunion key1 key2 —— 并集  
scard key ——数目  
srandmember key——随机返回  
sdiffstore key1 key2 key3  ——把key2和key3的差集保存到key1  
sinterstore key1 key2 key3  ——把key2和key3的交集保存到key1  
sunionstore key1 key2 key3  ——把key2和key3的并集保存到key1  
spop key ——随机移除集合中的一个元素  
smove source-key dest-key item ——如果集合source-key包含元素item，那么从集合source-key里面移除元素item，并将元素item添加到集合dest-key中

### zset（sorted-set）
zadd key n v n v   ——添加元素和相应的分数  
zscore key v  ——查看分数  
zcard key ——获取长度  
zrem key v v   ——删除  
zrange key start end ——  获取指定位置范围内的值  
zrange key start end withscores    ——获取指定范围内的值和分数    
zrevrange key start end withscores  
zremrangebyrank key start end ——下标范围删除  
zremrangebyscore key starts ends ——分数范围删除  
zincrby key number v  ——v+number  
zcount key starts ends  ——分数范围内的个数  
zrank key  v —— 返回成员v在有序集合中的排名  


### 常用操作：
keys *  ——获取所有key  
del key1 key2 ——删除  
exists key  ——是否存在  
rename key1 key2 ——重命名  
expire key time ——设置过期时间  
ttl key ——查看剩余时间  
type key——查看类型  
select number ——选择数据库（0-15）  
move key number ——将变量移到number数据库  

### hypeloglog
+ pfadd key  v v v ：影响基数估值则返回1否则返回0.若key不存在则创建  
+ pfcount key ：得到去重后的值  
+ pfmerge destkey key key2：合并多个key  

### bitmap
+ setbit key index v : 设置值  
+ getbit key index :获取值  
+ bitcount key [start,end] :获取某个范围的1的个数  
+ bitop op destkey key (op = or,and,not,xor)  bitmap 之间的运算  

## 发布与订阅
subscribe channel [channer...]   ——订阅给定的一个或多个频道  
unsubscribe [channer...] ——退订给定的一个或多个频道，如果没有指定，那么退订所有频道  
publish channel message  ——向给定频道发送消息  
psubscribe pattern [pattern...] ——订阅与给定模式相匹配的所有频道  
punsubscribe [pattern...]  —— 退订给定的模式，如果未指定，则退订所有的模式  

## Redis注意点
+ 如果用户对一个不存在的键或者一个保存了空串的键执行自增或者自减操作，那么Redis在执行操作时会将这个键的值当作是0来处理。如果用户尝试对一个值无法被解释为整数或者浮点数的字符串键执行自增或者自减操作，那么Redis将向用户返回一个错误。


## Redsi使用案例
### 从海量数据中查询某一前缀的key 
#### 方法一：利用key查找（不推荐）
> keys pattern：查找所有符合给定模式pattern的key  

通过这种方式查询可能会导致查询时间过长，导致redis其他服务卡顿。

#### 方法二：利用scan查找（推荐）
> scan cursor [MATCH pattern] [COUNT count]
+ 基于游标的迭代器，需要基于上一次的游标延续之前的迭代过程；
+ 以0作为游标开始一次新的迭代，直到命令返回游标0完成一次遍历；
+ 不保证每次执行都返回某个给定数量的元素，支持模糊查询；
+ 一次返回的数量不可控，只能是大概率符合count参数。 

### 如何通过Redis实现分布式锁
#### 方法一
> setnx key value ：如果key不存在，则创建并赋值
+ 时间复杂度：O(1)
+ 返回值：设置成功，返回1；设置失败，返回0。

> expire key seconds  
+ 设置key的生存时间，当key过期时（生存时间为0），会被自动删除。

#### 方法二
> set key value [EX seconds] [PX milliseconds] [NX|XX]
+ EX second ：设置键的过期时间为second秒
+ PX millisecond：设置键的过期时间为millisecond毫秒
+ NX：只有键不存在时，才对键进行设置操作
+ XX：只在键已经存在时，才对键进行设置操作
+ SET操作成功完成时，返回OK，否则返回nil 

## RDB和AOF
由于Redis是内存数据库，它将自己的数据库状态存储在内存中，如果不想办法将存储在内存中的数据库状态保存到磁盘里面，那么一旦服务器进程退出，服务器中的数据库状态也会消失不见。

#### RDB
+ SAVE：会阻塞Redis服务器进程，直到RDB文件创建完成为止，在服务器进程阻塞期间，服务器不能处理任何命令请求。
+ BGSAVE：会派生出一个子进程，然后由子进程负责创建RDB文件，父进程进行处理命令请求。

一旦BGSAVE开始执行了，SAVE命令和BGSAVE命令都会被服务器拒绝，避免产生竞争条件。如果BGSAVE命令正在执行，那么客户端发送的BGREWRITEAOF命令会被延迟到BGSAVE命令执行完毕之后执行；如果BGREWRITEAOF命令正在执行，那么客户端发送的BGSAVE命令会被服务器拒绝。避免产生大量的磁盘写入操作。

**将某段时间内的所有数据持久化到磁盘中，类似快照的行为。当在进行持久化的过程时有数据的更新，会把这些记录保存到备份文件中，最后会把备份文件拿来替换原文件。**
> 缺点：如果机器发生故障，容易丢失某个时间段内的数据。

#### AOF
**将所有写命令保存到AOF缓冲区中，根据appendfsync的值（默认为everysec）来对AOF文件同步，由于大量的写入命令会导致AOF文件过大，后台就会开启一个子进程对原AOF文件进行重写（合并命令），对文件进行压缩。如果在重写期间执行了写入命令，会将写入命令保存到AOF重写缓冲区中，等到AOF重写结束，再将AOF重写缓冲区中的内容追加到新AOF文件中，此时会对父进程阻塞，最后用新AOF文件替换原AOF文件。
> 缺点：由于恢复要进行的操作较多，可能会导致主线程阻塞。

## 主从配置（docker无redis.conf）
+ **docker启动主redis：** docker run -d --name redis-master -p 6379:6379 redis --requirepass "mypassword"
+ **docker启动从redis：** docker run -d --name redis-slave -p 6380:6379 redis --requirepass "mypassword"
+  **连接从redis并进行配置：**
         1.   auth <slave-password>
         2.   slaveof <master-ip> <master-port>。<master-ip>为主库服务ip，<master-port>表示主库所在端口，默认6379
         3.   config set masterauth <master-password>。<master-password>即为主库访问密码

#### redis.conf相关配置
``` xml
##设置当本机为slav服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步

    slaveof <masterip> <masterport>

###当master服务设置了密码保护时，slav服务连接master的密码

    masterauth <master-password>

##你可以配置salve实例是否接受写操作。可写的slave实例可能对存储临时数据比较有用(因为写入salve
##的数据在同master同步之后将很容易被删除

slave-read-only yes

# 是否在slave套接字发送SYNC之后禁用 TCP_NODELAY？
# 如果你选择“yes”Redis将使用更少的TCP包和带宽来向slaves发送数据。但是这将使数据传输到slave
# 上有延迟，Linux内核的默认配置会达到40毫秒
# 如果你选择了 "no" 数据传输到salve的延迟将会减少但要使用更多的带宽

repl-disable-tcp-nodelay no

# slave的优先级是一个整数展示在Redis的Info输出中。如果master不再正常工作了，哨兵将用它来
# 选择一个slave提升=升为master。
# 优先级数字小的salve会优先考虑提升为master，所以例如有三个slave优先级分别为10，100，25，
# 哨兵将挑选优先级最小数字为10的slave。
# 0作为一个特殊的优先级，标识这个slave不能作为master，所以一个优先级为0的slave永远不会被
# 哨兵挑选提升为master

slave-priority 100
```

## 缓存穿透和缓存雪崩
+ 缓存穿透：指大量查询不再缓存中存在的数据，此时会直接查询数据库。
> **解决方法：**
     1.采用布隆过滤器  
     2.为不存在的数据设置空值  
+ 缓存雪崩：大量缓存数据过期。
> **解决方法：**
      1.加锁排队  
      2.   


#### 布隆过滤器
首先构建长度足够的位阵列，所有位置的值都为0，然后创建m个hash函数，将所有元素根据m个hash函数得到的指定位位置的值都置为1。查询的时候，同样根据m个hash函数得到位位置的值，只要其中有0，说明一定不含有该元素，全为1则可能含有。



