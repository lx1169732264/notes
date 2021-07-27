





# 注解



## SpringBootApplication



```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@SpringBootConfiguration // ≈ @Configuration
@EnableAutoConfiguration //启用boot的自动装配机制
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
  
}
```



## EnableAutoConfiguration

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@AutoConfigurationPackage //将main包下所有组件注册到容器中
@Import(AutoConfigurationImportSelector.class) //加载自动装配类 xxxAutoconfiguration
public @interface EnableAutoConfiguration {

   String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

   Class<?>[] exclude() default {};

   String[] excludeName() default {};

}
```



### @AutoConfigurationPackage

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Import(AutoConfigurationPackages.Registrar.class)
public @interface AutoConfigurationPackage {

   String[] basePackages() default {};

   Class<?>[] basePackageClasses() default {};

}
```



![](image.assets/image-20210620230745237.png)



#### AutoConfigurationPackages.Registrar

<a name="扫描启动类所在包">getPackageNames()</a>

```java
static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {

  @Override
  public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
    register(registry, new PackageImports(metadata).getPackageNames().toArray(new String[0])); //getPackageNames()获取启动类所在的包,进而扫描启动类包下的所有文件
  }

  @Override
  public Set<Object> determineImports(AnnotationMetadata metadata) {
    return Collections.singleton(new PackageImports(metadata));
  }
}
```





### AutoConfigurationImportSelector

```java
public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware, ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {

}

public interface DeferredImportSelector extends ImportSelector { }

public interface ImportSelector {
  String[] selectImports(AnnotationMetadata var1); //AutoConfigurationImportSelector实现selectImports()，用于获取所有符合条件的类的全限定类名
}
```



#### selectImports

```java
public String[] selectImports(AnnotationMetadata annotationMetadata) {
  if (!isEnabled(annotationMetadata)) {
    return NO_IMPORTS;
  }
  AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
    .loadMetadata(this.beanClassLoader);
  AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(
    autoConfigurationMetadata, annotationMetadata);
  return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
}

protected boolean isEnabled(AnnotationMetadata metadata) {
  if (getClass() == AutoConfigurationImportSelector.class) {
    return getEnvironment().getProperty(EnableAutoConfiguration.ENABLED_OVERRIDE_PROPERTY, Boolean.class, true); //确认自动装配是否开启
  }
  return true;
}
```



#### loadMetadata

```java
public static AutoConfigurationMetadata loadMetadata(ClassLoader classLoader) {
  return loadMetadata(classLoader, PATH);
}

static AutoConfigurationMetadata loadMetadata(ClassLoader classLoader, String path) {
  try {
    Enumeration<URL> urls = (classLoader != null) ? classLoader.getResources(path)
      : ClassLoader.getSystemResources(path);
    Properties properties = new Properties();
    while (urls.hasMoreElements()) {
      properties.putAll(PropertiesLoaderUtils
                        .loadProperties(new UrlResource(urls.nextElement())));
    }
    return loadMetadata(properties);
  }
  catch (IOException ex) {
    throw new IllegalArgumentException(
      "Unable to load @ConditionalOnClass location [" + path + "]", ex);
  }
}

static AutoConfigurationMetadata loadMetadata(Properties properties) {
  return new PropertiesAutoConfigurationMetadata(properties);
}
```



![](image.assets/image-20210719132914580.png)

所有 Starter `META-INF/ 'PATH'`目录的启动配置类都会被读取到



### getAutoConfigurationEntry

```java
protected AutoConfigurationEntry getAutoConfigurationEntry(
  AutoConfigurationMetadata autoConfigurationMetadata,
  AnnotationMetadata annotationMetadata) {
  if (!isEnabled(annotationMetadata)) {
    return EMPTY_ENTRY;
  }
  AnnotationAttributes attributes = getAttributes(annotationMetadata);
  List<String> configurations = getCandidateConfigurations(annotationMetadata,
                                                           attributes);
  configurations = removeDuplicates(configurations);
  Set<String> exclusions = getExclusions(annotationMetadata, attributes);
  checkExcludedClasses(configurations, exclusions);
  configurations.removeAll(exclusions); //去除不需要的配置类,@ConditionalOnXXX 中的所有条件都满足，该类才会生效
  configurations = filter(configurations, autoConfigurationMetadata);
  fireAutoConfigurationImportEvents(configurations, exclusions);
  return new AutoConfigurationEntry(configurations, exclusions);
}
```











## @ConditionalOnXXX



- `@ConditionalOnBean`：当容器里有指定 Bean 的条件下
- `@ConditionalOnMissingBean`：当容器里没有指定 Bean 的情况下
- `@ConditionalOnSingleCandidate`：当指定 Bean 在容器中只有一个，或者虽然有多个但是指定首选 Bean
- `@ConditionalOnClass`：当类路径下有指定类的条件下
- `@ConditionalOnMissingClass`：当类路径下没有指定类的条件下
- `@ConditionalOnProperty`：指定的属性是否有指定的值
- `@ConditionalOnResource`：类路径是否有指定的值
- `@ConditionalOnExpression`：基于 SpEL 表达式作为判断条件
- `@ConditionalOnJava`：基于 Java 版本作为判断条件
- `@ConditionalOnJndi`：在 JNDI 存在的条件下差在指定的位置
- `@ConditionalOnNotWebApplication`：当前项目不是 Web 项目的条件下
- `@ConditionalOnWebApplication`：当前项目是 Web 项 目的条件下





## @ComponentScan

扫描[被注册的bean](#注册bean),默认会扫描启动类所在的包下所有的类，可以通过excludeFilters自定义不扫描的 bean



只有启动类所在包外的才需要用``@CompmentScan(basePackage={“”,””})``



@Component:标准一个普通的spring Bean类

@Service:标注一个业务逻辑组件类
@Repository:标注一个DAO组件类



@Controller	标注一个控制器组件类,返回页面

@RestController 数据直接以 JSON 或 XML 形式写入 HTTP Response中



Bean实例的名称默认是Bean类的首字母小写，其他部分不变





@Resource



 

**@Lazy**   **懒加载**

***注入userService时,CacheAspect中自定义的切面增强还没有被加载
 导致注入进去的是还未被动态代理的,原生的userService\***

 

@TableField(exist = false)	实体类中,数据库不存在的字段需要加上这个注解



@Transactional 	**public 的方法才起作用**

1)事务开始时，通过AOP机制，**生成代理connection对象**，并将其放入DataSource实例的某个与DataSourceTransactionManager相关的容器中。客户代码使用该connection连接数据库，执行所有数据库命令

2)事务结束时，回滚代理connection对象上执行的数据库命令，然后关闭该代理connection对象（事务结束后，回滚操作不会对已执行完毕的SQL操作命令起作用）





## @ControllerAdvice



用于定义@ExceptionHandler、@InitBinder、@ModelAttribute，并应用到所有@RequestMapping中



@ExceptionHandler  拦截异常，实现自定义异常处理

```
@ExceptionHandler(value = {UnauthorizedException.class})
public Object unauthorized() {
    Map<String, Object> map = new HashMap<>();
    map.put("code", -1);
    map.put("msg", "未授权，请联系管理员");
    return map;
}
```



@InitBinder   在其执行之前初始化数据绑定器

```
@InitBinder
    public void initBinder(WebDataBinder binder) {}
```











## 自动装配



从以下几个方面回答：

1. 什么是 SpringBoot 自动装配？
2. SpringBoot 是如何实现自动装配的？如何实现按需加载？
3. 如何实现一个 Starter？



> SpringBoot 定义了一套接口规范，这套规范规定：SpringBoot 在启动时会扫描外部引用 jar 包中的`META-INF/spring.factories`文件，将文件中配置的类型信息加载到 Spring 容器（此处涉及到 JVM 类加载机制与 Spring 的容器知识），并执行类中定义的各种操作。对于外部 jar 来说，只需要按照 SpringBoot 定义的标准，就能将自己的功能装置进 SpringBoot
>





## @PathVariable

`@PathVariable`用于路径参数，`@RequestParam`用于获取查询参数

```java
@GetMapping("/klasses/{klassId}/teachers")
public List<Teacher> getKlassRelatedTeachers(
         @PathVariable("klassId") Long klassId,
         @RequestParam(value = "type", required = false) String type ) {
...
}
```

如果我们请求的 url 是：`/klasses/{123456}/teachers?type=web`

那么我们服务获取到的数据就是：`klassId=123456,type=web



## @RequestBody

用于读取请求的body并且**Content-Type=application/json** 格式的数据，接收到数据之后会自动将数据绑定到 Java 对象上去。**使用`HttpMessageConverter`或者自定义的`HttpMessageConverter`将请求的 body 中的 json 字符串转换为 java 对象**



**一个请求方法只可以有一个`@RequestBody`，但是可以有多个`@RequestParam`和`@PathVariable`**。 如果你的方法必须要用两个 `@RequestBody`来接受数据的话，大概率是你的数据库设计或者系统设计出问题了！





## 读取配置文件

```yaml
library:
  location: 湖北武汉加油中国加油
  books:
    - name: 天才基本法
      description: 545
    - name: 时间的秩序
```



### @value

`@Value("${property}")`

```java
@Value("${wuhan2020}")
String wuhan2020;
```



### @ConfigurationProperties

读取配置信息并与bean绑定

```java
@Component
@ConfigurationProperties(prefix = "library")
class LibraryProperties {
  private String location;
  private List<Book> books;

  @Data
  static class Book {
    String name;
    String description;
  }
}
```



## Json处理



### 过滤Json数据



**`@JsonIgnoreProperties` 作用在类上,过滤掉特定字段不返回或者不解析。**

```java
//生成json时将userRoles属性过滤
@JsonIgnoreProperties({"userRoles"})
public class User {
    private List<UserRole> userRoles = new ArrayList<>();
}
```



`@JsonIgnore`作用在属性上

```java
public class User {
   //生成json时将userRoles属性过滤
    @JsonIgnore
    private List<UserRole> userRoles = new ArrayList<>();
}
```



### 格式化 json 数据

`@JsonFormat`



### 扁平化对象

`@JsonUnwrapped`

```java
public class Account {
  @JsonUnwrapped
    private Location location;
    @JsonUnwrapped
    private PersonInfo personInfo;
}
```



```json
//未扁平化前
{
  "location": {
    "provinceName":"湖北",
    "countyName":"武汉"
  },
  "personInfo": {
    "userName": "coder1234",
    "fullName": "shaungkou"
  }
}

//扁平化后
{
  "provinceName":"湖北",
  "countyName":"武汉",
  "userName": "coder1234",
  "fullName": "shaungkou"
}
```















## 异常处理



1. `@ControllerAdvice` :注解定义全局异常处理类
2. `@ExceptionHandler` :注解声明异常处理方法



## JPA

#### 8.1. 创建表

`@Entity`声明一个类对应一个数据库实体。

`@Table` 设置表名

```java
@Entity
@Table(name = "role")
public class Role {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String description;
    省略getter/setter......
}
```

#### 8.2. 创建主键

`@Id` ：声明一个字段为主键。

使用`@Id`声明之后，我们还需要定义主键的生成策略。我们可以使用 `@GeneratedValue` 指定主键生成策略。

**1.通过 `@GeneratedValue`直接使用 JPA 内置提供的四种主键生成策略来指定主键生成策略。**

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

JPA 使用枚举定义了 4 中常见的主键生成策略，如下：

_Guide 哥：枚举替代常量的一种用法_

```java
public enum GenerationType {

    /**
     * 使用一个特定的数据库表格来保存主键
     * 持久化引擎通过关系数据库的一张特定的表格来生成主键,
     */
    TABLE,

    /**
     *在某些数据库中,不支持主键自增长,比如Oracle、PostgreSQL其提供了一种叫做"序列(sequence)"的机制生成主键
     */
    SEQUENCE,

    /**
     * 主键自增长
     */
    IDENTITY,

    /**
     *把主键生成策略交给持久化引擎(persistence engine),
     *持久化引擎会根据数据库在以上三种主键生成 策略中选择其中一种
     */
    AUTO
}

```

`@GeneratedValue`注解默认使用的策略是`GenerationType.AUTO`

```java
public @interface GeneratedValue {

    GenerationType strategy() default AUTO;
    String generator() default "";
}
```

一般使用 MySQL 数据库的话，使用`GenerationType.IDENTITY`策略比较普遍一点（分布式系统的话需要另外考虑使用分布式 ID）。

**2.通过 `@GenericGenerator`声明一个主键策略，然后 `@GeneratedValue`使用这个策略**

```java
@Id
@GeneratedValue(generator = "IdentityIdGenerator")
@GenericGenerator(name = "IdentityIdGenerator", strategy = "identity")
private Long id;
```

等价于：

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

jpa 提供的主键生成策略有如下几种：

```java
public class DefaultIdentifierGeneratorFactory
		implements MutableIdentifierGeneratorFactory, Serializable, ServiceRegistryAwareService {

	@SuppressWarnings("deprecation")
	public DefaultIdentifierGeneratorFactory() {
		register( "uuid2", UUIDGenerator.class );
		register( "guid", GUIDGenerator.class );			// can be done with UUIDGenerator + strategy
		register( "uuid", UUIDHexGenerator.class );			// "deprecated" for new use
		register( "uuid.hex", UUIDHexGenerator.class ); 	// uuid.hex is deprecated
		register( "assigned", Assigned.class );
		register( "identity", IdentityGenerator.class );
		register( "select", SelectGenerator.class );
		register( "sequence", SequenceStyleGenerator.class );
		register( "seqhilo", SequenceHiLoGenerator.class );
		register( "increment", IncrementGenerator.class );
		register( "foreign", ForeignGenerator.class );
		register( "sequence-identity", SequenceIdentityGenerator.class );
		register( "enhanced-sequence", SequenceStyleGenerator.class );
		register( "enhanced-table", TableGenerator.class );
	}

	public void register(String strategy, Class generatorClass) {
		LOG.debugf( "Registering IdentifierGenerator strategy [%s] -> [%s]", strategy, generatorClass.getName() );
		final Class previous = generatorStrategyToClassNameMap.put( strategy, generatorClass );
		if ( previous != null ) {
			LOG.debugf( "    - overriding [%s]", previous.getName() );
		}
	}

}
```

#### 8.3. 设置字段类型

`@Column` 声明字段。

**示例：**

设置属性 userName 对应的数据库字段名为 user_name，长度为 32，非空

```java
@Column(name = "user_name", nullable = false, length=32)
private String userName;
```

设置字段类型并且加默认值，这个还是挺常用的。

```java
Column(columnDefinition = "tinyint(1) default 1")
private Boolean enabled;
```

#### 8.4. 指定不持久化特定字段

`@Transient` ：声明不需要与数据库映射的字段，在保存的时候不需要保存进数据库 。

如果我们想让`secrect` 这个字段不被持久化，可以使用 `@Transient`关键字声明。

```java
Entity(name="USER")
public class User {

    ......
    @Transient
    private String secrect; // not persistent because of @Transient

}
```

除了 `@Transient`关键字声明， 还可以采用下面几种方法：

```java
static String secrect; // not persistent because of static
final String secrect = “Satish”; // not persistent because of final
transient String secrect; // not persistent because of transient
```

一般使用注解的方式比较多。

#### 8.5. 声明大字段

`@Lob`:声明某个字段为大字段。

```java
@Lob
private String content;
```

更详细的声明：

```java
@Lob
//指定 Lob 类型数据的获取策略， FetchType.EAGER 表示非延迟 加载，而 FetchType. LAZY 表示延迟加载 ；
@Basic(fetch = FetchType.EAGER)
//columnDefinition 属性指定数据表对应的 Lob 字段类型
@Column(name = "content", columnDefinition = "LONGTEXT NOT NULL")
private String content;
```

#### 8.6. 创建枚举类型的字段

可以使用枚举类型的字段，不过枚举字段要用`@Enumerated`注解修饰。

```java
public enum Gender {
    MALE("男性"),
    FEMALE("女性");

    private String value;
    Gender(String str){
        value=str;
    }
}
```

```java
@Entity
@Table(name = "role")
public class Role {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String description;
    @Enumerated(EnumType.STRING)
    private Gender gender;
    省略getter/setter......
}
```

数据库里面对应存储的是 MAIL/FEMAIL。

#### 8.7. 增加审计功能

只要继承了 `AbstractAuditBase`的类都会默认加上下面四个字段。

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@MappedSuperclass
@EntityListeners(value = AuditingEntityListener.class)
public abstract class AbstractAuditBase {

    @CreatedDate
    @Column(updatable = false)
    @JsonIgnore
    private Instant createdAt;

    @LastModifiedDate
    @JsonIgnore
    private Instant updatedAt;

    @CreatedBy
    @Column(updatable = false)
    @JsonIgnore
    private String createdBy;

    @LastModifiedBy
    @JsonIgnore
    private String updatedBy;
}

```

我们对应的审计功能对应地配置类可能是下面这样的（Spring Security 项目）:

```java
@Configuration
@EnableJpaAuditing
public class AuditSecurityConfiguration {
    @Bean
    AuditorAware<String> auditorAware() {
        return () -> Optional.ofNullable(SecurityContextHolder.getContext())
                .map(SecurityContext::getAuthentication)
                .filter(Authentication::isAuthenticated)
                .map(Authentication::getName);
    }
}
```

简单介绍一下上面设计到的一些注解：

1. `@CreatedDate`: 表示该字段为创建时间时间字段，在这个实体被 insert 的时候，会设置值

2. `@CreatedBy` :表示该字段为创建人，在这个实体被 insert 的时候，会设置值

   `@LastModifiedDate`、`@LastModifiedBy`同理。

`@EnableJpaAuditing`：开启 JPA 审计功能。

#### 8.8. 删除/修改数据

`@Modifying` 注解提示 JPA 该操作是修改操作,注意还要配合`@Transactional`注解使用。

```java
@Repository
public interface UserRepository extends JpaRepository<User, Integer> {

    @Modifying
    @Transactional(rollbackFor = Exception.class)
    void deleteByUserName(String userName);
}
```

#### 8.9. 关联关系

- `@OneToOne` 声明一对一关系
- `@OneToMany` 声明一对多关系
- `@ManyToOne`声明多对一关系
- `MangToMang`声明多对多关系



## 参数校验



**JSR(Java Specification Requests）** 是一套 JavaBean 参数校验的标准，它定义了很多常用的校验注解，我们可以直接将这些注解加在我们 JavaBean 的属性上面，这样就可以在需要校验的时候进行校验了，非常方便！

校验的时候我们实际用的是 **Hibernate Validator** 框架。Hibernate Validator 是 Hibernate 团队最初的数据校验框架，Hibernate Validator 4.x 是 Bean Validation 1.0（JSR 303）的参考实现，Hibernate Validator 5.x 是 Bean Validation 1.1（JSR 349）的参考实现，目前最新版的 Hibernate Validator 6.x 是 Bean Validation 2.0（JSR 380）的参考实现。

SpringBoot 项目的 spring-boot-starter-web 依赖中已经有 hibernate-validator 包，不需要引用相关依赖。如下图所示（通过 idea 插件—Maven Helper 生成）：

![](image.assets/c7bacd12-1c1a-4e41-aaaf-4cad840fc073.png)



👉 **所有的注解，推荐使用 JSR 注解，即`javax.validation.constraints`，而不是`org.hibernate.validator.constraints`**

### 常用的验证注解

- `@NotEmpty` 被注释的字符串的不能为 null 也不能为空
- `@NotBlank` 被注释的字符串非 null，并且必须包含一个非空白字符
- `@Null` 被注释的元素必须为 null
- `@NotNull` 被注释的元素必须不为 null
- `@AssertTrue` 被注释的元素必须为 true
- `@AssertFalse` 被注释的元素必须为 false
- `@Pattern(regex=,flag=)`被注释的元素必须符合指定的正则表达式
- `@Email` 被注释的元素必须是 Email 格式。
- `@Min(value)`被注释的元素必须是一个数字，其值必须大于等于指定的最小值
- `@Max(value)`被注释的元素必须是一个数字，其值必须小于等于指定的最大值
- `@DecimalMin(value)`被注释的元素必须是一个数字，其值必须大于等于指定的最小值
- `@DecimalMax(value)` 被注释的元素必须是一个数字，其值必须小于等于指定的最大值
- `@Size(max=, min=)`被注释的元素的大小必须在指定的范围内
- `@Digits (integer, fraction)`被注释的元素必须是一个数字，其值必须在可接受的范围内
- `@Past`被注释的元素必须是一个过去的日期
- `@Future` 被注释的元素必须是一个将来的日期

### 验证Body

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Person {
    @NotNull(message = "classId 不能为空")
    private String classId;

    @Size(max = 33)
    @NotNull(message = "name 不能为空")
    private String name;

    @Pattern(regexp = "((^Man$|^Woman$|^UGM$))", message = "sex 值不在可选范围")
    @NotNull(message = "sex 不能为空")
    private String sex;

    @Email(message = "email 格式不正确")
    @NotNull(message = "email 不能为空")
    private String email;
}
```

我们在需要验证的参数上加上了`@Valid`注解，如果验证失败，它将抛出`MethodArgumentNotValidException`。

```java
@RestController
@RequestMapping("/api")
public class PersonController {

    @PostMapping("/person")
    public ResponseEntity<Person> getPerson(@RequestBody @Valid Person person) {
        return ResponseEntity.ok().body(person);
    }
}
```

### 验证请求参数(Path Variables 和 Request Parameters)

**一定一定不要忘记在类上加上 `Validated` 注解了，这个参数可以告诉 Spring 去校验方法参数**

```java
@RestController
@RequestMapping("/api")
@Validated
public class PersonController {

  @GetMapping("/person/{id}")
  public ResponseEntity<Integer> getPersonByID(@Valid @PathVariable("id") @Max(value = 5,message = "超过 id 的范围了") Integer id) {
    return ResponseEntity.ok().body(id);
  }
}
```

















# 配置文件优先级



@ConfigurationProperties

```java
//application.properties中配置  test.topAppKey=123456
@Configuration
@ConfigurationProperties("test")
public class TestConfig {

  private String topAppKey = "001234";

  public String getTopAppKey() {
    return topAppKey;
  }

  public void setTopAppKey(String topAppKey) {
    this.topAppKey = topAppKey;
  }
}
```





## IOC

Ioc—Inversion of Control 控制反转



主动创建依赖对象会导致类与类之间高耦合

把**创建和查找依赖对象的控制权交给容器**，依赖对象的获取被反转,由容器注入对象，低耦合



IoC 容器实际上就是存放各种对象的Map



### DI

DI—Dependency Injection 依赖注入

组件间依赖关系由容器在运行期决定，即由容器动态的将依赖关系注入到组件之中

应用程序的对象 需要IoC容器提供对象需要的 外部资源(对象、资源、常量数据)



IoC和DI是同一个概念的不同角度描述



## AOP

Aspect-Oriented Programming 面向切面编程

将那些与业务无关，**却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来**，便于**减少系统的重复代码**，**降低模块间的耦合度**，并**有利于未来的可拓展性和可维护性**。

**基于动态代理**，如果要代理的对象实现了某个接口，那么Spring AOP会使用**JDK Proxy**，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候Spring AOP会使用**Cglib** ，这时候Spring AOP会使用 **Cglib** 生成一个被代理对象的子类来作为代理





### Spring AOP VS AspectJ AOP

**Spring AOP 属于运行时增强，而 AspectJ 是编译时增强。** Spring AOP 基于代理(Proxying)，而 AspectJ 基于字节码操作(Bytecode Manipulation)。

Spring AOP 已经集成了 AspectJ  ，AspectJ  应该算的上是 Java 生态系统中最完整的 AOP 框架了。AspectJ  相比于 Spring AOP 功能更加强大，但是 Spring AOP 相对来说更简单，





# 前后端传值











# Bean



## @Scope



| singleton | prototype        | request                                           | session                                             | ~~global-session~~                                           |
| --------- | ---------------- | ------------------------------------------------- | --------------------------------------------------- | ------------------------------------------------------------ |
| 单例      | 每次请求创建新的 | 每次HTTP请求创建新的，该bean仅在当前request内有效 | 每次HTTP请求创建新的，该bean仅在当前 session 内有效 | 全局session作用域，仅仅在基于portlet的web应用中才有意义，Spring5已经没有了 |



**单例bean的线程安全问题**

一般情况下， `Controller`、`Service`、`Dao` 这些 Bean 是无状态的。无状态的 Bean 不能保存数据，因此线程安全



但若出现bean需要保存数据的场景

1. 将需要的可变成员变量保存在 `ThreadLocal`
2. 改变 Bean 的作用域为 “prototype”：每次请求都会创建一个新的 bean 实例，线程安全



## 注册bean



- `@Component` ：通用注解，标注任意类为 `Spring` 组件。如果不知道这个Bean属于哪个层，可以使用`@Component`
- @Configuration 声明配置类
- `@Repository`
- `@Service`
- `@Controller` : 对应 Spring MVC 控制层

`@RestController`=`@Controller和`+@`ResponseBody`,表示这是个控制器 bean,并且是将函数的返回值直 接填入 HTTP 响应体中,是 REST 风格的控制器



**@Component VS @Bean**

| @Component                                           | @Bean                                                        |
| ---------------------------------------------------- | ------------------------------------------------------------ |
| 作用于类                                             | 作用于方法                                                   |
| 配合`@ComponentScan`类路径扫描来自动装配到Spring容器 | 在方法中定义产生bean,`@Bean`告诉了Spring这是某个类的示例     |
|                                                      | 自定义性更强,很多地方只能通过 `@Bean` 来注册bean。比如引用第三方库中的类需要装配到`Spring`容器 |











## bean生命周期



- Bean 容器找到配置文件中 Spring Bean 的定义
- Bean 容器利用 反射 创建Bean实例
- 利用 `set()`设置一些属性值
- 如果 Bean 实现了 `BeanNameAware` 接口，调用 `setBeanName()`，定义Bean的名字
- 如果 Bean 实现了 `BeanClassLoaderAware` 接口，调用 `setBeanClassLoader()`，传入 `ClassLoader`对象的实例
- 与上面的类似，如果实现了其他 `*.Aware`接口，就调用相应的方法
- 如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessBeforeInitialization()` 方法
- 如果Bean实现了`InitializingBean`接口，执行`afterPropertiesSet()`方法。
- 如果 Bean 在配置文件中的定义包含 init-method 属性，执行指定的方法。
- 如果有和加载这个 Bean的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessAfterInitialization()` 方法
- 当要销毁 Bean 的时，如果 Bean 实现了 `DisposableBean` 接口，执行 `destroy()`
- 当要销毁 Bean 的时，如果 Bean 在配置文件中的定义包含 destroy-method 属性，执行指定的方法



## 生命周期行为

@PostConstruct	bean的初始化之前的方法

@PreDestory		bean销毁之前的方法







# MVC



![](image.assets/MVC.png)



1. 客户端（浏览器）发送请求，直接请求到 `DispatcherServlet`。
2. `DispatcherServlet` 根据请求信息调用 `HandlerMapping`，解析请求对应的 `Handler`。
3. 解析到对应的 `Handler`（也就是我们平常说的 `Controller` 控制器）后，开始由 `HandlerAdapter` 适配器处理。
4. `HandlerAdapter` 会根据 `Handler `来调用真正的处理器来处理请求，并处理相应的业务逻辑。
5. 处理器处理完业务后，会返回一个 `ModelAndView` 对象，`Model` 是返回的数据对象，`View` 是个逻辑上的 `View`。
6. `ViewResolver` 会根据逻辑 `View` 查找实际的 `View`。
7. `DispaterServlet` 把返回的 `Model` 传给 `View`（视图渲染）。
8. 把 `View` 返回给请求者（浏览器）



# Spring的设计模式



- **工厂设计模式** : Spring使用工厂模式通过 `BeanFactory`、`ApplicationContext` 创建 bean 对象。
- **代理设计模式** : Spring AOP 功能的实现。
- **单例设计模式** : Spring 中的 Bean 默认都是单例的。
- **模板方法模式** : Spring 中 `jdbcTemplate`、`hibernateTemplate` 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。
- **包装器设计模式** : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
- **观察者模式:** Spring 事件驱动模型就是观察者模式很经典的一个应用。
- **适配器模式** :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配`Controller`。









# Spring事务



@Transactional使用JDBC事务来进行事务控制,只能配置在**public**方法/类上,并且**类内部的调用不支持事务**,作用在类时,非public方法不生效

事务开始时，通过**AOP**机制，生成代理connection对象，并将其放入 DataSource 实例的某个与 DataSourceTransactionManager 相关的容器中



@Transactional 的事务由 基于接口/类的代理被创建而开启。所以在同一个类中一个方法调用另一个方法有事务的方法，事务是不会起作用的

spring 在扫描bean方法上是否包含@Transactional，如果包含则为这个bean动态地生成一个代理类，继承那个bean。此时@Transactional方法实际上由代理类在调用前开启事务

但**类内部的调用没有通过代理类**，而是直接通过原来的那个bean，所以就不会启动transaction





==在具体的类/类的方法上使用 @Transactional 注解，而不要使用在类所要实现的任何接口上==

在接口上使用 @Transactional 注解，只能设置了基于接口的代理时它才生效。因为注解是不能继承的，这就意味着如果正在使用基于类的代理时，那么事务的设置将不能被基于类的代理所识别，而且对象也将不会被事务代理所包装





**如果目标对象实现了接口，默认情况下会采用 JDK 的动态代理，如果目标对象没有实现了接口,会使用 CGLIB 动态代理。**

多提一嘴：`createAopProxy()` 方法 决定了是使用 JDK 还是 Cglib 来做动态代理，源码如下：

```java
public class DefaultAopProxyFactory implements AopProxyFactory, Serializable {

	@Override
	public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
		if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
			Class<?> targetClass = config.getTargetClass();
			if (targetClass == null) {
				throw new AopConfigException("TargetSource cannot determine target class: " +
						"Either an interface or a target is required for proxy creation.");
			}
			if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
				return new JdkDynamicAopProxy(config);
			}
			return new ObjenesisCglibAopProxy(config);
		}
		else {
			return new JdkDynamicAopProxy(config);
		}
	}
  .......
}
```

如果一个类或者一个类中的 public 方法上被标注`@Transactional` 注解的话，Spring 容器就会在启动的时候为其创建一个代理类，在调用被`@Transactional` 注解的 public 方法的时候，实际调用的是，`TransactionInterceptor` 类中的 `invoke()`方法。这个方法的作用就是在目标方法之前开启事务，方法执行过程中如果遇到异常的时候回滚事务，方法调用完成之后提交事务



## 失效的6种场景



### 非public的方法

`TransactionInterceptor` （事务拦截器）在目标方法执行前后进行拦截，`DynamicAdvisedInterceptor`（CglibAopProxy 的内部类）的 intercept 方法或 `JdkDynamicAopProxy` 的 invoke 方法会间接调用 `AbstractFallbackTransactionAttributeSource`的 `computeTransactionAttribute` 方法，获取Transactional 注解的事务配置信息

```java
protected TransactionAttribute computeTransactionAttribute(Method method,
                                                           Class<?> targetClass) {
  // Don't allow no-public methods as required.
  if (allowPublicMethodsOnly() && !Modifier.isPublic(method.getModifiers())) { //检查目标方法的修饰符是否为 public，不是 public则不会获取@Transactional 的属性配置信息
    return null;
  }
```



### 传播机制设置错误



PROPAGATION_SUPPORTS

PROPAGATION_NOT_SUPPORTED

PROPAGATION_NEVER

这三种不会回滚



### rollbackFor设置错误









## 事务管理接口



**编程式事务管理**

编程式事务管理是侵入性事务管理，使用TransactionTemplate



**声明式事务管理**

声明式事务管理建立在AOP上，在目标方法开始之前创建/加入事务，执行完目标方法后提交/回滚

编程式事务每次都要单独实现，但业务量大功能复杂时，使用编程式事务无疑是痛苦的，而声明式事务不同，声明式事务属于无侵入式，不会影响业务逻辑的实现，只需要在配置文件中做相关的事务规则声明或者通过注解的方式，便可以将事务规则应用到业务逻辑中。
 显然声明式优于编程式事务管理，这正是Spring倡导的非侵入式的编程方式。唯一不足的地方就是声明式事务管理的粒度是方法级别，而编程式事务管理是可以到代码块的，但是可以通过提取方法的方式完成声明式事务管理的配置





### PlatformTransactionManager

事务管理器，Spring 事务策略的核心

Spring 并不直接管理事务，而是提供了多种事务管理器,所有管理器实现了PlatformTransactionManager接口

管理器根据**`TransactionDefinition`**定义事务的超时时间/隔离级别/传播机制等



```java
public interface PlatformTransactionManager {
  //获得事务
  TransactionStatus getTransaction(@Nullable TransactionDefinition var1) throws TransactionException;
  //提交事务
  void commit(TransactionStatus var1) throws TransactionException;
  //回滚事务
  void rollback(TransactionStatus var1) throws TransactionException;
}
```



### TransactionDefinition

定义5个事务属性



| 隔离级别  | 传播机制    | 回滚规则    | 是否只读 | 超时    |
| --------- | ----------- | ----------- | -------- | ------- |
| isolation | propagation | rollbackFor | readOnly | timeout |



```java
public interface TransactionDefinition {
  int PROPAGATION_REQUIRED = 0;
  int PROPAGATION_SUPPORTS = 1;
  int PROPAGATION_MANDATORY = 2;
  int PROPAGATION_REQUIRES_NEW = 3;
  int PROPAGATION_NOT_SUPPORTED = 4;
  int PROPAGATION_NEVER = 5;
  int PROPAGATION_NESTED = 6;
  int ISOLATION_DEFAULT = -1;
  int ISOLATION_READ_UNCOMMITTED = 1;
  int ISOLATION_READ_COMMITTED = 2;
  int ISOLATION_REPEATABLE_READ = 4;
  int ISOLATION_SERIALIZABLE = 8;
  int TIMEOUT_DEFAULT = -1;
  // 返回事务的传播行为，默认值为 REQUIRED。
  int getPropagationBehavior();
  //返回事务的隔离级别，默认值是 DEFAULT
  int getIsolationLevel();
  // 返回事务的超时时间，默认值为-1。如果超过该时间限制但事务还没有完成，则自动回滚事务。
  int getTimeout();
  // 返回是否为只读事务，默认值为 false
  boolean isReadOnly();

  @Nullable
  String getName();
}
```





### TransactionStatus

标记事务的状态

```java
public interface TransactionStatus{
  boolean isNewTransaction(); // 是否是新的事务
  boolean hasSavepoint(); // 是否有恢复点
  void setRollbackOnly();  // 设置为只回滚
  boolean isRollbackOnly(); // 是否为只回滚
  boolean isCompleted; // 是否已完成
}
```



[PlatformTransactionManager.getTransaction](#PlatformTransactionManager),返回一个TransactionStatus对象













## rollbackFor

事务管理器捕捉unchecked异常（runtime exception）时回滚

如果在事务中抛出其他类型的异常，但却期望 Spring 能够回滚事务，就需要指定 **rollbackFor**属性



### 同一个类中的方法调用



### catch了异常

> A调用B
>
> B方法内部抛了异常，Acatch了B方法的异常，此时B回滚,A不回滚
>
> UnexpectedRollbackException: Transaction rolled back because it has been marked as rollback-only



当`Service B`抛出异常后，`Service B`标识当前事务需要rollback。但`Service A`中catch异常并进行处理，`Service A`正常`commit`。就出现了前后不一致，抛出`UnexpectedRollbackException`

`spring`事务在调用方法之前开始，方法执行完毕之后才执行`commit` / `rollback`，**事务是否提交取决于是否抛出`runtime异常`**。如果抛出`runtime exception` 并在方法中没有catch，则回滚

在业务方法中一般不需要catch异常，如果非要catch一定要抛出`throw new RuntimeException()`，或者注解中指定抛异常类型`@Transactional(rollbackFor=Exception.class)`，否则会导致事务失效，数据commit造成数据不一致，有些时候catch反倒会画蛇添足



### 数据库引擎不支持事务





### 传播机制

`TransactionDefinition.XXX`



| 支持当前事务   | PROPAGATION_REQUIRED 默认 | 加入外层/新建事务执行                                   | 一起回滚                              |
| -------------- | ------------------------- | ------------------------------------------------------- | ------------------------------------- |
|                | PROPAGATION_SUPPORT       | 加入外层,外层无事务则内层以非事务执行                   |                                       |
|                | PROPAGATION_MANDATORY     | 加入外层,外层非事务则抛异常                             |                                       |
| 不支持当前事务 | PROPAGATION_REQUES_NEW    | 新建事务,外层挂起                                       | 独立回滚                              |
|                | PROPAGATION_NOT_SUPPORT   | 非事务执行内层,外层挂起                                 |                                       |
|                | PROPAGATION_NEVER         | 非事务执行内层,外层有事务就抛异常                       |                                       |
| 混沌中立       | PROPAGATION_NESTED        | 外层有事务则加入外层,无事务则与PROPAGATION_REQUIRED一致 | 外层回滚则一起回滚,子事务可以单独回滚 |





## 隔离级别

通过``isolation =XXX``配置

| READ_UNCOMMITTED | 读未提交 |
| ---------------- | -------- |
| READ_COMMITTED   | 读提交   |
| REPEATABLE_READ  | 可重复读 |
| SERIALIZABLE     | 串行     |



```java
public interface TransactionDefinition {
    int ISOLATION_DEFAULT = -1; //使用后端数据库默认的隔离级别
    int ISOLATION_READ_UNCOMMITTED = 1;
    int ISOLATION_READ_COMMITTED = 2;
    int ISOLATION_REPEATABLE_READ = 4;
    int ISOLATION_SERIALIZABLE = 8;
}
```



为了方便使用，Spring 也相应地定义了一个枚举类：`Isolation`



```java
public enum Isolation {

	DEFAULT(TransactionDefinition.ISOLATION_DEFAULT),
	READ_UNCOMMITTED(TransactionDefinition.ISOLATION_READ_UNCOMMITTED),
	READ_COMMITTED(TransactionDefinition.ISOLATION_READ_COMMITTED),
	REPEATABLE_READ(TransactionDefinition.ISOLATION_REPEATABLE_READ),
	SERIALIZABLE(TransactionDefinition.ISOLATION_SERIALIZABLE);

	private final int value;

	Isolation(int value) {
		this.value = value;
	}

	public int value() {
		return this.value;
	}

}
```

下面我依次对每一种事务隔离级别进行介绍：





## 事务超时

事务可能涉及对数据库的锁定，长时间运行事务会不必要地占用数据库资源。这时就可以声明一个事务在特定秒数后自动回滚，不必等它自己结束。

由于超时时钟在一个事务启动的时候开始的，因此只对会启动新事务的传播行为来说，声明事务超时才有意义











## 注意事项

1. private/protected methods 不要标记 @Transactional
2. ==类内部调用 @Transactional 不起作用==
3. @Transactional 的方法内，不要去抓取数据库相关异常
4. 标记 @Transactional 的类/方法也能final











