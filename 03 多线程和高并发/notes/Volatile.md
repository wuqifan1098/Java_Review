# 面试题

## 1.Unsafe类详解一下（360企业安全）

Unsafe为我们提供了访问底层的机制，这种机制仅供java核心类库使用

（1）实例化一个类；

（2）修改私有字段的值；

（3）抛出checked异常；

（4）使用堆外内存；

（5）CAS操作；

（6）阻塞/唤醒线程；

## 2.CAS中unsafe调的哪个方法（360企业安全）

compareAndSwapInt()

## 3. 说说 synchronized 关键字和 volatile 关键字的区别

- **volatile关键字**是线程同步的**轻量级实现**，所以**volatile性能肯定比synchronized关键字要好**。但是**volatile关键字只能用于变量而synchronized关键字可以修饰方法以及代码块**。synchronized关键字在JavaSE1.6之后进行了主要包括为了减少获得锁和释放锁带来的性能消耗而引入的偏向锁和轻量级锁以及其它各种优化之后执行效率有了显著提升，**实际开发中使用 synchronized 关键字的场景还是更多一些**。
- **多线程访问volatile关键字不会发生阻塞，而synchronized关键字可能会发生阻塞**
- **volatile关键字能保证数据的可见性，但不能保证数据的原子性。synchronized关键字两者都能保证。**
- **volatile关键字主要用于解决变量在多个线程之间的可见性，而 synchronized关键字解决的是多个线程之间访问资源的同步性。**

## 4. volatile的原理

 volatile关键字保证了多线程环境下变量的可见性与有序性，**底层实现基于内存屏障（Memory Barrier）**。为了优化性能，现代**CPU工作时的指令执行顺序与应用程序的代码顺序其实是不一致的**（有些编译器也会进行这种优化），也就是所谓的乱序执行技术。

乱序执行可以提高CPU流水线的工作效率，只要**保证数据符合程序逻辑上的正确性**即可（遵循happens-before原则）。不过如今是多核时代，如果随便乱序而不提供防护措施那是会出问题的。每一个cpu上都会进行乱序优化，单cpu所保证的逻辑次序可能会被其他cpu所破坏。内存屏障就是针对此情况的防护措施。可以认为它是一个同步点（但它本身也是一条cpu指令）。例如在IA32指令集架构中引入的SFENCE指令，在该指令之前的所有写操作必须全部完成，读操作仍可以乱序执行。LFENCE指令则保证之前的所有读操作必须全部完成，另外还有粒度更粗的MFENCE指令保证之前的所有读写操作都必须全部完成。**内存屏障就像是一个保护指令顺序的栅栏，保护后面的指令不被前面的指令跨越。**

**将内存屏障插入到写操作与读操作之间，就可以保证之后的读操作可以访问到最新的数据**，因为屏障前的写操作已经把数据写回到内存（根据缓存一致性协议，不会直接写回到内存，而是改变该cpu私有缓存中的状态，然后通知给其他cpu这个缓存行已经被修改过了，之后另一个cpu在读操作时就可以发现该缓存行已经是无效的了，这时它会从其他cpu中读取最新的缓存行，然后之前的cpu才会更改状态并写回到内存）。

https://www.cnblogs.com/zyrblog/p/9881958.html

## 5 谈谈volatile （海康）

volatile是一种**JVM提供的轻量级的同步机制**，不会阻塞线程。Java内存模型告诉我们，各个线程会从将共享变量从主内存拷贝到工作内存，然后基于工作内存中的数据进行操作。线程对volatile修饰变量的**修改会立刻被其他线程锁感知**，不会出现数据脏读的现象，可以**保证可见性**。对任意**单个volatile变量的读/写具有原子性**，但类似于volatile++这种**复合操作不具有原子性。**

https://www.cnblogs.com/getting-better/p/3245993.html

**一方面是因为synchronized是一种锁机制，存在阻塞问题和性能问题，而volatile并不是锁，所以不存在阻塞和性能问题。**

另外一方面，因为**volatile借助了内存屏障来帮助其解决可见性和有序性问题，而内存屏障的使用还为其带来了一个禁止指令重排的附件功能，所以在有些场景中是可以避免发生指令重排的问题的。**

<https://mp.weixin.qq.com/s?__biz=MzI3NzE0NjcwMg==&mid=2650124577&idx=1&sn=4f471855e58d2ded0c9044a3a7e9c698&chksm=f36bac00c41c25166ec3ca43e5811984e4977e8cbbd5f14947fb4d0bc9c67b4a80c726699ff2&mpshare=1&scene=1&srcid=&sharer_sharetime=1566227411677&sharer_shareid=6bdaaaa7a7186e9db8bed8df0280488e&key=0d0f56805bab0f0bffbb452991d4089e22eca92e0394df0320e46cb8951a1521f68614aa3b3e18e18472614c2f06903a9abeaeece5a8fd9d84c6774c0176d02b55fda8541641aee7ac3a32a40e8ea5a7&ascene=1&uin=NjQwMDg5ODE2&devicetype=Windows+10&version=62060833&lang=zh_CN&pass_ticket=GyAV0HglPjKSSS6TWDUd4kfc2fKq6HQ%2Bovbj%2B75KzMzPFzV88qAK6%2Fda65%2FMv6J1>

## 6. volatile为什么不保证原子性?

Java中只有对**基本类型变量的赋值和读取是原子操作**，如i = 1的赋值操作，但是像j = i或者i++这样的操作都不是原子操作，因为他们都**进行了多次原子操作**，比如先读取i的值，再将i的值赋值给j，两个原子操作加起来就不是原子操作了。

所以，如果一个变量被volatile修饰了，那么肯定可以保证每次读取这个变量值的时候得到的值是最新的，但是一旦需要对变量进行**自增这样的非原子操作，就不会保证这个变量的原子性了。**

原文链接：https://blog.csdn.net/xdzhouxin/article/details/81236356

## 7. 为什么原子类可以保证原子性？

原子类基于CAS操作。

因为CAS是**基于乐观锁的，也就是说当写入的时候，如果寄存器旧值已经不等于现值，说明有其他CPU在修改，那就继续尝试。所以这就保证了操作的原子性。**

# 1.volatile是JVM提供的轻量级的同步机制

## 1.1 保证可见性

一个线程修改数据后，其他线程可以知道。

原理是在**每次访问变量时都会进行一次刷新**，因此每次访问都是**主内存中最新的版本**。所以 volatile 关键字的作用之一就是保证变量修改的实时可见性。

#### 内存屏障

内存屏障（[memory barrier](http://en.wikipedia.org/wiki/Memory_barrier)）是一个CPU指令。基本上，它是这样一条指令： **a) 确保一些特定操作执行的顺序； b) 影响一些数据的可见性(可能是某些指令执行后的结果)。**编译器和CPU可以在保证输出结果一样的情况下对指令重排序，使性能得到优化。插入一个内存屏障，相当于**告诉CPU和编译器先于这个命令的必须先执行，后于这个命令的必须后执行**。内存屏障另一个作用是强制更新一次不同CPU的缓存。例如，一个写屏障会把这个屏障前写入的数据刷新到缓存，这样任何试图读取该数据的线程将得到最新值，而不用考虑到底是被哪个cpu核心或者哪颗CPU执行的。

内存屏障（[memory barrier](http://en.wikipedia.org/wiki/Memory_barrier)）和volatile什么关系？上面的虚拟机指令里面有提到，如果你的字段是volatile，Java内存模型将在写操作后插入一个写屏障指令，在读操作前插入一个读屏障指令。这意味着如果你对一个volatile字段进行写操作，你必须知道：1、一旦你完成写入，任何访问这个字段的线程将会得到最新的值。2、在你写入前，会保证所有之前发生的事已经发生，并且任何更新过的数据值也是可见的，因为内存屏障会把之前的写入值都刷新到缓存。

## 1.2 不保证原子性

原子性：不可分割，完整性，某个线程做某个业务时，中间不可以被加塞或者被分割。要么同时成功，要么同时失败。



#### 如何解决

- 加synchronized
- **使用juc下的AtomicInteger**(CAS)

## 1.3 禁止指令重排

# 2.JMM谈谈

线程对变量的操作必须在工作内存中进行，首先将变量从主内存拷贝到自己的工作内存，然后对变量进行操作，操作完成后再将变量写回主内存。

### 2.1 可见性

### 2.2 原子性

### 2.3 有序性

# 3.你在哪里用到过volatile

- 双端检索机制（DLC）
- 状态标记量

# 4.volatile 与 synchronized 的比较

- volatile**轻量级**，只能**修饰变量**。synchronized**重量级**，还可**修饰方法**。
- volatile只能**保证数据的可见性，不能用来同步**，因为多个线程并发访问volatile修饰的变量不会阻塞。
- synchronized不仅**保证可见性**，而且还**保证原子性**

# CAS

判断内存的某个位置的值是否为预期值，如果是则更新，是CPU指令级的操作，这个过程是原子的。

## CAS原理？ 谈谈Unsafe

JAVA中的CAS操作都是通过sun包下Unsafe类实现，而**Unsafe类中的方法都是native方法，由JVM本地实现，是用C++写的。**CAS是**CPU指令级的操作，只有一步原子操作，所以非常快**。

getAndIncrement的内部：

```java
public final int getAndIncrement() {
return unsafe.getAndAddInt(this, valueOffset, 1);}
```

深入到getAndAddInt():
    
```java
public final int getAndAddInt(Object var1, long var2, int var4) {
int var5;
do {
    var5 = this.getIntVolatile(var1, var2);
} while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

return var5;}
```

首先我们发现compareAndSwapInt前面的this，那么它属于哪个类呢，我们看上一步getAndAddInt，前面是unsafe。这里我们进入的Unsafe类。这里要对**Unsafe类做个说明**。结合AtomicInteger的定义来说：

```java
public class AtomicInteger extends Number implements java.io.Serializable {
private static final long serialVersionUID = 6214790243416807050L;

// setup to use Unsafe.compareAndSwapInt for updates
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long valueOffset;

static {
    try {
        valueOffset = unsafe.objectFieldOffset
            (AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}

private volatile int value;
...
```

（１）从 AtomicInteger 的内部属性可以看出，它**依赖于Unsafe类获得对象内存地址访问**，偏移量valueOffset代表的该变量值在内存中的偏移地址，从而获取数据的。

（２）变量value用volatile修饰，**保证了多线程之间的内存可见性，当前线程可以拿到value最新的值**。

（３）CAS操作保证了AtomicInteger 可以安全的修改value 的值。

在AtomicInteger数据定义的部分，我们可以看到，其实**实际存储的值是放在value中的，除此之外我们还获取了unsafe实例，并且定义了valueOffset。**再看到static块，懂类加载过程的都知道，static块的加载发生于类加载的时候，是最先初始化的，这时候我们调用unsafe的objectFieldOffset从Atomic类文件中获取value的偏移量，那么valueOffset其实就是**记录value的偏移量的**。   

再回到上面一个函数getAndAddInt，我们看var5获取的是什么，通过调用unsafe的getIntVolatile(var1, var2)，这是个native方法，具体实现到JDK源码里去看了，其实就是获取**var1中，var2偏移量处的值。var1就是AtomicInteger，var2就是我们前面提到的valueOffset**,这样我们就从内存里获取到现在valueOffset处的值了。

现在重点来了，**compareAndSwapInt**（var1, var2, var5, var5 + var4）其实换成compareAndSwapInt（obj, offset, expect, update）比较清楚，意思就是如果obj内的value和expect相等，就证明没有其他线程改变过这个变量，那么就更新它为update，**如果这一步的CAS没有成功，那就采用自旋的方式继续进行CAS操作**，取出乍一看这也是两个步骤了啊，其实在JNI里是借助于一个CPU指令完成的。所以还是原子操作。

### 应用场景

- AtomicInteger**提供原子操作来进行Integer的使用**，通过线程安全的方式操作加减。

- AtomicInteger是在使用非阻塞算法实现并发控制，适合一些高并发场景

## CAS简单小结

Unsafe类+CAS思想（自旋）

## CAS应用

CAS有3个操作数，内存值V，旧的预期值A，要修改的更新值B。
当且仅当预期值A和内存值V相同时，将内存值V修改为B并返回true，否则什么都不做,并返回false。

## CAS缺点

### ABA问题

CAS需要在操作值的时候检查下**值有没有发生变化**，如果没有发生变化则更新，但是如果一个值**原来是A，变成了B，又变成了A**，那么使用CAS进行检查时会发现它的值没有发生变化，但是**实际上却变化**了。
**只管结果，不管过程。**

### 循环时间长开销大

**自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。**如果 JVM 能支持处理器提供的pause指令那么效率会有一定的提升，pause指令有两个作用，第一它可以延迟流水线执行指令（de-p ip eline）,使CPU不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零。第二它可以避免在退出循环的时候因内存顺序冲突（memory order violation）而引起CPU流水线被清空（CPU pipeline flush），从而提高CPU的执行效率。

### 只能保证一个共享变量的原子操作

当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是**对多个共享变量操作时，循环CAS就无法保证操作的原子性**。

### 原子引用

AtomicReference

### 时间戳原子引用

AtomicStampedReference

#### ABADemo

```java
public class ABADemo {
static AtomicReference<Integer> atomicReference = new AtomicReference<>(100);
static AtomicStampedReference<Integer> atomicStampedReference = new AtomicStampedReference<>(100,1);
public static void main(String[] args) {
    System.out.println("=========以下是ABA问题的解决===========");

    new Thread(() ->{
        int stamp = atomicStampedReference.getStamp();
        System.out.println(Thread.currentThread().getName()+"\t第1次版本号： "+stamp);
        try{
            TimeUnit.SECONDS.sleep(1);}catch (InterruptedException e){e.printStackTrace();}
            //为了t4拿到一样的版本号
            atomicStampedReference.compareAndSet(100,101,atomicStampedReference.getStamp(),atomicStampedReference.getStamp()+1);
            System.out.println(Thread.currentThread().getName()+"\t第2次版本号： "+atomicStampedReference.getStamp());
            atomicStampedReference.compareAndSet(101,100,atomicStampedReference.getStamp(),atomicStampedReference.getStamp()+1);
            System.out.println(Thread.currentThread().getName()+"\t第3次版本号： "+atomicStampedReference.getStamp());
    },"t3").start();

    new Thread(() ->{
        int stamp = atomicStampedReference.getStamp();
        System.out.println(Thread.currentThread().getName()+"\t第1次版本号： "+stamp);
        //暂停3秒t4线程，保证上面t3完成一次ABA操作
        try{
            TimeUnit.SECONDS.sleep(3);}catch (InterruptedException e){e.printStackTrace();}
            boolean result = atomicStampedReference.compareAndSet(100,2019,stamp,stamp+1);

        System.out.println(Thread.currentThread().getName() + result + "\t当前实际版本号： " + atomicStampedReference.getStamp());
    },"t4").start();

}
}
```








