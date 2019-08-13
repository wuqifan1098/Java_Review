# 面试题

## 1. 介绍一下ThreadLocal（新浪）

ThreadLocal**提供了线程的局部变量**，每个线程都可以通过set()和get()来对这个局部变量进行操作，而不会影响其它线程所对应的副本，避免了线程竞争，**实现了线程的数据隔离**。

## 2. 设计一个ThreadLocal（新浪）

# 一、对ThreadLocal的理解

ThreadLocal**提供了线程的局部变量**，每个线程都可以通过set()和get()来对这个局部变量进行操作，而不会影响其它线程所对应的副本，避免了线程竞争，**实现了线程的数据隔离**。

# 二、深入解析ThreadLocal类

结构概览

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/ThreadLocal%E7%BB%93%E6%9E%84%E6%A6%82%E8%A7%88.jpg)

清晰的看到**一个线程 Thread 中存在一个 ThreadLocalMap**，ThreadLocalMap 中的 **key 对应 ThreadLocal**，在此处可见 Map 可以存储多个 key 即 (ThreadLocal)。另外**Value 就对应着在 ThreadLocal 中存储的 Value**。

ThreadLocal的类声明：

```java
public class ThreadLocal<T>{

static class ThreadLocalMap {
```

可以看出ThreadLocal并没有继承自Thread，也没有实现Runnable接口。

### ThreadLocal原理

从 `Thread`类源代码入手。

```java
public class Thread implements Runnable {
 ......
//与此线程有关的ThreadLocal值。由ThreadLocal类维护
ThreadLocal.ThreadLocalMap threadLocals = null;

//与此线程有关的InheritableThreadLocal值。由InheritableThreadLocal类维护
ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
 ......
}
```

从上面`Thread`类 源代码可以看出`Thread` 类中有一个 `threadLocals` 和 一个 `inheritableThreadLocals` 变量，它们都是 `ThreadLocalMap` 类型的变量,我们可以把 **ThreadLocalMap 理解为ThreadLocal类实现的定制化的 HashMap**。默认情况下这两个变量都是null，只有当前线程调用 `ThreadLocal` 类的 `set`或`get`方法时才创建它们，**实际上调用这两个方法的时候，我们调用的是ThreadLocalMap类对应的 get()、set() 方法。**

ThreadLocal类提供的几个方法：

```java
public T get() { }
public void set(T value) { }
public void remove() { }
protected T initialValue() { }
```

1.ThreadLocal set  

```java
    public void set(T value) {
	// 获取当前线程对象
	Thread t = Thread.currentThread();
	// 根据当前线程的对象获取其内部Map
	ThreadLocalMap map = getMap(t);

	// 注释1
	if (map != null)
		map.set(this, value);
	else
	createMap(t, value);
	}
```

通过上面这些内容，我们足以通过猜测得出结论：**最终的变量是放在了当前线程的 ThreadLocalMap 中，并不是存在 ThreadLocal 上，ThreadLocal 可以理解为只是ThreadLocalMap的封装，传递了变量值。** `ThrealLocal` 类中可以通过`Thread.currentThread()`获取到当前线程对象后，直接通过`getMap(Thread t)`可以访问到该线程的`ThreadLocalMap`对象。

**每个Thread中都具备一个ThreadLocalMap，而ThreadLocalMap可以存储以ThreadLocal为key的键值对。** 比如我们在同一个线程中声明了两个 `ThreadLocal` 对象的话，会使用 `Thread`内部都是使用仅有那个`ThreadLocalMap` 存放数据的，`ThreadLocalMap`的 key 就是 `ThreadLocal`对象，value 就是 `ThreadLocal` 对象调用`set`方法设置的值。**ThreadLocal是 map结构是为了让每个线程可以关联多个 ThreadLocal变量。**这也就解释了 ThreadLocal 声明的变量为什么在每一个线程都有自己的专属本地变量。

如上所示，大部分解释已经在代码中做出，注意注释1处，得到 map 对象之后，用的 this 作为 key，this 在这里代表的是当前线程的 ThreadLocal 对象。 另外就是第二句根据 getMap 获取一个 ThreadLocalMap，其中getMap 中传入了参数 t (当前线程对象)，这样就能够获取每个线程的 ThreadLocal 了。

1. 先调用Thread类的静态方法获**得当前线程的Thread对象**，**每个线程对应的Thread对象都有一个ThreadLocalMap对象**的引用；

1. 获得当前线程的ThreadLocalMap对象；

1. 如果不为空就调用set方法，如果为空就调用createMap方法，传入参数为ThreadLocalMap为空的Thread对象和T类型的firstValue。

2.ThreadLocalMap

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/20190702135944.png)

ThreadLocalMap 是 ThreadLocal 的一个**内部类**，**用Entry数组来存储键值对，key是ThreadLocal对象，value则是具体的值**。值得一提的是，为了方便GC，Entry继承了WeakReference，也就是弱引用。**定义了一个哈希表**用于为每个线程都提供一个变量的副本。

```java
static class ThreadLocalMap {
 // Entry类继承了WeakReference<ThreadLocal<?>>
 // 即每个Entry对象都有一个ThreadLocal的弱引用（作为key），这是为了防止内存泄露。
 // 一旦线程结束，key变为一个不可达的对象，这个Entry就可以被GC了。
 static class Entry extends WeakReference<ThreadLocal<?>> {
     /** The value associated with this ThreadLocal. */
     Object value;
     Entry(ThreadLocal<?> k, Object v) {
         super(k);
         value = v;
     }
 }
 // ThreadLocalMap 的初始容量，必须为2的倍数
 private static final int INITIAL_CAPACITY = 16;

 // resized时候需要的table
 private Entry[] table;

 // table中的entry个数
 private int size = 0;

 // 扩容数值
 private int threshold; // Default to 0
 }
```

一起看一下其常用的构造函数：

```java
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
table = new Entry[INITIAL_CAPACITY];
int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
table[i] = new Entry(firstKey, firstValue);
size = 1;
setThreshold(INITIAL_CAPACITY);
}
```

构造函数的第一个参数就是本 ThreadLocal 实例 (this)，第二个参数就是要保存的线程本地变量。构造函数首先创建一个长度为16的 **Entry 数组**，然后计算出 firstKey 对应的哈希值，然后存储到 table 中，并设置 size 和 threshold。

3.ThreadLocalMap#set

ThreadLocal 中 put 函数最终调用了 ThreadLocalMap 中的 set 函数，跟进去看一看：

```java
private void set(ThreadLocal<?> key, Object value) {
Entry[] tab = table;
int len = tab.length;
int i = key.threadLocalHashCode & (len-1);

for (Entry e = tab[i];
     e != null;
     // 冲突了
     e = tab[i = nextIndex(i, len)]) {
    ThreadLocal<?> k = e.get();

    if (k == key) {
        e.value = value;
        return;
    }

    if (k == null) {
        replaceStaleEntry(key, value, i);
        return;
    }
}

tab[i] = new Entry(key, value);
int sz = ++size;
if (!cleanSomeSlots(i, sz) && sz >= threshold)
    rehash();
}
```

在上述代码中如果 Entry 在存放过程中冲突了，调用 nextIndex 来处理，如下所示。是否还记得 hashmap 中对待冲突的处理？这里好像是另一种套路：**只要 i 的数值小于 len，就加1取值**，官方术语称为：**线性探测法**。

4.ThreadLocal#get

看完了 set 函数，肯定是要关注 get 的，源码如下所示：

```java
public T get() {
// 获取Thread对象t
Thread t = Thread.currentThread();
// 获取t中的map
ThreadLocalMap map = getMap(t);
if (map != null) {
    ThreadLocalMap.Entry e = map.getEntry(this);
    if (e != null) {
        @SuppressWarnings("unchecked")
        T result = (T)e.value;
        return result;
    }
}
// 如果t中的map为空
return setInitialValue();
}
```

**如果不设置 ThreadLocal 的数值，默认就是 null**，来自于此。

1. 先获取当前线程的**thread对象**，再获取thread对象的t**hreadLocalMap对象**，然后根据当前的threadLocal对象取得**table数组对应下标的Entry对象**；

1. 如果Thread对象的ThreadLocalMap为空的话，就调用setInitialValue方法，**该方法初始化map并且放入null** ( initialValue的返回值为null )，可以通过覆盖该方法修改没有set时的初始值。

   总结：

   在插入过程中，根据ThreadLocal对象的hash值，定位到table中的位置i，过程如下：
   1、如果当前位置是空的，那么正好，**就初始化一个Entry对象放在位置i上；**
   2、不巧，位置i已经有Entry对象了，**如果这个Entry对象的key正好是即将设置的key，那么重新设置Entry中的value**；
   3、很不巧，位置i的Entry对象，和即将设置的key没关系，那么只能找下一个空位置
   作者：占小狼链接：https://www.jianshu.com/p/377bb840802f来源：简书
   
### 总结

1. 每个Thread维护着**一个ThreadLocalMap的引用**
2. ThreadLocalMap是ThreadLocal的内部类，**用Entry来进行存储**
3. 调用ThreadLocal的set()方法时，实际上就是往ThreadLocalMap设置值，key是ThreadLocal对象，值是传递进来的对象
4. 调用ThreadLocal的get()方法时，实际上就是往ThreadLocalMap获取值，**key是ThreadLocal对象**
5. **ThreadLocal本身并不存储值**，它只是**作为一个key来让线程从ThreadLocalMap获取value**。

 https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484118&idx=1&sn=da3e4c4cfd0642687c5d7bcef543fe5b&chksm=ebd743d7dca0cac19a82c7b29b5b22c4b902e9e53bd785d066b625b4272af2a6598a0cc0f38e&scene=21##wechat_redirect

# 三、作用

　Synchronized用于线程间的**数据共享**，而ThreadLocal则用于线程间的**数据隔离**。

# 四、适用场景：

(1)  当**很多线程需要多次使用同一个对象**，并且**需要该对象具有相同初始化值**；

(2)  适用于资源共享但不需要维护状态的情况，也就是一个线程对资源的修改，不影响另一个线程的运行；

(3) 基于ThreadLocal实现线程安全是采用"空间换时间"，synchronized顺序执行是"时间换取空间"。

# 五、ThreadLocal使用注意事项

1、ThreadLocal对象是一个弱引用

ThreadLocalMap中的**节点Entry继承了WeakReference类**，定义了一个类型为Object的value，用于存放塞到ThreadLocal里的值。

如果这里使用普通的key-value形式来定义存储结构，实质上就会造成**节点的生命周期与线程强绑定，只要线程没有销毁，那么节点在GC分析中一直处于可达状态，没办法被回收，而程序本身也无法判断是否可以清理节点。**

ThreadLocal对象是**一个继承自WeakReference的弱引用**，当把ThreadLocal的实例置为空以后，没有任何强引用指向ThreadLocal的实例，所以ThreadLocal的将会被GC回收。

生命周期只存活到下次GC前，可降低内存泄漏的风险。

### ThreadLocal 内存泄露问题

`ThreadLocalMap` 中使用的 key 为 `ThreadLocal` 的弱引用,而 value 是强引用。所以，如果 `ThreadLocal` 没有被外部强引用的情况下，在垃圾回收的时候会 key 会被清理掉，而 value 不会被清理掉。**这样一来，`ThreadLocalMap` 中就会出现key为null的Entry。假如我们不做任何措施的话，value 永远无法被GC 回收，这个时候就可能会产生内存泄露。**ThreadLocalMap实现中已经考虑了这种情况，在调用 `set()`、`get()`、`remove()`方法的时候，会清理掉 key 为 null 的记录。**使用完 `ThreadLocal`方法后 最好手动调用`remove()`方法**

ThreadLocal内存泄漏的根源是：**由于ThreadLocalMap的生命周期跟Thread一样长，如果没有手动删除对应key就会导致内存泄漏，而不是因为弱引用**。

想要避免内存泄露就要**手动remove()掉**！

```java
      static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
```

ThreadLocal对象是具有弱引用特点，虽然在一定程度上降低了内存泄漏的风险，但是在有线程复用如线程池的场景中，一个线程的寿命很长，大对象长期不被回收影响系统运行效率与安全，那么**一条强引用链的关系一直存在**：

Thread --> ThreadLocalMap-->Entry-->Value，

最终造成内存泄漏。

**如何避免内存泄漏**：

**调用ThreadLocal的get()、set()方法时完成后再调用remove方法**，**将Entry节点和Map的引用关系移除**，这样整个Entry对象在GC Roots分析后就变成不可达了，下次GC的时候就可以被回收。

3、哈希冲突怎么解决

ThreadLocalMap中解决哈希冲突的方式并非链表的方式，而是采用**线性探测**的方式，具体来说，就是简单的步长加1或减1，寻找下一个相邻的位置。

https://mp.weixin.qq.com/s/Jk5bfG2RozFcQ9acAmCE-g

### 为什么要弱引用

读到这里，如果不问不答为什么是这样的定义形式，为什么要用弱引用，等于没读懂源码。
因为如果这里使用普通的key-value形式来定义存储结构，**实质上就会造成节点的生命周期与线程强绑定，只要线程没有销毁，那么节点在GC分析中一直处于可达状态，没办法被回收，而程序本身也无法判断是否可以清理节点。**弱引用是Java中四档引用的第三档，比软引用更加弱一些，如果一个对象没有强引用链可达，那么一般活不过下一次GC。当某个ThreadLocal已经没有强引用可达，则随着它被垃圾回收，在ThreadLocalMap里对应的Entry的键值会失效，这为ThreadLocalMap本身的垃圾清理提供了便利。

# Demo

通过set(T)方法设置一个值，在当前线程下通过get()方法获取到原先设置的值。

```java
public class Profiler{
	//第一次get()方法调用会进行初始化（如果set方法没有调用），每个线程会调用一次
    private static final ThreadLocal<Long> TIME_THREADLOCAL = new ThreadLocal<Long>(){
        protected Long initialValue(){
            return System.currentTimeMillis();
        }
    };
    
    public static final void begin(){
        TIME_THREADLOCAL.set(System.currentTimeMillis());
    }
    
    public static final long end(){
        return System.currentTimeMills() - TIME_THREADLOCAL.get();
    }
    
    public static void main(String[] args) throws Exception{
        Profiler.begin();
        TimeUnit.SECONDS.sleep(1);
        System.out.printIn("Cost: " + Profiler.end() + " mills");
    }
}
```

