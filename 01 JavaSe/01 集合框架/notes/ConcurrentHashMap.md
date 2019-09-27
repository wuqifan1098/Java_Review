# 面试题

## 1. 介绍一下ConcurrentHashMap（贝贝）

ConcurrentHashMap是一种线程安全且高效的HashMap。在JDK1.7中底层采用了**分段的数组+链表**的实现，把内部分成若干个小HashMap，每一把锁用于锁容器中的一部分数据，那么当多线程访问容器里不同数据端的数据时，线程间就不会存在锁竞争，提高了并发访问的效率。当对HashEntry数组的数据进行修改时，必须先获得对应的Segement锁。在JDK1.8中放弃了Segement的概念，直接用**Node数组+链表+红黑树**的结构实现，并发控制使用 **synchronized 和 CAS 来操作**，在 CAS 操作失败时使用内置锁 synchronized。

## 2. 为什么要分段

HashTable容器在竞争激烈的并发环境下表现出效率低下的原因，是因为所有访问HashTable的线程都**必须竞争同一把锁**，那假如容器里有多把锁，每一把锁用于锁容器其中一部分数据，那么当多线程访问容器里不同数据段的数据时，线程间就不会存在锁竞争，从而可以有效的提高并发访问效率，这就是ConcurrentHashMap所使用的锁分段技术，首先将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。

原文链接：https://blog.csdn.net/dianzijinglin/article/details/80998555

## 3. ConcurrentHashMap分段锁怎么实现的

Segment 通过**继承 ReentrantLock 来进行加锁**，所以每次需要加锁的操作锁住的是一个 segment，这样只要保证每个 Segment 是线程安全的，也就实现了全局的线程安全。

## 4. 扩容

**Java7**

segment 数组不能扩容，扩容是 segment 数组**某个位置内部的数组 HashEntry<K,V>[] 进行扩容，扩容后，容量为原来的 2 倍。**

首先，我们要回顾一下触发扩容的地方，put 的时候，如果判断该值的插入会导致该 segment 的元素个数超过阈值，那么先进行扩容，再插值，读者这个时候可以回去 put 方法看一眼。

该方法不需要考虑并发，因为到这里的时候，是持有该 segment 的独占锁的。

<https://www.javadoop.com/post/hashmap>

**Java8**

在JDK8里面，去掉了分段锁，将锁的级别控制在了更细粒度的table元素级别，也就是说只需要锁住这个链表的head节点，并不会影响其他的table元素的读写，好处在于并发的粒度更细，影响更小，从而并发效率更好，但不足之处在于并发扩容的时候，由于操作的table都是同一个，不像JDK7中分段控制，所以这里需要等扩容完之后，所有的读写操作才能进行，所以扩容的效率就成为了整个并发的一个瓶颈点。好在Doug lea大神对扩容做了优化，本来在一个线程扩容的时候，如果影响了其他线程的数据，那么其他的线程的读写操作都应该阻塞，但Doug lea说你们闲着也是闲着，不如来一起参与扩容任务，这样人多力量大，办完事你们该干啥干啥，别浪费时间，于是在JDK8的源码里面就引入了一个ForwardingNode类，在一个线程发起扩容的时候，就会改变sizeCtl这个值。

扩容时候会判断这个值，如果超过阈值就要扩容，首先**根据运算得到需要遍历的次数i，然后利用tabAt方法获得i位置的元素f，初始化一个forwardNode实例fwd，如果f == null，则在table中的i位置放入fwd，否则采用头插法的方式把当前旧table数组的指定任务范围的数据给迁移到新的数组中，然后给旧table原位置赋值fwd**。直到遍历过所有的节点以后就完成了复制工作，把table指向nextTable，并更新sizeCtl为新数组大小的0.75倍 ，扩容完成。**在此期间如果其他线程的有读写操作都会判断head节点是否为forwardNode节点，如果是就帮助扩容。** 

https://www.cnblogs.com/lfs2640666960/p/9621461.html

## 5.Java8为什么放弃分段锁

加入**多个分段锁浪费内存空间**。

生产环境中， map 在放入时**竞争同一个锁的概率非常小，分段锁反而会造成更新等操作的长时间等待**。

为了**提高 GC 的效率**

实现降低锁的粒度

## 6. 为什么get不加锁

在1.8中ConcurrentHashMap的get操作全程不需要加锁，这也是它比其他并发集合比如hashtable、用Collections.synchronizedMap()包装的**hashmap;安全效率高的原因之一**。

get操作全程不需要加锁是因为**Node的成员val是用volatile修饰的**和数组用volatile修饰没有关系。

数组用volatile修饰主要是**保证在数组扩容的时候保证可见性**。

https://www.cnblogs.com/keeya/p/9632958.html

**get方法无需加锁，除非读到空的值才会加锁重读**。由于其中涉及到的**共享变量都使用volatile修饰**，volatile可以保证内存可见性，所以不会读取到过期数据。这是用**volatile替换锁的经典应用场景，不想HashTable需要加锁。**

## 7. CAS 和 synchronized 用在 ConcurrentHashMap 的什么地方？(CVTE)

大量应用来的CAS方法进行变量、属性的修改工作。  利用CAS进行无锁操作，可以大大提高性能。

ConcurrentHashMap定义了三个原子操作，用于对**指定位置的节点进行操作**。正是这些原子操作保证了

ConcurrentHashMap的线程安全。

https://blog.csdn.net/u010647035/article/details/86375981

## 8.为什么要使用CAS+Synchronized取代Segment+ReentrantLock

Synchronized是将每一个Node对象作为了一个锁,这样做的好处是什么呢?将锁细化了,也就是说,除非两个线程同时操作一个Node,注意,是一个Node而不是一个Node链表哦,那么才会争抢同一把锁.

如果使用ReentrantLock其实也可以将锁细化成这样的,只要让Node类继承ReentrantLock就行了,这样的话调用f.lock()就能做到和Synchronized(f)同样的效果,但为什么不这样做呢?

请大家试想一下,锁已经被细化到这种程度了,那么出现并发争抢的可能性还高吗?还有就是,哪怕出现争抢了,**只要线程可以在30到50次自旋里拿到锁,那么Synchronized就不会升级为重量级锁,而等待的线程也就不用被挂起,我们也就少了挂起和唤醒这个上下文切换的过程开销。**

但如果是ReentrantLock呢?它则只有在线程没有抢到锁,然后**新建Node节点后再尝试一次而已,不会自旋,而是直接被挂起,这样一来,我们就很容易会多出线程上下文开销的代价**。当然,你也可以使用tryLock(),但是这样又出现了一个问题,你怎么知道tryLock的时间呢?在时间范围里还好,假如超过了呢?

所以,在锁被细化到如此程度上,使用Synchronized是最好的选择了.这里再补充一句,Synchronized和ReentrantLock他们的开销差距是在释放锁时唤醒线程的数量,**Synchronized是唤醒锁池里所有的线程+刚好来访问的线程,而ReentrantLock则是当前线程后进来的第一个线程+刚好来访问的线程.**

如果是线程并发量不大的情况下,**那么Synchronized因为自旋锁,偏向锁,轻量级锁的原因,不用将等待线程挂起,偏向锁甚至不用自旋,所以在这种情况下要比ReentrantLock高效**

https://www.cnblogs.com/yangfeiORfeiyang/p/9694383.html

## 9. 扩容时读写怎么操作

  (1)对于get读操作，如果当前节点有数据，还没迁移完成，此时不影响读，能够正常进行。 

如果当前链表已经迁移完成，那么头节点会被设置成fwd节点，此时get线程会帮助扩容。 

(2)对于put/remove写操作，如果当前链表已经迁移完成，那么头节点会被设置成fwd节点，此时写线程会帮助扩容，如果扩容没有完成，当前链表的头节点会被锁住，所以写线程会被阻塞，直到扩容完成。  

https://www.cnblogs.com/lfs2640666960/p/9621461.html

# 1.存储结构

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/Concurrenthashmap-ds.png)


```java
static final class HashEntry<K,V> {
final int hash;
final K key;
volatile V value;
volatile HashEntry<K,V> next;
}
```

ConcurrentHashMap 和 HashMap 实现上类似，最主要的差别是 ConcurrentHashMap **采用了分段锁（Segment）**，每个分段锁维护**着几个桶（HashEntry）**，多个线程可以同时访问不同分段锁上的桶，从而使其并发度更高（并发度就是 Segment 的个数）。

**Segment 继承自 ReentrantLock。**

```java
static final class Segment<K,V> extends ReentrantLock implements Serializable {

private static final long serialVersionUID = 2249069246763182397L;

static final int MAX_SCAN_RETRIES =
    Runtime.getRuntime().availableProcessors() > 1 ? 64 : 1;
```

**Segment 类似于 HashMap，一个 Segment 维护着一个 HashEntry 数组**

```java
transient volatile HashEntry<K,V>[] table;

transient int count;

transient int modCount;

transient int threshold;

final float loadFactor;
}

final Segment<K,V>[] segments;
```

默认的并发级别为 16，也就是说默认创建 16 个 Segment。

```java
static final int DEFAULT_CONCURRENCY_LEVEL = 16;
```

# 2.size 操作

在执行 size 操作时，需要**遍历所有 Segment 然后把 count 累计起来**。

ConcurrentHashMap 在执行 size 操作时**先尝试不加锁，如果连续两次不加锁操作得到的结果一致**，那么可以认为这个结果是正确的。

尝试次数使用 RETRIES_BEFORE_LOCK 定义，该值为 2，retries 初始值为 -1，因此尝试次数为 3。

如果**尝试的次数超过 3 次，就需要对每个 Segment 加锁。**

# 3.方法

## get方法

**先经过一次再散列，然后使用这个散列值通过散列运算定位到Segment，再通过散列算法定位到元素。**

```java
public V get(Object key) {
    Segment<K,V> s; 
    HashEntry<K,V>[] tab;
    int h = hash(key);
    long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE; //定位Segement的散列算法
    //先定位Segment，再定位HashEntry
    if ((s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)) != null &&
        (tab = s.table) != null) {
        for (HashEntry<K,V> e = (HashEntry<K,V>) UNSAFE.getObjectVolatile
                 (tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE); //定位HashEntry的散列算法
             e != null; e = e.next) {
            K k;
            if ((k = e.key) == key || (e.hash == h && key.equals(k)))
                return e.value;
        }
    }
    return null;
}
```

**get方法无需加锁，除非读到空的值才会加锁重读**。由于其中涉及到的**共享变量都使用volatile修饰**，volatile可以保证内存可见性，所以不会读取到过期数据。这是用**volatile替换锁的经典应用场景，不想HashTable需要加锁。**

## put方法

put方法首先定位到Segment，然后在Segement里进行插入操作。第一步判断是否需要**对Segement里的HashEntry数组进行扩容**，第二步定位添加元素的位置，然后将其放在HashEntry数组里。

```java
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
        HashEntry<K,V> node = tryLock() ? null :
            scanAndLockForPut(key, hash, value);//tryLock不成功时会遍历定位到的HashEnry位置的链表（遍历主要是为了使CPU缓存链表），若找不到，则创建HashEntry。tryLock一定次数后（MAX_SCAN_RETRIES变量决定），则lock。若遍历过程中，由于其他线程的操作导致链表头结点变化，则需要重新遍历。
        V oldValue;
        try {
            HashEntry<K,V>[] tab = table;
            int index = (tab.length - 1) & hash;//定位HashEntry，可以看到，这个hash值在定位Segment时和在Segment中定位HashEntry都会用到，只不过定位Segment时只用到高几位。
            HashEntry<K,V> first = entryAt(tab, index);
            for (HashEntry<K,V> e = first;;) {
                if (e != null) {
                    K k;
                    if ((k = e.key) == key ||
                        (e.hash == hash && key.equals(k))) {
                        oldValue = e.value;
                        if (!onlyIfAbsent) {
                            e.value = value;
                            ++modCount;
                        }
                        break;
                    }
                    e = e.next;
                }
                else {
                    if (node != null)
                        node.setNext(first);
                    else
                        node = new HashEntry<K,V>(hash, key, value, first);
                    int c = count + 1;
					//若c超出阈值threshold，需要扩容并rehash。扩容后的容量是当前容量的2倍。这样可以最大程度避免之前散列好的entry重新散列，具体在另一篇文章中有详细分析，不赘述。扩容并rehash的这个过程是比较消耗资源的。
                    if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                        rehash(node);
                    else
                        setEntryAt(tab, index, node);
                    ++modCount;
                    count = c;
                    oldValue = null;
                    break;
                }
            }
        } finally {
            unlock();
        }
        return oldValue;
    }
```

（1）是否需要扩容

在插入元素之前会判断Segement里的HashEntry数组是否超过容量（threshold），如果超过阈值，则对数组进行扩容。值得一提的是，Segement的扩容判断比HashMap更加恰当，因为HashMap是**在插入元素后判断元素是否已经达到容量**，如果达到了就进行扩容，但是很有可能扩容之后没有新元素插入，这时HashMap就进行了一次无效的扩容。

（2）如何扩容

在扩容的时候，首先会创建一个容量是原来容量两倍的数组，然后将原数组的元素进行再散列后插入到新的数组里。为了高效，ConcurrentHashMap不会对整个容器进行扩容，而只对某个segement扩容。

# 4.同步方式

Segment **继承自 ReentrantLock**，所以我们可以很方便的对每一个 Segment 上锁。

对于读操作，获取 Key 所在的 Segment 时，需要保证可见性。具体实现上可以使用 volatile 关键字，也可使用锁。但使用锁开销太大，而使用 **volatile 时每次写操作都会让所有 CPU 内缓存无效，也有一定开销**。ConcurrentHashMap 使用如下方法保证可见性，取得最新的 Segment。

```java
Segment<K,V> s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)
```

对于写操作，**并不要求同时获取所有 Segment 的锁，因为那样相当于锁住了整个 Map**。它会先获取**该 Key-Value 对所在的 Segment 的锁**，获取成功后就可以像操作一个普通的 HashMap 一样操作该 Segment，并保证该Segment 的安全性。

它使用了**自旋锁**，如果 tryLock 获取锁失败，说明锁被其它线程占用，此时通过循环再次以 tryLock 的方式申请锁。如果在循环过程中该 Key 所对应的链表头被修改，则重置 retry 次数。如果 retry 次数超过一定值，则使用 lock 方法申请锁。

# 5.JDK 1.8 的改动

JDK 1.7 使用**分段锁机制**来实现并发更新操作，核心类为 Segment，它**继承自重入锁 ReentrantLock**，并发程度与 Segment 数量相等。

JDK1.8 的时候已经**摒弃了Segment的概念**，而是直接用 **Node 数组+链表+红黑树**的数据结构来实现，并发控制使用 **synchronized 和 CAS 来操作**，在 CAS 操作失败时使用内置锁 synchronized。

数据结构跟HashMap1.8的结构类似，数组+链表/红黑二叉树。

synchronized只锁定**当前链表或红黑二叉树的首节点**，这样只要hash不冲突，就不会产生并发，效率又提升N倍。

 ***在Java 7中***，ConcurrentHashMap把内部细分成了若干个小的HashMap，称之为***段（Segment）***，默认被分为16个段。**对于一个写操作而言，会先根据hash code进行寻址，得出该Entry应被存放在哪一个Segment，然后只要对该Segment加锁即可。**理想情况下，一个默认的ConcurrentHashMap可以同时接受16个线程进行写操作（如果都是对不同Segment进行操作的话）。分段锁对于size()这样的全局操作来说就没有任何作用了，想要得出**Entry的数量就需要遍历所有Segment，获得所有的锁，然后再统计总数。**事实上，ConcurrentHashMap会**先试图使用无锁的方式**统计总数，这个尝试会进行3次，如果在**相邻的2次计算中获得的Segment的modCount次数一致**，代表这两次计算过程中都没有发生过修改操作，那么就可以当做最终结果返回，否则，就要获得所有Segment的锁，重新计算size。

![img](https://img2018.cnblogs.com/blog/1157683/201810/1157683-20181031153848790-14787290.png)
     **而Java 8的ConcurrentHashMap，**它与Java 7的实现差别较大。完全放弃了段的设计，而是变回与HashMap相似的设计，使用buckets数组与分离链接法（同样会在超过阈值时树化，对于构造红黑树的逻辑与HashMap差别不大，只不过需要额外使用CAS来保证线程安全），锁的粒度也被细分到每个数组元素（因为HashMap在Java 8中也实现了不少优化，即使碰撞严重，也能保证一定的性能，而且Segment不仅臃肿还有弱一致性的问题存在），所以它的并发级别与数组长度相关（Java 7则是与段数相关）。

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/ConcurrentHashMap%20JDK1.8.png)

## table

所有数据都被存放在table数组中，大小是2的整数次幂，存储的元素分为三种类型

- TreeBin 用于包装红黑树结构的结点类型 ，它继承了Node，代表它也是个节点，hash值为-2

- ForwardingNode **扩容时存放的结点类型**，并发扩容的实现关键之一 ，是一个标记，代表此处已完成扩容，hash值为-1。

- Node 普通结点类型

这里先来说一下CAS+volatile的组合，这两个是整个JUC包的基石。volatile 读的内存语义：当读一个volatile变量时，JMM会把线程的本地内存置为无效，线程接下来将**从主内存中读取值**。volatile写的内存语义：当写一个volatile变量时，JMM会**到主内存中取读值**。JSR-133增强volatile内存语义：之前旧的JMM允许volatile变量与普通变量重排序，之后严格限制编译器和处理器对volatile变量与普通变量的重排序，确保volatile写-读与锁的释放-获取有相同语义。也就是说：写线程A在写这个volatile变量之前所有可见的共享变量的值都将立即变得对读线程B可见。A写一个volatile变量，随后B读到这个volatile变量，这个过程实质上是线程A通过主内存向线程B发送消息。而CAS同时具有volatile读/写的内存语义，以Intel X86来说，就是利用在CMPXCHG指令前添加lock前缀来实现。

volatile的读写和CAS可以实现线程之间的通信，整合到一起就实现了Concurrent包得以实现的基石。在阅读JUC下的类时会发现一个通用的模式：**volatile的共享变量，CAS原子更新实现线程同步，二者搭配来实现线程之间的通信。**很多操作都是**先读volatile变量此时的最新值，赋给局部变量，然后一顿操作，最后CAS进行同步，若过程中共享变量被其它线程更改则会导致CAS失败，重新尝试。**

## **Node**

Node是最核心的内部类，**它包装了key-value键值对**，所有插入ConcurrentHashMap的数据都包装在这里面。它与HashMap中的定义很相似，但是有一些差别**它对value和next属性设置了volatile同步锁**，它不允许调用setValue方法直接改变Node的value域，它增加了find方法辅助map.get()方法。

```java
   static class Node<K,V> implements Map.Entry<K,V> {  
       final int hash;  
       final K key;  
       volatile V val;//带有同步锁的value  
       volatile Node<K,V> next;//带有同步锁的next指针  
  
       Node(int hash, K key, V val, Node<K,V> next) {  
           this.hash = hash;  
           this.key = key;  
           this.val = val;  
           this.next = next;  
       }  
  
       public final K getKey()       { return key; }  
       public final V getValue()     { return val; }  
       public final int hashCode()   { return key.hashCode() ^ val.hashCode(); }  
       public final String toString(){ return key + "=" + val; }  
       //不允许直接改变value的值  
       public final V setValue(V value) {  
           throw new UnsupportedOperationException();  
       }  
  
       public final boolean equals(Object o) {  
           Object k, v, u; Map.Entry<?,?> e;  
           return ((o instanceof Map.Entry) &&  
                   (k = (e = (Map.Entry<?,?>)o).getKey()) != null &&  
                   (v = e.getValue()) != null &&  
                   (k == key || k.equals(key)) &&  
                   (v == (u = val) || v.equals(u)));  
       }  
  
       /** 
        * Virtualized support for map.get(); overridden in subclasses. 
        */  
       Node<K,V> find(int h, Object k) {  
           Node<K,V> e = this;  
           if (k != null) {  
               do {  
                   K ek;  
                   if (e.hash == h &&  
                       ((ek = e.key) == k || (ek != null && k.equals(ek))))  
                       return e;  
               } while ((e = e.next) != null);  
           }  
           return null;  
       }  
   }  
     
   这个Node内部类与HashMap中定义的Node类很相似，但是有一些差别  
   它对value和next属性设置了volatile同步锁  
   它不允许调用setValue方法直接改变Node的value域  
   它增加了find方法辅助map.get()方法  
```

**value和next都用volatile修饰，保证并发的可见性**

## **ForwardingNode**

```java
static final class ForwardingNode<K,V> extends Node<K,V> {
        final Node<K,V>[] nextTable;
        ForwardingNode(Node<K,V>[] tab) {
            super(MOVED, null, null, null);
            this.nextTable = tab;
}
```

ForwardingNode作用在扩容期间，hash值为MOVED 值为-1，当table[ i ] 是个ForwardingNode节点时，代表该位置节点已经移至新数组。

## **nextTable**

扩容时**新生成的数组**，其大小为原数组的两倍

## **sizeCtl**

**多线程之间，以volatile的方式读取sizeCtl属性，来判断ConcurrentHashMap当前所处的状态。通过cas设置sizeCtl属性，告知其他线程ConcurrentHashMap的状态变更**。

不同状态，sizeCtl所代表的含义也有所不同。

- 未初始化：
  - sizeCtl=0：表示没有指定初始容量。
  - sizeCtl>0：表示初始容量。

- 初始化中：
  - sizeCtl=-1,标记作用，**告知其他线程，正在初始化**
- 正常状态：
  - sizeCtl=0.75n ,扩容阈值
- 扩容中:
  - sizeCtl < 0 : 表示有其他线程正在执行扩容
  - sizeCtl = (resizeStamp(n) << RESIZE_STAMP_SHIFT) + 2 :表示此时只有一个线程在执行扩容

```java
 /** 
   * 盛装Node元素的数组 它的大小是2的整数次幂 
   * Size is always a power of two. Accessed directly by iterators. 
   */  
  transient volatile Node<K,V>[] table;  
  
/** 
   * Table initialization and resizing control.  When negative, the 
   * table is being initialized or resized: -1 for initialization, 
   * else -(1 + the number of active resizing threads).  Otherwise, 
   * when table is null, holds the initial table size to use upon 
   * creation, or 0 for default. After initialization, holds the 
   * next element count value upon which to resize the table. 
   hash表初始化或扩容时的一个控制位标识量。 
   负数代表正在进行初始化或扩容操作 
   -1代表正在初始化 
   -N 表示有N-1个线程正在进行扩容操作 
   正数或0代表hash表还没有被初始化，这个数值表示初始化或下一次进行扩容的大小 
    
   */  
  private transient volatile int sizeCtl;   
  // 以下两个是用来控制扩容的时候 单线程进入的变量  
   /** 
   * The number of bits used for generation stamp in sizeCtl. 
   * Must be at least 6 for 32bit arrays. 
   */  
  private static int RESIZE_STAMP_BITS = 16;  
/** 
   * The bit shift for recording size stamp in sizeCtl. 
   */  
  private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;  
    
    
  /* 
   * Encodings for Node hash fields. See above for explanation. 
   */  
  static final int MOVED     = -1; // hash值是-1，表示这是一个forwardNode节点  
  static final int TREEBIN   = -2; // hash值是-2  表示这时一个TreeBin节点  
```

## put

如果一个或多个线程正在对ConcurrentHashMap进行扩容操作，当前线程也要进入扩容的操作中。这个扩容的操作之所以能被检测到，是因为**transfer方法中在空结点上插入forward节点，如果检测到需要插入的位置被forward节点占有，就帮助进行扩容；**
 如果检测到**要插入的节点是非空且不是forward节点，就对这个节点加锁，这样就保证了线程安全**。尽管这个有一些影响效率，但是还是会比HashTable的synchronized要好得多。

putVal(K key, V value, boolean onlyIfAbsent)方法干的工作如下：
 1、检查key/value是否为空，如果为空，则抛异常，否则进行2
 2、进入for死循环，进行3
 3、检查table是否初始化了，如果没有，则调用initTable()进行初始化然后进行 2，否则进行4
 4、根据key的hash值计算出其应该在table中储存的位置i，取出table[i]的节点用f表示。
 根据f的不同有如下三种情况：
 1）如果table[i]==null(即该位置的节点为空，没有发生碰撞)，
 则利用CAS操作直接存储在该位置，如果CAS操作成功则退出死循环。
 2）如果table[i]!=null(即该位置已经有其它节点，发生碰撞)，碰撞处理也有两种情况
 2.1）检查table[i]的节点的**hash是否等于MOVED**，如果等于，则**检测到正在扩容，则帮助其扩容**
 2.2）说明table[i]的节点的**hash值不等于MOVED**，如果table[i]为链表节点，则将此节点插入链表中即可
 3 )     如果table[i]为树节点，则将此节点插入树中即可。插入成功后，进行 5
 5、如果table[i]的节点是链表节点，则检查table的第i个位置的链表是否需要转化为数，如果需要则调用treeifyBin函数进行转化

1、第一步根据给定的key的hash值找到其在table中的位置index。
 2、找到位置index后，存储进行就好了。

只是这里的存储有三种情况罢了，第一种：table[index]中没有任何其他元素，即此元素没有发生碰撞，这种情况直接存储就好了哈。第二种，table[i]存储的是一个链表，如果链表不存在key则**直接加入到链表尾部即可**，如果存在key则更新其对应的value。第三种，table[i]存储的是一个树，则按照树添加节点的方法添加就好。

```java
public V put(K key, V value) {  
        return putVal(key, value, false);  
    }  
  
    /** Implementation for put and putIfAbsent */  
    final V putVal(K key, V value, boolean onlyIfAbsent) {  
            //不允许 key或value为null  
        if (key == null || value == null) throw new NullPointerException();  
        //计算hash值  
        int hash = spread(key.hashCode());  
        int binCount = 0;  
        //死循环 何时插入成功 何时跳出  
        for (Node<K,V>[] tab = table;;) {  
            Node<K,V> f; int n, i, fh;  
            //如果table为空的话，初始化table  
            if (tab == null || (n = tab.length) == 0)  
                tab = initTable();  
            //根据hash值计算出在table里面的位置   
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {  
                //如果这个位置没有值 ，直接放进去，不需要加锁  
                if (casTabAt(tab, i, null,  
                             new Node<K,V>(hash, key, value, null)))  
                    break;                   // no lock when adding to empty bin  
            }  
            //当遇到表连接点时，需要进行整合表的操作  
            else if ((fh = f.hash) == MOVED)  
                tab = helpTransfer(tab, f);  
            else {  
                V oldVal = null;  
                //结点上锁  这里的结点可以理解为hash值相同组成的链表的头结点  
                synchronized (f) {  
                    if (tabAt(tab, i) == f) {  
                        //fh〉0 说明这个节点是一个链表的节点 不是树的节点  
                        if (fh >= 0) {  
                            binCount = 1;  
                            //在这里遍历链表所有的结点  
                            for (Node<K,V> e = f;; ++binCount) {  
                                K ek;  
                                //如果hash值和key值相同  则修改对应结点的value值  
                                if (e.hash == hash &&  
                                    ((ek = e.key) == key ||  
                                     (ek != null && key.equals(ek)))) {  
                                    oldVal = e.val;  
                                    if (!onlyIfAbsent)  
                                        e.val = value;  
                                    break;  
                                }  
                                Node<K,V> pred = e;  
                                //如果遍历到了最后一个结点，那么就证明新的节点需要插入 就把它插入在链表尾部  
                                if ((e = e.next) == null) {  
                                    pred.next = new Node<K,V>(hash, key,  
                                                              value, null);  
                                    break;  
                                }  
                            }  
                        }  
                        //如果这个节点是树节点，就按照树的方式插入值  
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
                    //如果链表长度已经达到临界值8 就需要把链表转换为树结构  
                    if (binCount >= TREEIFY_THRESHOLD)  
                        treeifyBin(tab, i);  
                    if (oldVal != null)  
                        return oldVal;  
                    break;  
                }  
            }  
        }  
        //将当前ConcurrentHashMap的元素数量+1  
        addCount(1L, binCount);  
        return null;  
    }  
      
```

在putVal函数，出现了如下几个函数
 1、casTabAt tabAt 等CAS操作
 2、initTable 作用是初始化table数组
 3、treeifyBin 作用是将table[i]的链表转化为树

```
1、根据 key 计算 hash 值

2、遍历数组 tab 
2.1、如果数组为空，进行数组初始化 initTable()
2.2、如果数组不为空，根据 hash 值找到在数组 tab 中的位置，并获取该位置的第一个节点元素对象
    2.2.1、如果数组该位置为空，使用 CAS 操作将值放入该位置
    2.2.2、如果数组该位置的首个元素的hash 值等于 MOVED ，说明在扩容，帮助数据迁移 
    helpTransfer
    
2.3、如果不满足以上（2.1、2.2）条件，执行以下逻辑：
    2.3.1、获取数组该位置的首节点的监视器锁
    2.3.2、首节点 hash 值大于 0，说明是链表，链表遍历插入值
    2.3.3、否则为红黑树，使用红黑树的 putTreeVal 方法，插入新值
    2.3.4、如果以上执行的是链表操作，判断链表长度是否达到限定值，是否需要转换为红黑树
```

https://blog.csdn.net/u010647035/article/details/86375981

### 总结

- 根据 key 计算出 hashcode 。

- 判断是否需要进行初始化。

- `f` 即为当前 key 定位出的 Node，如果为空表示当前位置可以写入数据，**利用 CAS 尝试写入**，失败则自旋保证成功。

- 如果当前位置的 `hashcode == MOVED == -1`,则需要进行扩容。

- 如果都不满足，**则利用 synchronized 锁写入数据**。

- 如果数量大于 `TREEIFY_THRESHOLD` 则要转换为红黑树。

  作者：crossoverJie链接：https://juejin.im/post/5b551e8df265da0f84562403

## Unsafe与CAS

在ConcurrentHashMap中，随处可以看到U, 大量使用了U.compareAndSwapXXX的方法，这个方法是**利用一个CAS算法实现无锁化的修改值的操作，他可以大大降低锁代理的性能消耗。**这个算法的基本思想就是不断地去比较当前内存中的变量值与你指定的一个变量值是否相等，如果相等，则接受你指定的修改的值，否则拒绝你的操作。因为当前线程中的值已经不是最新的值，你的修改很可能会覆盖掉其他线程修改的结果。这一点与乐观锁，SVN的思想是比较类似的。

unsafe代码块控制了一些属性的修改工作，比如最常用的SIZECTL 。  在这一版本的concurrentHashMap中，大量应用来的CAS方法进行变量、属性的修改工作。  利用CAS进行无锁操作，可以大大提高性能。

```java
ConcurrentHashMap定义了三个原子操作，用于对指定位置的节点进行操作。正是这些原子操作保证了ConcurrentHashMap的线程安全。
   //获得在i位置上的Node节点  
   static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {  
       return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);  
   }  
    //利用CAS算法设置i位置上的Node节点。之所以能实现并发是因为他指定了原来这个节点的值是多少  
    //在CAS算法中，会比较内存中的值与你指定的这个值是否相等，如果相等才接受你的修改，否则拒绝你的修改  
    //因此当前线程中的值并不是最新的值，这种修改可能会覆盖掉其他线程的修改结果  有点类似于SVN  
   static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,  
                                       Node<K,V> c, Node<K,V> v) {  
       return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);  
   }  
    //利用volatile方法设置节点位置的值  
   static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {  
       U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);  
   } 
```

## 扩容

1. 通过计算 CPU 核心数和 Map 数组的长度得到每个线程（CPU）要帮助处理多少个桶，并且这里每个线程处理都是平均的。默认每个线程处理 16 个桶。因此，如果长度是 16 的时候，扩容的时候只会有一个线程扩容。

2. 初始化临时变量 nextTable。将其在原有基础上扩容两倍。

3. 死循环开始转移。多线程并发转移就是在这个死循环中，根据一个 finishing 变量来判断，该变量为 true 表示扩容结束，否则继续扩容。

   3.1 进入一个 while 循环，分配数组中一个桶的区间给线程，默认是 16. 从大到小进行分配。当拿到分配值后，进行 i-- 递减。这个 i 就是数组下标。（`其中有一个 bound 参数，这个参数指的是该线程此次可以处理的区间的最小下标，超过这个下标，就需要重新领取区间或者结束扩容，还有一个 advance 参数，该参数指的是是否继续递减转移下一个桶，如果为 true，表示可以继续向后推进，反之，说明还没有处理好当前桶，不能推进`) 3.2 出 while 循环，进 if 判断，判断扩容是否结束，如果扩容结束，清空临死变量，更新 table 变量，更新库容阈值。如果没完成，但已经无法领取区间（没了），该线程退出该方法，并将 sizeCtl 减一，表示扩容的线程少一个了。如果减完这个数以后，sizeCtl 回归了初始状态，表示没有线程再扩容了，该方法所有的线程扩容结束了。（`这里主要是判断扩容任务是否结束，如果结束了就让线程退出该方法，并更新相关变量`）。然后检查所有的桶，防止遗漏。 3.3 如果没有完成任务，且 i 对应的槽位是空，尝试 CAS 插入占位符，让 putVal 方法的线程感知。 3.4 如果 i 对应的槽位不是空，且有了占位符，那么该线程跳过这个槽位，处理下一个槽位。 3.5 如果以上都是不是，说明这个槽位有一个实际的值。开始同步处理这个桶。 3.6 到这里，都还没有对桶内数据进行转移，只是计算了下标和处理区间，然后一些完成状态判断。同时，如果对应下标内没有数据或已经被占位了，就跳过了。

4. 处理每个桶的行为都是同步的。防止 putVal 的时候向链表插入数据。 4.1 如果这个桶是链表，那么就将这个链表根据 length 取于拆成两份，取于结果是 0 的放在新表的低位，取于结果是 1 放在新表的高位。 4.2 如果这个桶是红黑数，那么也拆成 2 份，方式和链表的方式一样，然后，判断拆分过的树的节点数量，如果数量小于等于 6，改造成链表。反之，继续使用红黑树结构。 4.3 到这里，就完成了一个桶从旧表转移到新表的过程。

好，以上，就是 transfer 方法的总体逻辑。还是挺复杂的。再进行精简，分成 3 步骤：

1. **计算每个线程可以处理的桶区间**。默认 16.
2. **初始化临时变量 nextTable**，扩容 2 倍。
3. 死循环，计算下标。完成总体判断。
4. 1 如果桶内有数据，同步转移数据。通常会像链表拆成 2 份。


作者：莫那·鲁道链接：https://juejin.im/post/5b00160151882565bd2582e0

https://www.jianshu.com/p/01098075b2bf

# 6.总结

## ConcurrentHashMap的Key和Value不允许null值

ConcurrentHashmap和Hashtable都是支持并发的，这样会有一个问题，当你通过get(k)获取对应的value时，如果获取到的是null时，你无法判断，它是put（k,v）的时候value为null，还是这个key从来没有做过映射。

JDK6,7中的ConcurrentHashmap主要**使用Segment来实现减小锁粒度**，把HashMap分割成若干个Segment，**在put的时候需要锁住Segment，get时候不加锁，使用volatile来保证可见性**，当要统计全局时（比如size），首先会尝试多次计算modcount来确定，这几次尝试中，是否有其他线程进行了修改操作，如果没有，则直接返回size。如果有，则需要依次锁住所有的Segment来计算。

jdk7中ConcurrentHashmap中，当长度过长碰撞会很频繁，链表的增改删查操作都会消耗很长的时间，影响性能,所以jdk8 中完全重写了concurrentHashmap,代码量从原来的1000多行变成了 6000多 行，实现上也和原来的分段式存储有很大的区别。

主要设计上的变化有以下几点:

1. 不采用segment而采用node，**锁住node来实现减小锁粒度**。

1. 设计了**MOVED状态 当resize的中过程中 线程2还在put数据，线程2会帮助resize**。

1. 使用**3个CAS操作来确保node的一些操作的原子性**，这种方式代替了锁。

1. sizeCtl的不同值来代表不同含义，起到了控制的作用。

   