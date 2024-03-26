![](image.assets/RPC.jpg)





分布式系统是若干独立计算机的集合，这些计算机对于用户来说就像单个相关系统.是建立在网络之上的软件系统。



* 单一应用架构
  * 网站流量小，所有功能部署在一起，以减少部署节点和成本。此时，用于简化增删改查的数据访问框架**(ORM)是关键。**
    * 缺点： 1、扩展难 2、协同开发难3、不利于升级维护
  * 访问量大，单一应用增加机器带来的加速度越来越小，将应用拆成互不相干的几个应用，以提升效率。此时，用于加速前端页面开发的**Web框架(MVC)是关键**。
    * 缺点： 公用模块无法重复利用，开发性的浪费

* 分布式服务架构
  * 当垂直应用越来越多，应用之间交互不可避免，将核心业务抽取出来，作为独立的服务。此时，用于提高业务复用及整合的分布式服务框架(**RPC)是关键。**

* 流动计算架构
  * 服务过多，需增加调度中心基于访问压力实时管理集群容量，提高集群利用率。此时，用于提高机器利用率的**资源调度和治理中心(SOA)是关键。**



==节点角色==

| Provider  |                                 |
| --------- | ------------------------------- |
| Consumer  |                                 |
| Registry  | 服务注册/发现的注册中心         |
| Monitor   | 统计服务调用次数/时间的监控中心 |
| Container | 服务运行容器                    |





# 安装(windows)

https://github.com/apache/dubbo-admin/tree/master



修改dubbo-admin-master	application.properties



 ![](.\image.assets\wps6.jpg)

打包mvn package

运行java -jar dubbo-admin-0.0.1-SNAPSHOT.jar



## 监控中心 monitor



```
进入dubbo-monitor-simple\src\main\resources\conf	修改 dubbo.properties文件的ip地址

mvn clean package -Dmaven.test.skip=true

解压 tar.gz 文件，并运行start.bat
```



Simple Monitor 挂掉不会影响到 Consumer 和 Provider 的调用，所以用于生产环境不会有风险

Simple Monitor 采用磁盘存储统计信息，请注意安装机器的磁盘限制，如果要集群，建议用mount共享磁盘。



# 整合springboot



三种方式

方式1：引入dubbo-starter，在application.properties配置属性，使用@Service【暴露服务】使用@Reference【引用服务】

方式2：保留xml配置文件; 导入dubbo-starter，使用@ImportResource导入dubbo的配置文件即可

方式3：使用注解API的方式， 将每一个组件手动创建到容器中,让dubbo来扫描组件



## 要点



==provider需要暴露服务,用apache.duubo的@service,既能注册bean,又能暴露服务==

==consumer只需要注册bean,使用spring的@service==



* **provider项目无需web依赖**,不需要向浏览器提供服务

* consumer	需要web依赖

* provider和consumer**部署在不同机器,端口才允许重复**

* **启动类需要加上@EnableDubbo注解**



由于provider和consumer都依赖于interface,部分公共依赖可以写在interface中



## provider



* 启动类

```java
@EnableDubbo
```

* 依赖

服务者只需要勾选devTools和spring Configuration就行,由于依赖interface项目所以不需要lombok

```java
       <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.7.3</version>
        </dependency>
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-dependencies-zookeeper</artifactId>
            <version>2.7.3</version>
            <type>pom</type>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```

* ServiceImpl	(provider)

```java
apache.duubo	@service
```

* properties

```shell
dubbo.application.name=boot-ego-user-service-provider
dubbo.registry.address=zookeeper://127.0.0.1:2181
dubbo.protocol.name=dubbo
dubbo.protocol.port=20880
#dubbo.monitor.protocol=registry
```



## consumer



* 启动类配置

```
@EnableDubbo //启动dubbo
```

* ServiceImpl

```java
@Service	Sping注解

@Reference		远程调用服务注解
```

* properties

```shell
dubbo.application.name=boot-ego-order-service-consumer
dubbo.registry.address=zookeeper://127.0.0.1:2181
dubbo.monitor.protocol=registry
server.port=8888
```

 

# 服务暴露过程



Dubbo 会在 Spring 实例化完 bean 之后，在刷新容器最后一步发布 ContextRefreshEvent 事件的时候，通知实现了 ApplicationListener 的 ServiceBean 类进行回调 onApplicationEvent 事件方法，Dubbo 会在这个方法中调用 ServiceBean 父类 ServiceConfig 的 export 方法，而该方法真正实现了服务的（异步或者非异步）发布。





# 负载均衡LoadBalance



* Random LoadBalance	基于权重随机调用	**默认**

* RoundRobin LoadBalance	基于**权重轮循**调用	权重大的被轮循到的次数多

* LeastActive LoadBalance   基于活跃数的调用	根据调用前后计数差,得到"延迟数"

* ConsistentHash LoadBalance	基于一致hash的调用

```
@Service(weight = 100, loadbalance = "roundrobin")	//provider设置权重与负载均衡
```



## 服务降级 2.2+



**服务器压力剧增时，对一些服务有策略地不处理或简单处理，从而释放服务器资源**



consumer禁用后,consumer发起的请求会被直接返回null,并不会报错	**不影响本地存根的执行**

consumer允许容错	在provider的响应时间过长,将直接返回null,并不会报错	能够减轻服务器的压力



## dubbo直连



绕过zookeeper注册,直接消费服务	指定服务提供者的ip与端口

![](image.assets/image-20200815124050556.png)



# 超时处理

 

## 提供者超时

可以精确到某个接口中的方法

![](image.assets/image-20200812154628800.png)

 消费者超时

![](image.assets/image-20200812155020064.png)



## 重试



* 配置的次数为重试的次数,==总次数=重试+1==

```java
<dubbo:provider timeout="6000" retries="3"></dubbo:provider>
```



* ==幂等操作==	多次请求对数据没有影响	**修改幂等**,重复修改最终结果一致	**删除也幂等**
* 非幂等操作	==只有添加非幂等,重试为0==



## 集群容错



* 读操作用 Failover 失败自动切换，默认重试两次其他服务器

* 写操作用 Failfast 快速失败，发一次调用失败就立即报错



| **Failover Cluster** | ==缺省==    失败自动切换。通常用于==读==操作                 |
| -------------------- | ------------------------------------------------------------ |
| **Failfast Cluster** | 快速失败，只发起一次调用，失败立即报错。通常用于非幂等性==写==操作 |
| **Failsafe Cluster** | 安全失败，出现异常时直接忽略。通常用于写入**审计日志**       |
| Failback Cluster     | 失败自动恢复，**记录失败请求，定时重发**。通常用于**消息通知** |
| Forking Cluster      | 并行调用多个服务器，只要一个成功即返回。通常用于**实时性要求较高的读操作**，但需要浪费更多服务资源。可通过 forks="2" 来设置最大并行数 |
| Broadcast Cluster    | 广播调用所有提供者，逐个调用，任意一台报错则报错 。通常用于通知所有提供者更新缓存或日志等本地资源信息 |





**集群模式配置**

```java
//注解
@Reference(timeout = 3000, cluster = "failover", retries = 0, stub = "com.lx.service.impl.StubUserServiceImpl")
    private UserService userService;

//xml
provider	<dubbo:service cluster="failsafe" />
consumer	<dubbo:reference cluster="failsafe" />
```





## 启动检查

  

* 消费者项目启动时,默认会检查提供者是否已注册,如果没有将启动失败

可以在方法/全局上配置启动不检查	check默认为true

```
@Reference(timeout = 3000, check = false)
```



## 配置优先级

有三个位置用于配置

​	方法>接口>全局	同级别的配置下==消费者的配置优先==

![](image.assets/image-20200813222238407.png)

![](image.assets/image-20200813222509289.png)

在@service/@Reference注解中指定超时时间,等价于接口配置

**超时,则会重新发两次请求**,依然超时则终止

一般***只在提供者进行配置***,提供者更清楚服务的状态



















# 灰度发布



新接口发布出现不兼容时，可以用版本号过渡，**版本号不同的服务相互不引用**。

之后进行**版本迁移**：

在低压力时间段，先**升级一半provider**为新版本	再将**所有consumer**升级为新版本

然后将剩下的一半提供者升级为新版本



版本号**支持正则匹配**,可以实现负载均衡



provider

```shell
 <!--声明要暴露的实现类的对象-->
    <bean id="userServiceImpl" class="service.impl.UserServiceImpl" ></bean>
    <!-- 指定需要暴露的服务 -->
    <dubbo:service timeout="2000" interface="service.UserService" ref="userServiceImpl" **version="1.0.0"**>
        <dubbo:method name="queryAllAddress" timeout="1000"></dubbo:method>
    </dubbo:service>

<!--声明要暴露的实现类的对象-->
    <bean id="userServiceImpl2" class="service.impl.UserServiceImpl2"></bean>
    <!-- 指定需要暴露的服务 -->
    <dubbo:service timeout="2000" interface="service.UserService" ref="userServiceImpl**2**" **version="1.0.1"**>
        <dubbo:method name="queryAllAddress" timeout="1000"></dubbo:method>
    </dubbo:service>

//在指定版本号之后,consumer必须指定版本号才能够正常调用
 <!--生成远程调用对象-->
<dubbo:reference timeout="3000" id="userService" interface="service.UserService" version="1.0.0" >
        <dubbo:method name="queryAllAddress" timeout="2000"></dubbo:method>
</dubbo:reference>
```



# 本地存根Stub

客户端通常只剩下接口，而实现全在服务器端，但**有时想在客户端也执行部分逻辑**，比如：做 ThreadLocal缓存/提前验证参数/调用失败后伪造容错数据等，此时就需要在 API 中带上Stub，客户端生成 Proxy 实例，会把 Proxy 通过构造函数传给 Stub，然后把 Stub 暴露给用户，Stub 可以决定要不要去调 Proxy。

**在调用之前或之后再执行一段逻辑,类似于切面**



* 注解方式

![](image.assets/image-20200815120037132.png)

 

* xml方式

```java
<!--生成远程调用对象-->
<dubbo:reference check="false" timeout="3000" id="userService" interface="service.UserService" version="1.0.0" stub="service.impl.StubUserServiceImpl">
        <dubbo:method name="queryAllAddress" timeout="2000"></dubbo:method>
</dubbo:reference>
```



```java
//存根类必须实现接口
public class StubUserServiceImpl implements UserService {

    //远程的服务接口对象
    private UserService userService;
    
	//对外提供生成接口对象的构造方法
    public StubUserServiceImpl(UserService userService) {
        this.userService = userService;
    }

    public List<UserAddress> queryAllAddress(String userId) throws InterruptedException {
        try {
            System.out.println("stub被执行");
            return userService.queryAllAddress(userId);
        } catch (Exception e) {
        //通过catch可以实现提供方的接口出现错误时,也能返回默认数据
            return Arrays.asList(new UserAddress(1, "error", "error"));        }  }}
```

 



# zookeeper宕机



注册中心宕机并不影响消费dubbo暴露的服务

![](.\image.assets\wps4.jpg)

可以看出,监控中心只负责同步消费者和提供者的数据

==在provider暴露完服务,consumer拉取服务列表之后,在本地就已经有缓存了,之后的服务调用都是consumer与provider的直接调用,不需要经过zookeeper==



* 数据中心宕掉后,仍能够通过**注册中心缓存**提供服务列表查询,但**不能注册新服务**

  * 注册中心集群，任意一台宕掉后，将自动切换到另一台

* 注册中心全部宕掉后，提供者和消费者仍能通过**本地缓存**通讯

* 提供者任意一台宕掉，不影响使用

* 提供者全部宕掉，消费者无法调用，无限重连

==即使zookeeper宕机,也不会造成严重的损失,实现高可用,减少系统不能提供服务的时间==











# Hystrix服务熔断



**回退机制**和**断路器功能**的线程和信号隔离，请求缓存和请求打包，以及监控和配置等功能



当提供者挂掉时,Hystrix可以提供默认调用的方法,当一段时间内提供者一直宕机,将引发服务熔断,不再向提供者请求服务,而直接调用默认方法



```
双方的启动类加上	@EnableHystrix    //启用hystrix
消费者启动类加上	@EnableCircuitBreaker	//启用hystrix的熔断保护
```



* 消费者加入依赖

```
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
			<version>2.2.4.RELEASE</version>
		</dependency>
```

 

* 定义具有服务熔断效果的Controller基类

@DefaultProperties	声明服务熔断后调用fallback()方法

fallbackMethod方法如果有参数,需要保持参数一致

```
@DefaultProperties(defaultFallback = "fallback")
public class BaseController {
    public AjaxResult fallback() {
        return AjaxResult.fail("服务器内部异常，请联系管理员");  }}
```



* 消费者Controller上 

  调用方法将经过Hystrix代理,当提供者出现问题/提供者不存在时,将调用fallback()方法

```
@HystrixCommand
```



**Hystrix类似本地存根**,都是在提供者无法正常提供服务时,返回一组默认的数据







# 设计原理

![](image.assets/dubbo.jpg)

·     config 配置层：对外配置接口，以 ServiceConfig, ReferenceConfig 为中心，可以直接初始化配置类，也可以通过 spring 解析配置生成配置类

·     proxy 服务代理层：服务接口透明代理，生成服务的客户端 Stub 和服务器端 Skeleton, 以 ServiceProxy 为中心，扩展接口为 ProxyFactory

·     registry 注册中心层：封装服务地址的注册与发现，以服务 URL 为中心，扩展接口为 RegistryFactory, Registry, RegistryService

·     **cluster 路由层**：封装多个提供者的**路由及负载均衡(4种)**，并桥接注册中心，以 Invoker 为中心，扩展接口为 Cluster, Directory, Router, LoadBalance

·     monitor 监控层：RPC 调用次数和调用时间监控，以 Statistics 为中心，扩展接口为 MonitorFactory, Monitor, MonitorService

·     protocol 远程调用层：封装 RPC 调用，以 Invocation, Result 为中心，扩展接口为 Protocol, Invoker, Exporter

·     exchange 信息交换层：封装请求响应模式，同步转异步，以 Request, Response 为中心，扩展接口为 Exchanger, ExchangeChannel, ExchangeClient, ExchangeServer

·     transport 网络传输层：抽象 mina 和 netty 为统一接口，以 Message 为中心，扩展接口为 Channel, Transporter, Client, Server, Codec

·     serialize 数据序列化层：可复用的一些工具，扩展接口为 Serialization, ObjectInput, ObjectOutput, ThreadPool





# 面试



Dubbo是阿里巴巴开源的基于 Java 的高性能 **RPC 分布式服务框架**，现已成为 Apache 基金会孵化项目。

能够满足高并发**小数据量**的 rpc 调用，在大数据量下的性能表现并不好，建议使用 rmi 或 http 协议

目前**不支持分布式事务**，后续可能采用基于 JTA/XA 规范实现



Dubbox 是继 Dubbo 停止维护后，当当网基于 Dubbo 做的一个扩展项目，支持了HTTP Restful 调用，更新了开源组件等



**为什么用Dubbo？**

内部使用了 Netty、Zookeeper，保证了高性能高可用性

使用 Dubbo 可以将核心业务抽取出来，作为独立的服务，提高业务复用灵活扩展，使前端应用能更快速的响应多变的市场需求

分布式架构可以承受更大规模的并发

![](image.assets/wps2-1597570300593.jpg) 

 

## Dubbo 和 Spring Cloud 区别



Dubbo和Spring Cloud没有好坏，只有适合不适合，不过我好像更倾向于使用 Dubbo, Spring Cloud 版本升级太快，组件更新替换太频繁，配置太繁琐



1）通信方式不同

Dubbo 使用的是 **RPC 通信**，而 Spring Cloud 使用的是**HTTP** RESTFul 方式。

2）组成部分不同

![](image.assets/wps3-1597570300594.jpg) 











**dubbo支持的协议****

· dubbo://（推荐）

· rmi://

· hessian://

· http://

· webservice://

· thrift://

· memcached://

· redis://

· rest://



**Dubbo内置了哪几种服务容器？**

· Spring Container

· Jetty Container

· Log4j Container

Dubbo 的服务容器只是一个简单的 Main 方法，并加载一个简单的 Spring 容器，用于暴露服务



## Dubbo注册中心



推荐使用 Zookeeper 作为注册中心，还有 Redis、Multicast、Simple 注册中心





## Dubbo 配置

![](image.assets/wps6-1597570300594.jpg) 



## 在 Provider 上配置的 Consumer 端的属性

1）timeout：方法调用超时
2）retries：失败重试次数，默认重试 2 次
3）loadbalance：负载均衡算法，默认随机
4）actives 消费者端，最大并发调用限制







**14、Dubbo推荐使用什么序列化框架，你知道的还有哪些？**

推荐使用Hessian序列化，还有FastJson、Java自带序列化



**15、Dubbo默认使用的是什么通信框架，还有别的选择吗？**

Dubbo 默认使用 Netty 框架，也是推荐的选择，另外内容还集成有Mina、Grizzly







**19、Dubbo支持服务多协议吗？**

Dubbo 允许配置多协议，在不同服务上支持不同协议或者同一服务上同时支持多种协议。



* 当一个服务接口有多种实现时怎么做
  * 用 group 属性来分组，服务提供方和消费方都指定同一个 group



## 服务上线兼容旧版本



用版本号过渡，多个不同版本的服务注册到注册中心，版本号不同的服务相互间不引用





## Dubbo可以对结果进行缓存吗？

可以，提供了声明式缓存，用于加速热门数据的访问速度，以减少用户加缓存的工作量



**23、Dubbo服务之间的调用是阻塞的吗？**

默认是同步等待结果阻塞的，支持异步调用。

==Dubbo是基于NIO的非阻塞实现并行调用==，客户端不需要启动多线程即可完成并行调用多个远程服务，相对多线程开销较小，异步调用会返回一个 Future 对象。

异步调用流程图如下。

![](image.assets/wps10-1597570300594.jpg) 

 

**29、如何解决服务调用链过长的问题？**

Dubbo 可以使用 Pinpoint 和 Apache Skywalking(Incubator) 实现分布式服务追踪，当然还有其他很多方案。



**32、Dubbo的管理控制台能做什么？**

路由规则，动态配置，服务降级，访问控制，权重调整，负载均衡





