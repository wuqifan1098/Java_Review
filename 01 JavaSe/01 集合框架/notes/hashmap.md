# 面试题

## 1.HashMap怎么使用才是线程安全的（360企业安全）

为什么线程不安全

首先如果**多个线程同时使用put方法添加元素**，而且假设正好存在两个 put 的 key 发生了碰撞(根据 hash 值计算的 bucket 一样)，那么根据 HashMap 的实现，这两个 key 会添加到数组的同一个位置，**这样最终就会发生其中一个线程的 put 的数据被覆盖**。第二就是如果多个线程同时检测到元素个数超过数组大小* loadFactor ，这样就会发生**多个线程同时对 Node 数组进行扩容**，都在重新计算元素位置以及复制数据，但是最终只有一个线程扩容后的数组会赋给 table，也就是说**其他线程的都会丢失，并且各自线程 put 的数据也丢失**。

HashMap 在**并发执行 put 操作时会引起死循环**，导致 CPU 利用率接近100%。**因为多线程会导致 HashMap 的 Entry链表形成环形数据结构，一旦形成环形数据结构，Entry 的 next 节点永远不为空，就会在获取 Entry 时产生死循环。**

如何线程安全的使用 HashMap。这个无非就是以下三种方式：

- Hashtable，Hashtable通过**对整个表上锁**实现线程安全，因此效率比较低。
- ConcurrentHashMap，它使用**分段锁**来保证线程安全。
- SynchronizedMap

## 2.请设计一个线程安全的HashMap

HashMap的核心在于如何解决哈希冲突，主流思路有两种，

- 开地址法(Open addressing). 即所有元素在一个一维数组里，遇到冲突则按照一定规则向后跳，假设元素x原本在位置hash(x)%m（m表示数组长度），那么第i次冲突时位置变为Hi = [hash(x) + di] % m，其中di表示第i次冲突时人为加上去的偏移量。偏移量di一般有如下3种计算方法，

	1. 线性探测法(Linear Probing)。非常简单，发现位子已经被占了，则向后移动1位，即d_i = id
​   i=i, i=1,2,3,...
   该算法最大的优点在于计算速度快，对CPU高速缓存友好；其缺点也非常明显，假设3个元素x1，x2，x3的哈希值都相同，记为p, x1先来，查看位置p，是空，则x1被映射到位置p，x2后到达，查看位置p，发生第一次冲突，向后探测一下，即p+1，该位置为空，于是x2映射到p+1, 同理，计算x3的位置需要探测位置p, p+1, p+2，也就是说对于发生冲突的所有元素，在探测过程中会扎堆，导致效率低下，这种现象被称为clustering。

	1. 二次探测法(Quadratic Probing)。d_i=ai^2+bi+cdi=ai2+bi+c, 其中a,b,c为系数，d_idi
​​    是i的二次函数，所以称为二次探测法。该算法的性能介于线性探测和双哈希之间。

	1. 双哈希法(Double Hashing)。偏移量di由另一个哈希函数计算而来，设为hash2(x)，则di=(hash2(x) % (m-1) + 1) * i
   拉链法(Chaining)。开一个定长数组，每个格子指向一个桶(Bucket, 可以用单链表或双向链表表示)，对每个元素计算出哈希并取模，找到对应的桶，并插入该桶。发生冲突的元素会处于同一个桶中。

- 拉链法(Chaining)。开一个定长数组，每个格子指向一个桶(Bucket, 可以用单链表或双向链表表示)，对每个元素计算出哈希并取模，找到对应的桶，并插入该桶。发生冲突的元素会处于同一个桶中。

如何将基于拉链法的HashMap改造为线程安全的呢？有以下几个思路，

- 将所有public方法都**加上synchronized**. 相当于设置了一把全局锁，所有操作都需要先获取锁，java.util.HashTable就是这么做的，性能很低
- 由于每个桶在逻辑上是相互独立的，将每个桶都加一把锁，如果两个线程各自访问不同的桶，就不需要争抢同一把锁了。这个方案的并发性比单个全局锁的性能要好，**不过锁的个数太多，也有很大的开销。**
- 锁分离(Lock Stripping)技术。第2个方法把锁的压力分散到了多个桶，理论上是可行的的，但是假设有1万个桶，就要新建1万个ReentrantLock实例，开销很大。可以**将所有的桶均匀的划分为16个部分，每一部分成为一个段(Segment)，每个段上有一把锁，这样锁的数量就降低到了16个**。JDK 7里的java.util.concurrent.ConcurrentHashMap就是这个思路。
- 在JDK8里，ConcurrentHashMap的实现又有了很大变化，它在锁分离的基础上，大量利用了了CAS指令。并且底层存储有一个小优化，当链表长度太长（默认超过8）时，链表就转换为红黑树。链表太长时，增删查改的效率比较低，改为红黑树可以提高性能。JDK8里的ConcurrentHashMap有6000多行代码，JDK7才1500多行。

## 3.为什么HashMap的容量一定要是2的整数次幂

length为2的整数次幂的话，h&(length-1)就相当于对length取模，这样便**保证了散列的均匀**，同时也提升了效率。

有助于HashMap中的元素**存放地更均匀**，**降低了hash碰撞的概率**，**提高了查找的效率和空间利用率**。

## 4.HashMap存储对象时，为什么要同时重写hashCode和equals

会出现我们认为的同一对象，却**因为hash值不同而导致hashmap中存了两个对象**，从而才需要进行hashcode方法的覆盖。

即使两个对象的 equals 方法时一致的，但是两个对象的hashCode 返回的 int 值不一致，导致put操作和get操作时，最终**存和取的数组下标不同，取出来就是null** 。为了保证同一个对象，保证在equals相同的情况下hashcode值必定相同。

## 5.JDK1.7 HashMap扩容时可能存在什么问题

当重新调整HashMap大小的时候，确实存在条件竞争，因为如果两个线程都发现HashMap需要重新调整大小了，它们会同时试着调整大小。在调整大小的过程中，由于采用**头插法，存储在LinkedList中的元素的次序会反过来，容易发生死循环**。

## 6.HashMap如何进行扩容

- 创建一个新的Entry空数组，**长度是原数组的2倍。**

- 遍历原Entry数组，**每个节点重新计算哈希值到新数组**，原来 table[i] 中的链表的所有节点，分拆到新的数组的 newTable[i] 和 newTable[i + oldLength] 位置上。

## 7.为啥hashmap不直接采用红黑树，而是当大于8个的时候才转换红黑树？

**当个数不多的时候，直接链表遍历更方便**，实现起来也简单。而**红黑树的实现要复杂的多**。	

红黑树**需要进行左旋，右旋操作，而单链表不需要，**

以下都是单链表与红黑树结构对比。

如果元素小于8个，**查询成本高，新增成本低**

如果元素大于8个，**查询成本低，新增成本高**

## 8.为什么hash冲突使用红黑树而不是AVL树？

红黑树和AVL树都是常用的平衡二叉搜索树，支持插入，删除和查找。

但是AVL树牺牲了插入删除性能，更加严格平衡，**可以提供更快的查找效果。**对于查找密集型，使用AVL树。

AVL的旋转比红黑树更难实现和调试。

红黑树**牺牲了一些查找性能 但其本身并不是完全平衡的二叉树**。因此**插入删除操作效率略高于AVL树**

对于插入密集型，使用红黑树。

## 9.说一下 HashMap 的存储逻辑（put() 函数）（华数）

- 如果定位到的数组位置没有元素 就直接插入。
- 如果定位到的数组位置有元素就和要插入的key比较，如果key相同就直接覆盖，如果key不相同，就判断p是否是一个树节点，如果是就调用e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value)将元素添加进入。如果不是就**遍历链表插入**(插入的是链表尾部)。

基本思路

1. 对key的hashCode()进行hash后计算数组下标index;

2. 如果当前数组table为null，进行resize()初始化；

3. 如果没碰撞直接放到对应下标的位置上；

4. 如果碰撞了，且节点已经存在，就替换掉 value；

5. 如果碰撞后**发现为树结构，挂载到树上。**

6. 如果碰撞后为链表，添加到链表尾，并判断链表如果过长(大于等于TREEIFY_THRESHOLD，默认8)，就把链表转换成树结构；

7. 数据 put 后，如果**数据量超过threshold，就要resize**。

   作者：冰洋链接：https://juejin.im/post/5aa47ef2f265da23a0492cc8

## 10.HashMap 存储元素时 key 完全一样该怎么处理？



## 11.为什么 JDK 1.7 是头插法，JDK 1.8 是尾插法？（美团）

因为JDK1.7是用单链表进行的纵向延伸，当采用头插法时会**容易出现逆序且环形链表死循环问题**。但是在JDK1.8之后是因为**加入了红黑树**使用尾插法，能够**避免出现逆序且链表死循环的问题**。

## 12.为什么 HashMap 中 String、Integer 这样的包装类适合作为 key 键

String、Integer都是**不可变类**，重写了equals()和hashcode()方法。保证了**Hash值的不可更改性&计算准确性**。
有效的**减少了Hash碰撞的概率。**

## 13.链表长度为多少的时候红黑树转为链表 （华数）

```java
// 当在扩容（resize（））时（HashMap的数据存储位置会重新计算），在重新计算存储位置后，当原有的红黑树内数量 <= 6时，则将 红黑树转换成链表
static final int UNTREEIFY_THRESHOLD = 6;
```

**HashMap 中关于红黑树的三个关键参数：**

|              **TREEIFY_THRESHOLD**              |                   **UNTREEIFY_THRESHOLD**                    |                   **MIN_TREEIFY_CAPACITY**                   |
| :---------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|              **一个桶的树化阈值**               |                   **一个树的链表还原阈值**                   |                  **哈希表的最小树形化容量**                  |
|     static final int TREEIFY_THRESHOLD = 8;     |          static final int UNTREEIFY_THRESHOLD = 6;           |         static final int MIN_TREEIFY_CAPACITY = 64;          |
| 链表转树的阀值，如果链表长度大于8时就转化为树。 | **当且仅当扩容时**，桶中元素个数小于这个值，就会把树形的桶元素还原（切分）为链表结构 | 当哈希表中的容量大于这个值时，表中的桶才能进行树形化，否则桶内元素太多时会扩容，而不是树形化。为了避免进行扩容、树形化选择的冲突，这个值不能小于 4 * TREEIFY_THRESHOLD |

## 14.JDK1.8对HashMap的优化（字节）

- 由 数组+链表 改为 **数组+链表+红黑树**

  链表过长会影响hashmap的性能

  在链表元素数量超过8时改为红黑树，少于6时改为链表，中间7不改是避免频繁转换降低性能。
  相对于链表，改为红黑树后碰撞元素越多查询效率越高。链表O(n)，红黑树O(logn)。

- 优化了高位运算的hash算法：h^(h>>>16)

  **将hashcode无符号右移16位，让高16位和低16位进行异或。**

- 扩容后，**元素要么是在原位置，要么是在原位置再移动2次幂的位置，且链表顺序不变**

  不需要rehash，**头插法改为尾插法**。因为尾插法数组链表不会倒置，多线程下不会出现死循环。

- **用Node取代了Entry**

## 15.ConcurrentHashMap和LinkedHashMap有什么区别？ （字节）

LinkedHashMap继承自HashMap，但是是有序的

hashmap写入慢，读取快；linkedhashmap是个有序链表读取慢，写入快；treeMap可以帮你自动做好升序排列。

ConcurrentHashMap，线程安全，读写还快 而且锁分离

## 16.为什么HashMap集合在储存数据的时候要使用哈希算法？

众所周知，HaspMap中存储的数据都是**不可重复**的并且是**无序**的，那么我们在存储一个新的数据的时候就需要判断这个数据原来是否存在，这个时候就需要**通过java中的equals方法来判断两个对象的实例是否相等，如果相等，那么就是重复数据。**但是，如果每增加一个元素就检查一次，那么当元素比较多时，添加新元素的时候就需要用equals比较很多次，这个时候存储的效率就非常低。

> 例如：如果一个HashMap集合中已经有10000个元素，这个时候添加一个新元素的时候就需要用equals比较10000次，再添加一个元素，就需要用equals比较10001次，存储的元素越多，效率就会越低。

于是，Java便采用哈希算法来**提高效率**。当向集合中添加新的元素的时候，**先将对象通过哈希算法(hashCode方法)计算得到哈希值（正整数），然后将哈希值和集合(数组)长度进行&运算，得到该对象在该数组存放位置的索引值**。如果这个位置上没有元素，就可以直接存储在这个位置上，如果这个位置上已经有元素了，就调用它的equals方法与新元素进行比较，相同的话就覆盖，不相同的话就发生碰撞，形成链表。
(哈希算法：https://www.cnblogs.com/xiohao/p/4389672.html)

## 17.为什么ConcurrentHashMap中的链表转红黑树的阀值是8？ 

理想情况下使用随机的哈希码，容器中节点分布在hash桶中的频率遵循泊松分布(具体可以查看http://en.wikipedia.org/wiki/Poisson_distribution)，按照泊松分布的计算公式计算出了桶中元素个数和概率的对照表，可以看到链表中元素个数为8时的概率已经非常小，再多的就更少了，所以原作者在选择链表元素个数时选择了8，是**根据概率统计而选择的。**

当hashCode离散性很好的时候，树型bin用到的概率非常小，因为数据均匀分布在每个bin中，几乎不会有bin中链表长度会达到阈值。但是在随机hashCode下，离散性可能会变差，然而JDK又不能阻止用户实现这种不好的hash算法，因此就可能导致不均匀的数据分布。不过理想情况下随机hashCode算法下所有bin中节点的分布频率会遵循泊松分布，我们可以看到，一个bin中链表长度达到8个元素的概率为0.00000006，几乎是不可能事件。

## 18. 位运算的好处

位运算(&)效率要比代替取模运算(%)高很多，主要原因是**位运算直接对内存数据进行操作，不需要转成十进制，因此处理速度非常快。**

## 19. hashMap什么时候扩容

- 判断当前个数是否大于等于阈值
- 当前存放是否发生哈希碰撞

因为上面这两个条件，所以存在下面这些情况

（1）、就是hashmap在存值的时候（默认大小为16，负载因子0.75，阈值12），可能达到最后存满16个值的时候，再存入第17个值才会发生扩容现象，**因为前16个值，每个值在底层数组中分别占据一个位置，并没有发生hash碰撞**。

（2）、当然也有可能存储更多值（超多16个值，最多可以存26个值）都还没有扩容。原理：前11个值全部hash碰撞，存到数组的同一个位置（这时元素个数小于阈值12，不会扩容），后面所有存入的15个值全部分散到数组剩下的15个位置（这时元素个数大于等于阈值，但是每次存入的元素并没有发生hash碰撞，所以不会扩容），前面11+15=26，所以在存入第27个值的时候才同时满足上面两个条件，这时候才会发生扩容现象。

## 20. 环形链表产生死循环的详解



## 21. hashMap使用的hash方法是什么(字节)

**1.7中的计算hash值的算法和1.8的实现是不一样的**，而hash值又关系到我们put新元素的位置、get查找元素、remove删

除元素的时候去通过indexFor查找下标。

HashMap中定位到桶的位置，根据**Key的hash值与数组的长度取模**来计算的。

取模可以改为：**hashCode & （length - 1）** 

看下JDK8中的hash 算法：

```java
 static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

首先是取key的hashCode算法，然后对低16位进行右移运算和异或运算。 

## 22. hashMap如何解决哈希碰撞

1.7采用的是链地址法，会使HashMap的性能从O(1)降到O(N)。

1.8使用红黑树，将最差性能从O(N)提高到O(logn)

## 23.如何减少hash碰撞

1.7 经过了四次Hash扰动

1.8 高16位和低16位异或，只做一次16位右移 异或

# 1.底层数据结构分析

## JDK1.8之前

JDK1.8 之前 HashMap 底层是 **数组和链表** 结合在一起使用也就是 链表散列。HashMap 通过 key 的 hashCode 经过扰动函数处理过后得到 hash 值，然后通过 (n - 1) & hash 判断当前元素存放的位置（这里的 n 指的是数组的长度），如果当前位置存在元素的话，就判断该元素与要存入的元素的 hash 值以及 key 是否相同，如果相同的话，直接覆盖，不相同就通过**拉链法**解决冲突。

所谓扰动函数指的就是 HashMap 的 hash 方法。使用 hash 方法也就是扰动函数是为了防止一些实现比较差的 hashCode() 方法 换句话说使用扰动函数之后可以减少碰撞。

JDK 1.8 HashMap 的 hash 方法源码:

JDK 1.8 的 hash方法 相比于 JDK 1.7 hash 方法更加简化，但是原理不变。

```java
static final int hash(Object key) {
  int h;
  // key.hashCode()：返回散列值也就是hashcode
  // ^ ：按位异或
  // >>>:无符号右移，忽略符号位，空位都以0补齐
  return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

对比一下 JDK1.7的 HashMap 的 hash 方法源码。

```java
static int hash(int h) {
// This function ensures that hashCodes that differ only by
// constant multiples at each bit position have a bounded
// number of collisions (approximately 8 at default load factor).

h ^= (h >>> 20) ^ (h >>> 12);
return h ^ (h >>> 7) ^ (h >>> 4);
}
```

相比于 JDK1.8 的 hash 方法 ，JDK 1.7 的 hash 方法的性能会稍差一点点，因为毕竟扰动了 4 次。

## 类的属性：

```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {
// 序列号
private static final long serialVersionUID = 362498820763181265L;    
// 默认的初始容量是16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;   
// 最大容量
static final int MAXIMUM_CAPACITY = 1 << 30; 
// 默认的填充因子
static final float DEFAULT_LOAD_FACTOR = 0.75f;
// 当桶(bucket)上的结点数大于这个值时会转成红黑树
static final int TREEIFY_THRESHOLD = 8; 
// 当桶(bucket)上的结点数小于这个值时树转链表
static final int UNTREEIFY_THRESHOLD = 6;
// 桶中结构转化为红黑树对应的table的最小大小
static final int MIN_TREEIFY_CAPACITY = 64;
// 存储元素的数组，总是2的幂次倍
transient Node<k,v>[] table; 
// 存放具体元素的集
transient Set<map.entry<k,v>> entrySet;
// 存放元素的个数，注意这个不等于数组的长度。
transient int size;
// 每次扩容和更改map结构的计数器
transient int modCount;   
// 临界值 当实际大小(容量*填充因子)超过临界值时，会进行扩容
int threshold;
// 加载因子
final float loadFactor;
}
```

内部存储结构使用的Node节点类源码:

```java
// 继承自 Map.Entry<K,V>
static class Node<K,V> implements Map.Entry<K,V> {
   final int hash;// 哈希值，存放元素到hashmap中时用来与其他元素hash值比较
   final K key;//键
   V value;//值
   // 指向下一个节点
   Node<K,V> next;
   Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }
    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }
    // 重写hashCode()方法
    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }
    // 重写 equals() 方法
    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

Node 是 HashMap 的一个内部类，**实现了 Map.Entry 接口，本质是就是一个映射（键值对）**。上图中的每个黑色圆点就是一个Node对象。

树节点类源码:

```java
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // 父
    TreeNode<K,V> left;    // 左
    TreeNode<K,V> right;   // 右
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;           // 判断颜色
    TreeNode(int hash, K key, V val, Node<K,V> next) {
        super(hash, key, val, next);
    }
    // 返回根节点
    final TreeNode<K,V> root() {
        for (TreeNode<K,V> r = this, p;;) {
            if ((p = r.parent) == null)
                return r;
            r = p;
   }
```

# 2.HashMap源码分析

```java
// 默认构造函数。
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all   other fields defaulted
 }
 
 // 包含另一个“Map”的构造函数
 public HashMap(Map<? extends K, ? extends V> m) {
     this.loadFactor = DEFAULT_LOAD_FACTOR;
     putMapEntries(m, false);//下面会分析到这个方法
 }
 
 // 指定“容量大小”的构造函数
 public HashMap(int initialCapacity) {
     this(initialCapacity, DEFAULT_LOAD_FACTOR);
 }
 
 // 指定“容量大小”和“加载因子”的构造函数
 public HashMap(int initialCapacity, float loadFactor) {
     if (initialCapacity < 0)
         throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
     if (initialCapacity > MAXIMUM_CAPACITY)
         initialCapacity = MAXIMUM_CAPACITY;
     if (loadFactor <= 0 || Float.isNaN(loadFactor))
         throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
     this.loadFactor = loadFactor;
     this.threshold = tableSizeFor(initialCapacity);
 }
```

# 3.重要方法

### put方法

JDK1.8 HashMap 的 put 方法源码如下:

```java
public V put(K key, V value) {
// 对key的hashCode()做hash
return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
           boolean evict) {
Node<K,V>[] tab; Node<K,V> p; int n, i;
// 步骤①：tab为空则创建
if ((tab = table) == null || (n = tab.length) == 0)
    n = (tab = resize()).length;
// 步骤②：计算index，并对null做处理 
if ((p = tab[i = (n - 1) & hash]) == null) 
    tab[i] = newNode(hash, key, value, null);
else {
    Node<K,V> e; K k;
    // 步骤③：节点key存在，直接覆盖value
    if (p.hash == hash &&
        ((k = p.key) == key || (key != null && key.equals(k))))
        e = p;
    // 步骤④：判断该链为红黑树
    else if (p instanceof TreeNode)
        e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
    // 步骤⑤：该链为链表
    else {
        for (int binCount = 0; ; ++binCount) {
            if ((e = p.next) == null) {
                p.next = newNode(hash, key,value,null);
                 //链表长度大于8转换为红黑树进行处理
                if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st  
                    treeifyBin(tab, hash);
                break;
            }
             // key已经存在直接覆盖value
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k)))) 
                       break;
            p = e;
        }
    }
    
    if (e != null) { // existing mapping for key
        V oldValue = e.value;
        if (!onlyIfAbsent || oldValue == null)
            e.value = value;
        afterNodeAccess(e);
        return oldValue;
    }
}
	++modCount;
// 步骤⑥：超过最大容量 就扩容
if (++size > threshold)
    resize();
afterNodeInsertion(evict);
return null;
}
```

put()图解

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/hashmap-put.png)

- 如果定位到的数组位置没有元素 就直接插入。
- 如果定位到的数组位置有元素就和要插入的key比较，如果key相同就直接覆盖，如果key不相同，就判断p是否是一个树节点，如果是就调用e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value)将元素添加进入。如果不是就遍历链表插入(插入的是链表尾部)。

#### 拉链法原理

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

应该注意到链表的插入是**以头插法方式**进行的，例如上面的 <K3,V3> 不是插在 <K2,V2> 后面，而是插入在链表头部。

查找需要分成两步进行：

- 计算键值对所在的桶；
- 在链表上顺序查找，时间复杂度显然和链表的长度成正比。

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/%E6%8B%89%E9%93%BE%E6%B3%95%E5%8E%9F%E7%90%86.png)

### get方法

```java
    /**
     * 传入key值返回value值
     * 若不存在则返回null
     */ 
     public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
    // 具体实现get-value 方法
     final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        // 若表不为空且长度不为0 且指定位置hash表内有元素
        if ((tab = table) != null && (n = tab.length) > 0 && (first = tab[(n - 1) & hash]) != null) {
        // 若key值相同 返回该节点
            if (first.hash == hash && // always check first node((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            // 若该节点有下一个节点 且该节点属于红黑树节点
            // 按照红黑树高效查找
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                // 直到下一个节点为空 循环 对比key值
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        // 若数组不存在hash算法的值 返回null
        return null;
    }
```

### resize方法

进行扩容，**会伴随着一次重新hash分配，并且会遍历hash表中所有的元素，是非常耗时的**。在编写程序中，要尽量避免resize。

源码：

```java
final Node<K,V>[] resize() {
Node<K,V>[] oldTab = table;
int oldCap = (oldTab == null) ? 0 : oldTab.length;
int oldThr = threshold;
int newCap, newThr = 0;
if (oldCap > 0) {
    // 超过最大值就不再扩充了，就只好随你碰撞去吧
    if (oldCap >= MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return oldTab;
    }
    // 没超过最大值，就扩充为原来的2倍
    else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
        newThr = oldThr << 1; // double threshold
}
else if (oldThr > 0) // initial capacity was placed in threshold
    newCap = oldThr;
else { 
    // signifies using defaults
    newCap = DEFAULT_INITIAL_CAPACITY;
    newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
}
// 计算新的resize上限
if (newThr == 0) {
    float ft = (float)newCap * loadFactor;
    newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ? (int)ft : Integer.MAX_VALUE);
}
threshold = newThr;
@SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
table = newTab;
if (oldTab != null) {
    // 把每个bucket都移动到新的buckets中
    for (int j = 0; j < oldCap; ++j) {
        Node<K,V> e;
        if ((e = oldTab[j]) != null) {
            oldTab[j] = null;
            if (e.next == null)
                newTab[e.hash & (newCap - 1)] = e;
            else if (e instanceof TreeNode)
                ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
            else { 
                Node<K,V> loHead = null, loTail = null;
                Node<K,V> hiHead = null, hiTail = null;
                Node<K,V> next;
                do {
                    next = e.next;
                    // 原索引
                    if ((e.hash & oldCap) == 0) {
                        if (loTail == null)
                            loHead = e;
                        else
                            loTail.next = e;
                        loTail = e;
                    }
                    // 原索引+oldCap
                    else {
                        if (hiTail == null)
                            hiHead = e;
                        else
                            hiTail.next = e;
                        hiTail = e;
                    }
                } while ((e = next) != null);
                // 原索引放到bucket里
                if (loTail != null) {
                    loTail.next = null;
                    newTab[j] = loHead;
                }
                // 原索引+oldCap放到bucket里
                if (hiTail != null) {
                    hiTail.next = null;
                    newTab[j + oldCap] = hiHead;
                }
            }
        }
    }
}
return newTab;
}
```

#### 扩容机制

```java
void resize(int newCapacity) {   //传入新的容量
Entry[] oldTable = table;    //引用扩容前的Entry数组
int oldCapacity = oldTable.length;         
if (oldCapacity == MAXIMUM_CAPACITY) {  //扩容前的数组大小如果已经达到最大(2^30)了
    threshold = Integer.MAX_VALUE; //修改阈值为int的最大值(2^31-1)，这样以后就不会扩容了
    return;
}
 
Entry[] newTable = new Entry[newCapacity];  //初始化一个新的Entry数组
transfer(newTable);                         //！！将数据转移到新的Entry数组里
table = newTable;                           //HashMap的table属性引用新的Entry数组
threshold = (int)(newCapacity * loadFactor);//修改阈值
}
```

这里就是使用一个容量更大的数组来代替已有的容量小的数组，transfer() 方法将原有 Entry 数组的元素拷贝到新的 Entry 数组里。

```java
void transfer(Entry[] newTable) {
Entry[] src = table;                   //src引用了旧的Entry数组
int newCapacity = newTable.length;
for (int j = 0; j < src.length; j++) { //遍历旧的Entry数组
    Entry<K,V> e = src[j];             //取得旧Entry数组的每个元素
    if (e != null) {
        src[j] = null;//释放旧Entry数组的对象引用（for循环后，旧的Entry数组不再引用任何对象）
        do {
            Entry<K,V> next = e.next;
            int i = indexFor(e.hash, newCapacity); //！！重新计算每个元素在数组中的位置
            e.next = newTable[i]; //标记[1]
            newTable[i] = e;      //将元素放在数组上
            e = next;             //访问下一个Entry链上的元素
        } while (e != null);
    }
}
}
```

#### rehash过程

单线程下的ReHash

- 用key对数组长度取余。

- 构成了 old hash表

- Hash表resize，然后所有的<key,value> 重新rehash

当hash表中的负载因子达到负载极限的时候，hash表会自动成倍的增加容量（桶的数量），**并将原有的对象重新的分配并加入新的桶内**，这称为rehash。

### 并发下的Rehash

当重新调整HashMap大小的时候，确实存在条件竞争，因为如果两个线程都发现HashMap需要重新调整大小了，它们会同时试着调整大小。**在调整大小的过程中，存储在LinkedList中的元素的次序会反过来，因为移动到新的bucket位置的时候，HashMap并不会将元素放在LinkedList的尾部，而是放在头部，这是为了避免尾部遍历(tail traversing)。**如果条件竞争发生了，那么就死循环了。这个时候，你可以质问面试官，为什么这么奇怪，要在多线程的环境下使用HashMap呢？

作者：changzhiqiang_ 
来源：CSDN 
原文：https://blog.csdn.net/crpxnmmafq/article/details/75331318 
版权声明：本文为博主原创文章，转载请附上博文链接！

https://www.jianshu.com/p/13c650a25ed3

## hash()函数分析（JDK 1.8）

```java
static final int hash(Object key) {
int h;
return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

实际步骤如下

```java
static final int hash(Object key) {
if(key == null)
return 0;
int h = key.hashCode();
int temp = h >>> 16;
int newHash = h ^ temp;
return newHash;
}
```

代码过程的直接翻译就是：**如果Key值为null，返回0；如果Key值不为空，返回原hash值和原hash值无符号右移16位的值按位异或的结果。**

**这里的Hash算法本质上就是三步：取key的hashCode值、高位运算、取模运算。**

对于任意给定的对象，只要它的hashCode()返回值相同，那么程序调用方法一所计算得到的Hash码值总是相同的。我们首先想到的就是把hash值对数组长度取模运算，这样一来，元素的分布相对来说是比较均匀的。但是，模运算的消耗还是比较大的，在HashMap中是这样做的：调用方法二来计算该对象应该保存在table数组的哪个索引处。这个方法非常巧妙，它通过h & (table.length -1)来得到该对象的保存位，而HashMap底层数组的长度总是2的n次方，这是HashMap在速度上的优化。**当length总是2的n次方时，h& (length-1)运算等价于对length取模，也就是h%length，但是&比%具有更高的效率。**
    在JDK1.8的实现中，优化了高位运算的算法，通过hashCode()的高16位异或低16位实现的：(h = k.hashCode()) ^ (h >>> 16)，主要是从速度、功效、质量来考虑的，这么做可以在数组table的length比较小的时候，也能保证考虑到高低bit都参与到Hash的计算中，同时不会有太大的开销。

参考：

https://www.cnblogs.com/zyrblog/p/9881958.html

https://www.zhihu.com/question/20733617



# 与 HashTable 的比较

- HashTable **使用 synchronized** 来进行同步。
- HashMap 可以插入键为 null 的 Entry。
- HashMap 的迭代器是 fail-fast 迭代器。
- HashMap 不能保证随着时间的推移 Map 中的元素次序是不变的。

# 总结

`HashMap`是基于`Map`接口实现的一种键-值对`<key,value>`的存储结构，允许`null`值，同时**非有序，非同步(即线程不安全)**。`HashMap`的底层实现是数组 + 链表 + 红黑树（JDK1.8增加了红黑树部分）。

`HashMap`定位元素位置是通过**键`key`经过扰动函数扰动后得到`hash`值**，然后再**通过`hash & (length - 1)`代替取模的方式进行元素定位的。**

`HashMap`是使用**链地址法**解决`hash`冲突的，当有冲突元素放进来时，会将此元素插入至此位置链表的最后一位，形成单链表。当存在位置的链表长度 大于等于 8 时，`HashMap`会将链表 转变为 红黑树，以此提高查找效率。

`HashMap`的容量是2的n次方，**有利于提高计算元素存放位置时的效率，也降低了`hash`冲突的几率**。因此，我们使用`HashMap`存储大量数据的时候，最好先预先指定容器的大小为2的n次方，即使我们不指定为2的n次方，`HashMap`也会把容器的大小设置成最接近设置数的2的n次方，如，设置`HashMap`的大小为 7 ，则`HashMap`会将容器大小设置成最接近7的一个2的n次方数，此值为 8 。

`HashMap`的负载因子表示哈希表空间的使用程度（或者说是哈希表空间的利用率）。**当负载因子越大，则`HashMap`的装载程度就越高。也就是能容纳更多的元素，元素多了，发生`hash`碰撞的几率就会加大，从而链表就会拉长，此时的查询效率就会降低。当负载因子越小，则链表中的数据量就越稀疏，此时会对空间造成浪费，但是此时查询效率高。**

`HashMap`不是线程安全的，`Hashtable`则是线程安全的。但`Hashtable`是一个遗留容器，如果我们不需要线程同步，则建议使用`HashMap`，如果需要线程同步，则建议使用`ConcurrentHashMap`。

在多线程下操作`HashMap`，由于存在扩容机制，当`HashMap`调用`resize()`进行自动扩容时，可能会导致死循环的发生。

我们在使用`HashMap`时，**最好选择不可变对象作为`key`。例如`String`，`Integer`等不可变类型作为`key`是非常明智的。**

