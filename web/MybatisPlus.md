## 2，特性

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







# #和$的区别



* 传入的参数在SQL中显示不同
  * \#传入的参数在SQL中显示为字符串，自动加引号
  * $传入的参数在SqL中直接显示为传入的值

* \#防注入的风险（语句拼接）,$无法防止注入

* $方式一般用于传入数据库对象，例如传入表名。

* ==排序时使用order by 动态参数时用$==





# 一级缓存



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



# 二级缓存



指mapper映射文件。二级缓存的作用域是同一个namespace下的mapper映射文件内容，多个SqlSession共享。Mybatis需要手动设置启动二级缓存。

二级缓存是默认启用的(要生效需要对每个Mapper进行配置)，如想取消，则可以通过Mybatis配置文件中的元素下的子元素来指定cacheEnabled为false



# 分页原理



Mybatis 使用 RowBounds 对象进行分页，也可以直接编写 sql 实现分页，也可 以使用 Mybatis 的分页插件。 2）分页插件的原理：实现 Mybatis 提供的接口，实现自定义插件，在插件的拦截方法内拦截待执行的 sql，然后重写 sql。 举例：select * from student，拦截 sql 后重写为：select t.* from （select * from student）t limit 0，10





















