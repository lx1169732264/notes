# 常用注解

pojo是一个统称，可以是DTO、可以是VO、可以是PO、可以是domain，这些都叫pojo
 po、do、domain三者区别很小，用处都是和数据库进行对应

DTO是数据传输对象，简单点说就是传参数用的。比如一张表30个字段，但是传参的时候只需要传5个字段，这个时候使用dto，可以避免过多的无用数据，也可以隐藏后端表结构，往往是前端传参给后端、controller、service、dao三层之间传递使用。
 VO就是view Object，专门负责给前端展示数据。VO是业务对象，业务上需要什么字段它就给什么字段，比如上面说的学生表，在给前端展示的时候需要展示名称：class_name这个字段，那么vo里写的就不是classId，而是className或者班级类

 

do、vo、dto这些类之间不需要给继承关系，从设计思想来讲也不能给继承关系。互相之间的属性复制使用spring提供的BeanUtils.copyProperties。如果是集合这种数据较多的属性复制，就先转成json字符串再转成另一个类的List

 

@Deprecated，用来表示某个类或属性或方法已经过时

@SuppressWarnings用来压制程序中出来的警告，比如在没有用泛型或是方法已经过时的时候





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



@ModelAttribute   把值绑定到所有的Model中

```
    @ModelAttribute
    public void addAttributes(Model model) {
        model.addAttribute("author", "lx");
    }
```





## JSON

@JsonInclude(Include.NON_NULL) 是springmvc中的标注，剔除返回json中的null

@JsonIgnore	忽略不想传给前台的的属性





//将该标记放在属性上，如果该属性为NULL则不参与序列化 
//如果放在类上边,那对这个类的全部属性起作用 
//Include.Include.ALWAYS 默认 
//Include.NON_DEFAULT 属性为默认值不序列化 
//Include.NON_EMPTY 属性为 空（“”） 或者为 NULL 都不序列化 
//Include.NON_NULL 属性为NULL 不序列化 











## Junit注解

@BeforeClass – 表示在类中的任意public static void方法执行之前执行

@AfterClass – 表示在类中的任意public static void方法执行之后执行

@Before – 表示在任意使用@Test注解标注的public void方法执行之前执行

@After – 表示在任意使用@Test注解标注的public void方法执行之后执行

@AfterRunning: 返回通知, 在方法返回结果之后执行

@AfterThrowing: 异常通知, 在方法抛出异常之后

@Around: 环绕通知, 围绕着方法执行

 

 

# 日期格式处理

@JsonFormat           后台到前台

properties文件中有相同的配置spring.mvc.date-format

@DateTimeFormat     前台到后台

properties文件中有相同的配置spring.jackson.date-format

spring.jackson.time-zone

 

@Respostory

@Compment      把切面类加入到IOC容器中

@**EnableAspectJAutoProxy**   //开启对AspectJ语法风格的支持

@ControllerAdvise  当Controller出现异常时,跳转页面

@RestControllerAdvise         ,返回json

 

@bean作用在方法上

@import引入其他的配置文件 

@ComponentScan(“”) 配置扫描

 

当ioc容器里有多个同名对象时

@Qualifier 合格者，表明哪个bean是需要的

​    Qualifier的参数必须是之前用@Bean注解过的

@Primary  指明优先级

 

@Configuration(proxyBeanMethods = false) proxyBeanMethods决定了配置类是否会被代理,如果@Bean方法间没有调用关系的话可以把 proxyBeanMethods 设置为 false。否则，方法内部引用的类生产的类和 Spring 容器中类是两个类。

 

 

@ConditionalOnBean // 当给定的在bean存在时,则实例化当前Bean @ConditionalOnMissingBean // 当给定的在bean不存在时,则实例化当前Bean @ConditionalOnClass // 当给定的类名在类路径上存在，则实例化当前Bean @ConditionalOnMissingClass // 当给定的类名在类路径上不存在，则实例化当前Bean

 

 

@ConfigurationProperties(prefix = "spring.redis") 配置类注解,prefix是配置时的前缀

@EnableConfigurationProperties(RedisProperties.class)     加载配置类

 

 

**@Lazy**   **懒加载**

***注入userService时,CacheAspect中自定义的切面增强还没有被加载
 导致注入进去的是还未被动态代理的,原生的userService\***

 

***@TableField(exist = false)\***  ***实体类中,数据库不存在的字段需要加上这个注解\***

 

@Transactional 	**public 的方法才起作用**

1)事务开始时，通过AOP机制，**生成代理connection对象**，并将其放入DataSource实例的某个与DataSourceTransactionManager相关的容器中。客户代码使用该connection连接数据库，执行所有数据库命令

2)事务结束时，回滚代理connection对象上执行的数据库命令，然后关闭该代理connection对象（事务结束后，回滚操作不会对已执行完毕的SQL操作命令起作用）

## 五大元注解

### @Target：限制注解能修饰的对象范围

/**用于描述类、接口(包括注解类型) 或enum声明*/

  TYPE,

  /** 用于描述域 Field declaration (includes enum constants) */

  FIELD,

  /**用于描述方法 Method declaration */

  METHOD,

  /**用于描述参数 Formal parameter declaration */

  PARAMETER,

  /**用于描述构造器 Constructor declaration */

  CONSTRUCTOR,

  /**用于描述局部变量 Local variable declaration */

  LOCAL_VARIABLE,

  /** 注解类 */

  ANNOTATION_TYPE,

  /**用于描述包 Package declaration */

  PACKAGE,

  /**用来标注类型参数 Type parameter declaration   */

  TYPE_PARAMETER,

  /**所有类型 */

  TYPE_USE

### @Retention 

Retention注解有一个RetentionPolicy类型的枚举属性

有3个值：CLASS RUNTIME  SOURCE标明注解应该如何去保持,这也是它的生命周期
 RetentionPolicy.SOURCE：注解只保留在源文件，当Java文件编译成class文件的时候，注解被遗弃；
 RetentionPolicy.CLASS：注解被保留到class文件，但jvm加载class文件时候被遗弃，默认RetentionPolicy.RUNTIME：注解不仅被保存到class文件中，jvm加载class文件之后，仍然存在；
 这3个生命周期分别对应于：Java源文件(.java文件) ---> .class文件 ---> 内存中的字节码。


生命周期长度 SOURCE < CLASS < RUNTIME ，所以前者能作用的地方后者一定也能作用。
 需要在运行时动态获取注解信息，那只能用RUNTIME注解，比如@Deprecated使用RUNTIME注解
 在编译时进行预处理操作，比如生成一些辅助代码（如 ButterKnife），就用 CLASS注解；
 只是检查性的操作，比如 @Override 和 @SuppressWarnings，使用SOURCE 注解。

### @Documented将注解包含在javadoc中

默认情况下，javadoc是不包括注解的，但如果使用了@Documented注解，则相关注解类型信息会被包含在生成的文档中

### @Inherited指明父类注解会被子类继承得到

### @Repeatable

指明注解为可重复注解，可以在同一个地方多次使用

## @Scheduled定时

八大参数

### Cron 定时时间

@Scheduled(cron = "0 0 5 * * ?")      [秒] [分] [小时] [日] [月] [周] [年]

允许正则表达式,

?    不指定值

\-    区间

,    指定多个值

/    递增触发。秒”5/15” 表示从5秒开始，每增15秒触发

L    最后。对于日字段，表示当月的最后一天.对于周字段上设置”6L”这样的格式,则表示“本月最后一个星期五”

W   离指定日期的最近的工作日(周一至周五). 例如在日字段上置”15W”，表示离每月15号最近的那个工作日触发。如果15号正好是周六，则找最近的周五(14号)触发, 如果15号是周未，则找最近的下周一(16号)触发.如果15号正好在工作日(周一至周五)，则就在该天触发。如果指定格式为 “1W”,它则表示每月1号往后最近的工作日触发。如果1号正是周六，则将在3号下周一触发。(注，”W”前只能设置具体的数字,不允许区间”-“)。

\#    序号(表示每月的第几个周几)，例如在周字段上设置”6#3”表示在每月的第三个周六.注意如果指定”#5”,正好第五周没有周六，则不会触发该配置

’L’和‘W’组合使用。在日字段上设置”LW”,则表示在本月的最后一个工作日触发；周字段的设置，若使用英文字母是不区分大小写的，即MON与mon相同。

### zone时区.一般留空

fixedDelay上一次执行完毕后多长时间再执行

@Scheduled(fixedDelay = 5000) //上一次执行完毕时间点之后5秒再执行

fixedDelayString 同上的字符串形式,支持占位符

@Scheduled(fixedDelayString = "5000") //上一次执行完毕时间点之后5秒再执行
 fixedRate上一次开始执行后多长时间再执行

@Scheduled(fixedRate = 5000) //上一次开始执行时间点之后5秒再执行

fixedRateString同上的字符串形式。支持占位符

initialDelay第一次延迟多长时间后再执行

@Scheduled(initialDelay=1000, fixedRate=5000) //第一次延迟1秒后执行，之后按fixedRate的规则每5秒执行一次

initialDelayString同上的字符串形式。支持占位符





# 跨域



Request Method: OPTIONS		会在真实请求之前先用OPTIONS验证是否拥有跨域权限



















# Environment类(Spring自身的类)

可以把Spring应用的运行时分成两个部分：一个是Spring应用本身，一个是Spring应用所处的环境。

定时注解@Scheduled有时需要获取当前的运行环境(用户信息,配置文件信息等)

Environment在容器中是一个抽象的集合，是指应用环境的2个方面：profiles和properties。

\1. Profile

  不管是XML还是注解，Beans都有可能指派给profile配置。Environment环境对象的作用，对于profiles配置来说，它能决定当前激活的是哪个profile配置，和哪个profile是默认。

\2. Properties

  properties来源于properties文件、JVM properties、system环境变量、JNDI、servlet context parameters上下文参数、专门的properties对象，Maps等等。对于properties来说，Environment对象可以提供给用户方便的服务接口、方便撰写配置、方便解析配置。
     environment.getProperty获取配置文件中的属性

 





# 异常错误



## unreachable code编译错误

Java检查到他们后面的语句都无法执行下去，

* 跳到下一次循环或其他地方
* 死循环，无法执行下一句



Checked exception:这类异常都是Exception的子类 

Unchecked exception: 这类异常都是RuntimeException的子类





## ConcurrentModificationException

方法检测到对象的并发修改，不允许修改时，抛出异常



### modCount修改次数

modCount被定义在ArrayList的父类AbstractList中，初值为0

```
protected transient int modCount = 0;
```

当发生修改 ,modCount+1 ,通过比对modCount实现了快速失败原则



在ArrayList的add方法中

```
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;}
```

ensureCapacityInternal方法判断时都需要扩容 ,该方法调用了ensureExplicitCapacity ,使modCount+1

```
private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));}

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;
        if (minCapacity - elementData.length > 0)
            grow(minCapacity); }
```



**在迭代时只能用迭代器进行删除**

单线程情况：

（1）使用Iterator提供的remove方法，用于删除当前元素。

（2）建立一个集合，记录需要删除的元素，之后统一删除。

（3）不使用Iterator进行遍历，需要之一的是自己保证索引正常。

（4）使用并发集合类来避免ConcurrentModificationException，比如使用CopyOnArrayList，而不是ArrayList。

多线程情况：

（1）使用并发集合类，如使用ConcurrentHashMap或者CopyOnWriteArrayList。













# 调试



## 创建指定大小文件

在目录下进入cmd

fsutil file createnew test.txt 字节数



# adb命令

adb logcat -c
adb logcat > C:\Users\howlett\Desktop\a.txt













# MIME

| acx     | application/internet-property-stream    |
| ------- | --------------------------------------- |
| ai      | application/postscript                  |
| aif     | audio/x-aiff                            |
| aifc    | audio/x-aiff                            |
| aiff    | audio/x-aiff                            |
| asf     | video/x-ms-asf                          |
| asr     | video/x-ms-asf                          |
| asx     | video/x-ms-asf                          |
| au      | audio/basic                             |
| avi     | video/x-msvideo                         |
| axs     | application/olescript                   |
| bas     | text/plain                              |
| bcpio   | application/x-bcpio                     |
| bin     | application/octet-stream                |
| bmp     | image/bmp                               |
| c       | text/plain                              |
| cat     | application/vnd.ms-pkiseccat            |
| cdf     | application/x-cdf                       |
| cer     | application/x-x509-ca-cert              |
| class   | application/octet-stream                |
| clp     | application/x-msclip                    |
| cmx     | image/x-cmx                             |
| cod     | image/cis-cod                           |
| cpio    | application/x-cpio                      |
| crd     | application/x-mscardfile                |
| crl     | application/pkix-crl                    |
| crt     | application/x-x509-ca-cert              |
| csh     | application/x-csh                       |
| css     | text/css                                |
| dcr     | application/x-director                  |
| der     | application/x-x509-ca-cert              |
| dir     | application/x-director                  |
| dll     | application/x-msdownload                |
| dms     | application/octet-stream                |
| doc     | application/msword                      |
| dot     | application/msword                      |
| dvi     | application/x-dvi                       |
| dxr     | application/x-director                  |
| eps     | application/postscript                  |
| etx     | text/x-setext                           |
| evy     | application/envoy                       |
| exe     | application/octet-stream                |
| fif     | application/fractals                    |
| flr     | x-world/x-vrml                          |
| gif     | image/gif                               |
| gtar    | application/x-gtar                      |
| gz      | application/x-gzip                      |
| h       | text/plain                              |
| hdf     | application/x-hdf                       |
| hlp     | application/winhlp                      |
| hqx     | application/mac-binhex40                |
| hta     | application/hta                         |
| htc     | text/x-component                        |
| htm     | text/html                               |
| html    | text/html                               |
| htt     | text/webviewhtml                        |
| ico     | image/x-icon                            |
| ief     | image/ief                               |
| iii     | application/x-iphone                    |
| ins     | application/x-internet-signup           |
| isp     | application/x-internet-signup           |
| jfif    | image/pipeg                             |
| jpe     | image/jpeg                              |
| jpeg    | image/jpeg                              |
| jpg     | image/jpeg                              |
| js      | application/x-javascript                |
| latex   | application/x-latex                     |
| lha     | application/octet-stream                |
| lsf     | video/x-la-asf                          |
| lsx     | video/x-la-asf                          |
| lzh     | application/octet-stream                |
| m13     | application/x-msmediaview               |
| m14     | application/x-msmediaview               |
| m3u     | audio/x-mpegurl                         |
| man     | application/x-troff-man                 |
| mdb     | application/x-msaccess                  |
| me      | application/x-troff-me                  |
| mht     | message/rfc822                          |
| mhtml   | message/rfc822                          |
| mid     | audio/mid                               |
| mny     | application/x-msmoney                   |
| mov     | video/quicktime                         |
| movie   | video/x-sgi-movie                       |
| mp2     | video/mpeg                              |
| mp3     | audio/mpeg                              |
| mpa     | video/mpeg                              |
| mpe     | video/mpeg                              |
| mpeg    | video/mpeg                              |
| mpg     | video/mpeg                              |
| mpp     | application/vnd.ms-project              |
| mpv2    | video/mpeg                              |
| ms      | application/x-troff-ms                  |
| mvb     | application/x-msmediaview               |
| nws     | message/rfc822                          |
| oda     | application/oda                         |
| p10     | application/pkcs10                      |
| p12     | application/x-pkcs12                    |
| p7b     | application/x-pkcs7-certificates        |
| p7c     | application/x-pkcs7-mime                |
| p7m     | application/x-pkcs7-mime                |
| p7r     | application/x-pkcs7-certreqresp         |
| p7s     | application/x-pkcs7-signature           |
| pbm     | image/x-portable-bitmap                 |
| pdf     | application/pdf                         |
| pfx     | application/x-pkcs12                    |
| pgm     | image/x-portable-graymap                |
| pko     | application/ynd.ms-pkipko               |
| pma     | application/x-perfmon                   |
| pmc     | application/x-perfmon                   |
| pml     | application/x-perfmon                   |
| pmr     | application/x-perfmon                   |
| pmw     | application/x-perfmon                   |
| pnm     | image/x-portable-anymap                 |
| pot,    | application/vnd.ms-powerpoint           |
| ppm     | image/x-portable-pixmap                 |
| pps     | application/vnd.ms-powerpoint           |
| ppt     | application/vnd.ms-powerpoint           |
| prf     | application/pics-rules                  |
| ps      | application/postscript                  |
| pub     | application/x-mspublisher               |
| qt      | video/quicktime                         |
| ra      | audio/x-pn-realaudio                    |
| ram     | audio/x-pn-realaudio                    |
| ras     | image/x-cmu-raster                      |
| rgb     | image/x-rgb                             |
| rmi     | audio/mid                               |
| roff    | application/x-troff                     |
| rtf     | application/rtf                         |
| rtx     | text/richtext                           |
| scd     | application/x-msschedule                |
| sct     | text/scriptlet                          |
| setpay  | application/set-payment-initiation      |
| setreg  | application/set-registration-initiation |
| sh      | application/x-sh                        |
| shar    | application/x-shar                      |
| sit     | application/x-stuffit                   |
| snd     | audio/basic                             |
| spc     | application/x-pkcs7-certificates        |
| spl     | application/futuresplash                |
| src     | application/x-wais-source               |
| sst     | application/vnd.ms-pkicertstore         |
| stl     | application/vnd.ms-pkistl               |
| stm     | text/html                               |
| svg     | image/svg+xml                           |
| sv4cpio | application/x-sv4cpio                   |
| sv4crc  | application/x-sv4crc                    |
| swf     | application/x-shockwave-flash           |
| t       | application/x-troff                     |
| tar     | application/x-tar                       |
| tcl     | application/x-tcl                       |
| tex     | application/x-tex                       |
| texi    | application/x-texinfo                   |
| texinfo | application/x-texinfo                   |
| tgz     | application/x-compressed                |
| tif     | image/tiff                              |
| tiff    | image/tiff                              |
| tr      | application/x-troff                     |
| trm     | application/x-msterminal                |
| tsv     | text/tab-separated-values               |
| txt     | text/plain                              |
| uls     | text/iuls                               |
| ustar   | application/x-ustar                     |
| vcf     | text/x-vcard                            |
| vrml    | x-world/x-vrml                          |
| wav     | audio/x-wav                             |
| wcm     | application/vnd.ms-works                |
| wdb     | application/vnd.ms-works                |
| wks     | application/vnd.ms-works                |
| wmf     | application/x-msmetafile                |
| wps     | application/vnd.ms-works                |
| wri     | application/x-mswrite                   |
| wrl     | x-world/x-vrml                          |
| wrz     | x-world/x-vrml                          |
| xaf     | x-world/x-vrml                          |
| xbm     | image/x-xbitmap                         |
| xla     | application/vnd.ms-excel                |
| xlc     | application/vnd.ms-excel                |
| xlm     | application/vnd.ms-excel                |
| xls     | application/vnd.ms-excel                |
| xlt     | application/vnd.ms-excel                |
| xlw     | application/vnd.ms-excel                |
| xof     | x-world/x-vrml                          |
| xpm     | image/x-xpixmap                         |
| xwd     | image/x-xwindowdump                     |
| z       | application/x-compress                  |
| zip     | application/zip                         |



 

# REST



## 无状态stateless



在请求中传递SessionID是unRESTful的，而将用户的credentials(认证信息)包含在每个请求里是RESTful的

“状态”通常指的是为两个相互关联的用户交互操作保留的某种公共信息，常被用来存储工作流或用户状态信息。这些信息可以被指定不同的作用域如page，request，session或全局作用域，而存储他们的责任也同样可以由Client端或Server端负责

虽然存储状态为企业软件开发带来了诸多便利，但是它也给分布式系统的其他方面带来了许多限制，比如在负载均衡方面，在有状态的模式下，一个用户的请求必须被提交到保存有其相关状态信息的服务器上，否则这些请求可能无法被理解，这也就意味着在此模式下服务器端无法对用户请求进行自由调度。于此相关的另一个问题是容错性，倘若保有用户信息的服务器宕机，那么该用户最近的所有交互操作将无法被透明地移送至备用服务器上，除非该服务器时刻与主服务器同步全部用户的状态信息。此外，由于**HTTP**本身无状态，开发人员必须通过模拟实现状态的钝化与激活

无状态指的是请求必须与其他请求隔离，当请求端提出请求时，**请求本身包含了相应端为相应这一请求所需的全部信息**。这一约束的出现改善了分布式系统的可见性、可靠性以及可伸缩性。如果一个网站期望用户以A->B->C的流程来交互，而在执行至B时回退的话，那么系统很有可能不是按照其所期望的方式运行，因为用户的状态可能被不可逆地修改了。反过来，任何用户可以在浏览器地址栏中输入http://www.google.com/search?q=RESTful&start=100来获得从第一百条开始的关于RESTful的记录，并且当Google服务器瘫痪时，相关用户请求会被透明地移送至其他服务器。



* RESTful的2中状态
  * 应用状态  某一特定请求相关的状态信息
  * 资源状态  某一存储在服务器端资源在某一时刻的特定状态，该状态不会因为用户请求而改变，任何用户在同一时刻对该资源的请求都会获得这一状态的表现（Representation）。



RESTful架构要求服务器端不保有任何与特定HTTP请求相关的资源，所以==应用状态必须由请求方在请求过程中提供==。那么为什么传递一个session ID是违背REST架构风格而传递user credentials却不是。实际上“传递某种表示状态的信息”到服务器不是“有状态”的表现。**有状态和无状态与请求本身没有多大关联，重要的是状态信息是由请求方还是响应方负责保存**。在Session ID可以被认为是一个用来标识某一会话状态的Key，将其传递给服务器端意味着这样一个请求：“请帮我取出这个状态信息”，也就是说这个请求假设响应方保有着状态信息。由于与某一特定请求相关的状态属于应用状态，而RESTful架构要求任何此类状态由请求方负责提供，所以传递Session ID被认为是unRESTful的做法。反过来，user credential作为一种应用状态，是被期望由请求方提供的，所以在请求中传递user credentials是符合RESTful架构规范的。





















## .class 和.jar 类型的文件存放位置？

.class 文件放在 WEB-INF/classes 文件下，.jar 文件放在 WEB-INF/lib文件夹下



## char 型变量中能不能存储一个中文汉字？

java采用unicode编码，2个字节（16位）来表示一个字符， 无论是汉字还是数字，字母，或其他语言都可以存储。





## assert

assertion(断言)在软件开发中是一种常用的调试方式，很多开发语言中都支持这种机制。一般来说，assertion 用于保证程序最基本、关键的正确性。assertion 检查通常在开发和测试时开启。为了提高性能，在软件发布后， assertion 检查通常是关闭的。在实现中，断言是一个包含布尔表达式的语句，在执行这个语句时假定该表达式为 true；如果表达式计算为 false，那么系统会报告一个 AssertionError。

断言用于调试目的：

assert(a > 0); // throws an AssertionError if a <= 0

断言可以有两种形式：

assert Expression1;

assert Expression1 : Expression2 ;

Expression1 应该总是产生一个布尔值。

Expression2 可以是得出一个值的任意表达式；这个值用于生成显示更多调

试信息的字符串消息。

断言在默认情况下是禁用的，要在编译时启用断言，需使用 source 1.4 标

记：

javac -source 1.4 Test.java

要在运行时启用断言，可使用-enableassertions 或者-ea 标记。

要在运行时选择禁用断言，可使用-da 或者-disableassertions 标记。

要在系统类中启用断言，可使用-esa 或者-dsa 标记。还可以在包的基础上

启用或者禁用断言。可以在预计正常情况下不会到达的任何位置上放置断言。

断言可以用于验证传递给私有方法的参数。不过，断言不应该用于验证传递

给公有方法的参数，因为不管是否启用了断言，公有方法都必须检查其参数。

不过，既可以在公有方法中，也可以在非公有方法中利用断言测试后置条件。

另外，断言不应该以任何方式改变程序的状态。

## Session

### Session的两种实现方法

#### 1、基于Cookie实现Session

服务器为客户端创建并维护Session对象，用于存放数据。同时会产生SessionID，服务器以Cookie的方式将SessionID存放在客户端。当浏览器再次访问该服务器时，会将SessionID作为Cookie信息带到服务器，服务器可以通过该SessionID检索到以前的Session对象，并对其进行访问。需要注意的是，此时的Cookie中仅仅保存了一个SessionID，而相对较多的会话数据保存在服务器端对应的Session对象中，由服务器来统一维护，这样一定程度保证了会话数据安全性，但增加了服务器端的内存开销。

存放在客户端的用于保存SessionID的Cookie会在浏览器关闭时清除。我们把用户打开一个浏览器访问某个应用开始，到关闭浏览器为止交互过程称为一个“会话”。在一个“会话”过程中，可能会向同一个应用发出了多次请求，这些请求将共享一个Session对象，因为这些请求携带了相同的SessionID信息。

#### 2、基于URL重写

Session对象的正常使用要依赖于Cookie。如果考虑到客户端浏览器出于安全的考虑禁用了Cookie，应该使用URL重写的方式使Session在客户端禁用Cookie的情况下继续生效。

### session 与 cookie 的区别

存储角度：

Session是服务器端的数据存储技术，cookie是客户端的数据存储技术

解决问题角度：

 Session 解决的是同一用户不同请求的数据共享问题，cookie 解决的是不同用户不同请求的请求数据的共享问题

 生命周期角度：

 Session的id是依赖于cookie来进行存储的，浏览器关闭 id 就会失效

Cookie 可以单独设置其在浏览器的存储时间。

## HTTP状态码

### 2xx	成功处理了请求

### 3xx (重定向)表示要完成请求，需要进一步操作。

304 (未修改) 自从上次请求后，请求的网页未修改过。 服务器返回此响应时，不会返回网页内容。

### 4xx(请求错误)请求可能出错，妨碍服务器的处理

400 (错误请求) 服务器不理解请求的语法。

403 (禁止) 服务器拒绝请求。

### 5xx(服务器错误) 

500 (服务器内部错误) 服务器遇到错误，无法完成请求。

501 (尚未实施) 服务器不具备完成请求的功能。 例如，服务器无法识别请求方法

502 (错误网关) 服务器作为网关或代理，从上游服务器收到无效响应。

503 (服务不可用) 服务器目前无法使用(由于超载或停机维护)。 通常，这只是暂时状态。

504 (网关超时) 服务器作为网关或代理，但是没有及时从上游服务器收到请求。

505 (HTTP 版本不受支持) 服务器不支持请求中所用的 HTTP 协议版本。



### Ajax 的工作原理

异步的javascript和xml,通过XmlHttpRequest对象来向服务器发异步请求，从服务器获得数据，然后用javascript来操作DOM而更新页面。从而实现向服务器提出请求和处理响应，而不阻塞用户。达到无刷新的效果

### JSON 及其作用

JSON是一种轻量级的数据交换格式，采用完全独立于语言的文本格式，是理想的数据交换格式。同时，JSON 是 JavaScript 原生格式，这意味着在 JavaScript 中处理 JSON 数据不须要任何特殊的 API 或工具包。

在 JSON 中，有两种结构：对象和数组。

 {} 对象

 [] 数组

 , 分隔属性

 : 左边为属性名，右边为属性值

属性名可用可不用引号括起，属性值为字符串一定要用引号括起

## Servlet

### Servlet生命周期

Web容器加载Servlet并将其实例化后，Servlet生命周期开始，容器运行init()方法进行Servlet的初始化；请求到达时调用service方法，service方法调用与请求对应的doGet或doPost等方法；当服务器关闭或项目被卸载时Servlet 实例被销毁，此时会调用Servlet的destroy方法

### JSP 和 Servlet关系

先有 Servlet，针对 Servlet 缺点推出 JSP。JSP 是 Servlet 的一种特殊形式

Servlet是特殊的Java程序，运行于服务器的JVM中，能够依靠服务器的支持向浏览器提供显示内容。JSP本质上是Servlet的一种简易形式，JSP被处理成类似于Servlet的Java程序，可以简化页面内容的生成。

不同点在于，Servlet 的应用逻辑在Java文件中，并完全从表示层中的HTML分离开来。而JSP是Java语言和HTML的组合。JSP侧重于视图，Servlet侧重于控制逻辑，在MVC架构模式中，JSP适合充当视图而Servlet 适合充当控制器（controller）

 

每个 JSP 页面就是一个 Servlet 实例——JSP 页面由系统翻译成 Servlet，

Servlet 再负责响应用户请求。



## 客户端存数据3种方法



* cookie

  会失效 ,下次请求cookie会被携带一起发送 ,不适合存储大量数据

* sessionStorage

  页面关闭则失效

* localStorage

  浏览器缓存被清空则失效



## 单点登录、域用户、常规登录、AD域

* 单点登录  
  * 用户只需要登录一次就可以访问所有相互信任的应用系统。（Single Sign On，简称为 SSO）
  * 各个server拿到同一个ID，都有办法检验ID有效 ,得到用户信息。

实现步骤：

登录应用1，服务器前验证 ——> Over（不通过，不通过验证失败）

​                     ——> 返回ticket（验证通过） ——> 下次访问应用,2，发送ticket验证  ——> 以后登录应用2不需要再次登录（验证通过）

实现SSO，具备条件所有应用系统共享一个身份认证系统。

　　统一的认证系统是SSO的前提之一。认证系统的主要功能是将用户的登录信息和用户信息库相比较，对用户进行登录认证；认证成功后，认证系统应该生成统一的认证标志（ticket），返还给用户。另外，认证系统还应该对ticket进行效验，判断其有效性。所有应用系统均能够识别和提取ticket信息

　　要实现SSO的功能，让用户只登录一次，就必须让应用系统能够识别已经登录过的用户。应用系统应该能对ticket进行识别和提取，通过与认证系统的通讯，能自动判断当前用户是否登录过，从而完成单点登录的功能。

 

2、域用户

（1）何为域？何为域用户？

域，域就是一方诸侯，有自己的权限和领土范围。（自己的装逼解释）；而领地内所有管辖和被管辖的都是域用户。

（2）域，还是域用户？

看了所有能看到的解释，都不满意。自己定义下，域的主要是，域对内的规则，域内用户的权限，域和域之间的规则；同域内用户之间的关系，权限。域，例如计算机里的域用户是需要服务器，进行域的创建，

 

（3）域账号

域账号，修改域帐号有关数据，直接修改域帐号服务器中的帐号，其他计算机就可立即获取更新后的帐号数据；本地账号，域中有数十台计算机，而且每一台计算机都必须有相同的帐号

3、AD域

AD域是Active Directory的缩写，它是基于windows的一个组合，它可以集中控制加入域的所有计算机的权限，更高效的分配权限、提高资料的安全性、节省管理成本。

域用户（我这里指的是创建域的这个用户），在任何一台加入域的计算机上都有管理员的权限。





# Spring

 

## Spring事务传播机制



*  原子性（Atomicity）：事务是一个原子操作，由一系列动作组成。事务的原子性确保动作要么全部完成，要么完全不起作用。

*  一致性（Consistency）：一旦事务完成（不管成功还是失败），系统必须确保它所建模的业务处于一致的状态，而不会是部分完成部分失败。在现实中的数据不应该被破坏。

*  隔离性（Isolation）：可能有许多事务会同时处理相同的数据，因此每个事务都应该与其他事务隔离开来，防止数据损坏。

*  持久性（Durability）：一旦事务完成，无论发生什么系统错误，它的结果都不应该受到影响，这样就能从任何系统崩溃中恢复过来。通常情况下，事务的结果被写到持久化存储器中。

## Spring事务的配置方式

### 1. 编程式事务管理

编程式事务管理是侵入性事务管理，使用TransactionTemplate或者直接使用PlatformTransactionManager，对于编程式事务管理，Spring推荐使用TransactionTemplate。

### 2. 声明式事务管理

声明式事务管理建立在AOP之上，其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，执行完目标方法之后根据执行的情况提交或者回滚。
 编程式事务每次实现都要单独实现，但业务量大功能复杂时，使用编程式事务无疑是痛苦的，而声明式事务不同，声明式事务属于无侵入式，不会影响业务逻辑的实现，只需要在配置文件中做相关的事务规则声明或者通过注解的方式，便可以将事务规则应用到业务逻辑中。
 显然声明式优于编程式事务管理，这正是Spring倡导的非侵入式的编程方式。唯一不足的地方就是声明式事务管理的粒度是方法级别，而编程式事务管理是可以到代码块的，但是可以通过提取方法的方式完成声明式事务管理的配置。

## 事务的传播机制

事务的传播性一般用在事务嵌套的场景，事务方法里调用了另外一个事务方法，需要事务传播机制的配置确定两个方法是各自作为独立的方法提交还是内层的事务合并到外层的事务一起提交

·     PROPAGATION_REQUIRED默认

外层有事务，则加入外层事务。没有则新建一个事务执行

·     PROPAGATION_REQUES_NEW

外层事务挂起，开启新事务，当前事务执行完毕，恢复上层事务的执行。如果外层没有事务，执行当前新开的事务

·     PROPAGATION_SUPPORT
 外层有事务，则加入外层事务，外层没有事务，以非事务方式执行。完全依赖外层的事务

·     PROPAGATION_NOT_SUPPORT

不支持事务，如果外层有事务则挂起，执行完当前代码，则恢复外层事务，无论是否异常都不会回滚当前的代码

·     PROPAGATION_NEVER

不支持外层事务，即如果外层有事务就抛出异常

·     PROPAGATION_MANDATORY
 与NEVER相反，如果外层没有事务则抛出异常

·     PROPAGATION_NESTED可以保存状态保存点，当前事务回滚到某一个点，从而避免所有的嵌套事务都回滚，如果子事务没有把异常吃掉，基本还是会引起全部回滚的



## 只读

如果一个事务只对数据库执行读操作，那么该数据库就可能利用那个事务的只读特性，采取某些优化措施。通过把一个事务声明为只读，可以给后端数据库一个机会来应用那些它认为合适的优化措施。由于只读的优化措施是在一个事务启动时由后端数据库实施的， 因此，只有对于那些具有可能启动一个新事务的传播行为（REQUIRES_NEW、EQUIRED、NESTED）的方法来说，将事务声明为只读才有意义。

## 事务超时

事务可能涉及对数据库的锁定，长时间运行事务会不必要地占用数据库资源。这时就可以声明一个事务在特定秒数后自动回滚，不必等它自己结束。

由于超时时钟在一个事务启动的时候开始的，因此，只有对于那些具有可能启动一个新事务的传播行为（REQUIRES_NEW、REQUIRED、NESTED）的方法来说，声明事务超时才有意义。

## 回滚规则

在默认设置下，事务只在出现运行时异常（runtime exception）时回滚，而在出现受检查异常（checked exception）时不回滚。

不过，可以声明在出现特定受检查异常时像运行时异常一样回滚。同样，也可以声明一个事务在出现特定的异常时不回滚，即使特定的异常是运行时异常。

## Spring声明式事务配置参考

* 事务的传播性：
  @Transactional(propagation=Propagation.REQUIRED)

* 事务的隔离级别：
  @Transactional(isolation = Isolation.READ_UNCOMMITTED)

读取未提交数据(会出现脏读, 不可重复读) 基本不使用

* 只读：@Transactional(readOnly=true)
  该属性用于设置当前事务是否为只读事务，设置为true表示只读，false则表示可读写，默认值为false。

* 事务的超时性：@Transactional(timeout=30)

* 回滚：
  * 指定单一异常类：@Transactional(rollbackFor=RuntimeException.class)
  * 指定多个异常类：@Transactional(rollbackFor={RuntimeException.class, Exception.class})







## 注解还是XML

1、应用的基本配置用xml，比如：数据源、资源文件等；

2、业务开发用注解，比如：Service中注入bean等；

## SpringBoot的优点

1，创建独立的spring应用程序。

2，嵌入的tomcat jetty 或者undertow 不用部署WAR文件。

3，允许通过Maven来根据需要获取starter

4，尽可能的使用自动配置spring

5，提供生产就绪功能，如指标，健康检查和外部配置

6，绝对没有代码生成，对XML没有要求配置

　

## 传统开发模式

所有的功能打包在一个 WAR包里，基本没有外部依赖（除了容器），部署在一个JEE容器（Tomcat，JBoss，WebLogic）里，包含了 DO/DAO，Service，UI等所有逻辑。

 

* 优点：

①开发简单，集中式管理

②基本不会重复开发

③功能都在本地，没有分布式的管理和调用消耗

* 缺点：

1、效率低：开发都在同一个项目改代码，相互等待，冲突不断

2、维护难：代码功功能耦合在一起，新人不知道何从下手

3、不灵活：构建时间长，任何小修改都要重构整个项目，耗时

4、稳定性差：一个微小的问题，都可能导致整个应用挂掉

5、扩展性不够：无法满足高并发下的业务需求

6、对服务器的性能要求要统一，要高