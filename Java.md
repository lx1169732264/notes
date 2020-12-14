# String



## 空构造器

 

调用构造器创造string，性能低下且内存开销大

==string内容相同，Java认为它们代表同一个String对象==

```java
//底层维护字符数组
private final char value[];

/** Note that use of this constructor is unnecessary since Strings are immutable.
JDK源码上的注解: 注意，由于字符串是不可变的，因此不需要使用此构造函数。
用构造函数创建string是无意义的,并且性能低
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

==对于equals相等的字符串，在常量池中永远只有一份，在堆中有多份==

 

```java
String a="hello2";

String b="hello"+2;

System.out.println((a==b));

输出为：true。因为 ”hello” +2在编译期就已经被优化成 “hello2”，因此在运行期变量 a 和 b 指向的是同一个对象(在字符串常量池)

String a="hello2";

String b="hello";

String c=b+2;

System.out.println((a==c));

输出为 false。由于有符号引用的存在，所以String c=b+2不会在编译期间被优化，不会把 b+2 当做字面量处理，因此生成的对象是保存在堆上的。所以a和c不是指向同一个对象。
```





## String/Builder/Buffer区别

* 相同点：
  * 都用 final 修饰，不能派生子类
* 操作的相关方法类似
  
* 不同点：
  * String只读，内容不能改变，StringBuffer和StringBuilder类表示的字符串对象可以直接进行修改，在修改的同时地址值不会发生改变。
  * StringBuilder是JDK1.5新特性，和StringBuffer的方法完全相同，线程不安全,性能高。
  * ==String、Buffer、Builder 类型不同，无法用 equals()方法比较内容==
  * Builder线程不安全



## String为什么final

* 若允许被继承，则其高度的被使用率可能会降低程序的性能

* 为了安全。JDK中的核心类比如 String，内部很多方法的实现都不是 java 编写的，只是==调用操作系统的 API，也就是本地方法调用==，如果这种类可以被继承并重写，将导致操作系统面临风险





# 序列化



将实例对象的状态信息写入字节流,通过socket进行传输或者持久化.然后通过反序列化恢复对象状态



## 两种实现方法

* 实现Serializable接口，所有的序列化将会自动进行

* 实现**Externalizable接口**,在writeExternal方法中进行手工指定所要序列化的变量

 

## Serializable原理



==有AB两个类，B含有指向A类对象的引用，进行实例化时,系统会将a的数据复制一份到b，从文件中恢复对象(重新加载到内存)，对象a同时存在两份==

　　

1.保存到磁盘的对象获得序列号

2.保存对象时，先检查该对象是否被保存了

3.如果保存过，查看是否有修改,否则更新对象

以上的步骤,序列化机制解决了对象引用的问题

 

* 注意事项
  * 序列化，**只对状态进行保存，不管对象的方法**
  * **父类实现序列化，**子类自动序列化
  * 对象的**实例变量引用其他对象，**引用对象自动序列化
  * 并非所有的对象都可以序列化
    　　* 安全方面:对象拥有private，public等字段，在写入文件，或者进行RMI传输时,private等域不受保护
      * 资源分配方面:比如socket/thread类，如果可以序列化，进行传输或者保存，也无法对他们进行重新的资源分配

 

## serialversionUID

目的是序列化对象版本控制。如果在新版本中这个值修改了，新版本就不兼容旧版本，反序列化时会抛出InvalidClassException异常。如果修改较小，比如仅仅是增加了一个属性，我们希望向下兼容，那就不用修改；如果我们删除了一个属性，或者更改了类的继承关系，必然不兼容旧数据，这时就应该手动更新版本号

 

serialVersionUID = 1L意义:

有两种生成方式：一个是默认的1L， 一个是根据类名、接口名、成员方法及属性等来生成一个64位的哈希字段，如： -8940196742313994740L

不指定 serialVersionUID将导致添加或修改类中的任何字段时, 已序列化类将无法恢复



# 继承



超类和子类 成员变量名称相同

并不会重写父类的成员变量,**子类中将有两个相同名称的变量**

```java
public class A {  
    public int m = 1;  
 
    public void view(){
        System.out.println(this.m);
    }  
}  
 
public class B extends A{  
 
    public int m = 2;  
 
/**
 *  @Override
 *  public void view(){
 *      System.out.println(this.m);		//2
 *      System.out.println(super.m);	//1
 *  }
 */  
 
    public static void main(String[] args) {
        B b = new B();
        b.view();		//1
    }  
}  
```





 

# 接口



## default 默认方法



实现接口需要实现其所有的抽象方法,当接口加入新方法时，我们就需要对项目重新编写

使用**default**修饰， 定义**方法体**。**子类会默认实现default** ，避免修改代码



这个default是jdk8新关键字，**和访问限定修饰符“default”不是一个概念**，与switch中的default功能完全不同

实际上是**public default**,省略了public



与抽象类的不同：抽象类更多的是提供一个模板，子类之间的某个流程大致相同，仅仅是某个步骤可能不一样（模板方法设计模式），这个时候使用抽象类，该步骤定义为抽象方法。而default关键字是用于扩展







以前，一个接口如果增减一个方法，则所有实现它的类都需要改动。default可以少改一些代码——这反而违背了Java的一个核心思想：尽量把问题暴露在编译时

前面说过，接口的核心用法，是在多态时暴露恰当的method。如果只是直接用子类的实例进行调用，那么直接写方法就好，不用给自己头上套一个牢笼。而如果一个接口增加一个方法，则说明所有实现它的类都必须思考一个问题：该如何实现这个方法。即使大多数思考的结果是，直接给个空实现，这个思考过程也是必须的。体现在写代码上，就是每个子类的文件都需要去改一下。如果考虑编程效率问题，那么这也是该由编辑器或者IDE去解决的（实际上这很难吗？一定要我把IDEA里那个功能的名称说出来吗？），而非Java——**Java就是一种用编程效率来换取代码质量的语言**。

default method的出现，让程序员又多了一个犯错的大门类，而没有解决什么实质性问题，更违背了Java的初心







## 静态方法



接口的静态方法不会被实现类所实现

**只用于内部调用**



两个接口定义**相同静态方法**，实现类实现这两个接口，并不会产生错误，编译器通过**反射**来区分是哪个接口下的方法

两个接口定义**相同非静态方法**，并且一个实现类同时实现了这两个接口，那么必须在实现类中重写默认方法，否则编译失败。

**静态方法调用    类名.方法**   通过**反射**来区分哪个接口下的方法

​    **非静态         对象.方法** 





Java 8让接口可以声明并实现多个静态方法，以及静态常量，这其实没有太大改变

本来在Java 7，静态方法就不是属于实例的，而是属于类的。实例的方法才有继承关系，可以选择重写或沿用。而静态方法虽然能用实例调用，但更应该用类来调用，也不能重写

由于Java在文件级作用域只能有类或接口，所以把静态方法和变量放在类中

Utils/Constants里面只有静态。这是对文件级作用域不能放方法和常量的一种妥协，本来这些方法和常量应该不属于任何一个类的。这里的类只起到了命名空间的作用

在Java 7这两种类有时会用抽象类。但这有一个问题，虽然不能被实例化，但可以被继承。为了避免被继承，有时会不辞辛劳地用一个final类，实现一个private构造器，来做进一步的保护。在Java 8以后，用接口来实现，也是一个思路

这可以视为接口在抽象类手里抢的第一单生意，但这不算什么





## 函数式接口



**只有一个抽象方法**

### @FunctionalInterface

只在编译期起作用，如@Override注解。编译期会强制检查该接口是否符合函数式接口的条件，不符合则会报错。**即使不使用，只要满足定义也是函数式接口**。

![image-20200826214004498](image.assets/image-20200826214004498.png)



### Supplier



仅一个无参的方法： T get() 

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





## Lambda



函数式编程思想,只关注做什么,不关注怎么做

**lambda不是语法糖**

语法糖 :写法不同,实现原理相同,如增强for循环



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



### 省略规则



参数类型可以省略,但只能都省略或都不省略

参数只有一个,小括号能省略

大括号内语句只有一条,大括号/分号/return关键词能省略



### 延迟执行



先合并字符串,再判断level==1,决定要不要执行方法 

```java
public class Demo01Logger {
    private static void log(int level, String msg) {
        if (level == 1) {
            System.out.println(msg);}}
  
    public static void main(String[] args) {
        String msgA = "Hello";
        String msgB = "World";
        log(1, msgA + msgB);}}
```



优化后,先判断,后执行字符串合并

```java
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







### 方法引用 ::



```
print(e -> System.out.println(e));
```

这个lambda只是接收了参数,并将它打印,而打印的方法有现成的System.out.println

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



### Comparator



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













## 接口和抽象类的区别



* 相同
  * 抽象类和接口均包含抽象方法，类必须实现所有的抽象方法，否则是抽象类

  * **抽象类和接口都不能实例化**，位于继承树的顶端，用来被其他类继承和实现

* 不同
  * 接口中只能定义全局静态常量，不能定义变量。抽象类中可以定义常量和变量
  * 接口中所有的方法都是abstract。==抽象类不全为abstract==
  * 抽象类的方法可以有private、protected、prublic和package local四种类型，接口只有public类型
  * **接口不能定义构造方法/成员变量**,抽象类中可以有构造方法，但不能用来实例化
  * 单继承多接口
  * 子类和抽象类是is-a关系，和接口是like-a关系

 



接口可以继承接口

==抽象类可以实现接口==，抽象类可以继承实体类,可以有main方法



==最主要区别还是设计理念==

*  接口  实现类仅仅是实现了接口定义的约定。接口定义了“做什么”，实现类负责“怎么做”，体现了功能和实现分离的原则。**接口和实现是has-a **

*  抽象类体现继承关系，目的是复用代码，定义了各个子类的相同代码，可以认为父类是一个实现了部分功能的“中间产品”，而子类是“最终产品”。**父类和子类是is-a**



本质上，**抽象类的设计主要是保护一个类不被实例化，或者允许一个类包含未实现的抽象方法。而接口的设计主要是为了多态**



接口更适合来实现多态，仅暴露了必要的method,高度封装



一个抽象类或普通的父类引用（Reference），也同样可以代表不同的子类，但有两个问题。一是需要被多态引用的子类，必须是这个家族谱系的，不能是其它家族的。这在单继承下，会有很大限制，导致一些情况无法实现。二是暴露了过多的method给调用方。指定为Closable，仅暴露了一个close()方法，其它都是隐藏的。





## Java 9的private method改变了什么？



Java 9中，接口再次增强，可以实现private method和private static method。

其实，要理解设计思路很简单。既然可以写实现代码了，那么是不是有一些共用的代码需要封装、又不希望暴露给外部？那就加private吧！以前只有抽象方法的时候，自然没有这需求。而Java 8增加static method和default method后，没有一起增加这个功能，简直就是设计上的缺失！

在有了这俩以后，接口和抽象类的区别，只剩下了构造器、成员变量、以及单继承。

单继承是Java相对C++的一个重要改进。以前（C++）把夫妻都计入族谱时，发现好乱，一夫多妻嘛；现在（Java）只着重记男丁，族谱顿时就简明多了。

而可以多重继承的接口，本来因为没有构造器、成员变量、方法实现（都是妻妾），所以没有多重继承的烦恼。现在有了default method，烦恼就来了——这是开历史的倒车。

再往下发展，假如接口也支持成员变量、没有构造器也能构造实例，那就成为了支持多重继承的**新式类**，和老式类**分庭抗礼**——忽然有一种聊Python的感觉——那么问题来了，老式类要不要考虑全盘废弃呢？



## 抽象类与接口的完美配合



抽象方法会被下游用户利用多态来调用->接口



此外，即使一个项目的所有抽象方法都是会被多态调用、是在接口声明的，也同样可能存在抽象类，因为它们有一个完美配合。很多时候，一个接口的多个方法，在实现时是分层级的。在这个类只能实现其中一部分，在其子类才能实现剩余部分，这时父类就需要是抽象方法

比如，有一个接口叫『可繁衍的』，其中有两个方法，一是『进食』，二是『性交』。『人类』implements了这个接口，但是只能实现其中『进食』这个方法。而其子类『男人』和『女人』，分别实现了不同的『性交』方法。这种情况下，『人类』必然是无法实例化的抽象类









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

  * concat   合并流,Stream的静态方法

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















# 集合



List 以特定索引来存取元素，可重复

Set 不能存放重复元素（equals()区分是否重复）

Map 保存键值对映射，映射关系可以是一对一或多对一

Set 和 Map 容器都有基于哈希存储和排序树（红黑树）的两种实现版本，基于哈希存储的版本理论存取时间复杂度为O(1)，而基于排序树版本的实现在插入或删除元素时会按照元素或元素的键（key）构成排序树从而达到排序和去重的效果。



![](image.assets/image-20201207212855338.png)



集合主要分为两种：Collection 和 Map。Collection 是 List 和 Set 接口的父接 口；ArrayList 和 LinkedList 是 List 的实现类；HashSet 和 TreeSet 是 Set 的实现类； LinkedHashSet 是 HashSet 的子类。HashMap 和 TreeMap 是 Map 的实现类； LinkedHashMap 是 HashMap 的子类







需要多个线程操作同一个容器，那么可以通过工具类 Collections 中的 synchronizedList 方法将其转换成线程安全的容器后再使 用（这其实是装潢模式最好的例子，将已有对象传入另一个类的构造器中创 建新的对象来增加新功能）。



## 集合VS数组



数组不是面向对象，集合更灵活

1）**数组的效率高**,==能存放基本类型==,容量固定

4）==数组无法判断实际有多少元素，length只告诉了array的容量==

5）集合有多种实现方式和不同的适用场合，数组仅采用顺序表方式

6）集合以类的形式存在，具有封装、继承、多态等类的特性，通过简单的方法和属性调用即可实现各种复杂操作，提高效率







## List、Map、Set区别



List与Set实现公共父接口Collection,都是单列元素的集合

Set不允许重复，不能有两个相等的对象 ，所以Set的add()返回boolean

Set无序，只能以Iterator遍历



List表示有先后顺序的集合

==一个对象可以被反复存储进List中，相当于集合中有多个索引指向了这个对象==





Map与List和Set不同，它是双列的集合

可以获得所有的key的集合，可以获得所有value的集合，还可以获得key和value组合成的Map.Entry对象的集合







## Set



### HashSet



1）哈希表的查询快，O（1）

3）系统类已经覆盖了hashCode 方法,**自定义类放入hash类集合，必须重写hashcode**。不重写调用的是Object的hashcode,比较地址



==向哈希Set中添加数据的原理==

首先计算hashCode，得到一个位置用来存放当前对象，如在该位置没有一个对象存在的话，直接增加进去。

如果在该位置有对象，进行**equals，如果false,再进行一次散列，将该对象放到散列后计算出的新地址里**。如果true，那么集合认为集合中已经存在该对象了，不会再将该对象增加到集合中了

**hashCode决定数据在表中的存储位置，equals判断是否存在相同数据**







### TreeSet



元素不允许重复且==有序(自然顺序)==,底层存储结构是**二叉树**,**中序遍历保证有序**，存入元素时需要**和树中元素进行对比**,保证不重复









## List



### AbstractList



```java
public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {
  	//都是直接抛出异常
		public E set(int index, E element) {  throw new UnsupportedOperationException();}
    public void add(int index, E element) {   throw new UnsupportedOperationException();}
    public E remove(int index) {    throw new UnsupportedOperationException();}
}
```



#### SubList



```java
//ArrayList
public List<E> subList(int fromIndex, int toIndex) {
    subListRangeCheck(fromIndex, toIndex, size);
    return new SubList(this, 0, fromIndex, toIndex);
}

    static void subListRangeCheck(int fromIndex, int toIndex, int size) {
        if (fromIndex < 0)    throw new IndexOutOfBoundsException("fromIndex = " + fromIndex);
        if (toIndex > size)    throw new IndexOutOfBoundsException("toIndex = " + toIndex);
        if (fromIndex > toIndex)  throw new IllegalArgumentException("fromIndex(" + fromIndex +") > toIndex(" + toIndex + ")");
    }
```





```java
//AbstractList的内部类SubList,与ArrayList没有继承关系，无法强转
class SubList<E> extends AbstractList<E> {
    private final AbstractList<E> l;
    private final int offset;
    private int size;

  	//把原列表的部分属性赋值给SubList
  	//SubList并没有重新创建一个List，而是直接引用了原有的List（父类的视图）
    SubList(AbstractList<E> list, int fromIndex, int toIndex) {
        if (fromIndex < 0)   throw new IndexOutOfBoundsException("fromIndex = " + fromIndex);
        if (toIndex > list.size())   throw new IndexOutOfBoundsException("toIndex = " + toIndex);
        if (fromIndex > toIndex)     throw new IllegalArgumentException("fromIndex("+fromIndex +")>toIndex("+toIndex+")");
        l = list;
        offset = fromIndex;
        size = toIndex - fromIndex;
        this.modCount = l.modCount;	//重点
//对父/子List做的非结构性修改（non-structural changes），都会影响到彼此
//对子List做结构性修改，操作同样会反映到父List上
//对父List做结构性修改，会抛出异常ConcurrentModificationException
      
//如果需要对subList作修改，又不想动原list。那么可以创建subList的深度拷贝
```





### ArrayList



少量数据时效率高

初始大小10,扩容为1.5倍





### CopyOnWriteArrayList



底层维护了transient(序列化) volatile(可见)的数组

```java
private transient volatile Object[] array;
```









### LinkedList



大量数据时效率高









### Vector



内部实现与ArrayList一致

**在方法中进行了同步,线程安全**

**扩容为2倍**,适合数据量大的存储





### 检索、插入、删除效率



* ArrayList和Vector中，从指定的位置检索，或在末尾插入、删除都是O(1)

  * 其他位置为O(n-i)，n为元素的个数，i下标。需要执行(n-i)个对象的位移操作

* LinkedList
  * 插入、删除集合中任何位置都O(1)
  * 索引时O(i),i下标







### List的多态



List list = new ArrayList() 与 ArrayList alist = new ArrayList()

List接口有多个实现类，现在你用的是ArrayList，也许哪一天需要换成LinkedList或者Vector等等，这时你只要改变一行就行      

这就是面向接口编程,LinkedList和ArrayList都实现了List接口

在List list时,并不知道实例化了Linked还是Array,但是这个list都是要去add()的

这也是多态的体现,父类引用指向子类对象







### Arrays.asList



**不建议使用于基本数据类型的数组**

该方法将数组与List列表链接起来：当更新其一个时，另一个自动更新



==得到的List长度不可变==,不支持add()、remove()、clear()等,会抛出java.lang.UnsupportedOperationException

```java
public static <T> List<T> asList(T... a) {
        return new ArrayList<>(a);	//是Arrays的内部类ArrayList(继承自AbstractList),没有实现add()、remove(),调用将直接抛出异常
    }

==体现的是适配器模式，只是转换接口，后台的数据仍是数组==
private static class ArrayList<E> extends AbstractList<E>implements RandomAccess, java.io.Serializable{
        private final E[] a;

        ArrayList(E[] array) { a = Objects.requireNonNull(array); }
```



转化列表后只是用来遍历，用Arrays.asList()

List还要添加或删除元素，new java.util.ArrayList，然后循环添加元素





## Map



| 集合类            | Key            | Value          | Super       | 说明            |
| ----------------- | -------------- | -------------- | ----------- | --------------- |
| Hashtable         | 不允许null     | 不允许null     | Dictionary  | 安全            |
| ConcurrentHashMap | **不允许null** | **不允许null** | AbstractMap | 锁分段技术(CAS) |
| TreeMap           | 不允许null     | **允许null**   | AbstractMap | 不安全          |
| HashMap           | **允许null**   | **允许null**   | AbstractMap | 不安全          |



### HashMap



* 是Hashtable的轻量级实现（非线程安全的实现）

* 允许空键/值

* 继承了AbstractMap,实现了Map，克隆，序列化接口

```java
HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {
//AbstractMap已经实现过了Map接口，而HashMap又继承了AbstractMap，这样使得HashMap已经实现了Map接口，然而HashMap又再次去实现了Map接口,这是JDK中多此一举的失误
  
AbstractMap<K,V> implements Map<K,V> {
```



* HashMap把Hashtable的contains方法去掉了，改成containsvalue和containsKey。因为contains方法容易让人引起误解



![](image.assets/image-20201206204317626.png)





#### 按位与2次幂容量



hash&(length-1)	==	hash%length

* 取余不断在除法,效率低,并且负数取余仍是负数,还需要转为正数

* 按位与: 	hash&(length-1)

​			(length-1)  1111

​			(hash)   1001

​			     =1001

当length-1不为全1,即length不为2的幂,将出现0,而0的部分按位与永远为0

将导致0的桶永远放不进







#### 成员变量



```java
DEFAULT_INITIAL_CAPACITY = 1 << 4;	//默认初始容量
MAXIMUM_CAPACITY = 1 << 30    最大容量

//红黑树长度小于6则会转回链表,红黑树的log(n)，log(8) = 3,log(6)≈ 2.6
//链表平均查找长度是 log(n/2)，log(8) = 4，log(6)=3
UNTREEIFY_THRESHOLD = 6    
TREEIFY_THRESHOLD = 8;
MIN_TREEIFY_CAPACITY = 64	//超过这个值，才能树化,否则只是扩容

Node<K, V>[] table
Set<Entry<K, V>> entrySet	存放缓存
size	元素个数(kv数量，不是数组的长度)
int modCount	//修改次数
int threshold		扩容的阈值（容量*负载因子)
float loadFactor	负载因子(太小导致数组的利用率低)
```



#### 构造



```java
public HashMap() {	this.loadFactor = DEFAULT_LOAD_FACTOR; }
public HashMap(int initialCapacity) {   this(initialCapacity, DEFAULT_LOAD_FACTOR);}

public HashMap(int initialCapacity, float loadFactor) {
      if (initialCapacity < 0)   throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
      if (initialCapacity > MAXIMUM_CAPACITY)  initialCapacity = MAXIMUM_CAPACITY;
//Float.isNaN()判断非法数值,,经过多次运算后float可能会出现非法情况，如除数为0.0,在Float中NaN是引用类型，每个NaN都是不同对象
      if (loadFactor <= 0 || Float.isNaN(loadFactor)) throw new IllegalArgumentException("Illegal load factor:" +loadFactor);
        this.loadFactor = loadFactor;
  //tableSizeFor()判断initialCapacity是否为2^n,不是则扩大initialCapacity
        this.threshold = tableSizeFor(initialCapacity);
    }

		//修正容量为2^n
    static final int tableSizeFor(int cap) {
      //-1为了防止cap已经是2^n
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }


//参数为Map的构造
public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }

    final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
        int s = m.size();//获取map元素个数
        if (s > 0) {
            if (table == null) {
              //+1.0F 让计算出的size向上取整,获得更大的容量,避免resize(可能刚创建,元素就均匀分布到每个数组,触发扩容)
                float ft = ((float)s / loadFactor) + 1.0F;
                int t = ((ft < (float)MAXIMUM_CAPACITY) ?(int)ft : MAXIMUM_CAPACITY);
              //得到的t大于阈值->更新阈值
                if (t > threshold)   threshold = tableSizeFor(t);
            }
          	//已初始化,元素个数>阈值->扩容
            else if (s > threshold)    resize();
          
          //遍历放入新map
            for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
                K key = e.getKey();
                V value = e.getValue();
                putVal(hash(key), key, value, false, evict);
```



#### get



```java
public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }

final Node<K,V> getNode(int hash, Object key) {
  //first存放对应下标链表的第一个元素
        Node<K,V>[] tab; Node<K,V> first, e; 
  			int n; K k;
  //复制table到tab,判空
        if ((tab = table) != null && (n = tab.length) > 0 &&(first = tab[(n - 1) & hash]) != null) {
          
          //下标第一个元素的key就是要找的key
            if (first.hash == hash && ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
          
          //下标第一个元素不是,寻找下一个并赋值给e
            if ((e = first.next) != null) {
              	//红黑树的方法
                if (first instanceof TreeNode)		 return ((TreeNode<K,V>)first).getTreeNode(hash, key);
              //遍历链表,直到下个节点不存在  
              do {
                //寻找对应key的位置
                    if (e.hash == hash &&((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
```



##### hash



```java
    static final int hash(Object key) {
        int h;
//null==key -> 返回0		null不能调用hashCode(),所以给了默认值0		->		hashMap支持Null的Key
//hashCode()与hashCode()右移16的异或运算(相同为0,否则1)	->	高16位与低16位都参与异或,减少碰撞概率
//当length很小 -> 高位全0,低位不同	->	容易碰撞
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```



```shell
#按位异或,计算hash
1111 1111 1111 1111 1111 0000 1110 1010	// key.hashCode()结果
0000 0000 0000 0000 1111 1111 1111 1111	// 右移16位
1111 1111 1111 1111 0000 1111 0001 0101 // hash

#对于(length - 1) & hash		假设length 16
0000 0000 0000 0000 0000 0000 0000 1111 // 15 length-1
1111 1111 1111 1111 0000 1111 0001 0101
0000 0000 0000 0000 0000 0000 0000 0101 // 5


#假设不进行16位右移操作
1111 1111 1111 1111 1111 0000 1110 1010 //key.hashCode()结果
0000 0000 0000 0000 0000 0000 0000 1111 // 15
0000 0000 0000 0000 0000 0000 0000 1010 // 10

1010 0110 0011 1111 1111 0000 1110 1010	//高位不同,低位相同
0000 0000 0000 0000 0000 0000 0000 1111 // 15
0000 0000 0000 0000 0000 0000 0000 1010 // 10
#容易碰撞
```



#### put



![](image.assets/HashMap.png)



* 先通过hash值计算出key映射到哪个桶
  * 桶上没有碰撞，直接插入
  * 如果出现碰撞
    * 用红黑树处理冲突，调用红黑树的方法插入数据
    * 链式方法插入,插入后判断长度是否达到树化临界值
    * 键重复，更新value

* 插入后如果size大于阈值threshold，扩容



* 底层数组创建机制
  * 7，创建对象时就创建了数组
  * ==8，首次put才创建==




```java
public V put(K key, V value) {  return putVal(hash(key), key, value, false, true); }

//onlyIfAbsent=true,不改变现有的值		evict=false,table为新建状态
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; 
  			//n数组长度		i key的hash值
  			int n, i;
  
  			//table为空/长度为0	->	resize()实例化一个数组,赋值给tab		tab.length赋值给n
        if ((tab = table) == null || (n = tab.length) == 0)        n = (tab = resize()).length;
  
  		//hash&(length-1)计算当前key的下标		获取当前下标的Node,赋值给p
  		//p==null -> 当前数组下标位置没有存储值 -> 创建新Node,插入数组
        if ((p = tab[i = (n - 1) & hash]) == null)       tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
          
          //寻找键值对所在node的位置
          //元素的hash==传入的hash,并且key相等			将node赋值给e(更新旧值)
            if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))	  e = p;
            else if (p instanceof TreeNode)   e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            
          	//key不相等,并且不是treeNode
          	else {
              	//binCount记录循环次数,判断是否需要树化
                for (int binCount = 0; ; ++binCount) {
                  
                  	//到达链表末尾
                    if ((e = p.next) == null) {
                      	//将数据插入表尾.next
                        p.next = newNode(hash, key, value, null);
                      
                      	//循环次数>=树化边界值-1	->	树化
                        if (binCount >= TREEIFY_THRESHOLD - 1)  	treeifyBin(tab, hash);
                        break;
                    }
                  
                  	//当前位置的key与要put的key相同,跳出循环
                    if (e.hash == hash &&((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
          
          //e不为空,则之前找到了键值对的Node
            if (e != null) {
                V oldValue = e.value;
              
              	//onlyIfAbsent=false,需要改变现有的值,将传入参数value赋值给e,返回e的旧值
                if (!onlyIfAbsent || oldValue == null)       e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
  
  			//更新修改次数
        ++modCount;
        if (++size > threshold)     resize();
        afterNodeInsertion(evict);
        return null;
```



##### 7 死锁隐患

 

原先:	3->5->7

多线程resize时,可能同时3->7	7->3,出现循环,查询3/7时出现死锁

==可以通过一组HashCode相同的object实现dos(拒绝服务攻击)==,被存放在同一个桶中,HashMap退化为链表,查询复杂度O(n)

 

put时元素超过了容量*负载因子,触发resize()和rehash()，将原数组重新hash到新数组，在多线程的环境下，其他元素也在put，如果hash值相同，可能出现同时在同一数组下用链表表示，造成闭环，导致在get时会出现死循环





##### resize



==链表长度>=8，数组长度<64,也会扩容==



```java
Node<K,V> loHead = null, loTail = null;

Node<K,V> hiHead = null, hiTail = null;
```

扩容时,将原链表拆为两个高低位的链表

比如16个桶,哈希码11111…….11101

​							 1111	=1101

扩容32个桶,					11111	=11101

扩容后第一位只能是0或1,并且桶中的元素被分配在了1xxx和0xxx两个新桶中,元素保持原先的顺序.而保持了顺序就降低了多线程中,顺序调换出现的死锁概率



每次扩容都是翻倍，与原来的(n-1)&hash的结果相比，只是多了一个二进制位，所以节点要么在原来的位置，要么就被分配到 原位置+原容量 这个位置。















#### remove



```java
//返回被删除key的value
public V remove(Object key) {
        Node<K,V> e;
        return (e = removeNode(hash(key), key, null, false, true)) == null ? null : e.value;
    }

//matchValue=true	仅当值相同时移除		movable=true	移除时移动其他节点
final Node<K,V> removeNode(int hash, Object key, Object value, boolean matchValue, boolean movable) {
        Node<K,V>[] tab; Node<K,V> p; 
  			int n, index;
  // (tab = table) != null  把hash桶赋值给tab，并且判断tab是否为nul
  // (n = tab.length) > 0 获取tab的长度，赋值给n，判断n是否大于0
  // (p = tab[index = (n - 1) & hash]) != null 根据hash计算索引位置，赋值给index, 并从tab中取出该位置的元素，赋值给p，并判断p!=nul
        if ((tab = table) != null && (n = tab.length) > 0 &&(p = tab[index = (n - 1) & hash]) != null) {
            
          	//进到这里,说明table不为空,并且key对应table下标的链表不为空
          	Node<K,V> node = null, e; K k; V v;
          
          	//链表表头就是待删除元素
            if (p.hash == hash &&((k = p.key) == key || (key != null && key.equals(k))))	  node = p;
            
          	//遍历链表
          	else if ((e = p.next) != null) {
                if (p instanceof TreeNode)   node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
                else {
                    do {
                        if (e.hash == hash && ((k = e.key) == key ||(key != null && key.equals(k)))) {
                            node = e;
                            break;
                        }
                        p = e;
                    } while ((e = e.next) != null);
                }
            }
          
            if (node != null && (!matchValue || (v = node.value) == value ||(value != null && value.equals(v)))) {
                if (node instanceof TreeNode)
                    ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
                
              	//node == p	->	说明node是第一个节点,将下一个节点赋值给当前下标(删除)
              	else if (node == p)    tab[index] = node.next;
                else    p.next = node.next;
                ++modCount;
                --size;
                afterNodeRemoval(node);
                return node;
            }
        }
        return null;
```



#### 遍历map



==forEach是Map接口新增的default方法==

```
map.forEach((key, value) -> {
        System.out.println(key + ":" + value);
    });
```



#### 基本类型不能做为键值



* 泛型约束为Object类型
  * map.put(1, “Java”)，实际上是将1进行了自动装箱操作,变为了 Integer类型

* 引用数据类型重写了HashCode()和 equals()两个方法，能==保证key的唯一性==







### ConcurrentHashMap



CAS+同步锁+Node+红黑树

对桶进行分段，每个分段用锁保护,**锁只加在数组头节点**，让锁的粒度小，并发性能高

==不接受空key/value==







#### CAS无锁定算法



在ConcurrentHashMap中，很多的操作都会依靠CAS算法完成

```java
  // Unsafe mechanics  CAS保障了哪些成员变量操作是原子性的
    private static final sun.misc.Unsafe U;
    private static final long LOCKSTATE;
      static {
                U = sun.misc.Unsafe.getUnsafe();
                Class<?> k = TreeBin.class; //操作TreeBin,后面会介绍这个类
             LOCKSTATE = U.objectFieldOffset(k.getDeclaredField("lockState"));
--------------------------------------------------------------------------------------
    private static final sun.misc.Unsafe U;
    private static final long SIZECTL;
    private static final long TRANSFERINDEX;
    private static final long BASECOUNT;
    private static final long CELLSBUSY;
    private static final long CELLVALUE;
    private static final long ABASE;
    private static final int ASHIFT;

    static {
        //以下变量会在下面介绍到
            U = sun.misc.Unsafe.getUnsafe();
            Class<?> k = ConcurrentHashMap.class;
            SIZECTL = U.objectFieldOffset(k.getDeclaredField("sizeCtl"));
            TRANSFERINDEX = U.objectFieldOffset(k.getDeclaredField("transferIndex"));
            BASECOUNT = U.objectFieldOffset(k.getDeclaredField("baseCount"));
            CELLSBUSY = U.objectFieldOffset(k.getDeclaredField("cellsBusy"));
            Class<?> ck = CounterCell.class;
            CELLVALUE = U.objectFieldOffset(ck.getDeclaredField("value"));
            Class<?> ak = Node[].class;
            ABASE = U.arrayBaseOffset(ak);
            int scale = U.arrayIndexScale(ak);
            if ((scale & (scale - 1)) != 0)
                throw new Error("data type scale not a power of two");
            ASHIFT = 31 - Integer.numberOfLeadingZeros(scale);}

//3个原子性操作方法：
    static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
        return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
    }

    static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,Node<K,V> c, Node<K,V> v) {
        return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v); }

    static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
        U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);}
```



1.7- 的ConcurrentHashMap中是锁定了Segment

jdk1.8+ 锁定的是一个Node头节点，减小了锁的粒度，性能和冲突都会减少

```java
//这段代码其实是在扩容阶段对头节点的锁定，其实还有很多地方不一一列举。
               synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        Node<K,V> ln, hn;
                        if (fh >= 0) {
                            int runBit = fh & n;
                            Node<K,V> lastRun = f;
                            for (Node<K,V> p = f.next; p != null; p = p.next) {
                                int b = p.hash & n;
                                if (b != runBit) {
                                    runBit = b;
                                    lastRun = p;
                                }
                            }
                            if (runBit == 0) {
                                ln = lastRun;
                                hn = null;
                            }
                            else {
                                hn = lastRun;
                                ln = null;
                            }
                            for (Node<K,V> p = f; p != lastRun; p = p.next) {
                                int ph = p.hash; K pk = p.key; V pv = p.val;
                                if ((ph & n) == 0)
                                    ln = new Node<K,V>(ph, pk, pv, ln);
                                else
                                    hn = new Node<K,V>(ph, pk, pv, hn);
                            }
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                        else if (f instanceof TreeBin) {
                        .....  }    }
```





#### 成员变量



```java
		private static int RESIZE_STAMP_BITS = 16;

    private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;

    private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;

//判断当前状态	MOVED代表该节点是个forwarding Node，表示有线程处理过了
    static final int MOVED     = -1;
    static final int TREEBIN   = -2;

//sizeCtl标志控制符
//-N	正在进行初始化或扩容操作
//N		hash表还没有被初始化，这个数值表示初始化或下一次进行扩容的大小,始终是当前容量的0.75倍
		private transient volatile int sizeCtl;
```



#### 内部类



Node

```java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
//用volatile修饰value/next，使得value和next具有可见性和有序性，保证线程安全
  同时大家仔细看过代码就会发现同时它还错了一个find的方法，该方法主要是用户寻找某一个节点。
        volatile V val;
        volatile Node<K,V> next;
  
//setValue（）方法直接抛出异常，禁止用该方法设置value
public final V setValue(V value) {   throw new UnsupportedOperationException();  }
```



TreeNode

```java
//TreeNode继承自Node,附带next指针
static final class TreeNode<K,V> extends Node<K,V> {
        TreeNode<K,V> parent;  // red-black tree links
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;
   
//TreeBin内部封装了TreeNode,树的根节点为TreeBin,子结点为TreeNode
//TreeNode的next指针可以在TreeBin中寻找下一个TreeNode，这也是与HashMap的区别
static final class TreeBin<K,V> extends Node<K,V> {
        TreeNode<K,V> root;
        volatile TreeNode<K,V> first;
        volatile Thread waiter;
        volatile int lockState;
        // values for lockState
        static final int WRITER = 1; // set while holding write lock
        static final int WAITER = 2; // set when waiting for write lock
        static final int READER = 4; // increment value for setting read lock
```



ForwordingNode

```java
//主要是在扩容阶段，链接两个table的节点类，nextTable用于指向下一个table，但并不是说有2个table，而是在扩容的时,线程读取到为空或者处理完的节点设置为forwordingNode，别的线程发现这个forwordingNode会继续向后执行遍历，解决了多线程安全的问题
//处理节点时会对节点上锁,保证线程安全
static final class ForwardingNode<K,V> extends Node<K,V> {
        final Node<K,V>[] nextTable;
        ForwardingNode(Node<K,V>[] tab) {
            super(MOVED, null, null, null);
            this.nextTable = tab;
        }

        Node<K,V> find(int h, Object k) {
            // loop to avoid arbitrarily deep recursion on forwarding nodes
            outer: for (Node<K,V>[] tab = nextTable;;) {
                Node<K,V> e; int n;
                if (k == null || tab == null || (n = tab.length) == 0 ||
                    (e = tabAt(tab, (n - 1) & h)) == null)
                    return null;
                for (;;) {
                    int eh; K ek;
                    if ((eh = e.hash) == h &&
                        ((ek = e.key) == k || (ek != null && k.equals(ek))))
                        return e;
                    if (eh < 0) {
                        if (e instanceof ForwardingNode) {
                            tab = ((ForwardingNode<K,V>)e).nextTable;
                            continue outer;
                        }
                        else
                            return e.find(h, k);        }
                    if ((e = e.next) == null)
                        return null;    } }   }}
```







#### transfer



在数组扩容时,**有且只能由一个线程构建一个nextTable**，然后把原table复制到nextTable中，复制可以多线程共同操作。在复制过程中有一定的规律和算法操控



数组桶中的3种存储情况：空，链表头，TreeBin头

1. 数组中某个值为空，放置**forwordingNode**
2. 不为空，是链表头结点，就拆分为两个链表，存储到nextTable对应的两个位置
3. 不为空，是TreeBin头结点，此处存储着红黑树，先判断需不需要把树转链表，做完一系列的处理，然后把对应的结果存储在nextTable的对应两个位置



==拆分为两个链表的原因==

* 由hash()和扩容策略决定

* 在原先数组中，下标的计算是(lenth-1) & hash())，哈希值相同的都会在同一个链表中，而且lenth都是2的倍数

* 扩容会扩大原先数组的两倍，如容量8（0111）二进制，扩大一倍(1111)，按位与得到前三位一致，只有第四位不一样

* 9&7=1，1&7=1,扩容后，9&15=9，1&15=1，只有第四位不一样，原先在1位置的元素重新hash之后，只能得到1或则9(1+length)的位置，是对称的



```java
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
        int n = tab.length, stride; //stride 主要和CPU相关
        //主要是判断CPU处理的量，如果小于16则直接赋值16
        if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
            stride = MIN_TRANSFER_STRIDE; // subdivide range
        if (nextTab == null) {            // initiating只能有一个线程进行构造nextTable，如果别的线程进入发现不为空就不用构造nextTable了
            try {
                @SuppressWarnings("unchecked")
                Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1]; //把新的数组变为原来的两倍，这里的n<<1就是向左移动一位，也就是乘2
                nextTab = nt;
            } catch (Throwable ex) {      // try to cope with OOME
                sizeCtl = Integer.MAX_VALUE;
                return;
            }
            nextTable = nextTab;
            transferIndex = n; //原先扩容大小
        }
        int nextn = nextTab.length;
        //构造一个ForwardingNode用于多线程之间的共同扩容情况
        ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
        boolean advance = true; //遍历的确认标志
        boolean finishing = false; // to ensure sweep before committing nextTab
        //遍历每个节点
        for (int i = 0, bound = 0;;) {
            Node<K,V> f; int fh; //定义一个节点和一个节点状态判断标志fh
            while (advance) {
                int nextIndex, nextBound;
                if (--i >= bound || finishing)
                    advance = false;
                else if ((nextIndex = transferIndex) <= 0) {
                    i = -1;
                    advance = false;
                }
                //下面就是一个CAS计算
                else if (U.compareAndSwapInt
                         (this, TRANSFERINDEX, nextIndex,
                          nextBound = (nextIndex > stride ?
                                       nextIndex - stride : 0))) {
                    bound = nextBound;
                    i = nextIndex - 1;
                    advance = false;
                }
            }
            if (i < 0 || i >= n || i + n >= nextn) {
                int sc;
                //如果原table已经复制结束
                if (finishing) {
                    nextTable = null; //可以看出在扩容的时候nextTable只是类似于一个temp用完会丢掉
                    table = nextTab;
                    sizeCtl = (n << 1) - (n >>> 1); //修改扩容后的阀值，应该是现在容量的0.75倍
                    return;//结束循环
                }
                //采用CAS算法更新SizeCtl。
                if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                    if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                        return;
                    finishing = advance = true;
                    i = n; // recheck before commit
                }
            }
            //CAS算法获取某一个数组的节点，为空就设为forwordingNode
            else if ((f = tabAt(tab, i)) == null)
                advance = casTabAt(tab, i, null, fwd);
           //如果这个节点的hash值是MOVED，就表示这个节点是forwordingNode节点，就表示这个节点已经被处理过了，直接跳过
            else if ((fh = f.hash) == MOVED)
                advance = true; // already processed
            else {
            //对头节点进行加锁，禁止别的线程进入
                synchronized (f) {
                //CAS校验这个节点是否在table对应的i处
                    if (tabAt(tab, i) == f) {
                        Node<K,V> ln, hn;
                        //如果这个节点的确是链表节点
                        //把链表拆分成两个小列表并存储到nextTable对应的两个位置
                        if (fh >= 0) {
                            int runBit = fh & n;
                            Node<K,V> lastRun = f;
                            for (Node<K,V> p = f.next; p != null; p = p.next) {
                                int b = p.hash & n;
                                if (b != runBit) {
                                    runBit = b;
                                    lastRun = p;
                                }
                            }
                            if (runBit == 0) {
                                ln = lastRun;
                                hn = null;
                            }
                            else {
                                hn = lastRun;
                                ln = null;
                            }
                            for (Node<K,V> p = f; p != lastRun; p = p.next) {
                                int ph = p.hash; K pk = p.key; V pv = p.val;
                                if ((ph & n) == 0)
                                    ln = new Node<K,V>(ph, pk, pv, ln);
                                else
                                    hn = new Node<K,V>(ph, pk, pv, hn);
                            }
                            //CAS存储在nextTable的i位置上
                            setTabAt(nextTab, i, ln);
                            //CAS存储在nextTable的i+n位置上
                            setTabAt(nextTab, i + n, hn);
                            //CAS在原table的i处设置forwordingNode节点，表示这个这个节点已经处理完毕
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                        //如果这个节点是红黑树
                        else if (f instanceof TreeBin) {
                            TreeBin<K,V> t = (TreeBin<K,V>)f;
                            TreeNode<K,V> lo = null, loTail = null;
                            TreeNode<K,V> hi = null, hiTail = null;
                            int lc = 0, hc = 0;
                            for (Node<K,V> e = t.first; e != null; e = e.next) {
                                int h = e.hash;
                                TreeNode<K,V> p = new TreeNode<K,V>
                                    (h, e.key, e.val, null, null);
                                if ((h & n) == 0) {
                                    if ((p.prev = loTail) == null)
                                        lo = p;
                                    else
                                        loTail.next = p;
                                    loTail = p;
                                    ++lc;  }
                                else {
                                    if ((p.prev = hiTail) == null)
                                        hi = p;
                                    else
                                        hiTail.next = p;
                                    hiTail = p;
                                    ++hc;   }          }
                            //如果拆分后的树的节点数量已经少于6个就需要重新转化为链表
                            ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                (hc != 0) ? new TreeBin<K,V>(lo) : t;
                            hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                (lc != 0) ? new TreeBin<K,V>(hi) : t;
                                //CAS存储在nextTable的i位置上
                            setTabAt(nextTab, i, ln);
                              //CAS存储在nextTable的i+n位置上
                            setTabAt(nextTab, i + n, hn);
                            //CAS在原table的i处设置forwordingNode节点，表示这个这个节点已经处理完毕
                            setTabAt(tab, i, fwd);
                            advance = true; 
```





#### put



* 判断kv是否为空,为空报错

* 根据key的hash值定位到桶位置,判断if(table==null)，先初始化table

* 再判断table中指定的桶
  * ==null,直接插入(无需加锁)
  * ==桶的hash=-1,即MOVED状态（节点为forwordingNode），说明有线程正在进行扩容，当前线程协助扩容==

* 开始put()
  * 桶为链表
    * 遍历检查是否有相同的key?更新:插入
    * 链表>8?树:void
  * 桶为红黑树
    * 插入
    * 自平衡

* 插入后,map已存储数量+1,==在addCount方法中判断是否需要扩容==



##### addCount



```java
    private final void addCount(long x, int check) {
        CounterCell[] as; long b, s;
        if ((as = counterCells) != null ||
            !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
            CounterCell a; long v; int m;
            boolean uncontended = true;
            if (as == null || (m = as.length - 1) < 0 ||
                (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
                !(uncontended =U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
                fullAddCount(x, uncontended);
                return;
            }
            if (check <= 1)
                return;
            s = sumCount();
        }
        //是否需要进行扩容操作
        if (check >= 0) {
            Node<K,V>[] tab, nt; int n, sc;
            while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
                   (n = tab.length) < MAXIMUM_CAPACITY) {
                int rs = resizeStamp(n);
                //如果小于0就说明已经再扩容或者已经在初始化
                if (sc < 0) {
                    if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                        sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||transferIndex <= 0)
                        break;
                        //如果是正在扩容就协助扩容
                    if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                        transfer(tab, nt);
                }
                //如果正在初始化就首次发起扩容
                else if (U.compareAndSwapInt(this, SIZECTL, sc,(rs << RESIZE_STAMP_SHIFT) + 2))
                    transfer(tab, null);
                s = sumCount();
```















### HashTable



双数组

线程安全,效率低

所有涉及多线程都加上了synchronized关键字来**锁住整个table**













### LinkedHashMap



==有序==,用HashMap操作数据结构，用LinkedList维护顺序

```java
LinkedHashMap<K,V> extends HashMap<K,V>  implements Map<K,V>
```



只定义了3个属性

```java
//最久访问的节点或最先插入的节点
transient LinkedHashMap.Entry<K,V> head;
//尾部为最近访问的或最近插入的节点
transient LinkedHashMap.Entry<K,V> tail;
//false插入顺序存储		true查询顺序存储		默认false
final boolean accessOrder;
```



构造器	，**默认都采用插入顺序存储**,都通过调用父类来创建对象

```java
public LinkedHashMap() {
        super();
        accessOrder = false; }

//map参数
public LinkedHashMap(Map<? extends K, ? extends V> m) {
    super();
    accessOrder = false;
    putMapEntries(m, false);}
```



Entry
```java
static class Entry<K,V> extends HashMap.Node<K,V> {
  	//维护Entry的先后顺序
    Entry<K,V> before, after;
  	//kv,next,hash都继承自HashMap
    Entry(int hash, K key, V value, Node<K,V> next) { super(hash, key, value, next);    }}
```

![](image.assets/image-20201113235444857.png)

LinkedHashMap 内部类 Entry 继承自 HashMap 内部类 Node，并新增了两个属性before 和 afte用于维护双向链表

==HashMap 的内部类 TreeNode 不继承自己的内部类 Node，却继承自 LinkedHashMap 内部类 Entry,使得TreeNode 具备了和其他 Entry 一起组成链表的能力(多态)==



==TreeNode的大小约是Node对象的2倍==，仅在桶中包含足够多的节点时才被使用。当桶中的节点数量变少时，TreeNode会被转成 Node=

当用户实现的 hashCode 方法具有良好分布性时,不会转为红黑树，TreeNode将很少被使用





#### 链表初始化



==插入第一个节点时创建链表==,此时head和tail同时指向第一个节点

后续的节点将接在tail指向的节点后面,同时更新tail



插入元素

LinkedHashMap没有覆写父类的 put 方法，而是直接使用了父类的实现

再put之后,调用linkNodeLast()更新链表的tail

```java
    private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
        LinkedHashMap.Entry<K,V> last = tail;
        tail = p;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;   }   }
```



删除元素

没有重写hashMap的删除

```java
HashMap:
	//remove()调用了removeNode()
    public V remove(Object key) {
        Node<K,V> e;
        return (e = removeNode(hash(key), key, null, false, true)) == null ?null : e.value;
    }
//removeNode()执行完HashMap所需的删除逻辑时,回调了afterNodeRemoval(),HashMap中的afterNodeRemoval()为抽象方法
final Node<K,V> removeNode() {
        ...
afterNodeRemoval(node);}

// LinkedHashMap 中覆写
void afterNodeRemoval(Node<K,V> e) { // unlink
    LinkedHashMap.Entry<K,V> p =
        (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
    // 将 p 节点的前驱后后继引用置空
    p.before = p.after = null;
    // b 为 null，表明 p 是头节点
    if (b == null)
        head = a;
    else
        b.after = a;
    // a 为 null，表明 p 是尾节点
    if (a == null)
        tail = b;
    else
        a.before = b;
//删除逻辑
1.根据 hash 定位到桶位置
2.遍历链表
3.从链表中移除要删除的节点
```



#### 维护访问顺序

```
get/getOrDefault/replace	3种改变顺序的方法
```




```java
// LinkedHashMap 中覆写
public V get(Object key) {
    Node<K,V> e;
    if ((e = getNode(hash(key), key)) == null)
        return null;
//accessOrder==true，按访问顺序排序
    if (accessOrder)
        afterNodeAccess(e);
    return e.value;
}

    void afterNodeAccess(Node<K,V> e) { // move node to last
        LinkedHashMap.Entry<K,V> last;
        if (accessOrder && (last = tail) != e) {
            LinkedHashMap.Entry<K,V> p =
                (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
            p.after = null;
            if (b == null)
                head = a;
            else
                b.after = a;
            if (a != null)
                a.before = b;
            else
                last = b;
            if (last == null)
                head = p;
            else {
                p.before = last;
                last.after = p;
            }
            tail = p;
            ++modCount;    }}
```







### TreeMap



红黑树实现

线程不安全



#### TreeMap按Value排序

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







### TreeMap 和 TreeSet 在排序时如何比较元素？Collections.sort（）方法如何比较元素？



TreeSet	实现Comparable接口，该接口提供了比较元素的**compareTo()方法**，当插入元素时会**回调**该方法比较元素的大小

TreeMap	实现Comparable接口,根据键对元素进行排序

Collections工具类的sort方法有两种重载的形式，

第一种要求实现Comparable 接口以实现元素的比较

第二种要求传入第二个参数，参数是 Comparator 接口的子类型（重写compare方法），其实就是是通过接口注入算法，也是对回调模式的应用







## Comparable(函数式接口)



一个类实现了Comparable接口，就意味着“**该类支持排序**”,可以通过 Collections.sort（或 Arrays.sort）进行排序,也可以用作“有序映射(如TreeMap)”中的键或“有序集合(TreeSet)”中的元素，而不需要指定比较器



```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```





## Comparator(内部比较器)



==策略模式==

比较器接口





## Iterator



==Iterator只能单向移动==



```java
forEachRemaining(Consumer<? super E> action) 为剩余元素执行指定的操作,直到所有的元素都完成或抛出异常
hasNext()  如果迭代器中还有元素，则返回true
next()		返回迭代器中的下一个元素
remove()	删除当前元素//唯一能安全地在迭代过程中修改集合的方式,并且每次next()只能一次remove()
```





### ListIterator



继承于Iterator接口,功能更强大

只能用于List类型的访问

==双向移动,保留迭代器前一个和后一个元素的索引==



```java
listIterator(n)	//指定迭代器初始位置
set()	//替换当前元素.
previous()	//前一个元素
```



### Iterator和ListIterator区别



* 只有ListIterator有add()
* ListIterator有hasPrevious()和previous()方法，可以**逆序遍历**
* ListIterator用nextIndex()和previousIndex()指定索引位置
* ListIterator在遍历同时set()修改



### foreach



foreach是通过iterator实现的遍历

对于数组,for和foreach差不多

对于链表，foreach效率高





### fail-fast机制



在迭代时对该集合进行修改，迭代器抛出 ConcurrentModificationException(检测到对象的并发修改)

该异常不会始终指出对象已经由不同线程并发修改，如果**单线程违反了规则，同样也有可能会抛出该异常**

迭代器的快速失败行为无法得到保证，它不能保证一定会出现该错误，但会尽最大努力抛出ConcurrentModificationException异常



为提高并发修改集合的正确性,依赖于此异常是错误的

正确做法是：

* ConcurrentModificationException 仅用于检测 bug

* synchronized  (阻塞问题)

* 使用CopyOnWriteArrayList来替换ArrayList，其所有修改值的操作都对底层数组复制来实现
  * 无需同步,就能实现并发
  * 遍历操作的数量远超可变操作数量



ArrayList

```java
 public E next() {    
            checkForComodification();    
            /** 省略 */    
        }    

        public void remove() {    
            if (this.lastRet < 0)    
                throw new IllegalStateException();    
            checkForComodification();    
            /** 省略 */    
        }

//迭代器调用next()、remove()都会调用checkForComodification()，该方法主要就是检测modCount == expectedModCount,若不等则抛出ConcurrentModificationException 异常，从而产生fail-fast机制。

//expectedModCount 在Itr中定义：int expectedModCount = ArrayList.this.modCount;它的值是不可能修改的，会变的只有modCount
//modCount在 AbstractList 中定义的，为全局变量
protected transient int modCount = 0; 
```



无论add、remove、clear方法,都会导致modCount的改变

```java
public boolean add(E paramE) {    
    ensureCapacityInternal(this.size + 1);    
    /** 省略此处代码 */    
}    

private void ensureCapacityInternal(int paramInt) {    
    if (this.elementData == EMPTY_ELEMENTDATA)    
        paramInt = Math.max(10, paramInt);    
    ensureExplicitCapacity(paramInt);    
}    

private void ensureExplicitCapacity(int paramInt) {    
    this.modCount += 1;    //修改modCount    
    /** 省略此处代码 */    
}    

ublic boolean remove(Object paramObject) {    
    int i;    
    if (paramObject == null)    
        for (i = 0; i < this.size; ++i) {    
            if (this.elementData[i] != null)    
                continue;    
            fastRemove(i);    
            return true;    
        }    
    else    
        for (i = 0; i < this.size; ++i) {    
            if (!(paramObject.equals(this.elementData[i])))    
                continue;    
            fastRemove(i);    
            return true;    
        }    
    return false;    
}    

private void fastRemove(int paramInt) {    
    this.modCount += 1;   //修改modCount    
    /** 省略此处代码 */    
}    

public void clear() {    
    this.modCount += 1;    //修改modCount    
    /** 省略此处代码 */    
}
```















## Collections类



​	专门用来操作集合类 ，提供一系列静态方法实现对各种集合的搜索、排序、线程安全化等操作





### toArray



toArray的空参方法返回的是Object[]

可以调用有参构造,传入有类型的初始数组作为参数,因为参数只起到声明类型的作用,参数数组的长度可以为0

```java
public <T> T[] toArray(T[] a) {
        if (a.length < size)
            // 调用Arrays.copyOf(),传入了参数的class属性,利用反射
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)    a[size] = null;
        return a;
    }
```







### unmodifiableList()



==装饰器模式==

传入一个List实例la，返回这个list的只读视图，类型依然是List

之后对视图进行add、remove等改变其内容的操作,直接抛出异常UnsupportedOperationException

```java
static class UnmodifiableList<E> extends UnmodifiableCollection<E> implements List<E> {

        final List<? extends E> list;

        UnmodifiableList(List<? extends E> list) {
            super(list);
            this.list = list;
        }

        public boolean equals(Object o) {return o == this || list.equals(o);}
        public int hashCode()           {return list.hashCode();}

        public E get(int index) {return list.get(index);}
        public E set(int index, E element) {
            throw new UnsupportedOperationException();
        }
        public void add(int index, E element) {    throw new UnsupportedOperationException();}
        public E remove(int index) {   throw new UnsupportedOperationException(); }
        public int indexOf(Object o)            {return list.indexOf(o);}
        public int lastIndexOf(Object o)        {return list.lastIndexOf(o);}
        public boolean addAll(int index, Collection<? extends E> c) {    throw new UnsupportedOperationException();}

        @Override
        public void replaceAll(UnaryOperator<E> operator) {   throw new UnsupportedOperationException();}
        @Override
        public void sort(Comparator<? super E> c) {  throw new UnsupportedOperationException();}
```





















## 队列



offer，add区别：

堆满插入时add抛出 unchecked 异常

offer()返回false

 

poll，remove区别：

remove() 和 poll() 方法都是从队列中删除第一个元素。remove() 的行为与 Collection 接口的版本相似

poll() 被空集合调用时不抛出异常，只是返回 null

 

peek，element区别：

element() 和 peek() 用于在队列的头部查询元素。

在队列为空时， element() 抛出一个异常，而 peek() 返回 null





### Deque



双端队列接口，继承自Queue接口

实现类	LinkedList、ArrayDeque、LinkedBlockingDeque



Deque接口扩展了 Queue 接口。在将双端队列用作队列时，将得到 FIFO（先进先出）行为。将元素添加到双端队列的末尾，从双端队列的开头移除元素。从 Queue 接口继承的方法完全等效于 Deque 方法

| **Queue方法** | **等效Deque方法** |
| ------------- | ----------------- |
| add(e)        | addLast(e)        |
| offer(e)      | offerLast(e)      |
| remove()      | removeFirst()     |
| poll()        | pollFirst()       |
| element()     | getFirst()        |
| peek()        | peekFirst()       |



也可用作 LIFO（后进先出）堆栈。在将双端队列用作堆栈时，元素被推入双端队列的开头并从双端队列开头弹出。堆栈方法完全等效于 Deque 方法

| **堆栈方法** | **等效Deque方法** |
| ------------ | ----------------- |
| push(e)      | addFirst(e)       |
| pop()        | removeFirst()     |
| peek()       | peekFirst()       |











### PriorityQueue 优先队列 1.5+



* **基于优先堆**的**无界**队列，**容量不受限制,会自动扩容**，但可以指定初始容量

* **不允许空值，不支持不可比较的对象**，如自定义类

* **队头最小**
* PriorityQueue是**非线程安全**的，==PriorityBlockingQueue线程安全==（实现BlockingQueue接口）











# NIO



![](image.assets/nio与io区别.png)

* io
  * 阻塞的IO模型
  * 单向
  * 以最基础的字节流处理数据

* nio
  * ==多路复用==的IO模型
  * 利用缓冲实现了数据在channel中的**双向**传输
  * 以块的方式处理数据,效率高
  * 通道和缓冲区
    * 缓冲区可以分片，只读/直接/间接缓冲区





## Buffer类4个属性与方法





* buffer	标记当前的position

* capacity	最大容量

* limit	可以操作数据的个数

* position	正在被操作数据的位置
  * position<=limit<=capacity



put() get()存取数据

flip()读数据模式	开启之后,当调用get()时, 会将position调为0, limit调为当前最大存储位置,然后再执行get()方法, 不然position并不在起始位置

rewind()重置读数据模式	再次将position调为0, limit调为当前最大存储位置

clear()	**并不会删除数据**,只是将三个属性初始化,里面的数据处于"被遗忘"状态,position和limit都被初始化,难以读取数据

**mark()	记录当前的position位置**

**reset()	配合mark()的使用,回到mark的位置**



## 通道



* IO
  * cpu需要建立若干io接口来进行io操作,这将导致cpu被占用
  * 后来引入了**DMA**直接存储器访问,cpu将io操作交给DMA进行,DMA先向cpu申请资源,然后形成**DMA总线**,不过总线的过多也会导致总线冲突,影响性能

* NIO
  * **channel类似于DMA总线**,是一个完全独立的处理器,专门用于处理io,不需要向cpu申请资源

![](.\image.assets\image-20200809142947422.png)



### 主要实现类

* FileChannel	      本地传输
* SocketChannel        TCP
* ServerSocketChannel  TCP
* DatagramChannel      UDP



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

非直接	allocate()分配缓冲区,缓冲区在**jvm**

直接	allocateDirect(),在**物理内存**

jvm对于直接缓冲区,会尽量避免使用中间缓冲区进行数据的读写,而是直接在缓冲区上进行io操作

分配**直接**缓冲区需要**更大的成本**,也**不会被gc回收**,会影响应用程序的内存

![image-20200809134909724](.\image.assets\image-20200809134909724.png)

对于非直接缓冲区,物理磁盘的数据先读取至内核地址空间,再被copy到jvm内存,最后到应用程序

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

如果在一个try/catch中关闭多个流,将会导致关闭时其中一个流,抛出异常,程序中断,之后的流将不再被关闭!!!

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



**进程**：在操作系统中能独立运行，并作为资源分配的基本单位。它表示运行中的程序。运行程序是进程从创建->运行->消亡的过程

**线程**：是比进程更小的执行单位，能够完成进程中的一个功能，也被称为轻量级进程。进程在执行的过程中可以产生多个线程

同类的多个**线程共享进程的堆和方法区**，但**每个线程有自己的程序计数器、虚拟机栈和本地方法栈**，所以系统在各个线程之间作切换工作时，负担比进程小

![](image.assets/JVM模型(线程).png)





| 并行 | 同时多个进程运行                  |
| ---- | --------------------------------- |
| 并发 | 上下文快速切换,造成同时运行的假象 |
| 串行 | 按先后顺序进行                    |



* 守护线程 daemon
  * 即使没有主动创建线程,后台也会有多个线程,如主线程(用户线程),gc线程(守护线程)

**JVM必须保证用户线程执行完毕,但无需等待守护线程执行完毕**







线程的运行由**调度器安排调度**,调度器由操作系统控制,先后顺序无法干预

对同一份资源操作时,存在资源抢夺问题,需要并发控制

线程会带来额外开销,如cpu调度时间,并发控制开销

**每个线程在自己的工作内存交互,内存控制不当会造成数据不一致**



==线程安全主要体现在以下3个方面==

* 原子性：互斥访问，同时只能有一个线程进行操作
* 可见性：线程对主内存的修改可以及时被其他线程观察到
* 有序性：上下两个互不关联的语句不会被指令重排序
  * 指令重排序是指处理器为了性能优化，在无关联的代码的执行是可能会和代码顺序不一致
  * 如果编译器推迟执行一个操作，其他线程可能在这个操作执行完之前都不会看到该操作的结果，这反映了缓存的影响





上下文切换

**CPU给每个线程分配CPU时间片**,不停地切换线程，在切换前会保存上一个任务的状态，以便下次切换回这个任务时，可以再加载这个任务的状态。所以**任务从保存到加载的过程就是一次上下文切换**。**上下文切换会影响多线程的执行速度**







每个线程都有一个独立的工作内存，用于存储线程私有的数据

Java内存模型规定主内存是共享内存区域，所有线程都可以访问

线程**对变量的操作在工作内存中进行**（线程安全问题的根本原因）

* 首先要将变量从主内存拷贝到工作内存

* 然后对变量进行操作，再将变量写回主内存

* **因此不同的线程间无法访问对方的工作内存**，==线程间的通信(传值)必须通过主内存来完成==,多线程对共享变量进行修改时，都是对自己工作内存的副本进行操作，相互不可见。主内存中共享变量的结果是不可预知的





## start() 会执行 run() 

new 一个 Thread，线程进入初始状态；

调用 start()方法，会启动一个线程并使线程进入了就绪状态，当分配到时间片后就可以开始运行了。 start() 会执行线程的相应准备工作，然后自动执行 run() 方法的内容，这是真正的多线程工作

而直接执行 run() 方法，会把 run 方法当成一个 main 线程下的普通方法去执行，并不会在某个线程中执行它，并不是多线程工作

**总结： 调用 start 方法可启动线程并使线程进入就绪状态，而 run 方法只是 thread 的一个普通方法调用，还是在主线程里执行**









## CAS无锁定算法



Compare And Swap，比较与交换

* 内存地址V，旧值A和新值B
* 执行过程中发现V上的值与A相等(主内存未被修改)，就将V更新为B。不匹配不执行任何操作
* 无锁定,线程不必等待锁定，效率高



* 缺点
  * CPU开销大  反复尝试更新，却一直不成功，给CPU压力
  * 不能保证代码块的原子性   **只保证变量的原子性操作**



### ABA问题



**当一个值从A->B，又B->A，普通CAS机制会误判通过检测**



| 线程1 | 获取当前值A | 期望更新为B |
| ----- | ----------- | ----------- |
| 线程2 | 获取当前值A | 期望更新为B |
| 线程3 |             | 期望更新为A |



* 1执行成功，A->B	同时2阻塞

* 3获取当前值B,2仍然阻塞

* 3执行，B->A

* 2恢复，由于阻塞之前获得“当前值A”，并且compare()内存地址V中的实际值也是A，所以A->B



**虽然代码逻辑无问题,但业务逻辑不合理**



* 100存款，提款50

* 提款机出问题，提款操作被同时提交两次，开启了两个线程，都获取当前值100，要更新成50
  * 正常的业务为1次成功,1次失败

* 1执行成功，100->50	2阻塞

* 发生汇款50	2仍然阻塞

* 汇款成功，50->100

* 2恢复,100->50    被提款了2次



==版本号==

真正要做到严谨的CAS，compare()不仅要比较A和V的实际值，还要比较变量的版本号是否一致

每次修改都更新版本号



AtomicStampedReference类就实现了用版本号作比较机制



























## JUC 1.5+





### CountDownLatch 闭锁



Java的concurrent包里面的CountDownLatch其实可以把它看作一个计数器，只不过这个计数器的操作是原子操作，同时只能有一个线程去操作这个计数器，也就是同时只能有一个线程去减这个计数器里面的值。



- 当一个或多个线程调用await方法时，这些线程会阻塞
- 其它线程调用countDown方法会将计数器减1(调用countDown方法的线程不会阻塞)
- 计数器的值变为0时，因await方法阻塞的线程会被唤醒，继续执行



向CountDownLatch对象设置一个初始的数字作为计数值，任何调用这个对象上的await()方法都会阻塞，直到这个计数器的计数值被其他的线程减为0为止。



适用场景:

有一个任务想要往下执行，但必须要等到其他的任务执行完毕后才可以继续往下执行。假如我们这个想要继续往下执行的任务调用一个CountDownLatch对象的await()方法，其他的任务执行完自己的任务后调用同一个CountDownLatch对象上的countDown()方法，这个调用await()方法的任务将一直阻塞等待，直到这个CountDownLatch对象的计数值减到0为止



```java
//需求:要求6个线程都执行完了,mian线程最后执行
public class CountDownLatchDemo {
    public static void main(String[] args) throws Exception{

        CountDownLatch countDownLatch=new CountDownLatch(6);
        for (int i = 1; i <=6; i++) {
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+"\t");
                countDownLatch.countDown();
            },i+"").start();
        }
        countDownLatch.await();
        System.out.println(Thread.currentThread().getName()+"\t班长关门走人，main线程是班长");
    }
}
```





### CyclicBarrier



可循环(Cyclic) 使用的屏障(barrier)

让一组线程到达屏障(同步点)时被阻塞,直到所有线程到达屏障时,唤醒所有被拦截的线程



```java
    //集齐7颗龙珠就能召唤神龙
	public class CyclicBarrierDemo {
	    public static void main(String[] args) {
	        // public CyclicBarrier(int parties, Runnable barrierAction) {}
	        CyclicBarrier cyclicBarrier=new CyclicBarrier(7,()->{
	            System.out.println("召唤龙珠");
	        });
	        for (int i = 1; i <=7; i++) {
	            final int temp=i;
	            new Thread(()->{
	                System.out.println(Thread.currentThread().getName()+"\t收集到了第"+temp+"颗龙珠");
	                try {
	                    cyclicBarrier.await();
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                } catch (BrokenBarrierException e) {
	                    e.printStackTrace();
	                }
	            }).start();
	        }
	
	    }
	}
```





### Semaphore 信号量



- acquire（获取） 成功获取,信号量–
- release（释放）信号量++，然后唤醒等待的线程
- 信号量主要用于两个目的，一个是用于多个共享资源的互斥使用，另一个用于并发线程数的控制



```java
public class SemaphoreDemo {
	    public static void main(String[] args) {
	        Semaphore semaphore=new Semaphore(3);
	        for (int i = 1; i <=6; i++) {
	            new Thread(()->{
	                try {
	                    System.out.println(Thread.currentThread().getName()+"\t抢占了车位");
	                    semaphore.acquire();
	                    System.out.println(Thread.currentThread().getName()+"\t离开了车位");
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                }finally {
	                    semaphore.release();
	                }
	            },String.valueOf(i)).start();
	        }
	    }
	}
```







 



### 原子操作类



juc.atomic包

AtomicBoolean，AtomicUInteger，AtomicLong。分别用于Boolean，Integer，Long类型的原子性操作



Atomic操作类的底层用到了“CAS机制”



AtomicInteger

```java
public class AtomicInteger extends Number implements java.io.Serializable {
    // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;
 
    static {
        try {
            valueOffset = unsafe.objectFieldOffset(AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }
 
    private volatile int value;
 
    public final int incrementAndGet() {
      //this对象本身	valueOffset记录value在主内存的地址
        return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
    }
}
```



Unsafe

```java
public final class Unsafe {
    private static final Unsafe theUnsafe;

 public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            var5 = this.getIntVolatile(var1, var2);
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
        return var5;
    }
 
//native方法	var1当前对象	var2 valueOffsetvalue在主内存的地址	var4常量1	var5根据var1/2,调用getIntVolatile()得到的值
//当var1==var5,才更新
 public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
 
 public native int getIntVolatile(Object var1, long var2);
}
```





## ThreadLocal



为每个线程**创造资源的副本**,而不是共享资源。隔离每个线程存取数据的行为，给线程特定空间来保管该线程的独享资源



ThreadLocal内部类ThreadLocalMap，存储每个线程的变量的副本，数据存储以后，只有指定线程可以得到存储数据

```java
class Thread implements Runnable {
  //每个线程一个ThreadLocalMap
 ThreadLocal.ThreadLocalMap threadLocals = null;
}
   

public void set(T value) {
      //当前线程
      Thread t = Thread.currentThread();
      ThreadLocalMap map = getMap(t);
      //当前线程持有map就直接set，没有则创建并set
      if (map != null)
          map.set(this, value);
      else
          createMap(t, value);
  }
  
//每个线程持有一个ThreadLocalMap对象
ThreadLocalMap getMap(Thread t) {return t.threadLocals;}
   
void createMap(Thread t, T firstValue) {
      //每个新线程Thread都会实例化一个ThreadLocalMap赋值给成员变量threadLocals
      t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```



**ThreadLocal的静态内部类ThreadLocalMap,维护数组table，ThreadLocal确定一个数组下标，对应value存储位置**

```java
ThreadLocalMap {
        private static final int INITIAL_CAPACITY = 16;
        private Entry[] table;
        private int size = 0;
        private int threshold; // Default to 0
  
//Entry为ThreadLocalMap静态内部类，对ThreadLocal的弱引用,让ThreadLocal和储值形成key-value的关系
static class Entry extends WeakReference<ThreadLocal<?>> {
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
           super(k);
           value = v;
    }
}

//ThreadLocalMap构造方法
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
        //创建了长度为16的Entry数组
        table = new Entry[INITIAL_CAPACITY];
        //位运算,计算出存放位置
        int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
        table[i] = new Entry(firstKey, firstValue);
        size = 1;
        setThreshold(INITIAL_CAPACITY);
```



每个Thread持有一个ThreadLocalMap的实例threadLocals-> thread:table = 1:1



```java
//在某一线程声明了ABC三种类型的ThreadLocal
//一个Thread只有一个ThreadLocalMap -> ABC对应同一个ThreadLocalMap对象,但存储在table数组的不同位置
ThreadLocal<A> sThreadLocalA = new ThreadLocal<A>();
ThreadLocal<B> sThreadLocalB = new ThreadLocal<B>();
ThreadLocal<C> sThreadLocalC = new ThreadLocal<C>();
```



ABC在table中的位置通过这里确定

```java
  //ThreadLocalMap中set方法。
  private void set(ThreadLocal<?> key, Object value) {
            Entry[] tab = table;
            int len = tab.length;
            //获取索引值
            int i = key.threadLocalHashCode & (len-1);

            //遍历tab如果已经存在则更新值
            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                ThreadLocal<?> k = e.get();

                if (k == key) {
                    e.value = value;
                    return;
                }

                if (k == null) {
                    replaceStaleEntry(key, value, i);
                    return;
                }
            }
            
            //如果上面没有遍历成功则创建新值
            tab[i] = new Entry(key, value);
            int sz = ++size;
            //满足条件数组扩容x2
            if (!cleanSomeSlots(i, sz) && sz >= threshold)
                rehash();
        }
```



在ThreadLocalMap中的set方法与构造方法能看到

- `int i = key.threadLocalHashCode & (len-1)`
- `int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1)`
  将threadLocalHashCode进行一个位运算（取模）得到索引i



threadLocalHashCode是通过原子操作类AtomicInteger的自增确定的

```java
    private final int threadLocalHashCode = nextHashCode();

    private static AtomicInteger nextHashCode = new AtomicInteger();

    private static final int HASH_INCREMENT = 0x61c88647;//0x61c88647是斐波那契散列乘数，避免hash冲突

    private static int nextHashCode() {
        //每次new ThreadLocal会使threadLocalHashCode自增，增量为0x61c88647
        return nextHashCode.getAndAdd(HASH_INCREMENT);
    }
```



对于ThreadLocal对应的索引i是确定的

同一Thread的不同ThreadLocal，这些ThreadLocal实例共享table数组，每个ThreadLocal实例在table中的索引i不同

不同Thread间的table独立



### get方法



```java
//ThreadLocal中get方法
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
    
//ThreadLocalMap中getEntry方法
private Entry getEntry(ThreadLocal<?> key) {
       int i = key.threadLocalHashCode & (table.length - 1);
       Entry e = table[i];
       if (e != null && e.get() == key)
            return e;
       else
            return getEntryAfterMiss(key, i, e);
   }
```

理解了set方法，get方法也就清楚明了，无非是通过计算出索引直接从数组对应位置读取即可。







### ThreadLocal特性



ThreadLocal和Synchronized都是为了解决多线程中相同变量的访问冲突问题

不同的点是

- Synchronized通过线程等待，牺牲时间来解决访问冲突
- ThreadLocal通过每个线程单独一份存储空间，牺牲空间来解决冲突，**具有线程隔离效果，只有在线程内才能获取到对应的值**
  - 正因为线程隔离特性，当数据是以线程为作用域并且不同线程具有不同的数据副本的时候,就可以考虑采用ThreadLocal
  - 如android中Looper、ActivityThread以及AMS











## 线程6种状态

![](image.assets/image-20200904093701936.png)

* 新建 NEW

* 就绪/运行 RUNNABLE   排队等待操作系统分配 CPU时间片

  * Thread.start()  NEW-> RUNNABLE
  * 就绪不代表立即运行,还需要和其他线程竞争CPU

* 阻塞 BLOCKED  等待锁

  * synchronized

* 等待 WAITING   等待另一个线程执行特定的动作

  * wait() => notify()

* 计时等待 TIMED_WAITING，和WAITING类似，只是多了超时时间

  * wait(long timeout) / Thread.join(long timeout) 设置了超时时间时，才会进入此状态

* 终止 TERMINATED，线程死亡	**死亡线程将无法再次start**



## 线程的方法



线程可以获得多个锁



### 创建线程3种方式



* 继承Java.lang.Thread类，重写run()        **Thread本身继承了Runnable**
  * 优势：编写简单
  * 劣势：单继承,无法继承其它父类



* 实现 Java.lang.Runnable 接口，实现 run()
  * 优势：可继承其它类
  * 劣势：较复杂，如需访问当前线程，需Thread.currentThread()



* 实现Callable接口  (有返回值,可以抛出异常)

```java
//1.实现Callable接口,定义返回值类型
public class TestCallable implements Callable<String> {

  //2.重写call()方法,需要抛出异常
    @Override
    public String call() {  return Thread.currentThread().getName(); }

    public static void main(String[] args) throws ExecutionException, InterruptedException  {
        //3.创建执行服务
        ExecutorService service = Executors.newFixedThreadPool(3);
        //4.提交执行
        Future<String> result = service.submit(new TestCallable());
        //5.获取返回值
        String str = result.get();
        //6.关闭服务	需要抛出2个异常
        service.shutdown();}}
```









* setPriority()	更改优先级	优先级范围1-10

​	优先级低的也有可能被先调用,全看cpu心情,这将导致性能倒置 :优先级高的一直在等待

* yield()	**暂停但不阻塞**正在执行的线程对象,**转入就绪状态**,cpu有可能再次调度到礼让线程,导致礼让失败
* sleep()	
  * **Thread类的静态方法**,转入阻塞转态
  * ==计时结束自动苏醒,不涉及线程间通信==,监控状态依然保持,==不释放锁==
  * **sleep()可以在任何地方使用**
  * 异常抛出InterruptedException
  * **超时/interrupt()唤醒**

* wait 
  * Object的方法,==释放对象锁==
  * **wait()必须在同步控制方法和同步代码块中使用**
  * ==notify()/计时结束 苏醒==



* join()	合并线程,相当于插队,其他线程等待该线程终止	容易造成线程阻塞

`interrupt()	中断线程(不推荐)`

* isAlive	是否存活

* start()    此时线程处于就绪状态，并没有运行，得到 cpu 时间片**再执行 run()方法** .run()方法只是类的一个普通方法而已，**如果直接调用 run 方法，程序中依然只有主线程**，还是要顺序执行
* Thread.state / thread.getState()	获取线程状态
* **线程同时启动**    for 循环，调用 wait()方法，让所有线程等待,再调用 notifyAll(), 同时启动所有线程



Object

```java
final void wait() 	等待其它线程通知

void wait(long timeout) 线程等待指定毫秒参数的时间

final void wait(longtimeout,int nanos)线程等待指定毫秒、微妙的时间

final void notify()唤醒一个处于等待状态的线程。注意的是在调用此方法的时候，并不能确切的唤醒某一个等待状态的线程，而是由 JVM 确定唤醒哪个线程，而且不是按优先级。

final void notifyAll()唤醒同一个对象上所有调用 wait()方法的线程，注意并不是给所有唤醒线程一个对象的锁，而是让它们竞争
```





**不推荐调用jdk的stop,destroy方法停止线程**,可以在源码看到这些方法加上了@Deprecated注解,表示方法过时

应该用boolean标志,boolean=false停止线程 **,让线程自己停下来,而不是被动停止**



JDK 1.5 还提供了信号量(semaphore)机制，信号量可以用来限制对某个共享资源进行访问的线程的数量。在对资源进行访问之前，线程必须得到信号量的许可（调用 Semaphore 对象的 acquire()方法）；在完成对资源的访问后，线程必须向信号量归还许可（调用 Semaphore 对象的 release()方法）。









## 锁



**对象锁分为三种：共享资源、this、当前类的字节码文件对象**



### 分类



#### 悲观/乐观



* 悲观
  * 认为别人会修改，每次拿数据的时都上锁，别的线程block
  * 传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁



* 乐观
  * 认为别人不修改，不上锁，但在更新时判断在此期间别人有没有去更新数据(版本号),在更新数据时提高版本号.提交时的版本低于目前版本,将回滚
  * 适用于并发量不大的场景,省去了锁的开销，加大了系统的整个吞吐量
  * 如果并发量大,上层应用不断进行retry，降低性能





#### 同步锁



每个对象都有一个内置锁,一个对象只有一个锁

当程序运行到非静态的 synchronized 同步方法上时，自动获得与正在执行代码类的当前实例（this 实例）有关的锁

获得一个对象的锁也称为获取锁、 锁定对象、在对象上锁定或在对象上同步

一个线程获得该锁，就没有其他线程可以获得锁，直到第一个线程释放/返回锁





#### 公平/非公平



公平锁：当线程发现已经有线程在排对获取锁了，那么它必须排队，**除非线程已占有锁，此次重入**

非公平锁：只有一种情况需排队： 如果当前全局处于读锁状态，且等待队列中第一个等待线程想获取写锁，那么当前线程能够获取到读锁的条件为：当前线程获取了写锁，还未释放；当前线程获取了读锁，这一次只是重入读锁而已；其它情况当前线程入队尾



#### 可重入



线程外层函数获得锁后,再进入该线程的内层方法会自动获取锁 **(锁对象是同一个)**

**可以在一定程度上避免死锁**

ReentrantLock,synchronized



#### 隐式/显示



区分点在于有没有显示声明锁

隐式:synchronized

显示:lock





#### 自旋



**SMP架构中的一种low-level的同步机制**

当线程A想要获取一把自旋锁,而该锁又被其它线程锁持有时，线程A会在一个循环中自旋以检测锁是否可用



- **由于自旋时不释放CPU，因而持有自旋锁的线程应该尽快释放自旋锁，否则等待该自旋锁的线程会一直自旋，浪费CPU时间**
- **持有自旋锁的线程在sleep前应释放自旋锁**







### 死锁



多个线程各自占有一部分共享资源,并互相等待

常发生于**一个同步块同时拥有2个以上对象的锁**



**4个必要条件**

* 互斥	一个资源同时被多个进程使用

* 请求与保持	一个进程请求资源而阻塞,对已有的资源保持不释放

* 不剥夺	进程已获得的资源在未使用完之前,不会被抢夺

* 循环等待	若干个进程之间形成循环等待资源

只要打破一个条件就能避免死锁



- **避免一个线程同时获取多个锁**
- **避免一个线程在锁内同时占用多个资源，尽量保证每个锁只占用一个资源。**
- **使用定时锁**
- **对于数据库锁，加锁和解锁必须在一个数据库连接里，否则会出现解锁失败的情况**

* 无锁函数（cas）或 重入锁（ReentrantLock）

　

### 活锁



体现了一种谦让的美德，每个线程都想把资源让给对方

但可能会产生一直将资源让来让去，导致资源在两个线程间跳动而无法使某一线程真正的到资源并执行



### 饥饿



线程一直获取不到资源，一直无法执行。

优先级太低 或 某一线程一直占着某种资源不放



解决方法：

　　与死锁相比，饥饿现象还是有可能在一段时间之后恢复执行的。**设置优先级尽量避免饥饿**







**尽量不要嵌套同步**

synchronized (对象) { 

​	synchronized (对象) { }

}







## 多线程带来的问题





#### 线程安全



某一线程从开始访问到结束访问，该数据被其他的线程所修改，表现形式为数据的缺失/不一致

　　线程安全问题发生的条件：	存在共享资源，非原子性地操作该共享资源



解决思路：

　　　　**尽量不使用共享变量，将不必要的共享变量变成局部变量来使用**

　　　　**synchronized/Lock**。

　　　　**ThreadLocal为每一个线程建立一个变量的副本，各个线程间独立操作，互不影响**



#### 性能问题



线程的生命周期开销是非常大的，**线程的创建到销毁会占用大量的内存**。

如果不合理的创建了多个线程，cup的处理器数量小于了线程数量，将会有很多的线程被闲置，闲置的线程将会占用大量的内存，为垃圾回收带来很大压力，同时cpu在分配线程时还会消耗其性能



解决思路：

　　**利用线程池**，模拟一个池，预先创建有限合理个数的线程放入池中，当需要执行任务时从池中取出空闲的先去执行任务，执行完成后将线程归还到池中，这样就**减少了线程的频繁创建和销毁，节省内存开销**和减小了垃圾回收的压力。同时因为任务到来时本身线程已经存在，减少了创建线程时间，提高了执行效率，而且合理的创建线程池数量还会使各个线程都处于忙碌状态，提高任务执行效率

线程池还提供了拒绝策略，当任务数量到达某一临界区时，线程池将拒绝任务的进入，保持现有任务的顺利执行，减少池的压力



#### 活跃性问题





　

#### **（四）阻塞**

　　阻塞是用来形容多线程的问题，**几个线程之间共享临界区资源，那么当一个线程占用了临界区资源后，所有需要使用该资源的线程都需要进入该临界区等待，等待会导致线程挂起，一直不能工作，这种情况就是阻塞**，**如果某一线程一直都不释放资源，将会导致其他所有等待在这个临界区的线程都不能工作**。当我们使用synchronized或重入锁时，我们得到的就是阻塞线程，如论是synchronized或者重入锁，都会在试图执行代码前，得到临界区的锁，如果得不到锁，线程将会被挂起等待，知道其他线程执行完成并释放锁且拿到锁为止。

　　解决方法：

　　**可以通过减少锁持有时间，读写锁分离，减小锁的粒度，锁分离，锁粗化等方式来优化锁的性能**。

> 临界区：
>
> 　　临界区是用来表示一种公共的资源（共享数据），它可以被多个线程使用，但是在每次只能有一个线程能够使用它，当临界区资源正在被一个线程使用时，其他的线程就只能等待当前线程执行完之后才能使用该临界区资源。
>
> 　　比如办公室办公室里有一支笔，它一次只能被一个人使用，假如它正在被甲使用时，其他想要使用这支笔的人只能等甲使用完这支笔之后才能允许另一个人去使用。这就是临界区的概念。







## 线程通信



wait + notify 解决线程通信

这两个都是Object的方法,只能在同步方法或同步代码块中**成对使用**,否则抛出IIIegalMonitorStateException



### 管程法

生产者把产品放入**缓冲区**,消费者从缓冲区拿

每次操作时判断缓冲区的容量,满了则生产者不生产,空了消费者不消费

![image-20200909221411140](image.assets/image-20200909221411140.png)



### 信号灯法

判断**标志位**,如果为真,等待,如果为假,唤醒

每次进行操作时判断标志位,决定wait或者是执行





## volatile



==volatile弱同步，不保证线程安全==	只能用于变量

synchronized强同步



一、volatile 的内存语义是：

* 写入时，JMM 把工作内存中的**变量值立即刷新到主内存,并通知其他线程**
  * 其他线程的写入/读取,直接工作内存中的副本，重新去主内存获取
* 有序性,产生==内存屏障==，防止指令重排,必定能够顺序执行
* 确保指令重排序时不会把其后面的指令排到内存屏障之前/之后
* volatile 变量不会被缓存在寄存器或者对其他处理器不可见的地方，因此在读取 volatile 类型的变量时总会返回最新写入的值





二、volatile 底层的实现机制

把 volatile变量和非volatile变量都生成汇编代码，会发现 volatile 变量多出一个 lock 前缀指令

1. 重排序时不能把后面的指令重排序到内存屏障之前的位置

2、使得本 CPU 的 Cache 写入内存

3、写入动作也会引起别的 CPU 无效化其 Cache，相当于让新写入的值对别的线程可见



### volatile VS synchronized



| volatile                           | synchronized               |
| ---------------------------------- | -------------------------- |
| 线程同步的轻量级实现,性能高        |                            |
| **只能用于变量**                   | **可以修饰方法以及代码块** |
| ==主要解决变量在多线程间的可见性== | 多线程间访问资源的同步性   |
| 不保证数据的原子性                 | 可见+原子性                |









### 线程不安全案例

```java
public static volatile int c = 0;
    
    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 100000; i++) {
            new Thread(() -> {
                c++;	//非原子性操作
                System.out.println(c);
            }
            ).start();
        }
        Thread.sleep(5000);
    }

volatile修饰的变量具有原子性- >c 具有原子性，但c++不具有 -> c = c + 1，已经存在了多步操作。所以c具有原子性，但是c++这个操作不具有原子性
```









## synchronized



**悲观锁**,认为程序中的并发情况严重，所以严防死守

让没有得到锁资源的线程进入BLOCKED状态，而后在争夺到锁资源后恢复为RUNNABLE状态，退出或抛出异常时必须释放锁



用于**对象/方法(实例+静态方法)**,==不能同步变量和类==

- **对于普通同步方法，锁是当前实例对象。**
- **对于静态同步方法，锁是当前类的Class对象。**
- **对于同步代码块，锁是synchronized括号里配置的对象**



**synchronized 同步语句块的实现使用的是 monitorenter 和 monitorexit 指令****

monitorenter 指令指向同步代码块的开始位置

monitorexit 指令则指明同步代码块的结束位置

执行 monitorenter 指令时，线程试图获取锁也就是获取 monitor的持有权。当计数器为0则可以成功获取，获取后将锁计数器设为1也就是加1

执行 monitorexit 指令后，将锁计数器设为0，表明锁被释放

如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止

(==monitor对象存在于每个Java对象的对象头中== -> synchronized用到的锁是存在**Java对象头**,也是为什么Java中任意对象可以作为锁的原因) 







尽管JAVA 1.6为synchronized做了优化,如**偏向锁、轻量级锁、自旋锁、适应性自旋锁、锁消除、锁粗化**等技术来减少锁操作的开销

但在最终转变为重量级锁之后，性能仍比较低

面对这种情况们就可以使用“**原子操作类**”



**锁主要存在四种状态，依次是：无锁状态、偏向锁状态、轻量级锁状态、重量级锁状态**，他们会随着竞争的激烈而逐渐升级

==锁不可降级==，锁的升级策略提高获得锁和释放锁的效率



### 同步代码块/方法



synchronized控制对象的访问,每个对象对应一把锁,必须获得该方法的对象的锁才能执行方法,否则会线程阻塞

方法执行完毕,才会释放锁,让下一个线程拿到锁



* 同步代码块	synchronized (对象) { }
  * 同步代码块在方法内部的==对象上==加锁。



* 同步方法：
  * 在==方法上==加synchronized,锁的范围大，性能差
  * ==同步方法默认锁定this,即当前类,所以不需要指明对象==

**在静态方法中，默认锁定类对象**















## Lock 1.5+



**乐观锁**，认为程序中的并发情况不严重，让线程不断去重试更新



**需要显式指定起始位置lock()和终止位置unlock()**,一般会在finally块中写unlock()以防死锁

能完成synchronized的所有功能,有更精确的线程语义和更好的性能

==synchronized会自动释放锁，Lock必须手动在finally块释放==



```java
lock()	以阻塞方式来获取锁，如果获取到了锁，立即返回；如果别的线程持有锁，则当前线程等待，直到获取锁后返回

tryLock()以非阻塞的方式获取锁。只是尝试性的去获取一下锁，如果获取到锁，立即返回true

tryLock(longtimeout,TimeUnit unit)。如果获取到了锁，立即返回true，否则会等待参数给定的时间单元，在等待的过程中，如果获取到了锁，就立即返回true。如果等待超时，返回false。

lockInterruptibly（）	如果获取到了锁，立即返回；如果没有获取到锁，当前线程处于休眠状态，或者被别的线程中断（会受到InterruptedException异常）。与lock()方法的区别在于lock优先考虑获取锁，如果没有获取到锁，会一直处于阻塞状态，忽略interrupt（）方法，待获取锁成功后，才响应中断。lockInterruptibly 优先考虑响应中断，而不是响应锁的普通获取或重入获取

Unlock()释放锁
```



==有多少个lock,就有多少个unlock,成对出现,否则会使其他线程处于等待状态==







### AQS



AbstractQueuedSynchronizer抽象的队列式同步器

除synchronized之外的锁,都基于aqs机制



![](image.assets/AQS结构.png)



如果被请求的共享资源空闲，则将当前线程设置为有效，并锁定共享资源

如果被请求的共享资源被占用，线程阻塞,加入CLH队列,等待合适时机唤醒



CLH是一个虚拟的双向队列，不存在队列实例，仅存在节点间的关联关系
**AQS将请求共享资源的线程封装成CLH锁队列的结点，实现锁的分配**，用volatile修饰共享变量state，线程通过CAS去改变state，成功则获取锁成功，失败则进入等待队列，等待被唤醒

**注意：AQS是自旋锁：**在等待唤醒的时候，使用自旋（while(!cas())）的方式，不停地尝试获取锁，直到被其他线程获取成功

**实现了AQS的锁有：自旋锁、互斥锁、读锁写锁、条件产量、信号量、栅栏都是AQS的衍生物**

![](image.assets/CLH队列.png)



state是共享资源，其访问方式有如下三种：
getState();setState();compareAndSetState();



```java
public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer implements java.io.Serializable {
 
    /** AQS类内部维护一个FIFO的双向队列，负责同步状态的管理
    当前线程获取同步状态失败时，同步器会将当前线程以及等待状态等构造成一个节点Node并加入同步队列；
    当同步状态释放时，会把首节点中线程唤醒，使其再次尝试同步状态 */
    private transient volatile Node head;
    private transient volatile Node tail;
 
    //lock的占用状态；在ReentrantLock中，0空闲,1被占用，>1可重入占用
    private volatile int state;
```



```java
//AbstractQueuedSynchronizer内部类Node
static final class Node {
      	//两种资源共享方式
        //Share共享模式，可以被多个线程获取，比如读写锁中的读锁,Semaphore、CountDownLatch、ReadWriteLock，CyclicBarrier
        static final Node SHARED = new Node();
        //Exclusive独占模式，比如读写锁中的写锁,ReentrantLock
        static final Node EXCLUSIVE = null;
 
        //等待超时/被中断，获取锁的请求被取消,需要从同步队列中移除
        static final int CANCELLED =  1;

        //当前节点如果释放了同步状态或者被取消，将会通知后继节点，使后继节点的线程得以运行
        static final int SIGNAL    = -1;

        //节点在等待队列中，节点的线程等待在Condition上，当其他线程对Condition调用了signal()，该节点从等待队列 -> 条件队列
        static final int CONDITION = -2;

        //表示下一次共享式同步状态获取将会被无条件地传播下去
        static final int PROPAGATE = -3;

        /**
         * 当前节点的信号量状态 (1,0,-1,-2,-3)5种状态
         * 使用CAS更改状态，volatile保证线程可见性，高并发场景下，即被一个线程修改后，状态会立马让其他线程可见。
         */
        volatile int waitStatus;
      
        volatile Node prev;
        volatile Node next;
        volatile Thread thread;
    }
```



Node阻塞队列的体现：等待队列，条件队列

两种队列都是基于Node节点构建的，每个节点的Node都会有信号量，也就是属性waitSate



等待队列双向链表结构，每个节点记录的有前驱节点和后驱节点

![](image.assets/阻塞队列.png)



线程被阻塞之后会被放在等待队列里，head的属性thread是null，不放任何线程信息，仅为标记位





==条件队列是单向链表	只能存储独占==	Condition

![](image.assets/条件队列.png)



不同资源共享方式的同步器,争用共享资源的方式也不同



同步器的设计基于==模板方法==模式

AQS定义了独占模式的acquire()和release()方法，共享模式的acquireShared()和releaseShared()方法.还定义了抽象方法tryAcquire()、tryAcquiredShared()、tryRelease()和tryReleaseShared()由子类实现，tryAcquire()和tryAcquiredShared()分别对应独占模式和共享模式下的锁的尝试获取，就是通过这两个方法来实现公平性和非公平性，在尝试获取中，如果新来的线程必须先入队才能获取锁就是公平的，否则就是非公平的。这里可以看出AQS定义整体的同步器框架，具体实现放手交由子类实现



如果需要自定义同步器：

1. 使用者继承AbstractQueuedSynchronizer并重写指定的方法。（这些重写方法很简单，无非是对于共享资源state的获取和释放）
2. 将AQS组合在自定义同步组件的实现中，并调用其模板方法，而这些模板方法会调用使用者重写的方法

只需要实现state的获取和释放方式即可，至于具体线程等待队列的维护，AQS已经在顶层实现好了

| tryAcquire(int)       | 独占方式获取资源,需要CAS | 成功true，失败false                                       |
| --------------------- | ------------------------ | --------------------------------------------------------- |
| tryRelease(int)       | 独占方式释放资源,需要CAS |                                                           |
|                       |                          |                                                           |
| tryAcquireShared(int) | 共享方式获取资源,需要CAS | 负数失败；0成功，但没有剩余可用资源；正数成功，有剩余资源 |
| tryReleaseShared(int) | 共享方式释放资源,需要CAS | 释放后允许唤醒后续等待结点返回true，否则false             |









ReentrantLock为例：

state初始化0，表示未锁定

A线程lock()时，会调用tryAcquire()独占锁并将state+1.之后其他线程再想tryAcquire的时候就会失败，直到A线程unlock（）到state=0为止，其他线程才有机会获取该锁。A释放锁之前，自己也是可以重复获取此锁（state累加），这就是可重入的概念。
注意：获取多少次锁就要释放多少次锁，保证state是能回到0



以CountDownLatch为例，任务分N个子线程去执行，state就初始化 为N，N个线程并行执行，每个线程执行完之后countDown（）一次，state就会CAS减一。当N子线程全部执行完毕，state=0，会unpark()主调用线程，主调用线程就会从await()函数返回，继续之后的动作



一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现tryAcquire-tryRelease、tryAcquireShared-tryReleaseShared中的一种

但AQS也支持自定义同步器同时**实现独占和共享两种方式，如ReentrantReadWriteLock**



在acquire() acquireShared()两种方式下，线程在等待队列中都是忽略中断的，**acquireInterruptibly()/acquireSharedInterruptibly()是支持响应中断**的





**AQS的简单应用**
Mutex：不可重入互斥锁，锁资源（state）只有两种状态：0：未被锁定；1：锁定







### ReentrantLock



重入独占锁,当前持有该锁的线程能够多次获取该锁，无需等待



* 可中断
  * lockInterruptibly（）可以中断争夺锁的操作，抢锁的时候会check是否被中断，中断直接抛出异常，退出抢锁
  * Synchronized只有抢锁的过程，不可干预

- 快速反馈
  - trylock（） 和 trylock（tryTimes），不等待或者限定时间等待获取锁，更灵活。避免死锁发生
- 读写锁
  - ReentrantReadWriteLock类实现了读写锁的功能，类似于Mysql，锁自身维护一个计数器，读锁可以并发的获取，写锁只能独占。
  - synchronized全是独占锁
- Condition
  - ReentrantLock提供了比Sync更精准的线程调度工具，Condition，一个lock可以有多个Condition，比如在生产消费的业务下，一个锁通过控制生产Condition和消费Condition精准控制。



```java
public ReentrantLock() {
  			//默认非公平锁，可以避免大量的线程切换，提高性能
        sync = new NonfairSync();
    }

//构造时传入boolean来决定是不是公平/非公平锁
public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```





```java
//获得锁3种方式
lock(),	如果获取了锁立即返回，如果别的线程持有锁，当前线程则一直处于休眠状态，直到获取锁
tryLock(),	如果获取了锁立即返回true，如果别的线程正持有锁，立即返回false 
tryLock(long timeout,TimeUnit unit)	如果获取了锁定立即返回true，如果别的线程正持有锁，会等待参数给定的时间，在等待的过程中，如果获取了锁定，就返回true，如果等待超时返回false
  
  private final ReentrantLock lock = new ReentrantLock();
        lock.lock();					//在try外面加锁
        try {
            ...
        } finally {
            lock.unlock();}		//在finally解锁
```





获得独占非公平锁

```java
protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
        
        
final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            //0表示没有线程占用锁,可以拿锁
  					int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```







```java
//ReentrantLock
static final class FairSync extends Sync {

  			//加锁
        final void lock() {  acquire(1);  }

  			//获得独占公平锁
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
          //0表示没有线程占用锁,可以拿锁
            int c = getState();
            if (c == 0) {
                //队列里面没有线程在等待,才能抢锁(公平的体现)			cas算法原子操作改变state值，state值又被volitale修饰，保证并发下修改state的安全性
                if (!hasQueuedPredecessors() && compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)   throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
}



//AbstractQueuedSynchronizer
//加锁
public final void acquire(int arg) {
  			//加锁失败
        if (!tryAcquire(arg) &&
            //addWaiter()独占方式加入CLH等待队列
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }

private Node addWaiter(Node mode) {
  			//将当前线程封装为Node
        Node node = new Node(Thread.currentThread(), mode);
  
  			//将当前节点插入队尾(CAS)
        Node pred = tail;
        if (pred != null) {
            node.prev = pred;
          	//CAS
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
  			//tail不为null 或 CAS失败,enq()
        enq(node);
        return node;
}


private Node enq(final Node node) {
  	//死循环,不断尝试CAS
    for (;;) {
        Node t = tail;
      	//队列为空
        if (t == null) {
            //初始化，创建空的head,指向tail
            if (compareAndSetHead(new Node()))   tail = head;
          
        } else {
            node.prev = t;
            //当前节点set为tail
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```





AQS里面的线程唤醒不会唤醒所有的等待线程，只唤醒head的next（head不放线程），实现顺序唤醒

而object的notify方法和notifyall方法会唤醒所有线程,无序

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        //acquireQueued()唤醒线程
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}

final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
          	//当前结点的前驱结点
            final Node p = node.predecessor();
          	//当前驱结点是head，才tryAcquire()尝试获取锁，成功后顶替head		保证等待时间最长的节点head.next能最先获得锁(公平)
            if (p == head && tryAcquire(arg)) {
              	//获取同步状态成功，将当前结点设置为头结点
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            /**
             * 如果前驱节点不是Head，通过shouldParkAfterFailedAcquire判断是否应该阻塞
             通过park(this)阻塞当前线程,避免无意义的自旋，
             * 前驱节点信号量为-1，当前线程可以安全被阻塞
             */
            if (shouldParkAfterFailedAcquire(p, node) && parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```











#### 可重入的体现



```java
private ReentrantLock lock = new ReentrantLock(false);
    private Thread[] threads = new Thread[3];

    public Test() {
        for (int i = 0; i < 3; i++) {
            threads[i] = new Thread(() -> {
                for (int i1 = 0; i1 < 2; i1++) {
                    try {
                        lock.lock();
                        Thread.sleep(100);
                        System.out.println(Thread.currentThread().getName());
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } finally {
                        lock.unlock();
                    }
                }
            });
        }
        for (Thread thread : threads) {
            thread.start();
        }
    }
运行结果0-0-1-1-2-2
```



这段代码每个线程2次获取锁/释放锁

Thread0先获取锁，之后sleep 100ms，那么等待获取锁的同步队列为：

head -> thread1 -> thread2 -> thread0 -> thread1 -> thread2

从运行结果可知，第二次获取锁的还是thread0,与同步队列违背



==非公平锁获取锁最大的不一样的地方在于：线程外层函数获得锁后,再进入该线程的内层方法会自动获取锁 **(锁对象是同一个)**==

所以以上程序的同步队列应该为:

head -> thread1 -> thread2



公平/非公平锁源代码主要的2个不同点：

```java
//公平
final void lock() {
            acquire(1);
        }

public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }

protected final boolean tryAcquire(int acquires) {
        ...
            if (c == 0) {
              //队列里面没有线程在等待,才能抢锁(公平的体现)
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
           ...
        }


//非公平
final void lock() {
  					//直接尝试获取锁，无视同步队列
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
  					//直接获取失败时,才考虑同步队列
            else
                acquire(1);
        }

final boolean nonfairTryAcquire(int acquires) {
         ...
            if (c == 0) {
              //缺少了hasQueuedPredecessors(),不需要判断当前线程是否为head.next，也不需要判断当前线程是否在同步队列中
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
         ...
        }

```









#### unlock



```java
public void unlock() {
    sync.release(1);
}

public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }

protected final boolean tryRelease(int releases) {
            int c = getState() - releases;//1-1=0
            if (Thread.currentThread() != getExclusiveOwnerThread())    throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);//当前线程设置为null
            }
            setState(c);//当前状态设置为0
            return free;
        }

private void unparkSuccessor(Node node) {

        int ws = node.waitStatus;
        if (ws < 0)     compareAndSetWaitStatus(node, ws, 0);

        Node s = node.next;
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)   s = t;
        }
        if (s != null)  LockSupport.unpark(s.thread);
    }
```













- 3种让线程等待唤醒的方法
  - Object的wait()+notify()唤醒线程
    - wait()/notify(必须成对出现,并且**顺序不能反**)
  - Condition的await()+signal()方法唤醒线程
    - await()+signal()必须成对出现,并且**顺序不能反**
  - LockSupport类可以阻塞当前线程以及唤醒指定被阻塞的线程



### ReentrantReadWriteLnock



读写锁

读锁在同一时刻允许多个线程访问，通过重写int tryAcquireShared(int arg)以及boolean tryReleaseShared(int arg)

写锁为独占锁，通过重写boolean tryAcquire(int arg)以及boolean tryRelease(int arg)



#### Sync



ReentrantReadWriteLnock内部类Sync



AQS中只提供了int state表示锁的状态

**高16位表示读锁，低16位表示写锁**,读/写的state最大值均为65535，即所有获得了锁的线程的拿到锁的总次数(由于是重入锁，每个线程可以拿到n个锁)

由于读写锁主要的应用场景为多读少写，可以重写state的前24位为读锁，后8位为写锁

sharedCount!=0 表示分配了读锁，exclusiveCount != 0 表示分配了写锁

sharedCount和exclusiveCount 一般不会同时不为 0，==只有当线程占用了写锁，才可以重入获取读锁，反之不成立==

![](image.assets/ReentrantReadWriteLnock状态.png)



```java
abstract static class Sync extends AbstractQueuedSynchronizer {
  
       static final int SHARED_SHIFT   = 16;
       // 由于读锁用高位部分，所以读锁个数+1 = 状态值+ 2^16
       static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
       // 写锁的可重入的最大次数、读锁允许的最大数量
       static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1;
       // 写锁掩码，用于状态的低16位有效值
       static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;
       // 读锁计数，当前持有读锁的线程数
      static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }
      // 写锁计数(重入次数)
      static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }
  
  		//每个线程特定的 read 持有计数。存放在ThreadLocal，不需要线程安全
			static final class HoldCounter {
            int count = 0;
        		// 使用id而不是引用,避免保留垃圾。注意这是个常量
            final long tid = getThreadId(Thread.currentThread());
        }

        //采用继承来重写 initialValue()	这样就不用进行这样的处理：如果ThreadLocal没有当前线程的计数，则new一个，再放进ThreadLocal里。
        static final class ThreadLocalHoldCounter extends ThreadLocal<HoldCounter> {
            public HoldCounter initialValue() {
                return new HoldCounter();
            }
        }
  
  			//保存当前线程重入读锁的次数的容器。在读锁重入次数为 0 时移除
  			private transient ThreadLocalHoldCounter readHolds;

        //最近一个成功获取读锁的线程的计数。通常情况下，下一个释放线程是最后一个获取线程,节约查找时间
        private transient HoldCounter cachedHoldCounter;
  
		//firstReader是最后一个把 共享计数 从 0 -> 1 并且还未释放锁 的线程,如果不存在则为null
     * firstReader 不能导致gc失败保留垃圾，因此在 tryReleaseShared 里设置为null
     * firstReader及其计数firstReaderHoldCount是不会放入 readHolds 的
     */
    private transient Thread firstReader = null;
  
  	//firstReader 的重入计数
    private transient int firstReaderHoldCount;
  
    Sync() {
        readHolds = new ThreadLocalHoldCounter();
        setState(getState()); // 利用 volatile 确保 readHolds 的内存可见性
    }
```



Sync中有两个方法是抽象的，子类必须实现

```java
//公平模式
static final class FairSync extends Sync {
				//writerShouldBlock和readerShouldBlock方法都表示当有别的线程也在尝试获取锁时，是否应该阻塞
        final boolean writerShouldBlock() {
          //对于公平模式，hasQueuedPredecessors()表示等待队列是否还有线程。一旦有，当前线程也该被挂起
            return hasQueuedPredecessors();
        }
        final boolean readerShouldBlock() {
            return hasQueuedPredecessors();
        }
    }

//非公平模式
static final class NonfairSync extends Sync {
				// 写线程不公平,不需要阻塞,允许闯入
        final boolean writerShouldBlock() {    return false; }
        final boolean readerShouldBlock() {		return apparentlyFirstQueuedIsExclusive();  }
    }

//当head.next是独占线程，为了防止独占线程(写线程)饥饿等待，则后入线程排队，否则可以闯入
//也就是说,head.next想获取写锁，则读线程阻塞
final boolean apparentlyFirstQueuedIsExclusive() {
        Node h, s;
        return (h = head) != null &&
            (s = h.next)  != null &&
            !s.isShared()         &&
            s.thread != null;
    }
```



==如果当前全局处于读锁状态，且等待队列中第一个等待线程想获取写锁，那么当前线程能获取读锁的条件为==

1. 当前线程获取了写锁，还未释放
2. 当前线程获取了读锁，进行重入读锁

其它情况当前线程入队尾。这样处理提高效率，也避免想获取写锁的线程饥饿，得不到执行机会





例如：C请求写锁，由于当前AB拥有读锁，写锁获取失败，线程C入队列,阻塞并等待其他线程释放锁唤醒

![](image.assets/ReentrantReadWriteLnock案例1.png)

此时线程D想获取读锁，上面规则，队列中第一个等待线程C请求写锁，为避免写锁迟迟获取不到，并且D不是重入获取读锁，所以D也入队

![](image.assets/ReentrantReadWriteLnock案例2.png)



#### 获取读锁



在以下几种情况，获取读锁会失败：

（1）有线程持有写锁，且该线程不是当前线程

（2）写锁空闲 且  公平策略决定 读线程应当被阻塞，除了重入获取，其他获取读锁失败

（3）读锁数量达到最多，抛出异常

除了以上三种情况，该线程会循环尝试获取读锁直到成功

```java
public final void acquireShared(int arg){
    if(tryAcquireShared(arg) < 0){  // 调用子类, 获取共享 lock  返回 < 0, 表示失败
        doAcquireShared(arg);       // 调用 doAcquireShared 当前 线程加入 Sync Queue 里面, 等待获取 lock
    }
}

protected final int tryAcquireShared(int unused) {
            Thread current = Thread.currentThread();
            int c = getState();
  					//有线程持有写锁，且该线程不是当前线程，获取锁失败
            if (exclusiveCount(c) != 0 && getExclusiveOwnerThread() != current)    return -1;
  
  					//获取读锁计数
            int r = sharedCount(c);
  					//不应该阻塞 && 读锁数<MAX_COUNT && 设置state成功 -> 获取锁成功
            if (!readerShouldBlock() &&   r < MAX_COUNT && compareAndSetState(c, c + SHARED_UNIT)) {
              	if (r == 0) {
                  //下面对firstReader的处理：firstReader是不会放到readHolds里的，这样，在读锁只有一个的情况下，就避免了查找readHolds
                    firstReader = current;
                    firstReaderHoldCount = 1;
                } else if (firstReader == current) {
                    firstReaderHoldCount++;
                  
                  ///非 firstReader 读锁重入计数更新
                } else {
                    HoldCounter rh = cachedHoldCounter;
                    if (rh == null || rh.tid != current.getId())
                        cachedHoldCounter = rh = readHolds.get();
                    else if (rh.count == 0)
                        readHolds.set(rh);
                    rh.count++;
                }
                return 1;
            }
            //获取读锁失败，放入循环重试
            return fullTryAcquireShared(current);
        }


final int fullTryAcquireShared(Thread current) {
            HoldCounter rh = null;
            for (;;) {
                int c = getState();
                if (exclusiveCount(c) != 0) {
                    if (getExclusiveOwnerThread() != current)
                        return -1;     //1.有线程持有写锁，且该线程不是当前线程，获取锁失败
                    //2.有线程持有写锁，且该线程是当前线程，则应该放行让其重入获取锁，否则会造成死锁。
                } else if (readerShouldBlock()) {
                    //3.写锁空闲  且  公平策略决定 读线程应当被阻塞
                      // 下面的处理是说，如果是已获取读锁的线程重入读锁时，
                      // 即使公平策略指示应当阻塞也不会阻塞。
                      // 否则，这也会导致死锁的。
                    if (firstReader == current) {
                        // assert firstReaderHoldCount > 0;
                    } else {
                        if (rh == null) {
                            rh = cachedHoldCounter;
                            if (rh == null || rh.tid != current.getId()) {
                                rh = readHolds.get();
                                if (rh.count == 0)
                                    readHolds.remove();
                            }
                        }
                        //4.需要阻塞且是非重入(还未获取读锁的)，获取失败。
                        if (rh.count == 0)
                            return -1;
                    }
 
                }
                //5.写锁空闲  且  公平策略决定线程可以获取读锁
                if (sharedCount(c) == MAX_COUNT)//6.读锁数量达到最多
                    throw new Error("Maximum lock count exceeded");
                //7. 申请读锁成功，下面的处理跟tryAcquireShared是类似的。
                if (compareAndSetState(c, c + SHARED_UNIT)) {
                    if (sharedCount(c) == 0) {
                        firstReader = current;
                        firstReaderHoldCount = 1;
                    } else if (firstReader == current) {
                        firstReaderHoldCount++;
                    } else {
                        if (rh == null)
                            rh = cachedHoldCounter;
                        if (rh == null || rh.tid != current.getId())
                            rh = readHolds.get();
                        else if (rh.count == 0)
                            readHolds.set(rh);
                        rh.count++;
                        cachedHoldCounter = rh; // cache for release
                    }
                    return 1;
                }
```





#### 获取共享lock



```java
private void doAcquireShared(int arg){
    final Node node = addWaiter(Node.SHARED);       // 1. 将当前的线程封装成 Node 加入到 Sync Queue 里面
    boolean failed = true;
 
    try {
        boolean interrupted = false;
        for(;;){
            final Node p = node.predecessor();      // 2. 获取当前节点的前继节点 (当一个n在 Sync Queue 里面, 并且没有获取 lock 的 node 的前继节点不可能是 null)
            if(p == head){
                int r = tryAcquireShared(arg);      // 3. 判断前继节点是否是head节点(前继节点是head, 存在两种情况 (1) 前继节点现在占用 lock (2)前继节点是个空节点, 已经释放 lock, node 现在有机会获取 lock); 则再次调用 tryAcquireShared 尝试获取一下
                if(r >= 0){
                    setHeadAndPropagate(node, r);   // 4. 获取 lock 成功, 设置新的 head, 并唤醒后继获取  readLock 的节点
                    p.next = null; // help GC
                    if(interrupted){               // 5. 在获取 lock 时, 被中断过, 则自己再自我中断一下(外面的函数可能需要这个参数)
                        selfInterrupt();
                    }
                    failed = false;
                    return;
                }
            }
 
            if(shouldParkAfterFailedAcquire(p, node) && // 6. 调用 shouldParkAfterFailedAcquire 判断是否需要中断(这里可能会一开始 返回 false, 但在此进去后直接返回 true(主要和前继节点的状态是否是 signal))
                    parkAndCheckInterrupt()){           // 7. 现在lock还是被其他线程占用 那就睡一会, 返回值判断是否这次线程的唤醒是被中断唤醒
                interrupted = true;
            }
        }
    }finally {
        if(failed){             // 8. 在整个获取中出错(比如线程中断/超时)
            cancelAcquire(node);  // 9. 清除 node 节点(清除的过程是先给 node 打上 CANCELLED标志, 然后再删除)
        }

```



独占锁模式获取成功以后设置头结点然后返回中断状态，结束流程。而共享锁模式获取成功以后，调用了setHeadAndPropagate方法，从方法名就可以看出除了设置新的头结点以外还有一个传递动作，一起看下代码：



```java
//两个入参，一个是当前成功获取共享锁的节点，一个就是tryAcquireShared方法的返回值，注意上面说的，它可能大于0也可能等于0
    private void setHeadAndPropagate(Node node, int propagate) {
        Node h = head; //记录当前头节点
        //设置新的头节点，即把当前获取到锁的节点设置为头节点
        //注：这里是获取到锁之后的操作，不需要并发控制
        setHead(node);
        //这里意思有两种情况是需要执行唤醒操作
        //1.propagate > 0 表示调用方指明了后继节点有可能需要被唤醒，因为此方法是获取读锁过程调用，那么后面节点很可能也要获取读锁
        //2.头节点后面的节点需要被唤醒（waitStatus<0），不论是老的头结点还是新的头结点
        if (propagate > 0 || h == null || h.waitStatus < 0) {
            Node s = node.next;
            //如果当前节点的后继节点是共享类型获取没有后继节点，则进行唤醒
            //这里可以理解为除非明确指明不需要唤醒（后继等待节点是独占类型），否则都要唤醒
            //这里的初衷是   后一个节点正好是共享节点，就唤醒，实现共享，独占有锁释放时候唤醒
            if (s == null || s.isShared())
                //后面详细说
                doReleaseShared();
        }
    }
 
    private void setHead(Node node) {
        head = node;
        node.thread = null;
        node.prev = null;
    }
```



这个唤醒操作在releaseShared()方法里也会调用。唤醒后面想获取锁的节点。



```java
private void doReleaseShared() {
        for (;;) {
            //唤醒操作由头结点开始，注意这里的头节点已经是上面新设置的头结点了
            //其实就是唤醒上面新获取到共享锁的节点的后继节点
            Node h = head;
            if (h != null && h != tail) {
                int ws = h.waitStatus;
                //表示后继节点需要被唤醒
                if (ws == Node.SIGNAL) {
                    //这里需要控制并发，因为入口有setHeadAndPropagate跟releaseShared两个，避免两次unpark
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                        continue;      
                    //执行唤醒操作      
                    unparkSuccessor(h);
                }
                //如果后继节点暂时不需要唤醒，则把当前节点状态设置为PROPAGATE确保以后可以传递下去
                else if (ws == 0 &&
                         !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                    continue;                
            }
            //如果头结点没有发生变化，表示设置完成，退出循环
            //如果头结点发生变化，比如说其他线程获取到了锁，为了使自己的唤醒动作可以传递，必须进行重试
            if (h == head)                   
                break;
        }
```



#### 读锁的释放



```java

 public final boolean releaseShared(int arg) {
        if (tryReleaseShared(arg)) {
            doReleaseShared();
            return true;
        }
        return false;
    }
释放锁tryReleaseShared由子类Sync实现
 
protected final boolean tryReleaseShared(int unused) {
    Thread current = Thread.currentThread();
    // 清理firstReader缓存 或 readHolds里的重入计数
    if (firstReader == current) {
        // assert firstReaderHoldCount > 0;
        if (firstReaderHoldCount == 1)
            firstReader = null;
        else
            firstReaderHoldCount--;
    } else {
        HoldCounter rh = cachedHoldCounter;
        if (rh == null || rh.tid != current.getId())
            rh = readHolds.get();
        int count = rh.count;
        if (count <= 1) {
            // 完全释放读锁
            readHolds.remove();
            if (count <= 0)
                throw unmatchedUnlockException();
        }
        --rh.count; // 主要用于重入退出
    }
    // 循环在CAS更新状态值，主要是把读锁数量减 1
    for (;;) {
        int c = getState();
        int nextc = c - SHARED_UNIT;
        if (compareAndSetState(c, nextc))
            // 释放读锁对其他读线程没有任何影响，
            // 但可以允许等待的写线程继续，如果读锁、写锁都空闲。
            return nextc == 0;
    }
}
```





#### 写锁的获取



写锁的获取和ReentrantLock独占锁的锁获取过程几乎一样，除了tryAcquire（）方法，要考虑读锁的情况

在以下情况，写锁获取失败：

（1） 写锁为0，读锁不为0   或者写锁不为0，且当前线程不是已获取独占锁的线程，锁获取失败。

（2）写锁数量已达到最大值，写锁获取失败。

（3）当前线程应该阻塞，或者设置同步状态state失败，获取锁失败



```java
public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }


protected final boolean tryAcquire(int acquires) {
 
            Thread current = Thread.currentThread();
            int c = getState();
            int w = exclusiveCount(c);
            if (c != 0) {
                // 1.写锁为0，读锁不为0    或者写锁不为0，且当前线程不是已获取独占锁的线程，锁获取失败
                if (w == 0 || current != getExclusiveOwnerThread())
                    return false;
                //2. 写锁数量已达到最大值，写锁获取失败
                if (w + exclusiveCount(acquires) > MAX_COUNT)
                    throw new Error("Maximum lock count exceeded");
                // Reentrant acquire
                setState(c + acquires);
                return true;
            }
            //3.当前线程应该阻塞，或者设置同步状态state失败，获取锁失败。
            if (writerShouldBlock() ||
                !compareAndSetState(c, c + acquires))
                return false;
            setExclusiveOwnerThread(current);
            return true;
        }
```





#### 写锁的释放



```java
public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
 
protected final boolean tryRelease(int releases) {
            if (!isHeldExclusively())
                throw new IllegalMonitorStateException();
            int nextc = getState() - releases;
            boolean free = exclusiveCount(nextc) == 0;
            if (free)
                setExclusiveOwnerThread(null);
            setState(nextc);
            return free;
        }
```



#### 总结



获取读锁和释放读锁

获取锁的过程：

当线程调用acquireShared()申请获取锁资源时，如果成功，则进入临界区
当获取锁失败时，则创建一个共享类型的节点并进入一个FIFO等待队列，然后被挂起等待唤醒。
当队列中的等待线程被唤醒以后就重新尝试获取锁资源，如果成功则唤醒后面还在等待的共享节点并把该唤醒事件传递下去，即会依次唤醒在该节点后面的所有共享节点，然后进入临界区，否则继续挂起等待
释放锁过程：

当线程调用releaseShared()进行锁资源释放时，如果释放成功，则唤醒队列中等待的节点，如果有的话



跟独占锁相比，共享锁的主要特征在于当一个在等待队列中的共享节点成功获取到共享锁后,必须依次唤醒后面所有可以跟它一起共享当前锁资源的节点，毫无疑问，这些节点必须也是在等待共享锁（这是大前提，如果等待的是独占锁，那前面已经有一个共享节点获取锁了，它肯定是获取不到的）

当共享锁被释放的时候，可以用读写锁为例进行思考，当一个读锁被释放，此时不论是读锁还是写锁都可以竞争资源













读写锁还提供了一些新的方法，比如final int getReadHoldCount()，返回当前线程获取读锁的次数。由于读状态保存的是所有获取读锁的线程读锁次数的总和，因此每个线程自己的读锁次数需要单独保存，引入了ThreadLocal，由线程自身维护



![](image.assets/ReentrantReadWriteLnock继承关系.png)































### synchronized VS lock



竞争激烈时，lock性能会比synchronize好，**竞争不激烈时，synchronize比lock性能好，synchronize会根据锁的竞争情况，从偏向锁-->轻量级锁-->重量级锁升级**



| 类别         | synchronized                                   | Lock                                          |
| :----------- | :--------------------------------------------- | :-------------------------------------------- |
| 存在层次     | 关键字，在jvm层面上                            | 是一个接口                                    |
| 锁的释放     | 执行结束/**发生异常**，jvm**自动**让线程释放锁 | 必须在finally中**手动释放**，容易死锁         |
| 等待/通知    | wait()+notify()                                | Condition+signalAll()                         |
| 获取锁的状态 | 无法判断                                       | 可以判断(tryLock)                             |
| 锁类型       | **可重入** 不可中断 ==只能非公平+悲观==        | **支持重入** ==公平/不公平== CAS **中断等待** |
| 性能         | 少量同步                                       | 大量同步                                      |
|              | 隐式                                           | 显示                                          |
|              |                                                | 只有代码块锁,没有方法锁                       |
| 响应中断     | 线程将一直等待,不能响应中断                    | 支持响应中断                                  |





### LockSupport



LockSupport是线程阻塞工具类，所有的方法都是静态方法，可以让线程在任意位置阻塞，阻塞之后也有对应的唤醒方法



通过park()+unpark(thread)实现阻塞/唤醒	调用的Unsafe中的native代码





Permit(许可）概念

每个线程都有许可(permit),只有两个值1/0，默认0

可以把许可看成是一种(0,1)信号量，但与Semaphore不同的是，permit不能累加



```java
//park()阻塞
//permit默认是0，所以一开始调用park()，当前线程阻塞，直到别的线程将当前线程的permit设置为1时, park()会被唤醒，将permit设置为0并返回
public static void park() {
    UNSAFE.park(false, 0L);
}

//unpark()唤醒
//调用unpark(thread)后，会将thread的permit设置成1(多次unpark()不会累加),自动唤醒之前阻塞的LockSupport.park()
public static void unpark(Thread thread) {
        if (thread != null)
            UNSAFE.unpark(thread);
    }
```





LockSupport不用持有锁/加锁，性能高

先后顺序，不容易导致卡死(因为unpark获得了一个凭证，之后再调用park方法，不会阻塞)

















### Condition



线程同步

synchronized

wait()+notify()可以实现等待/通知模式

Condition接口也提供了类似Object的监视器方法，与Lock配合可以实现等待/通知模式





Object和Condition接口的对比

![](image.assets/Condition区别.png)



condition对象依赖于lock对象



```java
public Lock lock = new ReentrantLock();
    public Condition condition = lock.newCondition();

    public static void main(String[] args) {
        Test useCase = new Test();
        ExecutorService executorService = Executors.newFixedThreadPool(2);
        executorService.execute(useCase::conditionWait);
        executorService.execute(useCase::conditionSignal);
    }

    public void conditionWait() {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "拿到锁了");
            System.out.println(Thread.currentThread().getName() + "等待信号");
            condition.await();
            System.out.println(Thread.currentThread().getName() + "拿到信号");
        } catch (Exception ignored) {

        } finally {
            lock.unlock();
        }
    }

    public void conditionSignal() {
        lock.lock();
        try {
            Thread.sleep(5000);
            System.out.println(Thread.currentThread().getName() + "拿到锁了");
            condition.signal();
            System.out.println(Thread.currentThread().getName() + "发出信号");
        } catch (Exception ignored) {

        } finally {
            lock.unlock();
        }
    }

pool-1-thread-1拿到锁了
pool-1-thread-1等待信号
pool-1-thread-2拿到锁了
pool-1-thread-2发出信号
pool-1-thread-1拿到信号
```



将Condition对象作为成员变量。当调用await()后，当前线程会释放锁并在此等待

而其他线程调用Condition对象的signal()方法，通知当前线程后，当前线程才从await()返回，并且在返回前已经获取了锁





condition是条件队列

当一个线程在调用了await方法以后，直到线程等待的某个条件为真的时候才会被唤醒,为线程提供了更加简单的等待/通知模式



Condition必须配合锁一起使用，因为对共享状态变量的访问发生在多线程环境下。一个Condition的实例必须与一个Lock绑定，因此Condition一般都是作为Lock的内部实现



```java
await() 	当前线程在接到信号或被中断之前一直处于等待状态
await(long time, TimeUnit unit) ：造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态
awaitNanos(long nanosTimeout) ：造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。返回值表示剩余时间，如果在nanosTimesout之前唤醒，那么返回值 = nanosTimeout - 消耗时间，如果返回值 <= 0 ,则可以认定它已经超时了
awaitUninterruptibly() ：造成当前线程在接到信号之前一直处于等待状态。【注意：该方法对中断不敏感】
awaitUntil(Date deadline) ：造成当前线程在接到信号、被中断或到达指定最后期限之前一直处于等待状态。如果没有到指定时间就被通知，则返回true，否则表示到了指定时间，返回返回false
  
signal() ：唤醒一个等待线程。该线程从等待方法返回前必须先获得与Condition相关的锁
signal()All ：唤醒所有等待线程。能够从等待方法返回的线程必须先获得与Condition相关的锁
```







ConditionObject是AQS的内部类

每个Condition对象都包含一个队列(条件队列)。条件队列是一个FIFO的队列，在队列中的每个节点都包含了一个线程引用，该线程就是在Condition对象上条件的线程



#### await



线程调用了Condition.await()，会释放锁、构造成节点加入条件队列并进入等待状态

新增节点就是将尾部节点指向新增的节点。节点引用更新本来就是在获取锁以后的操作，所以不需要CAS保证。同时也是线程安全的操作

![](image.assets/条件队列插入.png)



当从await方法返回的时候,会获取condition相关联的锁。当队列中的节点被唤醒的时候，则唤醒节点的线程开始尝试获取同步状态

如果不是通过其他线程调用Condition.signal()方法唤醒，而是对条件线程进行中断，则会抛出InterruptedException







#### signal



在调用signal()方法前,必须先判断是否获取到了锁。接着获取条件队列的首节点，将其移动到等待队列

并利用LockSupport唤醒节点中的线程,被唤醒的线程将从await方法中的while循环中退出。随后加入到等待队列的竞争中

竞争成功则会返回到await()之前的状态



![](image.assets/条件队列弹出.png)

调用Condition的signal()方法，将会唤醒在等待队列中等待最长时间的节点（head.next）





#### 总结



调用await()后，将当前线程加入Condition条件队列中,并释放锁,否则别的线程就无法拿到锁 -> 死锁

自旋(while)挂起，不断检测节点是否在等待队列中，如果是则尝试获取锁，否则挂起

当线程被signal方法唤醒，被唤醒的线程将从await()方法中的while循环中退出来，然后调用acquireQueued()方法竞争同步状态







#### 生产者消费者模式



```java
private LinkedList<Object> buffer;    //生产者容器
    private int maxSize;           //容器最大值
    private Lock lock;
    private Condition fullCondition;
    private Condition notFullCondition;

    BoundedQueue(int maxSize) {
        this.maxSize = maxSize;
        buffer = new LinkedList<>();
        lock = new ReentrantLock();
        fullCondition = lock.newCondition();
        notFullCondition = lock.newCondition();
    }

    /**
     * 生产者
     */
    public void put(Object obj) throws InterruptedException {
        lock.lock();
        try {
            while (maxSize == buffer.size()) {
                notFullCondition.await();       //满了，添加的线程进入等待状态
            }
            buffer.add(obj);
            fullCondition.signal(); //通知
        } finally {
            lock.unlock();
        }
    }

    /**
     * 消费者
     */
    public Object get() throws InterruptedException {
        Object obj;
        lock.lock();
        try {
            while (buffer.size() == 0) { //队列中没有数据了 线程进入等待状态
                fullCondition.await();
            }
            obj = buffer.poll();
            notFullCondition.signal(); //通知
        } finally {
            lock.unlock();
        }
        return obj;
    }
```











## 线程池（thread pool）



创建和销毁对象是很费时间的，**虚拟机将试图跟踪每一个对象，以便能够在对象销毁后进行垃圾回收**

线程池也**利于管理线程的个数与活跃数,统一分配、调优和监控**





### 线程池执行流程



任务被提交到线程池，会先判断当前线程数量是否小于corePoolSize，如果小于则创建线程来执行提交的任务，否则将任务放入workQueue队列，如果workQueue满了，则判断当前线程数量是否小于maximumPoolSize,如果小于则创建线程执行任务，否则就会调用handler，以表示线程池拒绝接收任务





### 线程池参数





 	

线程空闲时间 

空闲的线程保留的时间 keepAliveTime

阻塞队列大小 



**corePoolSize：线程池的基本大小**，当提交一个任务到线程池时，线程池会创建一个线程来执行任务，即使其他空闲的基本线程能够执行新任务也会创建线程，等到需要执行的任务数大于线程池基本大小时就不再创建。说白了就是，即便是线程池里没有任何任务，也会有corePoolSize个线程在候着等任务。

**maximumPoolSize:最大线程数**

corePollSize	核心线程数	平时的流量需要的线程数

**keepAliveTime:线程的存活时间**。当线程池里的线程数大于corePoolSize时，如果等了keepAliveTime时长还没有任务可执行，则线程退出。

**unit**：指定keepAliveTime的单位，比如秒:TimeUnit.SECONDS

**workQueu**e：用于保存等待执行任务的**阻塞队列**，提交的任务将会被放到这个队列里

**threadFactory：线程工厂，用来创建线程**,给线程起名

**handler：拒绝策略**，即当线程和队列都已经满了的时候，应该采取什么样的策略来处理新提交的任务。默认策略是AbortPolicy（抛出异常），其他的策略还有：CallerRunsPolicy(只用调用者所在线程来运行任务)、DiscardOldestPolicy(丢弃队列里最近的一个任务，并执行当前任务)、DiscardPolicy(不处理，丢弃掉)







### 几种常见的线程池



　　可以创建（**Executors.newXXX**）3种类型的ThreadPoolExecutor：**FixedThreadPool**、**SingleThreadExecutor**、**CachedThreadPool**。

- FixedThreadPool

  ：

  可重用固定线程数的线程池

  。（适用于负载比较重的服务器）

  - **FixedThreadPool使用无界队列LinkedBlockingQueue作为线程池的工作队列**
  - 该线程池中的线程数量始终不变。当有一个新的任务提交时，线程池中若有空闲线程，则立即执行。若没有，则新的任务会被暂存在一个任务队列中，待有线程空闲时，便处理在任务队列中的任务。

- SingleThreadExecutor

  ：

  只会创建一个线程执行任务。

  （适用于需要保证顺序执行各个任务；并且在任意时间点，没有多线程活动的场景。）

  - ***\*SingleThreadExecutorl也使用无界队列LinkedBlockingQueue作为工作队列\****
  - 若多余一个任务被提交到该线程池，任务会被保存在一个任务队列中，待线程空闲，按先入先出的顺序执行队列中的任务。

- **CachedThreadPool：\**是一个会根据需要调整线程数量的线程池\**。**

  （大小无界，适用于执行很多的短期异步任务的小程序，或负载较轻的服务器）

  - **CachedThreadPool使用没有容量的SynchronousQueue作为线程池的工作队列，但CachedThreadPool的maximumPool是无界的。**
  - 线程池的线程数量不确定，但若有空闲线程可以复用，则会优先使用可复用的线程。若所有线程均在工作，又有新的任务提交，则会创建新的线程处理任务。所有线程在当前任务执行完毕后，将返回线程池进行复用。

- **ScheduledThreadPool**：继承自ThreadPoolExecutor。它主要用来**在给定的延迟之后运行任务，或者定期执行任务**。使用DelayQueue作为任务队列。



### 线程池的工作队列





当没有足够的线程去处理任务时，可以将任务放进队列中，以队列先进先出的特性来执行工作任务

核心线程满了，进队列，队列也满了，创建新线程，直到达到最大线程数，之后再超出，会进入拒绝rejectedExecution



- **ArrayBlockingQueue**：是一个**基于数组结构的有界阻塞队列**，此队列按FIFO（先进先出）原则对元素进行排序。
- **LinkedBlockingQueue**：是一个**基于链表结构的阻塞队列**，此队列按FIFO排序元素，吞吐量通常要高于ArrayBlockingQueue。静态工厂方法Executors.newFixedThreadPool()使用了这个队列。
- **SynchronousQueue**：是一个**不存储元素的阻塞队列**。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于Linked-BlockingQueue，静态工厂方法Executors.newCachedThreadPool使用了这个队列。
- **PriorityBlockingQueue**：一个**具有优先级的无限阻塞队列**。





























Executor		总接口,只定义了execute()执行线程方法

ExecutorService extends Executor 子接口,定义了shutdown()关闭 submit()等方法

```
abstract class AbstractExecutorService implements ExecutorService
实现了ExecutorService的方法
```



class ThreadPoolExecutor extends AbstractExecutorService

```
void execute(Runnable command){}	//执行Runnable线程,无返回值
```



```
<T>Future<T> submit(Callable<T> task)	//执行Callable线程,有返回值
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



























# 反射



在**运行状态**中，对于任意一个实体类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性；这种动态获取信息以及动态调用对象方法的功能称为java语言的反射机制。



**在程序运行过程中，操作对象**。可以提高程序扩展性和复用性，**解耦**



适用场景

* 在编译时不知道该对象或类可能属于哪些类，通过反射可以使程序代码访问装载到JVM中的类的内部信息
* 反射提高了灵活性和扩展性，**低耦合**。它允许**程序创建和控制任何类的对象，无需提前硬编码**目标类
* Struts、Hibernate、Spring 在实现过程中都采用了该技术
* 反射是**解释操作**，用于字段和方法接入时效率低
* 会模糊程序内部逻辑：程序人员希望在源代码中看到程序的逻辑，反射等绕过了源代码的技术，因而会带来维护问题。**反射代码比直接代码更复杂**





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











## Field



代表类的成员变量。**成员变量（字段）!=成员属性**。User类中有name变量，则它有name字段。如果**没有get/setName，就没有name属性**。**如果有get/set,不管字段是否存在，都认为有这个属性**



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

setAccessible(true)忽略访问修饰符



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
  * 破坏了单例模式

 



## 注解



注解本身并不起任何作用,只作为标识	通过反射来获取注解,再根据注解的参数执行业务



* 作用分类：
  * 编写文档：通过代码中标识的注解生成文档（Swagger）
  * 代码分析：通过代码里的注解对代码进行分析（逻辑判断）
  * 编译检查：通过代码里对应的注解让编译器实现基本的编译检查（Override，Deprecated，FunctionalInterface）

* JDK中预定义的一些注解
  * Override：检测该注解标识的方法是否继承自父类
  * Deprecated：标识方法、类、字段等已经过时，后续的版本可能会将其移除
  * SuppressWarnings：压制警告



### 元注解



==用于描述注解的适用范围==



* @Target	作用范围
  
* ```shell
  Type：作用于类
  METHOD：作用于方法
  FIELD：作用于字段
  PACKAGE
  ElementType取值
  PARAMETER
  TYPE_PARAMETER  标注类型参数
  CONSTRUCTOR
  LOCAL_VARIABLE  局部变量
  ANNOTATION_TYPE  注解类
  TYPE_USE  所有类型
  ```

* @Retention：描述注解被保留的阶段
  
  * RetentionPolicy.RUNTIME：当前描述的注解，会保留到class字节码文件中，并被jvm读取到	**默认**
  
  * RetentionPolicy.SOURCE：注解只保留在源文件，当Java文件编译成class文件的时候，注解被遗弃
  
  * RetentionPolicy.CLASS：注解被保留到class文件，但jvm加载class文件时候被遗弃
  
    生命周期长度 SOURCE < CLASS < RUNTIME
  
    需要在运行时动态获取注解信息，那只能用RUNTIME注解，比如@Deprecated使用RUNTIME注解
     在编译时进行预处理操作，比如生成一些辅助代码（如 ButterKnife），就用 CLASS注解；
     只是检查性的操作，比如 @Override 和 @SuppressWarnings，使用SOURCE 注解。
  
* @Documented：描述注解是否被抽取到javadoc中

* @Inherited：描述注解是否可以被继承

* @Repeatable:指明注解为可重复注解，可以在同一个地方多次使用

* @Scheduled

  ```shell
  #Cron 定时时间		允许正则表达式
  @Scheduled(cron = "0 0 5 * * ?")      [秒] [分] [小时] [日] [月] [周] [年]
  ?    不指定值
  \-    区间
  ,    指定多个值
  /    递增触发。秒”5/15” 表示从5秒开始，每增15秒触发
  L    最后。对于日字段，表示当月的最后一天.对于周字段上设置”6L”这样的格式,则表示“本月最后一个星期五”
  W   离指定日期的最近的工作日(周一至周五). 例如在日字段上置”15W”，表示离每月15号最近的那个工作日触发。如果15号正好是周六，则找最近的周五(14号)触发, 如果15号是周未，则找最近的下周一(16号)触发.如果15号正好在工作日(周一至周五)，则就在该天触发。如果指定格式为 “1W”,它则表示每月1号往后最近的工作日触发。如果1号正是周六，则将在3号下周一触发。(注，”W”前只能设置具体的数字,不允许区间”-“)。
  \#    序号(表示每月的第几个周几)，例如在周字段上设置”6#3”表示在每月的第三个周六.注意如果指定”#5”,正好第五周没有周六，则不会触发该配置
  ’L’和‘W’组合使用。在日字段上设置”LW”,则表示在本月的最后一个工作日触发；周字段的设置，若使用英文字母是不区分大小写的，即MON与mon相同
  
  # zone时区.一般留空
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
  ```











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

* 如果定义了属性，在使用属性的时需要赋值

* ==只有一个属性且名称value，则可以省略==

* 数组赋值时用{}封装

* ==属性中的返回值==类型有下列取值：
  * 基本数据类型
  * String
  * 枚举
  * 注解
  * 以上类型的数组



## 泛型擦除



```java
    List<User> list= new ArrayList<>();
    Class<? extends List> listClass = list.getClass();
    Method add = listClass.getDeclaredMethod("add", Object.class);
		add.setAccessible(true);
  //通过invoke()在运行时向list插入Integer数据,避免了编译时的泛型检验
    add.invoke(list, 5);
```





# JVM





## 6种存储方式



1．寄存器（register）**最快,位于处理器内部**。数量有限，由编译器根据需求分配。**无法直接控制**

2．栈（stack）位于**随机访问存储器RAM**，**通过“栈指针”从处理器获得直接支持**。==指针向下移动，分配新内存；向上移动，释放内存==。创建程序时，**编译器必须知道栈内所有数据的大小和生命周期**，因为它必须生成相应的代码，以便上下移动指针,**限制了程序的灵活性**

3．堆（heap）通用内存池,位于**随机访问存储器RAM**，用于存放对象。堆不同于堆栈的好处是：**编译器不需要知道要分配多少空间，也不必知道数据的生命周期,灵活性高,效率低**

4．静态存储（static storage）位于**随机访问存储器RAM**,存放运行时一直存在的数据

5．常量存储（constant storage）位于**代码内部**，这样安全。**有时在嵌入式中常量会和其它部分隔离，位于只读存储器ROM（read-only memory）**

6．非RAM存储（non-RAM storage）位于**程序外**，不受程序控制，没有运行时也存在,**字节流/持久化对象**,对象被转化成可以存放在其它媒介上的事物，在需要时可恢复



## JMM



![](image.assets/内存模型.png)

### 栈

**-Xss**



* 虚拟机和物理机都有代码执行能力,物理机执行引擎建立在处理器、硬件指令集、操作系统层面,虚拟机执行引擎由自己实现，用于执行虚拟机字节码指令集
* 执行时输入字节码文件,进行字节码解析,可能通过解释器执行,也可能通过执行编译器产生本地代码执行
* 虚拟机栈==分配的内存大小确定==
  
  * ==对象引用,基本类型,指令地址==
  * ==局部变量表==(大小固定,运行期间不变)
  * 存放**对象引用和局部变量**
    * 32位变量槽（Slot），Slot至少能存放一个boolean、byte、char、short、int、float、reference类型的数据
    * **局部变量必须手动赋值**，不会被被赋初值,不像类变量那样有加载过程中有准备阶段
  * ==操作数栈==(工作空间)
  * 后入先出，最大深度在编译时确定。每32位对应栈容量1
    * 方法在执行过程中，各种字节码指令往操作数栈中读/写(出入栈)
    * Jvm的解释执行引擎就是基于操作数栈的执行引擎
  * ==动态连接==
    * **每个栈帧都包含一个指向运行时常量池中该栈帧所属方法的引用**,从而支持方法调用过程中的动态连接
    * class文件的常量池中有大量的符号引用，字节码中的方法调用以常量池指向的方法的符号引用作为参数,这些符号引用一部分会在类加载阶段（解析）或首次使用的时转化为直接引用，这种转化成为静态解析，另一部分成为动态连接
  * ==方法返回地址(出口)==
  * 只有正常/异常完成能退出方法
      * 正常出口：执行引擎遇到返回的字节码指令，读取栈帧中的方法返回地址，将返回值传递给上层的方法调用者
      * 异常出口：遇到未处理的异常(本地异常表没有匹配的异常处理器),执行引擎不会读取方法返回地址，上层调用者不会得到任何返回值
    
  * ==方法退出:把当前栈帧出栈。恢复上层方法的局部变量表和操作数栈，把返回值压入调用者栈帧的操作数栈中，调整PC计数器,执行下一条指令==
    * 一般把动态连接、方法返回地址和其他附加信息全部归为一类，成为栈帧信息
  * 线程私有，生命周期与线程相同,方法调用进栈,结束出栈
  * 效率高.由操作系统自动分配,有专门的寄存器存放栈的地址，压栈出栈有专门指令
  * 按先后定义的顺序依次压栈，**相邻变量的地址之间不会存在其它变量**。栈的内存地址由高到低，**后定义的变量地址低于先定义的变量**
  * **如果线程请求的深度大于虚拟机所允许的深度，StackOverflowError**
  * **如果虚拟机栈动态扩展，而扩展时无法申请到足够的内存，OutOfMemoryError**
* 本地方法栈
  
  * 为虚拟机使用到本地方法服务（native）

==栈是运行时单位，解决程序运行时方法调用/执行，堆是存储单位，解决数据存储==



### 堆 heap

-Xms -Xmx



* 只有1个,被所有线程共享,虚拟机启动时创建
* 存储==对象和对象的class信息(操作指令)==
* ==堆是gc的主要区域==
  * 新生代   分三个区,默认占比 8:1:1,方便采用**复制-清除策略**
  * 区分空闲/使用区,将存活的对象复制进空闲区，**避免碎片问题**。虽然复制后使用区没有碎片，但下一次GC，Eden和使用区里都存在需要回收的对象,从而导致碎片
  * **Survivor from/to区交替空闲** -> 新生代实际可用90%
    * Eden主要存放新创建对象,==分配内存时需要加锁==
      * 线程在Eden上被分配独享的空间TLAB（Thread Local Allocation Buffer）,==在TLAB分配内存不需要加锁==，JVM给线程中的对象分配内存时尽量在TLAB分配,对象过大或TLAB用完时，则仍在堆上进行分配
      * Eden满时
        * **进入老年代的对象大小在GC前未知**
        * 之前晋升到老年代的平均值>老年代剩余空间，**full GC**
        * < 则判断参数HandlePromotionFailure(是否允许担保失败,默认允许)，有担保则**Minor GC**
          * 大对象进入年老代,保证Eden区空间充足  -XX:PretenureSizeThreshold=
          * 长期存活进入老年代，避免from/to重复复制,以及Survivor空间不足  -XX:MaxTenuringThreshold= 默认15次
          * **动态对象年龄判断**,年龄相同的对象>Survivor/2，**超出部分放入老年代,保证老年代空间充足**
      * GC后,使用/空闲区互换,**年龄+1**
  * 年老代  存活时间较久，较大的对象
* 堆的申请和释放工作由程序员控制，容易**内存泄漏**->己动态分配的堆内存未释放或无法释放



![](image.assets/堆分区.png)



#### TLAB

-XX:TLABSize



Thread Local Allocation Buffer



堆区线程共享,由于频繁创建实例,在并发环境下从堆区中划分内存空间是线程不安全的,但为避免多个线程操作同一地址,需要使用加锁等机制,将导致分配速度降低,所以需要TLAB



**JVM为每个线程在Eden区分配了一个私有缓存区**(一般只占Eden的1%)

使得线程可以不加锁地在多线程的情况下创建对象

快速分配策略:当TLAB空间不足时,重新创建TLAB.多线程同时分配内存时,使用TLAB可以避免线程安全问题,提升内存分配的吞吐量

所有OpenJDK衍生出来的JVM都提供了TLAB的设计



- 尽管不是所有的对象实例都能够在TLAB中成功分配内存,但JVM确实是将TLAB作为内存分配的首选
- 在程序中,开发人员可以通过选项"-XX:UseTLAB"设置是否开启TLAB空间。
- 一旦对象在TLAB空间分配内存失败时, JVM就会尝试着通过使用加锁机制确保数据操作的原子性,从而直接在Eden空间中分配内存













### 主/工作内存



![img](image.assets/20170328111450634.jpg)

内存模型分主内存和工作内存

* 主内存存储所有变量，共享

* 工作内存线程私有，保存变量值的副本，**线程对变量的操作在工作内存中完成，结束后放回主内存**

**主内存可粗略认为是堆，工作内存认为是栈**



CPU从内存取数据到寄存器，然后进行处理，但内存处理速度远低于CPU，导致CPU在处理指令时花费很多时间在等待内存做准备工作，于是在寄存器和主内存间添加了**CPU缓存**，CPU缓存小，但访问速度比主内存快



每个线程都需要从主内存中读取操作，所有的变量存储在主内存中，每个线程从**主内存**中获得变量的值

然后从图中可以看到每个线程获得数据之后会放入自己的**工作内存**，这个就是java内存模型的规定之二，保证每个线程操作的都是从主内存拷贝的**副本**，也就是说线程不能直接写主内存的变量，需要把主内存的变量值读取之后放入自己的工作内存中的变量副本中，然后操作这个副本。

线程与线程之间无法直接访问对方工作内存中的变量



![](image.assets/20170328142908765.png)



基本执行步骤：

1. lock（锁定）：线程读主内存时,把变量锁定
2. unlock（解锁）：读完后解锁，别的线程就可以进入操作
3. read（读取）：把变量值放入工作内存
4. load（加载）：把read操作得到的值放入工作内存变量副本中
5. use（使用）：把工作内存中的一个变量值传递给执行引擎
6. assign（赋值）：执行引擎赋值给工作内存的变量
7. store（存储）：把工作内存中的变量的值传到主内存
8. write（写入）：把store操作的值写入主内存变量





### 方法/静态区

**-XX:MaxPermSize**



* 只有1个,共享
* 存储==已被虚拟机加载的class信息(编译后的代码),类型信息,static变量,常量，编译期生成的常量==等**唯一的元素**
* ==永久代,GC主要进行常量池回收，类型卸载==
* 方法表
  * 实现动态调用的核心,存放在方法区中的类型信息中
  * 方法区的类型信息指向方法表，方法表指向具体方法,这些方法中包括从父类继承/自身重写





#### 运行时常量池

**-XX:PermSize和-XX:MaxPermSize**



- **常量池** 是Class文件的一部分,可以理解为Class文件中的资源仓库，它是Class文件结构中与其他项目资源关联最多的数据类型,**数据在编译期被确定**
  - **字符串常量池**：编译期,类中产生的字符串类型数据
  - **运行时常量池**：虚拟机加载Class后把常量池中的数据放入运行时常量池
    - 包含==基本类型和对象型的常量值==
    - ==小于127的Byte,Short,Integer,Long,Character== **不包括浮点数**
    - **编译期生成的常量**,如string的intern()



* 2类常量:
  * 字面量：字符串、声明为final的常量值等
  * 符号引用：类和接口的完全限定名、字段的名称和描述符、方法的名称和描述符













==方法区中的类型信息跟在堆中的class对象不同==

* 方法区中class类型信息只有唯一的实例（各个线程共享)

* 堆中可以有多个该class对象

* ==通过堆中的class对象访问到方法区中类信息(反射)==

![](image.assets/image-20201024224300360.png)





### 直接内存

**可通过-XX:MaxDirectMemorySize指定，如果不指定，则默认与Java堆的最大值（-Xmx指定）一样**。

- **直接内存（Direct Memory）并不是虚拟机运行时数据区的一部分，也不是Java虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用，而且也可能导致OutOfMemoryError异常出现**。





### 程序计数器



* **线程私有**
  * 字节码解释器 通过改变计数器的值来确定下一条指令，从而实现代码的流程控制，如：顺序执行、选择、循环、异常处理
* **为了线程切换能恢复到正确的位置，每条线程都需要一个独立的程序计数器**
  * 执行的是 native的话，程序计数器记录的是 undefined 地址，只有执行的是 Java 代码时程序计数器记录的才是下一条指令的地址
* **不会OutofMemoryError**





## 代码的3个阶段



* .java	经过javac编译->	.class

* .class	经过ClassLoader类加载器->	class类对象
  * class对象包含字段,构造方法,成员方法信息

* class对象	创建对象->	对象









## GC



传统的 C/C++语言，程序员负责回收已经分配内存

* 显式回收垃圾回收的缺点：

1）程序忘记回收，从而导致内存泄露，降低系统性能。

2）程序错误回收程序核心类库的内存，导致系统崩溃。

 

* Java由JRE在后台自动回收不再使用的内存

1）可以提高编程效率,保护程序的完整性。

3）影响性能,==Jvm必须跟踪程序中有用的对象==，确定哪些是无用的

 

* 回收机制

  * ==只回收堆内存里的对象==,不回收栈/物理连接
  * 无法控制回收执行时间,可以通过 System.gc()或者 Runtime.getRuntime().gc()来请求回收
  * ==将对象的引用变量设置为 null，暗示可以回收==
  * 回收任何对象之前，总会先调用它的 finalize 方法,但==不要主动调用finalize== ，应该交给垃圾回收机制调用





### 4种引用



* 强引用
  * ==不会被GC回收==，并且在java.lang.ref里也没有对应类型
  * Object obj = new Object()这里的obj便是强引用

* 软引用(SoftReference)
  * ==内存不足才回收==
  * 实现内存敏感的高速缓存,和引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收，Jvm就会把这个软引用加入到与之关联的引用队列中

* 弱引用（WeakReference）
  * ==一旦发现弱引用的对象，就回收==。由于GC是优先级很低的线程， 不一定会很快发现弱引用的对象
  * 可以和引用队列联合使用，如果弱引用所引用的对象被垃圾回收，Java虚拟机就会把这个弱引用加入到与之关联的引用队列中
  * **常用于Map数据结构中，占用内存空间较大的对象**

* 虚引用（PhantomReference）
  * 主要用来跟踪对象被垃圾回收器回收的活动
  * **必须和引用队列联合使用**
  * ==回收时发现虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中==
  * 程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收。如果已被加入，就可以在所引用的对象的内存被回收之前采取必要的行动
  * 由于Object.finalize()方法的不安全/低效，常用虚引用完成**对象回收前的资源释放工作****







## 创建/分配/访问



* 创建

遇到new指令时，首先检查这个指令的参数是否能在常量池中定位到一个类的符号引用，并且检查这个符号引用代表的类是否已被加载、连接和初始化过。

如果没有，那必须先执行相应的类的加载过程。

 

* 内存分配	==对象内存的大小在类加载完成后已经确定==，分配空间等同于从堆中划分确定大小的内存

* 两种分配方式
  * 指针碰撞：所有用过的内存在一边，空闲内存在另一边，中间放着一个指针作为分界点的指示器，

把指针往空闲内存那边挪一段与对象大小相等的距离。再使用Serial，ParNew等收集器，

（也就是用复制算法，标记-整理算法的收集器），分配算法通常采用指针碰撞。

* ​	空闲列表：虚拟机维护一个列表，记录哪些内存是可用的，分配的时候从列表中找到一块足够大的空间划分给对象，并更新列表。

使用CMS这种基于标记-清除算法的收集器，通常用空闲列表

 

　　对象创建在虚拟机中时非常频繁的行为，即使是仅仅修改一个指针指向的位置，在并发情况下也并不是线程安全的，可能出现正在给对象A分配内存，指针还没来得及修改，对象B又同时使用了原来的指针来分配内存的情况。

同步

　　虚拟机采用CAS配上失败重试的方式保证更新操作的原子性

本地线程分配缓冲（Thread Local Allocation Buffer, TLAB）

　　把内存分配的动作按照线程划分为在不同的空间之中进行，即每个线程在Java堆中预先分配一小块内存（TLAB）。

　　哪个线程要分配内存，就在哪个线程的TLAB上分配。只有TLAB用完并分配新的TLAB时，才需要同步锁定。

 

　　内存分配完之后，虚拟机要将分配到的内存空间都初始化为零值（不包括对象头），保证了对象的实例字段在Java代码中可以不赋初始值就直接使用。

 

**对象的内存布局：**

　　对象在内存中可分为3个部分，对象头，实例数据，对齐填充。

　　对象头的第一部分用于存储对象自身的**运行时数据**，如对象的哈希码，GC分代年龄，锁状态标志，线程持有的锁等。

　　另一部分是类型指针，即对象指向它的类元数据的指针，通过这个来确定这个对象是哪个类的实例。

　　实例数据是对象真正存储的有效信息。

 

**对象的访问定位：**

　　程序要通过栈上的reference数据来操作堆上的具体对象。对象的访问方式有**使用句柄**和**直接指针**。

　　**使用句柄**：java堆会划分一块内存作为句柄池，reference中存的是对象的**句柄地址**，而句柄中包含了**对象的实例数据的地址和类型数据的地址（在方法区）**。

优点：对象被移动，reference不用修改，只会改变句柄中保存的地址。

　　**使用直接指针**：reference中存的是对象的地址，对象中分一小块内存保存类型数据的地址。优点：速度快。













## 类加载机制



类加载机制	验证， 准备， 解析

准备	**为类的类变量（非对象变量）分配内存,初始值，准备类中每个字段、方法和实现接口所需的数据结构**





![](image.assets/1541314-20191003104706537-1884792960.png)



```java
public class Singleton {
    //1.静态变量    调用了非静态的构造器  将优先加载非静态,跳过静态
    //4.构造方法结束,赋值
    private static Singleton instance = new Singleton();

    //3.构造方法    此时x=y=1
    public Singleton() {   x++;   y++; }

    //5.静态变量赋值  此时x=1,y=0
    private static int x,y = 0;

    //2.非静态变量
    private int z = 1;

    public static Singleton getInstance() {   return instance; }

    public static void main(String[] args) {
        //6.main方法体     结果x=1,y=0
        getInstance();   }}
```



```java
public class Singleton2 {

    //3.构造方法    此时x=y=1
    public Singleton2() {     x++; y++;    }

    //1.静态变量赋值  此时x=y=0
    private static int x = 0, y;
    //2.静态变量调用构造方法
    //4.构造器返回值赋值
    private static Singleton2 instance = new Singleton2();

    public static Singleton2 getInstance() {   return instance;    }

    public static void main(String[] args) {
        //5.main方法体       结果x=y=1
        getInstance();   }}
```



![img](image.assets/20160812142709857)

* 类引用调用的大致过程
  * 编译器将源代码编译成class文件，根据静态类型将调用的符号引用写到class文件
  * 在执行时，JVM根据class文件找到调用方法的符号引用，在方法表中找到偏移量
  * 然后根据this指针确定对象的实际类型，==使用实际类型的方法表(多态的实现)==，根据偏移量在实际类型的方法表中找到方法则直接调用，否则，按照继承关系从下往上搜索



实例方法和变量的内存是在运行时分配的，所以地址(内存的偏移)无法固定。静态方法无法调用实例方法和变量,实例方法可以调用静态方法和变量。



类方法执行时,对象还未创建,==类方法不能被实例调用==

在类方法中调用实例方法,将优先执行完所有实例方法,==类方法可以调用实例方法==



### 静态代码块

```java
 static{ }
```

* 属于类
* 类被加载的时运行，只运行一次，优先于各种代码块以及构造函数
  * 静态代码块**主动运行**,所以不能在方法体中
  * 静态方法是被动运行,通过类名或对象名访问
  * 普通方法是实例化后运行,通过对象访问

* 一般用于项目启动加载配置文件



### 构造代码块

```
{}
```

* 属于类

* 创建对象时调用，**每次创建对象都会调用**，优先于构造函数执行

* 不实例化对象，构造代码块不会执行

* 构造方法被重载,不能事先确定到底执行哪个,构造代码块却一定被执行

  

### 构造函数

* 属于类
* 命名为类名,不带返回值。普通函数可以和构造函数同名，但有返回值

* 主要用于在类的对象创建时定义初始化的状态。不能被void修饰,以区分其他有返回值的方法

* 不能被直接调用，必须通过new

* 默认先调用父类的无参构造



### 普通代码块

-    构造代码块是在**类中**定义的，
-    普通代码块是在**方法体中**定义的







* 静态和动态分开处理
  * 静态加载： 静态变量，静态代码块
  * 动态加载： 
  * **静态/实例方法在调用的才会执行**
  * 当静态加载中遇到需要加载动态的情况：**先加载动态再加载静态**（因为非静态可以访问静态，而静态不能访问非静态）
  * 静态变量声明必须放在使用前面

* main是否第一句先执行
  * main也是静态方法，首先加载main所在的类,不执行main，优先加载类中其他静态部分

* 父类、子类加载顺序

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

3、clone()

4、反序列化，调用 java.io.ObjectInputStream 对象的readObject()方法。

(1)和(2)显式调用构造

(3)(4)不调用构造







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



## == hash equals



(1)对象相同（equals 方法返回 true），则hashCode相同

(2)hashCode 相同，并不一定相同



**==是关系运算符，equals()是方法**

* ==    
  * 基本类型，比较值
  * 引用类型，比较地址
  * ==不能比较没有父子关系的两个对象==

* equals() 
  * 重写equals()，比较内容
  * Object == equals 都比较地址

* hashCode()
  * 如果equals()相等，hashCode相等
  * 从某一应用程序的一次执行到同一应用程序的另一次执行，hashCode()无需保持一致。
  * 以下情况不是必需的
    * equals()不相等，hashCode()必定不相等,但hashCode()必定不相等可以提高哈希表的性能
    * equals()被重写时，有必要重写 hashCode 方法，以维护 hashCode 方法的常规协定





## 对象克隆



* ==实现 Cloneable 接口==并重写 Object 类中的 clone()方法；

* 实现 Serializable 接口，通过对象的==序列化和反序列化==，可以实现深度克隆,更重要的是支持==泛型限定==



### 深/浅克隆

浅度拷贝即直接赋值，拷贝的只是原始对象的引用地址，在堆中仍然共用一块内存。而深度拷贝为新对象在堆中重新分配一块内存，所以对新对象的操作不会影响原始对象。

要将可变对象和不可变对象相互转换，或者需要==操作新对象的时候不影响原始对象，用深度拷贝== ==copy-on-write==原则就是利用深度拷贝来实现的

 

### hutool克隆



CopyOptions定义了克隆规则		setIgnoreNullValue忽略null

```
BeanUtil.copyProperties(来源,目标, CopyOptions.create().setIgnoreNullValue(true).setIgnoreError(true));
```





# 拆箱装箱 1.5+



装箱：基本数据类型->包装器类型      valueOf方法

拆箱：包装器类型->基本数据类型      xxxValue方法



==基本数据类型不是面向对象（没有属性、方法）==，实际使用时存在很多的不便（比如集合的元素只能是Object）。所以需要包装类



## 8种基本类型



* **Java基本类型的大小不随机器结构的变化而变化**。大小的不可更改使得Java移植能力强

* ==基本数据类型只能值传递，封装类按照引用传递==

* 基本类型在栈；对象在堆，对象的引用在栈
  * 基本类型在栈，效率高，但可能内存泄漏

* 基本类型在声明时自动分配空间/赋值，==引用数据类型声明时只在分配了引用空间==，必须通过实例化在堆内存中开辟空间后才赋值

* ==数组也是引用对象==，**将一个数组赋值给另一个数组只是在栈中添加了一个引用变量**，而堆内存中数据并没有增加只是多了一个引用指向改数据空间。所以某个数组所做的修改在另一个数组中也可以看到

| 基本类型 | 大小(字节)            | 默认值         | 封装类    |                           |
| -------- | --------------------- | -------------- | --------- | ------------------------- |
| byte     | 1                     | (byte)0        | Byte      | -128~127                  |
| short    | 2                     | (short)0       | Short     | -32768~32767              |
| int      | 4                     | 0              | Integer   | -2^31^~2^31^-1            |
| long     | 8                     | 0L             | Long      |                           |
| float    | 1符号+8指数+23尾数=4  | 0.0f           | Float     | -3.4e^45^~3.4e^38^        |
| double   | 1符号+11指数+52尾数=8 | 0.0d           | Double    | -1.79e^308^ ~ +1.79e^308^ |
| boolean  | -                     | false          | Boolean   |                           |
| char     | 2                     | \u0000  (null) | Character |                           |



```java
short s1 = 1; 
s1 = s1 + 1;		//错误,s1 + 1为int,需要强转
s1 += 1;				//正确,被优化为s1 = (short)(s1 + 1)

float f=3.4			//错误,3.4 是双精度数，将双精度型（double）赋值给浮点型（float）
float f =(float)3.4	或 float f =3.4F		//正确
```







## Character



```java
isLetter()		是否为字母
isDigit()		是否为数字
isWhitespace()	是否为空白字符
  
isUpperCase()	是否是大写字母
isLowerCase()	是否是小写字母
toUpperCase()	转化为大写
toLowerCase()
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



## try/catch



* 不管有没有异常，finally都会执行
* catch中return，finally依然执行
* return的是表达式,finally无法改变返回值;return的是引用类型，finally能改变返回值
* finally代码中最好不要包含return，程序会提前退出，也就是说返回的值不是try或catch中的值





## 内存溢出/泄漏



* 内存泄露(OOM)：对象不被GC回收，始终占用内存。==分配的对象可达但已无用==
  * 内存泄露是内存溢出的一种诱因，不是唯一因素

* 内存溢出：**无法满足内存分配需求**
  * 栈溢出(SOF)：递归太深而发生堆栈溢出
  * **静态的集合类过多**
  * 数据库、网络、输入输出流，没有显式关闭
    * GC只负责回收，无法判断对象是否正在使用资源
  * ==单例对象中拥有另一个对象的引用的话，这个被引用的对象就不能被及时回收==
    * 解决办法是单例对象中持有的其他对象使用弱引用，其占用的内存会被回收



除了程序计数器外，虚拟机内存的其他几个运行时区域都有发生OutOfMemoryError(OOM)异常的可能



* Heap堆溢出：
  * 通过内存映像分析工具对dump出来的堆转存快照进行分析，重点是确认内存中的对象是否是必要的，**先分清是因为内存泄漏(Memory Leak)还是内存溢出(Memory Overflow)**
    * **内存泄漏,对象不需要了，内存和引用没被回收**，通过工具查看泄漏对象到GCRoots的引用链。找到泄漏对象是通过怎样的路径与GC Roots关联并导致垃圾收集器无法自动回收
    * 内存溢出，检查虚拟机的参数(-Xmx与-Xms)的设置是否适当

* 栈溢出
  * 线程请求的栈深度大于虚拟机所允许的最大深度，将抛出StackOverflowError异常。
  * 虚拟机在扩展栈时无法申请到足够的内存空间，则抛出OutOfMemoryError异常
  * 当栈的大小越大可分配的线程数就越少
  * 递归调用，大量循环或死循环，全局变量过多，数组、List、map数据过大
  * 栈一般默认为1-2m

* 常量池溢出
  * 异常信息：OutOfMemoryError:PermGenspace
  * 如果要向运行时常量池中添加内容，最简单的做法就是使用String.intern()这个Native方法。该方法的作用是：如果池中已经包含一个等于此String的字符串，则返回代表池中这个字符串的String对象；否则，将此String对象包含的字符串添加到常量池中，并且返回此String对象的引用
  * 常量池分配在方法区内，可以通过-XX:PermSize和-XX:MaxPermSize限制方法区的大小，从而间接限制其中常量池的容量

* 方法区溢出
  * 异常信息：OutOfMemoryError:PermGenspace
  * 存放Class的相关信息，如类名、访问修饰符、常量池、字段描述、方法描述等。
  * 类如果要被垃圾收集器回收，条件很苛刻。在经常动态生成大量Class的应用中，要特别注意这点



## 避免泄露/溢出



1、尽早释放无用对象的引用

2、避免用String

3、尽量少用静态变量，因为静态变量存放在方法区，基本不参与垃圾回收

4、避免循环创建对象

5、大概计算一下数据量的最大值，设定所需内存空间值。









# 修饰符/关键字



==重写的访问修饰符只能比父类大==

![img](image.assets/wps3-1603184298721.jpg) 



## 父类成员在子类的访问权限

Public继承方式    不改变父类的访问权限

protected          private不变,其余都变为protected

private            都改成private



## 子类成员在外部的访问权限

**父类的private     只有父类能访问**

private方式继承的非private成员    只有子类的成员函数能访问,子类的子类/外部不能访问

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
@Documented
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.SOURCE)
public @interface Native {}
```

native是**java调用非java代码的接口**

定义Native Method时,**并不需要提供实现**,其实现体将由**非java语言在外面实现****



```java
 //Native Method的声明更像是描述非java代码在java中的大致模样
 public class IHaveNatives
   {  native public void Native1( int x ) ;
      native static public long Native2() ;
      native synchronized private float Native3( Object o ) ;
      native void Native4( int[] ary ) throws Exception ; }
```



==native可以与所有修饰符连用，除abstract==,与abstract的无实现相违背



==native method可以返回任何java类型，包括非基本类型==，而且同样可以进行异常控制。这些方法的实现体可以制一个异常并且将其抛出。当native method接收到非基本类型,如Object时，可以访问非基本类型的内部，**但这将使native method依赖于所访问的java类的实现**。可以在一个native method的本地实现中访问所有的java特性，但会导致依赖于所访问的java特性的实现，这远不如使用java特性方便



native method不会对其他类调用这些本地方法产生任何影响，调用者甚至不知道它所调用的是一个本地方法。JVM将控制调用本地方法的所有细节。
如果含有本地方法的类被继承，**子类会继承这个本地方法并且可以用java重写**，本地方法被fianl标识，继承后不能被重写。
本地方法扩充了jvm,在sun的java的并发实现中，许多与操作系统的接触点都用到了本地方法，使java能够超越java运行时的界限。



==JVM怎样使Native Method跑起来==
当类第一次被使用时，这个类的字节码会被加载到内存。在这个被加载的字节码入口,维持着该类所有方法描述符的list，这些方法描述符包含：方法代码存于何处，有哪些参数，修饰符等等。
native修饰符将有一个指向该方法的实现的指针。这些实现在一些DLL文件内，它们会被操作系统加载到java程序的地址空间。当带有本地方法的类被加载时，其相关的DLL并未被加载，因此指向方法实现的指针并不会被设置。**当本地方法被调用之前，这些DLL才会被加载**，这是通过调用java.system.loadLibrary()实现的。





## transient瞬态



为了避免被序列号,可以声明为**静态或瞬态**

生命周期仅存于调用者的内存中,不会被持久化

只能修饰非本地变量，不能修饰方法和类

 

一旦变量被transient修饰，变量将不再是对象持久化的一部分，该变量内容在序列化后无法获得访问。

**实现Externalizable接口 ,则无视transient**

















## sql.Date和util.Date 

1） java.sql.Date 是 java.util.Date 的子类，是一个包装了毫秒值的瘦包装器，允许JDBC将毫秒值标识为 SQL DATE 值

2）java.sql.Date是针对 SQL 语句使用的，只包含日期而没有时间部分。

以下操作中容易出现不易被发现的 BUG：获得一个 JAVA 里的日期对象。 从数据库里读取日期 试图比较两个日期对象是否相等。如果毫秒部分丢失，本来认为相等的两个日期对象用 Equals 方法可能返回 false。sql.Timestamp比util.Date类精确度要高





## Switch



1.5前，只能是byte，short，char，int类型(或其包装类)的常量表达式

1.5后，引入枚举enum

1.7后，exper还可以是String类型。

**long在所有版本都不行**



1.7通过hashCode(),将string转换为int,switch(String)只是语法糖,在相应位置插入了强制转换代码，底层并没有修改

==Switch中的String必须先判空==

```java
//在编译后的class中
String string = "Hello";
            String s;
            switch ((s = string).hashCode()){
            case 2301506: 
                //用equals进行安全检查（避免hash相同值不同）
                if (!s.equals("Java"))
```



每个case要么通过continue/break/return等来终止，要么注释说明程序将继续执行到哪一个case为止

必须包含一个default语句并且放在最后







# 内部类





## 匿名内部类



* 没有名字,==不能继承,但可以作为接口,由另一个内部类实现==
  * 在匿名内部类里创建新方法没有太大意义，但它可以通过覆盖父类的方法达到神奇效果

* ==无法向下强转==，持有对一个匿名内部类对象引用的变量类型一定是它的直接或间接父类类型。

* 没有构造函数,依赖父类的构造函数来实例化,把创建对象的任务交给了父类







## 静态内部类和内部类区别



静态内部类不需要有指向外部类的引用。但非静态内部类需要持有对外部类的引用。

静态内部类可以有静态成员(方法，属性)，而非静态内部类则不能有静态成员(方法，属性)。

静态内部类只能访问外部类的静态成员。非静态内部类能够访问外部类的静态和非静态成员。

实例化方式不同：

1) 静态内部类：不依赖于外部类的实例，直接实例化内部类对象

2) 非静态内部类：通过外部类的对象实例生成内部类对象







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





















