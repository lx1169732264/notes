

# 代码规范



## 编码规约



### 命名规范



* 抽象类命名使用Abstract或Base开头；异常类命名使用Exception结尾；测试类命名以它要测试的类的名称开始，以Test结尾



* POJO类中布尔类型变量都不要加is前缀，否则部分框架解析会引起序列化错误
  * 在建表约定第一条，表达是与否的值采用is_xxx的命名方式，所以，需要在 <resultMap>设置从is_xxx到xxx的映射关系。

反例：定义为基本数据类型Boolean isDeleted的属性，它的方法也是isDeletedO，RPC框架在反向解 析的时候，"误以为"对应的属性名称是deleted，导致属性获取不到，进而抛出异常



【强制】包名统一使用小写，点分隔符之间有且仅有一个自然语义的英语单词。包名统一使用单数形式，但是类名如果有复数含义，类名可以使用复数形式。

正例：应用工具类包名为com.alibaba.ai.util、类名为MessageUtils (此规则参考spring的框架结构)



* 避免父子类的成员变量/不同代码块的局部变量之间采用相同的命名，使可读性降低

说明：子类、父类成员变量名相同，即使是public类型的变量也是能够通过编译，而局部变量在同一方法 内的不同代码块中同名也是合法的，但是要避免使用。对于非setter/getter的参数名称也要避免与成员变量名称相同



* 【强制】杜绝完全不规范的缩写，避免望文不知义。
  * 反例：AbstractClass "缩写"命名成AbsClass ； condition "缩写"命名成condi，此类随意缩写严重 降低了代码的可阅读性



【推荐】为了达到代码自解释的目标，任何自定义编程元素在命名时，使用尽量完整的单词 组合来表达其意。

正例：在JDK中，表达原子更新的类名为：AtomicReferenceFieldUpdater。

反例：int a的随意命名方式。



\1. 【推荐】在常量与变量的命名时，表示类型的名词放在词尾，以提升辨识度。

正例：startTime / workQueue / nameList / TERMINATED_THREAD_COUNT

反例：startedAt / QueueOfWork / listName / COUNT_TERMINATED_THREAD

\2. 【推荐】如果模块、接口、类、方法使用了设计模式，在命名时需体现出具体模式。

说明：将设计模式体现在名字中，有利于阅读者快速理解架构设计理念。

正例： public class OrderFactory;

public class LoginProxy;

public class ResourceObserver;

\3. 【推荐】接口类中的方法和属性不要加任何修饰符号(public也不要加)，保持代码的简洁 性，并加上有效的 Javadoc 注释。尽量不要在接口里定义变量，如果一定要定义变量，肯定 是与接口方法相关，并且是整个应用的基础常量。

正例：接口方法签名 void commit();

接口基础常量 String COMPANY = "alibaba";

反例：接口方法定义 public abstract void f();

说明：JDK8中接口允许有默认实现，那么这个default方法，是对所有实现类都有价值的默认实现。

\4. 接口和实现类的命名有两套规则：

1)【强制】对于Service和DAO类，基于SOA的理念，暴露出来的服务一定是接口，内部的实现类用 Impl 的后缀与接口区别。

正例： CacheServiceImpl 实现 CacheService 接口。

2 )【推荐】如果是形容能力的接口名称，取对应的形容词为接口名(通常是-able的形容词)。

正例： AbstractTranslator 实现 Translatable 接口。

\5. 【参考】枚举类名带上 Enum 后缀，枚举成员名称需要全大写，单词间用下划线隔开。

说明：枚举其实就是特殊的类，域成员均为常量，且构造方法被默认强制是私有。

正例：枚举名字为 ProcessStatusEnum 的成员名称：SUCCESS / UNKNOWN_REASON。

\6. 【参考】各层命名规约：

A) Service/DAO层方法命名规约

1)获取单个对象的方法用get做前缀。

2 )获取多个对象的方法用list做前缀，复数形式结尾如：listObjects。

3 )获取统计值的方法用count做前缀。

4) 插入的方法用 save/insert 做前缀。

5) 删除的方法用 remove/delete 做前缀。

6) 修改的方法用 update 做前缀。

B) 领域模型命名规约

1) 数据对象：xxxDO，xxx即为数据表名。

2) 数据传输对象： xxxDTO， xxx 为业务领域相关的名称。

3) 展示对象： xxxVO， xxx 一般为网页名称。

4 ) POJO是DO/DTO/BO/VO的统称，禁止命名成xxxPOJO。



方法命名：一般有io消耗的方法，名称用findXxxxx, POJO类里自己写的方法名不允许使用getXxxx, 一律使用obtainXxxxx, 如果方法里面涉及复杂计算的化可以使用computeXxxx





### 常量定义



【强制】不允许任何魔法值(即未经预先定义的常量)直接出现在代码中

​	反例:String key = "Id#taobao_" + tradeld;

​			cache.put(key, value);	//代码复制时漏掉下划线，导致缓存get击穿



【强制】在long或者Long赋值时，数值后使用大写的L，不能是小写的I，小写容易跟数字 1 混淆，造成误解。

说明：Long a = 2l;写的是数字的21，还是Long型的2。



【推荐】不要使用一个常量类维护所有常量，要按常量功能进行归类，分开维护

说明：大而全的常量类，杂乱无章，使用查找功能才能定位到修改的常量，不利于理解和维护

正例：缓存相关常量放在类CacheConsts下；系统配置相关常量放在类ConfigConsts下



【推荐】常量的复用层次有五层：跨应用共享常量、应用内共享常量、子工程内共享常量、 包内共享常量、类内共享常量。

1) 跨应用共享常量：放置在二方库中，通常是client.jar中的constant目录下。

2) 应用内共享常量：放置在一方库中，通常是子模块中的constant目录下。

反例：易懂变量也要统一定义成应用内共享常量，两位工程师在两个类中分别定义了 "YES"的变量： 类 A 中： public static final String YES = "yes";

类 B 中：public static final String YES = "y";

A.YES.equals(B.YES)，预期是true，但实际返回为false，导致线上问题。

2 )子工程内部共享常量：即在当前子工程的constant目录下。

3 )包内共享常量：即在当前包下单独的constant目录下。

4 )类内共享常量：直接在类内部private static final定义。



【推荐】如果变量值仅在一个固定范围内变化用 enum 类型来定义



### 代码格式



【强制】大括号内非空则：

1 ) 左大括号前不换行

2 ) 左大括号后换行

3 ) 右大括号后还有 else ? 不换行 : 换行



【强制】左小括号和字符之间不出现空格；同样，右小括号和字符之间也不出现空格；而左大括号前需要空格



【强制】任何二目、三目运算符的左右两边都需要加一个空格



【强制】采用4个空格缩进，禁止使用tab字符,IDEA设置tab为4个空格时，请勿勾选Use tab character



【强制】//与注释内容只有一个空格



【强制】在进行类型强制转换时，右括号与强制转换值之间不需要任何空格隔开。

正例：

long first = 1000000000000L;

int second = (int)first + 2;



【强制】单行字符数限制不超过 120 个，超出需要换行，换行时遵循如下原则：

1）第二行相对第一行缩进 4 个空格，从第三行开始，不再继续缩进，参考示例。

2）运算符与下文一起换行。

3）方法调用的点符号与下文一起换行。

4）方法调用中的多个参数需要换行时，在逗号后进行。 

5）在括号前不要换行，见反例

正例：

StringBuilder sb = new StringBuilder(); 

// 超过 120 个字符的情况下，换行缩进 4 个空格，点号和方法名称一起换行

sb.append("Jack").append("Ma")... 

.append("alibaba")... 

.append("alibaba")... 

.append("alibaba");

反例：

StringBuilder sb = new StringBuilder(); 

// 超过 120 个字符的情况下，不要在括号前换行

sb.append("Jack").append("Ma")...append 

("alibaba"); 

// 参数很多的方法调用可能超过 120 个字符，不要在逗号前换行

method(args1, args2, args3, ... 

, argsX); 



【强制】IDE 的 text file encoding 设置为 UTF-8; IDE 中文件的换行符使用 Unix 格式，不要使用 Windows 格式



【推荐】单个方法的总行数不超过 80 行。

说明：除注释之外的方法签名、左右大括号、方法内代码、空行、回车的总行数不超过80 

正例：分清**红花和绿叶，个性和共性**，绿叶逻辑单独出来成为额外方法，使主干代码更加清晰；共性逻辑抽取成为共性方法，便于复用和维护





### OOP规约



【强制】避免通过一个类的对象引用访问此类的静态变量或静态方法，无谓增加编译器解析成本，直接用类名访问



【强制】相同参数类型，相同业务含义，才可以使用 Java 的可变参数，避免使用 Object。

说明：可变参数必须放置在参数列表的最后。（提倡同学们尽量不用可变参数编程）

正例：public List<User> listUsers(String type, Long... ids) {...}



【强制】外部正在调用或者二方库依赖的接口，不允许修改方法签名，避免对接口调用方产生影响。接口过时必须加@Deprecated 注解，并清晰地说明采用的新接口或者新服务是什么。



【强制】不能使用过时的类或方法

说明：java.net.URLDecoder 中的方法 decode(String encodeStr) 这个方法已经过时，应该使用双参数decode(String source, String encode)。接口提供方既然明确是过时接口，那么有义务同时提供新的接口；作为调用方来说，有义务去考证过时方法的新实现是什么



【强制】所有整型包装类对象之间值的比较，全部使用 equals()

说明：-128 ~ 127 范围内的 Integer 对象是在 IntegerCache.cache 产生，会复用已有对象，区间之外的所有数据，都会在堆上产生



【强制】浮点数的等值判断，基本数据类型不能用==来比较，包装数据类型不能用 equals( )

说明：浮点数采用“尾数+阶码”的编码方式，二进制无法精确表示大部分的十进制小数

反例：

```java
 //反例
float a = 1.0f - 0.9f;
float b = 0.9f - 0.8f;
a == b		//false
  
Float x = Float.valueOf(a);
Float y = Float.valueOf(b);
x.equals(y)	//false

//正例
//(1) 指定一个误差范围，两个浮点数的差值在此范围之内，则认为是相等的。
 float a = 1.0f - 0.9f;
 float b = 0.9f - 0.8f;
 float diff = 1e-6f;
 Math.abs(a - b) < diff	//true

   
//(2) 使用 BigDecimal 来定义值，再进行浮点数的运算操作。
 BigDecimal a = new BigDecimal("1.0");
 BigDecimal b = new BigDecimal("0.9");
 BigDecimal c = new BigDecimal("0.8");

 BigDecimal x = a.subtract(b);
 BigDecimal y = b.subtract(c);
 x.equals(y)	//true
```



==【强制】为了防止精度损失，禁止使用构造方法BigDecimal(double)的方式把double值转 化为 BigDecimal 对象==

说明：BigDecimal(double)存在精度损失风险，在精确计算或值比较的场景中可能会导致业务逻辑异常。 如： BigDecimal g = new BigDecimal(0.1f); 实际的存储值为： 0.10000000149

正例：优先推荐入参为String的构造方法，或使用BigDecimal的valueOf方法，此方法内部其实执行了 Double的toString，而Double的toString按double的实际能表达的精度对尾数进行截断

BigDecimal recommendl = new BigDecimal("0.1");

BigDecimal recommend2 = BigDecimal.valueOf(O.I);





==关于基本数据类型与包装数据类型的使用标准如下==

1. 【强制】所有的 POJO 类属性必须使用包装数据类型

   说明:数据库可能是null，因为自动拆箱，用基本数据类型接收有NPE风险

   反例：比如显示成交总额涨跌情况，即正负x%，x为基本数据类型，RPC调用不成功时， 返回的是默认值，页面显示为 0%，这是不合理的，应该显示成中划线。所以包装数据类型的 null 值，能够表示额外的信息，如：远程调用失败，异常退出

2. 【强制】 RPC 方法的返回值和参数必须使用包装数据类型

3. 【推荐】所有的**局部变量使用基本数据类型**

   作用域只在方法内的变量，直接在栈内存中存储，怎么性能高就怎么定义

4. 【强制】定义DO/DTO/VO等POJO类时，不要设定任何属性默认值

   说明：POJO类属性没有初值是提醒使用者在需要使用时，必须自己显式地进行赋值，任何NPE问题，或者入库检查，都由使用者来保证





【强制】序列化类新增属性时，不要修改serialVersionUID字段，避免反序列失败；如果完全不兼容升级，避免反序列化混乱，修改 serialVersionUID 值

说明：注意 serialVersionUID 不一致会抛出序列化运行时异常



【强制】构造方法/get/se禁止加入任何业务逻辑，如果有初始化逻辑，请放在 init 方法中



【强制】POJO类必须写toString方法。如果继承了另一个 POJO 类，在前面追加super.toString

说明：在方法执行抛出异常时，可以直接调用POJO的toString()方法打印其属性值，便于排查问题



【强制】禁止在POJO类中，同时存在对应属性xxx的isXxx()和getXxx()方法

说明：框架在调用属性xxx的提取方法时，并不能确定哪个方法一定是被优先调用到,Mybatis 和 Hibernate 框架是根据获取方法找到对应属性，因此上述定义可能存在问题



【推荐】使用索引访问用String的split方法得到的数组时，需做最后一个分隔符后有无内容的检查，否则会有抛 IndexOutOfBoundsException 的风险。

说明：

```java
String str = "a,b,c,,";
String[] ary = str.split(",");
System.out.println(ary.length);	// 预期大于 3，结果是3
```



【推荐】 类内方法定义的顺序：公有方法或保护方法 > 私有方法 > getter / setter 方法



【推荐】循环体的字符串的拼接，使用StringBuilder的append



【推荐】下列情况用final

1. 不允许被继承的类，如： String 类
2.  不允许修改引用的域对象
3. 不允许被覆写的方法，如： POJO 类的 set 
4. 不允许运行过程中重新赋值的局部变量
5. 避免上下文重复使用一个变量，使用final可以强制重新定义一个变量，方便更好地进行重构



【推荐】**慎用Object的clone方法来拷贝对象**

说明：对象 clone 方法默认是浅拷贝，若想实现深拷贝需覆写 clone 方法实现域对象的深度遍历式拷贝



【推荐】类成员与方法访问控制从严

1. 如果不允许外部直接通过 new 来创建对象，那么构造方法必须是 private
2. 工具类不允许有public或default构造方法
3. 类非 static 成员变量并且与子类共享，必须是 protected
4. 类非 static 成员变量并且仅在本类使用，必须是 private
5. 类static成员变量如果仅在本类使用，必须是private
6. 若是static成员变量，考虑是否为final
7. 类成员方法只供类内部调用，必须是 private
8. 类成员方法只对继承类公开，那么限制为 protected

任何类、方法、参数、变量，严控访问范围。过于宽泛的访问范围，不利于模块解耦

思考：如果是一个 private 的方法，想删除就删除，可是一个 public 的 service 成员方法或成员变量，删除一下，不得手心冒点汗吗？





### 集合处理



【强制】关于 hashCode 和 equals 的处理，遵循如下规则：

1. 只要覆写equals，就必须覆写hashCode
2. **Set存储的是不重复的对象，依据hashCode和equals进行判断，所以Set存储的对象必须覆写这两个方法**
3. 如果自定义对象作为Map的键，那么必须覆写hashCode和equals

说明：**String已覆写hashCode和equals方法**



【强制】==ArrayList的subList结果不可强转成ArrayList==，否则会抛出ClassCastException异常

说明：subList返回的是ArrayList的内部类SubList，并不是ArrayList而是ArrayList的一个视图，==对subList的所有操作最终会反映到原列表上==



【强制】使用Map的方法keySet()/values()/entrySet()返回集合对象时，不可以对其进行添加元素操作，否则会抛出 UnsupportedOperationException 异常



【强制】Collections类返回的对象，如：emptyList()/singietonList()等都是 immutable list，不可对其进行添加或者删除元素的操作

反例：如果查询无结果，返回Collections.emptyList()空集合对象，调用方一旦进行了添加元素的操作，会触发 UnsupportedOperationException 异常



【强制】在subList场景中，高度注意对原集合元素的增加或删除，均会导致子列表的遍 历、增加、删除产生 ConcurrentModificationException 异常



【强制】使用集合转数组的方法，必须使用集合的toArray(T[] array)，==传入类型一致、长度为0的空数组==

反例：直接使用toArray无参方法存在问题，此方法返回值只能是Object[]类，若强转其它类型数组将出 现 ClassCastException 错误。

正例：

```java
List<String> list = new ArrayList<>(2);
list.add("guan");
list.add("bao");
String[] array = list.toArray(new String[0]);
//0	动态创建与size相同的数组，性能最好
//0<  <list.size	重新创建大小等于size的数组，增加GC负担
//list.size		在高并发时，数组创建完成前size变大,数组需要重新创建，负面影响与上相同
//>list.size	空间浪费，且在size处插入null值，存在NPE隐患
```





【强制】在使用Collection接口任何实现类的addAII()方法时，都要对输入集合参数进行 NPE判断

说明：在ArrayList#addAII方法的第一行代码即Object]] a = c.toArrayO;其中c为输入集合参数，如果 为null，则直接抛出异常



【强制】使用工具类Arrays.asList转换成集合后，不能使用其修改集合相关的方法，会抛UnsupportedOperationException

说明：asList的返回对象是一个Arrays内部类，并没有实现集合的修改方法。Arrays.asList体现的是适配器模式，只是转换接口，后台的数据仍是数组



【强制】泛型通配符<? extends T>来接收返回的数据，此写法的泛型集合不能使用add方 法，而<? super T>不能使用get方法，作为接口调用赋值时易出错

说明：PECS(Producer Extends Consumer Super)原则：

1. 频繁往外读取内容的，适合 用<? extends T>
2. 经常往里插入的，适合用<? super T>



【强制】在无泛型限制定义的集合赋值给泛型限制的集合时，在使用集合元素时，需要进行 instanceof 判断，避免抛出 CIassCastException

说明：毕竟泛型是在JDK5后才出现，考虑到向前兼容，编译器是允许非泛型集合与泛型集合互相赋值



【强制】不要在foreach循环里进行元素的remove/add操作。remove元素请使用Iterator 方式，如果并发操作，需要对 Iterator 对象加锁



【强制】在JDK7版本及以上，Comparator实现类要满足如下三个条件，不然Arrays.sort , Collections.sort 会抛 IllegalArgumentException 异常

1. x，y 的比较结果和 y，x 的比较结果相反
2. x>y,y>z,则 x>z
3. x=y，则xz和yz比较结果相同

**反例：下例中没有处理相等的情况,交换两个对象判断结果并不互反,不符合第一个条件,在实际使用中可能会出现异常**

```java
new Comparator<Student>() {
@Override
public int compare(Student o1, Student o2) {
return o1.getId() > o2.getId() ? 1 : -1;
}
};
```



【推荐】集合泛型定义时，在JDK7及以上，使用diamond(菱形泛型<>)语法或全省略

正例：HashMap<String, String> userCache = new HashMap<>(16);





【推荐】集合初始化时，指定集合初始值大小。

说明：HashMap 使用 HashMap(int initialCapacity)初始化。

正例：initialCapacity =(需要存储的元素个数/负载因子)+ 1。注意负载因子(即loader factor)默认 为0.75 ，如果暂时无法确定初始值大小，请设置为 16(即默认值)。

反例： HashMap 需要放置 1024 个元素，由于没有设置容量初始大小，随着元素不断增加，容量 7 次被 迫扩大， resize 需要重建 hash 表，严重影响性能



【推荐】==使用entrySet遍历Map类集合KV，而不是keySet方式进行遍历==,**如果是JDK8， 使用 Map.forEach 方法**

keySet其实是遍历了 2次，一次是转为Iterator对象，另一次是从hashMap中取出key所对应 的value

而entrySet只是遍历了一次就把key和value都放到了 entry中，效率高



【参考】合理利用好集合的有序性(sort)和稳定性(order)，避免集合的无序性(unsort)和不稳定性(unorder)带来的负面影响

说明：有序性是指遍历的结果是按某种比较规则依次排列的。稳定性指集合每次遍历的元素次序是一定的。如：ArrayList 是 order/unsort ； HashMap 是 unorder/unsort ； TreeSet 是 order/sort



【参考】利用Set元素唯一的特性，可以快速对一个集合进行去重操作，避免使用List的 contains 方法进行遍历、对比、去重操作。





























### 注释规约



【强制】类、类属性、类方法的注释必须使用 Javadoc 规范，使用/**内容*/格式，不得使用 // xxx 方式。

说明：在IDE编辑窗口中，Javadoc方式会提示相关注释,调用方法时，不进入方法即可悬浮提示方法、参数、返回值的意义，提高阅读效率



【强制】所有的抽象方法（包括接口中的方法）必须要用Javadoc注释、除了返回值、参数、 异常说明外，还必须指出该方法做什么事情，实现什么功能。一并说明对子类的实现要求，或者调用注意事项





【强制】所有的枚举类型字段必须有注释，说明每个数据项的用途



【推荐】与其“半吊子”英文来注释，不如用中文注释把问题说清楚。专有名词与关键字保持英文原文即可

反例："TCP连接超时"解释成"传输控制协议连接超时"，理解反而费脑筋



【推荐】==代码修改时，注释也要进行相应的修改==，尤其是参数、返回值、异常、核心逻 辑等的修改



【参考】谨慎注释掉代码。在上方详细说明，而不是简单地注释掉。如果无用，则删除

说明：代码被注释掉有两种可能性： 

1）后续会恢复此段代码逻辑。

2）永久不用,建议直接删掉（代码仓库已保存了历史代码）



【参考】对于注释的要求

1. 能够准确反映设计思想和代码逻辑
2. 能够描述业务含义，使别的程序员能够迅速了解到代码背后的信息。完全没有注释的大段代码对于阅读者形同天书，注释是给自己看的，即使隔很长时间，也能清晰理解当时的思路；注释也是给继任者看的，使其能够快速接替自己的工作。



【参考】好的命名、代码结构是自解释的，注释力求精简准确、表达到位。避免出现注释的一个极端：过多过滥的注释，代码的逻辑一旦修改，修改注释是相当大的负担。



【参考】特殊注释标记，注明标记人与标记时间。注意及时处理这些标记

1. 待办事宜（ TODO ） : （标记人，标记时间， ［ 预计处理时间 ］）		表示需要实现，但目前还未实现的功能
2. 错误，不能工作（FIXME）:（标记人，标记时间，［预计处理时间］）    标记某代码是错误的，而且不能工作，需要及时纠正的情况





### 控制语句



【强制】在高并发场景中，使用**区间代替等值**判断作为中断或退出的条件

反例：判断剩余奖品数量等于0时，终止发放奖品，但因为并发处理错误导致奖品数量瞬间变成了负数，导致活动无法终止



【参考】下列情形，需要进行参数校验：

1) 调用频次低的方法

2) 执行时间开销很大的方法。此情形中，参数校验时间几乎可以忽略不计，但如果因为参数错误导致中间执行回退，或者错误，那得不偿失

3) 需要极高稳定性和可用性的方法

4) 对外提供的开放接口，不管是RPC/API/HTTP接口

5) 敏感权限入口



【参考】下列情形，不需要进行参数校验：

1) 被循环调用的方法。但在方法说明里必须注明外部参数检查要求

2) 底层调用频度比较高的方法，参数错误不太可能到底层才会暴露问题。一般DAO层与Service层都在同一个应用中，部署在同一台服务器中，所以DAO的参数校验，可以省略

3）被声明成private只会被自己代码所调用的方法，如果能确定调用方法的代码传入参数已经做过检查或者肯定不会有问题，此时可以不校验参数













## 设计规约



\1. 【强制】存储方案和底层数据结构的设计获得评审一致通过，并沉淀成为文档。

说明：有缺陷的底层数据结构容易导致系统风险上升，可扩展性下降，重构成本也会因历史数据迁移和系 统平滑过渡而陡然增加，所以，存储方案和数据结构需要认真地进行设计和评审，生产环境提交执行后， 需要进行double check。

正例：评审内容包括存储介质选型、表结构设计能否满足技术方案、存取性能和存储空间能否满足业务发 展、表或字段之间的辩证关系、字段名称、字段类型、索引等；数据结构变更（如在原有表中新增字段） 也需要进行评审通过后上线。

\2. 【强制】在需求分析阶段，如果与系统交互的User超过一类并且相关的User Case超过5 个，使用用例图来表达更加清晰的结构化需求。

\3. 【强制】如果某个业务对象的状态超过3个，使用状态图来表达并且明确状态变化的各个触 发条件。

说明：状态图的核心是对象状态，首先明确对象有多少种状态，然后明确两两状态之间是否存在直接转换 关系，再明确触发状态转换的条件是什么。

正例：淘宝订单状态有已下单、待付款、已付款、待发货、已发货、已收货等。比如已下单与已收货这两 种状态之间是不可能有直接转换关系的。

\4. 【强制】如果系统中某个功能的调用链路上的涉及对象超过3个，使用时序图来表达并且明 确各调用环节的输入与输出。

说明：时序图反映了一系列对象间的交互与协作关系，清晰立体地反映系统的调用纵深链路。

\5. 【强制】如果系统中模型类超过5个，并且存在复杂的依赖关系，使用类图来表达并且明确 类之间的关系。

说明：类图像建筑领域的施工图，如果搭平房，可能不需要，但如果建造蚂蚁Z空间大楼，肯定需要详细 的施工图。

\6. 【强制】如果系统中超过2个对象之间存在协作关系，并且需要表示复杂的处理流程，使用 活动图来表示。

说明：活动图是流程图的扩展，增加了能够体现协作关系的对象泳道，支持表示并发等。

\7. 【推荐】需求分析与系统设计在考虑主干功能的同时，需要充分评估异常流程与业务边界。 反例：用户在淘宝付款过程中，银行扣款成功，发送给用户扣款成功短信，但是支付宝入款时由于断网演 练产生异常，淘宝订单页面依然显示未付款，导致用户投诉。

\8. 【推荐】类在设计与实现时要符合单一原则。

说明：单一原则最易理解却是最难实现的一条规则，随着系统演进，很多时候，忘记了类设计的初衷。

\9. 【推荐】谨慎使用继承的方式来进行扩展，优先使用聚合/组合的方式来实现。

说明：不得已使用继承的话，必须符合里氏代换原则，此原则说父类能够出现的地方子类一定能够出现， t匕如，"把钱交出来"，钱的子类美元、欧元、人民币等都可以出现。

\10. 【推荐】系统设计时，根据依赖倒置原则，尽量依赖抽象类与接口，有利于扩展与维护。

说明：低层次模块依赖于高层次模块的抽象，方便系统间的解耦。

\11. 【推荐】系统设计时，注意对扩展开放，对修改闭合。

说明：极端情况下，交付线上生产环境的代码都是不可修改的，同一业务域内的需求变化，通过模块或类 的扩展来实现。

\12. 【推荐】系统设计阶段，共性业务或公共行为抽取出来公共模块、公共配置、公共类、公共 方法等，避免出现重复代码或重复配置的情况。

说明：随着代码的重复次数不断增加，维护成本指数级上升。

\13. 【推荐】避免如下误解：敏捷开发=讲故事+编码+发布。

说明：敏捷开发是快速交付迭代可用的系统，省略多余的设计方案，摒弃传统的审批流程，但核心关键点 上的必要设计和文档沉淀是需要的。

反例：某团队为了业务快速发展，敏捷成了产品经理催进度的借口，系统中均是勉强能运行但像面条一样 的代码，可维护性和可扩展性极差，一年之后，不得不进行大规模重构，得不偿失。

\14. 【参考】系统设计主要目的是明确需求、理顺逻辑、后期维护，次要目的用于指导编码。 说明：避免为了设计而设计，系统设计文档有助于后期的系统维护和重构，所以设计结果需要进行分类归 档保存。

\15. 【参考】设计的本质就是识别和表达系统难点，找到系统的变化点，并隔离变化点。

说明：世间众多设计模式目的是相同的，即隔离系统变化点。

\16. 【参考】系统架构设计的目的：

•确定系统边界。确定系统在技术层面上的做与不做。

•确定系统内模块之间的关系。确定模块之间的依赖关系及模块的宏观输入与输出。

•确定指导后续设计与演化的原则。使后续的子系统或模块设计在规定的框架内继续演化。

•确定非功能性需求。非功能性需求是指安全性、可用性、可扩展性等。

\17. 【参考】在做无障碍产品设计时，需要考虑到：

•所有可交互的控件元素必须能被tab键聚焦，并且焦点I顺序需符合自然操作逻辑。

•用于登陆校验和请求拦截的验证码均需提供图形验证以外的其它方式。

•自定义的控件类型需明确交互方式。





Entity所有属性都使用包装类型，都没有默认值，对于不不能为空的字段，需要在构造方法里面进行初始化，数据库需要添加约束

















## 名词解释



POJO ( Plain Ordinary Java Object):在本手册中，POJO 专指只有 setter / getter / toString 的简单类，包括 DO/DTO/BO/VO 等。

GAV ( Groupld、Artifactctld、Version ) : Maven 坐标，是用来唯一标识 jar 包。

OOP ( Object Oriented Programming ):本手册泛指类、对象的编程处理方式。

ORM ( Object Relation Mapping ):对象关系映射，对象领域模型与底层数据之间的转换， 本文泛指iBATIS, mybatis等框架。

NPE (java.lang.NullPointerException ):空指针异常。

SOA ( Service-Oriented Architecture ):面向服务架构，它可以根据需求通过网络对松散耦合 的粗粒度应用组件进行分布式部署、组合和使用，有利于提升组件可重用性，可维护性。

OOM ( Out Of Memory ):源于 java.lang.OutOfMemoryError，当 JVM 没有足够的内存来 为对象分配空间并且垃圾回收器也无法回收空间时，系统出现的严重状况。

一方库：本工程内部子项目模块依赖的库(jar包)

二方库：公司内部发布到中央仓库，可供公司内部其它应用依赖的库(jar包)







## 程序员修炼之道



==注重实效==



### 重复



DRY原则	Don`t Repeat YourSelf 不要重复你自己

不要在系统各处对知识重复

重复将导致一处的修改,将需要记得修改其他处





#### 重复产生原因



**强加的重复**

含有重复信息的文档

糟糕的代码才需要很多注释,把低级的知识放在代码中,把注释保留给高级说明,否则每次改动都要修改注释

多平台各自需要自己的编程语言/库/开发环境





**无意的重复**

当涉及到多个互相依赖的数据元素,容易出现不规范数据



对于线段类,起点终点是必须的,但长度非必须

![](image.assets/image-20210110105002931.png)

可能会因为性能原因违反DRY法则(缓存数据避免重复计算),但可以通过局部化的方式,让DRY的违反不暴露给外界

![](image.assets/image-20210110110036652.png)





**无耐心的重复**

欲速则不达,为了省事拷贝代码,以后可能会损失更多时间

开发者的懒惰会造成问题



**开发者之间的重复**

最难被检测和处理,整个功能集都可能在重复,并且重复可能在几年内都不会被发现,从而导致维护问题

开发者需要主动的交流,或让某个团队成员担任项目资料管理员

让源代码树中制定一个中央区域,用于存放脚本

阅读他人的源码与文档,不管是非正式的还是代码复查



**Make it easy to reuse**	让复用变得容易

更重要的是营造一种环境,在其中能够轻松地找到复用的东西,**如果寻找起来麻烦,大家都不会去复用**







### 正交性

几何学中表示相交为直角的两条直线

计算技术中表示不相依赖/解耦,发生变化时不会影响其他事物



**非正交**

驾驶飞机,所有的控制输入都有**次级效应**,操作左操作杆需要补偿性地操作右操作杆,并踩右踏板

此时每一项的改变都会影响其他所有的控制





**Eliminate Effects Between Unrelated Things**	消除无关事物间的影响

**独立,内聚**		不要把知识分散在多个系统组件中



* 优点

**局部化**	开发/测试所需时间降低

促进复用	组件有明确具体的,良好定义的责任

组合	两个组件分别能做M和N件事,如果正交,在组合后能做M*N件事

降低风险	模块出现问题不会扩散至整个系统,新模块的替换也变得容易

​			针对组件的测试更容易设计

​			第三方组件的接口被隔离在局部,不会与特定的产品/平台捆绑在一起



#### 方式



**团队**

成员之间责任的重叠将使得成员对责任感到困惑,改动将需要整个团队开会

需要将团队责任划分到得到了良好定义的小组



**设计**

系统由一组相互协作的模块组成,模块的实现不依赖于其他模块的功能

有时这些组件被组织成多个层次,每层提供一级抽象,每层都只使用其下面一层提供的抽象,改动底层无需修改上层,降低模块间依赖失控的风险



**第三方**

引入第三方时思考是否会对现有产生影响,这使得能够轻易地更换供应商



**编码**

保持代码解耦,避免向其他模块暴露或依赖

避免使用全局数据(单例模式)

避免编写相似的函数(策略模式)



**测试**

组件之间的交互是形式化并且有限的,更多的测试可以在单个模块级进行,而无需集成测试





### 可撤销性



**There are no final decisions**	不存在最终决策



**灵活的架构**

代码/维持架构/部署/供应商集成灵活





### 曳光弹



曳光弹与常规弹药交错着装在弹药袋,会留下烟火的轨迹,曳光弹击中则弹药也击中

**曳光弹往往比费力计算更可取**,反馈是及时的,与弹药工作在同一环境,外部影响小



为了在代码中获得同样的效果,需要能够快速直观,可重复地从需求出发,满足需求

在曳光代码中保留着任何一段产品都有的错误检查,结构,文档,它只是功能不全而已,但当各组件之间实现了端到端的连接,增加功能就变得非常容易,所以**曳光代码无需丢弃**



* 优点

快速交付,用户能够尽早看到,提前演示

提前构建结构

更容易感知工作进度





**曳光 VS 原型**

原型在对概念进行试验后,就进行了丢弃,而曳光则贯穿了开发流程

原型制作生成用过就扔的代码,曳光代码虽然简约,但是完整





### 调试



bug报告的准确性会在经第三方之手时进一步降低	需要观察报告bug用户的操作



复现bug

数据可视化

断点

向别人解释代码过程

当bug是由脏数据导致的,检查能否通过参数检查更早地隔离它







# Web 页面请求过程

### 1. DHCP 配置主机信息

- 假设主机最开始没有 IP 地址以及其它信息，那么就需要先使用 DHCP 来获取。

- 主机生成一个 DHCP 请求报文，并将这个报文放入具有目的端口 67 和源端口 68 的 UDP 报文段中。

- 该报文段则被放入在一个具有广播 IP 目的地址(255.255.255.255) 和源 IP 地址（0.0.0.0）的 IP 数据报中。

- 该数据报则被放置在 MAC 帧中，该帧具有目的地址 FF:\<zero-width space\>FF:\<zero-width space\>FF:\<zero-width space\>FF:\<zero-width space\>FF:FF，将广播到与交换机连接的所有设备。

- 连接在交换机的 DHCP 服务器收到广播帧之后，不断地向上分解得到 IP 数据报、UDP 报文段、DHCP 请求报文，之后生成 DHCP ACK 报文，该报文包含以下信息：IP 地址、DNS 服务器的 IP 地址、默认网关路由器的 IP 地址和子网掩码。该报文被放入 UDP 报文段中，UDP 报文段有被放入 IP 数据报中，最后放入 MAC 帧中。

- 该帧的目的地址是请求主机的 MAC 地址，因为交换机具有自学习能力，之前主机发送了广播帧之后就记录了 MAC 地址到其转发接口的交换表项，因此现在交换机就可以直接知道应该向哪个接口发送该帧。

- 主机收到该帧后，不断分解得到 DHCP 报文。之后就配置它的 IP 地址、子网掩码和 DNS 服务器的 IP 地址，并在其 IP 转发表中安装默认网关。

### 2. ARP 解析 MAC 地址

- 主机通过浏览器生成一个 TCP 套接字，套接字向 HTTP 服务器发送 HTTP 请求。为了生成该套接字，主机需要知道网站的域名对应的 IP 地址。

- 主机生成一个 DNS 查询报文，该报文具有 53 号端口，因为 DNS 服务器的端口号是 53。

- 该 DNS 查询报文被放入目的地址为 DNS 服务器 IP 地址的 IP 数据报中。

- 该 IP 数据报被放入一个以太网帧中，该帧将发送到网关路由器。

- DHCP 过程只知道网关路由器的 IP 地址，为了获取网关路由器的 MAC 地址，需要使用 ARP 协议。

- 主机生成一个包含目的地址为网关路由器 IP 地址的 ARP 查询报文，将该 ARP 查询报文放入一个具有广播目的地址（FF:\<zero-width space\>FF:\<zero-width space\>FF:\<zero-width space\>FF:\<zero-width space\>FF:FF）的以太网帧中，并向交换机发送该以太网帧，交换机将该帧转发给所有的连接设备，包括网关路由器。

- 网关路由器接收到该帧后，不断向上分解得到 ARP 报文，发现其中的 IP 地址与其接口的 IP 地址匹配，因此就发送一个 ARP 回答报文，包含了它的 MAC 地址，发回给主机。

### 3. DNS 解析域名

- 知道了网关路由器的 MAC 地址之后，就可以继续 DNS 的解析过程了。

- 网关路由器接收到包含 DNS 查询报文的以太网帧后，抽取出 IP 数据报，并根据转发表决定该 IP 数据报应该转发的路由器。

- 因为路由器具有内部网关协议（RIP、OSPF）和外部网关协议（BGP）这两种路由选择协议，因此路由表中已经配置了网关路由器到达 DNS 服务器的路由表项。

- 到达 DNS 服务器之后，DNS 服务器抽取出 DNS 查询报文，并在 DNS 数据库中查找待解析的域名。

- 找到 DNS 记录之后，发送 DNS 回答报文，将该回答报文放入 UDP 报文段中，然后放入 IP 数据报中，通过路由器反向转发回网关路由器，并经过以太网交换机到达主机。

### 4. HTTP 请求页面

- 有了 HTTP 服务器的 IP 地址之后，主机就能够生成 TCP 套接字，该套接字将用于向 Web 服务器发送 HTTP GET 报文。

- 在生成 TCP 套接字之前，必须先与 HTTP 服务器进行三次握手来建立连接。生成一个具有目的端口 80 的 TCP SYN 报文段，并向 HTTP 服务器发送该报文段。

- HTTP 服务器收到该报文段之后，生成 TCP SYN ACK 报文段，发回给主机。

- 连接建立之后，浏览器生成 HTTP GET 报文，并交付给 HTTP 服务器。

- HTTP 服务器从 TCP 套接字读取 HTTP GET 报文，生成一个 HTTP 响应报文，将 Web 页面内容放入报文主体中，发回给主机。

- 浏览器收到 HTTP 响应报文后，抽取出 Web 页面内容，之后进行渲染，显示 Web 页面







































# 常用注解



pojo是一个统称，可以是DTO、可以是VO、可以是PO、可以是domain，这些都叫pojo
 po、do、domain三者区别很小，用处都是和数据库进行对应

DTO是数据传输对象，简单点说就是传参数用的。比如一张表30个字段，但是传参的时候只需要传5个字段，这个时候使用dto，可以避免过多的无用数据，也可以隐藏后端表结构，往往是前端传参给后端、controller、service、dao三层之间传递使用。
 VO就是view Object，专门负责给前端展示数据。VO是业务对象，业务上需要什么字段它就给什么字段，比如上面说的学生表，在给前端展示的时候需要展示名称：class_name这个字段，那么vo里写的就不是classId，而是className或者班级类

 

do、vo、dto这些类之间不需要给继承关系，从设计思想来讲也不能给继承关系。互相之间的属性复制使用spring提供的BeanUtils.copyProperties。如果是集合这种数据较多的属性复制，就先转成json字符串再转成另一个类的List

 

@Deprecated，用来表示某个类或属性或方法已经过时

@SuppressWarnings用来压制程序中出来的警告，比如在没有用泛型或是方法已经过时的时候





## Spring注解



@Component:标准一个普通的spring Bean类
@Controller:标注一个控制器组件类
@Service:标注一个业务逻辑组件类
@Repository:标注一个DAO组件类



Bean实例的名称默认是Bean类的首字母小写，其他部分不变



@Resource



**定制spring容器中bean的生命周期行为**

@PostConstruct	bean的初始化之前的方法

@PreDestory		bean销毁之前的方法









## mvc



```
@Controller		负责处理由DispatcherServlet 分发的请求,把用户请求的数据经过业务处理层处理之后封装成一个Model ，然后再把该Model 返回给对应的View 进行展示
标记的类就是一个SpringMVC Controller 对象。分发处理器将会扫描使用了该注解的类的方法，并检测该方法是否使用了@RequestMapping 注解。@Controller 只是定义了一个控制器类，而使用@RequestMapping 注解的方法才是真正处理请求的处理器。单单使用@Controller 标记在一个类上还不能真正意义上的说它就是SpringMVC 的一个控制器类，因为这个时候Spring 还不认识它。那么要如何做Spring 才能认识它呢？这个时候就需要我们把这个控制器类交给Spring 来管理。有两种方式：








```











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

 





# 关键词



## assert 断言



在软件开发中是一种常用的调试方式，很多开发语言中都支持这种机制。一般来说，assertion 用于保证程序最基本、关键的正确性。assertion 检查通常在开发和测试时开启。为了提高性能，在软件发布后， assertion 检查通常是关闭的。在实现中，断言是一个包含布尔表达式的语句，在执行这个语句时假定该表达式为 true；如果表达式计算为 false，那么系统会报告一个 AssertionError。

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

启用或者禁用断言。可以在预计正常情况下不会到达的任何位置上放置断言

断言可以用于验证传递给私有方法的参数。不过，断言不应该用于验证传递

给公有方法的参数，因为不管是否启用了断言，公有方法都必须检查其参数

不过，既可以在公有方法中，也可以在非公有方法中利用断言测试后置条件





 

# 日期格式处理



```
@JsonFormat           后台到前台
properties文件中有相同的配置spring.mvc.date-format

@DateTimeFormat     前台到后台
properties文件中有相同的配置spring.jackson.date-format

spring.jackson.time-zone
```



 

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

 

@TableField(exist = false)	实体类中,数据库不存在的字段需要加上这个注解

 

@Transactional 	**public 的方法才起作用**

1)事务开始时，通过AOP机制，**生成代理connection对象**，并将其放入DataSource实例的某个与DataSourceTransactionManager相关的容器中。客户代码使用该connection连接数据库，执行所有数据库命令

2)事务结束时，回滚代理connection对象上执行的数据库命令，然后关闭该代理connection对象（事务结束后，回滚操作不会对已执行完毕的SQL操作命令起作用）













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



## 日志



五个级别

错误级别(ERROR) －－指系统发生了严重的问题，系统无法自行恢复，需要立刻调查，例如: NPE, 数据库不可用等
警告级别(WARN) －－指系统可以继续运行，但是存在潜在风险，一般而言，高可靠的系统应该具备平滑处理警告事件的能力。警告日志例子包括，接收到错误参数而改用默认值，达到运行最大线程数而抛弃当前
信息级别(INFO) －－重要信息点，这些信息对于问题定位、数据分析应该提供重要帮助。例如：定期启动的任务事件
调试级别(DEBUG) －－ 系统运行的详细日志，包括参数值的打印
跟踪级别(TRACE) －－更加详细的日志，一般而言，用于客户端产品的收集



























## 创建指定大小文件

在目录下进入cmd

fsutil file createnew test.txt 字节数



## adb命令



adb logcat -c
adb logcat > C:\Users\howlett\Desktop\a.txt







# 正则



|     字符     |                             描述                             |
| :----------: | :----------------------------------------------------------: |
|      \       | 将下一个字符标记为一个特殊字符、或一个原义字符、或一个向后引用、或一个八进制转义符。例如，“`n`”匹配字符“`n`”。“`\n`”匹配一个换行符。串行“`\\`”匹配“`\`”而“`\(`”则匹配“`(`”。 |
|      ^       | 匹配输入字符串的开始位置。如果设置了RegExp对象的Multiline属性，^也匹配“`\n`”或“`\r`”之后的位置。 |
|      $       | 匹配输入字符串的结束位置。如果设置了RegExp对象的Multiline属性，$也匹配“`\n`”或“`\r`”之前的位置。 |
|      *       | 匹配前面的子表达式零次或多次。例如，zo*能匹配“`z`”以及“`zoo`”。*等价于{0,}。 |
|      +       | 匹配前面的子表达式一次或多次。例如，“`zo+`”能匹配“`zo`”以及“`zoo`”，但不能匹配“`z`”。+等价于{1,}。 |
|      ?       | 匹配前面的子表达式零次或一次。例如，“`do(es)?`”可以匹配“`does`”或“`does`”中的“`do`”。?等价于{0,1}。 |
|    {*n*}     | *n*是一个非负整数。匹配确定的*n*次。例如，“`o{2}`”不能匹配“`Bob`”中的“`o`”，但是能匹配“`food`”中的两个o。 |
|    {*n*,}    | *n*是一个非负整数。至少匹配*n*次。例如，“`o{2,}`”不能匹配“`Bob`”中的“`o`”，但能匹配“`foooood`”中的所有o。“`o{1,}`”等价于“`o+`”。“`o{0,}`”则等价于“`o*`”。 |
|  {*n*,*m*}   | *m*和*n*均为非负整数，其中*n*<=*m*。最少匹配*n*次且最多匹配*m*次。例如，“`o{1,3}`”将匹配“`fooooood`”中的前三个o。“`o{0,1}`”等价于“`o?`”。请注意在逗号和两个数之间不能有空格。 |
|      ?       | 当该字符紧跟在任何一个其他限制符（*,+,?，{*n*}，{*n*,}，{*n*,*m*}）后面时，匹配模式是非贪婪的。非贪婪模式尽可能少的匹配所搜索的字符串，而默认的贪婪模式则尽可能多的匹配所搜索的字符串。例如，对于字符串“`oooo`”，“`o+?`”将匹配单个“`o`”，而“`o+`”将匹配所有“`o`”。 |
|      .       | 匹配除“`\`*`n`*”之外的任何单个字符。要匹配包括“`\`*`n`*”在内的任何字符，请使用像“`(.|\n)`”的模式。 |
|  (pattern)   | 匹配pattern并获取这一匹配。所获取的匹配可以从产生的Matches集合得到，在VBScript中使用SubMatches集合，在JScript中则使用$0…$9属性。要匹配圆括号字符，请使用“`\(`”或“`\)`”。 |
| (?:pattern)  | 匹配pattern但不获取匹配结果，也就是说这是一个非获取匹配，不进行存储供以后使用。这在使用或字符“`(|)`”来组合一个模式的各个部分是很有用。例如“`industr(?:y|ies)`”就是一个比“`industry|industries`”更简略的表达式。 |
| (?=pattern)  | 正向肯定预查，在任何匹配pattern的字符串开始处匹配查找字符串。这是一个非获取匹配，也就是说，该匹配不需要获取供以后使用。例如，“`Windows(?=95|98|NT|2000)`”能匹配“`Windows2000`”中的“`Windows`”，但不能匹配“`Windows3.1`”中的“`Windows`”。预查不消耗字符，也就是说，在一个匹配发生后，在最后一次匹配之后立即开始下一次匹配的搜索，而不是从包含预查的字符之后开始。 |
| (?!pattern)  | 正向否定预查，在任何不匹配pattern的字符串开始处匹配查找字符串。这是一个非获取匹配，也就是说，该匹配不需要获取供以后使用。例如“`Windows(?!95|98|NT|2000)`”能匹配“`Windows3.1`”中的“`Windows`”，但不能匹配“`Windows2000`”中的“`Windows`”。预查不消耗字符，也就是说，在一个匹配发生后，在最后一次匹配之后立即开始下一次匹配的搜索，而不是从包含预查的字符之后开始 |
| (?<=pattern) | 反向肯定预查，与正向肯定预查类拟，只是方向相反。例如，“`(?<=95|98|NT|2000)Windows`”能匹配“`2000Windows`”中的“`Windows`”，但不能匹配“`3.1Windows`”中的“`Windows`”。 |
| (?<!pattern) | 反向否定预查，与正向否定预查类拟，只是方向相反。例如“`(?<!95|98|NT|2000)Windows`”能匹配“`3.1Windows`”中的“`Windows`”，但不能匹配“`2000Windows`”中的“`Windows`”。 |
|     x\|y     | 匹配x或y。例如，“`z|food`”能匹配“`z`”或“`food`”。“`(z|f)ood`”则匹配“`zood`”或“`food`”。 |
|    [xyz]     | 字符集合。匹配所包含的任意一个字符。例如，“`[abc]`”可以匹配“`plain`”中的“`a`”。 |
|    [^xyz]    | 负值字符集合。匹配未包含的任意字符。例如，“`[^abc]`”可以匹配“`plain`”中的“`p`”。 |
|    [a-z]     | 字符范围。匹配指定范围内的任意字符。例如，“`[a-z]`”可以匹配“`a`”到“`z`”范围内的任意小写字母字符。 |
|    [^a-z]    | 负值字符范围。匹配任何不在指定范围内的任意字符。例如，“`[^a-z]`”可以匹配任何不在“`a`”到“`z`”范围内的任意字符。 |
|      \b      | 匹配一个单词边界，也就是指单词和空格间的位置。例如，“`er\b`”可以匹配“`never`”中的“`er`”，但不能匹配“`verb`”中的“`er`”。 |
|      \B      | 匹配非单词边界。“`er\B`”能匹配“`verb`”中的“`er`”，但不能匹配“`never`”中的“`er`”。 |
|     \cx      | 匹配由x指明的控制字符。例如，\cM匹配一个Control-M或回车符。x的值必须为A-Z或a-z之一。否则，将c视为一个原义的“`c`”字符。 |
|      \d      |               匹配一个数字字符。等价于[0-9]。                |
|      \D      |              匹配一个非数字字符。等价于[^0-9]。              |
|      \f      |              匹配一个换页符。等价于\x0c和\cL。               |
|      \n      |              匹配一个换行符。等价于\x0a和\cJ。               |
|      \r      |              匹配一个回车符。等价于\x0d和\cM。               |
|      \s      | 匹配任何空白字符，包括空格、制表符、换页符等等。等价于[ \f\n\r\t\v]。 |
|      \S      |          匹配任何非空白字符。等价于[^ \f\n\r\t\v]。          |
|      \t      |              匹配一个制表符。等价于\x09和\cI。               |
|      \v      |            匹配一个垂直制表符。等价于\x0b和\cK。             |
|      \w      |    匹配包括下划线的任何单词字符。等价于“`[A-Za-z0-9_]`”。    |
|      \W      |        匹配任何非单词字符。等价于“`[^A-Za-z0-9_]`”。         |
|    \x*n*     | 匹配*n*，其中*n*为十六进制转义值。十六进制转义值必须为确定的两个数字长。例如，“`\x41`”匹配“`A`”。“`\x041`”则等价于“`\x04&1`”。正则表达式中可以使用ASCII编码。. |
|    \*num*    | 匹配*num*，其中*num*是一个正整数。对所获取的匹配的引用。例如，“`(.)\1`”匹配两个连续的相同字符。 |
|     \*n*     | 标识一个八进制转义值或一个向后引用。如果\*n*之前至少*n*个获取的子表达式，则*n*为向后引用。否则，如果*n*为八进制数字（0-7），则*n*为一个八进制转义值。 |
|    \*nm*     | 标识一个八进制转义值或一个向后引用。如果\*nm*之前至少有*nm*个获得子表达式，则*nm*为向后引用。如果\*nm*之前至少有*n*个获取，则*n*为一个后跟文字*m*的向后引用。如果前面的条件都不满足，若*n*和*m*均为八进制数字（0-7），则\*nm*将匹配八进制转义值*nm*。 |
|    \*nml*    | 如果*n*为八进制数字（0-3），且*m和l*均为八进制数字（0-7），则匹配八进制转义值*nm*l。 |
|    \u*n*     | 匹配*n*，其中*n*是一个用四个十六进制数字表示的Unicode字符。例如，\u00A9匹配版权符号（©）。 |



|         用户名          | /^[a-z0-9_-]{3,16}$/                                         |
| :---------------------: | ------------------------------------------------------------ |
|          密码           | /^[a-z0-9_-]{6,18}$/                                         |
|       十六进制值        | /^#?([a-f0-9]{6}\|[a-f0-9]{3})$/                             |
|        电子邮箱         | /^([a-z0-9_\.-]+)@([\da-z\.-]+)\.([a-z\.]{2,6})$/ /^[a-z\d]+(\.[a-z\d]+)*@([\da-z](-[\da-z])?)+(\.{1,2}[a-z]+)+$/ |
|           URL           | /^(https?:\/\/)?([\da-z\.-]+)\.([a-z\.]{2,6})([\/\w \.-]*)*\/?$/ |
|         IP 地址         | /((2[0-4]\d\|25[0-5]\|[01]?\d\d?)\.){3}(2[0-4]\d\|25[0-5]\|[01]?\d\d?)/ /^(?:(?:25[0-5]\|2[0-4][0-9]\|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]\|2[0-4][0-9]\|[01]?[0-9][0-9]?)$/ |
|        HTML 标签        | /^<([a-z]+)([^<]+)*(?:>(.*)<\/\1>\|\s+\/>)$/                 |
|     删除代码\\注释      | (?<!http:\|\S)//.*$                                          |
| Unicode编码中的汉字范围 | /^[\u2E80-\u9FFF]+$/                                         |







## 二、匹配单个字符

**.**   可以用来匹配任何的单个字符，但是在绝大多数实现里面，不能匹配换行符；

**.**   是元字符，表示它有特殊的含义，而不是字符本身的含义。如果需要匹配 . ，那么要用 \ 进行转义，即在 . 前面加上 \ 。

正则表达式一般是区分大小写的，但也有些实现不区分。

**正则表达式**  

```
C.C2018
```

**匹配结果**  

My name is   **CyC2018**  .

## 三、匹配一组字符

**[ ]**   定义一个字符集合；

0-9、a-z 定义了一个字符区间，区间使用 ASCII 码来确定，字符区间在 [ ] 中使用。

**-**   只有在 [ ] 之间才是元字符，在 [ ] 之外就是一个普通字符；

**^**   在 [ ] 中是取非操作。

**应用**  

匹配以 abc 为开头，并且最后一个字母不为数字的字符串：

**正则表达式**  

```
abc[^0-9]
```

**匹配结果**  

1.   **abcd**  
2.   abc1
3.   abc2

## 四、使用元字符

### 匹配空白字符

| 元字符 |         说明         |
| :----: | :------------------: |
|  [\b]  | 回退（删除）一个字符 |
|   \f   |        换页符        |
|   \n   |        换行符        |
|   \r   |        回车符        |
|   \t   |        制表符        |
|   \v   |      垂直制表符      |

\r\n 是 Windows 中的文本行结束标签，在 Unix/Linux 则是 \n。

\r\n\r\n 可以匹配 Windows 下的空白行，因为它匹配两个连续的行尾标签，而这正是两条记录之间的空白行；

### 匹配特定的字符

#### 1. 数字元字符

| 元字符 |           说明            |
| :----: | :-----------------------: |
|   \d   |  数字字符，等价于 [0-9]   |
|   \D   | 非数字字符，等价于 [^0-9] |

#### 2. 字母数字元字符

| 元字符 |                      说明                      |
| :----: | :--------------------------------------------: |
|   \w   | 大小写字母，下划线和数字，等价于 [a-zA-Z0-9\_] |
|   \W   |                   对 \w 取非                   |

#### 3. 空白字符元字符

| 元字符 |                 说明                  |
| :----: | :-----------------------------------: |
|   \s   | 任何一个空白字符，等价于 [\f\n\r\t\v] |
|   \S   |              对 \s 取非               |

\x 匹配十六进制字符，\0 匹配八进制，例如 \xA 对应值为 10 的 ASCII 字符 ，即 \n。

## 五、重复匹配

-   **\+**   匹配 1 个或者多个字符
-   **\**  * 匹配 0 个或者多个字符
-   **?**   匹配 0 个或者 1 个字符

**应用**  

匹配邮箱地址。

**正则表达式**  

```
[\w.]+@\w+\.\w+
```

[\w.] 匹配的是字母数字或者 . ，在其后面加上 + ，表示匹配多次。在字符集合 [ ] 里，. 不是元字符；

**匹配结果**  

**abc.def\<span\>@\</span\>qq.com**  

-   **{n}**   匹配 n 个字符
-   **{m,n}**   匹配 m\~n 个字符
-   **{m,}**   至少匹配 m 个字符

\* 和 + 都是贪婪型元字符，会匹配尽可能多的内容。在后面加 ? 可以转换为懒惰型元字符，例如 \*?、+? 和 {m,n}? 。

**正则表达式**  

```
a.+c
```

**匹配结果**  

**abcabcabc**  

由于 + 是贪婪型的，因此 .+ 会匹配更可能多的内容，所以会把整个 abcabcabc 文本都匹配，而不是只匹配前面的 abc 文本。用懒惰型可以实现匹配前面的。

## 六、位置匹配

### 单词边界

**\b**   可以匹配一个单词的边界，边界是指位于 \w 和 \W 之间的位置；**\B** 匹配一个不是单词边界的位置。

\b 只匹配位置，不匹配字符，因此 \babc\b 匹配出来的结果为 3 个字符。

### 字符串边界

**^**   匹配整个字符串的开头，**$** 匹配结尾。

^ 元字符在字符集合中用作求非，在字符集合外用作匹配字符串的开头。

分行匹配模式（multiline）下，换行被当做字符串的边界。

**应用**  

匹配代码中以 // 开始的注释行

**正则表达式**  

```
^\s*\/\/.*$
```



## 七、使用子表达式

使用   **( )**   定义一个子表达式。子表达式的内容可以当成一个独立元素，即可以将它看成一个字符，并且使用 * 等元字符。

子表达式可以嵌套，但是嵌套层次过深会变得很难理解。

**正则表达式**  

```
(ab){2,}
```

**匹配结果**  

**ababab**  

**|**   是或元字符，它把左边和右边所有的部分都看成单独的两个部分，两个部分只要有一个匹配就行。

**正则表达式**  

```
(19|20)\d{2}
```

**匹配结果**  

1.   **1900**  
2.   **2010**  
3.   1020

**应用**  

匹配 IP 地址。

IP 地址中每部分都是 0-255 的数字，用正则表达式匹配时以下情况是合法的：

- 一位数字
- 不以 0 开头的两位数字
- 1 开头的三位数
- 2 开头，第 2 位是 0-4 的三位数
- 25 开头，第 3 位是 0-5 的三位数

**正则表达式**  

```
((25[0-5]|(2[0-4]\d)|(1\d{2})|([1-9]\d)|(\d))\.){3}(25[0-5]|(2[0-4]\d)|(1\d{2})|([1-9]\d)|(\d))
```

**匹配结果**  

1.   **192.168.0.1**  
2.   00.00.00.00
3.   555.555.555.555

## 八、回溯引用

回溯引用使用   **\n**   来引用某个子表达式，其中 n 代表的是子表达式的序号，从 1 开始。它和子表达式匹配的内容一致，比如子表达式匹配到 abc，那么回溯引用部分也需要匹配 abc 。

**应用**  

匹配 HTML 中合法的标题元素。

**正则表达式**  

\1 将回溯引用子表达式 (h[1-6]) 匹配的内容，也就是说必须和子表达式匹配的内容一致。

```
<(h[1-6])>\w*?<\/\1>
```

**匹配结果**  

1.   **&lt;h1\>x&lt;/h1\>**  
2.   **&lt;h2\>x&lt;/h2\>**  
3.   &lt;h3\>x&lt;/h1\>

### 替换

需要用到两个正则表达式。

**应用**  

修改电话号码格式。

**文本**  

313-555-1234

**查找正则表达式**  

```
(\d{3})(-)(\d{3})(-)(\d{4})
```

**替换正则表达式**  

在第一个子表达式查找的结果加上 () ，然后加一个空格，在第三个和第五个字表达式查找的结果中间加上 - 进行分隔。

```
($1) $3-$5
```

**结果**  

(313) 555-1234

### 大小写转换

| 元字符 |                说明                |
| :----: | :--------------------------------: |
|   \l   |        把下个字符转换为小写        |
|   \u   |        把下个字符转换为大写        |
|   \L   | 把\L 和\E 之间的字符全部转换为小写 |
|   \U   | 把\U 和\E 之间的字符全部转换为大写 |
|   \E   |           结束\L 或者\U            |

**应用**  

把文本的第二个和第三个字符转换为大写。

**文本**  

abcd

**查找**  

```
(\w)(\w{2})(\w)
```

**替换**  

```
$1\U$2\E$3
```

**结果**  

aBCd

## 九、前后查找

前后查找规定了匹配的内容首尾应该匹配的内容，但是又不包含首尾匹配的内容。

向前查找使用   **?=**   定义，它规定了尾部匹配的内容，这个匹配的内容在 ?= 之后定义。所谓向前查找，就是规定了一个匹配的内容，然后以这个内容为尾部向前面查找需要匹配的内容。向后匹配用 ?\<= 定义（注: JavaScript 不支持向后匹配，Java 对其支持也不完善）。

**应用**  

查找出邮件地址 @ 字符前面的部分。

**正则表达式**  

```
\w+(?=@)
```

**结果**  

**abc**  @qq.com

对向前和向后查找取非，只要把 = 替换成 ! 即可，比如 (?=) 替换成 (?!) 。取非操作使得匹配那些首尾不符合要求的内容。

## 十、嵌入条件

### 回溯引用条件

条件为某个子表达式是否匹配，如果匹配则需要继续匹配条件表达式后面的内容。

**正则表达式**  

子表达式 (\\() 匹配一个左括号，其后的 ? 表示匹配 0 个或者 1 个。 ?(1) 为条件，当子表达式 1 匹配时条件成立，需要执行 \) 匹配，也就是匹配右括号。

```
(\()?abc(?(1)\))
```

**结果**  

1.   **(abc)**  
2.   **abc**  
3.   (abc

### 前后查找条件

条件为定义的首尾是否匹配，如果匹配，则继续执行后面的匹配。注意，首尾不包含在匹配的内容中。

**正则表达式**  

 ?(?=-) 为前向查找条件，只有在以 - 为前向查找的结尾能匹配 \d{5} ，才继续匹配 -\d{4} 。

```
\d{5}(?(?=-)-\d{4})
```

**结果**  

1.   **11111**  
2.   22222-
3.   **33333-4444**  











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



 







# Session



### 两种实现方法



#### Cookie实现Session



服务器为客户端创建并维护Session对象，用于存放数据。同时会产生SessionID，服务器以Cookie的方式将SessionID存放在客户端。，此时的Cookie中仅仅保存了一个SessionID，而相对较多的会话数据保存在服务器端对应的Session对象中，由服务器来统一维护，这样一定程度保证了会话数据安全性，但**增加了服务器端的内存开销**

Cookie会在浏览器关闭时清除,称为一个“会话”。

一个“会话”中的多次请求，共享一个Session对象，携带了相同的SessionID



#### URL重写



Session对象的正常使用要依赖于Cookie。

但客户端出于安全的考虑禁用了Cookie，需要用URL重写的方式使Session在客户端禁用Cookie的情况下继续生效



### session VS cookie



存储角度：

Session是服务器端的数据存储技术，cookie是客户端的数据存储技术

解决问题角度：

Session	同一用户不同请求的数据共享

cookie	不同用户不同请求的数据的共享

生命周期角度：

Session的id	依赖cookie存储

Cookie	可以单独设置其在浏览器的存储时间





### Ajax 的工作原理

异步的javascript和xml,通过XmlHttpRequest对象来向服务器发异步请求，从服务器获得数据，然后用javascript来操作DOM而更新页面。从而实现向服务器提出请求和处理响应，而不阻塞用户。达到无刷新的效果



### JSON 及其作用



轻量级的数据交换格式，采用完全**独立于语言**的格式。也是 JavaScript 原生格式，这意味着在 JavaScript 中处理 JSON 数据不须要任何特殊的 API 或工具包。

在 JSON 中，有两种结构：对象和数组。

 {} 对象

 [] 数组

 , 分隔属性

 : 左边为属性名，右边为属性值

属性名可用可不用引号括起，属性值为字符串一定要用引号括起



## Servlet



### Servlet生命周期

Web容器加载Servlet并将其实例化后，Servlet生命周期开始，容器运行init()方法进行Servlet的初始化；

请求到达时调用service()，service()调用对应的doGet/Post

当服务器关闭或项目被卸载时Servlet 实例被销毁，此时会调用destroy()



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

 



## IOC/AOP



Ioc—Inversion of Control，控制反转，是一种设计思想

将设计好的对象交给容器控制，并非在对象内部直接控制



**谁控制谁，控制什么：**而IoC是有专门容器来创建对象，即Ioc容器控制对象创建,依赖对象的获取被反转了





松耦合

主动创建依赖对象，从而导致类与类之间高耦合，难测试

IoC容器后，把创建和查找依赖对象的控制权交给容器，由容器注入组合对象，所以对象与对象之间松耦合，更方便测试，利于复用，使得程序的整个体系结构灵活

**IoC很好的体现迪米特法则；即由IoC容器帮对象找相应的依赖对象并注入，而不是由对象主动去找。**



### 1.3、IoC和DI

　　**DI—Dependency Injection，即“依赖注入”**：**组件之间依赖关系**由容器在运行期决定，形象的说，即**由容器动态的将某个依赖关系注入到组件之中**。**依赖注入的目的并非为软件系统带来更多功能，而是为了提升组件重用的频率，并为系统搭建一个灵活、可扩展的平台。**通过依赖注入机制，我们只需要通过简单的配置，而无需任何代码就可指定目标需要的资源，完成自身的业务逻辑，而不需要关心具体的资源来自何处，由谁实现。

　　理解DI的关键是：“谁依赖谁，为什么需要依赖，谁注入谁，注入了什么”，那我们来深入分析一下：

　　●**谁依赖于谁：**当然是**应用程序依赖于IoC容器**；

　　●**为什么需要依赖：****应用程序需要IoC容器来提供对象需要的外部资源**；

　　●**谁注入谁：**很明显是**IoC容器注入应用程序某个对象，应用程序依赖的对象**；

　　**●注入了什么：**就是**注入某个对象所需要的外部资源（包括对象、资源、常量数据）**。

　　**IoC和DI**由什么**关系**呢？其实它们**是同一个概念的不同角度描述**，由于控制反转概念比较含糊（可能只是理解为容器控制对象这一个层面，很难让人想到谁来维护对象关系），所以2004年大师级人物Martin Fowler又给出了一个新的名字：“依赖注入”，相对IoC 而言，**“****依赖注入”****明确描述了“被注入对象依赖IoC****容器配置依赖对象”。**







### IoC(控制反转)

　　首先想说说**IoC（Inversion of Control，控制反转）**。这是**spring的核心**，贯穿始终。**所谓IoC，对于spring框架来说，就是由spring来负责控制对象的生命周期和对象间的关系。**这是什么意思呢，举个简单的例子，我们是如何找女朋友的？常见的情况是，我们到处去看哪里有长得漂亮身材又好的mm，然后打听她们的兴趣爱好、qq号、电话号、ip号、iq号………，想办法认识她们，投其所好送其所要，然后嘿嘿……这个过程是复杂深奥的，我们必须自己设计和面对每个环节。传统的程序开发也是如此，在一个对象中，如果要使用另外的对象，就必须得到它（自己new一个，或者从JNDI中查询一个），使用完之后还要将对象销毁（比如Connection等），对象始终会和其他的接口或类藕合起来。

　　那么IoC是如何做的呢？有点像通过婚介找女朋友，在我和女朋友之间引入了一个第三者：婚姻介绍所。婚介管理了很多男男女女的资料，我可以向婚介提出一个列表，告诉它我想找个什么样的女朋友，比如长得像李嘉欣，身材像林熙雷，唱歌像周杰伦，速度像卡洛斯，技术像齐达内之类的，然后婚介就会按照我们的要求，提供一个mm，我们只需要去和她谈恋爱、结婚就行了。简单明了，如果婚介给我们的人选不符合要求，我们就会抛出异常。整个过程不再由我自己控制，而是有婚介这样一个类似容器的机构来控制。**Spring所倡导的开发方式**就是如此，**所有的类都会在spring容器中登记，告诉spring你是个什么东西，你需要什么东西，然后spring会在系统运行到适当的时候，把你要的东西主动给你，同时也把你交给其他需要你的东西。所有的类的创建、销毁都由 spring来控制，也就是说控制对象生存周期的不再是引用它的对象，而是spring。对于某个具体的对象而言，以前是它控制其他对象，现在是所有对象都被spring控制，所以这叫控制反转。**

### 2.2、DI(依赖注入)

　　**IoC的一个重点是在系统运行中，动态的向某个对象提供它所需要的其他对象。这一点是通过DI（Dependency Injection，依赖注入）来实现的**。比如对象A需要操作数据库，以前我们总是要在A中自己编写代码来获得一个Connection对象，有了 spring我们就只需要告诉spring，A中需要一个Connection，至于这个Connection怎么构造，何时构造，A不需要知道。在系统运行时，spring会在适当的时候制造一个Connection，然后像打针一样，注射到A当中，这样就完成了对各个对象之间关系的控制。A需要依赖 Connection才能正常运行，而这个Connection是由spring注入到A中的，依赖注入的名字就这么来的。那么DI是如何实现的呢？ Java 1.3之后一个重要特征是反射（reflection），它允许程序在运行的时候动态的生成对象、执行对象的方法、改变对象的属性，spring就是通过反射来实现注入的。

　　理解了IoC和DI的概念后，一切都将变得简单明了，剩下的工作只是在spring的框架中堆积木而已。

## IoC**(控制反转)**DI**(依赖注入)**

　　在平时的java应用开发中，我们要实现某一个功能或者说是完成某个业务逻辑时至少需要两个或以上的对象来协作完成，在没有使用Spring的时候，每个对象在需要使用他的合作对象时，自己均要使用像new object() 这样的语法来将合作对象创建出来，这个合作对象是由自己主动创建出来的，创建合作对象的主动权在自己手上，自己需要哪个合作对象，就主动去创建，创建合作对象的主动权和创建时机是由自己把控的，而这样就会使得对象间的耦合度高了，A对象需要使用合作对象B来共同完成一件事，A要使用B，那么A就对B产生了依赖，也就是A和B之间存在一种耦合关系，并且是紧密耦合在一起，而使用了Spring之后就不一样了，创建合作对象B的工作是由Spring来做的，Spring创建好B对象，然后存储到一个容器里面，当A对象需要使用B对象时，Spring就从存放对象的那个容器里面取出A要使用的那个B对象，然后交给A对象使用，至于Spring是如何创建那个对象，以及什么时候创建好对象的，A对象不需要关心这些细节问题(你是什么时候生的，怎么生出来的我可不关心，能帮我干活就行)，A得到Spring给我们的对象之后，两个人一起协作完成要完成的工作即可。

　　所以**控制反转IoC(Inversion of Control)是说创建对象的控制权进行转移，以前创建对象的主动权和创建时机是由自己把控的，而现在这种权力转移到第三方**，比如转移交给了IoC容器，它就是一个专门用来创建对象的工厂，你要什么对象，它就给你什么对象，有了 IoC容器，依赖关系就变了，原先的依赖关系就没了，它们都依赖IoC容器了，通过IoC容器来建立它们之间的关系。

　　这是我对Spring的IoC**(控制反转)**的理解。DI**(依赖注入)**其实就是IOC的另外一种说法，DI是由Martin Fowler 在2004年初的一篇论文中首次提出的。他总结：**控制的什么被反转了？就是：获得依赖对象的方式反转了。**



## Spring事务的配置方式



@Transactional使用JDBC事务来进行事务控制,只能配置在public方法/类上,并且类内部的调用不支持事务

事务开始时，通过AOP机制，生成代理connection对象，并将其放入 DataSource 实例的某个与 DataSourceTransactionManager 相关的容器中





spring 所有的事务管理策略类都继承自 PlatformTransactionManager

```java
public interface PlatformTransactionManager extends TransactionManager {
  TransactionStatus getTransaction(@Nullable TransactionDefinition var1);

  void commit(TransactionStatus var1) throws TransactionException;

  void rollback(TransactionStatus var1) throws TransactionException;
}
```





### 回滚原则



spring事务管理器会捕捉未处理的异常,根据规则决定是否回滚

默认只在unchecked异常（runtime exception）时回滚









### 1. 编程式事务管理



编程式事务管理是侵入性事务管理，使用TransactionTemplate或者直接使用PlatformTransactionManager，对于编程式事务管理，Spring推荐使用TransactionTemplate。



### 2. 声明式事务管理



声明式事务管理建立在AOP之上，其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，执行完目标方法之后根据执行的情况提交或者回滚。
 编程式事务每次实现都要单独实现，但业务量大功能复杂时，使用编程式事务无疑是痛苦的，而声明式事务不同，声明式事务属于无侵入式，不会影响业务逻辑的实现，只需要在配置文件中做相关的事务规则声明或者通过注解的方式，便可以将事务规则应用到业务逻辑中。
 显然声明式优于编程式事务管理，这正是Spring倡导的非侵入式的编程方式。唯一不足的地方就是声明式事务管理的粒度是方法级别，而编程式事务管理是可以到代码块的，但是可以通过提取方法的方式完成声明式事务管理的配置



### 传播机制

通过`TransactionDefinition.XXX`配置

事务嵌套时，事务调用了另外一个事务，需要依赖传播机制决定各自提交/内层的事务合并到外层提交

PROPAGATION_REQUIRED	默认		加入外层/新建事务执行

PROPAGATION_REQUES_NEW		外层挂起，新建事务，当前事务执行完毕，恢复上层事务的执行。如果外层没有事务，执行当前新开的事务

PROPAGATION_SUPPORT			加入外层/非事务执行

PROPAGATION_NOT_SUPPORT		外层挂起，不支持事务地执行内层代码,内层不会回滚

PROPAGATION_NEVER			外层有事务就抛出异常

PROPAGATION_MANDATORY		外层没有事务则抛出异常

PROPAGATION_NESTED		支持状态保存点，当前事务回滚到某一个点，前提是子事务没有把异常吃掉





### 隔离级别

通过``isolation =XXX``配置

| Isolation.READ_UNCOMMITTED | 读未提交 |
| -------------------------- | -------- |
| Isolation.READ_COMMITTED   | 读提交   |
| Isolation.REPEATABLE_READ  | 可重复读 |
| Isolation.SERIALIZABLE     | 串行     |





## 事务超时

事务可能涉及对数据库的锁定，长时间运行事务会不必要地占用数据库资源。这时就可以声明一个事务在特定秒数后自动回滚，不必等它自己结束。

由于超时时钟在一个事务启动的时候开始的，因此，只有对于那些具有可能启动一个新事务的传播行为（REQUIRES_NEW、REQUIRED、NESTED）的方法来说，声明事务超时才有意义













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







# 系统架构



## 面向服务架构



SOA是一个组件模型，它将应用程序拆分成不同服务,每个服务可以独立部署

==服务之间松耦合,服务内部是高内聚,每个服务只关注完成一个功能==

通过接口和契约将服务联系起来

接口是采用中立的方式进行定义的，独立于实现服务的硬件平台、操作系统和编程语言。这使得构建在各种各样的系统中的服务以统一和通用的方式进行交互



* 优点	测试容易 可伸缩性强 可靠性强 跨语言 团队协作容易 系统迭代容易

* 缺点	运维成本高，部署数量多 接口兼容多版本 分布式系统的复杂性 分布式事务



###  微服务架构的六种常用设计模式



代理设计模式 聚合设计模式 链条设计模式 聚合链条设计模式,数据共享设计模式 异步消息设计模式



### 网关服务



网关服务，通常是外部访问服务的唯一接口，访问内部的所有服务都必须先经过网关服务。网关服务的主要功能是消息解析过滤，路由，转发等



云存储网关

- 保护数据的加密技术
- 压缩。重复数据删除
- 实现更快性能的WAN优化
- 快照
- 版本控制
- 数据保护





## AKF 拆分原则



AKF可扩展立方（Scalability Cube） 。这个立方体中沿着三个坐标轴设置分别为：X、Y、Z。

Y轴扩展会将庞大的整体应用拆分为多个服务。

X 轴扩展通过绝对平等地复制服务与数据，以解决容量和可用性的问题。其实就是将微服务运行多个实例，做集群加负载均衡的模式

Z 轴扩展通常是指基于请求者或用户独特的需求，进行系统划分，并使得划分出来的子系统是相互隔离但又是完整的。以生产汽车的工厂来举例：福特公司为了发展在中国 的业务，或者利用中国的廉价劳动力，在中国建立一个完整的子工厂，与美国工厂一样，负责完整的汽车生产。



## DAO 模式

DataAccess Object

为数据库或其他持久化机制提供了抽 象接口的对象，在不暴露数据库实现细节的前提下提供了各种数据操作。为了建立一个 健壮的 Java EE 应用，应该将所有对数据源的访问操作进行抽象化后封装在一个公共 API 中。用程序设计语言来说，就是建立一个接口，接口中定义了此应用程序中将会用到的 所有事务方法。在这个应用程序中，当需要和数据源进行交互的时候则使用这个接口， 并且编写一个单独的类来实现这个接口，在逻辑上该类对应一个特定的数据存储。DAO 尚学堂 Java 面试题大全及参考答案 408 模式实际上包含了两个模式，一是 Data Accessor（数据访问器），二是 Data Object （数据对象），前者要解决如何访问数据的问题，而后者要解决的是如何用对象封装数据。

