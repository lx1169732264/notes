# 概述

分布式协调服务，**高性能的分布式数据一致性的解决方案**



* 数据发布/订阅
  * 推模式  服务器主动往客户端推送信息
  * 拉模式  客户端主动去服务端请求目标数据（定时轮询）

Zookeeper采用两种方式结合：发布者将数据发布到Zookeeper集群节点上，订阅者订阅节点，那么在服务端数据发生变化时，通知客户端去获取这些信息



* 服务注册 提供者启动时，注册临时节点。宕机时，节点会自动的从zookeeper上删除
* 命名服务 顺序节点生成全局唯一ID
* **数据发布/订阅** 监听 ZooKeeper 上节点的变化来实现配置的动态更新
* **分布式锁**：通过创建唯一节点获得分布式锁，当获得锁的一方执行完相关代码或者是挂掉之后就释放锁



## zookeeper的CP

* 顺序一致性：leader内部维护了发送队列,保证发送的顺序性;利用tcp协议保证follower接收时的顺序性. leader广播给follower的消息将**按序分配`cZxid`**,follower发现接收到的消息乱序时,就会重新从leader进行同步
* 原子性：要么整个集群都更新成功，要么全体失败
* 单一系统映像：客户端无论连到哪一台服务器，看到的都是同样的系统视图。这意味着，如果一个客户端在同一个会话中连接到一台新的服务器，它所看到的系统状态不会更老。当服务器故障，导致客户端需要连接其他服务器时，所有滞后于故障服务器的服务器都不会接受请求
* 持久性：更新操作是持久的,不受到服务器故障影响
* **能够在高并发的情况下保证节点创建的全局唯一性**






## 4种znode



每个子目录项都被称作为znode(目录节点)，==znode本身可以存储数据(文件夹本身能存数据)==

* **PERSISTENT	持久化目录节点**    断开连接后，节点依旧存在
* **PERSISTENT_SEQUENTIAL	持久化顺序编号目录节点**  断开连接后，节点依旧存在，进行顺序编号
* **EPHEMERAL	临时节点**  断开连接后，删除
* **EPHEMERAL_SEQUENTIAL	临时顺序编号目录节点**



每个 znode 由**stat**状态信息 + **data**节点存放的数据的具体内容 组成

```bash
get 节点名   #获取节点内容

[zk: 127.0.0.1:2181(CONNECTED) 6] get /dubbo
# 该数据节点关联的数据内容为空
null
# 下面是该数据节点的一些状态信息，其实就是 Stat 对象的格式化输出
cZxid = 0x2
ctime = Tue Nov 27 11:05:34 CST 2018
mZxid = 0x2
mtime = Tue Nov 27 11:05:34 CST 2018
pZxid = 0x3
cversion = 1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 1
```



| znode 状态信息  | 解释                                                         |
| --------------- | ------------------------------------------------------------ |
| cZxid           | create ZXID，即该数据节点被创建时的事务 id                   |
| ctime           | create time，即该节点的创建时间                              |
| mZxid           | modified ZXID，即该节点最终一次更新时的事务 id               |
| mtime           | modified time，即该节点最后一次的更新时间                    |
| pZxid           | 该节点的子节点列表最后一次修改时的事务 id，只有子节点列表变更才会更新 pZxid，子节点内容变更不会更新 |
| **cversion**    | 子节点版本号                                                 |
| **dataVersion** | 数据节点内容版本号，节点创建时为 0                           |
| **aclVersion**  | 节点ACL 版本号                                               |
| ephemeralOwner  | 创建该临时节点的会话的 sessionId；如果当前节点为持久节点，则 ephemeralOwner=0 |
| dataLength      | 数据节点内容长度                                             |
| numChildren     | 当前节点的子节点个数                                         |



### 会话

只不过 `zk` 客户端和服务端是通过 **`TCP` 长连接** 维持的会话机制

会话还有对应的事件，比如 `CONNECTION_LOSS 连接丢失事件`、`SESSION_MOVED 会话转移事件`、`SESSION_EXPIRED 会话超时失效事件` 



## ACL权限控制

AccessControlLists

对于 znode 操作的权限，ZooKeeper 提供了以下 5 种：

- **CREATE** : 能创建子节点
- **READ**：能获取节点数据和列出其子节点
- **WRITE** : 能设置/更新节点数据
- **DELETE** : 能删除子节点
- **ADMIN** : 能设置节点 ACL 的权限

其中尤其需要注意的是，**CREATE** 和 **DELETE** 这两种权限都是针对 **子节点** 的权限控制。

对于身份认证，提供了以下几种方式：

- **world**：默认方式，所有用户都可无条件访问。
- **auth** :不使用任何 id，代表任何已认证的用户。
- **digest** :用户名:密码认证方式：*username:password* 。
- **ip** : 对指定 ip 进行限制









## 集群的角色

ZooKeeper 中没有选择传统的 Master/Slave 概念，而是引入了 Leader、Follower 和 Observer 三种角色

![ZooKeeper 集群中角色](image.assets/zookeeper-cluser-roles.png)ZooKeeper 集群中的所有机器通过一个 **Leader 选举过程** 来选定一台称为 “**Leader**” 的机器，Leader 既可以为客户端提供写服务又能提供读服务。除了 Leader 外，**Follower** 和 **Observer** 都只提供读服务



| 角色     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| Leader   | 为客户端提供读和写的服务，负责投票的发起和决议，更新系统状态。 |
| Follower | 为客户端提供读服务，如果是写服务则转发给 Leader。参与选举过程中的投票。 |
| Observer | 为客户端提供读服务，如果是写服务则转发给 Leader。**不参与选举过程中的投票**。在不影响选举性能的情况下提升集群的读性能 |



```shell
#vim想要成为observer的配置文件
peerType=observer

#在所有 server的配置文件中，修改server.X配置项，在那些observer的节点上加上:observer后缀。
server.1=IP:2181:3181:observer
#这样配置后，所有节点都知道哪些节点是observer
```



### Watcher机制

`Watcher` 为事件监听器，是 `zk` 非常重要的一个特性，很多功能都依赖于它，它有点类似于订阅的方式，即客户端向服务端 **注册** 指定的 `watcher` ，当服务端符合了 `watcher` 的某些事件或要求则会 **向客户端发送事件通知** ，客户端收到通知后找到自己定义的 `Watcher` 然后 **执行相应的回调方法**



![](image.assets/image-20201120144857293.png)





# 安装



```shell
#配置java环境变量

#解压并修改位置
tar -zxvf zookeeper-3.4.14.tar.gz -C /usr/local
cd /usr/local/
mv zookeeper-3.4.14/ zookeeper

#修改配置文件
cd /usr/local/zookeeper/conf
#修改zoo_sample.cfg 为zoo.cfg [一定要改]
mv zoo_sample.cfg  zoo.cfg
#编辑zoo.cfg
vi zoo.cfg

dataDir=/usr/myzk1/data
ClientPort=2181
```



```shell
#启动
/usr/local/zookeeper/bin/zkServer.sh start
#停止
/usr/local/zookeeper/bin/zkServer.sh stop
#查看zk的运行状态
/usr/local/zookeeper/bin/zkServer.sh status
#连接
/usr/local/zookeeper/bin/zkCli.sh
```





# 指令



```shell
create [-s] [-e] path data acl		创建节点
	-s	顺序节点	在节点后面加上数字序号
	-e	临时节点。默认创建持久节点
	path节点路径，data节点数据，acl权限控制

get	/节点	获取指定节点的内容/属性

get	/节点	watch	节点监控

set path data 更新节点内容

delete	/节点		删除节点,存在子节点时无法删除,要用rmr path 删除当前节点及子节点

```





# 配置



```shell
#毫秒	心跳时间	最小的session超时时间为两倍心跳时间
tickTime =2000

#Follower跟随者与Leader初始连接时能容忍的最多心跳数，限定集群中的Zookeeper连接到Leader的时限
initLimit =10	LF连接时限

#Leader与Follower最大响应时间单位，超过syncLimit * tickTime，从服务器列表中删除Follwer。
syncLimit =5	LF同步通信时限

#数据文件目录+数据持久化路径,主要用于保存Zookeeper中的数据
dataDir

#监听客户端连接的端口
clientPort =2181
```



# 集群搭建



| 机器编号 | Ip 地址         | 端口 |
| -------- | --------------- | ---- |
| Zk-1     | 192.168.186.128 | 2181 |
| Zk-2     | 192.168.186.128 | 2182 |
| Zk-3     | 192.168.186.128 | 2183 |



```shell
#复制3份zookeeper,每个创建2个目录,1个文件
#创建data数据目录
mkdir /usr/local/zookeeper-cluster/zk1/data
#创建myid文件,写入id号
vim /usr/local/zookeeper-cluster/zk1/data/myid

1	(id号)

#创建log日志目录
mkdir /usr/local/zookeeper-cluster/zk1/log

#修改配置文件
#数据目录，三个zk数据目录要不同
dataDir=/usr/local/zookeeper-cluster/zk1/data
#日志目录,可以相同
dataLogDir=/usr/local/zookeeper-cluster/zk1/log
# zookeeper 的client的端口号
clientPort=2181

# 集群额外的配置
# 第几个服务器（1，2，3来自数据目录的一个myid文件，该文件里面保存着当前集群的标识（1，2，3））
# 第一个端口：集群内部数据复制的端口 第二个端口代表：选举端口端口可以重复,但复制和选举不能相同端口
server.1=192.168.186.128:2888:3888
server.2=192.168.186.128:2889:3889
server.3=192.168.186.128:2887:3887
```



# 处理请求



集群中的每个server(包括observer)都能为客户端提供读、写服务。

* 对于客户端的读请求，server直接从它本地的内存数据库中取出数据返回给客户端，不联系leader

* 对于客户端的写请求，需要修改znode，必须在集群中进行协调
  * 收到写请求的server将请求发送给leader
  * leader收到写请求后，==首先计算这次写操作之后的状态，然后将请求转换成带有状态的事务(版本号、zxid)==
  * leader将这个事务以提案的方式广播出去(proposal)
  * 所有follower收到proposal后，进行投票，投票完成后**返回ack给leader**
    * 投票两种方式：(1)确认提案；(2)丢弃提案表示不同意
  * leader收集投票结果，投票过半提案通过,leader向所有server发送commit提交通知
  * 所有节点将事务**写入事务日志，并提交**
  * 提交后，收到写请求的那个server向客户端返回成功信息

![](image.assets/image-20201120141058337.png)



# 选举（ZAB协议）

**leader的选举本质上注册一个指定位置的临时节点**,并不断刷新节点的存活时间,follower在这个节点上注册watcher,监听节点的变更通知.

在leader1宕机过程中,临时节点失效的的消息通知到了follower,触发了重新选举,选举出新leader2并重新注册临时节点

leader1恢复后,发现临时节点已被注册,就自动转变为follower



**票据** 在选举过程中,各个server之间传输票据,主要包含3个信息

1. epoch：票据是否过期
2. zxid：要推举节点的事务id
3. myid：要推举节点的节点id



* 初始阶段,所有server都投给自己

* 半数机制：半数以上机器存活，集群可用。==奇数台服务器==,能够**避免脑裂**问题,出现网络分区时不会出现2个leader

* id机制:未决定leader的情况下,`zxid` 大的优先，如果相同那么就 `myid` 大的优先

 ==投票是极其消耗时间的,可以通过serverID从大到小启动,一轮完成投票==

| server1                             | server2               | server3                             |
| ----------------------------------- | --------------------- | ----------------------------------- |
| 启动,myid=1,zxid=0                  | 启动,myid=2,zxid=0    | 未启动                              |
| zxid相同,改投server2 myid=2,zxid=0  | myid=2,zxid=0         |                                     |
|                                     | 成为leader,选举结束   |                                     |
|                                     |                       |                                     |
|                                     |                       | 启动时已有leader,myid=2,zxid=1      |
|                                     |                       | 成为follower                        |
|                                     |                       |                                     |
|                                     | 一段时间后……          |                                     |
| 本地最大zxid=9                      |                       | 宕机,本地最大zxid=9                 |
|                                     | 广播zxid=10的消息     |                                     |
| zxid=10的消息被接收,本地最大zxid=10 |                       |                                     |
|                                     | **宕机**,触发重新选举 |                                     |
| myid=1,**zxid=10**                  |                       | 恢复并参与投票myid=3,**zxid=9**     |
| myid=1,zxid=10                      |                       | zxid较小,改投server1 myid=1,zxid=10 |
| 成为leader,选举结束                 |                       | **从leader同步数据**                |

**消息广播过程中宕机,不会导致数据不一致**

server2宕机时,消息只广播给了server1,但server3没收到消息

此时server1的zxid一定是更大的,成为了leader. 而server3从leader同步了缺失的数据



**leader宕机恢复,丢弃未完成的消息**

在未广播`commit`消息到follower之前,leader1就宕机的话

1. followers回滚事务,并选举出leader2
2. leader1恢复,转变为follower并废弃之前的提案,从leader2中同步数据





# 读写锁的实现

所有创建节点必须有序

当读请求**没有比自己更小的节点，或比自己小的节点都是读请求** ，则可以获取到读锁。**若比自己小的节点中有写请求** ，则当前客户端无法获取到读锁，只能等待前面的写请求完成。

写请求（获取独占锁），若 **没有比自己更小的节点** ，则获取到写锁。若发现 **有比自己更小的节点，无论是读操作还是写操作，当前客户端都无法获取到写锁** ，等待所有前面的操作完成。

**读请求监听比自己小的最后一个写请求节点，写请求只监听比自己小的最后一个节点**













# java



```
		<dependency>
			<groupId>com.101tec</groupId>
			<artifactId>zkclient</artifactId>
			<version>0.11</version>
		</dependency>
```



增删改查

```java
创建create
修改writeData
查询readData
删除delete
public class TestCRUD {
	private static final String SERVERSTRING = "192.168.33.115:2181,192.168.33.115:2182,192.168.33.115:2183";
	private static ZkClient zkClient=null;
	static {zkClient=new ZkClient(SERVERSTRING,10000,10000);}
	public static void main(String[] args) {}
}
```



监听节点之前需要序列化

```java
public class CustomerSerializer implements ZkSerializer {

	private String charset = "UTF-8";
	
	public CustomerSerializer() {
	}
	public CustomerSerializer(String charset) {
		this.charset = charset;
	}
	/**
	 * 序列化
	 */
	@Override
	public byte[] serialize(Object data) throws ZkMarshallingError {
		try {
			byte[] bytes = String.valueOf(data).getBytes(charset);
			return bytes;
		} catch (UnsupportedEncodingException e) {
			throw new ZkMarshallingError("Wrong Charset:" + charset);
		}
	}
	/**
	 * 反序列化
	 */
	@Override
	public Object deserialize(byte[] bytes) throws ZkMarshallingError {
		String result = null;
		try {
			result = new String(bytes, charset);
		} catch (UnsupportedEncodingException e) {
			throw new ZkMarshallingError("Wrong Charset:" + charset);	}
		return result;}}
```



监听

```java
public class TestWatch {
	private static final String SERVERSTRING = "192.168.33.115:2181,192.168.33.115:2182,192.168.33.115:2183";
	private static ZkClient zkClient=null;
	
	static {
		zkClient=new ZkClient(SERVERSTRING,10000,10000,new CustomerSerializer());
	}
	public static void main(String[] args) throws IOException {
		
		//监听当前节点的改变和删除事件
		zkClient.subscribeDataChanges("/sanguo", new IZkDataListener() {
			
			/**
			 * 当dataPath被删除时回调用的方法
			 */
			@Override
			public void handleDataDeleted(String dataPath) throws Exception {
				System.out.println("handleDataDeleted:"+dataPath);
			}
			
			/**
			 * 当dataPath这个节点的数据发生改变时回调用
			 * data:改变这之后的值
			 */
			@Override
			public void handleDataChange(String dataPath, Object data) throws Exception {
				System.out.println("handleDataChange:"+dataPath+"  "+data);
			}
		});
		
		//监听子节的变化
		zkClient.subscribeChildChanges("/sanguo", new IZkChildListener() {
			
			/**
			 * parentPath父节点路径
			 * currentChilds  当前父节点下面的所有子节点
			 */
			@Override
			public void handleChildChange(String parentPath, List<String> currentChilds) throws Exception {
				
			}
		});
		System.out.println("监听启动成功");
		System.in.read();//不让程序结束	}}
```











