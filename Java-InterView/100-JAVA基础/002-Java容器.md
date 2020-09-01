# 一、概述
- 容器主要包括 Collection 和 Map 两种，Collection 存储着对象的集合，而 Map 存储着键值对（两个对象）的映射表。
![image](0084EDA09EF8441E83B9888C8ECB5DB9)
![image](3ECEF5BDB9C74AB3A32059C54B144D87)
## 1. Set
### TreeSet
- 基于红黑树实现，支持有序性操作，例如根据一个范围查找元素的操作。但是查找效率不如 HashSet，HashSet 查找的时间复杂度为 O(1)，TreeSet 则为 O(logN)。

### HashSet
- 基于哈希表实现，支持快速查找，但不支持有序性操作。并且失去了元素的插入顺序信息，也就是说使用 Iterator 遍历 HashSet 得到的结果是不确定的。

### LinkedHashSet
- 具有 HashSet 的查找效率，并且内部使用双向链表维护元素的插入顺序。

## 2.List
### ArrayList
- 基于动态数组实现，支持随机访问。
### Vector
- 和 ArrayList 类似，但它是线程安全的。
### LinkedList
- 基于双向链表实现，只能顺序访问，但是可以快速地在链表中间插入和删除元素。不仅如此，LinkedList 还可以用作栈、队列和双向队列。
## 3.Map
### TreeMap
- 基于红黑树实现。

### HashMap
- 基于哈希表实现。

### HashTable
- 和 HashMap 类似，但它是线程安全的，这意味着同一时刻多个线程同时写入 HashTable不会导致数据不一致。它是遗留类，不应该去使用它，而是使用 **ConcurrentHashMap** 来支持线程安全，ConcurrentHashMap 的效率会更高，因为 ConcurrentHashMap 引入了**分段锁**。

### LinkedHashMap
- 使用双向链表来维护元素的顺序，顺序为插入顺序或者最近最少使用（LRU）顺序。

# 二、源码分析
## ArrayList
### 1.概览
> 基于什么？ 优点是什么？默认长度？
- 因为 ArrayList是基于数组实现的，所以支持快速随机访问。RandomAccess 接口标识着该类支持快速随机访问，数组的默认大小为 10。

### 2.扩容
> 扩容长度？ 扩容引发的问题？如何避免、或者优化？
- 添加元素时使用ensureCapacityInternal()方法来保证容量足够，如果不够时，需要使用 grow() 方法进行扩容，新容量的大小为 oldCapacity + (oldCapacity >> 1)，也就是旧容量的 1.5 倍。
- 扩容操作需要调用 Arrays.copyOf() 把原数组整个复制到新数组中，这个操作代价很高，因此最好在创建 ArrayList 对象时就指定大概的容量大小，减少扩容操作的次数。

### 3.删除
> 需要注意的点？
- 需要调用 System.arraycopy() 将 index+1 后面的元素都复制到 index 位置上，该操作的时间复杂度为 O(N)，可以看到 ArrayList 删除元素的代价是非常高的。

### 4. 序列化
> 是怎样序列化的？
- ArrayList 基于数组实现，并且具有动态扩容特性，因此保存元素的数组不一定都会被使用，那么就没必要全部进行序列化。
- 保存元素的数组 elementData 使用 transient 修饰，该关键字声明数组默认不会被序列化。

### 5.Fail-Fast（快速失败）
> java内部是怎样实现的？
- modCount 用来记录 ArrayList 结构发生变化的次数。结构发生变化是指添加或者删除至少一个元素的所有操作，或者是调整内部数组的大小，仅仅只是设置元素的值不算结构发生变化。
- **在进行序列化或者迭代等操作时，需要比较操作前后 modCount 是否改变，如果改变了需要抛出ConcurrentModificationException(并发修改异常)**
- 通过for循环遍历删除，会抛出数组角标越界异常
- 正确删除list元素的方法
```
Iterator<Integer> iterator = list.iterator();  
while(iterator.hasNext()){  
    int i = iterator.next();  
    if(i == 1){  
        iterator.remove();  //迭代器的删除方法，而不是list的删除方法
    }  
}
```

## Vector
- 与ArrayList基本类似，但是使用了 synchronized 进行同步，是线程安全的。

### 1.扩容
- Vector 的构造函数可以传入capacityIncrement参数，它的作用是在扩容时使容量 capacity 增长 capacityIncrement。如果这个参数的值小于等于0，扩容时每次都令 capacity 为原来的两倍。
- 调用没有 capacityIncrement的构造函数时，capacityIncrement 值被设置为 0，也就是说默认情况下 Vector 每次扩容时容量都会翻倍。
### 3. 与 ArrayList 的比较
- Vector 是同步的，因此开销就比 ArrayList 要大，访问速度更慢。最好使用 ArrayList 而不是Vector，因为同步操作完全可以由程序员自己来控制；
- Vector 每次扩容请求其大小的 2倍（也可以通过构造函数设置增长的容量），而 ArrayList 是 1.5 倍。

### 4.替代方案 Collections.synchronizedList 与 CopyOnWriteArrayList

#### synchronizedList
- add方法：通过关键字synchronized同步
```
public void add(int index, E element) {
    synchronized (mutex) {list.add(index, element);}
}
```
- get方法：synchronized
```
public V get(Object key) {
   synchronized (mutex) {return m.get(key);}
}

```
#### CopyOnWriteArrayList
- 写操作在一个复制的数组上进行，读操作还是在原始数组中进行，读写分离，互不影响。
- 写操作需要加锁，防止并发写入时导致写入数据丢失。
- 写操作结束之后需要把原始数组指向新的复制数组。
- CopyOnWriteArrayList在写操作的同时允许读操作，大大提高了读操作的性能，因此很适合读多写少的应用场景。
- 内存占用：在写操作时需要复制一个新的数组，使得内存占用为原来的两倍左右；
- 数据不一致：读操作不能读取实时性的数据，因为部分写操作的数据还未同步到读数组中。
- **CopyOnWriteArrayList 不适合内存敏感以及对实时性要求很高的场景。**

#### 总结
- CopyOnWriteArrayList，发生修改时候做copy，新老版本分离，保证读的高性能，适用于以读为主，读操作远远大于写操作的场景中使用，比如缓存。
- Collections.synchronizedList则可以用在CopyOnWriteArrayList不适用，但是有需要同步列表的地方，读写操作都比较均匀的地方。

## LinkedList
### 1.概述
- 基于双向链表实现，使用 Node 存储链表节点信息。

### 2. 与 ArrayList 的比较
> 数组和链表的区别
- 数组支持随机访问，但插入删除的代价很高，需要移动大量元素；
- 链表不支持随机访问，但插入删除只需要改变指针。

## HashMap
###  1.概述
基于数组+链表实现，支持随机访问，默认容量为16，默认扩容是之前长度的2倍。且长度必须为2的N次幂。

内部包含了一个 Entry 类型的数组 table。Entry 存储着键值对。它包含了四个字段，从 next 字段我们可以看出 Entry 是一个链表。即数组中的每个位置被当成一个桶，一个桶存放一个链表。
```java
static class Entry<K,V> implements Map.Entry<K,V> {  
    final K key;  
    V value;  
    Entry<K,V> next;  
    final int hash;
}
```
1.8 是一个node类型的数组，字段与1.7相同。

HashMap最多只允许**一条记录的键为null**，允许**多条记录的值为null**。

HashMap非线程安全，即任一时刻可以有多个线程同时写HashMap，可能会导致数据的不一致。如果需要满足线程安全，可以用 **Collections的synchronizedMap**方法使HashMap具有线程安全的能力，或者使用**ConcurrentHashMap**。

### 2.扩容
- 默认扩容是之前长度的2倍
### 3. HashMap 1.7和1.8的不同点
> http://www.360doc.com/content/20/0304/13/10240337_896634470.shtml

> https://youzhixueyuan.com/the-underlying-structure-and-principle-of-hashmap.html

#### 1.实现方式更改
- JDK1.7用的是头插法，JDK1.8及之后使用的都是尾插法
- JDK1.7是用单链表进行的纵向延伸，当采用头插法就是能够提高插入的效率，但是在多线程的环境下会容易出现**逆序且环形链表死循环问题**。
``` 
在多线程的情况下，HasMap如果出现自动扩容，就需要rehash,重新计算hash表的所有元素，因为1.7采用的是头插法，那么就有可能出现链表死循环问题。

https://www.cnblogs.com/chanshuyi/p/java_collection_hashmap_17_infinite_loop.html
```
- 在JDK1.8之后是因为加入了红黑树使用尾插法，能够避免出现逆序且链表死循环的问题。
#### 2.扩容后数据存储位置的计算方式也不一样（未理解）
![image](011897B794B94582A23E086BBE4DB163)
#### 3.解决冲突的方式修改
- JDK1.7的时候使用的是数组+单链表的数据结构。
- JDK1.8及之后，使用的是数组链表+红黑树的数据结构（当链表的深度达到8的时候，也就是默认阈值，就会自动扩容把链表
转成红黑树的数据结构来把时间复杂度从O(n)变成 O(logN) 提高了效率.
#### HasMap JDK1.8 
- ![image](C35E106C2B9645BCACF65EF241F69EC4)

### 4.HashMap 的长度为什么是2的幂次方
为了能让 HashMap存取高效，尽量较少碰撞，也就是要尽量把数据分配均匀，用之前还要先做对数组的长度取模运算。

取余操作中如果除数是2的幂次则等价于与其除数减一的与操作 **（也就是说hash%length==hash&(length-1)的前提是 length 是2的 n 次方；）**

 并且采用二进制位操作,相对于取余操作能够提高运算效率，这就解释了 HashMap 的长度为什么是2的幂次方。

### 5.与 Hashtable 的比较
- Hashtable 使用 synchronized 来进行同步。
- HashMap 可以插入键为 null 的 Entry。
- HashMap 的迭代器是 fail-fast 迭代器。
- HashMap 不能保证随着时间的推移 Map 中的元素次序是不变的。

### 6.为什么映射中的key不可以是基础类型
HashMap是利用HashCode()来区别两个不同的对象。

而基础类型里面没有hashcode()方法，而且不能设置为null，所以不允许HashMap中定义基础类型的key

![image](3630B381B09F40C9A0B7886A96D71B35)

### 7.如何解决Hash冲突
![image](9B19BB735E264578A1F4FE536F56FC05)
## ConcurrentHashMap
### 1.存储结构 JDK1.7
![image](52F5856B1EC44322865A8ABF9A8744C7)
- ConcurrentHashMap 和 HashMap 实现上类似，最主要的差别是 ConcurrentHashMap 采用了分段锁（Segment），每个分段锁维护着几个桶（HashEntry），多个线程可以同时访问不同分段锁上的桶，从而使其并发度更高（并发度就是 Segment 的个数）。
- Segment 继承自 ReentrantLock
- 默认的并发级别为 16，也就是说默认创建 16 个 Segment。
- 底层采用数组+链表的存储结构

### 2. JDK1.8
> https://www.cnblogs.com/xiaowangbangzhu/p/10551500.html

- jdk1.8的实现已经抛弃了Segment分段锁机制，利用CAS+Synchronized来保证并发更新的安全。
- 底层采用数组+链表+红黑树的存储结构。
- 键值对节点使用的Node类型的数组。Node数组包含4个主要字段，key，value及key的hash值的数据结构。其中value和next都用**volatile**修饰，保证并发的可见性。

![image](56F0F373DCBE458E9554E0797939B545)

## LinkedHashMap
### 1.存储结构
- 继承自 HashMap，因此具有和 HashMap 一样的快速查找特性。
- 内部维护了一个双向链表，用来维护插入顺序或者 LRU (最近最少使用)顺序。
- accessOrder 决定了顺序，默认为 false，此时维护的是插入顺序。
- LinkedHashMap的数据存储和HashMap的结构一样采用(数组+单向链表)的形式，只是在每次节点Entry中增加了用于维护顺序的before和after变量维护了一个双向链表来保存LinkedHashMap的存储顺序，当调用迭代器的时候不再使用HashMap的的迭代器，而是自己写迭代器来遍历这个双向链表即可。
