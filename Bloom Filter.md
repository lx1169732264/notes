Bloom Filter可以看作由 ==二进制数组 + n个哈希函数== 组成的数据结构

优点	占用空间更少并且效率更高

缺点	返回的结果是概率性的，而不是非常准确的。添加到集合中的元素越多，误报的可能性就越大。并且存放在布隆过滤器的数据不容易删除



**插入元素时：**

根据n个哈希函数计算出n个哈希值，在位数组中把对应的n个下标的值置1

**判断是否存在时：**

再次进行相同的哈希计算出n个哈希值

判断位数组中的每个哈希值对应的 都为1 ? 存在 : 不存在



由于hash函数的不确定性,使得==Bloom Filter认为存在时,有极小概率是不存在的,但认为不存在时,则一定不存在==

例如不同字符串的hash值可能相同,在两个相同hash的字符串在进行插入时,会被误判.**可以适当增加位数组大小或者调整hash函数**





### Java手动实现布隆过滤器

需要：

1. 一个合适大小的位数组保存数据
2. 几个不同的哈希函数
3. 添加元素到位数组（布隆过滤器）的方法实现
4. 判断给定元素是否存在于位数组（布隆过滤器）的方法实现。

下面给出一个我觉得写的还算不错的代码（参考网上已有代码改进得到，对于所有类型对象皆适用）：

```java
import java.util.BitSet;

public class MyBloomFilter {

  /**
     * 位数组的大小
     */
  private static final int DEFAULT_SIZE = 2 << 24;
  /**
     * 通过这个数组可以创建 6 个不同的哈希函数
     */
  private static final int[] SEEDS = new int[]{3, 13, 46, 71, 91, 134};

  /**
     * 位数组。数组中的元素只能是 0 或者 1
     */
  private BitSet bits = new BitSet(DEFAULT_SIZE);

  /**
     * 存放包含 hash 函数的类的数组
     */
  private SimpleHash[] func = new SimpleHash[SEEDS.length];

  /**
     * 初始化多个包含 hash 函数的类的数组，每个类中的 hash 函数都不一样
     */
  public MyBloomFilter() {
    // 初始化多个不同的 Hash 函数
    for (int i = 0; i < SEEDS.length; i++) {
      func[i] = new SimpleHash(DEFAULT_SIZE, SEEDS[i]);
    }
  }

  /**
     * 添加元素到位数组
     */
  public void add(Object value) {
    for (SimpleHash f : func) {
      bits.set(f.hash(value), true);
    }
  }

  /**
     * 判断指定元素是否存在于位数组
     */
  public boolean contains(Object value) {
    boolean ret = true;
    for (SimpleHash f : func) {
      ret = ret && bits.get(f.hash(value));
    }
    return ret;
  }

  /**
     * 静态内部类。用于 hash 操作！
     */
  public static class SimpleHash {

    private int cap;
    private int seed;

    public SimpleHash(int cap, int seed) {
      this.cap = cap;
      this.seed = seed;
    }

    /**
         * 计算 hash 值
         */
    public int hash(Object value) {
      int h;
      return (value == null) ? 0 : Math.abs(seed * (cap - 1) & ((h = value.hashCode()) ^ (h >>> 16)));
    }

  }
}
```

测试：

```java
        String value1 = "https://javaguide.cn/";
        String value2 = "https://github.com/Snailclimb";
        MyBloomFilter filter = new MyBloomFilter();
        System.out.println(filter.contains(value1));
        System.out.println(filter.contains(value2));
        filter.add(value1);
        filter.add(value2);
        System.out.println(filter.contains(value1));
        System.out.println(filter.contains(value2));
```

Output:

```
false
false
true
true
```

测试：

```java
        Integer value1 = 13423;
        Integer value2 = 22131;
        MyBloomFilter filter = new MyBloomFilter();
        System.out.println(filter.contains(value1));
        System.out.println(filter.contains(value2));
        filter.add(value1);
        filter.add(value2);
        System.out.println(filter.contains(value1));
        System.out.println(filter.contains(value2));
```

Output:

```java
false
false
true
true
```

### Guava自带的布隆过滤器

自己实现的目的主要是为了让自己搞懂布隆过滤器的原理，Guava 中布隆过滤器的实现算是比较权威的，所以实际项目中我们不需要手动实现一个布隆过滤器。

首先我们需要在项目中引入 Guava 的依赖：

```java
        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>28.0-jre</version>
        </dependency>
```

实际使用如下：

我们创建了一个最多存放 最多 1500个整数的布隆过滤器，并且我们可以容忍误判的概率为百分之（0.01）

```java
        // 创建布隆过滤器对象
        BloomFilter<Integer> filter = BloomFilter.create(
                Funnels.integerFunnel(),
                1500,
                0.01);
        // 判断指定元素是否存在
        System.out.println(filter.mightContain(1));
        System.out.println(filter.mightContain(2));
        // 将元素添加进布隆过滤器
        filter.put(1);
        filter.put(2);
        System.out.println(filter.mightContain(1));
        System.out.println(filter.mightContain(2));
```

在我们的示例中，当`mightContain（）` 方法返回*true*时，我们可以99％确定该元素在过滤器中，当过滤器返回*false*时，我们可以100％确定该元素不存在于过滤器中。

**Guava 提供的布隆过滤器重大的缺陷就是只能单机使用（另外，容量扩展也不容易），而现在互联网一般都是分布式的场景。为了解决这个问题，我们就需要用到 Redis 中的布隆过滤器了。**

### Redis布隆过滤器

#### 6.1介绍

Redis v4.0 之后有了 Module（模块/插件） 功能，Redis Modules 让 Redis 可以使用外部模块扩展其功能 。布隆过滤器就是其中的 Module

另外，官网推荐了一个 RedisBloom  作为 Redis 布隆过滤器的 Module,地址：https://github.com/RedisBloom/RedisBloom. 其他还有：

- redis-lua-scaling-bloom-filter （lua 脚本实现）：https://github.com/erikdubbelboer/redis-lua-scaling-bloom-filter
- pyreBloom（Python中的快速Redis 布隆过滤器） ：https://github.com/seomoz/pyreBloom



#### 6.2使用Docker安装

如果我们需要体验 Redis 中的布隆过滤器非常简单，通过 Docker  就可以了！我们直接在 Google 搜索**docker redis bloomfilter**https://hub.docker.com/r/redislabs/rebloom/ 

**具体操作如下：**

```
➜  ~ docker run -p 6379:6379 --name redis-redisbloom redislabs/rebloom:latest
➜  ~ docker exec -it redis-redisbloom bash
root@21396d02c252:/data# redis-cli
127.0.0.1:6379> 
```

#### 6.3常用命令一览

>  注意：   key:布隆过滤器的名称，item : 添加的元素。

1. **`BF.ADD `**：将元素添加到布隆过滤器中，如果该过滤器尚不存在，则创建该过滤器。格式：`BF.ADD {key} {item}`。
2. **`BF.MADD `** : 将一个或多个元素添加到“布隆过滤器”中，并创建一个尚不存在的过滤器。该命令的操作方式`BF.ADD`与之相同，只不过它允许多个输入并返回多个值。格式：`BF.MADD {key} {item} [item ...]` 。
3. **`BF.EXISTS` ** : 确定元素是否在布隆过滤器中存在。格式：`BF.EXISTS {key} {item}`。
4. **`BF.MEXISTS`** ： 确定一个或者多个元素是否在布隆过滤器中存在格式：`BF.MEXISTS {key} {item} [item ...]`。

另外，`BF.RESERVE` 命令需要单独介绍一下：

这个命令的格式如下：

`BF.RESERVE {key} {error_rate} {capacity} [EXPANSION expansion] `。

下面简单介绍一下每个参数的具体含义：

1. key：布隆过滤器的名称
2. error_rate :误报的期望概率。这应该是介于0到1之间的十进制值。例如，对于期望的误报率0.1％（1000中为1），error_rate应该设置为0.001。该数字越接近零，则每个项目的内存消耗越大，并且每个操作的CPU使用率越高。
3. capacity:  过滤器的容量。当实际存储的元素个数超过这个值之后，性能将开始下降。实际的降级将取决于超出限制的程度。随着过滤器元素数量呈指数增长，性能将线性下降。

可选参数：

- expansion：如果创建了一个新的子过滤器，则其大小将是当前过滤器的大小乘以`expansion`。默认扩展值为2。这意味着每个后续子过滤器将是前一个子过滤器的两倍

#### 6.4实际使用

```shell
127.0.0.1:6379> BF.ADD myFilter java
(integer) 1
127.0.0.1:6379> BF.ADD myFilter javaguide
(integer) 1
127.0.0.1:6379> BF.EXISTS myFilter java
(integer) 1
127.0.0.1:6379> BF.EXISTS myFilter javaguide
(integer) 1
127.0.0.1:6379> BF.EXISTS myFilter github
(integer) 0
```

