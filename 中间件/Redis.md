

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

### 启动

/usr/local/redis/bin/redis-server /root/myredis/redis.conf

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



### 停止

```
/usr/local/redis/bin/redis-cli shutdown
#或者
pkill redis-server
```



## 配置

* daemonize yes	以守护进程方式运行
  * 会把pid写入/var/run/redis.pid文件，可以通过pidfile指定



* timeout 300	客户端闲置多长时间后关闭连接，0表示永不关闭



* loglevel verbose	指定日志级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose

 

\7. 日志记录方式，默认为标准输出，如果配置Redis为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给/dev/null

 logfile stdout

\8. 设置数据库的数量

 databases 16

\9. 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合

 save <seconds> <changes>

 Redis默认配置文件中提供了三个条件：

 save 900 1

 save 300 10

 save 60 10000

 分别表示900秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改。

\10. 指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大

 rdbcompression yes

\11. 指定本地数据库文件名，默认值为dump.rdb

 dbfilename dump.rdb

\12. 指定本地数据库存放目录

 dir ./

\13. 设置当本机为slav服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步

 slaveof <masterip> <masterport>

\14. 当master服务设置了密码保护时，slav服务连接master的密码

 masterauth <master-password>

\15. 设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过AUTH <password>命令提供密码，默认关闭

 requirepass foobared

\16. 设置同一时间最大客户端连接数，默认无限制，Redis可以同时打开的客户端连接数为Redis进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis会关闭新的连接并向客户端返回max number of clients reached错误信息

 maxclients 128

\17. 指定Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的Key，当此方法处理后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis新的vm机制，会把Key存放内存，Value会存放在swap区

 maxmemory <bytes>

\18. 指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为no

 appendonly no

\19. 指定更新日志文件名，默认为appendonly.aof

  appendfilename appendonly.aof

\20. 指定更新日志条件，共有3个可选值： 

 no：表示等操作系统进行数据缓存同步到磁盘（快） 

 always：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全） 

 everysec：表示每秒同步一次（折衷，默认值）

 appendfsync everysec

\21. 指定是否启用虚拟内存机制，默认值为no，简单的介绍一下，VM机制将数据分页存放，由Redis将访问量较少的页即冷数据swap到磁盘上，访问多的页面由磁盘自动换出到内存中

  vm-enabled no

\22. 虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个Redis实例共享

  vm-swap-file /tmp/redis.swap

\23. 将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据都是内存存储的(Redis的索引数据 就是keys),也就是说,当vm-max-memory设置为0的时候,其实是所有value都存在于磁盘。默认值为0

  vm-max-memory 0

\24. Redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，vm-page-size是要根据存储的数据大小来设定的，作者建议如果存储很多小对象，page大小最好设置为32或者64bytes；如果存储很大大对象，则可以使用更大的page，如果不确定，就使用默认值

  vm-page-size 32

25.设置swap文件中的page数量，由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中的，磁盘上每8个pages将消耗1byte的内存。

  vm-pages 134217728

26.设置访问swap文件的线程数,不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，会造成长时间的延迟。默认值为4

  vm-max-threads 4

\27. 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启

 glueoutputbuf yes

\28. 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法

 hash-max-zipmap-entries 64

 hash-max-zipmap-value 512

\29. 指定是否激活重置哈希，默认为开启（后面在介绍Redis的哈希算法时具体介绍）

 activerehashing yes

\30. 指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件

 include /path/to/local.conf













# 数据类型



Redis中不存在表这个概念，首先考虑哪种数据类型适合业务，此外，我们无法像在关系数据库中那样，使用sql来操作Redis中的数据，需要直接使用API发送对应的命令，来操作想要操作的数据



1.字符串	Redis中所有键必须是字符串。

2.list    类似双向链表。

3.hash	Redis数据集本身就可以看做一个哈希，而Reidis的数据对象也可以再次使用哈希，其字段和值必须是字符串

4.set	唯一，无序

5.zset	有序集合



1，单进程单线程

多线程处理可能涉及到锁/线程切换消耗CPU/线程安全三个问题

Redis的单线程可以通过在单机开多个Redis实例来弥补

2， 默认16个数据库，类似数组下表从零开始，初始为零号库

统一密码管理，16个库密码相同

3，切换数据库

Select [index]   切换对应下标的数据库

4，常用基本命令

dbsize查看当前数据库的key的数量

flushdb：清空当前库

Flushall；清空全部库











* 基本指令
  * keys * 		获取所有的key	可以跟正则
  * move key1 		将key移动到其他数据库,目标库有则不能移动
  * randomkey  	从当前数据库中随机返回
  * lpush key a b c	将若干值插入到key(存入list)
  * type key 		查看key类型
  * del key 		删除key
  * exists key 		判断是否存在key
  * expire key 10 	过期(秒)
  * pexpire key 1000 	毫秒
  * persist key 	删除过期时间
  * ttl key 		查看还有多少秒过期，-1永不过期，-2已过期

* String
  * getrange name 0 -1 		字符串分割  0 -1是全部	其中-1等价于n-1,即末尾下标.
  * getset name new_value 修改key对应值，返回旧值
  * mset k1 v1 k2 v2 		批量设置
  * mget key1 key2 批量获取

  * setnx key value 			不存在就插入
  * setrange key index value 从index开始替换value

  * incr age 递增
  * incrby age 10 递增

  * decr age 递减
  * decrby age 10 递减

  * incrbyfloat 增减浮点数
  * append 追加

  * strlen 长度
  * object encoding key  得到key 的类型  string里面有三种编码
    * int	能够用64位有符号整数表示的字符串
    * embstr 长度<=39字节的字符串，性能高
    *  raw  用于>=39字节的

* list
  * lpush mylist a b c 左插入
  * rpush mylist x y z 右插入
  * lrange mylist 0 -1  取出数据集合  0 -1取出所有      0  1取第一个和第二个
    * 可以用lrange实现分页
  * lpop mylist 弹出最后一个元素
  * rpop mylist 弹出第一个元素
  * llen mylist 长度
  * lrem mylist count value 删除

    * count > 0 : 头->尾搜索，移除COUNT个等于VALUE的元素
    * count < 0 : 尾->头搜索
    * count = 0 : 移除所有与 VALUE 相等的值。
  * lindex mylist 2 指定索引的值
  * lset mylist 2 n 索引设值
  * ltrim mylist 0 4    修剪(trim)，不在指定区间之内的元素都将被删除
    * Java分割是左闭右开,redis是左右闭
  * linsert mylist before/after a   在元素前或后插入元素。 当元素不存在或空列表时，不执行任何操作,当key不是列表类型，返回一个错误。
  * rpoplpush list list2
    *  移除列表的最后一个元素，并将该元素添加到另一个列表并返回。

## hash

```
    hset myhash name cxx
         |--字段已经存在，旧值将被覆盖。
    hmget myhash       批量获取
    hgetall myhash     获取所有
    hexists myhash name        是否存在
    hsetnx myhash score 100    存在则不做处理,不存在则设置
    hincrby myhash id 1        按1递增
    hdel myhash name           删除
    hkeys myhash       只取key
    hvals myhash       只取value
    hlen myhash        长度
```

## set

```
    sadd myset redis   添加
    smembers myset     获取所有
    srem myset set1    删除
    sismember myset set1 判断是否存在
    scard key_name     长度
    sdiff | sinter | sunion    差|交|并集
    srandmember 随机获取集合中的元素
    spop 从集合中弹出一个元素
```

## zset

Zset增加了一个**权重参数score**，实现有序排列。

```
    zadd zset 1 one
    zincrby zset 1 one 增长分数
    zscore zset two 获取分数
    zrange zset 0 -1 withscores 范围值
    zrangebyscore zset 10 25 withscores 指定范围的值
    zrangebyscore zset 10 25 withscores limit 1 2 分页
    Zrevrangebyscore zset 10 25 withscores 指定范围的值
    zcard zset 元素数量
    Zcount zset 获得指定分数范围内的元素个数
    Zrem zset one two 删除一个或多个元素
    Zremrangebyrank zset 0 1 按照排名范围删除元素
    Zremrangebyscore zset 0 1 按照分数范围删除元素
    Zrank zset 0 -1 分数最小的元素排名为0
    Zrevrank zset 0 -1 分数最大的元素排名为0
```



# 持久化



RDB【Redis DataBase】	数据快照持久



























# NOSQL

 NoSQL指"不仅仅是SQL"，泛指非关系型的数据库。强调Key-Value Stores和文档数据库的优点。



关系型数据库问题

 1：不能满足高性能查询需求

​	语言和存储结构是面向对象，但是数据库却是关系的，在存储或者查询时，需要做转换。ORM框架可以简化这个过程，但性能低。

 2：应用程序规模的变大

​	需要储存更多的数据、服务更多的用户以及需求更多的计算能力。



## NoSQL数据库类型



* 键值（Key-Value）数据库
  * 适用场景：
    * 储存用户信息，比如会话、配置文件、参数、购物车等。这些信息一般都和ID（键）挂钩，这种情景下键值数据库是个很好的选择。

  * 不适用场景：

    * 通过值查询

    * 需要储存数据之间的关系。**不能通过两个或以上的键来关联数据**

    * **不支持回滚**。

      

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





| RDBMS                          | NoSQL                                              |
| ------------------------------ | -------------------------------------------------- |
| 高度组织化结构化数据           | 代表着不仅仅是SQL                                  |
| 结构化查询语言（SQL）          | 没有声明性查询语言                                 |
| 数据和关系都存储在单独的表中。 | 没有预定义的模式                                   |
| 数据操纵语言，数据定义语言     | 键 - 值对存储，列存储，文档存储，图形数据库        |
| 严格的一致性                   | 最终一致性，而非ACID【原子，一致，隔离，持久】属性 |
| 基础事务                       | 非结构化和不可预知的数据                           |
|                                | CAP定理【一致性，可用性，容错性】                  |
|                                | 高性能，高可用性和可伸缩性                         |





![image-20201116232318384](image.assets/image-20201116232318384.png)



## 常见的NoSQL数据库



* Memcached

挥发性(临时性)的键值存储

一般作为关系型数据库的缓存来使用

具有非常快的处理速度

由于存在数据丢失的可能，所以一般用来处理不需要持久保存的数据

用于需要使用expires时(需要定期清除数据)

使用一致性散列(Consistent Hashing)算法来分散数据

* Tokyo Tyrant

持久性的键值存储

用来处理需要持久保存，高速处理的数据

具有非常快的处理速度

用于不需要定期清除的数据

使用一致性散列(Consistent Hashing)算法来分散数据

* Redis

兼具Memcached和Tokyo Tyrant优势的键值存储

擅长处理数组类型的数据

具有非常快的处理速度

可以高速处理时间序列的数据，易于处理集合运算

拥有很多可以进行原子操作的方法

使用一致性散列(Consistent Hashing)算法来分散数据

* MongoDB

面向无需定义表结构的文档数据

具有非常快的处理速度

通过BSON的形式可以保存和查询任何类型的数据

无法进行JOIN处理，但是可以通过嵌入(embed)来实现同样的功能

使用sharding(范围分割)算法来分散数据



## Redis简介

​	Redis它和Memcache一样，Redis数据缓存在计算机内存中，不同的是，**Memcache只能将数据缓存到内存中，无法自动定期写入硬盘**。而Redis能实现数据的持久化



* 特点
  * 操作都是原子性的，支持对多个操作合并后的原子性执行
  * 支持多种数据结构：string,list,hash,set,zset
  * 持久化，主从复制（集群）
  * 支持过期时间，支持事务，消息订阅。



* 应用场景
  * 数据缓存（提高访问性能）
  * 会话管理（session cache，保存web会话信息）
  * 排行榜/计数器（NGINX+lua+redis计数器进行IP自动封禁）
  * 消息队列（构建实时消息系统，聊天，群聊）









