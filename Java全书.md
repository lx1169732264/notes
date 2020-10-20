

# String

 

如果经常对字符串进行各种各样的修改，那么使用 String 来代表字符串的话会引起很大的内存开销。因为 String 对象建立之后不能再改变，所以对于每一个不同的字符串，都需要一个 String 对象来表示。这时，应该考虑使用StringBuffer 类，它允许修改，而不是每个不同的字符串都要生成一个新的对象。并且，这两种类的对象转换十分容易。

调用构造器创造string，性能低下且内存开销大，并且没有意义，因为String对象不可改变，所以对于内容相同的字符串，指向同一个String

上面的结论还基于这样一个事实：对于字符串常量，如果内容相同，Java认为它们代表同一个 String 对象。关键字 new 调用构造器，总是会创建一个新的对象，无论内容是否相同。









## String空构造器

 

```
//底层维护字符数组
private final char value[];

	/** Note that use of this constructor is unnecessary since Strings are immutable.
JDK源码上的注解: 注意，由于字符串是不可变的，因此不需要使用此构造函数。
用构造函数创建string是无意义的 ,并且性能低
*/
    public String() {
    //对于new String(),仅仅是分配了空字符串的数组地址,并没有产生新的对象
        this.value = "".value; }
```

 

## indexof("")

indexof对于不存在的字符返回-1	但对于空字符串返回0

 

对于空字符串的下标获取,

首先调用这个方法

```
public int indexOf(String str) {
  return indexOf(str, 0);}
```



对于不指定下标的indexof()方法,默认分配0的初始下标

```
public int indexOf(String str, int fromIndex) {
  return indexOf(value, 0, value.length,
      str.value, 0, str.value.length, fromIndex);}
```



```
public int indexOf(String str, int fromIndex) {
  return indexOf(value, 0, value.length,
      str.value, 0, str.value.length, fromIndex);}
```



对所有indexof方法的处理最终都是在调用这个方法

```
static int indexOf(char[] source, int sourceOffset, int sourceCount,
    char[] target, int targetOffset, int targetCount,
    int fromIndex) {

	//在这里规定了,空字符串时,fromIndex =sourceCount=0,返回0
  if (fromIndex >= sourceCount) {
    return (targetCount == 0 ? sourceCount : -1);
  }
  if (fromIndex < 0) {
    fromIndex = 0;
  }
  if (targetCount == 0) {
    return fromIndex;
  }
}
```





## 反转(递归)

public class A{

 public static String reverse(String originStr) {

if(originStr == null || originStr.length() <= 1)

 return originStr;

 return reverse(originStr.substring(1)) +

originStr.charAt(0);

 }



## 编码转换

答：代码如下所示:

String s1 = "你好";

String s2 = newString(s1.getBytes("GB2312"), "ISO-8859-1");



## 替换

s.replaceAll("原字符","替代字符");



## String编译优化

字符串对象创建有两种形式，

1.字面量形式，如String str = "aaa"	"aaa" 存进字符串常量池

2.new 							new 创建的存进了堆

 

 对于字符串，其对象的引用都是存储在栈中的，如果是编译期已经创建好的就存储在常量池中(双引号定义的或final修饰并且能在编译期就能确定的)，如果是运行期才能确定的就存储在堆中（如：new关键字创建出来的）。

对于equals相等的字符串，在常量池中永远只有一份，在堆中有多份

 

String a="hello2";

String b="hello"+2;

System.out.println((a==b));

 输出为：true。因为 ”hello” +2在编译期就已经被优化成 “hello2”，因此在运行期变量 a 和 b 指向的是同一个对象(在字符串常量池)

String a="hello2";

String b="hello";

String c=b+2;

System.out.println((a==c));

输出为 false。由于有符号引用的存在，所以String c=b+2不会在编译期间被优化，不会把 b+2 当做字面量处理，因此生成的对象是保存在堆上的。所以a和c不是指向同一个对象。



### String s=new String(“abc”);创建了几个对象

两个或一个，”abc”对应一个对象，这个对象放在字符串常量缓冲区，常量”abc”不管出现多少遍，都是缓冲区中的那一个。New String 每写一遍，就创建一个新的对象，它一句那个常量”abc”对象的内容来创建出一个新String 对象。如果以前就用过’abc’，直接从缓冲区拿。

## String/Builder/Buffer区别

* 相同点：
  * 都使用 final 修饰，不能派生子类，操作的相关方法也类似例如获取字符串长度等；

* 不同点：
  * String只读，内容不能改变，StringBuffer和StringBuilder类表示的字符串对象可以直接进行修改，在修改的同时地址值不会发生改变。
  * StringBuilder是JDK1.5新特性，和StringBuffer的方法完全相同，线程不安全,性能高。
  * String、StringBuffer、StringBuilder 三者类型不同，无法用 equals()方法比较内容



## String为什么final 

* 若允许被继承，则其高度的被使用率可能会降低程序的性能

* 为了安全。JDK中的核心类比如 String，内部很多方法的实现都不是 java 编写的，只是==调用操作系统的 API，也就是本地方法调用==，如果这种类可以被继承并重写，将导致操作系统面临风险





# 序列化

序列化能够将实例对象的状态信息写入到字节流,使其可以通过socket进行传输或者持久化.然后通过反序列化恢复对象状态

## 两种用途

很多应用需要对某些对象进行序列化，让它们离开内存空间，入住物理硬盘，以便长期保存。比如最常见的是 Web 服务器中的 Session 对象，当有 10 万用户并发访问，就有可能出现 10 万个 Session 对象，内存可能吃不消，于是 Web 容器就会把一些 seesion 先序列化到硬盘中，等要用了，再把保存在硬盘中的对象还原到内存中。当两个进程在进行远程通信时，彼此可以发送各种类型的数据。无论是何种类型的数据，都会以二进制序列的形式在网络上传送。发送方需要把这个 Java 对象转换为字节序列，才能在网络上传送；接收方则需要把字节序列再恢复为 Java 对象。

 

## 两种实现方法

实现Serializable接口，所有的序列化将会自动进行

实现**Externalizable接口** ,在writeExternal方法中进行手工指定所要序列化的变量

 

## Serializable原理

有AB两个类，B含有一个指向A类对象的引用，进行实例化{ A a = new A(); B b = new B(); }，在内存中分配了两个空间，在写入文件时 ,b包含对a的引用，系统会将**a的数据复制一份到b中，从文件中恢复对象时(重新加载到内存)，**内存分配了三个空间，而对象**a同时存在两份**

　　

　　1.保存到磁盘的所有对象都获得一个序列号(1, 2, 3等等)

　　2.当要保存一个对象时，先检查该对象是否被保存了。

3.如果以前保存过，只需写入"与已经保存的具有序列号x的对象相同"的标记，否则，保存该对象通过以上的步骤序列化机制解决了对象引用的问题！

 

　　实例化的过程中相关注意事项
　　a）序列化时，**只对对象的状态进行保存，而不管对象的方法
　　b）当一个父类实现序列化，**子类自动实现序列化，不需要显式实现Serializable接口；
　　c）当一个对象的**实例变量引用其他对象，**引用对象自动序列化；
　　d）并非所有的对象都可以序列化，至于为什么不可以，有很多原因了，比如：
　　1.安全方面的原因，比如一个对象拥有private，public等field，对于一个要传输的对象，比如写到文件，或者进行RMI传输 等等，在序列化进行传输的过程中，这个对象的private等域是不受保护的。
　　2. 资源分配方面的原因，比如socket，thread类，如果可以序列化，进行传输或者保存，也无法对他们进行重新的资源分配，而且，也是没有必要这样实现。

 

## serialversionUID

目的是序列化对象版本控制。如果在新版本中这个值修改了，新版本就不兼容旧版本，反序列化时会抛出InvalidClassException异常。如果修改较小，比如仅仅是增加了一个属性，我们希望向下兼容，那就不用修改；如果我们删除了一个属性，或者更改了类的继承关系，必然不兼容旧数据，这时就应该手动更新版本号

 

serialVersionUID = 1L意义:

有两种生成方式：一个是默认的1L， 一个是根据类名、接口名、成员方法及属性等来生成一个64位的哈希字段，如： -8940196742313994740L

不指定 serialVersionUID将导致添加或修改类中的任何字段时, 已序列化类将无法恢复



## 防止被序列化

声明为**静态或瞬态**

 

## 瞬态transient

生命周期仅存于调用者的内存中 ,不会被持久化

只能修饰**非本地变量，不能修饰方法和类 ,该类已Serializable接口

 

一旦变量被transient修饰，变量将不再是对象持久化的一部分，该变量内容在序列化后无法获得访问。

**实现Externalizable接口  ,则无视transient**

 

 





# 接口

## 默认方法

实现接口需要实现其所有的抽象方法 ,当接口加入新方法时，我们就需要对项目重新编写

使用**default**修饰， 定义**方法体**。default方法所有的子类会**默认实现** ，可以避免修改代码



这个default是jdk8新关键字，**和访问限定修饰符“default”不是一个概念**，与switch中的default功能完全不同.

实际上是**public default** ,省略了public



与抽象类的不同：抽象类更多的是提供一个模板，子类之间的某个流程大致相同，仅仅是某个步骤可能不一样（模板方法设计模式），这个时候使用抽象类，该步骤定义为抽象方法。而default关键字是用于扩展



## 静态方法

接口的静态方法不会被实现类所实现

**只用于内部调用**



两个接口定义**相同静态方法**，实现类实现这两个接口，并不会产生错误，编译器通过**反射**来区分是哪个接口下的方法

两个接口定义**相同非静态方法**，并且一个实现类同时实现了这两个接口，那么必须在实现类中重写默认方法，否则编译失败。

**静态方法调用    类名.方法**   通过**反射**来区分哪个接口下的方法

​    **非静态         对象.方法** 



## 函数式接口 

**只有一个抽象方法**

### @FunctionalInterface

只在编译期起作用，如@Override注解。编译期会强制检查该接口是否符合函数式接口的条件，不符合则会报错。**即使不使用，只要满足定义也是函数式接口**。

![image-20200826214004498](image.assets/image-20200826214004498.png)



### Supplier

java.util.function.Supplier<T> 接口仅包含一个无参的方法： T get() 。

用来获取一个泛型参数指定类型的对象数据。由于这是一个函数式接口，这也就意味着对应的Lambda表达式需要“对外提供”一个符合泛型类型的对象数据。

```
   public void testGetUser() {
        User user = getUser(User::new);}

    private User getUser(Supplier<User> supplier) {
        return supplier.get();}
```

 

​	Supplier求数组元素的最小值

```
public void testGetMin(){
        int[] arr={5,3,100,10};
        int min = getMin(()->{
            int minNum=arr[0];
            for (int i : arr) {
                if (i<minNum) minNum=i;}
            return minNum;
        });}

    private Integer getMin(Supplier<Integer> supplier) {
        return supplier.get();
    }
```

 

### Consumer接口

java.util.function.Consumer<T> 

**与Supplier接口相反**，它不是生产一个数据，而是消费一个数据



**抽象方法：accept**，消费一个指定泛型的数据

```
public void testConsumer() {
        User user = new User();
        setUserDefaultSex(u -> u.setSex("nan"), user);
        //user的sex被改变}
        
    private void setUserDefaultSex(Consumer<User> consumer, User user) {
        consumer.accept(user);}
```



**默认方法：andThen**

**方法的参数和返回值全都是 Consumer 类型**，那么就可以实现效果：消费数据的时候，首先做一个操作，然后再做一个操作，实现组合

要想实现组合，需要两个或多个Lambda表达式

```
public void testConsumer2() {
        User user = new User();
        setUserNameAndSex(u -> u.setSex("nan"), u -> u.setName("aa"), user);
        System.out.println(user.getSex() + user.getName());}

    private void setUserNameAndSex(Consumer<User> one, Consumer<User> two, User user) 			{one.andThen(two).accept(user);}
```

 

### Predicate接口

对某种类型的数据进行判断，**得到boolean**结果

**抽象方法：test** 	用于条件判断

```
public void testPredicate() {
        longThan(s -> s.length() > 5, "hello!!");}

    private void longThan(Predicate<String> predicate, String str) {
        boolean flag = predicate.test(str);}
```



**默认方法：and** 

**默认方法：or** 

```
public void testSuccess() {
    successMan(s -> s.contains("富"), s -> s.contains("帅"), "高富帅");}

private void successMan(Predicate<String> one, Predicate<String> two, String str) {
    boolean flag = one.or(two).test(str);}
```

**默认方法：negate** 	取反





### Function接口



**抽象方法：apply** 

java.util.function.Function<T,R>根据 T类型的参数得到 R类型的返回值

```
public void testFunction() {
        Integer value = parseInteger(Integer::parseInt, "10");}

    private Integer parseInteger(Function<String, Integer> function, String str) {
        return function.apply(str);}
```

**默认方法：andThen**



# Lambda

函数式编程思想 ,只关注做什么 ,不关注怎么做

**lambda不是语法糖**

语法糖 :写法不同 ,实现原理相同 ,如增强for循环



(类型 参数1, 类型 参数2....) -> {代码}

**参数没有则留空**

**参数必须是函数式接口@FunctionalInterface**



## 省略规则

参数类型可以省略 ,但只能都省略或都不省略

参数只有一个 ,小括号能省略

大括号内语句只有一条 ,大括号/分号/return关键词能省略



## 延迟执行

new了对象后，不一定会被使用

```
public class Demo01Logger {
    private static void log(int level, String msg) {
        if (level == 1) {
            System.out.println(msg);}}
    public static void main(String[] args) {
        String msgA = "Hello";
        String msgB = "World";
        log(1, msgA + msgB);}}
```

先合并了字符串 ,再判断level==1 ,决定要不要执行方法 

```
@FunctionalInterface
public interface MessageBuilder {
    String buildMessage();
}
public class Demo02LoggerLambda {
    private static void log(int level, MessageBuilder builder) {
        if (level == 1) {
            System.out.println(builder.buildMessage());}}

    public static void main(String[] args) {
        String msgA = "Hello";
        String msgB = "World";
        log(1, () -> msgA + msgB  );}}
```

优化后 ,先判断 ,后执行字符串合并





## 方法引用 ::

```
print(e -> System.out.println(e));
```

这个lambda只是接收了参数 ,并将它打印 ,而打印的方法有现成的System.out.println

```
print(System.out::println);
```



如果Lambda要表达的函数方案已存在于某个方法的实现中，可以通过::来引用该方法

* Lambda写法	 s -> System.out.println(s); 	拿到参数之后经Lambda之手，继而传递给 System.out.println 方法去处理

* 方法引用写法	 System.out::println     直接让println 方法来取代Lambda

* 对象名引用	user :: getName

* 构造器引用	User::new

* 类名引用		User::getName

* super引用成员方法  super::sayHello

  ```
  public class Woman extends Human {
      @Override
      public void sayHello() {
          System.out.println("大家好,我是Man!");}
  
      public void method(Greetable g) {
          g.greet();}
  
      public void show() {
          method(super::sayHello);}}
  ```

* this引用	this::buyHouse

```
public class Husband {
    private void buyHouse() {
        System.out.println("买套房子");}

    private void marry(Richable lambda) {
        lambda.buy();}

    public void beHappy() {
        marry(this::buyHouse);}}
```



# Stream

得益于Lambda所带来的函数式编程，引入全新的Stream概念，用于解决集合类库的弊端。

Stream是集合元素的函数模型，不是集合，也不是数据结构，其本身不存储任何元素（或地址）,只是在原数据集上定义了一组操作。也不会改变原有数据

Stream流不保存数据，Stream操作是尽可能惰性的，即每当访问到流中的一个元素，才会在此元素上执行这一系列操作。



不使用stream ,在进行list过滤时 ,可能需要写多个for循环

```
list.stream().filter(s -> s.startsWith("张"))
                .filter(s -> s.length() == 3)
                .forEach(System.out::println);
```

 stream().forEach用的多线程方式，调用线程池的时候额外耗费时间。但在循环内处理的时间长，或要循环调用远程接口，多线程的性能高

## 获取流

* Collection接口获取流

Collection 接口中加入了default方法stream() 获取流，所有实现类均可获取流。

* Map获取流

java.util.Map 接口不是 Collection 的子接口，且其K-V数据结构不符合流元素的单一特征，所以获取对应的流

需要分key、value或entry等情况：

```
public static void main(String[] args) {
        Map<String, String> map = new HashMap<>();
        Stream<String> keyStream = map.keySet().stream();
        Stream<String> valueStream = map.values().stream();
        Stream<Map.Entry<String, String>> entryStream = map.entrySet().stream();}
```

* 数组获取流 

如果使用的不是集合或映射而是数组，由于数组对象不可能添加默认方法，所以 Stream 接口中提供了静态方法of() ，使用很简单：

```
public static void main(String[] args) {
        String[] array = {"张无忌", "张翠山", "张三丰", "张一元"};
        Stream<String> stream = Stream.of(array);}
```



## 常用方法

* forEach
* forEachOrdered     按原顺序输出 ,foreach时无序的
* filter        将一个流转换成另一个子集流

```
Stream<String> result = original.filter(s -> s.startsWith("张"));
```

* map      将流中的元素映射到另一个流	将list截取字符串 ,映射到stream

```
Stream<String> stream = list.stream().map(e -> e.substring(2));
```

* count   统计个数

* limit    取前几个

* skip     跳过前几个                          **limit + skip实现分页**

* concat   合并流 ,Stream的静态方法

```
 Stream<String> result = Stream.concat(streamA, streamB);
```

* collect   转化为Collection/Map/数组

```
list.stream().collect(Collectors.toList());
```

* distinct   去重





## parallelStream 并行流

通过默认的ForkJoinPool，提高多线程任务的速度，默认线程数量等于运行计算机上的处理器数量

Java8为ForkJoinPool添加了一个通用线程池，这个线程池用来处理那些没有被显式提交到任何线程池的任务。当调用Arrays类上添加的新方法时，自动并行化就会发生。





# HashMap



继承了AbstractMap

实现了Map ，克隆，序列化接口

```
HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
```



AbstractMap已经实现过了Map接口，而HashMap又继承了AbstractMap，这样使得HashMap已经实现了Map接口，然而HashMap又再次去实现了Map接口

这是JDK中多此一举的失误

```
AbstractMap<K,V> implements Map<K,V> {
```





## 按位与2次幂容量

取余:xxx%16  不断在做除法,效率低,并且负数取余仍是负数,还需要转为正数

按位与: 	hash&(length-1)

​			(length-1)  1111

​			(hash)   1001

​			     =1001

当length-1不为全1,即length不为2的幂,将出现0,而0的部分按位与永远为0

将导致0的桶永远放不进



## 7 HashMap死锁隐患



![img](image.assets/wps1-1600480677581.jpg) 

原先:	3->5->7

resize:	7à3

多线程环境下,可能同时3->7	7->3,出现循环

![img](image.assets/wps2-1600480677595.jpg) 

当查询时就会出现死锁



==可以通过精心设计的一组object实现dos(拒绝服务攻击)==

大量的object的HashCode相同,使得它们被存放在同一个桶中,使得HashMap退化为链表,而链表的查询复杂度O(n)

 

## 8 HashMap

### hash方法

  static final int hash(Object key) {

​    int h;

​    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16); }

jdk7中,容易出现低位相同,高位不同的hash	如1101……….1111

​											1001………1111

将高位与地位异或(不进位的加法),能够减少碰撞的概率

 

### resize方法

  Node<K,V> loHead = null, loTail = null;

  Node<K,V> hiHead = null, hiTail = null;

扩容时,将原链表拆为两个高低位的链表

比如16个桶,哈希码11111…….11101

​							 1111	=1101

扩容32个桶,					11111	=11101

扩容后第一位只能是0或1,并且桶中的元素被分配在了1xxx和0xxx两个新桶中,元素保持原先的顺序.而保持了顺序就降低了多线程中,顺序调换出现的死锁问题

 

Map.getOrDefault((Object key, V defaultValue),取不到key时,将返回默认的value



## 底层数组创建机制

8之前，创建对象时就创建了数组

8之后，**首次调用put**才创建



## hash相等时

会产生hash碰撞

key值相同则替换，否则加到后面		**比较key用equals**





## 成员变量



* MAXIMUM_CAPACITY = 1 << 30    最大容量

* UNTREEIFY_THRESHOLD = 6    链表的值小于6则会从红黑树转回链表

* MIN_TREEIFY_CAPACITY** = 64	超过这个值，才能进行树形化

* Node<K, V>[] table	table用来初始化(必须是二的n次幂)

* Set<Entry<K, V>> entrySet	存放缓存

* size	元素个数

* int modCount	修改次数

* int threshold		下次扩容的临界值，（容量\*负载因子)

* float loadFactor	哈希表的负载因子









# NIO

![nio与io区别](image.assets/nio与io区别.png)

io是单向的输入流不能用于输出

而nio利用缓冲实现了数据在channel中的双向传输



nio为不同的数据类型提供了不同种类的缓冲

allocate()	分配指定大小的缓冲区



## Buffer类4个属性与方法

buffer	标记当前的position

capacity	最大容量

limit	可以操作数据的个数

position	正在被操作数据的位置

​	position<=limit<=capacity



put() get()存取数据

flip()读数据模式	开启之后,当调用get()时, 会将position调为0, limit调为当前最大存储位置,然后再执行get()方法, 不然position并不在起始位置

rewind()重置读数据模式	再次将position调为0, limit调为当前最大存储位置

clear()	**并不会删除数据**,只是将三个属性初始化 ,里面的数据处于"被遗忘"状态 ,position和limit都被初始化,难以读取数据

**mark()	记录当前的position位置**

**reset()	配合mark()的使用,回到mark的位置**



## 直接缓冲区与非直接缓冲区

非直接	调用allocate()方法分配缓冲区 ,缓冲区在**jvm**

直接	allocateDirect(),在**物理内存**

jvm对于直接缓冲区,会尽量避免使用中间缓冲区进行数据的读写,而是直接在缓冲区上进行io操作

分配**直接**缓冲区需要**更大的成本** ,也**不会被gc回收** ,会影响应用程序的内存

![image-20200809134909724](.\image.assets\image-20200809134909724.png)

对于非直接缓冲区 ,物理磁盘的数据先读取至内核地址空间,再被copy到jvm内存,最后到应用程序

点开allocate()方法也可以看到返回的是heap堆缓冲

![image-20200809142318554](.\image.assets\image-20200809142318554.png)



![image-20200809141028612](.\image.assets\image-20200809141028612.png)

对于直接缓冲区 ,应用程序通过物理内存映射文件直接与物理磁盘交换数据 省略了copy的步骤

直到gc释放了应用程序与物理内存映射文件的引用 ,才会销毁链接

直接缓冲区的建立与销毁是成本很高的 ,而gc无法及时回收会导致浪费

所以直接缓冲区适合长时间的连接,大文件的传输



## 通道

最早,cpu需要建立若干io接口来进行io操作,这将导致cpu被占用

后来引入了**DMA**直接存储器访问 ,cpu将io操作交给DMA进行 ,DMA先向cpu申请资源 ,然后形成**DMA总线** ,不过总线的过多也会导致总线冲突,最后影响性能

而channel通道就类似于DMA总线 ,是一个完全独立的处理器 ,专门用于处理io ,不需要向cpu申请资源

![image-20200809142947422](.\image.assets\image-20200809142947422.png)



### 主要实现类

* FileChannel	               本地传输
* SocketChannel             TCP
* ServerSocketChannel  TCP
* DatagramChannel        UDP

### 获取通道 getChannel()

本地

* FileInputStream/Output
* RandomAccessFile

Web

* Socket
* ServerSocket
* DatagramSocket

JDK1.7中NIO.2针对各个通道提供open()静态方法

JDK1.7中NIO.2的File工具类提供newByteChannel()方法



## 关闭

使用IO流往往需要多次使用try/catch

如果在一个try/catch中关闭多个流,将会导致关闭时其中一个流 ,抛出异常,程序中断,之后的流将不再被关闭!!!

需要一条一条的try/catch





# 多线程

## 并行/并发/串行

* 并行	有两个门,两个人从前后门进入 ,互不干扰

* 并发	两个人挤一个门进入

并行时同一时刻多个进程运行 ,并发是经过上下文快速切换 ,造成同时运行的假象

多线程代码是并发而不是并行 ,并发是因为多进程/多线程都是需要去完成的任务 ,不并行是因为**并行与否由操作系统的调度器决定**

* 串行    按先后顺序进行



## JUC包	java.util.concurrent	 

jdk1.5新特性 ,存放并发工具类

如CopyOnWriteArrayList ,底层维护了一个transient(序列化) volatile(唯一)的数组

```
final transient ReentrantLock lock = new ReentrantLock();//可重用锁
private transient volatile Object[] array;
```



即使没有主动创建线程 ,后台也会有多个线程 ,如主线程(用户线程) ,gc线程(守护线程)

线程的运行由调度器安排调度 ,调度器由操作系统控制 ,先后顺序无法干预

对同一份资源操作时 ,存在资源抢夺问题 ,需要加入并发控制

线程会带来额外开销 ,如cpu调度时间 ,并发控制开销

每个线程在自己的工作内存交互 ,内存控制不当会造成数据不一致



## ThreadLocal 原理

ThreadLocal 为每个线程创造一个资源的复本 ,而不是共享资源。将每一个线程存取数据的行为加以隔离，给每个线程特定空间来保管该线程所独享的资源

原理 : ThreadLocal 类中有一个Map，用于存储每一个线程的变量的副本。



## 创建线程3种方式

* 继承Java.lang.Thread类，并覆盖 run() 方法        Thread本身就继承了Runnable 

```
public class TestThread extends Thread {
    @Override
    public void run() {super.run();}

    public static void main(String[] args) {
        new TestThread().start();}}
```

优势：编写简单；

劣势：单继承 ,无法继承其它父类



* 实现 Java.lang.Runnable 接口，并实现 run()方法。

```
public class TestThread2 implements Runnable {
    @Override
    public void run() {}

    public static void main(String[] args) {
        TestThread2 testThread =new TestThread2();
        new Thread(testThread).start();}}
```

优势：可继承其它类，多线程可共享同一个Thread对象

劣势：编程方式稍微复杂，如需访问当前线程，需调用Thread.currentThread()



* 实现Callable接口  (有返回值 ,可以抛出异常)

1. 实现Callable接口 ,定义返回值类型
2. 重写call()方法 ,需要抛出异常
4. 创建执行服务  ExecutorService service = Executors.newFixedThreadPool(3);
5. 提交执行   Future<String> result = service.submit(new TestCallable());
6. 获取结果  result.get();
7. 关闭服务   service.shutdown();

```
public class TestCallable implements Callable<String> {

    @Override
    public String call() {
        return Thread.currentThread().getName(); }

    public static void main(String[] args) throws ExecutionException, InterruptedException  {
        //创建执行服务
        ExecutorService service = Executors.newFixedThreadPool(3);
        //提交执行
        Future<String> result = service.submit(new TestCallable());
        //获取返回值
        String str = result.get();
        //关闭服务	需要抛出2个异常
        service.shutdown();}}
```





## 线程的6种状态

![image-20200904093701936](image.assets/image-20200904093701936.png)

* 新建 NEW，线程被创建出来，但尚未启动时的线程状态；

* 就绪 RUNNABLE，表示可以运行的线程状态，它可能正在运行，或者是在排队等待操作系统给它分配 CPU 资源；

比如Thread.start方法就是将线程从NEW状态 转换成 RUNNABLE 状态。

* 阻塞 BLOCKED，处于阻塞状态的线程正在等待监视器锁

比如等待执行 synchronized 代码块或者使用 synchronized 标记的方法。

* 等待 WAITING，等待另一个线程执行某个特定的动作。

比如，一个线程调用了Object.wait()方法，那它就在等待另一个线程调用Object.notify() 或 Object.notifyAll() 方法。

* 计时等待 TIMED_WAITING，和上者类似，只是多了一个超时时间。

比如调用了有超时时间设置的方法 Object.wait(long timeout) 和 Thread.join(long timeout) 等这些方法时，它才会进入此状态；

* 终止 TERMINATED，线程死亡	**死亡线程将无法再次start**



## 线程的方法



* setPriority()	更改优先级	优先级范围1-10

​	优先级低的也有可能被先调用 ,全看cpu心情 ,这将导致性能倒置 :优先级高的一直在等待

* yield()	**暂停但不阻塞**正在执行的线程对象 ,**转入就绪状态** ,cpu有可能再次调度到礼让线程 ,导致礼让失败

* sleep()	**转入阻塞转态** ,存在异常抛出InterruptedException ,**监控状态依然保持,不会释放锁**   

**wait 是Object的方法 ,会导致放弃对象锁**，进入等待此对象的等待锁定池。只有唤醒此对象后才进入对象锁定池，准备获得对象锁进行运行状态



* join()	合并线程 ,相当于插队 ,其他线程等待该线程终止	容易造成线程阻塞

`interrupt()	中断线程(不推荐)`

isAlive	是否存活

* start()    此时线程处于就绪状态，并没有运行，得到 cpu 时间片**再执行 run()方法** .run()方法只是类的一个普通方法而已，**如果直接调用 run 方法，程序中依然只有主线程**，还是要顺序执行
* Thread.state / thread.getState()	获取线程状态
* **线程同时启动**    for 循环，调用 wait()方法，让所有线程等待 ,再调用 notifyAll(), 同时启动所有线程



final void wait() 	等待其它线程通知

void wait(long timeout) 线程等待指定毫秒参数的时间

final void wait(longtimeout,int nanos)线程等待指定毫秒、微妙的时间

final void notify()唤醒一个处于等待状态的线程。注意的是在调用此方法的时候，并不能确切的唤醒某一个等待状态的线程，而是由 JVM 确定唤醒哪个线程，而且不是按优先级。

final void notifyAll()唤醒同一个对象上所有调用 wait()方法的线程，注意并不是给所有唤醒线程一个对象的锁，而是让它们竞争



**不推荐调用jdk的stop ,destroy方法停止线程** ,可以在源码看到这些方法加上了@Deprecated注解 ,表示方法过时

应该用boolean标志 ,boolean=false停止线程 **,让线程自己停下来 ,而不是被动停止**



JDK 1.5 通过 Lock 接口提供了显式(explicit)的锁机制，增强了灵活性以及对线程的协调。Lock 接口中定义了加锁（lock()）和解锁(unlock())的方法，同时还提供了 newCondition()方法来产生用于线程之间通信的Condition 对象；

JDK 1.5 还提供了信号量(semaphore)机制，信号量可以用来限制对某个共享资源进行访问的线程的数量。在对资源进行访问之前，线程必须得到信号量的许可（调用 Semaphore 对象的 acquire()方法）；在完成对资源的访问后，线程必须向信号量归还许可（调用 Semaphore 对象的 release()方法）。





## 守护线程 daemon

**JVM必须保证用户线程执行完毕 ,但无需等待守护线程执行完毕**



如日志记录,监控内存 ,垃圾回收





## 线程同步

每个线程在自己的工作内存交互 ,内存没有同步会造成数据不一致



### 不安全线程案例

```
List<String> list = new ArrayList<String>();
for (int i = 0; i <= 10000; i++) {
    new Thread(() -> list.add(Thread.currentThread().getName())).start();
}
Thread.sleep(4000);
System.out.println(list.size());//不到10000	list.add()时,两个线程同时add,导致list被修改而不是添加
```



### 同步代码块/方法

synchronized控制对象的访问 ,每个对象对应一把锁 ,必须获得该方法的对象的锁才能执行方法 ,否则会线程阻塞

方法执行完毕 ,才会释放锁 ,让下一个线程拿到锁



* 同步代码块	synchronized (对象) { }
  * 同步代码块在方法内部的==对象上==加锁。



* 同步方法：public synchronized void xxx(int i) { }
  * 同步方法在==方法上==加synchronized ,锁的范围大，将导致性能差
  * ==同步方法默认锁定this ,即当前类,所以不需要指明对象==

**在静态方法中，都是默认锁定类对象**



### 锁

从jdk1.5开始 ,可以显式定义同步锁对象Lock ,实现同步

Lock接口 ,提供了对共享资源的独占访问 ,线程开始访问共享资源之前需要先获得Lock对象

锁保证了数据在方法中被访问时的正确性

锁会消耗性能 ,低优先级线程拿到排它锁 ,将导致性能倒置



ReentrantLock可重入锁 实现了Lock ,与synchronized相同并发性和内存语义

```
private final ReentrantLock lock = new ReentrantLock();
        lock.lock();					//在try外面加锁
        try {
            ...
        } finally {
            lock.unlock();}		//在finally解锁
```



#### Lock 与synchronized对比

Lock是显式锁 ,synchronized是隐式 ,出了作用域就释放

Lock只有代码块锁 ,没有方法锁

Lock在调度线程方面性能更好



**对象锁分为三种：共享资源、this、当前类的字节码文件对象**





#### 乐观锁与悲观锁

悲观锁，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会block直到它拿到锁。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。

乐观锁认为别人不会修改，所以不会上锁，但是在更新时判断在此期间别人有没有去更新数据，可以使用版本号等机制,在更新数据时会提高版本号,在提交时,提交版本低于目前版本,将回滚。乐观锁适用于多读的应用类型，可以提高吞吐量

两种锁各有优缺点，不可认为一种好于另一种，像乐观锁适用于写比较少的情况下，即冲突真的很少发生的时候，这样可以省去了锁的开销，加大了系统的整个吞吐量。但如果经常产生冲突，上层应用会不断的进行retry，反倒降低性能，所以这种情况下用悲观锁就比较合适。

#### 同步锁

 Java 中每个对象都有一个内置锁。 当程序运行到非静态的 synchronized 同步方法上时，自动获得与正在执行代码类的当前实例（this 实例）有关的锁。获得一个对象的锁也称为获取锁、 锁定对象、在对象上锁定或在对象上同步。 当程序运行到 synchronized 同步方法或代码块时才该对象锁才起作用。 一个对象只有一个锁。所以，如果一个线程获得该锁，就没有其他线程可以 获得锁，直到第一个线程释放（或返回）锁。这也意味着任何其他线程都不 能进入该对象上的 synchronized 方法或代码块，直到该锁被释放。 释放锁是指持锁线程退出了 synchronized 同步方法或代码块。 关于锁和同步，有一下几个要点： 1）只能同步方法，而不能同步变量和类； 2）每个对象只有一个锁；当提到同步时，应该清楚在什么上同步？也就是 说，在哪个对象上同步？ 3）不必同步类中所有的方法，类可以同时拥有同步和非同步方法。 4）如果两个线程要执行一个类中的 synchronized 方法，并且两个线程使 用相同的实例来调用方法，那么一次只能有一个线程能够执行方法，另一个需要等待，直到锁被释放。也就是说：如果一个线程在对象上获得一个锁， 就没有任何其他线程可以进入（该对象的）类中的任何一个同步方法。 5）如果线程拥有同步和非同步方法，则非同步方法可以被多个线程自由访 问而不受锁的限制。 6）线程睡眠时，它所持的任何锁都不会释放。 7）线程可以获得多个锁。比如，在一个对象的同步方法里面调用另外一个 对象的同步方法，则获取了两个对象的同步锁。 8）同步损害并发性，应该尽可能缩小同步范围。同步不但可以同步整个方 法，还可以同步方法中一部分代码块。 9）在使用同步代码块时候，应该指定在哪个对象上同步，也就是说要获取哪个对象的锁。

#### 方法锁和静态方法锁的区别

静态方法，需要对Class对象加锁。

非静态方法，需要对本对象(this)加锁。





### 死锁

多个线程各自占有一部分共享资源 ,并发生互相等待

常发生于**一个同步块同时拥有2个以上对象的锁**



**4个必要条件**

* 互斥	一个资源同时被多个进程使用

* 请求与保持	一个进程请求资源而阻塞 ,对已有的资源保持不释放

* 不剥夺	进程已获得的资源在未使用完之前 ,不会被抢夺

* 循环等待	若干个进程之间形成循环等待资源

只要打破一个条件就能避免死锁



**尽量不要嵌套同步**

synchronized (对象) { 

​	synchronized (对象) { }

}





## 线程通信

wait + notify 解决线程通信

这两个都是Object的方法 ,只能在同步方法或同步代码块中使用 ,否则会抛出IIIegalMonitorStateException



### 管程法

生产者把产品放入**缓冲区** ,消费者从缓冲区拿

每次操作时判断缓冲区的容量 ,满了则生产者不生产 ,空了消费者不消费

![image-20200909221411140](image.assets/image-20200909221411140.png)



### 信号灯法

判断**标志位** ,如果为真 ,等待 ,如果为假 ,唤醒

每次进行操作时判断标志位 ,决定wait或者是执行









## volatile能否保证线程安全？

不能。volatile是一种弱的同步机制，如需要强线程安全，还需要使用 synchronized。

volatile 变量用来确保将变量的更新操作通知到其他线程。当把变量声明为 volatile 类型后，编译器与运行时都会注意到这个变量是共享的，因此不会将该变量上的操作与其他内存操作一起重排序。volatile 变量不会被缓存在寄存器或者对其他处理器不可见的地方，因此在读取 volatile 类型的变量时总会返回最新写入的值。

一、volatile 的内存语义是：

当写一个 volatile 变量时，JMM 会把该线程对应的本地内存中的共享**变量值立即刷新到主内存**中。

当读一个 volatile 变量时，JMM 会把该线程对应的本**地内存设置为无效，直接从主内存中读取**共享变量。

二、volatile 底层的实现机制

如果把加入 volatile 关键字的代码和未加入 volatile 关键字的代码都生成汇编代码，会发现加入 volatile 关键字的代码会多出一个 lock 前缀指令。

1 、重排序时不能把后面的指令重排序到内存屏障之前的位置

2、使得本 CPU 的 Cache 写入内存

3、写入动作也会引起别的 CPU 或者别的内核无效化其 Cache，相当于让

新写入的值对别的线程可见。













## synchronized 关键字的用法

synchronized 关键字可以将对象或者方法标记为同步，以实现对对象和方法的互斥访问，可以用 synchronized(对象) { … }定义同步代码块，或者在声明方法时将 synchronized 作为方法的修饰符



### synchronized和Lock的异同？

Lock是Java 5以后引入的新的API

相同点：Lock 能完成synchronized所实现的所有功能；

不同点：Lock 有比 synchronized 更精确的线程语义和更好的性能。synchronized会自动释放锁，而 Lock 一定要求程序员手工释放，并且必须在 finally 块中释放（这是释放外部资源的最好的地方）





## 线程池（thread pool）

创建和销毁对象是很费时间的，**虚拟机将试图跟踪每一个对象，以便能够在对象销毁后进行垃圾回收**  线程池也**利于管理线程的个数与活跃数**

### 线程池参数

最大线程数 maximumPoolSize	性能最高的线程数

核心线程数 corePollSize	平时的流量需要的线程数

线程空闲时间 

空闲的线程保留的时间 keepAliveTime

阻塞队列大小 



Executor		总接口 ,只定义了execute()执行线程方法

ExecutorService extends Executor 子接口 ,定义了shutdown()关闭 submit()等方法

```
abstract class AbstractExecutorService implements ExecutorService
实现了ExecutorService的方法
```



class ThreadPoolExecutor extends AbstractExecutorService

```
void execute(Runnable command){}	//执行Runnable线程,无返回值
```



```
<T>Future<T> submit(Callable<T> task)	//执行Callable线程 ,有返回值
```



```
Executors工具类(工厂模式),返回不同类型的线程池
定义了new线程池的方法
Executors.newFixedThreadPool(10);
```



SingleThreadExecutor

FixedThreadPool

WorkStealingPool

CachedThreadPool

ScheduledThreadPool



使用Executors去创建，而是通过ThreadPoolExecutor的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。 说明：

Executors创建线程池对象的弊端
        1）FixedThreadPool和SingleThreadPool:
  允许的请求队列长度为Integer.MAX_VALUE，可能会**堆积大量的请求**，从而导致OOM。
        2）CachedThreadPool:
  允许的创建线程数量为Integer.MAX_VALUE，可能会**创建大量的线程**，从而导致OOM。







### 队列

一个缓冲的工具，当没有足够的线程去处理任务时，可以将任务放进队列中，以队列先进先出的特性来执行工作任务

核心线程满了，进队列，队列也满了，创建新线程，直到达到最大线程数，之后再超出，会进入拒绝rejectedExecution







# 垃圾回收机制



传统的 C/C++语言，需要程序员负责回收已经分配内存。

* 显式回收垃圾回收的缺点：

1）程序忘记回收，从而导致内存泄露，降低系统性能。

2）程序错误回收程序核心类库的内存，导致系统崩溃。

 

* Java由JRE在后台自动回收不再使用的内存

1）可以提高编程效率。

2）保护程序的完整性。

3）其开销影响性能。==Java 虚拟机必须跟踪程序中有用的对象==，确定哪些是无用的。

 

* 回收机制

1）==只回收堆内存里的对象==空间,不回收栈内存数据

2）不回收物理连接，如数据库连接、IO、Socket

3）无法控制回收执行时间 ,可以通过 System.gc()或者 Runtime.getRuntime().gc()来请求回收

4）==将对象的引用变量设置为 null，暗示可以回收==

5）回收任何对象之前，总会先调用它的 finalize 方法 ,但==不要主动调用finalize== ，应该交给垃圾回收机制调用



# 类的加载机制



实例类型在实例化后，才开始占用内存
静态变量在编译的时候,变量名会被编译到 pe文件里去，运行的时通过**文件偏移和内存偏移来相对映射**，编译时变量已经以**基址+内存偏移**的方式存储了，但里面的值无意义。当第一次被调用才被初始化
但实例方法和变量的内存是在运行时分配的，所以地址(内存的偏移)无法固定。静态方法无法调用实例方法和变量 ,实例方法可以调用静态方法和变量。





类方法执行时 ,对象还未创建 ,所以==类方法不能被this调用==

在类方法中调用实例方法 ,将优先执行完所有实例方法 ,所以==在类方法中可以调用实例方法==







# Object 6个方法



```
public boolean equals(Object) 	比较地址
public native int hashCode() 	获取哈希码 	是native Method,不是用java实现的方法
public String toString()
public final native Class getClass() 		获取类结构信息 
protected void finalize() throws Throwable 	垃圾回收前执行的方法
protected native Object clone() throws CloneNotSupportedException 	克隆
public final void wait() throws InterruptedException 	多线程等待
public final native void notify() 			唤醒
public final native void notifyAll() 		唤醒所有等待线程
```



# 修饰符



==重写的访问修饰符只能比父类大==

![img](image.assets/wps3-1603184298721.jpg) 





## native



```
/**
 * Indicates that a field defining a constant value may be referenced
 * from native code.
 *
 * The annotation may be used as a hint by tools that generate native
 * header files to determine whether a header file is required, and
 * if so, what declarations it should contain.
 *
 * @since 1.8
 */
@Documented
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.SOURCE)
public @interface Native {}
```

native是**java调用非java代码的接口**

定义Native Method时 ,**并不需要提供实现** ,其实现体将由**非java语言在外面实现****



```
 //Native Method的声明更像是描述非java代码在java中的大致模样
 public class IHaveNatives
   {  native public void Native1( int x ) ;
      native static public long Native2() ;
      native synchronized private float Native3( Object o ) ;
      native void Native4( int[] ary ) throws Exception ; }
```



==native可以与所有修饰符连用，除abstract== ,与abstract的无实现相违背



==native method可以返回任何java类型，包括非基本类型==，而且同样可以进行异常控制。这些方法的实现体可以制一个异常并且将其抛出。当native method接收到非基本类型 ,如Object时，可以访问非基本类型的内部，**但这将使native method依赖于所访问的java类的实现**。可以在一个native method的本地实现中访问所有的java特性，但会导致依赖于所访问的java特性的实现，这远不如使用java特性方便



native method不会对其他类调用这些本地方法产生任何影响，调用者甚至不知道它所调用的是一个本地方法。JVM将控制调用本地方法的所有细节。
如果含有本地方法的类被继承，**子类会继承这个本地方法并且可以用java重写**，本地方法被fianl标识，继承后不能被重写。
本地方法扩充了jvm ,在sun的java的并发实现中，许多与操作系统的接触点都用到了本地方法，使java能够超越java运行时的界限。



==JVM怎样使Native Method跑起来==
当类第一次被使用时，这个类的字节码会被加载到内存。在这个被加载的字节码入口 ,维持着该类所有方法描述符的list，这些方法描述符包含：方法代码存于何处，有哪些参数，修饰符等等。
native修饰符将有一个指向该方法的实现的指针。这些实现在一些DLL文件内，它们会被操作系统加载到java程序的地址空间。当带有本地方法的类被加载时，其相关的DLL并未被加载，因此指向方法实现的指针并不会被设置。**当本地方法被调用之前，这些DLL才会被加载**，这是通过调用java.system.loadLibrary()实现的。



# 对象克隆



## 实现方式

* ==实现 Cloneable 接口==并重写 Object 类中的 clone()方法；

* 实现 Serializable 接口，通过对象的==序列化和反序列化==，可以实现深度克隆,更重要的是支持==泛型限定==



## 深度/浅度克隆

浅度拷贝即直接赋值，拷贝的只是原始对象的引用地址，在堆中仍然共用一块内存。而深度拷贝为新对象在堆中重新分配一块内存，所以对新对象的操作不会影响原始对象。

要将可变对象和不可变对象相互转换，或者需要==操作新对象的时候不影响原始对象，用深度拷贝== ==copy-on-write==原则就是利用深度拷贝来实现的

 

## hutool克隆



CopyOptions定义了克隆规则		setIgnoreNullValue忽略null

```
BeanUtil.copyProperties(来源,目标, CopyOptions.create().setIgnoreNullValue(true).setIgnoreError(true));
```







# 客户端存储数据3种方法



* cookie

  会失效 ,下次请求cookie会被携带一起发送 ,不适合存储大量数据

* sessionStorage

  页面关闭则失效



* localStorage

  浏览器缓存被清空则失效





# 内存泄漏和溢出

内存溢出	程序在申请内存时没有足够的内存，比如申请了integer,但存了long才能存下的数

内存泄露	程序在申请内存后，无法释放已申请的内存空间



# &和&&

* 共同点

&和&&都可以用作逻辑与运算符

* 不同点	&：两边的操作数或表达式都会参与计算。&&：左边 false 时，不再计算 ,具有==短路效果== ,效率高







# WEB

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





# SpringBoot

 

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



# 单点登录、域用户、常规登录、AD域

1、单点登录  

（1）啥是单点登录？

用户只需要登录一次就可以访问所有相互信任的应用系统。（Single Sign On，简称为 SSO）

（2）解决啥问题？

各个server拿到同一个ID，都能有办法检验出ID的有效性、并且能得到ID对应的用户信息。

（3）咋实现？

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

 



















