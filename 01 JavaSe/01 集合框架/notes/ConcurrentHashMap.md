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

Segment 继承自 ReentrantLock。

```java
static final class Segment<K,V> extends ReentrantLock implements Serializable {

private static final long serialVersionUID = 2249069246763182397L;

static final int MAX_SCAN_RETRIES =
    Runtime.getRuntime().availableProcessors() > 1 ? 64 : 1;
```

Segment 类似于 HashMap，一个 Segment 维护着一个 HashEntry 数组

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

如果尝试的次数超过 3 次，就需要对每个 Segment 加锁。

# 3.方法

get方法

先经过一次再散列，然后使用这个散列值通过散列运算定位到Segment，再通过散列算法定位到元素。

```java
public V get(Object key) {
    Segment<K,V> s; 
    HashEntry<K,V>[] tab;
    int h = hash(key);
    long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE;
    //先定位Segment，再定位HashEntry
    if ((s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)) != null &&
        (tab = s.table) != null) {
        for (HashEntry<K,V> e = (HashEntry<K,V>) UNSAFE.getObjectVolatile
                 (tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE);
             e != null; e = e.next) {
            K k;
            if ((k = e.key) == key || (e.hash == h && key.equals(k)))
                return e.value;
        }
    }
    return null;
}
```

get方法无需加锁，由于其中涉及到的**共享变量都使用volatile修饰**，volatile可以保证内存可见性，所以不会读取到过期数据。这是用volatile替换锁的经典应用场景。

put方法

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

put方法首先定位到Segment，然后在Segment进行插入操作。第一步判断是否需要对Segment里的HashEntry数组进行扩容，第二步添加元素的位置，然后放到hashEntry数组里。

# 4.同步方式

Segment **继承自 ReentrantLock**，所以我们可以很方便的对每一个 Segment 上锁。

对于读操作，获取 Key 所在的 Segment 时，需要保证可见性。具体实现上可以使用 volatile 关键字，也可使用锁。但使用锁开销太大，而使用 volatile 时每次写操作都会让所有 CPU 内缓存无效，也有一定开销。ConcurrentHashMap 使用如下方法保证可见性，取得最新的 Segment。

```java
Segment<K,V> s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)
```

对于写操作，并不要求同时获取所有 Segment 的锁，因为那样相当于锁住了整个 Map。它会先获取该 Key-Value 对所在的 Segment 的锁，获取成功后就可以像操作一个普通的 HashMap 一样操作该 Segment，并保证该Segment 的安全性。

它使用了**自旋锁**，如果 tryLock 获取锁失败，说明锁被其它线程占用，此时通过循环再次以 tryLock 的方式申请锁。如果在循环过程中该 Key 所对应的链表头被修改，则重置 retry 次数。如果 retry 次数超过一定值，则使用 lock 方法申请锁。

# 5.JDK 1.8 的改动

JDK 1.7 使用**分段锁机制**来实现并发更新操作，核心类为 Segment，它**继承自重入锁 ReentrantLock**，并发程度与 Segment 数量相等。

JDK1.8 的时候已经**摒弃了Segment的概念**，而是直接用 Node 数组+链表+红黑树的数据结构来实现，并发控制使用 **synchronized 和 CAS 来操作**，在 CAS 操作失败时使用内置锁 synchronized。

数据结构跟HashMap1.8的结构类似，数组+链表/红黑二叉树。

synchronized只锁定当前链表或红黑二叉树的首节点，这样只要hash不冲突，就不会产生并发，效率又提升N倍。

 ***在Java 7中***，ConcurrentHashMap把内部细分成了若干个小的HashMap，称之为***段（Segment）***，默认被分为16个段。对于一个写操作而言，会先根据hash code进行寻址，得出该Entry应被存放在哪一个Segment，然后只要对该Segment加锁即可。理想情况下，一个默认的ConcurrentHashMap可以同时接受16个线程进行写操作（如果都是对不同Segment进行操作的话）。分段锁对于size()这样的全局操作来说就没有任何作用了，想要得出Entry的数量就需要遍历所有Segment，获得所有的锁，然后再统计总数。事实上，ConcurrentHashMap会先试图使用无锁的方式统计总数，这个尝试会进行3次，如果在相邻的2次计算中获得的Segment的modCount次数一致，代表这两次计算过程中都没有发生过修改操作，那么就可以当做最终结果返回，否则，就要获得所有Segment的锁，重新计算size。

![img](https://img2018.cnblogs.com/blog/1157683/201810/1157683-20181031153848790-14787290.png)
     **而Java 8的ConcurrentHashMap，**它与Java 7的实现差别较大。完全放弃了段的设计，而是变回与HashMap相似的设计，使用buckets数组与分离链接法（同样会在超过阈值时树化，对于构造红黑树的逻辑与HashMap差别不大，只不过需要额外使用CAS来保证线程安全），锁的粒度也被细分到每个数组元素（因为HashMap在Java 8中也实现了不少优化，即使碰撞严重，也能保证一定的性能，而且Segment不仅臃肿还有弱一致性的问题存在），所以它的并发级别与数组长度相关（Java 7则是与段数相关）。

# 6.总结

JDK6,7中的ConcurrentHashmap主要**使用Segment来实现减小锁粒度**，把HashMap分割成若干个Segment，在put的时候需要锁住Segment，get时候不加锁，使用volatile来保证可见性，当要统计全局时（比如size），首先会尝试多次计算modcount来确定，这几次尝试中，是否有其他线程进行了修改操作，如果没有，则直接返回size。如果有，则需要依次锁住所有的Segment来计算。

jdk7中ConcurrentHashmap中，当长度过长碰撞会很频繁，链表的增改删查操作都会消耗很长的时间，影响性能,所以jdk8 中完全重写了concurrentHashmap,代码量从原来的1000多行变成了 6000多 行，实现上也和原来的分段式存储有很大的区别。

主要设计上的变化有以下几点:

1. 不采用segment而采用node，**锁住node来实现减小锁粒度**。
1. 设计了MOVED状态 当resize的中过程中 线程2还在put数据，线程2会帮助resize。
1. 使用3个CAS操作来确保node的一些操作的原子性，这种方式代替了锁。
1. sizeCtl的不同值来代表不同含义，起到了控制的作用。