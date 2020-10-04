传统的关系性数据库已经无法满足快速查询与插入数据的需求。这个时候NoSQL的出现暂时解决了这一危机。它通过**降低数据的安全性，减少对事务的支持，减少对复杂查询的支持**，来获取性能上的提升。

但是，有些场合NoSQL无法满足，所以还是需要使用关系性数据库。此时就需要做数据库集群，为了提高查询性能将一个数据库的数据分散到不同的数据库



Mycat能满足数据库数据大量存储；提高了查询性能;实现读写分离，分库分表



Mysql的表最大存储	500w条	数据

表查询的性能极限	二分查找平均情况下log(n)



==Mycat和Mysql区别==

  可以把上层看作是对下层的抽象，例如操作系统是对硬件的抽象。Mycat对数据库层做一个抽象，来管理这些数据库，而上层应用只需要面对一个数据库层的抽象或者说数据库中间件就行，这就是Mycat的核心作用。**数据库是对底层存储文件的抽象，而Mycat是对数据库的抽象。**



==Mycat原理==

它拦截了用户发送过来的SQL语句，首先对SQL语句做了一些特定的分析，如分片分析，路由分析，读写分离分析，缓存分析等，然后将此sql发往后端的真实数据库，并将返回的结果做适当处理，最终返回给用户



主库将所有的写操作记录在binlog日志中，并生成**log dump线程，将binlog日志传给从库的I/O线程**

**从库生成两个线程**，一个是I/O线程，另一个是SQL线程

 

I/O线程去请求主库的binlog日志，并将binlog日志中的文件写入**relay log（中继日志**）中

SQL线程会读取relay loy中的内容，并解析成具体的操作，来实现主从的操作一致，达到最终数据一致的目的



mycat只能路由，分布，==不能数据同步==，所以要数据同步必做还要使用mysql的读写分离，主从复制



# Mysql主从



Mysql主从又叫Replication、AB复制。

A与B两台机器做主从后，在A上写数据，另外一台B也会跟着写数据，实现数据实时同步

mysql主从是基于**binlog**，主需开启binlog才能进行主从

 

主从3个步骤

* 主创建**同步账户**授权给从
* 主将更改操作记录到binlog里
* 从将主的binlog事件（sql语句） 同步本机上并记录在relaylog里
* 从根据relaylog里面的sql语句按顺序执行



主从模式

* 一主一从

* 主主复制

* 一主多从---扩展系统读取的性能，因为读是在从库读取的

* 多主一从---5.7版本开始支持

* 联级复制



## docker搭建

* docker提取配置文件

docker run --name M1 -p 3307:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7

docker run --name M1S1 -p 3308:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7



* 修改配置文件

mkdir /root/mysqlms		创建配置文件的文件夹

docker cp M1:/etc/mysql/conf.d/docker.cnf m1.cnf 

docker cp M1S1:/etc/mysql/conf.d/docker.cnf m1s1.cnf



vim m1.cnf	主添加2条

server-id=1
log-bin=master.bin



vim m1s1.cnf	从添加server-id

server-id=2



* 配置文件复制进docker

docker cp m1.cnf M1:/etc/mysql/conf.d/docker.cnf

docker cp m1s1.cnf M1S1:/etc/mysql/conf.d/docker.cnf



docker restart M1 M1S1		重启docker



* 创建共享账户给从机

docker exec -it M1 bash    进入镜像

mysql -uroot -p123456

create user 'rep'@'%' identified by '123456';

grant replication slave on *.* to 'rep'@'%';				**replication 复制权限**

**flush privileges;**		刷新权限



==show master status;==

![image-20200920115058613](image.assets/image-20200920115058613.png)



* 配置从机

docker exec -it M1 bash

mysql -uroot -p123456



change master to master_host="120.76.132.188",master_port=3307,master_user="rep",master_password="123456",master_log_file="master.000001",master_log_pos=745;

master_log_file 为主机show master status 的文件名	master_log_pos同理



start slave ;		启动主从

==show slave status \G;==	校验主从状态，有下图2个yes代表配置成功

​			Slave_IO_Running: Yes
​            Slave_SQL_Running: Yes

* 如果第一个是connecting		可能从机上主机的ip端口错误

​		stop slave;		先关闭主从 ，再重新change master

* 第一个是no		检查server-id配置

* 如果第二个no	表示两个数据库并没有同步



## 主从操作规范



* 只能在主机里面执行DML 语句
* 使用navicat时，**不要在从机操作**！！！！会不同步

* 在从机里面可以执行查询语句

* 主机只有一台，但是从机可以有多台



# Mysql集群



### 优点

高可伸缩性：服务器集群具有很强的可伸缩性。 随着需求和负荷的增长，可以向集群系统添加更多的服务器。在这样的配置中，可以有多台服务器执行相同的应用和数据库操作。

高可用性：在不需要操作者干预的情况下，防止系统发生故障或从故障中自动恢复的能力。通过把故障服务器上的应用程序转移到备份服务器上运行，集群系统能够把正常运行时间提高到大于99.9%，大大减少服务器和应用程序的停机时间。

### 缺点

​    我们知道集群中的应用只在一台服务器上运行，如果这个应用出现故障，其它的某台服务器会重新启动这个应用，接管位于共享磁盘柜上的数据区，进而使应用重新正常运转。我们知道整个应用的接管过程大体需要三个步骤：侦测并确认故障、后备服务器重新启动该应用、接管共享的数据区。因此在切换的过程中需要花费一定的时间，原则上根据应用的大小不同切换的时间也会不同，越大的应用切换的时间越长。



mysql集群需要**5台起步**

| 名称 | Ip              | Port |
| ---- | --------------- | ---- |
| M1   | 192.168.149.128 | 3307 |
| M1S1 | 192.168.149.128 | 3308 |
| M1S2 | 192.168.149.128 | 3309 |
| M2   | 192.168.149.128 | 3310 |
| M2S1 | 192.168.149.128 | 3311 |

![image-20200920133844671](image.assets/image-20200920133844671.png)



\#docker run --name M1 -p 3307:3306 -e MYSQL_ROOT_PASSWORD=123456  -d mysql:5.7

\#docker run --name M1S1 -p 3308:3306 -e MYSQL_ROOT_PASSWORD=123456  -d mysql:5.7

docker run --name M1S2 -p 3309:3306 -e MYSQL_ROOT_PASSWORD=123456  -d mysql:5.7

docker run --name M2 -p 3310:3306 -e MYSQL_ROOT_PASSWORD=123456  -d mysql:5.7

docker run --name M2S1 -p 3311:3306 -e MYSQL_ROOT_PASSWORD=123456  -d mysql:5.7



M2是M1的集群，需要额外加上 log_slave_updates=1

不加上去的话，M2从M1复制过来的数据**不会被记录，导致M2S1不同步**

![image-20200920141955521](image.assets/image-20200920141955521.png)



\##docker cp m1.cnf M1:/etc/mysql/conf.d/docker.cnf

\##docker cp m1s1.cnf M1S1:/etc/mysql/conf.d/docker.cnf

docker cp m1s2.cnf M1S2:/etc/mysql/conf.d/docker.cnf

docker cp m2.cnf M2:/etc/mysql/conf.d/docker.cnf

docker cp m2s1.cnf M2S1:/etc/mysql/conf.d/docker.cnf



docker restart M1S2 M2 M2S1



* 修改M1S2         master_log_pos可能有改动，再去查询一次

docker exec -it M1S2 bash

mysql -uroot -p123456

==show master status \G;==	**在M1上查看**

change master to master_host="120.76.132.188",master_port=3307,master_user="rep",master_password="123456",master_log_file="master.000001",master_log_pos=1058;

start slave ;

show slave status \G;





* 修改M2      M2也有从机，需要创建共享账户

docker exec -it M2 bash

mysql -uroot -p123456



create user 'rep1'@'%' identified by '123456';

grant replication slave on *.* to 'rep1'@'%';

flush privileges;



==M2是M1的从机，修改master账户==

==show master status \G;==	**在M1上查看**

change master to master_host="120.76.132.188",master_port=3307,master_user="rep",master_password="123456",master_log_file="master.000001",master_log_pos=1058;

start slave ;

show slave status \G;



* 修改M2S1

==show master status \G;==	**在M2上查看**	**填M2的端口**	**用M2的共享账户rep1**

change master to master_host="120.76.132.188",master_port=3310,master_user="rep1",master_password="123456",master_log_file="master.000002",master_log_pos=154;

start slave ;

show slave status \G;



# Mycat安装



wget http://dl.mycat.io/1.6.7.1/Mycat-server-1.6.7.1-release-20190627191042-linux.tar.gz



* 启动

/usr/local/mycat/bin/mycat start

​	可能会内存不足，vim /usr/local/mycat/conf/wrapper.conf

* 连接

​	Mycat默认端口8066

在conf/server.xml定义了用户名和密码123456

![image-20200920160522595](image.assets/image-20200920160522595.png)



连接时error:138	8066的端口没有放行

​			error：10060	mycat启动失败



# 名词解释



* 逻辑库

实际应用并不需要知道中间件的存在，业务开发人员只需要知道数据库的概念

**数据库中间件可以被看做是一个或多个数据库集群构成的逻辑库**

![image-20200920161108827](image.assets/image-20200920161108827.png)

MYCAT服务区中的TESTDB库，只是逻辑上存在的数据库

在Mycat中逻辑库在{MYCAT_HOME}/conf/schema.xml 用<schema> 标签定义

![image-20200920161241948](image.assets/image-20200920161241948.png)



* 逻辑表

对应用来说，读写数据的表就是逻辑表。

逻辑表的数据来源可以是多个分片库，针对不同的数据分布和管理特点，将逻辑表又分为

1. 分片表

2. 全局表    每张表存放全部数据

3. ER表

4. 非分片表

   在schema.xml使用<table>标签对逻辑表进行定义



* 分片表

每个分片都有表的一部分数据，所有分片数据的合集构成了完整的表数据

```
<table name="company" primaryKey="ID" type="global" dataNode="dn1,dn2,dn3" />
```

如上，定义了3张分片表 dn1,dn2,dn3

```
<dataNode name="dn1" dataHost="localhost1" database="db1" />
<dataNode name="dn2" dataHost="localhost2" database="db2" />
<dataNode name="dn3" dataHost="localhost1" database="db3" />
```



* 分片规则

/conf/rule.xml中进行定义

内置规则：按时间、按自定义数字范围、十进制取模、程序指定，字符串Hash，一致性Hash等等

总体可将这些分片规则分为**离散型和连续型**两种

离散型分片规则数据分布均衡，对数据的处理并发能力强，但是对于分片的扩缩容存在较大的挑战。

连续性分片数据分布较集中，更符合业务特性，但是对数据的处理并发能力受限数据的分布，分片的扩缩容有更好的支持。



* 非分片表

对于数据量小的表，不需要进行数据切分

![image-20200920170150809](image.assets/image-20200920170150809.png)

只指定一个分片节点



* 分片节点

数据切分后，每个表分片所在的数据库就是分片节点，

可以认为一个DB实例就是一个节点

使用<dataNode>进行分片节点的定义

![image-20200920170309386](image.assets/image-20200920170309386.png)



* 节点主机

数据切分后，每个分片节点（dataNode）不一定都会独占一台机器，同一机器上面可以有多个分片数据库，这样一个或多个分片节点（dataNode）所在的机器就是节点主机,为了规避单节点主机并发数限制。

尽量将读写压力高的分片节点（dataNode）均衡的放在不同的节点主机，

schema.xml中使用<dataHost>进行分片节点的定义

```
<dataHost name="M1" maxCon="1000" minCon="10" balance="0"
        writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
   <heartbeat>select user()</heartbeat>
   <writeHost host="hostM1" url="120.76.132.188:3307" user="root"
            password="123456">
      <readHost host="M1S1" url="120.76.132.188:3308" user="root" password="xxx" />
      <readHost host="M1S2" url="120.76.132.188:3309" user="root" password="xxx" />
   </writeHost>
</dataHost>
```



# server.xml

* 端口

```
<property name="serverPort">8066</property>
```



* 账号

```
<user name="root" defaultAccount="true">
   <property name="password">123456</property>
   <property name="schemas">TESTDB</property></user>
```



# schema.xml

* 配置虚拟表

```
 <schema name="TESTDB" checkSQLschema="false" sqlMaxLimit="100">
   <table name="sys_user" primaryKey="ID" dataNode="dn1,dn2,dn3"  rule="sharding-by-intfile" /></schema>
```



* 配置数据节点dataNode

```
<dataNode name="dn1" dataHost="localhost1" database="db1" />
<dataNode name="dn2" dataHost="localhost1" database="db2" />
<dataNode name="dn3" dataHost="localhost1" database="db3" />
```

name 节点名称	dataHost 主机名	database 数据库名



* 配置节点主机dataHost

```
<dataHost name="localhost1" maxCon="1000" minCon="10" balance="0"
        writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
   <heartbeat>select user()</heartbeat>
   <writeHost host="M1" url="120.76.132.188:3307" user="root"
            password="123456">
      <readHost host="M1S1" url="120.76.132.188:3308" user="root" password="xxx" />
      <readHost host="M1S2" url="120.76.132.188:3309" user="root" password="xxx" />
   </writeHost>
</dataHost>
```



## maxCon/minCon

连接数。标签内嵌套的 writeHost、readHost标签都会根据这个实例化连接数



## balance 属性
负载均衡类型，目前的取值有 3 种：

1. balance="0"		不开启读写分离
2. balance="1"，全部的 readHost 与 stand by writeHost 参与select语句的负载均衡，简单的说，当双
主双从模式(M1->S1，M2->S2，并且 M1 与 M2 互为主备)，正常情况下，M2,S1,S2 都参与select语句的负载均衡。
3. balance="2"，所有读操作都随机的在 writeHost、readhost 上分发。
4. balance="3"，所有读请求随机的分发到 wiriterHost 对应的 readhost 执行，writerHost 不负担读压
力，注意 balance=3 只在 1.4 及其以后版本有，1.3 没有



## writeType 属性

废弃，改用switchType



## switchType

- -1 表示不自动切换	
- 1   **默认**，自动切换 
- 2  基于 MySQL 主从同步的状态决定是否     **心跳语句show slave stat**

* 3 基于 MySQL galary cluster 的切换机制（适合集群）心跳语句 show status like ‘wsrep%

## dbType

指定后端连接的数据库类型，目前支持二进制的 mysql 协议，还有其他使用 JDBC 连接的数据库。例如：
mongodb、oracle、spark 等



## dbDriver

指定连接后端数据库使用的驱动Driver，目前可选的值有 native 和 JDBC。

使用 native 的话，支持mysql 和 maridb。

其他使用 JDBC 



# 分片算法



## 枚举 sharding-by-intfile



* 适用场景

  适用于固定分片的场合	如，需要按照省/区保存，而省/区是固定的 



```
<tableRule name="sharding-by-intfile">
   <rule>
      <columns>sharding_id</columns>
      <algorithm>hash-int</algorithm>
   </rule>
</tableRule>

<function name="hash-int"
		class="io.mycat.route.function.PartitionByFileMap">
		<property name="mapFile">partition-hash-int.txt</property>
		<property name="type">0</property>
    	<property name="defaultNode">1</property> 
</function>
```

Columns	数据库字段名

Algorithm	分片算法

<property name="mapFile">partition-hash-int.txt</property>	指定算法文件名

<property name="type">0</property>		type默认为0，0 表示Integer，非零表示String

<property name="defaultNode">1</property> 	默认节点



partition-hash-int.txt

```
10000=0
10010=1
10020=2
```

sharding_id=10000	传入0号节点



## 取模分片



类似于轮循

* 优点：充分利用写入的负载均衡，写入快

* 缺点：写入失败后事务的回滚难

​		写入1，2，3	在3时失败，将导致12一起回滚	而12在不同的数据库



```
<tableRule name="leige-mo-rule">
	<rule>
		<columns>id</columns>
		<algorithm>leige-mo-rule-hash</algorithm>
	</rule>
</tableRule>

<function name="leige-mo-rule-hash" class="io.mycat.route.function.PartitionByMod"> 
    <!-- 有几个节点就配置几个 -->
    <property name="count">3</property> 
</function>
```



## auto-sharding-long



当数据达到指定额度时，才进行分库分表

* 缺点：没有负载均衡效果

* 优点：没有跨区回滚事务的风险



```
<tableRule name="auto-sharding-long">
      <rule>
          <columns>id</columns>
          <algorithm>rang-long</algorithm>
      </rule>
</tableRule>
<function name="rang-long"
        class="io.mycat.route.function.AutoPartitionByLong">
      <property name="mapFile">autopartition-long.txt</property>
</function>
```



autopartition-long.txt

```
0-500M=0      #0-500W条数据在第一分区
500M-1000M=1
1000M-1500M=2
```



## 固定分片hash算法



类似于十进制的求模运算，区别在于是**二进制的操作**,是取id的二进制低10位 。 

* 优点	按照 10进制取模运算，1-10会被分到10个分片，事务控制难，而此算法根据二进制则可能会**分到连续的分片**



```
<tableRule name="sharding-hash">
	<rule>
		<columns>id</columns>
		<algorithm>lx-sharding-hash</algorithm>
	</rule>
</tableRule>

<function name="lx-sharding-hash" class="io.mycat.route.function.PartitionByLong">
		<!-- 多少个dataNode就配置几个 -->
		<property name="partitionCount">2,1</property>
		<property name="partitionLength">256,512</property>
</function>
```

<property name="partitionCount">2,1</property>			拆分为2+1=3个节点
<property name="partitionLength">256,512</property>	节点的容量

![](image.assets/image-20200921155149891.png)

总节点3	2+1<=3

总容量	256*2+512=1024=2^10	总容量必须是2的幂



## 字符串ID分片



jump Consistent hash	JUC一致性哈希算法	零内存消耗，均匀，快速，简洁



```
<tableRule name="jch">
	<rule>
		<columns>id</columns>
		<algorithm>jump-consistent-hash</algorithm>
	</rule>
</tableRule>

<function name="jump-consistent-hash" class="io.mycat.route.function.PartitionByJumpConsistentHash">
   <property name="totalBuckets">3</property>
</function>
```

将字符串key分配给n个buckets



同样有**rehash**的概念	需要**预估最大容量**，并且使用负载因子来确定容器的size



## 自然月分片



```
<tableRule name="sharding-by-month"> 
    <rule> 
        <columns>create_time</columns> 
        <algorithm>sharding-by-month</algorithm> 
    </rule> 
</tableRule> 
<function name="sharding-by-month" class="org.opencloudb.route.function.PartitionByMonth"> 
    <property name="dateFormat">yyyy-MM-dd</property> 
    <property name="sBeginDate">2014-01-01</property> 
</function>
```



# 全局表



一个真实的业务系统中，往往存在大量的类似数据字典表的表，数据字典表具有以下几个特性：

• 数据变动不频繁；

• 数据规模不大，数据量在十万以内；

• **跟其他表（特别是分片表）关联查询**



MyCAT定义的全局表具有以下特性： 

* 全局表有所有数据的一份拷贝。crud时，所有的全局表都将受到影响

* **查询只从一个节点获取** 

* **全局表可以跟任意表进行 JOIN**操作 



schema.xml		type="global"，不指定分片规则

```
<table name="sys_user" type="global" primaryKey="ID" dataNode="dn1,dn2,dn3" />
```



如果不设置type="global"，也不设置路由规则，那么默认所有节点都会存数据

但是查询时会查询所有节点，将所有节点的数据汇总返回（重复数据）

因为是分别查询所有节点，distinct并不会起效



# ER表



子表与父表记录存放在同一个数据分片上，**子表依赖于父表**，通过**表分组**（Table Group）保证数据 Join 不会跨库操作。

这样一种表分组的设计方式是解决跨分片数据 join 的一种很好的思路，也是数据切分规划的重要一条规则。ER表中在schema.xml中使用<childTable>标签进行描述和定义

![image-20200920165844938](image.assets/image-20200920165844938.png)



# 分布式全局ID



* 全局唯一

不能出现重复的ID号，这个是最基础的要求

* 趋势递增	InnoDb引擎中使用的是聚集索引，多数据的RDBMS使用的是**Btree**索引数据，在主键的选择上面我们应该尽量有序保证写入性能

* 单调递增		保证下一个ID一定大于上一个ID。

* 信息安全        如果ID连续，恶意扒取就很容易

* 高可用    服务器不能宕机

* 低延时    毫秒级的生成

* 高QPS（Queries-per-second每秒查询率）：



## replace into插入语局



replace into跟insert类似	都是插入数据的sql语句

replace into首先尝试插入数据列表中，如果ID重复(根据主键或唯一索引判断)则先删除，再插入



## UUID  Universally Unique Identifier



包含32个16进制的数字，为8-4-4-4-12个字符，总长36位



* 优点	

性能高：本地生成，**没有网络消耗**

* 缺点
  * 不易于存储：UUID太长，16字节128位，通常以36长度的字符串表示，很多场景不适用。

  * 信息不安全：**基于MAC地址生成UUID的算法会造成MAC地址泄露**，这个漏洞曾被用于寻找梅丽莎病毒的制作者位置。
  * UUID是无序的，无法进行排序
  * MySQL官方建议主键要越短越好**
  * 对MySQL索引不利：作为数据库主键，**在InnoDB引擎下，UUID的无序性可能会引起数据位置频繁变动，影响性能**



## mycat默认支持的全局ID



### 本地文件方式	不推荐



server.xml		sequnceHandlerType设置为0

```
<property name="sequnceHandlerType">0</property>
```



schema.xml		autoIncrement="true"设置主键自增

```
<table name="sys_user" autoIncrement="true" primaryKey="ID" dataNode="dn1,dn2,dn3" />
```



sequence_conf.properties

```
<!--表名大写-->
HOTNEWS.HISIDS=
HOTNEWS.MINID=1001	//初始
HOTNEWS.MAXID=2000	//最大
HOTNEWS.CURID=1000	//当前
```



mycat启动后，会加载本地文件到内存	之后的操作都是在内存的

这将导致运行后，内存中的CURID当前ID和配置文件的不匹配

进而重启mycat时，读取到的是之前的CURID，造成ID重复

因此不推荐这个方法



### 本地时间戳方式



server.xml		sequnceHandlerType设置为2

```
<property name="sequnceHandlerType">2</property>
```



schema.xml		autoIncrement="true"设置主键自增

```
<table name="sys_user" autoIncrement="true" primaryKey="ID" dataNode="dn1,dn2,dn3" />
```



自带的sequence_time_conf.properties

```
<!--工作区id -->
WORKID=01
<!--数据中心id-->
DATAACENTERID=01
```



* 优点	不存在本地文件方式中，mycat重新发布需要修改sequence_conf的问题

* 缺点	存在**服务器时间波动**问题



### 分布式zookeeper



server.xml		sequnceHandlerType设置为4

```
<property name="sequnceHandlerType">4</property>
```



schema.xml		autoIncrement="true"设置主键**自增**	type="global"设置为**全局**表

```
<table name="sys_user" autoIncrement="true" type="global" primaryKey="ID" dataNode="dn1,dn2,dn3" />
```



conf/myid.properties

```
loadZk=true
zkURL=127.0.0.1:2181
clusterId=mycat-cluster-1
myid=mycat_fz_01
#clusterSize=3
clusterNodes=mycat_fz_01,mycat_fz_02,mycat_fz_04
#type=server
#boosterDataHosts=dataHost1
```



conf/sequence_distributed_conf.properties

```
INSTANCEID=ZK
<!--与myid中配置的clusterId一致-->
CLUSTERID=mycat-cluster-1
```



conf/sequence_conf.properties	**表名大写	配置末尾不能带空格**

```
# 自定义的ID自增规则
SYS_ZK.HISIDS=
SYS_ZK.MINID=1
SYS_ZK.MAXID=2000
SYS_ZK.CURID=0
```

MINID和MAXID决定了分配到一个节点的数据量为2000 ,在插入数据时的分配的ID顺序将是2001 ,4001 ,6001 ,可以充分发挥负载均衡效果 ,但回滚难度大

![image-20200925080029236](image.assets/image-20200925080029236.png)



* 优点	不存在本地文件方式中，mycat重新发布需要修改sequence_conf的问题

* 缺点	存在**服务器时间波动**问题



## 雪花算法

twitter开源分布式ID生成算法



* 优点：
  * **毫秒数在高位**，自增序列在低位，整个ID都是趋势递增的。****
  *  不依赖数据库等第三方系统，以服务的方式部署，稳定性更高，生成ID的性能也是非常高的。
  * 可以自定义工作机器ID的bit位，非常灵活。

* 缺点
  * 强依赖机器时钟，如果机器上时钟回拨，会导致发号重复或者服务会处于不可用状态

  * 此算法2039年废弃

  * 只适合规模较大的项目

    

![](image.assets/QQ拼音截图20200925080825.png)

1位符号位 ,ID不能为负数 ,符号位只能为0

**41位时间戳 ,最大表示到2039年**

10位工作机器ID =数据中心ID(n)+工作中心ID(m)	可以分成n+m<=10 ,最多2^n个数据

12位序列号 ,毫秒内的计数（每个节点每毫秒产生2^12个ID）每秒生成409.6万个



**共计==64==位，为一个Long型。(转换成字符串长度为18)**



```
public class SnowflakeIdWorker {
    
    /**
     * 开始时间截 (2015-01-01)
     */
    private final long twepoch = 1420041600000L;

    /**
     * 机器id所占的位数
     */
    private final long workerIdBits = 5L;

    /**
     * 数据标识id所占的位数
     */
    private final long datacenterIdBits = 5L;

    /**
     * 支持的最大机器id，结果是31 (这个移位算法可以很快的计算出几位二进制数所能表示的最大十进制数)
     */
    private final long maxWorkerId = -1L ^ (-1L << workerIdBits);

    /**
     * 支持的最大数据标识id，结果是31
     */
    private final long maxDatacenterId = -1L ^ (-1L << datacenterIdBits);

    /**
     * 序列在id中占的位数
     */
    private final long sequenceBits = 12L;

    /**
     * 机器ID向左移12位
     */
    private final long workerIdShift = sequenceBits;

    /**
     * 数据标识id向左移17位(12+5)
     */
    private final long datacenterIdShift = sequenceBits + workerIdBits;

    /**
     * 时间截向左移22位(5+5+12)
     */
    private final long timestampLeftShift = sequenceBits + workerIdBits + datacenterIdBits;

    /**
     * 生成序列的掩码，这里为4095 (0b111111111111=0xfff=4095)
     */
    private final long sequenceMask = -1L ^ (-1L << sequenceBits);

    /**
     * 工作机器ID(0~31)
     */
    private long workerId;

    /**
     * 数据中心ID(0~31)
     */
    private long datacenterId;

    /**
     * 毫秒内序列(0~4095)
     */
    private long sequence = 0L;

    /**
     * 上次生成ID的时间截
     */
    private long lastTimestamp = -1L;

    //==============================Constructors=====================================

    /**
     * 构造函数
     *
     * @param workerId     工作ID (0~31)
     * @param datacenterId 数据中心ID (0~31)
     */
    public SnowflakeIdWorker(long workerId, long datacenterId) {
        if (workerId > maxWorkerId || workerId < 0) {
            throw new IllegalArgumentException(String.format("worker Id can't be greater than %d or less than 0", maxWorkerId));
        }
        if (datacenterId > maxDatacenterId || datacenterId < 0) {
            throw new IllegalArgumentException(String.format("datacenter Id can't be greater than %d or less than 0", maxDatacenterId));
        }
        this.workerId = workerId;
        this.datacenterId = datacenterId;
    }

    // ==============================Methods==========================================

    /**
     * 获得下一个ID (该方法是线程安全的)
     *
     * @return SnowflakeId
     */
    public synchronized long nextId() {
        long timestamp = timeGen();

        //如果当前时间小于上一次ID生成的时间戳，说明系统时钟回退过这个时候应当抛出异常
        if (timestamp < lastTimestamp) {
            throw new RuntimeException(
                    String.format("Clock moved backwards.  Refusing to generate id for %d milliseconds", lastTimestamp - timestamp));
        }

        //如果是同一时间生成的，则进行毫秒内序列
        if (lastTimestamp == timestamp) {
            sequence = (sequence + 1) & sequenceMask;
            //毫秒内序列溢出
            if (sequence == 0) {
                //阻塞到下一个毫秒,获得新的时间戳
                timestamp = tilNextMillis(lastTimestamp);
            }
        }
        //时间戳改变，毫秒内序列重置
        else {
            sequence = 0L;
        }

        //上次生成ID的时间截
        lastTimestamp = timestamp;

        //移位并通过或运算拼到一起组成64位的ID
        return ((timestamp - twepoch) << timestampLeftShift)
                | (datacenterId << datacenterIdShift)
                | (workerId << workerIdShift)
                | sequence;
    }

    /**
     * 阻塞到下一个毫秒，直到获得新的时间戳
     *
     * @param lastTimestamp 上次生成ID的时间截
     * @return 当前时间戳
     */
    protected long tilNextMillis(long lastTimestamp) {
        long timestamp = timeGen();
        while (timestamp <= lastTimestamp) {
            timestamp = timeGen();
        }
        return timestamp;
    }

    /**
     * 返回以毫秒为单位的当前时间
     *
     * @return 当前时间(毫秒)
     */
    protected long timeGen() {
        return System.currentTimeMillis();
    }

    public static void main(String[] args) {
        SnowflakeIdWorker idWorker = new SnowflakeIdWorker(0, 0);
        for (int i = 0; i < 1000; i++) {
            long id = idWorker.nextId();
//            System.out.println(Long.toBinaryString(id));
            System.out.println(id);
        }
    }
}
```



## hutool分装的雪花算法



```
public class IdGeneratorSnowflake {

    private Log log = LogFactory.getCurrentLogFactory().createLog("IdGeneratorSnowflake");

    private long workId = 0;
    private long datacenterId = 1;
    private Snowflake snowflake = IdUtil.createSnowflake(workId, datacenterId);

    public static void main(String[] args) {
        IdGeneratorSnowflake snowflake = new IdGeneratorSnowflake();
        snowflake.init();
        for (int i = 0; i < 100; i++) {
            System.out.println(snowflake.snowflakeId());
        }
    }

    //@PostConstruct//启动项目时加载
    public void init() {
        try {
            workId = NetUtil.ipv4ToLong(NetUtil.getLocalhostStr());
            log.info("当前机工的workdId:" + workId);
        } catch (Exception e) {
            e.printStackTrace();
            log.warn("当前机器的workID获取失败", e);
            workId = NetUtil.getLocalhostStr().hashCode();
        }
    }

    /**
     * 生成Id
     */
    public synchronized long snowflakeId() {
        return snowflake.nextId();
    }
}
```



<img src="image.assets/QQ拼音截图20200926001206.png" style="zoom:150%;" />













