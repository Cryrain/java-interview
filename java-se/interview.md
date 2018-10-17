## 数据库相关

[数据库](https://github.com/CyC2018/CS-Notes/blob/master/notes/%E6%95%B0%E6%8D%AE%E5%BA%93%E7%B3%BB%E7%BB%9F%E5%8E%9F%E7%90%86.md)

* #### 事务

  * ACID原则
    * Atomicity 原子性，事务被视为不可分割的最小单元，事务的所有操作要么全部提交成功，要么全部失败回滚。
    * Consistency 一致性，数据库在事务执行前后都保持一致性状态。在一致性状态下，所有事务对一个数据的读取结果都是相同的。
    * Isolation 隔离性，一个事务所做的修改在最终提交以前，对其它事务是不可见的。
    * Durability 持久性，一旦事务提交，则其所做的修改将会永远保存到数据库中。即使系统发生崩溃，事务执行的结果也不能丢失。

* #### 并发一致性问题

  * 丢失修改 :T1 和 T2 两个事务都对一个数据进行修改，T1 先修改，T2 随后修改，T2 的修改覆盖了 T1 的修改。
  * 读脏数据:T1 修改一个数据，T2 随后读取这个数据。如果 T1 撤销了这次修改，那么 T2 读取的数据是脏数据。
  * 不可重复读:T2 读取一个数据，T1 对该数据做了修改。如果 T2 再次读取这个数据，此时读取的结果和第一次读取的结果不同。
  * 幻影读:T1 读取某个范围的数据，T2 在这个范围内插入新的数据，T1 再次读取这个范围的数据，此时读取的结果和和第一次读取的结果不同。

* #### 隔离级别

  * **未提交读（READ UNCOMMITTED）** 

    > 事务中的修改，即使没有提交，对其它事务也是可见的。

  * **提交读（READ COMMITTED）**

    > 一个事务只能读取已经提交的事务所做的修改。换句话说，一个事务所做的修改在提交之前对其它事务是不可见的。

  * **可重复读（REPEATABLE READ）**

    > 保证在同一个事务中多次读取同样数据的结果是一样的。

  * **可串行化（SERIALIZABLE）**

    > 强制事务串行执行。

  * | 隔离级别 | 脏读 | 不可重复读 | 幻影读 | 加锁读 |
    | -------- | ---- | ---------- | ------ | ------ |
    | 未提交读 | √    | √          | √      | ×      |
    | 提交读   | ×    | √          | √      | ×      |
    | 可重复读 | ×    | ×          | √      | ×      |
    | 可串行化 | ×    | ×          | ×      | √      |

## MySql

[Mysql](https://github.com/CyC2018/CS-Notes/blob/master/notes/MySQL.md)

* #### MySql索引

  * B+Tree 索引

    > 1. 数据结构
    >
    >    B Tree 指的是 Balance Tree，也就是平衡树。平衡树是一颗查找树，并且所有叶子节点位于同一层。
    >
    >    B+ Tree 是基于 B Tree 和叶子节点顺序访问指针进行实现，它具有 B Tree 的平衡性，并且通过顺序访问指针来提高区间查询的性能。
    >
    > 2. 是大多数 MySQL 存储引擎的默认索引类型

  * 索引优化

    > 1. 独立的列
    >
    >    在进行查询时，索引列不能是表达式的一部分，也不能是函数的参数，否则无法使用索引。
    >
    >    例如下面的查询不能使用 actor_id 列的索引：
    >
    >    SELECT actor_id FROM sakila.actor WHERE actor_id + 1 = 5;
    >
    > 2. 索引列的顺序
    >
    >    让选择性最强的索引列放在前面。
    >
    >    索引的选择性是指：不重复的索引值和记录总数的比值。最大值为 1，此时每个记录都有唯一的索引与其对应。选择性越高，查询效率也越高。
    >
    >    例如下面显示的结果中 customer_id 的选择性比 staff_id 更高，因此最好把 customer_id 列放在多列索引的前面。
    >
    >    ```
    >    SELECT COUNT(DISTINCT staff_id)/COUNT(*) AS staff_id_selectivity,
    >    COUNT(DISTINCT customer_id)/COUNT(*) AS customer_id_selectivity,
    >    COUNT(*)
    >    FROM payment;
    >       staff_id_selectivity: 0.0001
    >    customer_id_selectivity: 0.0373
    >                   COUNT(*): 16049
    >    ```

  * 索引的使用条件

    > - 对于非常小的表、大部分情况下简单的全表扫描比建立索引更高效；
    > - 对于中到大型的表，索引就非常有效；
    > - 但是对于特大型的表，建立和维护索引的代价将会随之增长。这种情况下，需要用到一种技术可以直接区分出需要查询的一组数据，而不是一条记录一条记录地匹配，例如可以使用分区技术。

* #### 存储引擎

  * InnoDB

    > 是 MySQL 默认的事务型存储引擎，只有在需要它不支持的特性时，才考虑使用其它存储引擎。
    >
    > 实现了四个标准的隔离级别，默认级别是可重复读（REPEATABLE READ）。在可重复读隔离级别下，通过多版本并发控制（MVCC）+ 间隙锁（Next-Key Locking）防止幻影读。
    >
    > 主索引是聚簇索引，在索引中保存了数据，从而避免直接读取磁盘，因此对查询性能有很大的提升。
    >
    > 内部做了很多优化，包括从磁盘读取数据时采用的可预测性读、能够加快读操作并且自动创建的自适应哈希索引、能够加速插入操作的插入缓冲区等。
    >
    > 支持真正的在线热备份。其它存储引擎不支持在线热备份，要获取一致性视图需要停止对所有表的写入，而在读写混合场景中，停止写入可能也意味着停止读取。

  * MyISAM

    > 设计简单，数据以紧密格式存储。对于只读数据，或者表比较小、可以容忍修复操作，则依然可以使用它。
    >
    > 提供了大量的特性，包括压缩表、空间数据索引等。
    >
    > 不支持事务。
    >
    > 不支持行级锁，只能对整张表加锁，读取时会对需要读到的所有表加共享锁，写入时则对表加排它锁。但在表有读取操作的同时，也可以往表中插入新的记录，这被称为并发插入（CONCURRENT INSERT）。
    >
    > 可以手工或者自动执行检查和修复操作，但是和事务恢复以及崩溃恢复不同，可能导致一些数据丢失，而且修复操作是非常慢的。
    >
    > 如果指定了 DELAY_KEY_WRITE 选项，在每次修改执行完成时，不会立即将修改的索引数据写入磁盘，而是会写到内存中的键缓冲区，只有在清理键缓冲区或者关闭表的时候才会将对应的索引块写入磁盘。这种方式可以极大的提升写入性能，但是在数据库或者主机崩溃时会造成索引损坏，需要执行修复操作。

  * **比较**

    >- 事务：InnoDB 是事务型的，可以使用 Commit 和 Rollback 语句。
    >- 并发：MyISAM 只支持表级锁，而 InnoDB 还支持行级锁。
    >- 外键：InnoDB 支持外键。
    >- 备份：InnoDB 支持在线热备份。
    >- 崩溃恢复：MyISAM 崩溃后发生损坏的概率比 InnoDB 高很多，而且恢复的速度也更慢。
    >- 其它特性：MyISAM 支持压缩表和空间数据索引。



## Redis

[引用自Redis](https://github.com/CyC2018/CS-Notes/blob/master/notes/Redis.md)

* #### 数据类型  

  | 数据类型 | 可以存储的值           | 操作                                                         |
  | -------- | ---------------------- | ------------------------------------------------------------ |
  | STRING   | 字符串、整数或者浮点数 | 对整个字符串或者字符串的其中一部分执行操作 对整数和浮点数执行自增或者自减操作 |
  | LIST     | 列表                   | 从两端压入或者弹出元素  对单个或者多个元素 进行修剪，只保留一个范围内的元素 |
  | SET      | 无序集合               | 添加、获取、移除单个元素 检查一个元素是否存在于集合中 计算交集、并集、差集 从集合里面随机获取元素 |
  | HASH     | 包含键值对的无序散列表 | 添加、获取、移除单个键值对 获取所有键值对 检查某个键是否存在 |
  | ZSET     | 有序集合               | 添加、获取、删除元素 根据分值范围或者成员来获取元素 计算一个键的排名 |

  * STRING

    ```java
    > set hello world
    OK
    > get hello
    "world"
    > del hello
    (integer) 1
    > get hello
    (nil)
    ```

  * LIST

    ```java
    > rpush list-key item
    (integer) 1
    > rpush list-key item2
    (integer) 2
    > rpush list-key item
    (integer) 3
    
    > lrange list-key 0 -1
    1) "item"
    2) "item2"
    3) "item"
    
    > lindex list-key 1
    "item2"
    
    > lpop list-key
    "item"
    
    > lrange list-key 0 -1
    1) "item2"
    2) "item"
    ```

  * SET

    ```java
    > sadd set-key item
    (integer) 1
    > sadd set-key item2
    (integer) 1
    > sadd set-key item3
    (integer) 1
    > sadd set-key item
    (integer) 0
    
    > smembers set-key
    1) "item"
    2) "item2"
    3) "item3"
    
    > sismember set-key item4
    (integer) 0
    > sismember set-key item
    (integer) 1
    
    > srem set-key item2  //删除值
    (integer) 1
    > srem set-key item2
    (integer) 0
    
    > smembers set-key
    1) "item"
    2) "item3"
    ```

  * HASH

    ```java
    > hset hash-key sub-key1 value1
    (integer) 1
    > hset hash-key sub-key2 value2
    (integer) 1
    > hset hash-key sub-key1 value1
    (integer) 0
    
    > hgetall hash-key
    1) "sub-key1"
    2) "value1"
    3) "sub-key2"
    4) "value2"
    
    > hdel hash-key sub-key2
    (integer) 1
    > hdel hash-key sub-key2
    (integer) 0
    
    > hget hash-key sub-key1
    "value1"
    
    > hgetall hash-key
    1) "sub-key1"
    2) "value1"
    ```

  * ZSET

    ```java
    > zadd zset-key 728 member1
    (integer) 1
    > zadd zset-key 982 member0
    (integer) 1
    > zadd zset-key 982 member0
    (integer) 0
    
    > zrange zset-key 0 -1 withscores
    1) "member1"
    2) "728"
    3) "member0"
    4) "982"
    
    > zrangebyscore zset-key 0 800 withscores
    1) "member1"
    2) "728"
    
    > zrem zset-key member1
    (integer) 1
    > zrem zset-key member1
    (integer) 0
    
    > zrange zset-key 0 -1 withscores
    1) "member0"
    2) "982"
    ```

* #### 分布式锁实现

  > 在分布式场景下，无法使用单机环境下的锁来对多个节点上的进程进行同步。
  >
  > 可以使用 Reids 自带的 SETNX 命令实现分布式锁，除此之外，还可以使用官方提供的 RedLock 分布式锁实现。

* #### 数据淘汰策略

  | 策略            | 描述                                                 |
  | --------------- | ---------------------------------------------------- |
  | volatile-lru    | 从已设置过期时间的数据集中挑选最近最少使用的数据淘汰 |
  | volatile-ttl    | 从已设置过期时间的数据集中挑选将要过期的数据淘汰     |
  | volatile-random | 从已设置过期时间的数据集中任意选择数据淘汰           |
  | allkeys-lru     | 从所有数据集中挑选最近最少使用的数据淘汰             |
  | allkeys-random  | 从所有数据集中任意选择数据进行淘汰                   |
  | noeviction      | 禁止驱逐数据                                         |

  > 作为内存数据库，出于对性能和内存消耗的考虑，Redis 的淘汰算法实际实现上并非针对所有 key，而是抽样一小部分并且从中选出被淘汰的 key。
  >
  > 使用 Redis 缓存数据时，为了提高缓存命中率，需要保证缓存数据都是热点数据。可以将内存最大使用量设置为热点数据占用的内存量，然后启用 allkeys-lru 淘汰策略，将最近最少使用的数据淘汰。
  >
  > Redis 4.0 引入了 volatile-lfu 和 allkeys-lfu 淘汰策略，LFU 策略通过统计访问频率，将访问频率最少的键值对淘汰.

* #### 持久化

  * RDB 持久化  **快照模式**

    > 将某个时间点的所有数据都存放到硬盘上。
    >
    > 可以将快照复制到其它服务器从而创建具有相同数据的服务器副本。
    >
    > 如果系统发生故障，将会丢失最后一次创建快照之后的数据。
    >
    > 如果数据量很大，保存快照的时间会很长。

  * AOF 持久化  **日志**

    > 将写命令添加到 AOF 文件（Append Only File）的末尾。
    >
    > 使用 AOF 持久化需要设置同步选项，从而确保写命令什么时候会同步到磁盘文件上。这是因为对文件进行写入并不会马上将内容同步到磁盘上，而是先存储到缓冲区，然后由操作系统决定什么时候同步到磁盘。有以下同步选项：
    >
    > | 选项     | 同步频率                 |
    > | -------- | ------------------------ |
    > | always   | 每个写命令都同步         |
    > | everysec | 每秒同步一次             |
    > | no       | 让操作系统来决定何时同步 |
    >
    > - always 选项会严重减低服务器的性能；
    > - everysec 选项比较合适，可以保证系统崩溃时只会丢失一秒左右的数据，并且 Redis 每秒执行一次同步对服务器性能几乎没有任何影响；
    > - no 选项并不能给服务器性能带来多大的提升，而且也会增加系统崩溃时数据丢失的数量。
    >
    > 随着服务器写请求的增多，AOF 文件会越来越大。Redis 提供了一种将 AOF 重写的特性，能够去除 AOF 文件中的冗余写命令。

## String

* 不可变

  * 可以缓存hash值，如hashmap等需要

  * **String Pool 的需要**

    > 如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool。

  * 线程安全

  * 安全性

    > String 经常作为参数，String 不可变性可以保证参数不可变。例如在作为网络连接参数的情况下如果 String 是可变的，那么在网络连接过程中，String 被改变，改变 String 对象的那一方以为现在连接的是其它主机，而实际情况却不一定是。

* StringBuider StringBuffer

  * StringBuffer线程安全，内部使用 synchronized 进行同步
  * StringBuider 线程不安全

* String Pool 字符串常量池

  * 字符串常量池（String Pool）保存着所有字符串字面量（literal strings），这些字面量在编译时期就确定。不仅如此，还可以使用 String 的 intern() 方法在运行过程中将字符串添加到 String Pool 中。
  * 在 Java 7 之前，String Pool 被放在运行时常量池中，它属于永久代。而在 Java 7，String Pool 被移到堆中。这是因为永久代的空间有限，在大量使用字符串的场景下会导致 OutOfMemoryError 错误。


## 缓存池

基本类型对应的缓冲池如下：

- boolean values true and false   
- all byte values
- short values between -128 and 127
- int values between -128 and 127
- char in the range \u0000 to \u007F

#### 示例

new Integer(123) 与 Integer.valueOf(123) 的区别在于：

- new Integer(123) 每次都会新建一个对象；

- Integer.valueOf(123) 会使用缓存池中的对象，多次调用会取得同一个对象的引用。

- 在 Java 8 中，Integer 缓存池的大小默认为 -128~127。

- 编译器会在自动装箱过程调用 valueOf() 方法，因此多个 Integer 实例使用自动装箱来创建并且值相同，那么就会引用相同的对象。

  ```java
  Integer m = 123;
  Integer n = 123;
  System.out.println(m == n); // true
  ```

#### Java值传递Or引用传递

> Java is always **pass-by-value**
>
> ```java
> public static void main(String[] args) {
>     Dog aDog = new Dog("Max");
>     // we pass the object to foo
>     foo(aDog);
>     // aDog variable is still pointing to the "Max" dog when foo(...) returns
>     aDog.getName().equals("Max"); // true
>     aDog.getName().equals("Fifi"); // false 
> }
> 
> public static void foo(Dog d) {
>     d.getName().equals("Max"); // true
>     // change d inside of foo() to point to a new Dog instance "Fifi"
>     d = new Dog("Fifi");
>     d.getName().equals("Fifi"); // true
> }
> ```

## 关键字

#### final

* 字段

  > - 对于基本类型，final 使数值不变；
  > - 对于引用类型，final 使引用不变，也就不能引用其它对象，但是被引用的对象本身是可以修改的。
  >
  > ```java
  > final int x = 1;
  > // x = 2;  // cannot assign value to final variable 'x'
  > final A y = new A();
  > y.a = 1;
  > ```

* 方法

  > 声明方法不能被子类重写。
  >
  > private 方法隐式地被指定为 final，如果在子类中定义的方法和基类中的一个 private 方法签名相同，此时子类的方法不是重写基类方法，而是在子类中定义了一个新的方法。

* 类

  声明类不允许被继承。

## 反射

[深入了解反射](https://www.sczyh30.com/posts/Java/java-reflection-1/)

每个类都有一个 **Class** 对象，包含了与类有关的信息。当编译一个新类时，会产生一个同名的 .class 文件，该文件内容保存着 Class 对象。

类加载相当于 Class 对象的加载，类在第一次使用时才动态加载到 JVM 中。也可以使用 `Class.forName("com.mysql.jdbc.Driver")` 这种方式来控制类的加载，该方法会返回一个 Class 对象。

反射可以提供运行时的类信息，并且这个类可以在运行时才加载进来，甚至在编译时期该类的 .class 不存在也可以加载进来。

Class 和 java.lang.reflect 一起对反射提供了支持，java.lang.reflect 类库主要包含了以下三个类：

- **Field** ：可以使用 get() 和 set() 方法读取和修改 Field 对象关联的字段；
- **Method** ：可以使用 invoke() 方法调用与 Method 对象关联的方法；
- **Constructor** ：可以用 Constructor 创建新的对象。

**获取class对象的方法集合**，主要有以下几个方法:

* `getDeclaredMethods` 方法返回类或接口声明的所有方法，包括公共、保护、默认（包）访问和私有方法，但不包括继承的方法。

```
public Method[] getDeclaredMethods() throws SecurityException
```

- `getMethods` 方法返回某个类的所有公用（public）方法，包括其继承类的公用方法。

```
public Method[] getMethods() throws SecurityException
```

- `getMethod` 方法返回一个特定的方法，其中第一个参数为方法名称，后面的参数为方法的参数对应Class的对象。

```
public Method getMethod(String name, Class<?>... parameterTypes)
```

**获取构造器信息**

获取类构造器的用法与上述获取方法的用法类似。主要是通过Class类的getConstructor方法得到Constructor类的一个实例，而Constructor类有一个newInstance方法可以创建一个对象实例:

```
public T newInstance(Object ... initargs)
```

此方法可以根据传入的参数来调用对应的Constructor创建对象实例。

**获取类的成员变量（字段）信息**

主要是这几个方法，在此不再赘述：

- `getFiled`：访问公有的成员变量
- `getDeclaredField`：所有已声明的成员变量，但不能得到其父类的成员变量

`getFileds` 和 `getDeclaredFields` 方法用法同上（参照 Method）。

## 异常

Throwable 可以用来表示任何可以作为异常抛出的类，分为两种： **Error** 和 **Exception**。其中 Error 用来表示 JVM 无法处理的错误，Exception 分为两种：

- **受检异常** ：需要用 try...catch... 语句捕获并进行处理，并且可以从异常中恢复；
- **非受检异常** ：是程序运行时错误，例如除 0 会引发 Arithmetic Exception，此时程序崩溃并且无法恢复

![异常](pic\异常.png)

引申：logback打印异常堆栈

```java
logger.info("error message:",e)
```

java 7以上 支持一个try块捕捉多个异常

``` java
catch(IOException | SQLException | Exception ex){
     logger.error(ex);
     throw new MyException(ex.getMessage());
}
```

try-with-resource

``` java
//try块里可以写多个资源的打开
static String readFirstLineFromFile(String path) throws IOException {
    try (BufferedReader br =
                   new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
}
```

## 泛型

[泛型详解](http://www.importnew.com/24029.html)

```java
public class Box<T> {
    // T stands for "Type"
    private T t;
    public void set(T t) { this.t = t; }
    public T get() { return t; }
}
```

#### 泛型方法

声明一个泛型方法很简单，只要在_**返回类型**_前面加上一个类似`<K, V>`的形式就行了：

```java
public class Util {
    public static <K, V> boolean compare(Pair<K, V> p1, Pair<K, V> p2) {
        return p1.getKey().equals(p2.getKey()) &&
               p1.getValue().equals(p2.getValue());
    }
}
public class Pair<K, V> {
    private K key;
    private V value;
    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }
    public void setKey(K key) { this.key = key; }
    public void setValue(V value) { this.value = value; }
    public K getKey()   { return key; }
    public V getValue() { return value; }
}
```

#### 边界符

```java
//T 必须实现comparable接口
public static <T extends Comparable<T>> int countGreaterThan(T[] anArray, T elem) {
    int count = 0;
    for (T e : anArray)
        if (e.compareTo(elem) > 0)
            ++count;
    return count;
}
```

#### 通配符 ？

**Producer Extends, Consumer Super**  PECS原则

频繁往外读取内容的，适合用上界Extends。

经常往里插入的，适合用下界Super。

- “Producer Extends” – 如果你需要一个只读List，用它来produce T，那么使用`? extends T`。
- “Consumer Super” – 如果你需要一个只写List，用它来consume T，那么使用`? super T`。
- 如果需要同时读取以及写入，那么我们就不能使用通配符了。

```java
// 参数为T的子类
class Fruit {}
class Apple extends Fruit {}
class Orange extends Fruit {}

//get时 extends
static class CovariantReader<T> {
    T readCovariant(List<? extends T> list) {
        return list.get(0);
    }
}


//add时 super
 static <T> void writeWithWildcard(List<? super T> list, T item) {
        list.add(item)
 }
```



1) 参数写成：T<? super B>，对于这个泛型，?代表容器里的元素类型，由于只规定了元素必须是B的超类，导致元素没有明确统一的“根”（除了Object这个必然的根），所以这个泛型你其实无法使用它，对吧，除了把元素强制转成Object。所以，对把参数写成这样形态的函数，你函数体内，只能对这个泛型做**插入操作，而无法读**

2) 参数写成： T<? extends B>，由于指定了B为所有元素的“根”，你任何时候都可以安全的用B来使用容器里的元素，但是插入有问题，由于供奉B为祖先的子树有很多，不同子树并不兼容，由于实参可能来自于任何一颗子树，所以你的插入很可能破坏函数实参，所以，对这种写法的形参，**禁止做插入操作，只做读取**

#### Array中可以用泛型吗?

　　Array事实上并不支持泛型，这也是为什么Joshua Bloch在Effective Java一书中建议使用List来代替Array，因为List可以提供编译期的类型安全保证，而Array却不能。	

## 注解

注解本质是一个继承了Annotation的特殊接口，其具体实现类是Java运行时生成的动态代理类。而我们通过反射获取注解时，返回的是Java运行时生成的动态代理对象$Proxy1。通过代理对象调用自定义注解（接口）的方法，会最终调用AnnotationInvocationHandler的invoke方法。该方法会从memberValues这个Map中索引出对应的值。而memberValues的来源是Java常量池。

#### 元注解

* @Documented –注解是否将包含在JavaDoc中

* @Retention –什么时候使用该注解

  >   ●   RetentionPolicy.SOURCE : 在编译阶段丢弃。这些注解在编译结束之后就不再有任何意义，所以它们不会写入字节码。@Override, @SuppressWarnings都属于这类注解。
  >   ●   RetentionPolicy.CLASS : 在类加载的时候丢弃。在字节码文件的处理中有用。注解默认使用这种方式
  >   ●   RetentionPolicy.RUNTIME : 始终不会丢弃，运行期也保留该注解，因此可以使用反射机制读取该注解的信息。我们自定义的注解通常使用这种方式。

* @Target – 表示该注解用于什么地方。默认值为任何元素，表示该注解用于什么地方

  >   ● ElementType.CONSTRUCTOR:用于描述构造器
  >   ● ElementType.FIELD:成员变量、对象、属性（包括enum实例）
  >   ● ElementType.LOCAL_VARIABLE:用于描述局部变量
  >   ● ElementType.METHOD:用于描述方法
  >   ● ElementType.PACKAGE:用于描述包
  >   ● ElementType.PARAMETER:用于描述参数
  >   ● ElementType.TYPE:用于描述类、接口(包括注解类型) 或enum声明

* @Inherited – 定义该注释和子类的关系

  > @Inherited 元注解是一个标记注解，@Inherited阐述了某个被标注的类型是被继承的。如果一个使用了@Inherited修饰的annotation类型被用于一个class，则这个annotation将被用于该class的子类

#### 注解示例

```java
/**
 * 水果供应者注解
 */
@Target(FIELD)   
@Retention(RUNTIME)
@Documented
public @interface FruitProvider {
    /**
     * 供应商编号
     */
    public int id() default -1;
    
    /**
     * 供应商名称
     */
    public String name() default "";
    
    /**
     * 供应商地址
     */
    public String address() default "";
}
```

#### 引申

如何获取注解

```java
//获取类上的注解
Class.getAnnotations() 获取所有的注解，包括自己声明的以及继承的
Class.getAnnotation(Class< A > annotationClass) 获取指定的注解，该注解可以是自己声明的，也可以是继承的
Class.getDeclaredAnnotations() 获取自己声明的注解
//获取方法上的注解
 method.getAnnotations();
 Annotation[] deMAnnos = method.getDeclaredAnnotations();
 Annotation subMAnno = method.getAnnotation(SubAnnotation.class);
//获取

```

## 容器



#### Collection

概览

![collection全家桶](pic\collection.png)



##### 1. Set

- TreeSet：基于红黑树实现，支持有序性操作，例如根据一个范围查找元素的操作。但是查找效率不如 HashSet，HashSet 查找的时间复杂度为 O(1)，TreeSet 则为 O(logN)。
- HashSet：基于哈希表实现，支持快速查找，但不支持有序性操作。并且失去了元素的插入顺序信息，也就是说使用 Iterator 遍历 HashSet 得到的结果是不确定的。
- LinkedHashSet：具有 HashSet 的查找效率，且内部使用双向链表维护元素的插入顺序。

##### 2. List

- ArrayList：基于动态数组实现，支持随机访问。
- Vector：和 ArrayList 类似，但它是线程安全的。
- LinkedList：基于双向链表实现，只能顺序访问，但是可以快速地在链表中间插入和删除元素。不仅如此，LinkedList 还可以用作栈、队列和双向队列。

##### 3. Queue

- LinkedList：可以用它来实现双向队列。	

- PriorityQueue：基于堆结构实现，可以用它来实现优先队列。

  > 优先队列，可以解决top K问题，内部结构为最大最小堆

#### Map

- TreeMap：基于红黑树实现。

  > 可以用来实现签名时的字段ascii码排序

- HashMap：基于哈希表实现。

- HashTable：和 HashMap 类似，但它是线程安全的，这意味着同一时刻多个线程可以同时写入 HashTable 并且不会导致数据不一致。它是遗留类，不应该去使用它。现在可以使用 ConcurrentHashMap 来支持线程安全，并且 ConcurrentHashMap 的效率会更高，因为 ConcurrentHashMap 引入了分段锁。

- LinkedHashMap：使用双向链表来维护元素的顺序，顺序为插入顺序或者最近最少使用（LRU）顺序

  > ```java
  > private static final int MAX_ENTRIES = 100;
  > 
  > //重写这个方法，可以控制在什么条件下移除最老的元素
  > //该方法被put和putall调用  默认返回false
  > @Override
  > protected boolean removeEldestEntry(Map.Entry eldest) {
  > 	//return false;
  >     return size() > MAX_ENTRIES;
  > }
  > ```

#### 容器中的设计模式

* 迭代器模式

  > Collection 实现了 Iterable 接口，其中的 iterator() 方法能够产生一个 Iterator 对象，通过这个对象就可以迭代遍历 Collection 中的元素。
  >
  > for-each

* 适配器模式

  > java.util.Arrays#asList() 可以把数组类型转换为 List 类型。
  >
  > ```java
  > @SafeVarargs
  > public static <T> List<T> asList(T... a)
  > ```
  >
  > 应该注意的是 asList() 的参数为泛型的变长参数，不能使用基本类型数组作为参数，只能使用相应的包装类型数组。
  >
  > ```java
  > Integer[] arr = {1, 2, 3};
  > List list = Arrays.asList(arr);
  > ```
  >
  > 也可以使用以下方式调用 asList()：
  >
  > ```java
  > List list = Arrays.asList(1, 2, 3);
  > ```



