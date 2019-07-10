# 1.简介

ReentrantLock： 

与synchronized不同，ReentrantLock是纯JAVA实现的，与底层JVM没有直接关联。显式锁ReentrantLock和同步工具类的实现基础都是**AQS**，AQS的实现基础是CAS，**通过实现一种自旋锁，循环调用CAS操作来实现加锁。**

# 2.原理

## 2.1 与 synchronized 的异同

ReentrantLock 和 synchronized 都是用于线程的同步控制，但它们在功能上来说差别还是很大的。对比下来 ReentrantLock 功能明显要丰富的多。

| 特性       | synchronized	 | ReentrantLock |
|----------|---------------|---------------|
| 可重入	     | 是	            | 是             |
| 响应中断     | 否             | 是             |
| 超时等待     | 否             | 是             |
| 公平锁      | 否             | 是             |
| 非公平锁     | 是             | 是             |
| 自动获取/释放锁 | 是             | 否             |
| 对异常的处理   | 自动释放锁	        | 需手动释放锁        |

**分析比较**

**相同点：**

（1）都是**可重入锁**，同一线程可以多次获得同一个锁

（2）都保证了**可见性和互斥性**

**不同点：**

（1）ReentrantLock是API级别的，Synchronized是JVM级别的；

（2）ReentrantLock是**显示的获得和释放锁**，Synchronized**隐式的获得和释放锁；**

（3）ReentrantLock**可以实现公平锁**，而Synchronized**只能是非公平锁；**

（4）ReentrantLock**可响应中断**，通过lock.lockInterruptibly()来实现这个机制；

（5）ReentrantLock通过Condition**可以绑定多个条件**，可用来实现分组唤醒需要唤醒的线程。

## 2.2 常用方法

（1）ReentrantLock 分为公平锁和非公平锁，可以通过构造方法来指定具体类型：

 public ReentrantLock()   默认非公平锁

 public ReentrantLock(boolean fair)  可实现公平锁，fair为true，公平锁


（2）锁操作

 public void lock()    lock方法中调用sync的lock方法获取锁

 public boolean tryLock()   尝试获取锁，成功则直接返回true，不成功立即返回false

 public void unlock()   调用sync的release方法释放锁

（3）获取Condition

public Condition newCondition()  ReentrantLock支持多个Condition

## 1.前言

在[ReentrantLock(重入锁)功能详解和应用演示](https://www.cnblogs.com/takumicx/p/9338983.html)这篇文章里我们讲解并演示了ReentrantLock(重入锁)的各种功能,其中就谈到ReentrantLock可以有公平锁和非公平锁的不同实现,只要在**构造它的时候传入不同的布尔值**,继续跟进下源码我们就能发现,关键在于实例化内部变量`sync`的方式不同,如下所示

```java
/**
 * Creates an instance of {@code ReentrantLock} with the
 * given fairness policy.
 *
 * @param fair {@code true} if this lock should use a fair ordering policy
 */
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

公平锁内部是FairSync,非公平锁内部是NonfairSync。而不管是FairSync还是NonfariSync,都间接继承自AbstractQueuedSynchronizer这个抽象类，如下图所示

- NonfairSync的类继承关系
  ![img](https://images2018.cnblogs.com/blog/1422237/201808/1422237-20180805183109489-1524063793.png)
- FairSync的类继承关系
  ![img](https://images2018.cnblogs.com/blog/1422237/201808/1422237-20180805183410668-314295960.png)

该抽象类为我们的加锁和解锁过程提供了统一的模板方法，只是一些细节的处理由该抽象类的实现类自己决定。所以在解读ReentrantLock(重入锁)的源码之前,有必要了解下AbstractQueuedSynchronizer。

## 2.AbstractQueuedSynchronizer介绍

### 2.1 AQS是构建同步组件的基础

AbstractQueuedSynchronizer,简称AQS,为构建不同的同步组件(重入锁,读写锁,CountDownLatch等)提供了可扩展的基础框架,如下图所示。
![img](https://images2018.cnblogs.com/blog/1422237/201808/1422237-20180805183454325-124283154.png)

AQS以模板方法模式在**内部定义了获取和释放同步状态的模板方法,并留下钩子函数供子类继承时进行扩展**,由子类决定在获取和释放同步状态时的细节,从而实现满足自身功能特性的需求。除此之外,AQS**通过内部的同步队列管理获取同步状态失败的线程,向实现者屏蔽了线程阻塞和唤醒的细节**。

### 2.2 AQS的内部结构(ReentrantLock的语境下)

AQS的内部结构主要由同步等待队列构成

#### 2.2.1 同步等待队列

AQS中同步等待队列的实现是**一个带头尾指针(这里用指针表示引用是为了后面讲解源码时可以更直观形象,况且引用本身是一种受限的指针)且不带哨兵结点(后文中的头结点表示队列首元素结点,不是指哨兵结点)的双向链表**。

```java
/**
 * Head of the wait queue, lazily initialized.  Except for
 * initialization, it is modified only via method setHead.  Note:
 * If head exists, its waitStatus is guaranteed not to be
 * CANCELLED.
 */
private transient volatile Node head;//指向队列首元素的头指针

/**
 * Tail of the wait queue, lazily initialized.Modified only via
 * method enq to add new wait node.
 */
private transient volatile Node tail;//指向队列尾元素的尾指针
```

**head是头指针,指向队列的首元素;tail是尾指针,指向队列的尾元素**。而队列的元素结点Node定义在AQS内部,主要有如下几个成员变量

```java
volatile Node prev; //指向前一个结点的指针

volatile Node next; //指向后一个结点的指针
volatile Thread thread; //当前结点代表的线程
volatile int waitStatus; //等待状态
```

- prev:指向前一个结点的指针
- next:指向后一个结点的指针
- thread:当前结点表示的线程,因为同步队列中的结点内部封装了之前竞争锁失败的线程,故而结点内部必然有一个对应线程实例的引用
- waitStatus:对于重入锁而言,主要有3个值。0:初始化状态；-1(SIGNAL):当前结点表示的线程在释放锁后需要唤醒后续节点的线程;1(CANCELLED):在同步队列中等待的线程等待超时或者被中断,取消继续等待。

------

同步队列的结构如下图所示
![img](https://images2018.cnblogs.com/blog/1422237/201808/1422237-20180805183911414-329333235.png)

为了接下来能够更好的理解加锁和解锁过程的源码,对该同步队列的特性进行简单的讲解:

- 1.同步队列是个先进先出(FIFO)队列,获取锁失败的线程将构造结点并加入队列的尾部,并阻塞自己。如何才能线程安全的实现入队是后面讲解的重点,毕竟我们在讲锁的实现,这部分代码肯定是不能用锁的。
- 2.**队列首结点可以用来表示当前正获取锁的线程**。
- 3.当前线程释放锁后将尝试**唤醒后续处结点中处于阻塞状态的线程**。

为了加深理解,还可以在阅读源码的过程中思考下这个问题：

这个同步队列是FIFO队列,也就是说先在队列中等待的线程将比后面的线程更早的得到锁,那ReentrantLock是如何基于这个FIFO队列实现非公平锁的？

#### 2.2.2 AQS中的其他数据结构(ReentrantLock的语境下)

- 同步状态变量

```java
/**
 * The synchronization state.
 */
private volatile int state;
```

这是一个带volatile前缀的int值,是一个类似计数器的东西。在不同的同步组件中有不同的含义。以ReentrantLock为例,state可以用来表示该锁被线程重入的次数。**当state为0表示该锁不被任何线程持有;当state为1表示线程恰好持有该锁1次(未重入);当state大于1则表示锁被线程重入state次**。因为这是一个会被并发访问的量,为了防止出现可见性问题要用volatile进行修饰。

- 持有同步状态的线程标志

```java
/**
 * The current owner of exclusive mode synchronization.
 */
private transient Thread exclusiveOwnerThread;
```

如注释所言,这是在独占同步模式下标记持有同步状态线程的。ReentrantLock就是典型的独占同步模式,该变量用来标识锁被哪个线程持有。

------

了解AQS的主要结构后,就可以开始进行ReentrantLock的源码解读了。由于非公平锁在实际开发中用的比较多,故以讲解非公平锁的源码为主。以下面这段对非公平锁使用的代码为例:

```java
/**
 * @author: takumiCX
 * @create: 2018-08-01
 **/
public class NoFairLockTest {


    public static void main(String[] args) {

        //创建非公平锁
        ReentrantLock lock = new ReentrantLock(false);

        try {

            //加锁
            lock.lock();

            //模拟业务处理用时
            TimeUnit.SECONDS.sleep(1);

        } catch (InterruptedException e) {
            e.printStackTrace();

        } finally {
            //释放锁
            lock.unlock();
        }

    }
}
```

## 3 非公平模式加锁流程

加锁流程从`lock.lock()`开始

```java
public void lock() {
    sync.lock();
}
```

进入该源码,正确找到sycn的实现类后可以看到真正有内容的入口方法

### 3.1加锁流程真正意义上的入口

```java
/**
 * Performs lock.  Try immediate barge, backing up to normal
 * acquire on failure.
 */
//加锁流程真正意义上的入口
final void lock() {
    //以cas方式尝试将AQS中的state从0更新为1
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());//获取锁成功则将当前线程标记为持有锁的线程,然后直接返回
    else
        acquire(1);//获取锁失败则执行该方法
}
```

首先尝试快速获取锁,**以cas的方式将state的值更新为1,只有当state的原值为0时更新才能成功**,因为state在ReentrantLock的语境下等同于锁被线程重入的次数,这意味着只有当前锁未被任何线程持有时该动作才会返回成功。**若获取锁成功,则将当前线程标记为持有锁的线程,然后整个加锁流程就结束了。若获取锁失败,则执行acquire方法**

```java
/**
 * Acquires in exclusive mode, ignoring interrupts.  Implemented
 * by invoking at least once {@link #tryAcquire},
 * returning on success.  Otherwise the thread is queued, possibly
 * repeatedly blocking and unblocking, invoking {@link
 * #tryAcquire} until success.  This method can be used
 * to implement method {@link Lock#lock}.
 *
 * @param arg the acquire argument.  This value is conveyed to
 *        {@link #tryAcquire} but is otherwise uninterpreted and
 *        can represent anything you like.
 */
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

该方法主要的逻辑都在if判断条件中,这里面有3个重要的方法tryAcquire()，addWaiter()和acquireQueued()，这三个方法中分别封装了加锁流程中的主要处理逻辑，理解了这三个方法到底做了哪些事情，整个加锁流程就清晰了。

### 3.2 尝试获取锁的通用方法:tryAcquire()

tryAcquire是AQS中定义的钩子方法,如下所示

钩子方法就是我们在模板方法中增加了判断标记，然后子类对外提供一个public接口setAlarm来让外界设置这个判断标记，这个判断标记就像是开关一样，**想让它ture和false都行**。这个isAlarm方法俗称钩子方法。有了钩子方法的模板方法模式才算完美，使得我们的控制行为更加的主动，更加的灵活。

```java
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}
```

**该方法默认会抛出异常,强制同步组件通过扩展AQS来实现同步功能的时候必须重写该方法**,ReentrantLock在公平和非公平模式下对此有不同实现,非公平模式的实现如下：

```java
protected final boolean tryAcquire(int acquires) {
    return nonfairTryAcquire(acquires);
}
```

底层调用了nonfairTryAcquire()
从方法名上我们就可以知道这是**非公平模式下尝试获取锁**的方法,具体方法实现如下

```java
/**
 * Performs non-fair tryLock.  tryAcquire is implemented in
 * subclasses, but both need nonfair try for trylock method.
 */
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();//获取当前线程实例
    int c = getState();//获取state变量的值,即当前锁被重入的次数
    if (c == 0) {   //state为0,说明当前锁未被任何线程持有
        if (compareAndSetState(0, acquires)) { //以cas方式获取锁
            setExclusiveOwnerThread(current);  //将当前线程标记为持有锁的线程
            return true;//获取锁成功,非重入
        }
    }
    else if (current == getExclusiveOwnerThread()) { //当前线程就是持有锁的线程,说明该锁被重入了
        int nextc = c + acquires;//计算state变量要更新的值
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);//非同步方式更新state值
        return true;  //获取锁成功,重入
    }
    return false;     //走到这里说明尝试获取锁失败
}
```

这是非公平模式下获取锁的通用方法。它囊括了当前线程在尝试获取锁时的所有可能情况：

- 1.当前锁未被任何线程持有(state=0),则**以cas方式获取锁,若获取成功则设置exclusiveOwnerThread为当前线程**,然后返回成功的结果；若cas失败,**说明在得到state=0和cas获取锁之间有其他线程已经获取了锁,返回失败结果。**
- 2.若锁已经被当前线程获取(state>0,exclusiveOwnerThread为当前线程),**则将锁的重入次数加1(state+1),然后返回成功结果**。因为该线程之前已经获得了锁,所以这个累加操作不用同步。
- 3.若当前锁已经被其他线程持有(state>0,exclusiveOwnerThread不为当前线程),则直接返回失败结果

因为我们**用state来统计锁被线程重入的次数**,所以当前线程尝试获取锁的操作是否成功可以简化为:state值是否成功累加1,是则尝试获取锁成功,否则尝试获取锁失败。

其实这里还可以思考一个问题:nonfairTryAcquire已经实现了一个囊括所有可能情况的尝试获取锁的方式,为何在刚进入lock方法时还要**通过compareAndSetState(0, 1)去获取锁**,毕竟后者只有在锁未被任何线程持有时才能执行成功,我们完全可以把compareAndSetState(0, 1)去掉,对最后的结果不会有任何影响。这种在进行通用逻辑处理之前针对某些特殊情况提前进行处理的方式在后面还会看到,一个直观的想法就是它能提升性能，而代价是牺牲一定的代码简洁性。

退回到上层的acquire方法,

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&  //当前线程尝试获取锁,若获取成功返回true,否则false
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))  //只有当前线程获取锁失败才会执行者这部分代码
        selfInterrupt();
}
```

tryAcquire(arg)返回成功,则说明当前线程成功获取了锁(第一次获取或者重入),由取反和&&可知,整个流程到这结束，只有当前线程获取锁失败才会执行后面的判断。先来看addWaiter(Node.EXCLUSIVE)
部分,这部分代码描述了当线程获取锁失败时如何安全的加入同步等待队列。这部分代码可以说是整个加锁流程源码的精华,充分体现了并发编程的艺术性。

### 3.3 获取锁失败的线程如何安全的加入同步队列:addWaiter()

这部分逻辑在addWaiter()方法中

```java
/**
 * Creates and enqueues node for current thread and given mode.
 *
 * @param mode Node.EXCLUSIVE for exclusive, Node.SHARED for shared
 * @return the new node
 */
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);//首先创建一个新节点,并将当前线程实例封装在内部,mode这里为null
    // Try the fast path of enq; backup to full enq on failure
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    enq(node);//入队的逻辑这里都有
    return node;
}
```

首先创建了一个新节点,并将当前线程实例封装在其内部,之后我们直接看enq(node)方法就可以了,中间这部分逻辑在enq(node)中都有,之所以加上这部分“重复代码”和尝试获取锁时的“重复代码”一样,对某些特殊情况
进行提前处理,牺牲一定的代码可读性换取性能提升。

```java
/**
 * Inserts node into queue, initializing if necessary. See picture above.
 * @param node the node to insert
 * @return node's predecessor
 */
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;//t指向当前队列的最后一个节点,队列为空则为null
        if (t == null) { // Must initialize  //队列为空
            if (compareAndSetHead(new Node())) //构造新结点,CAS方式设置为队列首元素,当head==null时更新成功
                tail = head;//尾指针指向首结点
        } else {  //队列不为空
            node.prev = t;
            if (compareAndSetTail(t, node)) { //CAS将尾指针指向当前结点,当t(原来的尾指针)==tail(当前真实的尾指针)时执行成功
                t.next = node;    //原尾结点的next指针指向当前结点
                return t;
            }
        }
    }
}
```

这里有两个CAS操作:

- compareAndSetHead(new Node()),CAS方式更新head指针,仅当原值为null时更新成功

```java
/**
 * CAS head field. Used only by enq.
 */
private final boolean compareAndSetHead(Node update) {
    return unsafe.compareAndSwapObject(this, headOffset, null, update);
}
```

- compareAndSetTail(t, node),CAS方式更新tial指针,仅当原值为t时更新成功

```java
/**
 * CAS tail field. Used only by enq.
 */
private final boolean compareAndSetTail(Node expect, Node update) {
    return unsafe.compareAndSwapObject(this, tailOffset, expect, update);
}
```

外层的for循环保证了所有获取锁失败的线程经过失败重试后最后都能加入同步队列。因为AQS的同步队列是不带哨兵结点的,故当队列为空时要进行特殊处理,这部分在if分句中。注意当前线程所在的结点不能直接插入
空队列,因为阻塞的线程是由前驱结点进行唤醒的。故先要插入一个结点作为队列首元素,当锁释放时由它来唤醒后面被阻塞的线程,从逻辑上这个队列首元素也可以表示当前正获取锁的线程,虽然并不一定真实持有其线程实例。

首先通过new Node()创建一个空结点，然后以CAS方式让头指针指向该结点(该结点并非当前线程所在的结点),若该操作成功,则将尾指针也指向该结点。这部分的操作流程可以用下图表示
![img](https://images2018.cnblogs.com/blog/1422237/201808/1422237-20180805183957849-785041089.png)

当队列不为空,则执行通用的入队逻辑,这部分在else分句中

```java
else {
            node.prev = t;//step1:待插入结点pre指针指向原尾结点
            if (compareAndSetTail(t, node)) { step2:CAS方式更改尾指针
                t.next = node; //原尾结点next指针指向新的结点
                return t;
            }
        }
```

首先当前线程所在的结点的前向指针pre指向当前线程认为的尾结点,源码中用t表示。然后以CAS的方式将尾指针指向当前结点,该操作仅当tail=t,即尾指针在进行CAS前未改变时成功。若CAS执行成功,则将原尾结点的后向指针next指向新的尾结点。整个过程如下图所示

![img](https://images2018.cnblogs.com/blog/1422237/201808/1422237-20180805184016249-998899089.png)

整个入队的过程并不复杂,是典型的CAS加失败重试的乐观锁策略。其中只有更新头指针和更新尾指针这两步进行了CAS同步,可以预见高并发场景下性能是非常好的。但是本着质疑精神我们不禁会思考下这么做真的线程安全吗？
![img](https://images2018.cnblogs.com/blog/1422237/201808/1422237-20180805184052040-1494856465.png)

- 1.队列为空的情况:
  因为队列为空,故head=tail=null,假设线程执行2成功,则在其执行3之前,因为tail=null,其他进入该方法的线程因为head不为null将在2处不停的失败,所以3即使没有同步也不会有线程安全问题。
- 2.队列不为空的情况:
  假设线程执行5成功,则此时4的操作必然也是正确的(当前结点的prev指针确实指向了队列尾结点,换句话说tail指针没有改变,如若不然5必然执行失败),又因为4执行成功,当前节点在队列中的次序已经确定了,所以6何时执行对线程安全不会有任何影响,比如下面这种情况

![img](https://images2018.cnblogs.com/blog/1422237/201808/1422237-20180805184134587-1898400226.png)

为了确保真的理解了它,可以思考这个问题:把enq方法图中的4放到5之后,整个入队的过程还线程安全吗？

到这为止,获取锁失败的线程加入同步队列的逻辑就结束了。但是线程加入同步队列后会做什么我们并不清楚,这部分在acquireQueued方法中

### 3.4 线程加入同步队列后会做什么:acquireQueued()

先看acquireQueued方法的源码

```java
/**
 * Acquires in exclusive uninterruptible mode for thread already in
 * queue. Used by condition wait methods as well as acquire.
 *
 * @param node the node
 * @param arg the acquire argument
 * @return {@code true} if interrupted while waiting
 */
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        //死循环,正常情况下线程只有获得锁才能跳出循环
        for (;;) {
            final Node p = node.predecessor();//获得当前线程所在结点的前驱结点
            //第一个if分句
            if (p == head && tryAcquire(arg)) { 
                setHead(node); //将当前结点设置为队列头结点
                p.next = null; // help GC
                failed = false;
                return interrupted;//正常情况下死循环唯一的出口
            }
            //第二个if分句
            if (shouldParkAfterFailedAcquire(p, node) &&  //判断是否要阻塞当前线程
                parkAndCheckInterrupt())      //阻塞当前线程
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

这段代码主要的内容都在for循环中,这是一个死循环,主要有两个if分句构成。第一个if分句中,当前线程首先会判断前驱结点是否是头结点,如果是则尝试获取锁,获取锁成功则会设置当前结点为头结点(更新头指针)。为什么必须前驱结点为头结点才尝试去获取锁？因为头结点表示当前正占有锁的线程,正常情况下该线程释放锁后会通知后面结点中阻塞的线程,阻塞线程被唤醒后去获取锁,这是我们希望看到的。然而还有一种情况,就是前驱结点取消了等待,此时当前线程也会被唤醒,这时候就不应该去获取锁,而是往前回溯一直找到一个没有取消等待的结点,然后将自身连接在它后面。一旦我们成功获取了锁并成功将自身设置为头结点,就会跳出for循环。否则就会执行第二个if分句:确保前驱结点的状态为SIGNAL,然后阻塞当前线程。

先来看shouldParkAfterFailedAcquire(p, node)，从方法名上我们可以大概猜出这是判断是否要阻塞当前线程的,方法内容如下

```java
/**
 * Checks and updates status for a node that failed to acquire.
 * Returns true if thread should block. This is the main signal
 * control in all acquire loops.  Requires that pred == node.prev.
 *
 * @param pred node's predecessor holding status
 * @param node the node
 * @return {@code true} if thread should block
 */
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;
    if (ws == Node.SIGNAL) //状态为SIGNAL

        /*
         * This node has already set status asking a release
         * to signal it, so it can safely park.
         */
        return true;
    if (ws > 0) { //状态为CANCELLED,
        /*
         * Predecessor was cancelled. Skip over predecessors and
         * indicate retry.
         */
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else { //状态为初始化状态(ReentrentLock语境下)
        /*
         * waitStatus must be 0 or PROPAGATE.  Indicate that we
         * need a signal, but don't park yet.  Caller will need to
         * retry to make sure it cannot acquire before parking.
         */
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```

可以看到针对前驱结点pred的状态会进行不同的处理

- 1.pred状态为SIGNAL,则返回true,表示要阻塞当前线程。
- 2.pred状态为CANCELLED,则一直往队列头部回溯直到找到一个状态不为CANCELLED的结点,将当前节点node挂在这个结点的后面。
- 3.pred的状态为初始化状态,此时通过compareAndSetWaitStatus(pred, ws, Node.SIGNAL)方法将pred的状态改为SIGNAL。

其实这个方法的含义很简单,就是确保当前结点的前驱结点的状态为SIGNAL,SIGNAL意味着线程释放锁后会唤醒后面阻塞的线程。毕竟,只有确保能够被唤醒，当前线程才能放心的阻塞。

但是要注意只有在前驱结点已经是SIGNAL状态后才会执行后面的方法立即阻塞,对应上面的第一种情况。其他两种情况则因为返回false而重新执行一遍
for循环。这种延迟阻塞其实也是一种高并发场景下的优化,试想我如果在重新执行循环的时候成功获取了锁,是不是线程阻塞唤醒的开销就省了呢？

最后我们来看看阻塞线程的方法parkAndCheckInterrupt

shouldParkAfterFailedAcquire返回true表示应该阻塞当前线程,则会执行parkAndCheckInterrupt方法,这个方法比较简单,底层调用了LockSupport来阻塞当前线程,源码如下:

```java
/**
 * Convenience method to park and then check if interrupted
 *
 * @return {@code true} if interrupted
 */
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);
    return Thread.interrupted();
}
```

该方法内部通过调用LockSupport的park方法来阻塞当前线程,不清楚LockSupport的可以看看这里。[LockSupport功能简介及原理浅析](https://www.cnblogs.com/takumicx/p/9328459.html)

下面通过一张流程图来说明线程从加入同步队列到成功获取锁的过程
![img](https://images2018.cnblogs.com/blog/1422237/201808/1422237-20180805184242438-1630878927.png)

概括的说,线程在同步队列中会尝试获取锁,失败则被阻塞,被唤醒后会不停的重复这个过程,直到线程真正持有了锁,并将自身结点置于队列头部。

### 3.5 加锁流程源码总结

ReentrantLock非公平模式下的加锁流程如下
![img](https://images2018.cnblogs.com/blog/1422237/201808/1422237-20180805184331148-1572746602.png)

## 4.非公平模式解锁流程

### 4.1 解锁流程源码解读

解锁的源码相对简单,源码如下：

```java
public void unlock() {
    sync.release(1);  
}
public final boolean release(int arg) {
    if (tryRelease(arg)) { //释放锁(state-1),若释放后锁可被其他线程获取(state=0),返回true
        Node h = head;
        //当前队列不为空且头结点状态不为初始化状态(0)   
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);  //唤醒同步队列中被阻塞的线程
        return true;
    }
    return false;
}
```

正确找到sync的实现类,找到真正的入口方法,主要内容都在一个if语句中,先看下判断条件tryRelease方法

```java
protected final boolean tryRelease(int releases) {
    int c = getState() - releases; //计算待更新的state值
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) { //待更新的state值为0,说明持有锁的线程未重入,一旦释放锁其他线程将能获取
        free = true; 
        setExclusiveOwnerThread(null);//清除锁的持有线程标记
    }
    setState(c);//更新state值
    return free;
}
```

**tryRelease其实只是将线程持有锁的次数减1,即将state值减1,若减少后线程将完全释放锁(state值为0),则该方法将返回true,否则返回false。**由于执行该方法的线程必然持有锁,故该方法不需要任何同步操作。
若当前线程已经完全释放锁,即锁可被其他线程使用,则还应该唤醒后续等待线程。不过在此之前需要进行两个条件的判断：

- h!=null是为了防止队列为空,即没有任何线程处于等待队列中,那么也就不需要进行唤醒的操作
- h.waitStatus != 0是为了防止队列中虽有线程,但该线程还未阻塞,由前面的分析知,线程在阻塞自己前必须设置前驱结点的状态为SIGNAL,否则它不会阻塞自己。

接下来就是唤醒线程的操作,unparkSuccessor(h)源码如下

```java
private void unparkSuccessor(Node node) {
    /*
     * If status is negative (i.e., possibly needing signal) try
     * to clear in anticipation of signalling.  It is OK if this
     * fails or if status is changed by waiting thread.
     */
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    /*
     * Thread to unpark is held in successor, which is normally
     * just the next node.  But if cancelled or apparently null,
     * traverse backwards from tail to find the actual
     * non-cancelled successor.
     */
    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);
}
```

一般情况下只要唤醒后继结点的线程就行了,但是后继结点可能已经取消等待,所以从队列尾部往前回溯,找到离头结点最近的正常结点,并唤醒其线程。

### 4.2 解锁流程源码总结

![img](https://images2018.cnblogs.com/blog/1422237/201808/1422237-20180805184348967-1221850147.png)

## 5. 公平锁相比非公平锁的不同

公平锁模式下,对锁的获取有严格的条件限制。在同步队列有线程等待的情况下,所有线程在获取锁前必须先加入同步队列。队列中的线程按加入队列的先后次序获得锁。
从公平锁加锁的入口开始,
![img](https://images2018.cnblogs.com/blog/1422237/201808/1422237-20180805185314873-766370811.png)

对比非公平锁,少了非重入式获取锁的方法,这是第一个不同点

接着看获取锁的通用方法tryAcquire(),该方法在线程未进入队列,加入队列阻塞前和阻塞后被唤醒时都会执行。
![img](https://images2018.cnblogs.com/blog/1422237/201808/1422237-20180805185331385-713414476.png)

在真正CAS获取锁之前加了判断,内容如下

```java
public final boolean hasQueuedPredecessors() {
    // The correctness of this depends on head being initialized
    // before tail and on head.next being accurate if the current
    // thread is first in queue.
    Node t = tail; // Read fields in reverse initialization order
    Node h = head;
    Node s;
    return h != t &&
        ((s = h.next) == null || s.thread != Thread.currentThread());
}
```

从方法名我们就可知道这是判断队列中是否有优先级更高的等待线程,队列中哪个线程优先级最高？由于头结点是当前获取锁的线程,队列中的第二个结点代表的线程优先级最高。
那么我们只要判断队列中第二个结点是否存在以及这个结点是否代表当前线程就行了。这里分了两种情况进行探讨:

1. 第二个结点已经完全插入,但是这个结点是否就是当前线程所在结点还未知,所以通过s.thread != Thread.currentThread()进行判断,如果为true,说明第二个结点代表其他线程。
2. 第二个结点并未完全插入,我们知道结点入队一共分三步：

- 1.待插入结点的pre指针指向原尾结点
- 2.CAS更新尾指针
- 3.原尾结点的next指针指向新插入结点

所以(s = h.next) == null 就是用来判断2刚执行成功但还未执行3这种情况的。这种情况第二个结点必然属于其他线程。
以上两种情况都会使该方法返回true,即当前有优先级更高的线程在队列中等待,那么当前线程将不会执行CAS操作去获取锁,保证了线程获取锁的顺序与加入同步队列的顺序一致，很好的保证了公平性,但也增加了获取锁的成本。

## 6. 一些疑问的解答

### 6.1 为什么基于FIFO的同步队列可以实现非公平锁？

**由FIFO队列的特性知,先加入同步队列等待的线程会比后加入的线程更靠近队列的头部,那么它将比后者更早的被唤醒,它也就能更早的得到锁。**从这个意义上,对于在同步队列中等待的线程而言,它们获得锁的顺序和加入同步队列的顺序一致，这显然是一种公平模式。然而,线程并非只有在加入队列后才有机会获得锁,哪怕同步队列中已有线程在等待,非公平锁的不公平之处就在于此。回看下非公平锁的加锁流程,**线程在进入同步队列等待之前有两次抢占锁的机会:**

- 第一次是非重入式的获取锁,只有在**当前锁未被任何线程占有(包括自身)时才能成功**;
- 第二次是在进入同步队列前,包含所有情况的获取锁的方式。

只有这两次获取锁都失败后,线程才会构造结点并加入同步队列等待。而线程释放锁时是**先释放锁(修改state值),然后才唤醒后继结点的线程的。**试想下这种情况,线程A已经释放锁,但还没来得及唤醒后继线程C,而这时另一个线程B刚好尝试获取锁,此时锁恰好不被任何线程持有,它将成功获取锁而不用加入队列等待。线程C被唤醒尝试获取锁,而此时锁已经被线程B抢占,故而其获取失败并继续在队列中等待。整个过程如下图所示
![img](https://images2018.cnblogs.com/blog/1422237/201808/1422237-20180805185020232-615609336.png)

如果以线程第一次尝试获取锁到最后成功获取锁的次序来看,非公平锁确实很不公平。因为在队列中等待很久的线程相比还未进入队列等待的线程并没有优先权,甚至竞争也处于劣势:在队列中的线程要等待其他线程唤醒,**在获取锁之前还要检查前驱结点是否为头结点**。在锁竞争激烈的情况下,在队列中等待的线程可能迟迟竞争不到锁。这也就非公平在高并发情况下会出现的饥饿问题。那我们再开发中为什么大多使用会导致饥饿的非公平锁？很简单,因为它性能好啊。

### 6.2 为什么非公平锁性能好

非公平锁对锁的竞争是抢占式的(队列中线程除外),线程在进入等待队列前可以进行两次尝试,这大大增加了获取锁的机会。这种好处体现在两个方面:

- 1.**线程不必加入等待队列就可以获得锁,不仅免去了构造结点并加入队列的繁琐操作,同时也节省了线程阻塞唤醒的开销,线程阻塞和唤醒涉及到线程上下文的切换和操作系统的系统调用,是非常耗时的**。在高并发情况下,如果线程持有锁的时间非常短,短到线程入队阻塞的过程超过线程持有并释放锁的时间开销,那么这种抢占式特性对**并发性能的提升会更加明显。**
- 2.减少CAS竞争。**如果线程必须要加入阻塞队列才能获取锁,那入队时CAS竞争将变得异常激烈,CAS操作虽然不会导致失败线程挂起,但不断失败重试导致的对CPU的浪费也不能忽视。**除此之外,加锁流程中至少有两处通过将某些特殊情况提前来减少CAS操作的竞争,增加并发情况下的性能。一处就是获取锁时将非重入的情况提前,如下图所示
  ![img](https://images2018.cnblogs.com/blog/1422237/201808/1422237-20180805185241957-1365580718.png)

另一处就是入队的操作,将同步队列非空的情况提前处理
![img](https://images2018.cnblogs.com/blog/1422237/201808/1422237-20180805185255573-1887096699.png)

这两部分的代码在之后的通用逻辑处理中都有,很显然属于重复代码,但因为避免了执行无意义的流程代码,比如for循环,获取同步状态等,高并发场景下也能减少CAS竞争失败的可能。

## 7. 阅读源码的收获

- 1.熟悉了ReentrantLock的内部构造以及加锁和解锁的流程,理解了非公平锁和公平锁实现的本质区别以及为何前者相比后者有更好的性能。以此为基础,我们可以更好的使用ReentrantLock。
- 2.通过对部分实现细节的学习,了解了如何以CAS算法构建无锁的同步队列,我们可以借鉴并以此来构建自己的无锁的并发容器。

https://www.cnblogs.com/takumicx/p/9402021.html



