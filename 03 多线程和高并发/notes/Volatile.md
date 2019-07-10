# 面试题

## 1.Unsafe类详解一下（360企业安全）

## 2.CAS中unsafe调的哪个方法（360企业安全）

compareAndSwapInt()

## 3. 说说 synchronized 关键字和 volatile 关键字的区别

- **volatile关键字**是线程同步的**轻量级实现**，所以**volatile性能肯定比synchronized关键字要好**。但是**volatile关键字只能用于变量而synchronized关键字可以修饰方法以及代码块**。synchronized关键字在JavaSE1.6之后进行了主要包括为了减少获得锁和释放锁带来的性能消耗而引入的偏向锁和轻量级锁以及其它各种优化之后执行效率有了显著提升，**实际开发中使用 synchronized 关键字的场景还是更多一些**。
- **多线程访问volatile关键字不会发生阻塞，而synchronized关键字可能会发生阻塞**
- **volatile关键字能保证数据的可见性，但不能保证数据的原子性。synchronized关键字两者都能保证。**
- **volatile关键字主要用于解决变量在多个线程之间的可见性，而 synchronized关键字解决的是多个线程之间访问资源的同步性。**

## 4. volatile的原理

 volatile关键字保证了多线程环境下变量的可见性与有序性，**底层实现基于内存屏障（Memory Barrier）**。为了优化性能，现代CPU工作时的指令执行顺序与应用程序的代码顺序其实是不一致的（有些编译器也会进行这种优化），也就是所谓的乱序执行技术。乱序执行可以提高CPU流水线的工作效率，只要保证数据符合程序逻辑上的正确性即可（遵循happens-before原则）。不过如今是多核时代，如果随便乱序而不提供防护措施那是会出问题的。每一个cpu上都会进行乱序优化，单cpu所保证的逻辑次序可能会被其他cpu所破坏。内存屏障就是针对此情况的防护措施。可以认为它是一个同步点（但它本身也是一条cpu指令）。例如在IA32指令集架构中引入的SFENCE指令，在该指令之前的所有写操作必须全部完成，读操作仍可以乱序执行。LFENCE指令则保证之前的所有读操作必须全部完成，另外还有粒度更粗的MFENCE指令保证之前的所有读写操作都必须全部完成。内存屏障就像是一个保护指令顺序的栅栏，保护后面的指令不被前面的指令跨越。将内存屏障插入到写操作与读操作之间，就可以保证之后的读操作可以访问到最新的数据，因为屏障前的写操作已经把数据写回到内存（根据缓存一致性协议，不会直接写回到内存，而是改变该cpu私有缓存中的状态，然后通知给其他cpu这个缓存行已经被修改过了，之后另一个cpu在读操作时就可以发现该缓存行已经是无效的了，这时它会从其他cpu中读取最新的缓存行，然后之前的cpu才会更改状态并写回到内存）。

https://www.cnblogs.com/zyrblog/p/9881958.html


# 谈谈你对volatile的理解

## 1.volatile是JVM提供的轻量级的同步机制

### 1.1 保证可见性

一个线程修改数据后，其他线程可以知道。

原理是在**每次访问变量时都会进行一次刷新**，因此每次访问都是**主内存中最新的版本**。所以 volatile 关键字的作用之一就是保证变量修改的实时可见性。

### 1.2 不保证原子性

原子性：不可分割，完整性，某个线程做某个业务时，中间不可以被加塞或者被分割。要么同时成功，要么同时失败。

#### 如何解决

- 加synchronized
- **使用juc下的AtomicInteger**(CAS)

### 1.3 禁止指令重排

## 2.JMM谈谈

线程对变量的操作必须在工作内存中进行，首先将变量从主内存拷贝到自己的工作内存，然后对变量进行操作，操作完成后再将变量写回主内存。

### 2.1 可见性

### 2.2 原子性

### 2.3 有序性

## 3.你在哪里用到过volatile

- 双端检索机制（DLC）
- 状态标记量

## 4.volatile 与 synchronized 的比较

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








