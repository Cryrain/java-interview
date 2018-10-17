## Collection源码

基于JDK1.8

#### ArrayList

- 实现RandomAccess 接口

- 数组的默认大小为 10。

  > ```java
  > private static final int DEFAULT_CAPACITY = 10;
  > ```

- **扩容**

  > 添加元素时使用 ensureCapacityInternal() 方法来保证容量足够，如果不够时，需要使用 grow() 方法进行扩容，新容量的大小为 `oldCapacity + (oldCapacity >> 1)`，也就是旧容量的 1.5 倍。
  >
  > 扩容操作需要调用 `Arrays.copyOf()` 把原数组整个复制到新数组中，这个操作代价很高，因此最好在创建 ArrayList 对象时就指定大概的容量大小，减少扩容操作的次数。
  >
  > ```java
  > public boolean add(E e) {
  >     ensureCapacityInternal(size + 1);  // Increments modCount!!
  >     elementData[size++] = e;
  >     return true;
  > }
  > 
  > private void ensureCapacityInternal(int minCapacity) {
  >     if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
  >         minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
  >     }
  >     ensureExplicitCapacity(minCapacity);
  > }
  > 
  > private void ensureExplicitCapacity(int minCapacity) {
  >     modCount++;
  >     // overflow-conscious code
  >     if (minCapacity - elementData.length > 0)
  >         grow(minCapacity);
  > }
  > 
  > private void grow(int minCapacity) {
  >     // overflow-conscious code
  >     int oldCapacity = elementData.length;
  >     int newCapacity = oldCapacity + (oldCapacity >> 1);
  >     if (newCapacity - minCapacity < 0)
  >         newCapacity = minCapacity;
  >     if (newCapacity - MAX_ARRAY_SIZE > 0)
  >         newCapacity = hugeCapacity(minCapacity);
  >     // minCapacity is usually close to size, so this is a win:
  >     elementData = Arrays.copyOf(elementData, newCapacity);
  > }
  > ```

- 删除元素

  > 需要调用 System.arraycopy() 将 index+1 后面的元素都复制到 index 位置上，该操作的时间复杂度为 O(N)，可以看出 ArrayList 删除元素的代价是非常高的。
  >
  > ```java
  > public E remove(int index) {
  >     rangeCheck(index);
  >     modCount++;
  >     E oldValue = elementData(index);
  >     int numMoved = size - index - 1;
  >     if (numMoved > 0)
  >         System.arraycopy(elementData, index+1, elementData, index, numMoved);
  >     elementData[--size] = null; // clear to let GC do its work
  >     return oldValue;
  > }
  > ```

- Fail-Fast 快速失败

  > modCount 用来记录 ArrayList 结构发生变化的次数。结构发生变化是指添加或者删除至少一个元素的所有操作，或者是调整内部数组的大小，仅仅只是设置元素的值不算结构发生变化。
  >
  > 在进行序列化或者迭代等操作时，需要比较操作前后 modCount 是否改变，如果改变了需要抛出 ConcurrentModificationException。
  >
  > ```java
  > //序列化
  > private void writeObject(java.io.ObjectOutputStream s)
  >     throws java.io.IOException{
  >     // Write out element count, and any hidden stuff
  >     int expectedModCount = modCount;
  >     s.defaultWriteObject();
  > 
  >     // Write out size as capacity for behavioural compatibility with clone()
  >     s.writeInt(size);
  > 
  >     // Write out all elements in the proper order.
  >     for (int i=0; i<size; i++) {
  >         s.writeObject(elementData[i]);
  >     }
  > 
  >     //这里比较是否发生了改变，改变则抛出异常
  >     if (modCount != expectedModCount) {
  >         throw new ConcurrentModificationException();
  >     }
  > }
  > ```

- 序列化

  > ArrayList 基于数组实现，并且具有动态扩容特性，因此保存元素的数组不一定都会被使用，那么就没必要全部进行序列化。
  >
  > 保存元素的数组 elementData 使用 transient 修饰，该关键字声明数组默认不会被序列化。
  >
  > ```
  > transient Object[] elementData; // non-private to simplify nested class access
  > 
  > ```
  >
  > ArrayList 实现了 writeObject() 和 readObject() 来控制只序列化数组中有元素填充那部分内容。
  >
  > ```java
  > private void readObject(java.io.ObjectInputStream s)
  >     throws java.io.IOException, ClassNotFoundException {
  >     elementData = EMPTY_ELEMENTDATA;
  > 
  >     // Read in size, and any hidden stuff
  >     s.defaultReadObject();
  > 
  >     // Read in capacity
  >     s.readInt(); // ignored
  > 
  >     if (size > 0) {
  >         // be like clone(), allocate array based upon size not capacity
  >         ensureCapacityInternal(size);
  > 
  >         Object[] a = elementData;
  >         // Read in all elements in the proper order.
  >         for (int i=0; i<size; i++) {
  >             a[i] = s.readObject();
  >         }
  >     }
  > }
  > private void writeObject(java.io.ObjectOutputStream s)
  >     throws java.io.IOException{
  >     // Write out element count, and any hidden stuff
  >     int expectedModCount = modCount;
  >     s.defaultWriteObject();
  > 
  >     // Write out size as capacity for behavioural compatibility with clone()
  >     s.writeInt(size);
  > 
  >     // Write out all elements in the proper order.
  >     for (int i=0; i<size; i++) {
  >         s.writeObject(elementData[i]);
  >     }
  > 
  >     if (modCount != expectedModCount) {
  >         throw new ConcurrentModificationException();
  >     }
  > }
  > ```
  >
  > 序列化时需要使用 ObjectOutputStream 的 writeObject() 将对象转换为字节流并输出。而 writeObject() 方法在传入的对象存在 writeObject() 的时候会去反射调用该对象的 writeObject() 来实现序列化。反序列化使用的是 ObjectInputStream 的 readObject() 方法，原理类似。
  >
  > ```java
  > ArrayList list = new ArrayList();
  > ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(file));
  > oos.writeObject(list);
  > ```

#### Vector

它的实现与 ArrayList 类似，但是使用了 synchronized 进行同步,一般不推荐使用

> ```java
> public synchronized boolean add(E e) {
>     modCount++;
>     ensureCapacityHelper(elementCount + 1);
>     elementData[elementCount++] = e;
>     return true;
> }
> 
> public synchronized E get(int index) {
>     if (index >= elementCount)
>         throw new ArrayIndexOutOfBoundsException(index);
> 
>     return elementData(index);
> }
> ```

- Vector 是同步的，因此开销就比 ArrayList 要大，访问速度更慢。最好使用 ArrayList 而不是 Vector，因为同步操作完全可以由程序员自己来控制；
- Vector 每次扩容请求其大小的 2 倍空间，而 ArrayList 是 1.5 倍。

##### 替代vector

可以使用 `Collections.synchronizedList();` 得到一个线程安全的 ArrayList。

```java
List<String> list = new ArrayList<>();
List<String> synList = Collections.synchronizedList(list);
```

也可以使用 concurrent 并发包下的 CopyOnWriteArrayList 类。

```java
List<String> list = new CopyOnWriteArrayList<>();
```

#### CopyOnWriteArrayList

写操作在一个复制的数组上进行，读操作还是在原始数组中进行，读写分离，互不影响。

写操作需要加锁，防止并发写入时导致写入数据丢失。

写操作结束之后需要把原始数组指向新的复制数组。

```java
public boolean add(E e) {
    //写加锁保证同步
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        //再每添加一个元素都要复制一份，然后在复制的数组里添加元素
        Object[] newElements = Arrays.copyOf(elements, len + 1);  
        newElements[len] = e;
        //添加完成后，读数组再指向新数组
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}

final void setArray(Object[] a) {
    array = a;
}
@SuppressWarnings("unchecked")
private E get(Object[] a, int index) {
    return (E) a[index];
}
```

#### 适用场景

CopyOnWriteArrayList 在写操作的同时允许读操作，大大提高了读操作的性能，因此很适合读多写少的应用场景。

但是 CopyOnWriteArrayList 有其缺陷：

- **内存占用**：在写操作时需要复制一个新的数组，使得内存占用为原来的两倍左右；
- **数据不一致**：读操作不能读取实时性的数据，因为部分写操作的数据还未同步到读数组中。

所以 CopyOnWriteArrayList 不适合内存敏感以及对实时性要求很高的场景。

#### LinkedList

基于双向链表实现，使用 Node 存储链表节点信息。

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;
}
```

每个链表存储了 first 和 last 指针：

```java
transient Node<E> first;
transient Node<E> last;
```

##### _ArrayList比较_

- ArrayList 基于动态数组实现，LinkedList 基于双向链表实现；
- ArrayList 支持随机访问，LinkedList 不支持；
- LinkedList 在任意位置添加删除元素更快。



#### HashMap

JDK1.7



##### 存储结构

内部包含了一个 Entry 类型的数组 table。

```java
transient Entry[] table;
```

Entry 存储着键值对。它包含了四个字段，从 next 字段我们可以看出 Entry 是一个链表。即数组中的每个位置被当成一个桶，一个桶存放一个链表。HashMap 使用拉链法来解决冲突，同一个链表中存放哈希值相同的 Entry。

![hashmap结构](C:/code/mycode/java-interview/java-se/pic/hashmap_entry.png)

```java
static class Entry<K,V> implements Map.Entry<K,V> {
    final K key;
    V value;
    Entry<K,V> next;   //链表
    int hash;      //节点hash值

    Entry(int h, K k, V v, Entry<K,V> n) {
        value = v;
        next = n;
        key = k;
        hash = h;
    }

    public final K getKey() {
        return key;
    }

    public final V getValue() {
        return value;
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry e = (Map.Entry)o;
        Object k1 = getKey();
        Object k2 = e.getKey();
        if (k1 == k2 || (k1 != null && k1.equals(k2))) {
            Object v1 = getValue();
            Object v2 = e.getValue();
            if (v1 == v2 || (v1 != null && v1.equals(v2)))
                return true;
        }
        return false;
    }

    public final int hashCode() {
        return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
    }

    public final String toString() {
        return getKey() + "=" + getValue();
    }
}
```

##### 拉链法的工作原理

```java
HashMap<String, String> map = new HashMap<>();
map.put("K1", "V1");
map.put("K2", "V2");
map.put("K3", "V3");
```

- 新建一个 HashMap，默认大小为 16；
- 插入 <K1,V1> 键值对，先计算 K1 的 hashCode 为 115，使用除留余数法得到所在的桶下标 115%16=3。
- 插入 <K2,V2> 键值对，先计算 K2 的 hashCode 为 118，使用除留余数法得到所在的桶下标 118%16=6。
- 插入 <K3,V3> 键值对，先计算 K3 的 hashCode 为 118，使用除留余数法得到所在的桶下标 118%16=6，插在 <K2,V2> 前面。

应该注意到链表的插入是以**头插法**方式进行的，例如上面的 <K3,V3> 不是插在 <K2,V2> 后面，而是插入在链表头部。

查找需要分成两步进行：

- 计算键值对所在的桶；
- 在链表上顺序查找，时间复杂度显然和链表的长度成正比。（jdk1.7）

##### PUT 操作

JDK1.8版本

```java
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            //n为当前table数组的长度
            n = (tab = resize()).length;
        //如果table的该位置为null，则把元素放到该位置，把table该位置的元素赋值给p
        //(n - 1) & hash]取模，寻找桶位置
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            //如果该位置头结点hash值一样并且key也相同，赋值给e
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            //如果该节点已经为treeNode，则调用putTreeVal添加元素
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        //头插法 插入节点
                        p.next = newNode(hash, key, value, null);
                        //如果该bucket容量超过REEIFY_THRESHOLD(8)，则转为红黑树结构
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            //如果相同则替换
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

- 如果key=null，则hash值为0，即放入table的第一个位置

  ```java
  public V put(K key, V value) {
      return putVal(hash(key), key, value, false, true);
  }
  
   static final int hash(Object key) {
          int h;
          return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
      }
  ```



##### 确定桶下标

1. 计算hahs值

2. 取模

   > **为什么table容量为2的n次方？**
   >
   > 令 x = 1<<4，即 x 为 2 的 4 次方，它具有以下性质：
   >
   > ```
   > x   : 00010000
   > x-1 : 00001111
   > 
   > ```
   >
   > 令一个数 y 与 x-1 做与运算，可以去除 y 位级表示的第 4 位以上数：
   >
   > ```
   > y       : 10110010
   > x-1     : 00001111
   > y&(x-1) : 00000010
   > 
   > ```
   >
   > 这个性质和 y 对 x 取模效果是一样的：
   >
   > ```
   > y   : 10110010
   > x   : 00010000
   > y%x : 00000010
   > 
   > ```
   >
   > 我们知道，位运算的代价比求模运算小的多，因此在进行这种计算时用位运算的话能带来更高的性能。
   >
   > 确定桶下标的最后一步是将 key 的 hash 值对桶个数取模：hash%capacity，如果能保证 capacity 为 2 的 n 次方，那么就可以将这个操作转换为位运算。
   >
   > ```java
   > static int indexFor(int h, int length) {
   >     return h & (length-1);
   > }
   > ```

##### 扩容

扩容相关的参数主要有：capacity、size、threshold 和 load_factor。

| 参数       | 含义                                                         |
| ---------- | ------------------------------------------------------------ |
| capacity   | table 的容量大小，默认为 16。需要注意的是 capacity 必须保证为 2 的 n 次方。 |
| size       | table 的实际使用量。                                         |
| threshold  | size 的临界值，size 必须小于 threshold，如果大于等于，就必须进行扩容操作。 |
| loadFactor | 装载因子，table 能够使用的比例，threshold = capacity * loadFactor。 |

```java
//当需要扩容时，新的容量为 2倍
newCap = oldCap << 1
```

初始化保证table容量为2的倍数，当用户初始化hashmap时指定容量则用下面这个方法来保证是2的倍数

```java
/**
 * Returns a power of two size for the given target capacity.
 */
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

#### Rehash

该步骤为重新把所有存储的Entry重新分布到新table中，这里并没有TreeNode，只有在put时才会有，rehash不会。

```java
threshold = newThr;
@SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
table = newTab;
if (oldTab != null) {
//rehash 循环开始
    for (int j = 0; j < oldCap; ++j) {
        Node<K,V> e;
        if ((e = oldTab[j]) != null) {
            oldTab[j] = null;
            if (e.next == null)
            //如果当前table没有，则设置e为head
                newTab[e.hash & (newCap - 1)] = e;
            else if (e instanceof TreeNode)
            //treenode红黑树处理方法
                ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
            else { // preserve order
                Node<K,V> loHead = null, loTail = null;
                Node<K,V> hiHead = null, hiTail = null;
                Node<K,V> next;
                //链表 头插法
                do {
                    next = e.next;
                    if ((e.hash & oldCap) == 0) {
                        if (loTail == null)
                            loHead = e;
                        else
                            loTail.next = e;
                        loTail = e;
                    }
                    else {
                        if (hiTail == null)
                            hiHead = e;
                        else
                            hiTail.next = e;
                        hiTail = e;
                    }
                } while ((e = next) != null);
                if (loTail != null) {
                    loTail.next = null;
                    newTab[j] = loHead;
                }
                if (hiTail != null) {
                    hiTail.next = null;
                    newTab[j + oldCap] = hiHead;
                }
            }
        }
    }
}
```



##### 与HashTable比较

- HashTable 使用 synchronized 来进行同步。
- HashMap 可以插入键为 null 的 Entry。(null的hash为0)
- HashMap 的迭代器是 fail-fast 迭代器。
- HashMap 不能保证随着时间的推移 Map 中的元素次序是不变的。

##### ConcurrentHashMap



##### JDK1.7 

采用分段锁 （Segment），每个分段锁维护着几个桶（HashEntry），多个线程可以同时访问不同分段锁上的桶，从而使其并发度更高（并发度就是 Segment 的个数）。Segment 继承自 ReentrantLock。



默认的并发级别为 16，也就是说默认创建 16 个 Segment。

```java
static final int DEFAULT_CONCURRENCY_LEVEL = 16;
```

##### size操作

每个 Segment 维护了一个 count 变量来统计该 Segment 中的键值对个数。

```
/**
 * The number of elements. Accessed only either within locks
 * or among other volatile reads that maintain visibility.
 */
transient int count;
```

在执行 size 操作时，需要遍历所有 Segment 然后把 count 累计起来。

ConcurrentHashMap 在执行 size 操作时先尝试不加锁，如果连续两次不加锁操作得到的结果一致，那么可以认为这个结果是正确的。

尝试次数使用 RETRIES_BEFORE_LOCK 定义，该值为 2，retries 初始值为 -1，因此尝试次数为 3。

如果尝试的次数超过 3 次，就需要对每个 Segment 加锁

##### JDK1.8改动

* 主要是加锁方式变化，抛弃了Segment分段锁，改为使用效率更好的CAS算法，

* put操作源码

  ```java
  final V putVal(K key, V value, boolean onlyIfAbsent) {
      if (key == null || value == null) throw new NullPointerException();
      int hash = spread(key.hashCode());
      int binCount = 0;
      for (Node<K,V>[] tab = table;;) {
          Node<K,V> f; int n, i, fh;
          if (tab == null || (n = tab.length) == 0)
              tab = initTable();
          else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
              //如果是空桶的话则用cas算法 直接插入，不锁
              if (casTabAt(tab, i, null,
                           new Node<K,V>(hash, key, value, null)))
                  break;                   // no lock when adding to empty bin
          }
          else if ((fh = f.hash) == MOVED)
              tab = helpTransfer(tab, f);
          else {
              V oldVal = null;
              //不是空桶的话，锁当前桶的链表头节点
              synchronized (f) {
                  if (tabAt(tab, i) == f) {
                      if (fh >= 0) {
                          binCount = 1;
                          for (Node<K,V> e = f;; ++binCount) {
                              K ek;
                              if (e.hash == hash &&
                                  ((ek = e.key) == key ||
                                   (ek != null && key.equals(ek)))) {
                                  oldVal = e.val;
                                  if (!onlyIfAbsent)
                                      e.val = value;
                                  break;
                              }
                              Node<K,V> pred = e;
                              if ((e = e.next) == null) {
                                  pred.next = new Node<K,V>(hash, key,
                                                            value, null);
                                  break;
                              }
                          }
                      }
                      else if (f instanceof TreeBin) {
                          Node<K,V> p;
                          binCount = 2;
                          if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                         value)) != null) {
                              oldVal = p.val;
                              if (!onlyIfAbsent)
                                  p.val = value;
                          }
                      }
                  }
              }
              if (binCount != 0) {
                  if (binCount >= TREEIFY_THRESHOLD)
                      treeifyBin(tab, i);
                  if (oldVal != null)
                      return oldVal;
                  break;
              }
          }
      }
      addCount(1L, binCount);
      return null;
  }
  ```

#### LinkedHashMap

##### 结构

继承自 HashMap，因此具有和 HashMap 一样的快速查找特性。

```java
public class LinkedHashMap<K,V> extends HashMap<K,V> implements Map<K,V>
```

内部维护了一个双向链表，用来维护插入顺序或者 LRU 顺序。

```java
/**
 * The head (eldest) of the doubly linked list.
 */
transient LinkedHashMap.Entry<K,V> head;

/**
 * The tail (youngest) of the doubly linked list.
 */
transient LinkedHashMap.Entry<K,V> tail;
```

accessOrder 决定了顺序，默认为 false，此时维护的是插入顺序。

```java
final boolean accessOrder;
```

LinkedHashMap 最重要的是以下用于维护顺序的函数，它们会在 put、get 等方法中调用。

```java
void afterNodeAccess(Node<K,V> p) { }
void afterNodeInsertion(boolean evict) { }
```

##### afterNodeAccess()

当一个节点被访问时，如果 accessOrder 为 true，则会将该节点移到链表尾部。也就是说指定为 LRU 顺序之后，在每次访问一个节点时，会将这个节点移到链表尾部，保证链表尾部是最近访问的节点，那么链表首部就是最近最久未使用的节点。

```java
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
        ++modCount;
    }
}
```

##### afterNodeInsertion()

在 put 等操作之后执行，当 removeEldestEntry() 方法返回 true 时会移除最晚的节点，也就是链表首部节点 first。

evict 只有在构建 Map 的时候才为 false，在这里为 true。

```java
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}
```

removeEldestEntry() 默认为 false，如果需要让它为 true，需要**继承 LinkedHashMap** 并且覆盖这个方法的实现，这在实现 LRU 的缓存中特别有用，通过移除最近最久未使用的节点，从而保证缓存空间足够，并且缓存的数据都是热点数据。

```java
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}
```

以下是使用 LinkedHashMap 实现的一个 LRU 缓存：

- 设定最大缓存空间 MAX_ENTRIES 为 3；
- 使用 LinkedHashMap 的构造函数将 accessOrder 设置为 true，开启 LRU 顺序；
- 覆盖 removeEldestEntry() 方法实现，在节点多于 MAX_ENTRIES 就会将最近最久未使用的数据移除。

```java
class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private static final int MAX_ENTRIES = 3;

    //OVERRIDE 该方法，返回true时，将移除链表头部的节点，即如果设置accessOrder为true，则
    //会头部节点即为最近最久未使用(访问),即lru
    protected boolean removeEldestEntry(Map.Entry eldest) {
        return size() > MAX_ENTRIES;
    }

    LRUCache() {
        //true为将accessOrder设置为true，开启lru，访问完某个节点后会将该节点放到链表尾部
        super(MAX_ENTRIES, 0.75f, true);
    }
}
public static void main(String[] args) {
    LRUCache<Integer, String> cache = new LRUCache<>();
    cache.put(1, "a");
    cache.put(2, "b");
    cache.put(3, "c");
    cache.get(1);
    cache.put(4, "d");
    System.out.println(cache.keySet());
}
[3, 1, 4]
```



#### WeakHashMap

##### 存储结构

WeakHashMap 的 Entry 继承自 WeakReference，被 WeakReference 关联的对象在下一次垃圾回收时会被回收。

WeakHashMap 主要用来实现缓存，通过使用 WeakHashMap 来引用缓存对象，由 JVM 对这部分缓存进行回收。

```
private static class Entry<K,V> extends WeakReference<Object> implements Map.Entry<K,V>
```

##### ConcurrentCache

Tomcat 中的 ConcurrentCache 使用了 WeakHashMap 来实现缓存功能。

ConcurrentCache 采取的是分代缓存：

- 经常使用的对象放入 eden 中，eden 使用 ConcurrentHashMap 实现，不用担心会被回收（伊甸园）；
- 不常用的对象放入 longterm，longterm 使用 WeakHashMap 实现，这些老对象会被垃圾收集器回收。
- 当调用 get() 方法时，会先从 eden 区获取，如果没有找到的话再到 longterm 获取，当从 longterm 获取到就把对象放入 eden 中，从而保证经常被访问的节点不容易被回收。
- 当调用 put() 方法时，如果 eden 的大小超过了 size，那么就将 eden 中的所有对象都放入 longterm 中，利用虚拟机回收掉一部分不经常使用的对象。

```java
public final class ConcurrentCache<K, V> {

    private final int size;

    private final Map<K, V> eden;

    private final Map<K, V> longterm;

    public ConcurrentCache(int size) {
        this.size = size;
        this.eden = new ConcurrentHashMap<>(size);
        this.longterm = new WeakHashMap<>(size);
    }

    public V get(K k) {
        V v = this.eden.get(k);
        if (v == null) {
            v = this.longterm.get(k);
            if (v != null)
                this.eden.put(k, v);
        }
        return v;
    }

    public void put(K k, V v) {
        if (this.eden.size() >= size) {
            this.longterm.putAll(this.eden);
            this.eden.clear();
        }
        this.eden.put(k, v);
    }
}
```

