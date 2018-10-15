

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