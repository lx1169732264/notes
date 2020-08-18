# 1 MongoDB介绍

## 1.1 应用场景

传统的关系型数据库（如MySQL），在应对当下互联网产品的某些需求面前，显得力不从心

* 数据库高写入负载需求
* 对海量数据的高效率存储和读取
* 对数据库的高扩展性的需求

这些场景中，MySQL处理起来显得很麻烦，**而MongoDB应对起来则很灵活。**

***

具体的应用场景：

视频直播：使用MongoDB存储点赞、评论、弹幕信息。

游戏场景：使用MongoDB存储用户积分、打怪记录、NPC对话记录。

社交场景：使用MongoDB存储朋友圈信息、浏览记录、评论、转发、收藏、附近的人、聊天记录。

电商场景：使用MongoDB存储收藏、购物车、浏览历史。

****

这些场景中，数据操作方面的共同特点是：

1. 数据量大
2. 写入操作频繁
3. 数据价值较低，能接受少量的数据丢失和不同步
4. 查询频率可能很频繁或者频率很低。

## 1.2 什么时候选择MongoDB

在架构选型上，除了上面4个特点外，还有什么场景可以使用到MongoDB？

* 业务不需要事务或者不需要复杂的连表查询
* 快速迭代开发，需求变更较快，表结构常常会因此修改
* 系统需要应对高达3000QPS（或者更高）的读写
* 业务数据需要GB、TB甚至PB级别的存储
* 系统接受数据在允许的范围内丢失
* 系统需要大量数据的查询，如地理位置。

如果遇到了以上的场景，就可以考虑MongoDB。当然你可以继续使用MySQL，这样可以降低引入一门新技术带来的维护、学习成本，但是以上场景使用MySQL往往会使系统存在瓶颈。

## 1.3 MongoDB简介

MongoDB是一个基于分布式文件存储的数据库。由C++语言编写。旨在为WEB应用提供可扩展的高性能数据存储解决方案。

MongoDB是一个介于关系数据库和**非关系数据库**之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。它支持的数据结构非常松散，是类似json的bson格式，因此可以存储比较复杂的数据类型。Mongo最大的特点是它支持的查询语言非常强大，其语法有点类似于面向对象的查询语言，几乎可以实现类似关系数据库单表查询的绝大部分功能，而且还支持对数据建立索引。

***

MongoDB特点

MongoDB 最大的特点是他支持的查询语言非常强大，其语法有点类似于面向对象的查询语言，几乎可以实现类似关系数据库 **单表查询的绝大部分功能** ，而且还支持对数据建立索引。它是一个面向集合的,模式自由的文档型数据库。具体特点总结如下： 

（1）面向集合存储，易于存储对象类型的数据 

（2）模式自由 

（3）支持动态查询 

（4）支持完全索引，包含内部对象 

（5）支持复制和故障恢复 

（6）使用高效的二进制数据存储，包括大型对象（如视频等） 

（7）自动处理碎片，以支持云计算层次的扩展性 

（8）支持 Python，PHP，Ruby，Java，C，C#，Javascript，Perl 及 C++语言的驱动程序，社区中也提供了对Erlang 及.NET 等平台的驱动程序 

（9） 文件存储格式为 BSON（一种 JSON 的扩展） 

## 1.4 MongoDB体系结构

MongoDB 的逻辑结构是一种层次结构。主要由：文档(document)、集合(collection)、数据库(database)这三部分组成的。逻辑结构是面向用户的，用户使用 MongoDB 开发应用程序使用的就是逻辑结构。 

（1）MongoDB 的文档（document），相当于关系数据库中的一行记录。 

（2）多个文档组成一个集合（collection），相当于关系数据库的表。 

（3）多个集合（collection），逻辑上组织在一起，就是数据库（database）。 

（4）一个 MongoDB 实例支持多个数据库（database）

![image-20200802195154600](https://ydsmarkdown.oss-cn-beijing.aliyuncs.com/md/20200802195155.png)

## 1.5 数据类型

| 数据类型   | 简介                                                         | 举例                                        |
| ---------- | ------------------------------------------------------------ | ------------------------------------------- |
| 字符串     | 字符串类型的数据                                             | {"name": "张三"}                            |
| 对象id     | 对象id是文档的12字节的唯一ID，一般key是固定的_id             | {"_id": ObjectId()}                         |
| 布尔值     | true/false                                                   | {"state": true}                             |
| 数组       | 值的集合，类似于json中的数组                                 | {"hobby": ["唱", "跳", "rap", "篮球"]}      |
| 64位浮点数 | BSON中数字只支持这种类型，没有整数类型。但是可以使用NumberInt(4字节整数)或者NumberLong(8字节符号整数)来表示整数 | {"score": 150},<br />{"age": NumberInt(18)} |
| null       | 表示空值或者未定义的对象                                     | {"sex": null}                               |
| undefined  | 表示未定义的对象                                             | {"sex": undefined}                          |
| 正则表达式 | BSON的正则采用JavaScript的正则表达式语法                     | {"pattern": /\d/}                           |
| 代码       | BSON可以包含JavaScript代码                                   | {"code": function() {xxx}}                  |

## 1.6 MongoDB的特点

1. **高性能**

   MongoDB提供高性能的数据持久性，对嵌入式数据模型的支持减少了数据库系统上的IO活动。

   支持索引查询，使查询的速度更加提高

   支持如mmapv1、wiredtiger等多种存储引擎

2. **高可用**

   MongoDB的复制工具称为副本集，它可以提供自动故障转移和数据冗余

3. **高扩展**

   MongoDB提供了水平可扩展性作为核心功能的一部分。

   分片将数据分布在一组集群的机器上。

   从3.4版本开始，MongoDB支持基于分片键创建数据区域，在一个平衡的集群中，MongoDB将一个区域所覆盖的读写只定向到该区域内的那些片

4. **丰富的查询支持**

   MongoDB支持丰富的查询语言，如数据聚合、文本搜索、地理空间查询等。

# 2 安装

[下载地址](https://www.mongodb.com/try/download/community)

![image-20200802201100423](https://ydsmarkdown.oss-cn-beijing.aliyuncs.com/md/20200802201100.png)

MongoDB的版本号是x.y.z，其中x是大版本，y为奇数的时候是开发板，y为偶数的时候是稳定版，z是修正版本号，z越大往往意味着bug越少。当前版本是4.4.0，不建议下载该版本，推荐下载4.0.19版本。

如果是Linux则下载Linux版本

## 2.1 windows系统安装

下载windows版本的zip包，放到一个目录中，解压到当前目录。

在解压目录中，手动创建一个目录存放数据文件，如data/db

**命令行启动**

在bin目录中按住键盘 **shift** ，右击鼠标，选择在此处打开命令窗口（如果打开的是powershell就需要手动进cmd切换到bin目录），然后输入如下命令：

```sh
mongod --dbpath=../data/db
```

mongodb默认的端口是27017，如果想改变默认的端口，可以在启动的时候使用 **--port** 来指定端口。

为了方便启动，可以将bin目录配置到环境变量中，这样就可以在任何地方使用mongodb的命令了。

**配置文件方式启动服务**

在解压目录中新建config文件夹，在里面新建配置文件 `mongod.conf`，内容如下

```yaml
storage:
  dbPath: H:\\mongodb\\data
```

注意：路径中不可以有Tab字符，路径有空格需要加双引号，双引号需要转义，对于\字符也需要换成\\\或者/

接着使用命令启动

```sh
mongod -f ../config/mongod.conf
或者
mongod --config ../config/mongod.conf
```

## 2.2 Linux系统安装

Linux部署MongoDB操作和Windows差不多

官网下载Linux版的tgz包，上传到Linux目录中，解压到当前目录

```sh
tar -xvf mongodb-linux-x86_64-4.0.19.tgz
```

移动解压后的文件夹到指定目录

```sh
mv mongodb-linux-x86_64-4.0.19 /usr/local/mongodb
```

新建数据目录和日志目录

```sh
# 数据存储目录
mkdir -p /mongodb/data/db
# 日志存储目录
mkdir -p /mongodb/log
```

新建配置文件，并修改内容

```sh
vi /mongodb/mongod.conf
```

配置文件内容

```yaml
systemLog:
  # MongoDB日志输出到文件
  destination: file
  # 日志文件路径
  path: "/mongodb/log/mongod.log"
  # 当mongod实例重新启动时，将新条目附加到现有日志文件的末尾
  logAppend: true
storage:
  # mongod实例存储数据的目录
  dbPath: "/mongodb/data/db"
  journal:
    # 启用或禁用持久性日志
    enabled: true
processManagement:
  # 启用在后台运行
  fork: true
net:
  # 这里是个坑，只能配置0.0.0.0或者本机ip。网上很多教程都是复制粘贴，很不负责任。如果需要限制ip访问，请使用云服务器的安全组
  bindIp: 0.0.0.0
  # 绑定的端口，默认是27017
  port: 27017
```

启动服务

```sh
/usr/local/mongodb/bin/mongod -f /mongodb/mongod.conf
```

启动后显示successfully，如果没有就说明配置文件有问题，启动失败了。

```sh
[root@iZ2zebq96d1zohii1vadzqZ mongodb]# /usr/local/mongodb/bin/mongod -f /mongodb/mongod.conf 
about to fork child process, waiting until server is ready for connections.
forked process: 1526
child process started successfully, parent exiting
```



查看服务是否启动成功

```sh
ps -aux|grep mongod
```

Linux下关闭MongoDB步骤（极不建议使用kill强制杀死进程，这样数据会存在损坏）

```sh
# 客户端登录服务
mongo --port 27017
# 切换到admin库
use admin
# 关闭服务
db.shutdownServer()
```

## 2.3 Docker安装

安装docker

```sh
yum -y install gcc
yum -y install gcc-c++
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum makecache fast
yum -y install docker-ce
systemctl start docker

上面全部复制，一起粘贴即可
下面是配置国内仓库，可以不进行

mkdir -p /etc/docker
vim  /etc/docker/daemon.json
在文件中添加如下配置
 #阿里云镜像加速器{"registry-mirrors": ["http://hub-mirror.c.163.com"] }
systemctl daemon-reload
systemctl restart docker

```



先创建目录存放数据、日志、配置文件

```sh
# 进入local
cd /usr/local
# 创建docker目录，以后的挂载文件都放到这里
mkdir -p docker
cd docker
mkdir -p mongodb
# 进入mongodb目录
cd mongdob
# 创建data、log、conf三个文件夹
mkdir data log conf
chmod 777 data
touch log/mongod.log
chmod 777 log/mongod.log
# 创建配置文件
vi conf/mongod.conf
```

配置文件中添加如下配置

```yaml
systemLog:
  # MongoDB日志输出到文件
  destination: file
  # 日志文件路径
  path: "/usr/local/docker/mongodb/log/mongod.log"
  # 当mongod实例重新启动时，将新条目附加到现有日志文件的末尾
  logAppend: true
storage:
  # mongod实例存储数据的目录
  dbPath: "/usr/local/docker/mongodb/data/db"
  journal:
    # 启用或禁用持久性日志
    enabled: true
processManagement:
  # 启用在后台运行
  fork: true
net:
  # 服务实例绑定的IP，默认是localhost，这里设置的是哪些ip可以访问
  bindIp: 0.0.0.0
  # 绑定的端口，默认是27017
  port: 27017
```

创建mongodb容器

```sh
docker run -di --name mongodb --restart=always --privileged -p 27017:27017 -v /usr/local/docker/mongodb/data:/data/db -v /usr/local/docker/mongodb/conf:/data/configdb -v /usr/local/docker/mongodb/log:/data/log/ mongo:latest -f /data/configdb/mongod.conf
```

> --restart=always Docker服务重启容器也启动
>
> --privileged -p 拥有真正的root权限
>
> -f 指定配置文件（注意，这里指定的是容器内路径，如果需要使用外部配置文件，则需要将容器内路径映射到外面）

## 2.4 连接mongodb

### 2.4.1 命令行连接

在命令行界面输入一下命令完成连接并登录

```sh
mongo
或者
mongo --host=ip地址 --port=27017
```

如果是docker，需要先进入docker容器

```sh
docker exec -it mongodb bash
```

### 2.4.2 Compass客户端连接

到官网下载compass [下载地址](https://www.mongodb.com/download-center/v2/compass?initial=true)

直接解压运行，界面上输入连接信息点击连接

![image-20200802214314158](https://ydsmarkdown.oss-cn-beijing.aliyuncs.com/md/20200802214314.png)

### 2.4.3 Navicat连接

Navicat安装、破解教程在资料中已经提供

Navicat不是MongoDB官方的连接软件，但是很好用，只需要下载12.1以上的navicat即可。

点击连接，选择mongodb

# 3 常用命令

## 3.1 数据库操作

### 3.1.1 选择和创建数据库

```sh
use 数据库名称
```

>如果数据库不存在就自动创建，如果存在则选择该数据库
>
>以下数据库名是保留的，可以直接访问
>
>admin：这是root数据库，如果将一个用户添加到这个数据库，这个用户就自动继承所有数据库的权限。一些特定的命令，如关闭数据库，只能在这个数据库运行
>
>local：这个数据库永远不会被复制，可以用来存储本地单台服务器的任意集合
>
>config：当mongodb分片设置时，config数据库用于保存分片信息

### 3.1.2 查看当前有权限查看的所有数据库

```sh
show dbs
或者
show databases
```

### 3.1.3 查看当前正在使用的数据库

```sh
db
```

> MongoDB默认的数据库是test，如果不选择数据库，集合就默认存放到test

### 3.1.4 删除数据库

```sh
db.dropDatabase()
```

> 该命令执行前需要进入一个数据库。删除的是当前数据库

## 3.2 集合操作

> 注意：MongoDB中，集合只有在内容被插入后才会创建。因此创建一个集合后，需要再插入一个文档。

集合，与MySql中的表类似

### 3.2.1 创建集合

```sh
db.createCollection(集合名称)
```

比如创建一个叫user的集合

```sh
db.createCollection("user")
```

接着查看当前数据库中的集合

```sh
show tables
或者
show collections
```

> 注意：
>
> 1. 集合名称不能为空，也不可以是空字符串。
> 2. system.是系统集合的前缀，创建集合不可以以这个为开头
> 3. 一般我们并不会使用显式的创建集合，因为当向一个集合中插入一个文档的时候，**如果集合不存在，就会自动创建集合**

### 3.2.2 集合删除

```sh
du.集合名称.drop()
```

如果删除成功，该方法返回true，否则返回false

## 3.3 文档增删改

### 3.3.1 文档插入

文档类似于MySql中的row，表示一行数据。MongoDB中文档的数据结构和JSON基本一样，是一种叫BSON的格式。

**单条数据插入**

插入方法使用 `insert()` 或者 `save()` 或者 `insertOne()` 方法

语法

```sh
db.集合名.insert(BSON格式文档内容)
或者
db.集合名.save(BSON格式文档内容)
或者
db.集合名.insertOne(BSON格式文档内容)
```

> - save()：如果 _id 主键存在则更新数据，如果不存在就插入数据。该方法新版本中已废弃，可以使用 **db.collection.insertOne()** 或 **db.collection.replaceOne()** 来代替。
> - insert(): 若插入的数据主键已经存在，则会抛 **org.springframework.dao.DuplicateKeyException** 异常，提示主键重复，不保存当前数据。

如向user集合中插入一条数据

```sh
db.user.insert({name: "张三", age: 23})
```

> 默认情况下，MongoDB使用 `_id` 字段作为主键，自动生成。如果手动指定，则使用手动指定的数据。
>
> `_id` 字段类型是ObjectId，手动指定id时可以使用ObjectId，也可以使用MongoDB支持的任意类型

**批量插入**

```sh
db.集合名.insertMany(
	[文档1, 文档2...],
	{
		ordered: 指定MongoDB是否有序插入。可选值
	}
)
```

> MongoDB的批量插入并不是同时成功或者同时失败。如果在插入过程中有一条数据失败，就会终止插入，但是在此之前的数据都会插入成功。

插入数据较多的情况下可能会报错，可以在命令前后使用try catch进行异常捕捉

```sh
try {
	db.集合名.insertMany()
} catch(e) {
	print(e)
}
```

### 3.3.2 更新文档

更新文档使用update方法

```sh
db.集合名.update(
	{BSON格式查询条件},
	{BSON格式要更新的内容},
	{
		upsert: 布尔类型，指定如果不存在update的记录，是否插入。默认false
		multi: 默认false，只更新找到的第一条记录。如果为true，则更新查询条件下的所有记录
	}
)
```

**覆盖修改**

修改用户编号是1的数据，姓名修改为 李四

```sh
db.user.update(
	{_id: "1"},
	{name, "李四"}
)
```

修改完成后发现这条文档只剩下name字段了，其他字段被覆盖掉了，因此实际使用不可以这么写。

**局部修改**

修改数据时，建议使用修改器 `$set` 来实现。还是上面那条需求。

```sh
db.user.update(
	{_id: "1"},
	{$set: {name: "李四"}}
)
```

修改成功，其他字段依然存在

**批量修改**

更新所有年龄是18岁的用户，年龄更新为19

```sh
db.user.update(
	{age: NumberInt(18)},
	{$set: {age: NumberInt(19)}},
	{multi: true}
)
```

如果不加multi参数，只会更新符合条件的一条数据

**列值自增**

如果想要 对某列的值进行自增操作，可以使用 `$inc` 

```sh
db.集合名.update(
	{_id: "1"},
	{$inc: {age: NumberInt(1)}}
)
```

### 3.3.3 删除文档

```sh
db.集合名称.remove(BSON格式条件)
#下面的语句会删除所有数据，慎用
db.集合名称.remove({})
```

## 3.4 文档查询

### 3.4.1 查询方法

查询使用 `find()` 或者 `findOne()` 方法进行

语法

```sh
db.集合名.find(BSON格式条件).pretty()
pretty是可选项。功能是格式化输出查询结果。
```

> `find()` 方法会查询出所有符合要求的数据
>
> `findOne()` 方法只会查询出第一条符合要求的数据

例如我要查询编号是1的用户

```sh
db.user.find({_id: "1"})
```

### 3.4.2 条件查询

条件查询的作用是让MongoDB可以像MySQL一样根据特定的条件从数据库中查询数据

常用的条件如下表

| 操作        | 格式                               | 范例                                 | MySQL中类似语句                         |
| ----------- | ---------------------------------- | ------------------------------------ | --------------------------------------- |
| 等于        | {key, value}                       | db.user.find({_id: 1})               | where id = 1                            |
| 小于        | {key, {$lt: value}                 | db.user.find({age: {$lt: 18}})       | where age < 18                          |
| 小于等于    | {key, {$lte: value}}               | db.user.find({age: {$lte: 18}})      | where age <= 18                         |
| 大于        | {key, {$gt: value}}                | db.user.find({age: {$gt: 18}})       | where age > 18                          |
| 大于等于    | {key, {$gte: value}}               | db.user.find({age: {$gte: 18}})      | where age >= 18                         |
| 不等于      | {key, {$ne: value}}                | db.user.find({age: {$ne: 18}})       | where age != 18                         |
| 包含        | {key, /value/}                     | db.user.find(name: /俊/)             | where name like '%俊%'                  |
| 以...开头   | {key, /^value/}                    | db.user.find(name: /^雷/)            | where name like '雷%'                   |
| 以...结尾   | {key, /value$/}                    | db.user.find(name: /华$/)            | where name like '%华'                   |
| 在...之中   | {key, {$in: [value1, value2...]}}  | db.user.find(age: {$in: [1,2,3]})    | where age in (1,2,3)                    |
| 不在...之中 | {key, {$nin: [value1, value2...]}} | db.user.find(age: {$nin: [1,2,3]})   | where age not in (1,2,3)                |
| 类型为...   | {key: {$type: "string"}}           | db.user.find(age: {$type: 'string'}) | 无。查询用户集合中age是String类型的数据 |

> 其中，/value/ 事实上是正则表达式查询。语法和JS中的正则一模一样。

### 3.4.3 条件连接查询

**AND 条件**

`find()` 方法可以传入多个键值对，类似于SQL的AND条件

如，查询性别是男，年龄在18岁的用户

```sh
db.user.find({sex: "1", age: "18"})
```

**OR 条件**

MongoDB提供了关键字 `$or` 用来实现or查询

语法

```sh
db.集合名.find({$or: [{key1, value1}, {key2, value2}]})
```

如，查询年龄是17或者19的用户 

```sh
db.user.find({$or: [{age: 17}, {age: 19}]})
```

**AND 和 OR 联合使用**

如，查询性别是男，并且年龄是17或者19的用户

```sh
db.user.find({sex: "1", $or: [{age: 17}, {age: 19}]})
```

### 3.4.4 投影查询

默认情况下，查询的结果是返回所有字段。这样的话性能可能会比较低。一般我们只查询需要的部分字段，就可以使用投影查询

如，只查询name, age字段

```sh
db.user.find({_id: "1"}, {name: 1, age: 1, _id: 0})
```

> 这里的_id默认是会被查询出来的，如果不想显示 _ id，就可以使用 _id: 0来隐藏该列

### 3.4.5 统计查询

语法

```sh
db.集合名.count(BSON格式条件)
```

统计所有性别为男的数据

```sh
db.user.count({age: "1"})
```

### 3.4.6 分页查询

使用 `limit` 读取指定数量的数据， `skip` 跳过指定数量的数据，二者搭配来进行分页查询。使用逻辑和MySql中的limit一样。

查询返回前三条记录，limit参数默认是20

```sh
db.集合.find().limit(3)
```

跳过前3条记录，skip参数默认是0

```sh
db.集合.find().skip(3)
```

每页查询2条数据

```sh
db.集合.find().skip(0).limit(2)
db.集合.find().skip(2).limit(2)
db.集合.find().skip(3).limit(2)
```

### 3.4.7 排序查询

排序查询使用 `sort` 方法。sort方法可以指定排序的字段，值为1升序，值为-1降序

根据性别升序排序，并根据年龄降序排序

```sh
db.user.find().sort({sex: 1, age: -1})
```

再进行分页，每页20条

```sh
db.user.find().sort({sex: 1, age: -1}).skip(0).limit(20)
```

## 3.5 角色权限操作（了解）

默认情况下，MongoDB运行时是没有启用和用户访问权限控制的，MongoDB不会对连接客户端进行用户验证，这样非常危险。

可以通过MongoDB启动时使用选项 `--auth` 或者在启动时指定的配置文件中添加 `auth=true` 来强制开启访问控制。

MongoDB是基于角色的访问控制，通过对一个用户授予一个或者多个角色来控制用户的权限。

### 3.5.1 角色查询

**查询所有角色权限（仅用户自定义角色）**

```sh
db.runCommand({rolesInfo: 1})
```

**查询所有角色权限（包含内置角色）**

```sh
db.runCommand({rolesInfo: 1, showBuiltinRoles: true})
```

**查询当前数据库中指定角色的权限**

```sh
db.runCommand({rolesInfo: 角色名})
```

**查询其他数据库中指定的角色权限**

```sh
db.runCommand({rolesInfo: {role: 角色名, db: 数据库名}})
```

**查询多个角色权限**

```sh
db.runCommand({rolesInfo: [角色名...., {role: 角色名, db: 数据库名}]})
```

**常见角色**

| 角色                 | 描述                                           |
| -------------------- | ---------------------------------------------- |
| read                 | 指定数据库的读权限                             |
| readWrite            | 指定数据库的读写权限                           |
| readAnyDatabase      | 任何数据的读权限（除了config和local）          |
| readWriteAnyDatabase | 任何数据的读写权限（除了config和local）        |
| userAdminAnyDatabase | 指定数据库创建修改用户的权限                   |
| dbAdminAnyDatabase   | 任何数据库的读取、清理、修改、压缩、统计等权限 |
| dbAdmin              | 指定数据库的读取、清理、修改、压缩、统计等权限 |
| userAdmin            | 指定数据库创建和修改用户                       |
| clusterAdmin         | 对整个集群或者数据库系统进行管理操作           |
| backup               | 备份MongoDB数据                                |
| restore              | 还原MongoDB数据                                |
| root                 | 超级权限                                       |

### 3.5.2 安全认证

#### 3.5.2.1 前置操作

关闭数据库

```sh
mongo --port 27017
use admin
db.shutdownServer()
```

#### 3.5.2.2 添加用户和权限

先不进行配置，正常启动数据库

```sh
/usr/local/mongodb/bin/mongod -f /usr/local/mongod.conf
```

登录数据库

```sh
/usr/local/mongodb/bin/mongo
```

**创建系统超级管理员mongoroot和admin数据库的管理用户mongoadmin**

```sh
# 切到admin
use admin
# 创建系统超级用户mongoroot，密码123456，角色root
db.createUser({user: "mongoroot", pwd: "123456", roles: ["root"]})
# 创建admin库的管理员
db.createUser({user: "mongoadmin",pwd: "123456", "roles": [{role: "userAdminAnyDatabase", db:"admin"}]})
# 查看已经创建的用户
db.system.users.find()
```

> MongoDB默认把用户信息存放到 `admin` 数据库的 `system.users` 表中
>
> 创建用户如果不指定数据库，则创建的用户在所有数据库上有效

#### 3.5.2.3 认证测试

```sh
# 切到admin
db.auth("账号", "密码")
```

#### 3.5.2.4 创建普通用户

```sh
# 切到bbs
use bbs
# 创建用户，拥有读写权限
db.createUser({user: "jigege", pwd: "123456", roles: [{role: "readWrite", db: "bbs"}]})
```

> 如果在启动mongodb时不开启认证，那么启动后不管是否用账号密码都可以成功连接数据库。
>
> 如果开启了认证，登陆的客户端必须使用admin中保存的角色

#### 3.5.2.5 开启认证

**先关闭数据库**

```sh
use admin
db.shutdownServer()
```

**重新启动mongodb，开启认证**

**参数方式**

```sh
/usr/local/mongodb/bin/mongod -f /mongodb/mongod.conf --auth
```

**配置文件方式**

编辑配置文件

```sh
vim /mongodb/mongof.conf
```

加入以下内容

```yaml
security:
  #开启授权认证
  authorization: enabled
```

正常启动

```sh
/usr/local/mongodb/bin/mongod -f /mongodb/mongod.conf
```

**连接时认证**

```sh
/usr/local/mongodb/bin/mongo --host 172.17.238.235 --port 27017 -u mongoroot -p 123456
```

对bbs数据库进行登录认证操作

```sh
/usr/local/mongodb/bin/mongo --host 172.17.238.235 --port 27017 -u jigege -p 123456
```

> -u 用户名
>
> -p 密码

## 3.6 暴力关闭数据库补救

暴力关闭数据库

```sh
ps -aux|grep monogodb
kill -9 进程号
```

暴力关闭后，可能会存在数据的损坏，导致mongodb无法启动，可以通过下面的操作恢复（如果失败那神仙也救不了你了）

**删除lock文件**

```sh
rm -f /mongodb/data/db/*.lock
```

**修复数据**

```sh
/usr/local/mongodb/bin/mongod --repair --dbpath=/mongodb/data/db
```



# 4 索引

索引通常能够极大的提高查询的效率，如果没有索引，MongoDB在读取数据时必须扫描集合中的每个文档并选取那些符合查询条件的记录。

这种扫描全集合的查询效率是非常低的，特别在处理大量的数据时，查询可以要花费几十秒甚至几分钟，这对网站的性能是非常致命的。

索引是特殊的数据结构，索引存储在一个易于遍历读取的数据集合中，索引是对数据库表中一列或多列的值进行排序的一种结构

MongoDB中使用 `B树` 数据结构存储索引，B树在本次课程中不作讲解。

## 4.1 索引分类

### 4.1.1 单字段索引

MongoDB支持在文档单个字段上创建用户定义的升序/降序索引，称之为 `单字段索引`

### 4.1.2 复合索引

MongoDB还支持多个字段的索引，称之为 `复合索引`

复合索引中对字段顺序也有要求，如果复合索引是由 {age: 1, sex: -1} 组成，就会先按照age正序排序，再按照sex倒序排序

### 4.1.3 地理空间索引

为了支持对地理坐标的有效查询，MongoDB提供了二维索引和二维球面索引

### 4.1.4  文本索引

MongoDB提供了文本索引，支持在集合中搜索字符串内容。

### 4.1.5 哈希索引

为了支持散列的分片，MongoDB提供了散列索引，它对字段值的散列进行索引。哈希索引只支持 = 条件查询，不支持范围查询。

## 4.2 索引操作

### 4.2.1 创建索引

语法

```sh
db.集合.createIndex(keys, options)
```

| 参数    | 描述                                                         |
| ------- | ------------------------------------------------------------ |
| keys    | 键值对，格式：{字段: 1或-1}，如{age: 1}表示在age字段上的升序索引，-1则为降序索引 |
| options | 可选参数，见下表                                             |

options配置

| 参数       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| background | 布尔类型，默认为false。指定是否后台创建索引。建索引的过程中会阻塞其它数据库操作，后台创建则可以避免这种问题 |
| unique     | 布尔类型，默认false。指定建立的索引是否唯一                  |
| name       | 索引名称。MongoDB默认通过建立索引的字段名和排序顺序生成索引名称 |

示例

**单字段索引**：对 `age` 字段建立索引

```sh
db.user.createIndex({age: 1})
```

**复合索引**：对 `age` 和 `sex` 同时创建复合索引 

```sh
db.user.createIndex({age: 1, sex: -1})
```

### 4.2.2 查看索引

**返回一个集合中所有的索引**

```sh
db.集合名.getIndexes()
```

其中，_id 是默认的索引。

MongoDB在创建集合的过程中，在_id字段上创建一个唯一的索引，名称为 \_id\_ ，该索引可以防止客户端插入两个具有相同值的文档，该索引不可以被删除。  

**查看看集合索引大小**

```sh
db.集合名.totalIndexSize()
```

### 4.2.3 删除索引

**删除指定索引**

```sh
db.集合名.dropIndex(索引名称或索引键值对)
```

如删除age上的升序索引

```sh
db.user.dropIndex({age: 1})
或者
db.user.dropIndex("age_1")
```

**删除所有索引**

```sh
db.集合名.dropIndexes()
```

该方法并不会将\_id索引删除，只能删除\_id之外的索引

### 4.2.4 查询分析

MongoDB 查询分析可以确保我们所建立的索引是否有效。

查询分析常用的方法是 `explain()` 

explain 操作提供了查询信息、使用索引、查询统计等，有利于我们对索引的优化。

先在user中创建age和sex索引

```sh
db.user.createIndex({age: 1, sex: 1})
```

再使用explain对查询进行分析

```sh
db.user.find({age: 18, sex: 1, _id: 1}).explain()
```

```sh
> db.user.find({age: 18, sex: 1}).explain()
{
	"queryPlanner" : {
		"plannerVersion" : 1,
		"namespace" : "bbs.user",
		"indexFilterSet" : false,
		"parsedQuery" : {
			"$and" : [
				{
					"age" : {
						"$eq" : 18
					}
				},
				{
					"sex" : {
						"$eq" : 1
					}
				}
			]
		},
		"winningPlan" : {
			"stage" : "FETCH",
			"inputStage" : {
				"stage" : "IXSCAN",
				"keyPattern" : {
					"age" : 1,
					"sex" : -1
				},
				"indexName" : "age_1_sex_-1",
				"isMultiKey" : false,
				"multiKeyPaths" : {
					"age" : [ ],
					"sex" : [ ]
				},
				"isUnique" : false,
				"isSparse" : false,
				"isPartial" : false,
				"indexVersion" : 2,
				"direction" : "forward",
				"indexBounds" : {
					"age" : [
						"[18.0, 18.0]"
					],
					"sex" : [
						"[1.0, 1.0]"
					]
				}
			}
		},
		"rejectedPlans" : [ ]
	},
	"serverInfo" : {
		"host" : "iZ2zebq96d1zohii1vadzqZ",
		"port" : 27017,
		"version" : "4.0.19",
		"gitVersion" : "7e28f4296a04d858a2e3dd84a1e79c9ba59a9568"
	},
	"ok" : 1
}

```



重点看 `stage` 和 `indexOnly` ，这两个字段有时候可能并不会存在。

stage：当值为 `COLLSCAN` 时，表示全集合扫描，这样的性能是比较低的。当值为 `IXSCAN` 时，是基于索引扫描，创建索引后我们需要保证查询是基于索引扫描的

indexOnly：为true时表示使用到了索引

### 4.2.5 覆盖索引查询

覆盖索引查询和MySQL中的类似。**当查询条件和所要查询的列全部都在索引中时，MongoDB会直接从索引返回结果。这样的查询性能非常的高**

# 5 Java操作MongoDB

## 5.1 mongodb-driver

mongodb-driver 是mongodb 官方推出的Java连接MongoDB的驱动包，类似于JDBC驱动。该包操作mongodb非常的不友好，这里只提一下有这个技术，感兴趣的可以自己看菜鸟教程学习一下。

[mongodb-driver](https://www.runoob.com/mongodb/mongodb-java.html)

## 5.2 SpringDataMongoDB

SpringData家族成员之一，酷帅狂拽吊炸天的MongoDB持久层框架，底层封装了mongodb-driver。

## 5.3 论坛用户功能案例

需求：对用户进行添加、修改、删除、分页查询操作。用户有关注数，表示该用户被关注的数量。

### 5.3.1 项目搭建

（1）创建项目 bbs，pom.xml引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

（2）application.yml中加入以下配置

```yaml
spring:
  #数据源配置
  data:
    mongodb:
      #主机地址
      host: 127.0.0.1
      #数据库
      database: bbsdb
      #默认端口是27017
      port: 27017

```

（3）启动项目，看控制台是否报错

### 5.3.2 表结构

| 字段        | 描述             |
| ----------- | ---------------- |
| _id         | ID               |
| name        | 姓名             |
| age         | 年龄             |
| sex         | 性别             |
| address     | 地址             |
| createdTime | 创建时间         |
| state       | 状态，1启用0禁用 |
| followNum   | 关注者数量       |

### 5.3.3 实体类编写

```java
package com.jg.mongo.pojo;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.index.CompoundIndex;
import org.springframework.data.mongodb.core.index.Indexed;
import org.springframework.data.mongodb.core.mapping.Document;
import org.springframework.data.mongodb.core.mapping.Field;

import java.io.Serializable;
import java.util.Date;

/**
 * 使用 @Document("user") 指定这个类对应 user 集合
 * 使用 @CompoundIndex(def = "{'id':1, 'age': -1}") 表示复合索引
 *
 * @Author: 杨德石
 * @Date: 2020/8/13 22:01
 * @Version 1.0
 */
@Document("user")
@CompoundIndex(def = "{'id':1, 'age': -1}")
public class User implements Serializable {

    /**
     * 主键，该属性会自动对应_id字段。
     * 如果该属性名称就叫id，那么注解可以省略
     */
    @Id
    private String id;

    /**
     * 指定该属性对应集合中的name列
     * 如果属性名已经和name对应了
     * 就可以不写
     */
    @Field("name")
    private String name;

    private Integer age;

    private String address;

    /**
     * 使用 @Indexed 指定单列索引
     */
    @Indexed
    private Integer sex;

    private Date createdTime;

    private Integer state;

    private Integer followNum;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public Integer getSex() {
        return sex;
    }

    public void setSex(Integer sex) {
        this.sex = sex;
    }

    public Date getCreatedTime() {
        return createdTime;
    }

    public void setCreatedTime(Date createdTime) {
        this.createdTime = createdTime;
    }

    public Integer getState() {
        return state;
    }

    public void setState(Integer state) {
        this.state = state;
    }

    public Integer getFollowNum() {
        return followNum;
    }

    public void setFollowNum(Integer followNum) {
        this.followNum = followNum;
    }
}

```

> 1. 实体类需要使用 `@Document` 注解标识为MongoDB文档，并指定集合名。
> 2. `@Id` 注解指定文档的主键，不建议省略
> 3. `@CompoundIndex` 注解指定复合索引，可以在Java类中添加索引，也可以在MongoDB中添加
> 4. `@Indexed` 注解指定单字段索引。

### 5.3.4 Dao编写

```java
package com.jg.mongo.dao;

import com.jg.mongo.pojo.User;
import org.springframework.data.mongodb.repository.MongoRepository;

/**
 * 继承MongoRepository，指定实体和主键的类型作为泛型
 *
 * @Author: 杨德石
 * @Date: 2020/8/13 22:07
 * @Version 1.0
 */
public interface UserRepository extends MongoRepository<User, String> {
}

```

> Dao层编写非常简单，只需要编写一个接口，继承 `MongoRepository` ，指定实体和主键类型即可

### 5.3.5 Service编写

```java
package com.jg.mongo.service;

import com.jg.mongo.dao.UserRepository;
import com.jg.mongo.pojo.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * @Author: 杨德石
 * @Date: 2020/8/13 22:08
 * @Version 1.0
 */
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    /**
     * 保存
     *
     * @param user
     */
    public void save(User user) {
        userRepository.save(user);
    }

    /**
     * 更新
     *
     * @param user
     */
    public void update(User user) {
        userRepository.save(user);
    }

    /**
     * 根据id删除
     *
     * @param id
     */
    public void deleteById(String id) {
        userRepository.deleteById(id);
    }

    /**
     * 查询所有
     *
     * @return
     */
    public List<User> findAll() {
        return userRepository.findAll();
    }

    /**
     * 根据id查询
     *
     * @param id
     * @return
     */
    public User findById(String id) {
        return userRepository.findById(id).get();
    }

}

```

> 我们发现，UserRepository里没有更新方法， 因为SpringDataMongoDB的save方法既有更新功能又有保存功能。

### 5.3.6 Junit测试

```java
package com.jg.mongo;

import com.jg.mongo.pojo.User;
import com.jg.mongo.service.UserService;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.Date;
import java.util.List;

@SpringBootTest
public class MongoApplicationTests {

    @Autowired
    private UserService userService;

    @Test
    public void testSave() {
        User user = new User();
        user.setAge(10);
        user.setSex(1);
        user.setName("高尔稽");
        user.setState(1);
        user.setFollowNum(0);
        user.setAddress("我也不知道你住哪");
        user.setCreatedTime(new Date());
        userService.save(user);
    }

    @Test
    public void testFindAll() {
        List<User> userList = userService.findAll();
        System.out.println(userList);
    }

    @Test
    public void testFindById() {
        User user = userService.findById("5f34036ce9a0ec5fdc5354a4");
        System.out.println(user);
    }

    @Test
    public void testUpdate() {
        User user = new User();
        user.setId("5f340372e9a0ec5fdc5354a6");
        user.setAge(10);
        user.setSex(1);
        user.setName("菜文稽");
        user.setState(1);
        user.setFollowNum(0);
        user.setCreatedTime(new Date());
        userService.update(user);
    }

    @Test
    public void testDelete() {
        userService.deleteById("5f34036ce9a0ec5fdc5354a5");
    }

}

```

### 5.3.7 使用JPA规范查询

![image-20200813221817310](C:%5CUsers%5Cyds%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20200813221817310.png)

根据启用状态和性别查询

Repository中编写如下代码

```java
    /**
     * 根据state和sex查询
     *
     * @param state
     * @param sex
     * @return
     */
    List<User> findByStateAndSex(Integer state, Integer sex);
```

Service中添加代码

```java
    /**
     * 根据state和sex查询
     *
     * @param state
     * @param sex
     * @return
     */
    public List<User> findByStateAndSex(Integer state, Integer sex) {
        return userRepository.findByStateAndSex(state, sex);
    }
```

单元测试

```java

    @Test
    public void testFindByStateAndSex() {
        List<User> list = userService.findByStateAndSex(1, 1);
        for (User user : list) {
            System.out.println(user);
        }
    }
```



### 5.3.8 MongoTemplate

现在有个需求，我们需要给某个用户的关注量+1，下面的代码是实现方案

```java
    public void incrFollowCount(String id) {
        User user = userRepository.findById(id).get();
        user.setFollowNum(user.getFollowNum() + 1);
        userRepository.save(user);
    }
```

该方案实现起来虽然简单，但是性能不高。

我们只需要给关注量+1，并不需要姓名、地址等这些数据，因此也就没必要查询出这些字段，甚至于根本就不需要查询操作，直接更新就可以了。

我们可以使用MongoTemplate解决这个需求。

```java
    public void incrFollowCount(String id) {
        // 构造查询对象
        Query query = Query.query(Criteria.where("_id").is(id));
        // 构造更新对象
        Update update = new Update();
        // follow字段+1
        update.inc("followNum");
        // 执行update
        mongoTemplate.updateFirst(query, update, User.class);
    }
```

### 5.3.9 分页查询

分页查询主要涉及两个类。一个是Page，一个是Pageable

Repository中编写代码

```java
    /**
     * 根据年龄分页查询
     *
     * @param age
     * @param pageable
     * @return
     */
    Page<User> findByAge(Integer age, Pageable pageable);
```

Service中添加代码

```java
    /**
     * 根据名称分页查询
     *
     * @param age
     * @param page
     * @param size
     * @return
     */
    public Page<User> findByAgePage(Integer age, int page, int size) {
        // 构造分页对象
        PageRequest pageRequest = PageRequest.of(page - 1, size);
        return userRepository.findByAge(age, pageRequest);
    }
```

测试类

```java
    @Test
    public void testFindByAgePage() {
        Page<User> users = userService.findByAgePage(18, 1, 2);
        System.out.println("总条数：" + users.getTotalElements());
        System.out.println("总页数：" + users.getTotalPages());
        System.out.println("本页数据：");
        List<User> userList = users.getContent();
        for (User user : userList) {
            System.out.println(user);
        }
    }
```

***

实际开发中我们可能会需要能够多条件分页查询，上面的场景可能不满足需求。使用MongoTemplate也可以解决分页问题

```java
    /**
     * 使用MongoTemplate分页查询
     *
     * @param page
     * @param size
     * @param user
     * @return
     */
    public List<User> findPageByTemplate(int page, int size, User user) {
        // 构造一个查询对象
        Query query = new Query();
        // 设置参数
        if (!StringUtils.isEmpty(user.getName())) {
            query.addCriteria(Criteria.where("name").regex(user.getName() + ".*"));
        }
        if (user.getAge() != null) {
            query.addCriteria(Criteria.where("age").lt(user.getAge()));
        }
        if (user.getSex() != null) {
            query.addCriteria(Criteria.where("sex").is(user.getSex()));
        }
        // 跳过多少条
        query.skip((page - 1) * size);
        // 取出多少条
        query.limit(size);
        // 构造排序对象
        Sort.Order order = new Sort.Order(Sort.Direction.DESC, "age");
        // 设置排序对象
        query.with(Sort.by(order));
        return mongoTemplate.find(query, User.class);
    }
```

### 5.3.10 @Query注解

我们使用SpringDataJpa操作MySql的时候，尽管JPA已经非常强大，仍然避免不了手写sql的场景。

SpringDataMongoDB也存在类似的情况，因此我们可能会需要手写MongoDB的查询语句，使之操作更加灵活。

手写查询语句可以使用 `@Query` 注解。

***

需求1：根据 state 和 age 查询

这里我们需要使用 `?数字占位符` 表达式来取出参数中指定位置的值，占位符从0开始。

```java
    /**
     * 根据state和age查询
     *
     * @param state
     * @param age
     * @return
     */
    @Query("{ state: ?0, age: ?1  }")
    List<User> selectEnableUserByAge(Integer state, Integer age);

```

***

需求2：有时候我们的参数可能过多，条件也可能不同，我们想传入一个实体类进行查询，直接取出实体类中的属性进行条件构造。

这里我们需要使用 `SpEL` 表达式，格式：`?#{}` 括号中使用 `[下标]` 来取出指定位置的参数，如 `?#{[0]}` 则取出第一个参数。之后直接取出参数中的指定属性即可，如 `?#{[1].age}` 就是取出第二个参数的age属性。

现在我们需要查询 **性别为女，或者年龄在18岁以下的所有启用中的用户**

```java
    /**
     * 根据实体类查询
     * 根据性别或者年龄，和状态查询
     *
     * @param user
     * @return
     */
    @Query("{  state: ?#{[0].state}, $or: [ {sex: ?#{[0].sex}}, {age: { $lt: ?#{[0].age} }}  ] }")
    List<User> selectByEntity(User user);
```

同时，如果我们只想获取指定的字段，我们还可以使用第二个参数 `fields` 来进行投影查询

```java
    /**
     * 根据实体类查询
     * 根据性别或者年龄，和状态查询
     *
     * @param user
     * @return
     */
    @Query(value = "{  state: ?#{[0].state}, $or: [ {sex: ?#{[0].sex}}, {age: { $lt: ?#{[0].age} }}  ] }",
            fields = "{ name:1, age:1 }")
    List<User> selectByEntity(User user);
```

***

案例三：分页查询（自行尝试实现）

### 5.3.11 SpringDataMongoDB连接认证

前面我们学到了安全认证。当数据库配置了安全认证后，想要使用SpringDataMongoDB连接MongoDB，就需要使用 `username:password@hostname/dbname` 格式 ，username为用户名，password为密码

```yaml
spring:
  data:
    mongodb:
      uri: mongodb://jigege:123456@39.102.38.0:27017/bbs?authSource=admin&authMechanism=SCRAM-SHA-1
```



# 6. 副本集

## 6.1 简介

MongoDB复制是将数据同步在多个服务器的过程。

复制提供了数据的冗余备份，并在多个服务器上存储数据副本，提高了数据的可用性， 并可以保证数据的安全性。复制还允许您从硬件故障和服务中断中恢复数据。

通俗来讲，副本集就是多台机器进行同一数据的异步同步，从而使多台机器拥有同一数据的多个脚本，并且能当主库宕机时在不需要用户干预的情况下自动切换到其他备份的数据库作为主库。并且，还可以利用副本访问做读写分离。

副本集的使用可以提供冗余和高可用性，提高系统负载。

## 6.2 MongoDB中的复制

mongodb的复制至少需要两个节点。其中一个是主节点，负责处理客户端请求，其余的都是从节点，负责复制主节点上的数据。

mongodb各个节点常见的搭配方式为：一主一从、一主多从。

主节点记录在其上的所有操作更改，从节点定期轮询主节点获取这些操作，然后对自己的数据副本执行这些操作，从而保证从节点的数据与主节点一致。

MongoDB复制结构图如下所示：

![img](https://images2017.cnblogs.com/blog/136834/201710/136834-20171017111507506-258737937.png)

![img](https://images2017.cnblogs.com/blog/136834/201710/136834-20171017111636974-294296509.png)

客户端从主节点读取数据，在客户端写入数据到主节点时， 主节点与从节点进行数据交互保障数据的一致性。

> **主从复制和副本集的区别**
>
> 主从复制和副本集的最大区别就是，副本集没有固定的主节点，整个集群会选出一个主节点，当主节点挂掉后，剩下的节点又会选出一个主节点。

## 6.3 副本集的角色

副本集主要有两种类型和三种角色

**两种类型：**

1. 主节点（Primary）：数据操作的主要连接点，允许读和写操作
2. 从节点（Secondaries）：数据冗余备份节点，可以读或选举为主节点

**角色：**

MongoDB主要有三种角色

主要成员（Primary）：主节点，主要接收所有的写操作

副本成员（Replicate）：主从节点通过备份操作以维护相同的数据集。只支持读操作，不支持写操作。拥有选举能力

仲裁者（Arbiter）：不保留任何数据的副本，只具有选举作用。副本成员也可以作为仲裁者。

![image-20200806231338141](https://ydsmarkdown.oss-cn-beijing.aliyuncs.com/md/20200806231338.png)

> 仲裁者永远是仲裁者，而主要成员与副本成员之间角色可能会发生变化。副本成员可能会因为主要成员的宕机而转为主要成员。主要成员也可能因为宕机并重新修复后成为副本成员。
>
> 尽可能保证主节点+副本的个数是奇数，这样就不需要加仲裁者就可以满足大多数的投票
>
> 如果主节点+副本的个数是偶数，建议加一个仲裁者。

除了这三个角色外，副本集的节点还可以是以下角色。这些做了解即可

|                     | 成为primary | 对客户端可见 | 参与选举 | 复制数据 |
| ------------------- | ----------- | ------------ | -------- | -------- |
| Default             | √           | √            | √        | √        |
| Secondary-Only      | /           | √            | √        | √        |
| Hidden              | /           | /            | √        | √        |
| Delayed（延迟同步） | /           | √            | √        | √        |
| Non-Voting          | √           | √            | /        | √        |

## 6.4 副本集的创建

我们本次课程以一主一副一仲裁为案例搭建副本集。

本次使用三台阿里云服务器。服务器不够的同学，可以在一台服务器进行操作，操作步骤类似，只需要配置三个不同的端口即可。

三台机器的ip分别为：`172.17.238.235` `172.17.238.236` `172.17.238.237`



### 6.4.1 创建主节点

**创建存放数据和日志的目录**

```sh
mkdir -p /mongodb/replica_sets/rs_1/log
mkdir -p /mongodb/replica_sets/rs_1/data/db
```

创建配置文件

```sh
vim /mongodb/replica_sets/rs_1/mongod.conf
```

mongod.conf

```yaml
systemLog:
  #日志输出为文件 
  destination: file
  #日志文件的路径 
  path: "/mongodb/replica_sets/rs_1/log/mongod.log"
  #mongod实例重新启动时，会将新条目附加到现有日志文件的末尾。 
  logAppend: true 
storage: 
  #数据目录。
  dbPath: "/mongodb/replica_sets/rs_1/data/db" 
  journal:
    #启用持久性日志
    enabled: true
processManagement:
  #启用守护进程模式。 
  fork: true
  #指定进程ID的文件位置
  pidFilePath: "/mongodb/replica_sets/rs_1/log/mongod.pid"
net:
  #服务实例绑定的IP
  bindIp: 0.0.0.0
  #绑定的端口 
  port: 27017
replication:
  #副本集的名称 
  replSetName: myrs
```

启动节点

```sh
/usr/local/mongodb/bin/mongod -f /mongodb/replica_sets/rs_1/mongod.conf
```

### 6.4.2 创建副本节点

**创建存放数据和日志的目录**

```sh
mkdir -p /mongodb/replica_sets/rs_2/log
mkdir -p /mongodb/replica_sets/rs_2/data/db
```

创建配置文件

```sh
vim /mongodb/replica_sets/rs_2/mongod.conf
```

mongod.conf

```yaml
systemLog:
  #日志输出为文件 
  destination: file
  #日志文件的路径 
  path: "/mongodb/replica_sets/rs_2/log/mongod.log"
  #mongod实例重新启动时，会将新条目附加到现有日志文件的末尾。 
  logAppend: true 
storage: 
  #数据目录。
  dbPath: "/mongodb/replica_sets/rs_2/data/db" 
  journal:
    #启用持久性日志
    enabled: true
processManagement:
  #启用守护进程模式。 
  fork: true
  #指定进程ID的文件位置
  pidFilePath: "/mongodb/replica_sets/rs_2/log/mongod.pid"
net:
  #服务实例绑定的IP 
  bindIp: 0.0.0.0
  #绑定的端口 
  port: 27017
replication:
  #副本集的名称 
  replSetName: myrs
```

启动节点

```sh
/usr/local/mongodb/bin/mongod -f /mongodb/replica_sets/rs_2/mongod.conf
```

###  6.4.3 创建仲裁节点

**创建存放数据和日志的目录**

```sh
mkdir -p /mongodb/replica_sets/rs_3/log
mkdir -p /mongodb/replica_sets/rs_3/data/db
```

创建配置文件

```sh
vim /mongodb/replica_sets/rs_3/mongod.conf
```

mongod.conf

```yaml
systemLog:
  #日志输出为文件 
  destination: file
  #日志文件的路径 
  path: "/mongodb/replica_sets/rs_3/log/mongod.log"
  #mongod实例重新启动时，会将新条目附加到现有日志文件的末尾。 
  logAppend: true 
storage: 
  #数据目录。
  dbPath: "/mongodb/replica_sets/rs_3/data/db" 
  journal:
    #启用持久性日志
    enabled: true
processManagement:
  #启用守护进程模式。 
  fork: true
  #指定进程ID的文件位置
  pidFilePath: "/mongodb/replica_sets/rs_3/log/mongod.pid"
net:
  #服务实例绑定的IP 
  bindIp: 0.0.0.0
  #绑定的端口 
  port: 27017
replication:
  #副本集的名称 
  replSetName: myrs
```

启动节点

```sh
/usr/local/mongodb/bin/mongod -f /mongodb/replica_sets/rs_3/mongod.conf
```

### 6.4.4 初始化副本集和主节点

连接任意一个节点（最好连接我们认为的主节点）

```sh
/usr/local/mongodb/bin/mongo --host=172.17.238.235 --port=27017
```

在配置副本集之前，现在的mongodb几乎所有的命令都是无法使用的。

初始化副本集

```sh
rs.initiate()
```

```json
{
	"info2" : "no configuration specified. Using a default configuration for the set",
	"me" : "iZ2zebq96d1zohii1vadzqZ:27017",
	"ok" : 1,
	"operationTime" : Timestamp(1596732614, 1),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1596732614, 1),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}
```

> ok的值为1则初始化成功
>
> 注意：这里显示的me不是ip:端口，需要修改

```sh
rs.conf()
```

展示出配置，可以看到host，我们需要修改的就是这个

```sh
# 导出配置到config
config = rs.conf()
# 设置第1个节点的优先级是10
config.members[0].host="172.17.238.235:27017"
# 重载配置
rs.reconfig(config)
```



### 6.4.5 查看副本集配置和状态

**查看副本集配置**

```sh
rs.conf()
或者
rs.config()
```

```json
{
	"_id" : "rs",
	"version" : 1,
	"protocolVersion" : NumberLong(1),
	"writeConcernMajorityJournalDefault" : true,
	"members" : [
		{
			"_id" : 0,
			"host" : "iZ2zebq96d1zohii1vadzqZ:27017",
			"arbiterOnly" : false,
			"buildIndexes" : true,
			"hidden" : false,
			"priority" : 1,
			"tags" : {
				
			},
			"slaveDelay" : NumberLong(0),
			"votes" : 1
		}
	],
	"settings" : {
		"chainingAllowed" : true,
		"heartbeatIntervalMillis" : 2000,
		"heartbeatTimeoutSecs" : 10,
		"electionTimeoutMillis" : 10000,
		"catchUpTimeoutMillis" : -1,
		"catchUpTakeoverDelayMillis" : 30000,
		"getLastErrorModes" : {
			
		},
		"getLastErrorDefaults" : {
			"w" : 1,
			"wtimeout" : 0
		},
		"replicaSetId" : ObjectId("5f2c34c6114dd6b9f2d4f225")
	}
}
```

> `_id:"rs" ` 副本集名称
>
> `members`：副本集成员数组
>
> `arbiterOnly`：该成员是否为仲裁节点
>
> `priority`：优先级权重值
>
> rs.conf() 命令，本质上其实是查询 `local` 库中 `system.replset` 集合中的数据
>
> ```sh
> db.system.replset.find()
> ```

**查看副本集状态**

```sh
rs.status()
```

```json
{
	"set" : "rs",
	"date" : ISODate("2020-08-06T16:55:59.842Z"),
	"myState" : 1,
	"term" : NumberLong(1),
	"syncingTo" : "",
	"syncSourceHost" : "",
	"syncSourceId" : -1,
	"heartbeatIntervalMillis" : NumberLong(2000),
	"optimes" : {
		"lastCommittedOpTime" : {
			"ts" : Timestamp(1596732954, 1),
			"t" : NumberLong(1)
		},
		"readConcernMajorityOpTime" : {
			"ts" : Timestamp(1596732954, 1),
			"t" : NumberLong(1)
		},
		"appliedOpTime" : {
			"ts" : Timestamp(1596732954, 1),
			"t" : NumberLong(1)
		},
		"durableOpTime" : {
			"ts" : Timestamp(1596732954, 1),
			"t" : NumberLong(1)
		}
	},
	"lastStableCheckpointTimestamp" : Timestamp(1596732914, 1),
	"electionCandidateMetrics" : {
		"lastElectionReason" : "electionTimeout",
		"lastElectionDate" : ISODate("2020-08-06T16:50:14.731Z"),
		"electionTerm" : NumberLong(1),
		"lastCommittedOpTimeAtElection" : {
			"ts" : Timestamp(0, 0),
			"t" : NumberLong(-1)
		},
		"lastSeenOpTimeAtElection" : {
			"ts" : Timestamp(1596732614, 1),
			"t" : NumberLong(-1)
		},
		"numVotesNeeded" : 1,
		"priorityAtElection" : 1,
		"electionTimeoutMillis" : NumberLong(10000),
		"newTermStartDate" : ISODate("2020-08-06T16:50:14.732Z"),
		"wMajorityWriteAvailabilityDate" : ISODate("2020-08-06T16:50:14.743Z")
	},
	"members" : [
		{
			"_id" : 0,
			"name" : "iZ2zebq96d1zohii1vadzqZ:27017",
			"health" : 1,
			"state" : 1,
			"stateStr" : "PRIMARY",
			"uptime" : 618,
			"optime" : {
				"ts" : Timestamp(1596732954, 1),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2020-08-06T16:55:54Z"),
			"syncingTo" : "",
			"syncSourceHost" : "",
			"syncSourceId" : -1,
			"infoMessage" : "",
			"electionTime" : Timestamp(1596732614, 2),
			"electionDate" : ISODate("2020-08-06T16:50:14Z"),
			"configVersion" : 1,
			"self" : true,
			"lastHeartbeatMessage" : ""
		}
	],
	"ok" : 1,
	"operationTime" : Timestamp(1596732954, 1),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1596732954, 1),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}
```

> `set:"rs"`：副本集名称
>
> `myState: 1`：说明状态正常
>
> `members`：副本集成员数组
>
> `stateStr`：该成员的角色
>
> `health:1`：该节点是健康的

### 6.4.5 添加副本

在主节点，添加其他成员到副本集

```sh
rs.add(host, arbiterOnly)
```

| 参数        | 描述                                   |
| ----------- | -------------------------------------- |
| host        | 副本集成员地址，格式一般是ip:端口      |
| arbiterOnly | 可选值，当为true时则表示该节点是仲裁者 |

仲裁节点也可以使用该命令，还可以使用下面的命令添加仲裁节点

```sh
rs.addArb(host)
```

添加了的副本可以使用 `rs.remove(host)` 移除

### 6.4.6 副本节点读操作

默认情况下，从节点只是一个备份，不是奴隶节点，从节点是没有读写权限的，可以通过设置增加读权限。

连接副本节点，执行下面的命令设置为奴隶节点，允许读操作

```sh
rs.slaveOk()
或者
rs.slaveOk(true)
或者db.getMongo().setSlaveOk()
```

如果要取消奴隶节点的读权限，可以执行下面的命令

```sh
rs.slaveOk(false)
```

执行完毕后，发现可以成功查询数据，实现了读写分离。

此外，仲裁者节点不存放任何数据，即使执行了这些命令依然是没有数据的。

### 6.4.7 选举机制

MongoDB副本集中会自动选举出主节点，触发选举的条件有以下几条

* 主节点宕机
* 主节点网络不可达（10秒没接收到心跳）
* 人工通过rs.stepDown(秒数)降级，指定节点多少秒内不参与选举

选举的规则是根据票数决定，票数越高，且获得了“大多数”成员的投票支持则节点获胜。大多数指 `节点数量/2 +1`

若票数相同，且都获得了大多数成员的支持，哪个节点的数据新哪个节点获胜。

***

在我们上面搭建副本集时，使用到了 `rs.conf()` 命令查看配置信息，有一个 `priority` 配置，表示优先级。优先级会影响到投票的票数，当优先级是1000时，投票就会获得1000票数。优先级越大， 就越可能获得多数成员的投票数。

> 仲裁结点的优先级必须是0，不可以是别的值，设置为0是为了让结点不具备选举权，只拥有投票权。

**修改优先级（了解）**

```sh
# 导出配置到config
config = rs.conf()
# 设置第1个节点的优先级是10
config.members[1].priority=10
# 重载配置
rs.reconfig(config)
```

### 6.4.8  SpringDataMongoDB连接副本集

连接副本集需要使用uri去连接，格式如下

```sh
mongodb://host1,host2,host3/数据库?connect=replicaSet&slaveOk=true&replicaSet=副本集名称
```



配置文件

```yaml
spring:
  data:
    mongodb:
      uri: mongodb://39.102.38.0:27017,39.102.35.47:27017,39.102.41.53:27017/bbs?connect=replicaSet&slaveOk=true&replicaSet=myrs

```

> slaveOk：开启副本节点读的功能
>
> connect=replicaSet：自动到副本集中选择读写的主机
>
> replicaSet：副本集名称

### 6.4.9 关闭副本集

建议依次关闭仲裁节点、副本节点、主节点

```sh
rs.stepDown()
use admin
db.shutdownServer()
```

## 6.5 安全认证（了解）

mongodb的各个节点成员之间使用内部身份验证，可以使用秘钥文件或者x.509证书。本次课程使用秘钥文件。

在秘钥文件（keyfile）身份验证中，副本集中每个mongodb都使用秘钥文件的内容作为共享密码。

### 6.5.1 通过主节点添加一个管理员账号

只需要在主节点操作，副本集会自动同步

```sh
# 切到admin
use admin
# 创建用户
db.createUser({user: "mongoroot", pwd: "123456", roles: ["root"]})
```

### 6.5.2 创建副本集认证的key文件

生成一个key文件到当前文件夹

```sh
#生成文件
openssl rand -base64 90 -out ./mongo.keyfile
#设置仅为文件所有者提供读取权限
chmod 400 ./mongo.keyfile
ll mongo.keyfile
```

> 所有副本集节点都必须使用同一个keyfile。可以在一个机器生成，再拷贝到其他机器

将文件拷贝到指定目录

```sh
cp mongo.keyfile /mongodb/replica_sets/rs_1
```

### 6.5.3 修改配置文件指定keyfile

分别编辑几个服务的mongod.conf文件，添加以下内容

```yaml
security:
  keyFile: /mongodb/replica_sets/rs_1/mongo.keyfile
  authorization: enabled
```

### 6.5.4 重启副本集

依次关闭仲裁节点、副本节点、主节点

```sh
rs.stepDown()
use admin
db.shutdownServer()
```

再分别启动三个节点

```sh
/usr/local/mongodb/bin/mongod -f /mongodb/replica_sets/rs_1/mongod.conf
/usr/local/mongodb/bin/mongod -f /mongodb/replica_sets/rs_2/mongod.conf
/usr/local/mongodb/bin/mongod -f /mongodb/replica_sets/rs_3/mongod.conf
```

### 6.5.5 添加普通账号

现用正常方式登录主节点，再进入bbs数据库，创建用户

```sh
use bbs
db.createUser({user: "jigege", pwd: "123456", roles: ["readWrite"])
```

重新连接，使用普通用户登录，查看数据

### 6.5.6 SpringDataMongoDB连接认证

```yaml
spring:
  data:
    mongodb:
      uri: mongodb://jigege:123456@39.102.38.0:27017,39.102.35.47:27017,39.102.41.53:27017/bbs?connect=replicaSet&slaveOk=true&replicaSet=myrs

```



# 7 分片

MongoDB除了副本集以外，还支持分片集群。分片可以满足MongoDB数据量大量增长的需求。

当MongoDB存储海量的数据时，一台机器可能不足以存储数据，也可能不足以提供可接受的读写吞吐量。这时，我们就可以通过在多台机器上分割数据，使得数据库系统能存储和处理更多的数据。 

## 7.1 概念

分片（有时候也叫分区）是一种跨多台机器分布数据的方法，是指将数据拆分，将其分散到不同的机器上，处理更多的负载。

分片有两种解决方案：垂直扩展和水平扩展。

垂直扩展：增加更多的CPU和存储资源来扩展容量。

水平扩展：将数据集分布在多个服务器上。水平扩展就是分片

***

**为什么使用分片**

- 复制所有的写入操作到主节点
- 延迟的敏感数据会在主节点查询
- 单个副本集限制在12个节点
- 当请求量巨大时会出现内存不足。
- 本地磁盘不足
- 垂直扩展需要更多的钞能力

## 7.2 分片包含的组件

![image-20200808213911186](https://ydsmarkdown.oss-cn-beijing.aliyuncs.com/md/20200808213911.png)

上图中主要有如下所述三个主要组件：

- Shard:

  分片，用于存储实际的数据块，实际生产环境中一个分片可以由多个副本集承担，防止主机单点故障

- Config Server:

  配置服务器。存储几群的元数据和配置。从MongoDB3.4开始，配置服务器必须是副本集

- Query Routers:

  路由，mongos充当查询路由器，在客户端和分片集群之间提供接口

本次课程的目标是搭建一个由两个分片节点副本集+一个配置节点副本集+两个路由节点的集群，总共11个服务节点。

![image-20200808220153211](https://ydsmarkdown.oss-cn-beijing.aliyuncs.com/md/20200808220153.png)

## 7.3 搭建分片节点副本集

### 7.3.1 第一套shard副本集

使用 `172.17.238.235` 服务器，启动3个MongoDB服务

**准备存放数据和日志的目录**

```sh
mkdir -p /mongodb/sharded_cluster/myshardrs01_27017/log
mkdir -p /mongodb/sharded_cluster/myshardrs01_27017/data/db
mkdir -p /mongodb/sharded_cluster/myshardrs01_27117/log
mkdir -p /mongodb/sharded_cluster/myshardrs01_27117/data/db
mkdir -p /mongodb/sharded_cluster/myshardrs01_27217/log
mkdir -p /mongodb/sharded_cluster/myshardrs01_27217/data/db
```

新建配置文件

```sh
vim /mongodb/sharded_cluster/myshardrs01_27017/mongod.conf
```

myshardrs01_27017配置内容

```yaml
systemLog:
  #日志输出为文件 
  destination: file
  #日志文件的路径 
  path: "/mongodb/sharded_cluster/myshardrs01_27017/log/mongod.log"
  #mongod实例重新启动时，会将新条目附加到现有日志文件的末尾。 
  logAppend: true 
storage: 
  #数据目录。
  dbPath: "/mongodb/sharded_cluster/myshardrs01_27017/data/db" 
  journal:
    #启用持久性日志
    enabled: true
processManagement:
  #启用守护进程模式。 
  fork: true
  #指定进程ID的文件位置
  pidFilePath: "/mongodb/sharded_cluster/myshardrs01_27017/log/mongod.pid"
net:
  #服务实例绑定的IP
  bindIp: 0.0.0.0
  #绑定的端口 
  port: 27017
replication:
  #副本集的名称 
  replSetName: myshardrs01
sharding:
  #分片角色，shard
  clusterRole: shardsvr
```

myshardrs01_27117配置内容

```sh
vim /mongodb/sharded_cluster/myshardrs01_27117/mongod.conf
```

```yaml
systemLog:
  #日志输出为文件 
  destination: file
  #日志文件的路径 
  path: "/mongodb/sharded_cluster/myshardrs01_27117/log/mongod.log"
  #mongod实例重新启动时，会将新条目附加到现有日志文件的末尾。 
  logAppend: true 
storage: 
  #数据目录。
  dbPath: "/mongodb/sharded_cluster/myshardrs01_27117/data/db" 
  journal:
    #启用持久性日志
    enabled: true
processManagement:
  #启用守护进程模式。 
  fork: true
  #指定进程ID的文件位置
  pidFilePath: "/mongodb/sharded_cluster/myshardrs01_27117/log/mongod.pid"
net:
  #服务实例绑定的IP
  bindIp: 0.0.0.0
  #绑定的端口 
  port: 27117
replication:
  #副本集的名称 
  replSetName: myshardrs01
sharding:
  #分片角色，shard
  clusterRole: shardsvr
```

myshardrs01_27217配置内容

```sh
vim /mongodb/sharded_cluster/myshardrs01_27217/mongod.conf
```

```yaml
systemLog:
  #日志输出为文件 
  destination: file
  #日志文件的路径 
  path: "/mongodb/sharded_cluster/myshardrs01_27217/log/mongod.log"
  #mongod实例重新启动时，会将新条目附加到现有日志文件的末尾。 
  logAppend: true 
storage: 
  #数据目录。
  dbPath: "/mongodb/sharded_cluster/myshardrs01_27217/data/db" 
  journal:
    #启用持久性日志
    enabled: true
processManagement:
  #启用守护进程模式。 
  fork: true
  #指定进程ID的文件位置
  pidFilePath: "/mongodb/sharded_cluster/myshardrs01_27217/log/mongod.pid"
net:
  #服务实例绑定的IP
  bindIp: 0.0.0.0
  #绑定的端口 
  port: 27217
replication:
  #副本集的名称 
  replSetName: myshardrs01
sharding:
  #分片角色，shard
  clusterRole: shardsvr
```

**启动三个服务**

```sh
/usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myshardrs01_27017/mongod.conf

/usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myshardrs01_27117/mongod.conf

/usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myshardrs01_27217/mongod.conf
```

**初始化副本集**

连接任意一个节点

```sh
/usr/local/mongodb/bin/mongo --host 172.17.238.235 --port 27017
```

初始化副本集（如果副本集名称有问题，记得修改名称）

```sh
rs.initiate()
```

添加副本节点

```sh
rs.add("172.17.238.235:27117")
```

添加仲裁节点

```sh
rs.addArb("172.17.238.235:27217")
```

### 7.3.2 第二套shard副本集

使用 `172.17.238.236` 服务器，启动3个MongoDB服务

**准备存放数据和日志的目录**

```sh
mkdir -p /mongodb/sharded_cluster/myshardrs02_27017/log
mkdir -p /mongodb/sharded_cluster/myshardrs02_27017/data/db
mkdir -p /mongodb/sharded_cluster/myshardrs02_27117/log
mkdir -p /mongodb/sharded_cluster/myshardrs02_27117/data/db
mkdir -p /mongodb/sharded_cluster/myshardrs02_27217/log
mkdir -p /mongodb/sharded_cluster/myshardrs02_27217/data/db
```

新建配置文件

```sh
vim /mongodb/sharded_cluster/myshardrs02_27017/mongod.conf
```

myshardrs02_27017配置内容

```yaml
systemLog:
  #日志输出为文件 
  destination: file
  #日志文件的路径 
  path: "/mongodb/sharded_cluster/myshardrs02_27017/log/mongod.log"
  #mongod实例重新启动时，会将新条目附加到现有日志文件的末尾。 
  logAppend: true 
storage: 
  #数据目录。
  dbPath: "/mongodb/sharded_cluster/myshardrs02_27017/data/db" 
  journal:
    #启用持久性日志
    enabled: true
processManagement:
  #启用守护进程模式。 
  fork: true
  #指定进程ID的文件位置
  pidFilePath: "/mongodb/sharded_cluster/myshardrs02_27017/log/mongod.pid"
net:
  #服务实例绑定的IP
  bindIp: 0.0.0.0
  #绑定的端口 
  port: 27017
replication:
  #副本集的名称 
  replSetName: myshardrs02
sharding:
  #分片角色，shard
  clusterRole: shardsvr
```

myshardrs02_27117配置内容

```sh
vim /mongodb/sharded_cluster/myshardrs02_27117/mongod.conf
```

```yaml
systemLog:
  #日志输出为文件 
  destination: file
  #日志文件的路径 
  path: "/mongodb/sharded_cluster/myshardrs02_27117/log/mongod.log"
  #mongod实例重新启动时，会将新条目附加到现有日志文件的末尾。 
  logAppend: true 
storage: 
  #数据目录。
  dbPath: "/mongodb/sharded_cluster/myshardrs02_27117/data/db" 
  journal:
    #启用持久性日志
    enabled: true
processManagement:
  #启用守护进程模式。 
  fork: true
  #指定进程ID的文件位置
  pidFilePath: "/mongodb/sharded_cluster/myshardrs02_27117/log/mongod.pid"
net:
  #服务实例绑定的IP
  bindIp: 0.0.0.0
  #绑定的端口 
  port: 27117
replication:
  #副本集的名称 
  replSetName: myshardrs02
sharding:
  #分片角色，shard
  clusterRole: shardsvr
```

myshardrs02_27217配置内容

```sh
vim /mongodb/sharded_cluster/myshardrs02_27217/mongod.conf
```

```yaml
systemLog:
  #日志输出为文件 
  destination: file
  #日志文件的路径 
  path: "/mongodb/sharded_cluster/myshardrs02_27217/log/mongod.log"
  #mongod实例重新启动时，会将新条目附加到现有日志文件的末尾。 
  logAppend: true 
storage: 
  #数据目录。
  dbPath: "/mongodb/sharded_cluster/myshardrs02_27217/data/db" 
  journal:
    #启用持久性日志
    enabled: true
processManagement:
  #启用守护进程模式。 
  fork: true
  #指定进程ID的文件位置
  pidFilePath: "/mongodb/sharded_cluster/myshardrs02_27217/log/mongod.pid"
net:
  #服务实例绑定的IP
  bindIp: 0.0.0.0
  #绑定的端口 
  port: 27217
replication:
  #副本集的名称 
  replSetName: myshardrs02
sharding:
  #分片角色，shard
  clusterRole: shardsvr
```

**启动三个服务**

```sh
/usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myshardrs02_27017/mongod.conf

/usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myshardrs02_27117/mongod.conf

/usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myshardrs02_27217/mongod.conf
```

**初始化副本集**

连接任意一个节点

```sh
/usr/local/mongodb/bin/mongo --host 172.17.238.236 --port 27017
```

初始化副本集（如果副本集名称有问题，记得修改名称）

```sh
rs.initiate()
```

添加副本节点

```sh
rs.add("172.17.238.236:27117")
```

添加仲裁节点

```sh
rs.addArb("172.17.238.236:27217")
```

## 7.4 搭建配置节点副本集

使用 `172.17.238.237` 服务器，启动3个MongoDB服务

**准备存放数据和日志的目录**

```sh
mkdir -p /mongodb/sharded_cluster/myconfrs01_27017/log
mkdir -p /mongodb/sharded_cluster/myconfrs01_27017/data/db
mkdir -p /mongodb/sharded_cluster/myconfrs01_27117/log
mkdir -p /mongodb/sharded_cluster/myconfrs01_27117/data/db
mkdir -p /mongodb/sharded_cluster/myconfrs01_27217/log
mkdir -p /mongodb/sharded_cluster/myconfrs01_27217/data/db
```

新建配置文件

```sh
vim /mongodb/sharded_cluster/myconfrs01_27017/mongod.conf
```

myconfrs01_27017配置内容

```yaml
systemLog:
  #日志输出为文件 
  destination: file
  #日志文件的路径 
  path: "/mongodb/sharded_cluster/myconfrs01_27017/log/mongod.log"
  #mongod实例重新启动时，会将新条目附加到现有日志文件的末尾。 
  logAppend: true 
storage: 
  #数据目录。
  dbPath: "/mongodb/sharded_cluster/myconfrs01_27017/data/db" 
  journal:
    #启用持久性日志
    enabled: true
processManagement:
  #启用守护进程模式。 
  fork: true
  #指定进程ID的文件位置
  pidFilePath: "/mongodb/sharded_cluster/myconfrs01_27017/log/mongod.pid"
net:
  #服务实例绑定的IP
  bindIp: 0.0.0.0
  #绑定的端口 
  port: 27017
replication:
  #副本集的名称 
  replSetName: myconfrs01
sharding:
  #分片角色，config
  clusterRole: configsvr
```

myconfrs01_27117配置内容

```sh
vim /mongodb/sharded_cluster/myconfrs01_27117/mongod.conf
```

```yaml
systemLog:
  #日志输出为文件 
  destination: file
  #日志文件的路径 
  path: "/mongodb/sharded_cluster/myconfrs01_27117/log/mongod.log"
  #mongod实例重新启动时，会将新条目附加到现有日志文件的末尾。 
  logAppend: true 
storage: 
  #数据目录。
  dbPath: "/mongodb/sharded_cluster/myconfrs01_27117/data/db" 
  journal:
    #启用持久性日志
    enabled: true
processManagement:
  #启用守护进程模式。 
  fork: true
  #指定进程ID的文件位置
  pidFilePath: "/mongodb/sharded_cluster/myconfrs01_27117/log/mongod.pid"
net:
  #服务实例绑定的IP
  bindIp: 0.0.0.0
  #绑定的端口 
  port: 27117
replication:
  #副本集的名称 
  replSetName: myconfrs01
sharding:
  #分片角色，config
  clusterRole: configsvr
```

myconfrs01_27217配置内容

```sh
vim /mongodb/sharded_cluster/myconfrs01_27217/mongod.conf
```

```yaml
systemLog:
  #日志输出为文件 
  destination: file
  #日志文件的路径 
  path: "/mongodb/sharded_cluster/myconfrs01_27217/log/mongod.log"
  #mongod实例重新启动时，会将新条目附加到现有日志文件的末尾。 
  logAppend: true 
storage: 
  #数据目录。
  dbPath: "/mongodb/sharded_cluster/myconfrs01_27217/data/db" 
  journal:
    #启用持久性日志
    enabled: true
processManagement:
  #启用守护进程模式。 
  fork: true
  #指定进程ID的文件位置
  pidFilePath: "/mongodb/sharded_cluster/myconfrs01_27217/log/mongod.pid"
net:
  #服务实例绑定的IP
  bindIp: 0.0.0.0
  #绑定的端口 
  port: 27217
replication:
  #副本集的名称 
  replSetName: myconfrs01
sharding:
  #分片角色，config
  clusterRole: configsvr
```

**启动三个服务**

```sh
/usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myconfrs01_27017/mongod.conf

/usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myconfrs01_27117/mongod.conf

/usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myconfrs01_27217/mongod.conf
```

**初始化副本集**

连接任意一个节点

```sh
/usr/local/mongodb/bin/mongo --host 172.17.238.237 --port 27017
```

初始化副本集（如果副本集名称有问题，记得修改名称）

```sh
rs.initiate()
```

添加两个副本节点（配置节点不需要仲裁者）

```sh
rs.add("172.17.238.237:27117")
rs.add("172.17.238.237:27217")
```

## 7.5 搭建路由节点

我们在 `172.17.238.235` 和 `172.17.238.236` 上分别搭建一个路由节点

### 7.5.1 第一个路由节点

创建存放日志的目录

```sh
mkdir -p /mongodb/sharded_cluster/mymongos_27018/log
```

创建配置文件

```sh
vi /mongodb/sharded_cluster/mymongos_27018/mongos.conf
```

配置文件内容

```yaml
systemLog:
  #日志输出为文件 
  destination: file
  #日志文件的路径 
  path: "/mongodb/sharded_cluster/mymongos_27018/log/mongod.log"
  #mongod实例重新启动时，会将新条目附加到现有日志文件的末尾。 
  logAppend: true 
processManagement:
  #启用守护进程模式。 
  fork: true
  #指定进程ID的文件位置
  pidFilePath: "/mongodb/sharded_cluster/mymongos_27018/log/mongod.pid"
net:
  #服务实例绑定的IP
  bindIp: 0.0.0.0
  #绑定的端口 
  port: 27018
sharding:
  #指定配置节点副本集
  configDB: myconfrs01/172.17.238.237:27017,172.17.238.237:27117,172.17.238.237:27217
```

启动mongos

```sh
/usr/local/mongodb/bin/mongos -f /mongodb/sharded_cluster/mymongos_27018/mongos.conf
```

登录mongos

```sh
/usr/local/mongodb/bin/mongo --host 172.17.238.235 --port 27018
```

目前只是连接了路由节点，还不能通过路由节点去操作分片集群，因此即使连接了，也无法写入数据。

### 7.5.2 在路由节点上设置分片

添加分片

```sh
sh.addShard("副本集名称/ip:port,ip:port,ip:port")
```

现在我们将第一套副本集添加进来

```sh
sh.addShard("myshardrs01/172.17.238.235:27017,172.17.238.235:27117,172.17.238.235:27217")
```

查看分片状态

```sh
sh.status()
```

将第二套副本集添加进来

```sh
sh.addShard("myshardrs02/172.17.238.236:27017,172.17.238.236:27117,172.17.238.236:27217")
```

查看分片状态

```sh
sh.status()
```

如果添加分片失败，可以移除分片。当只剩下最后一个分片时，无法删除。移除时会自动转移分片数据，转移完成后，需要再执行一次删除分片命令

```sh
use admin
db.runCommand({removeShard: "myshardrs01"})
```

***

**开启分片功能**

需要先开启数据库的分片功能

```sh
sh.enableSharding("库名")
```

开启库的分片功能后，才能对集合进行分片

```sh
sh.shardCollection("库名.集合名", {"key", 1})
```

在bbs数据库配置sharding

```sh
sh.enableSharding("bbs")
```

查看分片状态

```sh
sh.status()
```

***

**集合分片**

```sh
sh.shardCollection(namespace, key, unique)
```
如果片键添加错误，可以查看config下collections集合中的数据，删掉对应的片键即可

| 参数      | 描述                                                         |
| --------- | ------------------------------------------------------------ |
| namespace | 命名空间，格式用 `库名.集合名`                               |
| key       | 片键，片键是每条记录都必须包含，并且建立了索引的单个字段或者复合字段。MongoDB按照片键将数据划分到不同的数据库中，将数据库均匀的分不到所有分片中。MongoDB使用哈希的分片方式（随机平均分配）或者基于范围的分片方式（数值大小分配）。 |
| unique    | 当值为true的情况下，片键字段上会限制为唯一索引。哈希策略片键不支持唯一索引，默认是false |

#### 7.5.2.1 哈希策略分片

基于哈希分片，mongodb会计算一个字段的哈希值，并用这个值来创建数据块。拥有相近片键的文档，很可能不会存储在同一个数据块中，这样就使数据的分离性更好些。

使用name作为片键

```sh
sh.shardCollection("bbs.user", {"_id": "hashed"})
```

查看分片状态

```sh
sh.status()
```



#### 7.5.2.2 范围策略分片

范围策略分片，MongoDB按照片键的范围把数据分成不同部分。

假设有一条从0到无穷的直线，每一个片键都是这条线上的点，MongoDB就是把这条线划分为更短的不可重叠的片段，这就是每个数据块。每个数据块包含了片键一定范围内的数据。

基于范围策略的分片虽然范围查询性能相对较高，但是存在着比较严重的弊端。如果片键是线性增长的，就可能会导致一定时间内所有的请求都集中到了某个特定的数据块中，导致那个数据块的压力过大，违背了我们搭建集群的初衷。

而基于哈希的分片策略牺牲了范围查询的性能，但是保证了数据的均衡，保证了系统的高可用性。

无特殊情况，一般就只使用哈希策略分片，并且一般都是使用_id作为片键。

***

剩余的鸡哥命令

**显示集群的详细信息**

```sh
db.printShardingStatus()
```

**查看当前均衡器状态（了解）**

```sh
sh.getBalancerState()
```

### 7.5.3 分片后插入数据测试

像user中插入1000条数据测试

```sh
use bbs
for(var i=1;i<=1000;i++) { db.user.insert({_id: i+"", name:"稽哥"+i}) }
```

插入完毕后查看数量

```sh
db.user.count()
```

> 从路由上插入的数据，必须要包含片键，否则无法插入

分别登录两个分片副本集的主节点，查看数量

第一个分片副本集

```sh
/usr/local/mongodb/bin/mongo --host 172.17.238.235 --port 27017
```

```sh
use bbs
db.user.count()
```

第二个分片副本集

```sh
/usr/local/mongodb/bin/mongo --host 172.17.238.236 --port 27017
```

```sh
use bbs
db.user.count()
```

可以看到，1000条数据较为均匀的分布到了两个分片上。这种方式非常易于水平扩展，一旦数据需要更大的空间，再增加分片即可。

### 7.5.4 第二个路由节点

第二个节点在 `172.17.238.236` 上搭建

创建存放数据和日志的目录

```sh
mkdir -p /mongodb/sharded_cluster/mymongos_27018/log
```

创建配置文件

```sh
vi /mongodb/sharded_cluster/mymongos_27018/mongos.conf
```

配置文件内容

```yaml
systemLog:
  #日志输出为文件 
  destination: file
  #日志文件的路径 
  path: "/mongodb/sharded_cluster/mymongos_27018/log/mongod.log"
  #mongod实例重新启动时，会将新条目附加到现有日志文件的末尾。 
  logAppend: true 
processManagement:
  #启用守护进程模式。 
  fork: true
  #指定进程ID的文件位置
  pidFilePath: "/mongodb/sharded_cluster/mymongos_27018/log/mongod.pid"
net:
  #服务实例绑定的IP
  bindIp: 0.0.0.0
  #绑定的端口 
  port: 27018
sharding:
  #指定配置节点副本集
  configDB: myconfrs01/172.17.238.237:27017,172.17.238.237:27117,172.17.238.237:27217
```

启动mongos

```sh
/usr/local/mongodb/bin/mongos -f /mongodb/sharded_cluster/mymongos_27018/mongos.conf
```

登录mongos

```sh
/usr/local/mongodb/bin/mongo --host 172.17.238.236 --port 27018
```

登录后我们发现，第二个路由并不需要配置，因为分片配置都保存到了配置服务器中。

### 7.5.5 路由搭建过程中出错如何处理

我们使用 `sh.status()` 查看路由的状态，发现有 `databases` 和 `shards` 两个属性，分别表示分片的数据库和分片的集群，我们只需要把这两个给清除就好了。

因为集群中的配置都在配置节点上，因此我们只需要修改 `config` 数据库即可.

```sh
use config()
show tables
```

```sh
mongos> use config
switched to db config
mongos> show tables
actionlog
changelog
chunks
collections
databases
lockpings
locks
migrations
mongos
shards
system.sessions
tags
transactions
version
```

**移除分片**

我们使用 `db.shards.find()` 发现，查询出来的数据，和 `sh.status()` 中 `shards` 的数据吻合，因此，我们将该集合需要删除的分片，给删除

```sh
db.shards.remove({})
```

**移除分片数据库**

我们使用 `db.databases.find()` 发现，里面的数据正是我们配置的需要分片的数据库，直接删除即可.

```sh
db.databases.remove({_id: "bbs"})
```

此时，我们再使用 `sh.status()` 发现，已经没有了分片的配置。但是，我们的工作还没有结束。

**移除分片的集合**

我们使用 `db.collections.find()` 发现，有一条数据是我们配置的要分片的集合，直接删除这条数据

```sh
db.collections.remove({_id: "bbs.user"})
```

**移除数据块**

我们使用 `db.chunks.find()` 发现，里面还有一些和我们分片数据库相关的数据，全部删除

```sh
db.chunks.remove({_id: /^bbs.user/})
```

至此，我们完全移除了之前的分片配置



## 7.6 SpringDataMongoDB连接分片集群

当集群只有一个路由时，配置和打击板mongodb配置是一模一样的。

多个路由时，配置和副本集配置类似

```sh
spring:
  data:
    mongodb:
      uri: mongodb://172.17.238.235:27018,172.17.238.236:27018/bbs
```

## 7.7 关闭分片

使用shutdownServer依次关闭，按照分片服务器、配置服务器、路由的顺序依次关闭。而分片服务器和配置服务器分别又是副本集，因此又要按照仲裁者、子节点、主节点进行关闭。关闭的命令如下

```sh
rs.stepDowdn()
use admin
db.shutdownServer()
```

## 7.8 安全认证（了解）

### 7.8.1 复制keyfile

复制我们在副本集那一节创建的证书，到每台服务器每个节点下面（此步略）

### 7.8.2 修改配置文件

修改每个服务节点的配置文件如下

```yaml
security:
  keyFile: /mongodb/shard_cluster/myshardrs01/mongo.keyfile
  authorization: enabled
```

而mongos的配置则如下

```yaml
security:
  keyFile: /mongodb/shard_cluster/myshardrs01/mongo.keyfile
  authorization: enabled
```

> mongos的配置文件比mongod少了authorization，mongos并不用于存储数据，仅仅是路由的作用

### 7.8.3 重启路由节点

先关闭分片

之后必须依次启动配置节点、分片节点、路由节点（命令略）

### 7.8.4 创建账号

先通过localhost登录任意一个mongos可以

再创建一个管理员账号

```sh
use admin
db.createUser({user: "mongoroot", pwd: "123456", roles: ["root"]})
```

创建一个普通权限账号

先通过账号密码重新连接，之后执行下面的命令

```sh
use bbs
db.createUser({user: "jigege", pwd: "123456", roles: [{role: "readWrite", db: "bbs"}]})
```

> 分片中的账号信息不涉及同步问题。通过mongos添加的账号只会保存到配置节点

### 7.8.5 SpringDataMongoDB连接认证

和上面一样，加上账号密码即可，略。