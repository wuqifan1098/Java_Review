# 面试题

## 1. List去重的方法 （壹钱包）

- **通过HashSet踢除重复元素**

```java
Set<Integer> set = new HashSet<>(list);
```

此种方式是利用了Set的特性:元素不可重复,其底层原理是通过先计算每个对象的hash值,再比较元素值是否相同,如果相同,则保留最新的. 

- **循环list中的所有元素然后删除重复**

```java
 for (int i = 0; i < list.size() - 1; i++) {
            for (int j = list.size() - 1; j > i; j--) {
                if (list.get(j).equals(list.get(i))) {
                    list.remove(j);
                }
            }
}
```

- **用JDK1.8 Stream中对List进行去重**

```java
list.stream().distinct();
```

首先获得此list的Stream.然后调用distinct()方法,java8中提供流的方式对数据进行处理,非常快,底层用的是forkJoin框架,提供了并行处理,使得多个处理器同时处理流中的数据,所以耗时非常短 。

- **Lambda表达式去重**

# Arraylist 和 Vector 的区别

1.  Vector 和 ArrayList 几乎是完全相同的，唯一的区别在于**Vector 是同步的，因此开销就比 ArrayList 要大**，访问要慢。最好使用 ArrayList 而不是 Vector，因为同步完全可以由程序员自己来控制；

2.  Vector 每次扩容请求其大小的 2 倍空间，而 ArrayList 是 1.5 倍。

## 替代方案

synchronizedList

为了使用线程安全的 ArrayList，可以使用 **Collections.synchronizedList**(new ArrayList<>()); 
    
```java
List<String> list = new ArrayList<>();
List<String> synList = Collections.synchronizedList(list);
```

CopyOnWriteArrayList
![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/CopyOnWriteArrayList.png)

返回一个线程安全的 ArrayList，也可以使用 concurrent 并发包下的 **CopyOnWriteArrayList 类**；

CopyOnWrite 容器即写时复制的容器。通俗的理解是**当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行 Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。**这样做的好处是我们可以对 CopyOnWrite 容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。所以 CopyOnWrite 容器也是一种**读写分离**的思想，读和写不同的容器。

CopyOnWrite的缺点

CopyOnWrite 容器有很多优点，但是同时也存在两个问题，即**内存占用问题和数据一致性**问题。所以在开发的时候需要注意一下。

- 内存占用问题 在进行写操作的时候，内存里会**同时驻扎两个对象的内存**，旧的对象和新写入的对象。
- 数据一致性问题。CopyOnWrite 容器只能保证数据的最终一致性，**不能保证数据的实时一致性**，会读到原始容器里的旧值。

# Arraylist 和 LinkedList 的区别

1. ArrayList 基于**动态数组**实现，需要连续的地址，LinkedList 基于**双向循环链表**实现，开辟内存空间的时候**不需要等一个连续的地址**；

2. ArrayList 支持**随机访问**，LinkedList **不支持**，要移动指针；

3. LinkedList 在任意位置**添加删除元素**更快。

1. ArrayList与LinkedList都**不是线程安全的**。

# Arraylist 和 Stack 的区别

Stack：它继承于Vector。它的特性是：先进后出(FILO, FirstIn Last Out)。也是同步的。

# ArrayList 与 LinkedList remove方法的效率问题

**理论上**：对于随机增删操作，LinkedList平均效率优于ArrayList，因为LinkedList**只需要改变其排列的一个node结点就可以了**，而对于ArrayList来说删除一个元素，需要不断把后面的元素移到前面的位置上。

实际中：对于随机增删操作，有时LinkedList耗时明显大于ArrayList，不符合预期。

**原因分析**：经过分析源码可知，LinkedList的remove(int index)和remove(Object o)这两个方法的**时间复杂度均为O(n)**，并非为O(1)的时间复杂度。LinkedList中，首先要有个寻址过程，找到第一个出现的Object o，或者走到index这个位置，再进行操作，其中寻址很耗时；ArrayList的耗时**由寻址和拷贝两部分组成，寻址时间复杂度为O(1)**，拷贝很耗时。因此在某些特殊位置的删除操作下，可能会出现LinkedList耗时明显大于ArrayList。

结论：LinkedList效率优于ArrayList并非一成不变，而是根据情况不同，会有些反转。但是**对于大批量删除或插入操作**，LinkedList平均效率还是优于ArrayList的。

测试：将20万条String类型的数据分别添加到一个LinkedList和一个ArrayList中，且都是在第0位插入数据，ArrayList用了2265毫秒，而LinkedList只用了12毫秒，LinkedList的插入效率比ArrayList快180多倍。

# 总计

1. 在尾部插入数据，**数据量较小时LinkedList比较快，因为ArrayList要频繁扩容，当数据量大时ArrayList比较快，因为ArrayList扩容是当前容量*1.5，大容量扩容一次就能提供很多空间，当ArrayList不需扩容时效率明显比LinkedList高，**因为直接数组元素赋值不需new Node

2. 在首部插入数据，**LinkedList较快，因为LinkedList遍历插入位置花费时间很小，而ArrayList需要将原数组所有元素进行一次System.arraycopy**

3. 插入位置越往中间，**LinkedList效率越低，因为它遍历获取插入位置是从两头往中间搜，index越往中间遍历越久，因此ArrayList的插入效率可能会比LinkedList高**

4. 插入位置越往后，ArrayList效率越高，因为数组需要复制后移的数据少了，那么System.arraycopy就快了，因此在首部插入数据LinkedList效率比ArrayList高，尾部插入数据ArrayList效率比LinkedList高

5. LinkedList可以实现队列，栈等数据结构，这是它的优势