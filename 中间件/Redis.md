# 安装



安装gcc

```
yum install gcc-c++ 
```

解压

```
tar -zxvf redis-4.0.14.tar.gz
```

把解压的文件copy到/usr/local/src里面

```
cp -r /root/software/redis-4.0.14 /usr/local/src/
改名
cd /usr/local/src/
mv redis-4.0.14  redis
```

打开/usr/local/src/redis/deps进行编译依赖项

```
cd /usr/local/src/redis/deps
make hiredis lua jemalloc linenoise
```

打开/usr/local/src/redis进行编译

```
cd /usr/local/src/redis
make
```

安装到/usr/local/redis里

```
mkdir /usr/local/redis
make install PREFIX=/usr/local/redis
```

把配置文件移动到/root/myredis目录[目录可以自定义]

mkdir /root/myredis

cp /usr/local/src/redis/redis.conf /root/myredis

验证安装是否成功

```
cd /usr/local/redis/bin
ls
```

使用which命令查看系统里面是否有redis的服务

```
which redis-server
```



## 启动

/usr/local/redis/bin/redis-server  /usr/local/myredis/redis.conf

```
#客户端连接
/usr/local/redis/bin/redis-cli -h 127.0.0.1 -p 7001
```

开机自启

```
vim /etc/rc.local
　　加入
/usr/local/redis/bin/redis-server /root/myredis/redis-conf
文件地址                          运行哪个配置文件
```



## 停止

```
/usr/local/redis/bin/redis-cli shutdown
#或者
pkill redis-server
```





# 配置解析



NETWORK

```shell
##默认情况下，redis在server上所有有效的网络接口上监听客户端连接。绑定多个ip空格分隔
bind 0.0.0.0

protected-mode yes

port 6379

#backlog连接队列，队列总和=未完成三次握手队列 + 已经完成三次握手队列。
#高并发环境下需要高backlog值来避免慢客户端连接问题。注意Linux内核会将这个值减小到/proc/sys/net/core/somaxconn的值，所以需要确认增大somaxconn和tcp_max_syn_backlog两个值来达到想要的效果
tcp-backlog 511

#当客户端闲置多少秒后关闭连接， 0表示关闭该功能
timeout 0

#单位秒，周期性检测客户端是否还处于健康状态，避免服务器一直阻塞，建议60
tcp-keepalive 300
```



GRNERAL

```shell
daemonize no

#通过upstart和systemd管理Redis守护进程，这个参数是和具体的操作系统相关的
supervised no

#pid文件路径。默认/var/run/redis.pid 。
pidfile /var/run/redis_6379.pid。

# 日志级别。debug（记录大量日志信息，适用于开发、测试）verbose（较多日志信息）
#notice（适量日志信息，使用于生产环境）warning（仅有部分重要、关键信息才会被记录）
loglevel notice 

#日志文件位置，空字符串时为标准输出，如果以守护进程模式运行，将会输出到 /dev/null 。
logfile ""   

#是否把日志记录到系统日志
syslog-enabled no

#设置系统日志的id     如syslog-ident redis
syslog-ident

databases 16

#是否一直显示日志
always-show-logo yes
```



SNAPSHOTTING	快照

```shell
#second秒内,changes个keys发生改变则保存一次
save <seconds> <changes>
       save 900 1
       save 300 10
       save 60 10000

#保存失败，redis将停止接受写操作。直到后台保存进程重新启动,将再次允许写操作
#然而要是安装了靠谱的监控，不需要redis停止写操作来通知保存失败，改成 no
stop-writes-on-bgsave-error yes

#是否在dump.rdb压缩字符串，默认设置为yes。节约cpu资源可以设置no，这样的话数据集就可能会比较大。
rdbcompression yes

#是否CRC64校验rdb文件，性能损失10%
rdbchecksum yes

#rdb文件的名字
dbfilename dump.rdb
```



==SECURITY	安全==		6种淘汰策略

```shell
#设置密码
requirepass ****

#最大客户端连接数
maxclients 10000   

#到达内存上限会试图移除内部数据，移除规则通过maxmemory-policy来指定。
#如果无法根据移除规则来移除，或者设置“不允许移除”，则针对那些需要申请内存的指令返回错误信息，比如SET、LPUSH等,但对于无内存申请的指令，仍然会正常响应，如GET。
#主redis在设置内存使用上限时，需要在系统中留出一些内存空间给同步队列缓存，只有在你设置的是“不移除”的情况下，才不用考虑这个因素
maxmemory-policy
			#6种淘汰策略
                  （1）volatile-lru：LRU 过期
                  （2）allkeys-lru： LRU 最近最少原则
                  （5）volatile-ttl：TTL值最小,即最近要过期
                  一般不用
                  （3）volatile-random：过期移除随机key 
                  （4）allkeys-random：移除随机的key
    #默认,需修改     （6）noeviction：不移除。针对写操作，只返回错误信息	
    
#设置样本数量，LRU算法和最小TTL算法并非精确算法而是估算值，可以设置样本的大小，默认会检查这么多个key并选择其中LRU的那个
maxmemory-samples
```



AOF

```shell
#AOF重写期间是否禁止fsync；如果开启该选项，可以减轻文件重写时CPU和硬盘的负载（尤其是硬盘），但是可能会丢失AOF重写期间的数据；需要在负载和安全性之间进行平衡
no-appendfsync-on-rewrite no

#文件重写触发条件
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

#如果AOF文件结尾损坏，Redis启动时是否仍载入AOF文件
aof-load-truncated yes
```





# Redis线程模型

Redis的**网络IO和键值对读写**是由一个线程来完成的，这也是Redis对外提供的主要服务. Redis 通过**IO 多路复用** 来监听来自客户端的大量连接,这让Redis不需要额外创建多余的线程来监听客户端的连接，降低了资源消耗

由于redis并不区分读/写的线程,在高并发环境下进行大数据(IO读写时间长)的操作时,性能会急剧下降

`Redis`的其他功能，比如**持久化、异步删除、集群数据同步**等，其实是由额外的线程执行的



1. 单线程编程更容易维护
2. 单线程模型并不意味着程序不能并发地处理任务,Redis使用**I/O复用**机制并发处理客户端的请求
3. 基于内存,省去了cpu将数据从磁盘复制到内存的时间,**Redis的性能瓶颈是内存/网络带宽**,多线程并不能显著地提升redis的速度,反而会带来**多余的上下文切换和竞争**，死锁等问题



Redis在新版本中加入了一些**异步处理的删除操作**

1. 异步删除指令 `UNLINK`、`FLUSHALL ASYNC` 和 `FLUSHDB ASYNC`,适用于删除超大键值对时,不会在释放内存上消耗过多的时间.对于其他操作依然是单线程执行的
2. 多线程将客户端的数据包反序列化为redis内部模块可以执行的命令
3. 多线程将执行结果发送回客户端





# 基本类型





Select [index]   切换对应下标的数据库

dbsize	当前数据库的key的数量

flushdb	清空当前库

Flushall	清空全部库



* 基本指令
  * keys * 		获取所有的key	可以跟正则
  * info        查看redis 服务器状态和一些统计信息。
  * move key1 		将key移动到其他数据库,目标库有则不能移动
  * type key 		查看key类型
  * del key 		删除key
  * exists key 		是否存在key
  * **expire key seconds 	过期(秒)**
  * **pexpire key milliseconds   毫秒**
  
  * expireat key timestamp   某个时间戳（秒）之后过期；
  * pexpireat key millisecondsTimestamp：某个时间戳（毫秒）之后过期；
  * persist key 	删除过期时间
  * ttl key 		查看还有多少秒过期，-1永不过期，-2已过期
  * monitor      实时监听并返回redis服务器接收到的所有请求信息
  
  

## 底层数据结构

![](image.assets/v2-f79a11556c6a2f103ef7705c38fe2716_720w.webp)



redis的键值对也是用全局哈希表来存储的,在数据量过大时,也会存在hash冲突和rehash的问题

![](image.assets/v2-33f793913cb71a7c3dbbe4ceea926e6d_720w.webp)

对于string类型,查询时间复杂度为O(1)找到哈希桶+O(n)找到哈希桶的元素

对于集合类型,O(1)是不变的,之后要根据集合的底层数据结构进行计算



![](image.assets/v2-bd42e3dccfd1eca98096aa834d0580d6_720w.webp)





### 压缩列表

压缩列表底层用的是数组,但是数组的前三位存放`zlbytes`、`zltail`和`zllen`，分别表示**列表长度**、**列表尾的偏移量**和**列表中的entry个数**

所以访问第一个元素和最后一个元素通过前三个字段很快访问，时间复杂度O(1)，其他元素O(n)

![](image.assets/v2-77910ea4dd3c6976347a516c050b70f8_720w.webp)



### 跳表

跳表是在链表的基础上，增加了多级索引

![](image.assets/v2-7d53712b2807e0996d842796109be119_720w.webp)











### 渐进式rehash

redis有两个全局哈希表,在表1触发rehash时:

1. 给表2分配两倍的表1空间

2. 将表1中的数据重新映射到表2中

   如果在这个阶段客户端的请求改动了某个哈希桶的元素,则将整个哈希桶重新映射到表2,这使得**rehash无需阻塞用户请求**

3. 释放哈希表1的空间



![](image.assets/v2-49cdf54833d91e511fb50d1e362c750f_720w.webp)























## String

redis的string是二进制安全的,可以包含任何数据。如**数字，字符串，jpg图片或者序列化的对象**



| 命令                           | 介绍                             |
| ------------------------------ | -------------------------------- |
| SETNX key value                | 只有在 key 不存在时设置 key 的值 |
| MSET key1 value1 key2 value2 … | 设置一个或多个指定 key 的值      |
| MGET key1 key2 ...             | 获取一个或多个指定 key 的值      |
| STRLEN key                     | 返回 key 所储存的字符串值的长度  |
| INCR key                       | 将 key 中储存的数字值增一        |
| DECR key                       | 将 key 中储存的数字值减一        |
| EXISTS key                     | 判断指定 key 是否存在            |



**计数器** `SET`、`GET`、 `INCR`、`DECR`

**分布式锁** `SETNX key value`





## List

双向链表,list并没有提供`exist`,如果要判断元素是否存在,必须用set



![](image.assets/redis-list.png)

| 命令                         | 介绍                                    |
| ---------------------------- | --------------------------------------- |
| LPUSH key value1 value2 ...  | 插入列表头部n个元素                     |
| BLPUSH key value1 value2 ... | 阻塞地插入                              |
| LPOP                         | 弹出第一个元素                          |
| LSET key index value         | 指定下标赋值                            |
| LLEN key                     | 获取列表元素数量                        |
| lrange key index1 index2     | **分页**取出数据集合,index2=-1时,取所有 |



**消息队列** `lpush` + `rpop`

**栈** `lpush` + `lpop`

**最新动态** `lpush` + `lrange`



## Hash

string类型的键值对映射表 -> ==字段和值必须是字符串==

比较适合存储json,后续可以直接修改指定字段的值



| 命令                                      | 介绍                                                     |
| ----------------------------------------- | -------------------------------------------------------- |
| HSET key **field** value                  | 设置指定哈希表中指定字段的值                             |
| HMSET key field1 value1 field2 value2 ... | 同时将一个或多个 field-value (域-值)对设置到指定哈希表中 |
| HGET key field                            | 获取指定哈希表中指定字段的值                             |
| HMGET key field1 field2 ...               | 获取指定哈希表中一个或者多个指定字段的值                 |
| HGETALL key                               | 获取指定哈希表中所有的键值对                             |
| HEXISTS key field                         | 查看指定哈希表中指定的字段是否存在                       |
| HDEL key field1 field2 ...                | 删除一个或多个哈希表字段                                 |
| HLEN key                                  | 获取指定哈希表中字段的数量                               |
| HINCRBY key field increment               | 对指定哈希中的指定字段做运算操作（正数为加，负数为减）   |



## Set



| 命令                                  | 介绍                                       |
| ------------------------------------- | ------------------------------------------ |
| SADD key member1 member2 ...          | 向指定集合添加一个或多个元素               |
| SMEMBERS key                          | 获取指定集合中的所有元素                   |
| SCARD key                             | 获取指定集合的元素数量                     |
| **SISMEMBER** key member              | 判断指定元素是否在指定集合中               |
| **SINTER** key1 key2 ...              | **交集**                                   |
| SINTERSTORE destination key1 key2 ... | 将给定所有集合的交集存储在 destination 中  |
| **SUNION** key1 key2 ...              | **并集**                                   |
| SUNIONSTORE destination key1 key2 ... | 将给定所有集合的并集存储在 destination 中  |
| **好友推荐（差集）** key1 key2 ...    | **差集**                                   |
| SDIFFSTORE destination key1 key2 ...  | 将给定所有集合的差集存储在 destination 中  |
| SPOP key count                        | **随机**移除并获取指定集合中一个或多个元素 |
| SRANDMEMBER key count                 | **随机**获取指定集合中指定数量的元素       |



**网站UV统计** `SCARD`

**共同好友(交集)** `SINTER`

**好友推荐（差集）** `好友推荐（差集）`

**抽奖(随机)** `SPOP` `SRANDMEMBER`



## Zset

Zset增加了一个**权重参数score**，实现有序排列,还支持通过score范围获取元素



| 命令                                          | 介绍                                                         |
| --------------------------------------------- | ------------------------------------------------------------ |
| ZADD key score1 member1 score2 member2 ...    | 向指定有序集合添加一个或多个元素                             |
| ZCARD KEY                                     | 获取指定有序集合的元素数量                                   |
| ZSCORE key member                             | 获取指定有序集合中指定元素的 score 值                        |
| ZINTERSTORE destination numkeys key1 key2 ... | 将给定所有有序集合的交集存储在 destination 中，对相同元素对应的 score 值进行 SUM 聚合操作，numkeys 为集合数量 |
| ZUNIONSTORE destination numkeys key1 key2 ... | 求并集，其它和 ZINTERSTORE 类似                              |
| ZDIFFSTORE destination numkeys key1 key2 ...  | 求差集，其它和 ZINTERSTORE 类似                              |
| **ZRANGE** key start end                      | 获取指定有序集合 start 和 end 之间的元素（score 从低到高）   |
| ZREVRANGE key start end                       | 获取指定有序集合 start 和 end 之间的元素（score 从高到底）   |
| ZREVRANK key member                           | 获取指定有序集合中指定元素的排名(score 从大到小排序)         |

**微信步数排行榜 / 优先级队列** `ZREVRANGE`



## bitmap

存储连续的01二进制数组,每个元素的下标叫做 offset（偏移量）



| 命令                                  | 介绍                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| SETBIT key offset value               | 当offset超过位图长度,会触发扩容                              |
| GETBIT key offset                     |                                                              |
| BITCOUNT key start end                | 获取[start,end]之间值为1的元素个数                           |
| BITOP operation destkey key1 key2 ... | 对一个或多个 Bitmap 进行运算，可用运算符有 AND, OR, XOR 以及 NOT |



**用户签到** `SETBIT`

**签到统计** `BITCOUNT`



### HyperLogLog

基数计数概率算法,能够**预估**元素的数量,标准误差为0.81%



| 命令                                      | 介绍                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| PFADD key element1 element2 ...           | 添加一个或多个元素到 HyperLogLog 中                          |
| PFCOUNT key1 key2                         | 获取一个或者多个 HyperLogLog 的唯一计数                      |
| PFMERGE destkey sourcekey1 sourcekey2 ... | 将多个 HyperLogLog 合并到 destkey 中，destkey 会结合多个源，算出对应的唯一计数 |

**每日访问ip数统计 / 帖子uv统计** `PFADD` `PFCOUNT`



## Geospatial index

地理空间索引，简称 GEO，基于 Sorted Set 实现



| 命令                                             | 介绍                                                         |
| ------------------------------------------------ | ------------------------------------------------------------ |
| GEOADD key longitude1 latitude1 member1 ...      | 添加一个或多个元素对应的经纬度信息到 GEO 中                  |
| GEOPOS key member1 member2 ...                   | 返回给定元素的经纬度信息                                     |
| GEODIST key member1 member2 M/KM/FT/MI           | 返回两个给定元素之间的距离                                   |
| GEORADIUS key longitude latitude radius distance | 获取指定位置附近 distance 范围内的其他元素，支持 ASC(由近到远)、DESC（由远到近）、Count(数量) 等参数 |
| GEORADIUSBYMEMBER key member radius distance     | 类似于 GEORADIUS 命令，只是参照的中心点是 GEO 中的元素       |













# 持久化

集群时,一般只在master上进行持久化,slave不需要

如果同时开启RDB和AOF,在**恢复数据时优先AOF**,确保数据完整性



## RDB

Redis DataBase  存储数据==快照==

```shell
#second秒内,changes个keys发生改变则保存一次
save <seconds> <changes>
    save 900 1
    save 300 10
    save 60 10000
```







* 优点

  * 适合灾难恢复,**恢复速度快**,可以做**冷备**,定期将完整的快照文件传输到数据中心
  * 性能高,通过fork子线程实现快照,**不影响父进程**

  * 文件小,适合**数据量大的全量复制**

* 缺点
  * **不保证数据完整性和一致性**,两次快照之间存在时间间隔
  * 会先**将数据写入到临时文件中，再替换上次持久化好的文件**,数据集大将导致fork()耗时,AOF也需要fork(),但可以调整重写日志的频率

* 恢复数据
  * 将备份文件 (dump.rdb) 移动到 redis 安装目录并启动服务



### 创建RDB文件

1. `BGSAVE`

Redis会**fork**出一个==子进程==(注意这里不是子线程),然后子进程负责将快照写入硬盘，而父进程则继续处理命令请求

2. `SAVE`

主线程**阻塞**地创建快照文件. 通常只会在没有足够内存去执行BGSAVE命令的情况下，又或者即使等待持久化操作执行完毕也无所谓的情况下，才会使用这个命令



### fork

`fork`是进程用于创建自己拷贝的操作,子进程几乎是父进程的完整副本,只会存在以下区别:

1. 进程ID不同
2. 父进程ID不同
3. 子进程不继承父进程的内存锁

所以==fork子进程的内存和主进程是完全相同的,但又与主进程的内存互相独立==

但并不是在fork时就将主进程的内存进行了全量拷贝,这会导致大量的冗余,甚至是OOM. **fork进程被分配的是虚拟内存,映射到主进程的物理内存**,当主进程进行修改时,主进程才会以页为单位将内存拷贝到fork进程.由于Redis往往是读大于写,所以**写时拷贝**并不会影响redis的性能











## AOF

Append Only File 日志记录所有写操作



默认不开启aof持久化,需要手动改配置文件. 当**同时开启两种持久化方式时,默认使用AOF**文件来恢复原始的数据,确保数据完整

```shell
appendonly yes
```



```shell
auto-aof-rewrite-min-size 64MB // 当文件小于64M时不进行重写
auto-aof-rewrite-min-percenrage 100 // 当文件比上次重写后的文件大100%时进行重写
```



* 优点
  * **日志只追加不修改**,文件不容易损坏，redis启动时会读取该文件，执行全部指令以完成数据的恢复工作
  * **实时性高**,如果开启每个写命令都记录日志,就能确保数据的完整性

* 缺点
  * **文件比RDB大**,文件过大后会触发**重写**
  * 文件系统本身对文件的大小有限制,无法保存过大的文件
  * `appendfsync always`时,写入性能比较差  **如果是集群,可以选择只在slave上开启持久化**
  * 恢复速度慢
  
* 修复AOF文件	Redis-check-aof --fix



### 工作流程

1. **命令追加（append）**：所有的写命令追加到 AOF 缓冲区中

2. **文件写入（write）**：系统调用`write`将 AOF 缓冲区的数据写入到系统内核缓冲区（延迟写）

3. **文件同步（fsync）**：根据对应的持久化方式（`fsync` 策略）,定时系统调用 `fsync` 函数，`fsync` 将阻塞直到写入磁盘完成后返回，保证数据持久化

4. **文件重写（rewrite）**：定期对 AOF 文件进行重写，达到压缩的目的

5. **重启加载（load）**：当 Redis 重启时，可以加载 AOF 文件进行数据恢复



![AOF 工作基本流程](image.assets/aof-work-process.png)





### AOF 重写



```shell
auto-aof-rewrite-percentage 100 #上次rewrite之后的文件大小/目前大小 的百分比,0为禁用自动 AOF 重写
auto-aof-rewrite-min-size 64mb #触发重写的最小存储大小
这两条规则是与逻辑
```



1. 主进程fork出子进程执行``BGREWRITEAOF`

2. 子进程创建新的AOF文件,**读取数据库中的所有键值**,生成插入语句,作为AOF内容的开头

   主进程维护**AOF重写缓冲区**，记录重写过程中执行的写命令

3. 主进程将重写缓冲区中的所有内容追加到新的AOF文件

4. 替换掉旧的AOF文件

**在重写期间的写命令,会重复写入新/老两个aof文件**

![](image.assets/v2-f50f63f8d60530db6ed41fc99d966243_720w.webp)



### fsync策略



|         选项         |                 同步频率                 |
| :------------------: | :--------------------------------------: |
|  appendfsync always  |                每个写命令                |
| appendfsync everysec |                   每秒                   |
|    appendfsync no    | 让操作系统来决定何时同步(Linux一般为30s) |



### 为什么先执行命令再记录日志

Redis并没有MySQL那样的语法检查。所以先写日志在执行命令的话，日志中有可能记录了错误的命令，会影响数据恢复



## RDB VS AOF



|          | RDB                                        | AOF                 |
| -------- | ------------------------------------------ | ------------------- |
|          | 存储压缩过的二进制数据,文件小,适合数据备份 | 存储写命令,文件较大 |
| 恢复数据 | 直接解析还原数据,速度快                    | 依次执行命令,速度慢 |
|          | 没有办法实时或者秒级持久化数据             |                     |

如果丢失短时间内的数据并不会造成太大的影响,可以只用RDB

不建议单独使用AOF,因为时不时地创建一个 RDB 快照可以进行数据库备份、更快的重启以及解决 AOF 引擎错误

如果对数据的安全性要求比较高,可以同时开启AOF和RDB





## 混合持久化 4.0+

```shell
aof-use-rdb-preamble #混合持久化
```

AOF 重写的时候就直接**把 RDB 的内容写到 AOF 文件开头**

好处是可以结合 RDB 和 AOF 的优点, 快速加载同时避免丢失过多的数据

缺点是AOF里面的 RDB 部分是二进制压缩格式，可读性差





# 主从复制  2.8-

哨兵和集群都是在复制基础上实现高可用

持久化保证了redis重启丢失数据，但硬盘损坏会导致数据丢失，通过主从复制可以避免单点故障

==读写分离/容灾恢复/高可用==

slave是与master完全一致的副本,一旦master宕机,可以**手动**选择slave成为新的master,手动化也导致单独的主从复制不具备生产实用性



* 原理

  * Slave启动连接到master后,发送sync命令
  * Master接到命令启动后台的存盘进程，收集修改命令，传送整个数据文件到slave,完成全量复制
    * 全量复制：Master发送RDB文件给slave
    * 增量复制：Master将新的写入命令传给slave
  * slave只读不写	主机宕机后,slave原地待命
  * slave可以作为其他slave的Master, 减轻master的写压力,避免master需要负责太多slave
  * **变更master会清除数据，重新全量复制**
* 缺点
    * 故障恢复无法自动化
    * 写操作无法负载均衡
    * ==每个节点都必须保存完整的数据，扩展能力还是受限于单个节点的存储能力==

 

```shell
拷贝多个redis.conf文件,文件名加上端口号区分
开启daemonize yes
修改从机端口
修改从机Log文件名(可与主机同名)
修从机Dump.rdb文件名

#配置主从		每次与master断开之后，都要重新配置，除非配置进redis.conf文件
SLAVEOF 主库IP 主库端口

#查看当前的主从关系
Info replication

#停止主从，变成主数据库
SLAVEOF no one
```





# 哨兵	2.8+

哨兵模式在主从的基础上,提供了监控主从节点,自动切换master的功能

1. 监控主从节点的运行状态
2. 将Redis实例的运行故障信息通过API通知监控系统
3. **自动故障恢复**：当master故障时，哨兵会自动将某个从节点会升级为主节点，其他从节点会使用新的主节点进行主从复制，通知客户端使用新的主节点
4. **配置中心**：哨兵可以作为客户端服务发现的授权源，客户端连接到哨兵请求给定服务的Redis主节点地址。如果发生故障转移，哨兵会通知新的地址



* 缺点:写操作无法负载均衡；存储能力受单机的限制



```shell
	##	/myredis目录下新建sentinel.conf文件
	## 数字1表示主机挂掉后salve投票,得票1就能当主机.数字设置的越大,投票消耗的时间越长
sentinel monitor 自定义主机名 127.0.0.1 6379 1 
sentinel monitor port6379 127.0.0.1 6379 1

	## 同一哨兵监视多个主机,只需要在配置文件另起一行
	##哨兵配置集群:同一份配置文件复制多分,启动

启动
/usr/local/redis/bin/redis-sentinel /root/myredis/sentinel.conf
```





# 集群	2.8+



==去中心化	去掉路由，自己来路由==

**主从和哨兵只有主机负责写入,从机负责读取,容易造成性能瓶颈****



![](image.assets/image-20201117153234525.png)![](image.assets/image-20201117153237114.png)



| 机器编号 | ip              | port |
| -------- | --------------- | ---- |
| 1        | 192.168.186.128 | 7002 |
| 2        | 192.168.186.128 | 7003 |
| 3        | 192.168.186.128 | 7004 |
| 4        | 192.168.186.128 | 7005 |
| 5        | 192.168.186.128 | 7006 |
| 6        | 192.168.186.128 | 7007 |

Redis集群中内置了16384个哈希槽,可以将哈希槽理解为分表

进行set时,先对key进行crc16算法算出应该插入到哪个槽.进行get同理

 

```shell
新建redis集群文件夹
mkdir redis-cluster
复制一个server
cp /usr/local/redis/bin/redis-server ./

配置第一个redis
mkdir redis1
复制一个配置文件
cp /usr/local/src/redis/redis.conf redis1
修改配置
vim redis1/redis.conf

bind 0.0.0.0                    69行
port 7001                       92行
daemonize yes                   136行
# 打开aof 持久化
appendonly yes                  672行 
# 开启集群
cluster-enabled yes             814行
# 集群的配置文件,该文件自动生成   
cluster-config-file nodes-7001.conf  822行
# 集群的超时时间
cluster-node-timeout 15000         828行

将这个redis1复制6分,每份替换端口号
%s/7000/7001/g

安装完成后同时启动所有的redis
```



```shell
##docker搭建
docker pull inem0o/redis-trib
##启动
docker run -it --net host inem0o/redis-trib create --replicas 1 192.168.186.128:7002 192.168.186.128:7003 192.168.186.128:7004 192.168.186.128:7005 192.168.186.128:7006 192.168.186.128:7007

docker run -it --net host inem0o/redis-trib create --replicas 1 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 127.0.0.1:7006 127.0.0.1:7007

-it是为了可以输入
--net host 是为了上docker容器能连接上本地的宿主机
集群后客户端连接需要加上 -c
```



# Redis Module 4.0+



Module只要编译引入到Redis中就能轻松的实现我们某些需求的功能

- neural-redis 主要是神经网络的机器学
- RedisSearch 主要支持一些富文本的的搜索
- **RedisBloom** 支持分布式环境下的Bloom 过滤器
  - bloomfilter就类似于一个hash set，用于快速判某个元素是否存在于集合中，其典型的应用场景就是快速判断一个key是否存在于某容器，不存在就直接返回。布隆过滤器的关键就在于hash算法和容器大小



# RedisTemplate

## RedisTemplate<Object,Object>



若不设置序列化规则，它将使用JDK自动的序列化将对象转换为字节，存到Redis 里面

它可以存在对象到redis里面

如果对象没有序列化，那么默认使用的JDK的序列化方式

![](image.assets/image-20201117155849546.png)



```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootRedisTemplateTests {
    @Autowired
    private RedisTemplate<Object, Object> redisTemplate ; // 因为创建RedisTemplate 没有使用泛型信息来创建，泛型 本质还是Object，只不过泛型能自动推断并强转
   
    @Test
    public void testString() {
        redisTemplate.setKeySerializer(new  StringRedisSerializer()); // key的序列化用String 因为key 很多时候都是一个字符串
        redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer()); // 优先没有泛型的
        ValueOperations<Object, Object> valueOperations = redisTemplate.opsForValue();
//      valueOperations.set("boot-redis", "boot-value"); //对象->字符串 json
        User user = new User(1, "laolei", "xx.jpg", "78414842@qq.com");
//       KEY : com.sxt.domain.User:1 
//      com.fasterxml.jackson.databind.JsonSerializer 没有依赖jackson 之前大家可能使用spring-boot-web，这里面会自动依赖
        valueOperations.set(User.class.getName()+":"+user.getId(), user);
        
        // 若该对象的强转转换，则redis 内部会使用JackSon 的工具将字符串-> 转换为java 对象 ，那jackson 转换为对象时，需要一个对象的类型 ，其实它已经自动对象的类型了"@class": "com.sxt.domain.User",
        User object = (User)valueOperations.get(User.class.getName()+":"+user.getId());
        System.out.println(object.getName()+":"+object.getIcon()); }
    
    /**
     * hash
     */
    @Test
    public void testHash() {
        redisTemplate.setKeySerializer(new  StringRedisSerializer()); // key的序列化使用String 类型来完成 因为key 很多时候都是一个字符串
        redisTemplate.setHashKeySerializer(new  StringRedisSerializer());
        redisTemplate.setHashValueSerializer(new  StringRedisSerializer()); // 若都是string 则和StringRedisTempalte一样了
        HashOperations<Object, Object, Object> opsForHash = redisTemplate.opsForHash();
        opsForHash.put("redis-hash", "prop1", "value");
    }}
```



集群额外操作

```java
@Test
    public void testCluster() {
        ClusterOperations<Object, Object> opsForCluster = redisTemplate.opsForCluster();
        //关闭集群的7000端口的主机
        opsForCluster.shutdown(new RedisClusterNode("192.168.120.130", 7000));
    }
```



## StringRedisTemplate



StringRedisTemplate extends RedisTemplate<String,String>



```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootRedisApplicationTests {
	@Autowired
	private StringRedisTemplate redisTemplate;
	/**
	 * redis数据类型为String的操作
	 */
	@Test
	public void testString() {
		// 操作String类型
		ValueOperations<String, String> opsValue = redisTemplate.opsForValue();
		// 给redis 里面set 一个key
		opsValue.set("boot", "spring-boot"); // k -v 都是String
		// 从redis 里面获取key
		String value = opsValue.get("boot");
		System.out.println(value);
		// 从redis 里面或多个key
		List<String> asList = Arrays.asList("boot", "alll-menu-data");
		List<String> mulitValues = opsValue.multiGet(asList);
		System.out.println(mulitValues);
		// redis的自动增长
		Long increment = opsValue.increment("boot-incr", 2);// delta 可以+ 任意的数（步长）
		System.out.println(increment);
	}
	@Test
	public void testHash() {
		HashOperations<String, Object, Object> opsForHash = redisTemplate.opsForHash();
		// hset
		opsForHash.put("object-1", "name", "sxt"); // 后面的2 个参数都是object,但是只支持String 类型
		opsForHash.put("object-1", "age", "27"); // 后面的2 个参数都是object,但是只支持String 类型
		opsForHash.put("object-1", "sex", "man"); // 后面的2 个参数都是object,但是只支持String 类型
		Object value = opsForHash.get("object-1", "sex");
		System.out.println(value);
		// 取多个值
		List<Object> multiGet = opsForHash.multiGet("object-1", Arrays.asList("name", "sex"));
		System.out.println(multiGet);
	}
	@Test
	public void testZset() {
		ZSetOperations<String, String> opsForZSet = redisTemplate.opsForZSet();
		// 放到zset集合里面
		opsForZSet.add("lol", "sxt", 2500);
		opsForZSet.add("lol", "lz", 0);
		opsForZSet.add("lol", "ln", 1400);
		opsForZSet.add("lol", "ll", -10);
		opsForZSet.add("lol", "lt", 2700);
		Set<String> rangeAsc = opsForZSet.range("lol", 0, 2); // 通过排序取值 ll lz ln
		System.out.println(rangeAsc);
		Set<String> reverseRange = opsForZSet.reverseRange("lol", 0, 2);// lt lz ln
		System.out.println(reverseRange);
		Set<TypedTuple<String>> tuples = new HashSet<ZSetOperations.TypedTuple<String>>();
		tuples.add(new DefaultTypedTuple<String>("sxt", 1000.00));
		tuples.add(new DefaultTypedTuple<String>("lv", 1200.00));
		tuples.add(new DefaultTypedTuple<String>("lz", 2900.00));
		tuples.add(new DefaultTypedTuple<String>("lt", 100.00));
		// 若redis 存在该key ，则需要数据类型相同，不然报错
		opsForZSet.add("dnf", tuples);}}
```





## boot



![](image.assets/image-20201117154042392.png)

RedisAutoCongiguration	创建对象

RedisProperties	读取配置文件



```shell
#redis的配置
spring:
  redis:
    host: 
    port: 6379
    password: 
    jedis:
      pool:
        max-idle: 20
        max-active: 25
        min-idle: 10   
```



启动类

```
@EnableCaching
```



业务实现类

```java
//查询	cacheNames 缓存的前缀
@Cacheable(cacheNames = "",key = "#id")
//添加	CachePut = jedis.set	key 参数对象/对象属性		result	返回值对象
@CachePut(cacheNames = "",key = "#result.id")
//修改
@CachePut(cacheNames = "",key = "#result.id")
//删除
@CacheEvict(cacheNames = "",key = "#id")

//全局配置缓存
@CacheConfig

注解缓存缓存的是当前注解所在方法的返回值
```



更改序列化方式

```java
@Configuration
public class RedisConfig {
	@Bean
	 public RedisCacheConfiguration redisCacheConfiguration(CacheProperties cacheProperties) {
	        CacheProperties.Redis redisProperties = cacheProperties.getRedis();
	        RedisCacheConfiguration config = RedisCacheConfiguration
	                .defaultCacheConfig();
	        config = config.serializeValuesWith(RedisSerializationContext.SerializationPair
	        		//把默认的jdk的序列化方式变成jackson
	        		.fromSerializer(new GenericJackson2JsonRedisSerializer()));
	        if (redisProperties.getTimeToLive() != null) {
	            config = config.entryTtl(redisProperties.getTimeToLive());
	        }
	        if (redisProperties.getKeyPrefix() != null) {
	            config = config.prefixKeysWith(redisProperties.getKeyPrefix());
	        }
	        if (!redisProperties.isCacheNullValues()) {
	            config = config.disableCachingNullValues();
	        }
	        if (!redisProperties.isUseKeyPrefix()) {
	            config = config.disableKeyPrefix();
	        }
	        return config;    }}
```





## 键值设计



* value
  * JSON存储value
    * 标准，主流数据交换格式
    * 简单，结构清晰，相对于XML来说更轻量，易于解析
    * **语言无关,类型安全**，值是有类型的



* key
  * 系统：基础数据系统   user:sex:1=男 user:sex:0= 女
  * 模块：数据字典    sys:available:1=true  sys:available:0=false
  * 方法：根据数据字典类型查询
  * 参数：字典类型
  * ==抽象出key存储规则==,使用工具去查看时可以看出层级关系

common:sys:sex:1 男

user:1 {id:1.name:小明}



## ~~事务~~

**Redis的单线程决定了它并不需要事务**

隔离性：Redis单线程执行过程中，不会被其他客户端发送来的请求打断

没有隔离级别的概念：队列中的命令没有提交之前都不会被执行，不存在”事务间的可见性”这个让人万分头痛的问题

不保证原子性：redis==不支持回滚==





# 分布式锁



## 单机 setnx

1. 用**setnx**命令 [key,随机数],并设置**过期时间**,防止异常造成死锁. nx: 只有在key不存在才会执行

2. set成功的服务可以执行后续业务
3. 任务处理完后,用`del`指令删除key,也就是释放锁
   1. key过期了, 在删除前需要判断value是否是自己设置的
   2. key未过期, 直接删除



**释放锁时误删key的问题**

| 服务1                      | Redis        | 服务2              |
| -------------------------- | ------------ | ------------------ |
| set key 1,执行业务         |              |                    |
| get key 1,准备`del`释放锁  |              |                    |
|                            | key1自动过期 |                    |
|                            |              | set key 2,执行业务 |
| del key,此时删了服务2的key |              |                    |



**lua脚本保证del的原子性** Redis 在执行 Lua 脚本时，可以以原子性的方式执行，从而保证了锁释放操作的原子性。

```lua
// 释放锁时，先比较锁对应的 value 值是否相等，避免锁的误释放
if redis.call("get",KEYS[1]) == ARGV[1] then
  return redis.call("del",KEYS[1])
else
  return 0
end
```





**单点故障问题** Redis故障将导致整个服务失效

==不能使用主从==

* 主从复制是异步的,主节点故障时,锁信息可能未能同步到从节点
* 从节点选举成功后,没有锁信息,导致锁被其他服务重新获得













## Redisson锁续期

提供了一个专门用来监控和续期锁的 **Watch Dog（ 看门狗）**，如果操作共享资源的线程还未执行完成的话，Watch Dog 会不断地延长锁的过期时间



```java
// 1.获取指定的分布式锁对象
RLock lock = redisson.getLock("lock");
// 2.拿锁且不设置锁超时时间，具备 Watch Dog 自动续期机制
lock.lock();
// 3.执行业务
...
// 4.释放锁
lock.unlock();
```

**只有未指定锁超时时间，才会使用到 Watch Dog 自动续期机制**

```java
// 手动给锁设置过期时间，不具备 Watch Dog 自动续期机制
lock.lock(10, TimeUnit.SECONDS);
```







## 集群 RedLock

由于 Redis 集群数据同步到各个节点是异步的，如果在 Redis 主节点获取到锁后，在没有同步到其他节点时，Redis 主节点宕机了，此时新的 Redis 主节点依然可以获取锁，所以多个应用服务就可以同时获取到锁

![](image.assets/redis-master-slave-distributed-lock.png)



* 获取当前时间T1
* 向n个Redis节点循环使用相同的key和随机值获取锁,并获取锁的超时时间(==远小于锁的有效时间==,为了保证获取锁失败后,能够立刻尝试下一个节点)
* 记录客户端获得锁的总消耗时间T2
  * T2-T1<超时时间 && **半数以上**节点获取锁成功    -> 获得锁,锁的真正有效时间需要减去获取锁所使用的时间
  * 如果未获得锁,向所有Redis发送释放锁的指令

**只要大部分节点存活,依然可以获取/释放锁**.redlock通过直接操作redis节点,避免了主从不一致的问题



redlock在发生进程/线程/时钟的**大延迟**之后,会出现以下三个问题

### 故障重启后,锁未被持久化而重复上锁

有 A、B、C 这三个节点。

1. 客户端 1 在 A，B 上加锁成功。C 上加锁失败
2. 这时节点 B 崩溃重启了，但是由于持久化策略导致客户端 1 在 B 上的锁没有持久化下来。
3. 客户端 2 发起申请同一把锁的操作，在 B，C 上加锁成功
4. 这个时候就又出现同一把锁，同时被客户端 1 , 2 所持有了



方案: **延迟重启delayed restarts**

节点崩溃后，等待大于锁的过期时间（TTL）后再重启。这样保证了节点在重启前所参与的锁都过期. 但这会导致半数节点重启时,任何锁都无法成功,redis会不可用



### 线程暂停的安全隐患

1. 客户端 1 先去申请锁，并且成功获取到锁。之后客户端GC导致长时间的线程暂停
2. 线程暂停期间，客户端 1 获取的锁的超时失效了
3. 客户端 2成功获得锁,开始写入文件
4. 客户端 1 从线程暂停中恢复过来，他并不知道自己的锁过期了，还是会继续执行文件写入操作，导致客户端 2 写入的文件被破坏



方案: Redission的的看门狗机制



### 时钟跳跃后锁提前过期

1. 客户端 1 从 Redis 节点 A, B, C 成功获取了锁。由于网络问题，无法访问 D 和 E
2. 节点 C 时钟发生了向前跳跃，导致C上面维护的锁过期了
3. 客户端 2 从 Redis 节点 C, D, E 成功获取了同一个资源的锁。由于网络问题，无法访问 A 和 B
4. 客户端 1 和 2 都认为自己持有了锁



![图片](image.assets/640)



方案: 

禁止运维人员直接手动修改时钟

从NTP服务收到一个大的时钟更新后，运维应当采取少量多次的方式。多次修改，每次更新时间尽量小







# 缓存带来的问题



* 缓存穿透	缓存中==不存在==的数据被恶意请求
  * 加强参数校验，拦截非法请求
  * 对查询结果为空时,也放入缓存,并且**过期时间短**,避免后续数据库又有值
  * 对一定不存在的key进行过滤。把所有的可能存在的key放到Bitmap，查询时通过bitmap过滤
  * bloom filter,快速判断给定数据是否存在
* 缓存击穿  ==热点key过期==
  * 热点数据**永不过期**
  * 对请求进行**加锁同步**,第一个请求结果会被缓存,后续的请求都能用
    * 用SETNX去加分布式锁，当操作返回成功时，再进行load db的操作并回设缓存；否则，就重试整个get缓存的方法。
* 缓存雪崩     redis不可用/缓存集体失效
  * 对请求**加锁同步** 只允许单线程查询/写入，其他线程等待
  * 二级缓存，A1为原始缓存，A2为拷贝缓存，A1失效时，可以访问A2，**A1缓存失效时间设置为短期，A2设置为长期**
  * 不同key**不同过期时间**，让缓存失效时间点均匀
  * 服务降级,直接返回预定义信息、空值或是错误信息
* 用redis集群确保redis的可用性
  * ==缓存预热==  启动后将部分数据直接缓存
    * 数据量小,工程启动时进行加载缓存
    * 数据量大,设定定时脚本,进行缓存刷新
    * 数据量过大,优先保证热点数据提前加载到缓存
* 缓存降级
  * 缓存失效或缓存服务器宕机时,也不去访问数据库,直接返回默认数据



## 写入并发竞争

用分布式锁(redis/zookeeper),将客户端的写入操作序列化,避免并发写入





## 双写一致性 | 缓存读写策略

无法保证缓存和DB中数据一致



### 旁路缓存 Cache Aside Pattern

查询: 优先读缓存,其次数据库

修改: 优先改数据库,其次**删除缓存**

==缓存可能是查询n张表运算出来的数据,计算比较耗时,所以直接删除==



存在的问题:

1. 因为缓存操作的是内存,数据库操作的是磁盘,存在**缓存已更新,数据库还没更新**的情况,不过可能性比较低

2. 如果换成先改缓存,再改数据库,容易出现数据库还没来得及更新,就又被读出来当缓存了

3. 首次请求数据不在缓存中,导致大量请求直接到数据库  **热点数据提前缓存**



方案

1. 如果业务场景允许短暂的数据不一致,可以给缓存加一个比较短的过期时间,使数据不一致的影响时间缩短
2. 引入分布式锁,保证修改数据库和修改缓存是一个原子性的操作



### 读写穿透 Read/Write Through Pattern

查询: 优先读缓存,读不到的话,由**cache服务**从db加载数据到缓存后,返回响应

修改: 先查缓存中有没有,有的话更新缓存,并由cache服务同步至db;没有的话直接更新db

==cache服务对数据库的修改是同步的==,所以不会有数据不一致的问题



把 cache 视为主要数据存储,并定义了cache服务负责将缓存中的数据同步至db,这样减轻了应用程序的职责

旁路缓存是需要应用服务自己去保持数据库和缓存的数据一致,但读写穿透是用cache服务来做这件事

但**Redis并没有提供cache将数据写入db的功能**,所以这个不常用



### 异步缓存写入 Write Behind Pattern

与读写穿透一样,都是通过**cache服务**来保证数据一致性,但异步缓存写入保证的是**最终一致性**,在异步写入db之前,数据是不一致的



应用场景: Mysql的Innodb Buffer Pool,消息队列的异步写入磁盘,点赞/浏览量这一类数据变化频繁,但一致性要求不高的数据



## 缓存无底洞

为了满足业务要求添加了大量缓存节点，但是性能反而下降了

产生原因：缓存系统通常采用 hash 函数将 key 映射到对应的缓存节点，随着缓存节点数目的增加，键值分布到更多的节点上，导致客户端一次批量操作会涉及多次网络操作，这意味着批量操作的耗时会随着节点数目的增加而不断增大。此外，网络连接数变多，对节点的性能也有一定影响。

- 优化批量数据操作命令
- 减少网络通信次数
- 降低接入成本，使用长连接 / 连接池，NIO





## 主从切换数据丢失



### 异步复制导致数据丢失

主从的复制是异步的,存在部分数据还没复制到`slave`，`master`就宕机了. 此时推举新的slave,将丢弃了未同步的数据



**方案**

比对主从库上的复制进度差值 `master_repl_offset`和`slave_repl_offset`



### 从机同步到过期数据

slave会同步到master上未过期的数据

如果是设置为经过5min过期,由于slave是在异步复制后才获取到这条数据的,存在1min时间差. 这会导致master删除过期数据后,slave上还差1min才会过期,导致数据不同步



**方案**

用EXPIREAT和PEXPIREAT命令，将过期时间设置为具体的时间点



### 脑裂问题导致数据丢失

1. master与slave网络不通了,但master与client网络正常,读写都能正常响应
2. 此时哨兵认为master宕机,并选举slave作为新master.在选举过程中,client仍然在往旧master发送写指令
3. 选举出新master后,旧master会被作为`slave`挂到新的`master`上,这将导致旧master的数据被清空,并进行全量同步. 用户在上个阶段的写操作都将丢失



**方案**

排查客户端日志,查看主从切换的一段时间内,客户端是否还在跟旧master进行通信



## 解决数据丢失



`redis`提供了两个参数配置来限制主库的请求处理

```text
min-slaves-to-write：这个配置项设置了主库能进行数据同步的最少从库数量；
min-slaves-max-lag：这个配置项设置了主从库间进行数据复制时，从库给主库发送ACK消息的最大延迟（以秒为单位）。
```

`min-slaves-to-write`和`min-slaves-max-lag`这两个配置项搭配起来使用，分别给它们设置一定的阈值，假设为N和T。这两个配置项组合后的要求是，主库连接的从库中至少有N个从库，和主库进行数据复制时的`ACK`消息延迟不能超过T秒，否则，主库就不会再接收客户端的请求了。

举个例子，
假设我们将`min-slaves-to-write`设置为1，把`min-slaves-max-lag`设置为12s，把哨兵的`down-after-milliseconds`设置为10s，主库因为某些原因卡住了15s，导致哨兵判断主库客观下线，开始进行主从切换。同时，因为原主库卡住了15s，没有一个从库能和原主库在12s内进行数据复制，原主库也无法接收客户端请求了。这样一来，主从切换完成后，也只有新主库能接收请求，不会发生脑裂，也就不会发生数据丢失的问题了。







# NOSQL



**NoSQL指"不仅仅是SQL"**，泛指非关系型的数据库。强调Key-Value Stores和文档数据库的优点



关系型数据库问题

 1：不能满足高性能查询需求

​	语言和存储结构是面向对象，但是数据库却是关系的，在存储或者查询时，需要做转换。ORM框架可以简化这个过程，但性能低

 2：应用程序规模的变大

​	需要储存更多的数据、服务更多的用户以及需求更多的计算能力



## NoSQL数据库类型



* 键值（Key-Value）数据库

  * 适用场景：

    * 信息和ID挂钩
    * 用户信息，会话、配置文件、参数、购物车

  * 不适用场景：

    * 信息和值挂钩

    * 数据之间有关系。**不能通过两个或以上的键来关联数据**

    * **不支持回滚**

* 面向文档[MongoDB]    数据用XML、JSON或者JSONB等形式存储。

  * 适用场景：1.日志 2.分析
  * 不适用场景：不支持事务

* 列存储[HBASE]  数据储存在列族中，列族存储经常被一起查询的数据

  * 适用场景：
    * 1.日志 
    * 2.博客平台,我们储存每个信息到不同的列族中。举个例子，标签可以储存在一个，类别可以在一个，而文章则在另一个。
  * 不适用场景：1.不支持事务
    * 原型开发。模型设计之初，无法预测它的查询方式，而一旦查询方式改变，我们就必须重新设计列族。

* 图[Neo4J]    适用范围小，主要用于网络拓扑分析 如脉脉的人员关系图等



| RDBMS                      | NoSQL                                              |
| -------------------------- | -------------------------------------------------- |
| 高度组织化结构化数据       | 不仅仅是SQL                                        |
| 结构化查询语言（SQL）      | 没有声明性查询语言                                 |
| 数据和关系存储在单独的表中 | 没有预定义的模式                                   |
| 数据操纵语言，数据定义语言 | 键 - 值对存储，列存储，文档存储，图形数据库        |
| 严格的一致性               | **最终一致性**，而非ACID【原子，一致，隔离，持久】 |
| 基础事务                   | 非结构化和不可预知的数据                           |
|                            | CAP定理【一致性，可用性，容错性】                  |
|                            | 高性能，高可用性和可伸缩性                         |



![](image.assets/image-20201116232318384.png)



## NoSQL数据库



* Memcached

  * 挥发性(临时性)的键值存储,==无法持久化==,**一般作为关系型数据库的缓存来使用**
  * **支持的数据类型少****
  * 有过期功能expires
  * 使用一致性散列(Consistent Hashing)算法来分散数据

* Tokyo Tyrant

  * 持久性的键值存储,用来处理需要持久保存，高速处理的数据
  * 效率高
  * 不支持过期
  * 使用一致性散列(Consistent Hashing)算法来分散数据

* Redis

  * 擅长处理数组类型的数据
  * 可以高速处理时间序列的数据，易于处理集合运算
  * **原子操作**，支持对多个操作合并后的原子性执行
  * 使用一致性散列(Consistent Hashing)算法来分散数据
  * 应用场景
    * 数据缓存（提高访问性能）
    * 会话管理（session cache，保存web会话信息）
    * 排行榜/计数器（NGINX+lua+redis计数器进行IP自动封禁）
    * 消息队列（构建实时消息系统，聊天，群聊）

* MongoDB

  * 面向无需定义表结构的文档数据
  * 通过BSON的形式可以保存和查询任何类型的数据
  * 无法进行JOIN处理，但是可以通过嵌入(embed)来实现同样的功能

  * 使用sharding(范围分割)算法来分散数据





## bin目录

　　redis-benchmark：redis性能测试工具

　　redis-check-aof：检查aof日志的工具

　　redis-check-dump：检查rdb日志的工具

　　redis-cli：连接用的客户端

　　redis-server：redis服务进程









# Redis内存模型



**mem_allocator**：内存分配器，在编译时指定；可以是 libc 、jemalloc或tcmalloc，默认是jemalloc

**used_memory**：Redis分配器分配的内存总量（字节），包括使用的虚拟内存（swap）

**used_memory_rss**：Redis进程占操作系统的内存（字节），与top及ps命令看到的值一致；除了分配器分配的内存之外，还包括进程本身的内存、内存碎片等，**不包括虚拟内存**

mem_fragmentation_ratio：内存碎片比率，used_memory_rss / used_memory



## 内存划分



### **进程本身的内存**

主进程本身占用内存，如代码、常量池等,大约几兆

不会统计在used_memory中



### **缓冲内存**

* 客户端缓冲  存储客户端连接
* 复制积压缓冲  用于部分复制功能
* AOF缓冲    用于AOF重写时，保存最近的写入命令

由jemalloc分配,会统计在used_memory中



### **内存碎片**



分配、回收物理内存过程中产生内存碎片

不会统计在used_memory中





## Redis做异步队列

一般使用list结构作为队列，rpush生产消息，lpop消费消息。当lpop没有消息的时候，要适当sleep一会再重试。缺点：在消费者下线的情况下，生产的消息会丢失，得使用专业的消息队列如rabbitmq等。**能不能生产一次消费多次呢？**使用pub/sub主题订阅者模式，可以实现1:N的消息队列。





## SCAN系列命令注意事项

- SCAN的参数没有key，因为其迭代对象是DB内数据；
- 返回值都是数组，第一个值都是下一次迭代游标；
- 时间复杂度：每次请求都是O(1)，完成所有迭代需要O(N)，N是元素数量；
- 可用版本：version >= 2.8.0；









# 淘汰策略

当内存使用量超出时，会施行数据淘汰策略



- FIFO（First In First Out）：先进先出策略，最先进入的数据被淘汰

- LRU（Least Recently Used）：最近最久未使用策略。保证内存中的数据都是热点数据

- LFU（Least Frequently Used）：最不经常使用策略，优先淘汰一段时间内使用次数最少的数据



- 如果数据呈现幂律分布，也就是一部分数据访问频率高，一部分数据访问频率低，则使用`allkeys-lru`
- 如果数据呈现平等分布，也就是所有的数据访问频率都相同，则使用`allkeys-random`

|                 | 筛选范围                                              | 淘汰规则                                 |
| --------------- | ----------------------------------------------------- | ---------------------------------------- |
| volatile-lru    | 已设置过期时间的数据集                                | 最近最少使用                             |
| allkeys-lru     | 全部数据集                                            | 最近最少使用                             |
|                 |                                                       |                                          |
| volatile-ttl    | 已设置过期时间的数据集                                | 剩余过期时间最短                         |
|                 |                                                       |                                          |
| volatile-random | 已设置过期时间                                        | 任意                                     |
| allkeys-random  | 全部数据集                                            | 任意                                     |
|                 |                                                       |                                          |
| no-eviction     | 不淘汰数据                                            | 当内存不足以容纳新写入数据时，插入会报错 |
|                 |                                                       |                                          |
| 4.0+ 新增       |                                                       |                                          |
| volatile-lfu    | 从已设置过期时间的数据集中挑选 最不经常使用的数据淘汰 |                                          |
| allkeys-lfu     |                                                       |                                          |



# 过期键的删除策略



## 定时删除

在设置键的过期时间的同时创建定时任务，当键达到过期时间，立即执行对键的删除操作

- **优点：**对内存友好，及时释放
- **缺点：**对cpu不友好，影响服务器的响应时间和吞吐量



## 定期删除

每隔一定时间对数据库进行检查，由算法决定删除多少过期键以及检查多少数据库



通过限制删除操作执行的时长和频率来减少删除操作对CPU时间的影响



## 惰性删除

放任键过期不管，每次从键空间获取键时，检查键是否过期，过期则删除键

- **优点：**对cpu友好
- **缺点：**对内存不友好，容易内存泄露







# 布隆过滤器

位图+hash

对key进行**若干次hash**,并将hashcode取模后,在bitMap上标记为1

在进行查找时,将搜索值也进行上述操作,并判断bitMap,为0则一定不存在.为1,则因为存在哈希冲突,可能存在

**hash的次数越多,过滤结果越精准**



**缺陷**

1. 判断为存在的元素,可能实际上不存在
2. **不支持删除元素**,bitMap的一个位置可能对应了多个元素





# 案例分析



## redis服务器性能估算

一个 2 核 `CPU`、`4GB` 内存、`500GB` 磁盘的云主机运行 `Redis`，`Redis` 数据库的数据量大小差不多是 `2GB`，用了 `RDB` 做持久化保证。同时`Redis`主要以写操作为主，读写比例为`2:8`。那么此时做`RDB`持久化有什么风险？

**内存资源风险：**

1. 因为`RDB`是由`fork`子进程做持久化的，而本篇文章提到好多次写时复制这么个概念。那么在持久化的过程中，**写时复制就会重新分配整个实例80%的内存副本**（读写比例为`2:8`）。也就是`2GB * 0.8 = 1.6GB`。那么此时系统一共就`4GB`，接近饱和了。
2. 如果此时父进程又有大量新`key`写入，**而`Redis`中的数据又是保存到内存中的，所以机器的内存很容易被消耗殆尽。**

而这里又可以考虑是否开启`Swap`机制：

- **开启`Swap`机制，那么`Redis`会有一部分数据被换到磁盘上，当`Redis`访问这部分在磁盘上的数据时，性能会急剧下降**，已经达不到高性能的标准。
- 没有开启`Swap`机制，**会直接触发`OOM`，父子进程会面临被系统`kill`掉的风险**

**CPU资源风险：**

1. **子进程在做生成`RDB`快照的过程会消耗大量的`CPU`资源。**
2. 虽然`Redis`处理处理请求是单线程的，但`Redis Server`还有其他线程在后台工作，例如`AOF`每秒刷盘等这些操作。**即其他工作线程还需要占用`CPU`。**
3. 由于机器只有2核`CPU`，这也就意味着父进程占用了超过一半的`CPU`资源，**此时子进程做`RDB`持久化，可能会产生`CPU`竞争，导致的结果就是父进程处理请求延迟增大，子进程生成`RDB`快照的时间也会变长，整个`Redis Server`性能下降。**



`swap` 空间是硬盘上的一块区域。虚拟内存是由可访问的物理内存和`swap space`组成，也即`swap space`是虚拟内存的一部分。**`swap`存储那些暂时不活跃的内存页面。当操作系统决定要给活跃的进程分配物理内存空间并且可利用的物理内存不足时会用到swap space。**

**说白了，就是把一块磁盘空间或者一个本地文件，当做内存来使用（重点）。** 因此即使服务器的内存不足，也可以运行大内存的应用程序。





## 新客/留存客户统计

用一个集合统计所有登陆过的用户,key为 user:login

每天新建一个集合统计当天登录的用户,key为 user:login:yyyyMMdd

```shell
#计算新客  总客户-day1客户=day2新客
SDIFFSTORE user:new:20230222 user:login:20230222 user:login

#次日留存客户   取day1 day2登录用户的交集
SINTERSTORE user:rem:20230222 user:login:20230222 user:login:20230223
```

集合的计算复杂度较高,在数据量比较大时,建议将数据传取到客户端,让客户端来计算,避免阻塞redis实例



##  一个月内签到次数统计

签到与未签到是简单的0,1二值状态,可以用bitmap记录

```shell
SETBIT user:sign:1:202302 0 1 #id为1的用户在2023-02-01签到
SETBIT user:sign:1:202302 1 1 #id为1的用户在2023-02-02签到
#如果当天用户未签到,则位图默认为0,不用做处理

BITCOUNT user:sign:1:202302 #获取一个月内的签到次数
```



## 连续签到的用户数量统计

> 用户数量1e,需要算出10天内有多少用户连续签到



```shell
SETBIT user:sign:20230223 0 1 #创建1e位的bitmap,key为日期,0代表第一位用户
SETBIT user:sign:20230224 1 1
......10天的签到.....

bitop and key1 key2
.....10个bitmap求与......
与运算结果x, 则x二进制中1的个数就是连续签到的用户数
```



## 网页独立访客统计

> UV:Unique Visitor 网页的独立访客,通过IP去重



**Set统计UV**

```shell
SADD page1:uv user1
SCARD page1:uv #返回set中元素的个数
```

当用户数量大时,set里会有千万个用户. 并且每个网页都要用独立的set存储uv,大促时可能会有上万个网页需要统计,这会导致占用过多的内存



**Hset统计UV**

```bash
HSET page1:uv user1 1 #用户ID作为Hset的key,对这个用户 ID 记录一个值“1”，表示一个独立访客
HLEN page1:uv #统计 Hash 集合中的所有元素个数
```

和 Set 类型相似，当页面很多时，Hash 类型也会消耗很大的内存空间



**HyperLogLog统计UV**

HyperLogLog 是一种用于统计基数的数据集合类型,它计算基数所需的空间总是固定的，而且还很小，适合大量数据的统计

每个 HyperLogLog 只需要花费 12 KB 内存，就可以存储 2^64 个元素

```shell
PFADD page1:uv user1 user2 user3 user4 user5 #支持一次性记录多个用户
PFCOUNT page1:uv #统计结果是基于概率完成的，一般存在0.81%的误差
```





![](image.assets/eabc4e6519ac43f39b2a6cde26372ef4.png)









































