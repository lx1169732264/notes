# MP



## 特性

1. 无侵入：Mybatis-Plus 在 Mybatis 的基础上进行扩展，只做增强不做改变，引入 Mybatis-Plus 不会对您现有的 Mybatis 构架产生任何影响，而且 MP 支持所有 

2. Mybatis 原生的特性

3. 依赖少：仅仅依赖 Mybatis 以及 Mybatis-Spring

4. 损耗小：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作

5. 预防Sql注入：内置 Sql 注入剥离器，有效预防Sql注入攻击

6. 通用CRUD操作：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求

7. 多种主键策略：支持多达4种主键策略（内含分布式唯一ID生成器），可自由配置，完美解决主键问题

8. 支持热加载：Mapper 对应的 XML 支持热加载，对于简单的 CRUD 操作，甚至可以无 XML 启动

9. 支持ActiveRecord：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可实现基本 CRUD 操作

10. 支持代码生成：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用（P.S. 比 Mybatis 官方的 Generator 更加强大！）

11. 支持自定义全局通用操作：支持全局通用方法注入（ Write once, use anywhere ）

12. 支持关键词自动转义：支持数据库关键词（order、key......）自动转义，还可自定义关键词

13. 内置分页插件：基于 Mybatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通List查询

14. 内置性能分析插件：可输出 Sql 语句以及其执行时间，建议开发测试时启用该功能，能有效解决慢查询

15. 内置全局拦截插件：提供全表 delete 、 update 操作智能分析阻断，预防误操作







## #和$的区别



* 传入的参数在SQL中显示不同
  * \#传入的参数在SQL中显示为字符串，自动加引号
  * $传入的参数在SqL中直接显示为传入的值

* \#防注入的风险（语句拼接）,$无法防止注入

* $方式一般用于传入数据库对象，例如传入表名。

* ==排序时使用order by 动态参数时用$==





## 一级缓存



指Session缓存。一级缓存的作用域默认是一个SqlSession。Mybatis默认开启一级缓存。
也就是在同一个SqlSession中，执行相同的查询SQL，第一次会去数据库进行查询，并写到缓存中；
第二次以后是直接去缓存中取。
当执行SQL查询中间发生了增删改的操作，MyBatis会把SqlSession的缓存清空。

一级缓存的范围有SESSION和STATEMENT两种，默认是SESSION，如果不想使用一级缓存，可以把一级缓存的范围指定为STATEMENT，这样每次执行完一个Mapper中的语句后都会将一级缓存清除。
如果需要更改一级缓存的范围，可以在Mybatis的配置文件中，在下通过localCacheScope指定。

| 1    | `<setting name=``"localCacheScope"` `value=``"STATEMENT"``/>` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

建议不需要修改

**需要注意的是**
当Mybatis整合Spring后，直接通过Spring注入Mapper的形式，如果不是在同一个事务中每个Mapper的每次查询操作都对应一个全新的SqlSession实例，这个时候就不会有一级缓存的命中，但是在同一个事务中时共用的是同一个SqlSession。
如有需要可以启用二级缓存



## 二级缓存



指mapper映射文件。二级缓存的作用域是同一个namespace下的mapper映射文件内容，多个SqlSession共享。Mybatis需要手动设置启动二级缓存。

二级缓存是默认启用的(要生效需要对每个Mapper进行配置)，如想取消，则可以通过Mybatis配置文件中的元素下的子元素来指定cacheEnabled为false



## 分页原理



Mybatis 使用 RowBounds 对象进行分页，也可以直接编写 sql 实现分页，也可 以使用 Mybatis 的分页插件。 2）分页插件的原理：实现 Mybatis 提供的接口，实现自定义插件，在插件的拦截方法内拦截待执行的 sql，然后重写 sql。 举例：select * from student，拦截 sql 后重写为：select t.* from （select * from student）t limit 0，10



## 自增主键





AUTO	数据库自增 依赖数据库
NONE	未设置主键类型（默认雪花算法,可以通过注册填充插件进行填充）

//下面这三种类型,只有当插入对象id为空时 才会自动填充。
ID_WORKER	全局唯一（idWorker）数值类型

ID_WORKER_STR(5)	全局唯一（idWorker的字符串表示）

UUID	全局唯一（UUID）















# JPA



8种锁

~~Read,WRITE~~



OPTIMISTIC	乐观读	**默认**

OPTIMISTIC_FORCE_INCREMENT	乐观写




PESSIMISTIC_READ	悲观读,读之间共享,保证数据在读期间不受修改

PESSIMISTIC_FORCE_INCREMENT	悲观读,事务结束后**增加实体的版本号**，即使实体没有修改 

PESSIMISTIC_WRITE	悲观写,当多个并发更新失败几率较高时使用



NONE	无锁





## 创建查询



query builder机制内置为构建约束查询库的实体。 带前缀的机制`findXXBy`,`readAXXBy`,`queryXXBy`,`countXXBy`, `getXXBy`自动解析的其余部分。进一步引入子句可以包含表达式等`Distinct`设置不同的条件创建查询。 然而,第一个`By`作为分隔符来表示实际的标准的开始。 在一个非常基础的查询,可以定义条件`And`或者`Or`。





IgnoreCase	忽略大小写

OrderBy…Asc/Desc	排序



假设一个`Person`有一个`Address`与一个`Zipcode`

```java
List<Person> findByAddressZipCode(ZipCode zipCode);
//通过 _ 分割,降低错误概率	但实体类需要避免使用 _
->
List<Person> findByAddress_ZipCode(ZipCode zipCode);
```



入参与方法同名

```java
Page<User> findByLastname(String lastname, Pageable pageable);

Slice<User> findByLastname(String lastname, Pageable pageable);

List<User> findByLastname(String lastname, Sort sort);

List<User> findByLastname(String lastname, Pageable pageable);
```



限制个数

查询方法的结果通过关键字first或者top来限制,它们可以交替使用

在top/firest后添加数字来表示返回最大的结果数,默认1

```java
User findFirstByOrderByLastnameAsc();

User findTopByOrderByAgeDesc();

Page<User> queryFirst10ByLastname(String lastname, Pageable pageable);

Slice<User> findTop3ByLastname(String lastname, Pageable pageable);

List<User> findFirst10ByLastname(String lastname, Sort sort);

List<User> findTop10ByLastname(String lastname, Pageable pageable);
```



返回流

```java
@Query("select u from User u")
Stream<User> findAllByCustomQueryAndStream();
```



异步查询结果

```java
@Async
Future<User> findByFirstname(String firstname);	//返回值java.util.concurrent.Future

@Async
CompletableFuture<User> findOneByFirstname(String firstname);	//java.util.concurrent.CompletableFuture

@Async
ListenableFuture<User> findOneByLastname(String lastname);	//org.springframework.util.concurrent.ListenableFuture
```





## 关键词



And --- 等价于 SQL 中的 and 关键字，比如 findByUsernameAndPassword(String user, Striang pwd)；

Or --- 等价于 SQL 中的 or 关键字，比如 findByUsernameOrAddress(String user, String addr)；

Between --- 等价于 SQL 中的 between 关键字，比如 findBySalaryBetween(int max, int min)；

LessThan --- 等价于 SQL 中的 "<"，比如 findBySalaryLessThan(int max)；

GreaterThan --- 等价于 SQL 中的">"，比如 findBySalaryGreaterThan(int min)；

IsNull --- 等价于 SQL 中的 "is null"，比如 findByUsernameIsNull()；

IsNotNull --- 等价于 SQL 中的 "is not null"，比如 findByUsernameIsNotNull()；

NotNull --- 与 IsNotNull 等价；

Like --- 等价于 SQL 中的 "like"，比如 findByUsernameLike(String user)；

NotLike --- 等价于 SQL 中的 "not like"，比如 findByUsernameNotLike(String user)；

OrderBy --- 等价于 SQL 中的 "order by"，比如 findByUsernameOrderBySalaryAsc(String user)；

Not --- 等价于 SQL 中的 "！ ="，比如 findByUsernameNot(String user)；

In --- 等价于 SQL 中的 "in"，比如 findByUsernameIn(Collection<String> userList) ，方法的参数可以是 Collection 类型，也可以是数组或者不定长参数；

NotIn --- 等价于 SQL 中的 "not in"，比如 findByUsernameNotIn(Collection<String> userList) ，方法的参数可以是 Collection 类型，也可以是数组或者不定长参数；









@Version

在实体 bean 中使用,会自动对该实体使用**乐观锁**,无需锁声明

