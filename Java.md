[TOC]



# String

 

经常对字符串进行修改会引起很大的内存开销。

String对象建立之后不能再改变，对于不同字符串，需要不同String对象来表示。



## String空构造器

 

调用构造器创造string，性能低下且内存开销大

==string内容相同，Java认为它们代表同一个String对象==

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

==不存在返回-1	空字符串返回0==

 

对于空字符串的下标获取,首先调用这个方法

```java
public int indexOf(String str) {
  return indexOf(str, 0);}
```



对于不指定下标的indexof()方法,默认分配0的初始下标

```java
public int indexOf(String str, int fromIndex) {
  return indexOf(value, 0, value.length,
      str.value, 0, str.value.length, fromIndex);}
```



```java
public int indexOf(String str, int fromIndex) {
  return indexOf(value, 0, value.length,
      str.value, 0, str.value.length, fromIndex);}
```



对所有indexof方法的处理最终都是在调用这个方法

```java
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
    return fromIndex; }}
```





## 编码转换



newString("".getBytes("GB2312"), "ISO-8859-1");





## String编译优化

字符串对象创建有两种形式，

1.字面量形式，String str = "aa"	存进字符串常量池

2.new 							存进堆

 

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

序列化能够将实例对象的状态信息写入到字节流,从而通过socket进行传输或者持久化.然后通过反序列化恢复对象状态



比如Session 对象，当有 10 万用户并发访问出现 10 万个 Session 对象，内存吃不消，于是 Web 容器就会把一些 seesion 先序列化到硬盘中，等要用了，再把保存在硬盘中的对象还原到内存中。

当两个进程在进行远程通信时，彼此可以发送各种类型的数据。无论是何种类型的数据，都会以二进制序列的形式在网络上传送。发送方需要把这个 Java 对象转换为字节序列，才能在网络上传送；接收方则需要把字节序列再恢复为 Java 对象。

 

## 两种实现方法

* 实现Serializable接口，所有的序列化将会自动进行

* 实现**Externalizable接口** ,在writeExternal方法中进行手工指定所要序列化的变量

 

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



## 接口和抽象类的区别



* 相同
  * 抽象类和接口均包含抽象方法，类必须实现所有的抽象方法，否则是抽象类

  * 抽象类和接口都不能实例化，他们位于继承树的顶端，用来被其他类继承和实现

* 不同
  * 接口中只能定义全局静态常量，不能定义变量。抽象类中可以定义常量和变量
  * 接口中所有的方法都是abstract。==抽象类中不全为abstract==
  * **接口不能定义构造方法**,抽象类中可以有构造方法，但不能用来实例化，而在子类实例化是执行，完成属于抽象类的初始化操作。
  * 单继承多接口

 

接口可以继承接口

==抽象类可以实现接口==，抽象类可以继承实体类,可以有main方法



==最主要区别还是设计理念==

*  接口  实现类仅仅是实现了接口定义的约定。接口定义了“做什么”，而实现类负责“怎么做”，体现了功能和实现分离的原则。**接口和实现是has-a **

* 抽象类体现了一种继承关系，目的是复用代码，抽象类中定义了各个子类的相同代码，可以认为父类是一个实现了部分功能的“中间产品”，而子类是“最终产品”。**父类和子类是is-a**



# Lambda

函数式编程思想 ,只关注做什么 ,不关注怎么做

**lambda不是语法糖**

语法糖 :写法不同 ,实现原理相同 ,如增强for循环



(类型 参数1, 类型 参数2....) -> {代码}

**参数没有则留空,有则必须是函数式接口@FunctionalInterface**



* FunctionalInterface	起到注释作用,标明这是函数式接口,避免其他人写入方法
  * 函数式接口只有一个抽象方法
  * default方法某默认实现，不属于抽象方法
  * 接口重写了Object的公共方法也不算入内



lambda无法单独出现，需要函数式接口来盛放，lambda方法体是函数接口的实现



​     lambda表达式可以访问给它传递的变量，访问自己内外部的变量。但访问外部变量时,变量引用不可变,即一旦定义后，在后面就不能再随意修改引用

​	实例变量存在堆中，而局部变量存在栈，**lambda(匿名内部类) 会在另一个线程中执行**。如果在线程中要直接访问一个局部变量，可能线程执行时该局部变量已经被销毁了，而 **final 类型的局部变量在 Lambda 表达式(匿名类) 中其实是局部变量的一个拷贝**



==在lambda中，this不是指向lambda表达式产生的那个对象，而是声明它的外部对象==



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



## Comparator



```
//res是二维数组,需求是根据res[]进行排序,不考虑第二个[]
int[][] res = new int[n][2];
Arrays.sort(res,new Comparator<int[]>(){
            @Override
            public int compare(int[] o1, int[] o2) {
                return o1[0] - o2[0];         } });
                
//进一步简化	省略参数类型
 Arrays.sort(res, (o1, o2) -> o1[0] - o2[0]);
 
//Comparator类的内部实现，还有一个 comparing 方法
Arrays.sort(res, Comparator.comparingInt(o -> o[0]));
```









# Stream

得益于Lambda所带来的函数式编程，引入全新的Stream概念，用于解决集合类库的弊端。

Stream是集合元素的函数模型，不是集合，也不是数据结构，**其本身不存储任何元素或地址**,只是在原数据集上定义了一组操作。也不会改变原有数据

Stream流不保存数据，Stream操作是尽可能惰性的，即每当访问到流中的一个元素，才会在此元素上执行这一系列操作。



```
list.stream().filter(s -> s.startsWith("张"))
                .filter(s -> s.length() == 3)
                .forEach(System.out::println);
```

 stream().forEach



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



* 筛选与分片
  * forEach	调用的多线程，调用线程池要额外耗费时间,**无序**
  * forEachOrdered     按**原顺序**输出
  * limit    取前几个
  * distinct   去重
  * skip     跳过前几个
  * filter        将一个流转换成另一个子集流
* 映射

  * map      将流中的元素映射到另一个流

```java
Stream<String> stream = list.stream().map(e -> e.substring(2));
```

* flatMap



* 终止操作

  * allMatch 检查是否匹配所有元素 方法参数为断言型接口
  * anyMatch 检查是否匹配所有元素 方法参数为断言型接口
  * noneMatch 检查是否没有匹配所有元素 方法参数为断言型接口
  * findFirst 返回第一个元素 无方法参数
  * findAny 返回当前流的任意元素 无方法参数
  * count 返回流中的元素总个数 无方法参数
  * max 返回流的最大值 无方法参数
  * min 返回流中的最小值 无方法参数





* 归约

  * count   统计个数

  * concat   合并流 ,Stream的静态方法

    ```java
     Stream<String> result = Stream.concat(streamA, streamB);
    ```

  * reduce  将流中的元素反复结合起来，得到一个值。

```java
List<Integer> list1 = Arrays.asList(1,2,3,4,5,6,7,8,9,10);
Integer reduce = list1.stream().reduce(11, (x, y) -> x + y);
reduce ： 66
```



* 收集	方法参数为Collector。Collector由Collectors中的toList()，toSet(),toMap(Function(T,R) key,Function(T,R) value)等静态方法实现。
  * toList()
  * toMap()
  * toSet()

```
list.stream().collect(Collectors.toList());
```



* 分组
  *  Collectors.groupingBy()

```java
 public static void main(String[] args) {
        List<User> users = Arrays.asList(new User("张三", 19, 1000),
                new User("张三", 58, 2000),
                new User("李四", 38, 3000),
                new User("赵五", 48, 4000)
        );
        Map<String, List<User>> collect3 = users.stream().collect(Collectors.groupingBy(x -> x.getName()));
        System.out.println(collect3);

输出：{李四=[User{name='李四', age=38, salary=3000}], 张三=[User{name='张三', age=19, salary=1000}, User{name='张三', age=58, salary=2000}], 赵五=[User{name='赵五', age=48, salary=4000}]}
```







## parallelStream



​	通过默认的ForkJoinPool，提高多线程任务的速度，默认线程数量等于运行计算机上的处理器数量

​	**Java8为ForkJoinPool添加了一个通用线程池，用来处理没有被显式提交到任何线程池的任务**。当调用Arrays类上添加的新方法时，自动并行化就会发生。



## 使用原则



* 单核cpu，不用parallel stream

* 低数据量场景（size<=1000），stream不如iterator，但是这些任务运行时间都低于毫秒，效率的差距不明显
* 大数据量时（szie>10000），stream 的处理效率会高于 iterator，特别是使用了并行流，cpu恰好将线程分配到多个核心的条件下（parallel stream 底层使用的是 JVM 的 ForkJoinPool，分配线程很玄学,受引CPU环境影响，当没分配到多个cpu核心时，加上引用 forkJoinPool 的开销，运行效率可能还不如普通的 Stream）


* ==含有装箱类型，先转成对应的数值流==，减少频繁的拆箱、装箱的性能损失





# Iterator



```
 for (Iterator iterator = set.iterator(); iterator.hasNext();) {
            String string = (String) iterator.next();
            System.out.println(string);
        }
```



* forEachRemaining(Consumer<? super E> action)：为每个剩余元素执行给定的操作,直到所有的元素都已经被处理或行动将抛出一个异常

* hasNext()  如果迭代器中还有元素，则返回true。

* next()：返回迭代器中的下一个元素

* remove()：删除迭代器新返回的元素。



==Iterator只能单向移动==

==Iterator.remove()是唯一能安全地在迭代过程中修改集合==；如果在迭代过程中以任何其它的方式修改集合将会产生未知的行为。而且每调用一次next()方法，remove()方法只能被调用一次，如果违反这个规则将抛出一个异常。



## ListIterator

继承于Iterator接口,功能更强大,

只能用于各种List类型的访问。可以通过调用listIterator()方法产生一个指向List开始处的ListIterator, 还可以调用listIterator(n)方法创建一个一开始就指向列表索引为n的元素处的ListIterator。

==双向移动==

==产生迭代器前一个和后一个元素的索引==

用set()替换它访问过的最后一个元素.

用add()在next()方法返回的元素之前或previous()方法返回的元素之后插入一个元素.



## Iterator和ListIterator区别



* 只有ListIterator有add()
* ListIterator有hasPrevious()和previous()方法，可以**逆序遍历**
* ListIterator用nextIndex()和previousIndex()指定索引位置
* ListIterator在遍历同时set()修改





# 集合



List 以特定索引来存取元素，可有重复元素。

Set 不能存放重复元素（用对象的 equals()方法来区分元素是否重复）

Map 保存键值对映射，映射关系可以是一对一或多对一。

Set 和 Map 容器都有基于哈希存储和排序树（红黑树）的两种实现版本，基于哈希存储的版本理论存取时间复杂度为O(1)，而基于排序树版本的实现在插入或删除元素时会按照元素或元素的键（key）构成排序树从而达到排序和去重的效果。



==Collections类==

​	专门用来操作集合类 ，提供一系列静态方法实现对各种集合的搜索、排序、线程安全化等操作





## 集合和数组的比较

数组不是面向对象的，存在明显的缺陷，集合完全弥补了数组的一些缺点，比数组更灵活，可大大提高软件的开发效率.不同的集合框架类可适用于不同场合

1）**数组的效率高**于集合

2）**数组能存放基本数据类型**和对象，而集合类中只能放对象

3）**数组容量固定**，集合类容量动态改变

4）==数组无法判断实际有多少元素，length只告诉了array的容量==

5）集合有多种实现方式和不同的适用场合，而不像数组仅采用顺序表方式。

6）集合以类的形式存在，具有封装、继承、多态等类的特性，通过简单的方法和属性调用即可实现各种复杂操作，提高效率









### ArrayList 和 LinkedList 的区别和联系

* 相同点：
  * 都实现List接口，有序、不唯一

* 不同点：
  * ArrayList数组实现,长度可变，查询效率高
    * 初始容量10,满了时新建一个2倍容量的数组,并把原数组复制过去,实现扩容
  * LinkedList 采用双向链表存储方式。插入、删除元素时效率高





## HashSet



1）哈希表的查询快，时间复杂度为 O（1）

2）HashMap、Hashtable、HashSet 这些集合采用的是哈希表结构，需要用hashCode

3）系统类已经覆盖了hashCode 方法,**自定义类放入hash类集合，必须重写hashcode**。不重写调用的是Object的hashcode,比较地址。



==向哈希Set中添加数据的原理==

首先计算hashCode，得到一个位置用来存放当前对象，如在该位置没有一个对象存在的话，直接增加进去。

如果在该位置有对象，进行**equals，如果false,再进行一次散列，将该对象放到散列后计算出的新地址里**。如果true，那么集合认为集合中已经存在该对象了，不会再将该对象增加到集合中了。

**hashCode决定数据在表中的存储位置，而equals判断是否存在相同数据。**







## TreeSet



元素不允许重复且==有序(自然顺序)==,底层存储结构是二叉树,**中序遍历保证有序**，存入元素时需要**和树中元素进行对比**,保证不重复

3）可以通过 Comparable(外部比较器)和 Comparator(内部比较器)来指定比较策略，实现了 Comparable 的系统类可以顺利存入 TreeSet。自定

义类可以实现 Comparable 接口来指定比较策略。

4）可创建 Comparator 接口实现类来指定比较策略，并通过 TreeSet 构造方法参数传入。这种方式尤其对系统类非常适用。









## List的多态

List list = new ArrayList() 与 ArrayList alist = new ArrayList()

List接口有多个实现类，现在你用的是ArrayList，也许哪一天需要换成LinkedList或者Vector等等，这时你只要改变一行就行      

这就是面向接口编程,LinkedList和ArrayList都实现了List接口

在List list时,并不知道实例化了Linked还是Array,但是这个list都是要去add()的

这也是多态的体现,父类引用指向子类对象





## HashMap



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





### 按位与2次幂容量

取余:xxx%16  不断在做除法,效率低,并且负数取余仍是负数,还需要转为正数

按位与: 	hash&(length-1)

​			(length-1)  1111

​			(hash)   1001

​			     =1001

当length-1不为全1,即length不为2的幂,将出现0,而0的部分按位与永远为0

将导致0的桶永远放不进



### 7 HashMap死锁隐患



![img](image.assets/wps1-1600480677581.jpg) 

原先:	3->5->7

resize:	7à3

多线程环境下,可能同时3->7	7->3,出现循环

![img](image.assets/wps2-1600480677595.jpg) 

当查询时就会出现死锁



==可以通过精心设计的一组object实现dos(拒绝服务攻击)==

大量的object的HashCode相同,使得它们被存放在同一个桶中,使得HashMap退化为链表,而链表的查询复杂度O(n)

 

### 8 HashMap

#### hash方法

  static final int hash(Object key) {

​    int h;

​    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16); }

jdk7中,容易出现低位相同,高位不同的hash	如1101……….1111

​											1001………1111

将高位与地位异或(不进位的加法),能够减少碰撞的概率

 

#### resize方法

  Node<K,V> loHead = null, loTail = null;

  Node<K,V> hiHead = null, hiTail = null;

扩容时,将原链表拆为两个高低位的链表

比如16个桶,哈希码11111…….11101

​							 1111	=1101

扩容32个桶,					11111	=11101

扩容后第一位只能是0或1,并且桶中的元素被分配在了1xxx和0xxx两个新桶中,元素保持原先的顺序.而保持了顺序就降低了多线程中,顺序调换出现的死锁问题

 

Map.getOrDefault((Object key, V defaultValue),取不到key时,将返回默认的value



### 底层数组创建机制

8之前，创建对象时就创建了数组

8之后，**首次调用put**才创建



### hash相等时

会产生hash碰撞

key值相同则替换，否则加到后面		**比较key用equals**





### 成员变量



* MAXIMUM_CAPACITY = 1 << 30    最大容量

* UNTREEIFY_THRESHOLD = 6    链表的值小于6则会从红黑树转回链表

* MIN_TREEIFY_CAPACITY** = 64	超过这个值，才能进行树形化

* Node<K, V>[] table	table用来初始化(必须是二的n次幂)

* Set<Entry<K, V>> entrySet	存放缓存

* size	元素个数

* int modCount	修改次数

* int threshold		下次扩容的临界值，（容量\*负载因子)

* float loadFactor	哈希表的负载因子



### 基本类型不能做为键值



* 泛型约束为Object类型
  * map.put(1, “Java”)，实际上是将1进行了自动装箱操作,变为了 Integer类型

* 引用数据类型重写了HashCode()和 equals()两个方法，能==保证key的唯一性==





## LinkedHashMap



==有序,LinkedHashMap记录了添加数据的顺序==,底层存储结构是哈希表+**链表**，链表记录了添加数据的顺序



## TreeMap





### TreeMap按Value排序

TreeMap底层是根据红黑树的数据结构构建的，默认是key的自然排序

==将TreeMap的EntrySet转换成list，然后使用Collections.sor排序==

```
 Map<String,String> map = new TreeMap<String,String>();

List<Entry<String, String>> list = new ArrayList<Entry<String, String>>(map.entrySet());
Collections.sort(list,new Comparator<Map.Entry<String,String>>() {
//升序排序
public int compare(Entry<String, String> o1, Entry<String, String> o2) {
return o1.getValue().compareTo(o2.getValue()); } });

```







## TreeMap 和 TreeSet 在排序时如何比较元素？Collections工具类中的 sort（）方法如何比较元素？

TreeSet	实现Comparable接口，该接口提供了比较元素的**compareTo()方法**，当插入元素时会**回调**该方法比较元素的大小。

TreeMap	实现Comparable接口,根据键对元素进行排序

Collections工具类的sort方法有两种重载的形式，

第一种要求实现Comparable 接口以实现元素的比较

第二种要求传入第二个参数，参数是 Comparator 接口的子类型（重写compare方法），其实就是是通过接口注入算法，也是对回调模式的应用











## PriorityQueue 优先队列

1.5新特性,是**基于优先堆**的一个无界队列，元素通过默认自然排序或者通过提供的Comparator在队列实例化的时排序。

优先队列**不允许空值**，而且不支持non-comparable（不可比较）的对象，比如用户自定义的类。优先队列要求使用Java Comparable和Comparator接口给对象排序，并且在排序时会按照优先级处理其中的元素。

优先队列的**头是最小元素**。排序同样，随机地取其中一个。当我们获取队列时，返回队列的头对象。

优先队列**容量是不受限制**，但在创建时可以指定初始容量。向优先队列增加元素的时候，**队列会自动扩容**。

PriorityQueue是**非线程安全**的，所以Java提供了==PriorityBlockingQueue==（实现BlockingQueue接口）用于Java多线程环境。













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



## 通道

最早,cpu需要建立若干io接口来进行io操作,这将导致cpu被占用

后来引入了**DMA**直接存储器访问 ,cpu将io操作交给DMA进行 ,DMA先向cpu申请资源 ,然后形成**DMA总线** ,不过总线的过多也会导致总线冲突,最后影响性能

而**channel类似于DMA总线** ,是一个完全独立的处理器 ,专门用于处理io ,不需要向cpu申请资源

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



## 直接/非直接缓冲区

非直接	allocate()分配缓冲区 ,缓冲区在**jvm**

直接	allocateDirect(),在**物理内存**

jvm对于直接缓冲区,会尽量避免使用中间缓冲区进行数据的读写,而是直接在缓冲区上进行io操作

分配**直接**缓冲区需要**更大的成本** ,也**不会被gc回收** ,会影响应用程序的内存

![image-20200809134909724](.\image.assets\image-20200809134909724.png)

对于非直接缓冲区 ,物理磁盘的数据先读取至内核地址空间,再被copy到jvm内存,最后到应用程序

点开allocate()方法也可以看到返回的是heap堆缓冲

![image-20200809142318554](.\image.assets\image-20200809142318554.png)



![image-20200809141028612](.\image.assets\image-20200809141028612.png)

对于直接缓冲区,应用程序通过物理内存映射文件直接与物理磁盘交换数据 省略了copy的步骤

* 缺点
  * ==直到gc释放了应用程序与物理内存映射文件的引用,才会销毁链接==,映射文件的引用有可能延迟数十秒才会被回收
  * 直接缓冲区的建立与销毁是成本高,只适合长时间的连接,大文件的传输
  * ==直接缓冲区只能用ByteBuffer==





## 关闭

使用IO流往往需要多次使用try/catch

如果在一个try/catch中关闭多个流,将会导致关闭时其中一个流 ,抛出异常,程序中断,之后的流将不再被关闭!!!

需要一条一条的try/catch





## 非直接传输



```java
FileInputStream in = new FileInputStream("1.jpg");
        FileOutputStream out = new FileOutputStream("2.jpg");

        FileChannel inChannel = in.getChannel();
        FileChannel outChannel = out.getChannel();
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        while (inChannel.read(buffer) != -1) {
            //切换至读模式
            buffer.flip();
          //将缓冲区数据写入通道
            outChannel.write(buffer);
            buffer.clear();
        }
        
        out.close();
        in.close();
        inChannel.close();
        outChannel.close();
```



## 直接传输



* NonReadableChannelException
  * MapMode只有READ_WRITE模式,而在outChannel并没有授予StandardOpenOption.READ权限,导致文件不可读

* FileAlreadyExistsException
  * StandardOpenOption.CREATE_NEW在文件存在时,会直接报错,CREATE模式则覆盖源文件

```java
FileChannel inChannel = FileChannel.open(Paths.get("1.png"), StandardOpenOption.READ);
//CREATE_NEW,文件不存在则创建,存在则报错
//CREATE,不存在则创建,存在则覆盖
FileChannel outChannel = FileChannel.open(Paths.get("2.png"), StandardOpenOption.WRITE, StandardOpenOption.READ, StandardOpenOption.CREATE_NEW);

//内存映射文件
MappedByteBuffer inmap = inChannel.map(FileChannel.MapMode.READ_ONLY, 0, inChannel.size());
MappedByteBuffer outMap = outChannel.map(FileChannel.MapMode.READ_WRITE, 0, inChannel.size());

//直接对缓冲区进行数据的读写
byte[] bytes = new byte[inmap.limit()];
inmap.get(bytes);
outMap.put(bytes);

inChannel.close();
outChannel.close();
```



## 通道传输

底层也是用的直接传输



```java
FileChannel inChannel = FileChannel.open(Paths.get("1.png"), StandardOpenOption.READ);
FileChannel outChannel = FileChannel.open(Paths.get("2.png"), StandardOpenOption.WRITE, StandardOpenOption.READ, StandardOpenOption.CREATE_NEW);

inChannel.transferTo(0, inChannel.size(), outChannel);
//        outChannel.transferFrom(inChannel, 0, inChannel.size());

inChannel.close();
outChannel.close();
```













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



## 线程/工作内存/主内存



每个线程都有一个独立的工作内存，用于存储线程私有的数据

**Java内存模型规定变量都存储在主内存**，主内存是共享内存区域，所有线程都可以访问

线程**对变量的操作在工作内存中进行**（线程安全问题的根本原因）

* 首先要将变量从主内存拷贝到工作内存

* 然后对变量进行操作，再将变量写回主内存

* **因此不同的线程间无法访问对方的工作内存**，==线程间的通信(传值)必须通过主内存来完成== ,多个线程对一个共享变量进行修改时，都是对自己工作内存的副本进行操作，相互不可见。主内存中共享变量的结果是不可预知的





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





# 反射



在**运行状态**中，对于任意一个实体类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性；这种动态获取信息以及动态调用对象方法的功能称为java语言的反射机制。



**反射是在程序运行过程中，操作对象**。可以提高程序扩展性和复用性，可以解耦。





## Class



Class 类是反射的入口，用于获取与类相关的各种信息和方法

**每个类也可看做是一个对象**，有共同的Class来存放类的结构信息，能够通过相应方法取出相应信息：类名、属性、方法、构造方法、父类和接口



| asSubclass(Class<U>  clazz)              | 把传递的类的对象转换成代表其子类的对象 |
| ---------------------------------------- | -------------------------------------- |
| getClassLoader()                         | 获得类的加载器                         |
| getClasses()                             | 返回数组，包含公共类和接口类的对象     |
| getDeclaredClasses()                     | 返回数组，包含类和接口类的对象         |
| forName(String  className)               | 根据类名返回类的对象                   |
| getName()                                | 获得类的完整路径名字                   |
| newInstance()                            | 创建类的实例                           |
| getPackage()                             | 获得类的包                             |
| getSimpleName()                          | 获得类的名字                           |
| getSuperclass()                          | 获得当前类继承的父类的名字             |
| getInterfaces()                          | 获得当前类实现的类或是接口             |
| .class                                   | 获取当前对象的类                       |
|                                          |                                        |
| isAnnotation()                           | 如果是注解类型则返回true               |
| isArray()                                | 如果是一个数组类则返回true             |
| isEnum()                                 | 是枚举类则返回true                     |
| isInstance(Object obj)                   | 是该类的实例则返回true                 |
| isInterface()                            | 是接口类则返回true                     |
|                                          |                                        |
| getAnnotation(Class<A>  annotationClass) | 获得与参数类型匹配的公有注解对象       |
|                                          |                                        |
|                                          |                                        |
|                                          |                                        |











## Field

代表类的成员变量。**成员变量（字段）和成员属性是两个概念**。User类中有name变量，则它有name字段。如果**没有get/setName，就没有name属性**。**如果有get/set,不管字段是否存在，都认为有这个属性**



| getField(String name)          | 获得1个public字段  |
| ------------------------------ | ------------------ |
| getFields()                    | 获得所有public字段 |
| getDeclaredField(String  name) | 获得某个字段       |
| getDeclaredFields()            | 获得所有字段       |
| setAccessible(true)            | 忽略访问权限修饰符 |



getDeclaredField()访问非public字段时,会报错

```
can not access a member of class *** with modifiers "private"
```

通过SetAccessible(true)忽略访问修饰符



```java
    @Test
    public void testSet() throws Exception {
        User user = new User("张三", 23, "220202202002022222");
        Class<? extends User> userClass = user.getClass();
      
        Field idNumberField = userClass.getField("idNumber");
        // set方法：给对象的字段设置值。需要传入当前被操作的user对象
        idNumberField.set(user, "123456");
    }
```





## Method



| **方法**                           | **用途**                                 |
| ---------------------------------- | ---------------------------------------- |
| invoke(Object obj, Object... args) | 传递object对象及参数调用该对象对应的方法 |
| getName                            | 获取方法名                               |
| SetAccessible(true)                | 暴力反射，忽略访问权限修饰符             |

 

Invoke方法的用处：SpringAOP在切面方法执行的前后进行某些操作，就是使用的invoke方法。

| **方法**                                                    | **用途**               |
| ----------------------------------------------------------- | ---------------------- |
| getMethod(String name,  Class...<?> parameterTypes)         | 获得该类某个公有的方法 |
| getMethods()                                                | 获得该类所有公有的方法 |
| getDeclaredMethod(String name,  Class...<?> parameterTypes) | 获得该类某个方法       |
| getDeclaredMethods()                                        | 获得该类所有方法       |

 





 



## Constructor

 

| **方法**                                            | **用途**                               |
| --------------------------------------------------- | -------------------------------------- |
| getConstructor(Class...<?>  parameterTypes)         | 获得该类中与参数类型匹配的公有构造方法 |
| getConstructors()                                   | 获得该类的所有公有构造方法             |
| getDeclaredConstructor(Class...<?>  parameterTypes) | 获得该类中与参数类型匹配的构造方法     |
| getDeclaredConstructors()                           | 获得该类所有构造方法                   |
| newInstance(Object... initargs)                     | 根据传递的参数创建类的对象             |



* Class类的newInstance()只能无参构造
* Constructor的newInstance()能传递构造参数

```java
Class<Session> sessionClass = Session.class;
Constructor<Session> declaredConstructor = sessionClass.getDeclaredConstructor();
declaredConstructor.setAccessible(true);
Session session2 = declaredConstructor.newInstance();
```



* Constructor类违背了Java的一些思想
  * 可以无视private的构造方法,强行创建对象
  * 破坏了单例模式的规则

 



## 注解



注解本身并不起任何作用,只作为标识	通过反射来获取注解,再根据注解的参数执行业务



* 作用分类：
  * 编写文档：通过代码中标识的注解生成文档（Swagger）
  * 代码分析：通过代码里的注解对代码进行分析（逻辑判断）
  * 编译检查：通过代码里对应的注解让编译器实现基本的编译检查（Override，Deprecated，FunctionalInterface）

JDK中预定义的一些注解

Override：检测该注解标识的方法是否继承自父类

Deprecated：标识方法、类、字段等已经过时，后续的版本可能会将其移除

SuppressWarnings：压制警告



### 元注解

==元注解用于描述注解的适用范围==



* @Target	作用范围
  * Type：作用于类
  * METHOD：作用于方法
  * FIELD：作用于字段
  * ElementType取值

* @Retention：描述注解被保留的阶段
  * RetentionPolicy.RUNTIME：当前描述的注解，会保留到class字节码文件中，并被jvm读取到
* @Documented：描述注解是否被抽取到api文档中

* @Inherited：描述注解是否可以被继承

 



### 自定义注解



```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
    String name() default "lx";
    int value();
}
```



```
@MyAnnotation(123)
public class User{}
```



```java
public void testAnnotation() {
        Class<User> userClass = User.class;
        // 获取注解
        MyAnnotation myAnnotation = userClass.getAnnotation(MyAnnotation.class);
        // 注解不为空的时候进行处理
        if (myAnnotation != null) {
            // 获取打在User类上的注解的两个属性
            System.out.println(myAnnotation.name() + ":" + myAnnotation.value());
        }
    }
```



==注解本质上是一个接口，默认继承自Annotation接口==

* 如果定义了属性，在使用属性的时候需要给属性赋值。

* ==只有一个属性需要赋值，属性名称value，则可以省略value==

* 数组赋值时用{}封装

* ==属性中的返回值==类型有下列取值：
  * 基本数据类型
  * String
  * 枚举
  * 注解
  * 以上类型的数组



## 泛型擦除



```java
public void test() throws Exception {
    List<User> list= new ArrayList<>();
    Class<? extends List> listClass = list.getClass();
    Method add = listClass.getDeclaredMethod("add", Object.class);
  //通过invoke()在运行时向list插入Integer数据,避免了编译时的泛型检验
    add.invoke(list, 5);
    add.setAccessible(true);}
```







## 反射的使用场合和作用、及其优缺点



* 在编译时不知道该对象或类可能属于哪些类，程序只依靠运行时信息来发现该对象和类的真实信息。通过反射可以使程序代码访问装载到 JVM 中的类的内部信息

* 反射提高了 Java 程序的灵活性和扩展性，**低耦合**，提高自适应能力。它允许程序创建和控制任何类的对象，无需提前硬编码目标类

* Struts、Hibernate、Spring 在实现过程中都采用了该技术

* 反射基本上是**解释操作**，用于字段和方法接入时要远慢于直接代码。因此 Java 反射机制主要应用在对灵活性和扩展性要求很高的系统框架上。使用反射会模糊程序内部逻辑：程序人员希望在源代码中看到程序的逻辑，反射等绕过了源代码的技术，因而会带来维护问题。反射代码比相应的直接代码更复杂。







# JVM



## java代码的3个阶段



* .java	经过javac编译->	.class

* .class	经过ClassLoader类加载器->	class类对象
  * class对象包含字段,构造方法,成员方法信息

* class对象	创建对象->	对象













## 堆/栈/方法区



* 栈
  * 效率高.由操作系统自动分配释放,在硬件层级对栈提供支持：有专门的寄存器存放栈的地址，压栈出栈有专门的指令

  * 按先后定义的顺序依次压栈，**相邻变量的地址之间不会存在其它变量**。栈的内存地址生长方向与堆相反，由高到底，**后定义的变量地址低于先定义的变量**
  * 数据的生命周期**随函数的执行完成而结束**,==方法调用时进栈,执行结束出栈==
  * 存储==对象的引用,基本数据类型==
  * 3个部分：基本类型变量区、执行环境上下文、操作指令区(存放操作指令)

 

==栈是运行时单位，解决程序运行时的问题，堆是存储单位，解决数据存储==



* 堆 heap
  * 只有1个,被所有线程共享
  * 存储==对象==,对象包含与之对应的class信息.(class的目的是得到操作指令)
  * 堆的申请和释放工作由程序员控制，容易产生内存泄漏--己动态分配的堆内存未释放或无法释放



* 方法区/静态区
  * 只有1个,被线程共享
  * 存储==class文件信息,static变量==,等唯一的元素

  * 常量池,包含==基本类型和对象型的常量值==

    * Boolean,String及数组
    * ==小于127的Byte,Short,Integer,Long,Character== **不包括浮点数**

    



![image-20201024224300360](image.assets/image-20201024224300360.png)







## GC



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





## 内存泄漏和溢出

内存溢出	程序在申请内存时没有足够的内存，比如申请了integer,但存了long才能存下的数

内存泄露	程序在申请内存后，无法释放已申请的内存空间



# 类加载机制



```java
public class Singleton {
    //1.静态变量    调用了非静态的构造器  将优先加载非静态,跳过静态
    //4.构造方法结束 ,赋值
    private static Singleton instance = new Singleton();

    //3.构造方法    此时x=y=1
    public Singleton() {
        x++;
        y++;
    }

    private static int x;
    //5.静态变量赋值  此时x=1,y=0
    private static int y = 0;

    //2.非静态变量
    private int z = 1;

    public static Singleton getInstance() {
        return instance;
    }

    public static void main(String[] args) {
        //6.main方法体     结果x=1,y=0
        getInstance();
    }
}
```



```java
public class Singleton2 {

    //3.构造方法    此时x=y=1
    public Singleton2() {
        x++;
        y++;
    }

    //1.静态变量赋值  此时x=y=0
    private static int x = 0;
    private static int y;
    //2.静态变量调用构造方法
    //4.构造器返回值赋值
    private static Singleton2 instance = new Singleton2();

    public static Singleton2 getInstance() {
        return instance;
    }

    public static void main(String[] args) {
        //5.main方法体       结果x=y=1
        getInstance();
    }
}
```



实例类型在实例化后，才开始占用内存
静态变量在编译的时候,变量名会被编译到 pe文件里去，运行的时通过**文件偏移和内存偏移来相对映射**，编译时变量已经以**基址+内存偏移**的方式存储了，但里面的值无意义。当第一次被调用才被初始化
但实例方法和变量的内存是在运行时分配的，所以地址(内存的偏移)无法固定。静态方法无法调用实例方法和变量 ,实例方法可以调用静态方法和变量。





类方法执行时 ,对象还未创建 ,所以==类方法不能被this调用==

在类方法中调用实例方法 ,将优先执行完所有实例方法 ,所以==在类方法中可以调用实例方法==



## 静态代码块

```java
 static{ }
```

* 属于类
* 类被加载的时运行，只运行一次，优先于各种代码块以及构造函数
* 静态代码块**主动运行**,所以不能在方法体中
  * 静态方法是被动运行,通过类名或对象名访问
  * 普通方法是实例化后运行,通过对象访问

* 一般用于项目启动加载配置文件



## 构造代码块

```
{}
```

* 属于类

* 创建对象时调用，**每次创建对象都会调用**，优先于构造函数执行

* 不实例化对象，构造代码块不会执行

* 构造方法被重载,不能事先确定到底执行哪个,构造代码块却一定被执行

  

## 构造函数

* 属于类
* 命名为类名,不带返回值。普通函数可以和构造函数同名，但有返回值

* 主要用于在类的对象创建时定义初始化的状态。不能被void修饰,以区分其他有返回值的方法

* 不能被直接调用，必须通过new

* 默认先调用父类的无参构造



## 普通代码块

-    构造代码块是在**类中**定义的，
-    普通代码块是在**方法体中**定义的





## 各种类型变量的默认初始值

　　JVM 类加载机制中提到，类连接 （验证， 准备， 解析）中准备工作：

　　　　**为类的类变量（非对象变量）分配内存,初始值，准备类中每个字段、方法和实现接口所需的数据结构**

![](image.assets/1541314-20191003104706537-1884792960.png)





　　![img](image.assets/1541314-20191003105127516-1214242200.png)

# 实例化顺序



## 1、牢记：静态和非静态分开处理

　（1）使用到静态加载时，静态又分为： 静态变量， 静态代码块，按照书写顺序加载
　（2）非静态加载顺序： 按照非静态书写顺序加载 /执行
　（3）**静态方法，实例方法只有在调用的时候才会去执行**
　（4）当静态加载中遇到需要加载非静态的情况： **先加载非静态再加载静态**（因为非静态可以访问静态，而静态不能访问非静态）

```
public static Text t1 = new Text("t1");  
// 当加载静态变量是需要先加载构造器， 那就转为先加载所有非静态属性
```

## 2、静态变量声明  一定 放在使用前面



## 3、main是否第一句先执行

　　　　因为main方法虽然是一个特殊的静态方法，但是**还是静态方法**，此时**JVM会加载main方法所在的类，试图找到类中其他静态部分**，即首先会找main方法所在的类。



## 4、父类、子类加载顺序

　　1、父类的静态变量和静态块赋值（按照声明顺序）
　　2、自身的静态变量和静态块赋值（按照声明顺序）
　　3、main方法
　　3、父类的成员变量和块赋值（按照声明顺序）
　　4、父类构造器赋值
　　5、自身成员变量和块赋值（按照声明顺序）
　　6、自身构造器赋值
　　7、静态方法，实例方法只有在调用的时候才会去执行





# 创建对象方式

1、new 语句

2、反射,调用 java.lang.Class 或者 java.lang.reflect.Constructor类的 newInstance()实例方法。

3、调用对象的clone()方法。

4、运用反序列化手段，调用 java.io.ObjectInputStream 对象的readObject()方法。

(1)和(2)都会显式地调用构造函数

(3)是在内存上已有对象的影印，不会调用构造函数

(4)是从文件中还原类的对象，也不会调用构造函数。



# == hash equals



(1)如果两个对象相同（equals 方法返回 true），则hashCode相同；

(2)如果两个对象的 hashCode 相同，它们并不一定相同







**==是关系运算符，equals()是方法**

* ==    
  * 基本类型，比较值
  * 引用类型，比较地址
  * ==不能比较没有父子关系的两个对象==

* equals() 
  * 系统类一般已经覆盖了 equals()，比较的是内容
  * 自定义类如果没有覆盖 equals()，将调用父类equals（**Object 的==和 equals 比较的都是地址**）

 









# Object 6个方法



```java
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





# 6种数据存储方式



1．寄存器（register）。**最快,位于处理器内部**。数量有限，由编译器根据需求进行分配。**无法直接控制**

2．堆栈（stack）。位于**随机访问存储器RAM**（random-access memory），**通过“堆栈指针”从处理器获得直接支持**。==堆栈指针向下移动，分配新内存；向上移动，释放内存==。创建程序时，**编译器必须知道存储在堆栈内所有数据的大小和生命周期**，因为它必须生成相应的代码，以便上下移动堆栈指针。这一**约束限制了程序的灵活性**，所以对象并不存储于其中。

3．堆（heap）。通用内存池,位于**随机访问存储器RAM**，用于存放对象。堆不同于堆栈的好处是：**编译器不需要知道要分配多少存储区域，也不必知道数据的生命周期**。**灵活性高,效率低**

4．静态存储（static storage）。位于**随机访问存储器RAM**,这里的“静态”是指“在固定的位置”（尽管也在 RAM 里）。静态存储里存放程序运行时一直存在的数据。你可用关键字 Static 来标识一个对象的特定元素是静态的。

5．常量存储（constant storage）。**通常直接存放在程序代码内部**，这样做安全，因为它们永远不会被改变。有时在嵌入式系统中，常量本身会和其它部分隔离开，在这种情况下，存放在只读存储器ROM（read-only memory）。

6．非RAM存储（non-RAM storage）。**数据存活于程序之外**，不受程序控制，在程序没有运行时也可以存在。例如**“字节流对象”和“持久化对象”**。即使程序终止，它们仍可以保持自己的状态。这种存储方式的技巧在于：把对象转化成可以存放在其它媒介上的事物，在需要时，可恢复成常规的、基于 RAM 的对象。









# 对象克隆



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





## 拆箱装箱JDK1.5



装箱：基本数据类型->包装器类型      valueOf方法

拆箱：包装器类型->基本数据类型      xxxValue方法



==基本数据类型不是面向对象（没有属性、方法）==，实际使用时存在很多的不便（比如集合的元素只能是Object）。所以需要包装类



```java
short s1 = 1; 
s1 = s1 + 1;		//错误,s1 + 1为int,需要强转
s1 += 1;				//正确,被优化为s1 = (short)(s1 + 1)
```













# 异常

![](image.assets/异常.png)

两个子类:异常,错误

==异常能被程序本身可以处理，错误是无法处理==



*  Exception
  * 必须进行处理的异常，否则不能编译通过

* RuntimeException
  * 编译器不会检查**它，没有用try-catch/throws也会编译通过

* Error:是**程序无法处理的错误**，表示运行应用程序中较严重问题。
  * 大多数错误与代码编写者执行的操作无关。例如，Java虚拟机运行错误/OutOfMemoryError。错误发生时，Java虚拟机（JVM）一般会选择线程终止。















# 修饰符



==重写的访问修饰符只能比父类大==

![img](image.assets/wps3-1603184298721.jpg) 



## 父类成员在子类的访问权限

Public继承方式    不改变父类的访问权限

protected          private不变 ,其余都变为protected

private            都改成private



## 子类成员在外部的访问权限

**父类的private     只有父类能访问**

private方式继承的非private成员    只有子类的成员函数能访问 ,子类的子类/外部不能访问

protected方式继承的非private成员 	只有子类及子类的子类(非private继承) 能访问







## final



* 不能修饰构造方法。**修饰的类不能被继承，方法不能被重写**
* 修饰基本类型变量，值不能改变

* **修饰引用类型变量，栈内存中的引用不能改变**，但堆内存中对象的属性值可以改变

```java
 final Dog dog = new Dog("aa");
 dog.name = "bb";//正确
 dog = new Dog("cc");//错误
```





## static

一般在需要实现以下两个功能时使用静态变量：

1.在对象之间共享值时

2.方便访问变量时



* 生命周期不同。
  * 成员变量随对象的创建而存在，随着对象的被回收而释放
  * 静态变量随类的加载而存在，随着类的消失而消失

* 调用方式不同。
  * 成员变量只能被对象调用。
  * 静态变量可以被对象调用，还可以被类名调用。

* 数据存储位置不同。
  * 成员变量在堆
  * 静态变量在方法区的静态区

* 内存拷贝不同
  * 成员变量可以在内存有多个拷贝
  * 静态变量只能1个



static并不代表不可修改,它是能够时刻保持最新的值的静态变量

==静态是指不会随着函数的调用/退出发生变化==。下次调用时，这个值与上次调用一致

==static final全局常量才不能修改==

















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







# &和&&

* 共同点

&和&&都可以用作逻辑与运算符

* 不同点	&：两边的操作数或表达式都会参与计算。&&：左边 false 时，不再计算 ,具有==短路效果== ,效率高







### float f=3.4是否正确?

答:不正确。3.4 是双精度数，将双精度型（double）赋值给浮点型（float）

属于下转型 ,会造成精度损失，因此需要强

制类型转换 float f =(float)3.4; 或者写成 float f =3.4F;。





## sql.Date和util.Date 

1） java.sql.Date 是 java.util.Date 的子类，是一个包装了毫秒值的瘦包装器，允许JDBC将毫秒值标识为 SQL DATE 值

2）java.sql.Date是针对 SQL 语句使用的，只包含日期而没有时间部分。

以下操作中容易出现不易被发现的 BUG：获得一个 JAVA 里的日期对象。 从数据库里读取日期 试图比较两个日期对象是否相等。如果毫秒部分丢失，本来认为相等的两个日期对象用 Equals 方法可能返回 false。sql.Timestamp比util.Date类精确度要高







# 匿名内部类可不可以继承或实现接口？

匿名内部类是没有名字的内部类,不能继承其它类,但内部类可以作为接口,由另一个内部类实现.

1、由于匿名内部类没有名字，所以它没有构造函数,所以它必须完全借用父类的构造函数来实例化，换言之：匿名内部类完全把创建对象的任务交给了父类去完成。

2、在匿名内部类里创建新的方法没有太大意义，但它可以通过覆盖父类的方法达到神奇效果

3、匿名内部类没有名字，所以无法向下强转，持有对一个匿名内部类对象引用的变量类型一定是它的直接或间接父类类型。





## 静态内部类和内部类有什么区别

静态内部类不需要有指向外部类的引用。但非静态内部类需要持有对外部类的引用。

静态内部类可以有静态成员(方法，属性)，而非静态内部类则不能有静态成员(方法，属性)。

静态内部类只能访问外部类的静态成员。非静态内部类能够访问外部类的静态和非静态成员。

实例化方式不同：

1) 静态内部类：不依赖于外部类的实例，直接实例化内部类对象

2) 非静态内部类：通过外部类的对象实例生成内部类对象









