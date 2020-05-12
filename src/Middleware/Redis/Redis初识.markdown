# Redis初识
## 1.什么是Redis?
> Redis是一个开源(BSD许可)的内存数据结构存储，用作数据库、缓存和消息代理。它支持诸如字符串、散列、列表、集、带范围查询的排序集、位图、hyperloglog、带半径查询和流的地理空间索引等数据结构。Redis具有内置的复制、Lua脚本、LRU清除、事务和不同级别的磁盘持久性，
并通过Redis Sentinel和Redis集群的自动分区提供高可用性。以上来自于官方介绍，用自己的话说Redis的关键点是不同于Mysql的磁盘存储，Redis是基于内存,读取数据的速度远远大于Mysql
## 2.Redis的特点
* 基于内存存储
* 数据类型丰富
* 支持磁盘持久化
* 支持主从同步
* 支持集群分片
* 采用单线程处理任务(此处描述不准确，待补充)
* 使用非阻塞IO模型
## 3.Redis的数据类型
### 3.1 String
> 字符串，最基本的数据类型，底层采用SDS来存储
* 演示
```
127.0.0.1:6379> set test "2008"
OK
127.0.0.1:6379> get test
"2008"
```
### 3.2 Hash
> String作为key组成的hash表
* 演示
```
127.0.0.1:6379> hset test name "xiaoming"
(integer) 1
127.0.0.1:6379> hget test name
"xiaoming"
127.0.0.1:6379> hmset test name "xiaohong" age 25
OK
127.0.0.1:6379> hmget test name age
1) "xiaohong"
2) "25"
127.0.0.1:6379>
```
### 3.3 List
> 列表，按照String元素插入的顺序排序
* 演示
```
127.0.0.1:6379> lpush test a
(integer) 1
127.0.0.1:6379> lpush test b
(integer) 2
127.0.0.1:6379> lrange test 0 5
1) "b"
2) "a"
127.0.0.1:6379> llen test
(integer) 2
127.0.0.1:6379> lindex test 0
"b"
```
### 3.4 Set
> String元素组成的无序集合，不允许重复
* 演示
```
127.0.0.1:6379> sadd test a b c
(integer) 3
127.0.0.1:6379> sadd test 1
(integer) 1
127.0.0.1:6379> sadd test 2 3
(integer) 2
127.0.0.1:6379> sadd test 1
(integer) 0
127.0.0.1:6379> smembers test
1) "1"
2) "a"
3) "b"
4) "2"
5) "3"
6) "c"
```
### 3.5 Sorted Set
> 与Set相似，但可以为集合中的元素进行排序，集合中每个元素拥有一个分数，通过分数为每个元素排序，
集合中的每个元素不可以重复，但是分数可以重复
* 演示
```
127.0.0.1:6379> del test
(integer) 1
127.0.0.1:6379> zadd test 1 a
(integer) 1
127.0.0.1:6379> zadd test 2 b
(integer) 1
127.0.0.1:6379> zadd test 3 c
(integer) 1
127.0.0.1:6379> zadd test 0 aa
(integer) 1
127.0.0.1:6379> zrangebyscore test 0 5
1) "aa"
2) "a"
3) "b"
4) "c"
```
## 4.Redis 键的过期时间
### 4.1 时间存放在哪里？
> Redis中可以用EXPIRE等命令来设置过期时间，Redis数据库中有一个expires字段用来存储一个由键与过期时间组成的hash表
### 4.2 Redis是实时检测过期时间？
* 定时删除: 到达指定时间把过期的键进行删除
* 惰性删除: 每次通过键取值时判断键是否过期，如果过期则删除
* 定期删除：设置间隔时间，定期删除过期的键
> Redis采用惰性删除与定期删除两种方式
## 5.Redis持久化
> Redis基于内存存储数据，如果服务器出现故障导致Redis进程退出就会出现数据丢失的情况，因此需要定期的将数据持久化到磁盘上，数据持久化有两种方式RDB,AOF
### 5.1 RDB(快照)
> 制作快照生成RDB文件有两种方式SAVE与BGSAVE,Redis服务器启动时会自动载入安装目录的RDB文件，载入RDB文件期间Redis服务器处于阻塞状态
#### 5.1.1 SAVE
> SAVE命令会阻塞Redis的服务器进程，直到RDB文件被创建完毕
#### 5.1.2 BGSAVE
> BGSAVE命令首先检查有没有已经fork出的子进程或者正在执行的AOF任务，如果有则抛出错误,没有则fork出一个子进程来创建RDB文件且不会阻塞Redis主进程，redis采用copy-on-write实现备份。
* Redis默认自动执行的的BGSAVE配置，
```
redis.conf

save 900 1      900秒内如果有1个键发生变化则保存快照
save 300 10     300秒内如果有10个键发生变化则保存快照
save 60 10000   60秒内有10000键发生变化则保存快照
```
#### 5.1.3 RDB触发方式
* 手动触发: 主动输入SAVE命令或BGSAVE命令
* 自动触发: 
    1. 根据redis.conf配置里的 save m n 定时触发
    2. 主从复制时，主节点自动触发
    3. 执行Debug Reload
    4. 执行Shutdown且没有开启AOF持久化
#### 5.1.4 RDB的优缺点
* 全量数据快照，文件小，恢复快
* 无法保存最近一次快照之后的数据
### 5.2 AOF(Append-Only-File)持久化
> AOF持久化的方式采用保存除了查询以外的所有数据库变更指令,保存指令的方式以append形式追加到AOF文件中，AOF默认是关闭状态，修改redis.conf文件下 appendonly 为 yes即可开启，
> AOF开启后，主线程负责将命令写到缓冲区，同步线程负责同步数据到硬盘，同时主线程会对比上次同步的时间，如果大于2秒阻塞，小于则继续执行。
#### 5.2.1 持久化策略
* always
> 每条命令都会持久化
* everysec
> 每秒进行持久化
* no
> 根据操作系统来进行持久化
#### 5.2.2 AOF重写
> AOF是针对Redis的命令来存储的，因此就会导致一些问题:
```
127.0.0.1:6379> hmset test a 1
OK
127.0.0.1:6379> hmset test b 2
OK
127.0.0.1:6379> hmset test c 3
OK
```
> AOF文件过大，其中的命令完全可以进行合并如
```
127.0.0.1:6379> hmset test a 1 b 2 c 3
```
> 因此可以将AOF进行重写，Redis进行重写不会对现有AOF进行分析，而是读取当前的数据库来实现，命令行输入BGREWRITEAOF则开始进行重写
* AOF后台重写
> 另一种AOF重写的方式是Redis开启一个子进程，由子进程来执行重写操作，但是会导致数据出现不一致的问题，Redis采用AOF重写缓冲区来解决，
当Redis开启一个子进程进行重写，此时如果有新的命令则Redis将命令保存到缓冲区，子进程重写完毕后通知主进行将缓冲区命令写入AOF文件
#### 5.2.3 AOF的优缺点
* AOF内保存的指令可读性高，适合保存增量数据，数据不易丢失
* 文件体积大，恢复时间长
### 5.3 Redis数据恢复
> 在同时拥有AOF与RDB的情况下，Redis默认只会优先加载AOF然后启动，如没有AOF则加载RDB然后启动
### 5.4 Redis新的持久化方式
> Redis4.0推出了新的持久化方式，将AOF与RDB进行结合.....
## 6.Redis主从同步
> Redis需要主从同步的原因与Mysql类似，读写分离提高并发量，防止单台Redis挂掉导致数据丢失等问题,
Redis的主从同步一般有单个Master与多个Slave组成，Master负责写，Slave负责读
### 6.1 全同步过程
* Slave发送psync命令到Master
* Master启动一个后台进程用于将Redis的数据快照保存到文件中
* Master将保存数据快照期间接受到的写命令缓存起来
* Master的数据快照生成完毕后发送给Slave
* Slave执行Master发送过来的RDB文件
* Master将这期间收集的增量写命令发送给Slave
### 6.2 增量同步过程
* Master接受到用户指令后判断是否需要传播到Slave
* Master将操作记录追加AOF文件
* Master首先确定Slave，然后将指令写入到响应缓存中并发送给Slave
### 6.3 全同步开销
* BGSAVE时间
* RDB传输时间
* 从节点清空数据时间
* 从节点加载RDB时间
* AOF重写时间
## 7 内存淘汰策略
* noeviction：当内存不足以容纳新写入数据时，新写入操作会报错
* allkeys-lru：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的key
* allkeys-random：当内存不足以容纳新写入数据时，在键空间中，随机移除某个key
* volatile-lru：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，移除最近最少使用的key
* volatile-random：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，随机移除某个key
* volatile-ttl：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，有更早过期时间的key优先移除
## 参考
1. https://segmentfault.com/a/1190000016837791#articleHeader4
2. http://try.redis.io/
3. https://coding.imooc.com/class/303.html
4. https://www.jb51.net/article/156147.htm
