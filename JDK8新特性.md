



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



## 函数式接口 

**只有一个抽象方法**

### @FunctionalInterface

只在编译期起作用，如@Override注解。编译期会强制检查该接口是否符合函数式接口的条件，不符合则会报错。**即使不使用，只要满足定义也是函数式接口**。

![image-20200826214004498](image.assets/image-20200826214004498.png)



## Supplier

java.util.function.Supplier<T> 接口仅包含一个无参的方法： T get() 。用来获取一个泛型参数指定类型的对象数据。由于这是一个函数式接口，这也就意味着对应的Lambda表达式需要“对外提供”一个符合泛型类型的对象数据。

```
public class Demo08Supplier {   private static String getString(Supplier<String> function) {     return function.get();   }     public static void main(String[] args) {     String msgA = "Hello";     String msgB = "World";     System.*out*.println(*getString*(() -> msgA + msgB));   } }
```

 

## 练习：求数组元素的最小值

```
public class Demo02Test {   //定一个方法,方法的参数传递Supplier,泛型使用Integer    public static int getMax(Supplier<Integer> sup) {     return sup.get();   }     public static void main(String[] args) {     int arr[] = {2, 3, 4, 52, 333, 23};     //调用getMax方法,参数传递Lambda      int maxNum = *getMax*(() -> {       //计算数组的最大值        int max = arr[0];       for (int i : arr) {         if (i > max) {           max = i;         }       }       return max;     });     System.*out*.println(maxNum);   } }
```

 

## Consumer接口

java.util.function.Consumer<T> 接口则正好与Supplier接口相反，它不是生产一个数据，而是消费一个数据， 

其数据类型由泛型决定。

**抽象方法：accept**，意为消费一个指定泛型的数据

```
public class Demo09Consumer {   private static void consumeString(Consumer<String> function) {     function.accept("Hello");   }     public static void main(String[] args) {     *consumeString*(s -> System.*out*.println(s));   } }
```

**默认方法：andThen**

如果一个方法的参数和返回值全都是 Consumer 类型，那么就可以实现效果：消费数据的时候，首先做一个操作，然后再做一个操作，实现组合。而这个方法就是 Consumer 接口中的default方法 andThen

要想实现组合，需要两个或多个Lambda表达式即可，而 andThen 的语义正是“一步接一步”操作。例如两个步骤组合的情况

```
public class Demo10ConsumerAndThen {   private static void consumeString(Consumer<String> one, Consumer<String> two) {     one.andThen(two).accept("Hello");   }     public static void main(String[] args) {     *consumeString*(s -> System.*out*.println(s.toUpperCase()), s -> System.*out*.println(s.toLowerCase()));   } }
```

 

## 练习：格式化打印信息

下面的字符串数组当中存有多条信息，请按照格式“ 姓名：XX。性别：XX。 ”的格式将信息打印出来。要求将打印姓名的动作作为第一个 Consumer 接口的Lambda实例，将打印性别的动作作为第二个 Consumer 接口的Lambda实例，将两个 Consumer 接口按照顺序“拼接”到一起。

```
public class DemoConsumer {   public static void main(String[] args) {     String[] array = {"迪丽热巴,女", "古力娜扎,女", "马尔扎哈,男"};     *printInfo*(s -> System.*out*.print("姓名：" + s.split(",")[0]), s ->         System.*out*.println("。性别：" + s.split(",")[1] + "。"), array);   }     private static void printInfo(Consumer<String> one, Consumer<String> two, String[] array) {     for (String info : array) {       one.andThen(two).accept(info); // 姓名：迪丽热巴。性别：女。      }   } }
```

## Predicate接口

有时候我们需要对某种类型的数据进行判断，从而得到一个boolean值结果。这时可以使用java.util.function.Predicate<T> 接口。

**抽象方法：test** 

Predicate 接口中包含一个抽象方法： boolean test(T t) 。用于条件判断的场景：

```
public class Demo15PredicateTest {   private static void method(Predicate<String> predicate) {     boolean veryLong = predicate.test("HelloWorld");     System.*out*.println("字符串很长吗：" + veryLong);   }     public static void main(String[] args) {     *method*(s -> s.length() > 5);   } }
```

条件判断的标准是传入lambda表达式逻辑

**默认方法：and** 

既然是条件判断，就会存在与、或、非三种常见的逻辑关系。其中将两个 Predicate 条件使用“与”逻辑连接起来实现“并且”的效果时，可以使用default方法 and

如果要判断一个字符串既要包含大写“H”，又要包含大写“W”

```
public class Demo16PredicateAnd {   private static void method(Predicate<String> one, Predicate<String> two) {     boolean isValid = one.and(two).test("Helloworld");     System.*out*.println("字符串符合要求吗：" + isValid);   }     public static void main(String[] args) {     *method*(s -> s.contains("H"), s -> s.contains("W"));   } }
```

**默认方****法：or** 

如果希望实现逻辑“字符串包含大写H或者包含大写W”，那么代码只需要将“and”修改为“or”名称即可，其他都不变：

**默认方法：negate** 

表示取反

```
public class Demo17PredicateNegate {   private static void method(Predicate<String> predicate) {     boolean veryLong = predicate.negate().test("HelloWorld");     System.*out*.println("字符串很长吗：" + veryLong);   }     public static void main(String[] args) {     *method*(s -> s.length() < 5);   } }
```

## 练习：集合信息筛选

数组当中有多条“姓名+性别”的信息如下，请通过 Predicate 接口的拼装将符合要求的字符串筛选到集合ArrayList 中，需要同时满足两个条件：

\1. 必须为女生；

\2. 姓名为4个字。

```
public class DemoPredicate {   public static void main(String[] args) {     String[] array = {"迪丽热巴,女", "古力娜扎,女", "马尔扎哈,男", "赵丽颖,女"};     List<String> list = *filter*(array, s -> "女".equals(s.split(",")[1]), s -> s.split(",")[0].length() == 4);     System.*out*.println(list);   }     private static List<String> filter(String[] array, Predicate<String> one, Predicate<String> two) {     List<String> list = new ArrayList<>();     for (String info : array) {       if (one.and(two).test(info)) {         list.add(info);       }     }     return list;   } }
```

## Function接口

java.util.function.Function<T,R> 接口用来根据一个类型的数据得到另一个类型的数据，前者称为前置条件，后者称为后置条件。 

**抽象方法：apply** 

Function 接口中最主要的抽象方法为： R apply(T t) ，根据类型T的参数获取类型R的结果。使用的场景例如：将 String 类型转换为 Integer 类型

```
public class Demo11FunctionApply {   private static void method(Function<String, Integer> function) {     int num = function.apply("10");     System.*out*.println(num + 20);   }     public static void main(String[] args) {     *method*(s -> Integer.*parseInt*(s));   } }
```

**默认方法：andThen**

```
public class Demo12FunctionAndThen {   private static void method(Function<String, Integer> one, Function<Integer, Integer> two) {     int num = one.andThen(two).apply("10");     System.*out*.println(num + 20);   }     public static void main(String[] args) {     *method*(str -> Integer.*parseInt*(str) + 10, i -> i *= 10);   } }
```

## 练习：自定义函数模型拼接

请使用 Function 进行函数模型的拼接，按照顺序需要执行的多个函数操作为：

String str = "赵丽颖,20";

\1. 将字符串截取数字年龄部分，得到字符串；

\2. 将上一步的字符串转换成为int类型的数字； 

\3. 将上一步的int数字累加100，得到结果int数字。 

```
public class DemoFunction {   public static void main(String[] args) {     String str = "赵丽颖,20";     int age = *getAgeNum*(str, s -> s.split(",")[1], s -> Integer.*parseInt*(s), n -> n += 100);     System.*out*.println(age);   }     private static int getAgeNum(String str, Function<String, String> one, Function<String, Integer> two, Function<Integer, Integer> three) {     return one.andThen(two).andThen(three).apply(str);   } }
```











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



## 使用方法

```
public void testCook() {
    methodCook(() ->  System.out.println("做饭了"));}

private void methodCook(Cook cook) {cook.makeFood();}
```



# 方法引用 ::



## 冗余的lambda

```
public void print() {
    print(e -> System.out.println(e));
}

private void print(Consumer<String> consumer) {
    consumer.accept("Hello");
}
```

这个lambda只是接收了accept方法的参数 ,并将它打印 ,而打印的方法有现成的System.out.println

那么可以直接去引用这个方法

```
public void print() {
    print(System.out::println);
}
```



如果Lambda要表达的函数方案已经存在于某个方法的实现中，那么则可以通过双冒号来引用该方法作为Lambda的替代者。 

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





















