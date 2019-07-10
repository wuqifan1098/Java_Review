# 面试题

## 1. 你用过Java的J.U.C并发包吧，给我讲一下AQS的原理（vivo）

AQS核心思想是，**如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。**如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制AQS是**用CLH队列锁实现的，即将暂时获取不到锁的线程加入到队列中。**

CLH(Craig,Landin,and Hagersten)队列是一个虚拟的双向队列（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点（Node）来实现锁的分配。

作者：m_xiaoer 
原文：https://blog.csdn.net/m_xiaoer/article/details/73459444 

## 沉淀再出发：关于java中的AQS理解

### 一、前言

​    在java中有很多锁结构都继承自AQS(AbstractQueuedSynchronizer)这个抽象类如果我们仔细了解可以发现AQS的作用是非常大的，但是AQS的底层其实也是使用了大量的CAS，因此我们可以看到CAS的重要性了，但是CAS也是有缺陷的，但是在大部分使用的情况下，我们往往忽略了这种缺陷。

### 二、AQS的认识

####   2.1、AQS的基本概念

​    AQS（AbstractQueuedSynchronizer）就是抽象的队列式的同步器，AQS定义了一套多线程访问共享资源的同步器框架，许多同步类实现都依赖于它，AQS是一个Java提供的底层同步工具类，用一个int类型的变量表示同步状态，并提供了一系列的CAS操作来管理这个同步状态。AQS的主要作用是为Java中的并发同步组件提供统一的底层支持，如常用的ReentrantLock/Semaphore/CountDownLatch等等就是基于AQS实现的，用法是通过继承AQS实现其模版方法，然后将子类作为同步组件的内部类。

**AQS核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制AQS是用CLH队列锁实现的，即将暂时获取不到锁的线程加入到队列中。**

​    同步队列是AQS很重要的组成部分，它是一个双端队列，遵循FIFO原则，主要作用是用来**存放在锁上阻塞的线程，当一个线程尝试获取锁时，如果已经被占用，那么当前线程就会被构造成一个Node节点加入到同步队列的尾部，队列的头节点是成功获取锁的节点，当头节点线程释放锁时，会唤醒后面的节点并释放当前头节点的引用。**

![img](https://img2018.cnblogs.com/blog/1157683/201810/1157683-20181028154015825-926351069.png)

​       它维护了一个volatile int state（代表共享资源）和一个FIFO线程等待队列（多线程争用资源被阻塞时会进入此队列）。state的访问方式有三种:

```java
getState()
setState()
compareAndSetState()
```

​     AQS定义两种资源共享方式：**Exclusive**（独占，只有一个线程能执行，如ReentrantLock）和**Share**（共享，多个线程可同时执行，如Semaphore/CountDownLatch）。
​    不同的自定义同步器争用共享资源的方式也不同。自定义同步器在实现时只需要实现共享资源state的获取与释放方式即可，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS已经在顶层实现好了。自定义同步器实现时主要实现以下几种方法：

```java
1     isHeldExclusively()：该线程是否正在独占资源。只有用到condition才需要去实现它。
2     tryAcquire(int)：独占方式。尝试获取资源，成功则返回true，失败则返回false。
3     tryRelease(int)：独占方式。尝试释放资源，成功则返回true，失败则返回false。
4     tryAcquireShared(int)：共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
5     tryReleaseShared(int)：共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回true，否则返回false
```

![img](https://img2018.cnblogs.com/blog/1157683/201810/1157683-20181028170748502-1655076470.png)

　　ReentrantLock：state初始化为0，表示未锁定状态。**A线程lock()时，会调用tryAcquire()独占该锁并将state+1。**此后，其他线程再tryAcquire()时就会失败，**直到A线程unlock()到state=0（即释放锁）为止，其它线程才有机会获取该锁**。当然，释放锁之前，A线程自己是可以重复获取此锁的（state会累加），这就是可重入的概念。但要注意，**获取多少次就要释放多么次，这样才能保证state是能回到零态的。**
　　CountDownLatch：任务分为N个子线程去执行，**state也初始化为N**（注意N要与线程个数一致）。这N个子线程是并行执行的，每个子线程执行完后countDown()一次，state会CAS减1。**等到所有子线程都执行完后(即state=0)，会unpark()主调用线程，然后主调用线程就会从await()函数返回，继续后余动作。**
　　一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现tryAcquire-tryRelease、tryAcquireShared-tryReleaseShared中的一种即可。**但AQS也支持自定义同步器同时实现独占和共享两种方式，如ReentrantReadWriteLock。**

#### 2.1.1、acquire(int)

　　此方法是独占模式下线程获取共享资源的顶层入口。**如果获取到资源，线程直接返回，否则进入等待队列，直到获取到资源为止，且整个过程忽略中断的影响。**这也正是lock()的语义，当然不仅仅只限于lock()。获取到资源后，线程就可以去执行其临界区代码了。下面是acquire()的源码：

```java
1 public final void acquire(int arg) {
2      if (!tryAcquire(arg) &&
3          acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
4          selfInterrupt();
5 }
tryAcquire()尝试直接去获取资源，如果成功则直接返回；
addWaiter()将该线程加入等待队列的尾部，并标记为独占模式；
acquireQueued()使线程在等待队列中获取资源，一直获取到资源后才返回。如果在整个等待过程中被中断过，则返回true，否则返回false。
如果线程在等待过程中被中断过，它是不响应的。只是获取资源后才再进行自我中断selfInterrupt()，将中断补上。
```

#### tryAcquire(int)

　　此方法尝试去获取独占资源。如果获取成功，则直接返回true，否则直接返回false。如下是tryAcquire()的源码：

```java
protected boolean tryAcquire(int arg) {
     throw new UnsupportedOperationException();
}
```

　  AQS只是一个框架，**具体资源的获取/释放方式交由自定义同步器去实现，AQS这里只定义了一个接口，具体资源的获取交由自定义同步器去实现（通过state的get/set/CAS）**。至于能不能重入，能不能阻塞，那就看具体的自定义同步器怎么去设计了，当然，自定义同步器在进行资源访问时要考虑线程安全的影响。这里没有定义成abstract是因为独占模式下只用实现tryAcquire-tryRelease，而共享模式下只用实现tryAcquireShared-tryReleaseShared。如果都定义成abstract，那么每个模式也要去实现另一模式下的接口，这样设计可以**尽量减少不必要的工作量**。

####  addWaiter(Node)

　　此方法用于将当前线程加入到等待队列的队尾，并返回当前线程所在的结点。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
private Node addWaiter(Node mode) {
     //以给定模式构造结点。mode有两种：EXCLUSIVE（独占）和SHARED（共享）
    Node node = new Node(Thread.currentThread(), mode);
     //尝试快速方式直接放到队尾。
     Node pred = tail;
     if (pred != null) {
         node.prev = pred;
         if (compareAndSetTail(pred, node)) {
             pred.next = node;
             return node;
         }
     }
     //上一步失败或者初次加入，则采用终极自旋方式保证一定加入队尾
    enq(node);
    return node;
 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

​    Node结点是对每一个访问同步代码的线程的封装，其包含了需要同步的线程本身以及线程的状态，**如是否被阻塞，是否等待唤醒，是否已经被取消等。**变量waitStatus则表示当前被封装成Node结点的等待状态，共有4种取值CANCELLED、SIGNAL、CONDITION、PROPAGATE。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1     CANCELLED：值为1，在同步队列中等待的线程等待超时或被中断，需要从同步队列中取消该Node的结点，其结点的waitStatus为CANCELLED，即结束状态，      进入该状态后的结点将不会再变化。
2     SIGNAL：值为-1，被标识为该等待唤醒状态的后继结点，当其前继结点的线程释放了同步锁或被取消，将会通知该后继结点的线程执行。说白了，就是处于唤醒状态，              只要前继结点释放锁，就会通知标识为SIGNAL状态的后继结点的线程执行。
3     CONDITION：值为-2，与Condition相关，该标识的结点处于等待队列中，结点的线程等待在Condition上，当其他线程调用了Condition的signal()方法后，                CONDITION状态的结点将从等待队列转移到同步队列中，等待获取同步锁。
4     PROPAGATE：值为-3，与共享模式相关，在共享模式中，该状态标识结点的线程处于可运行状态。
5     0状态：值为0，代表初始化状态。
6     AQS在判断状态时，通过用waitStatus>0表示取消状态，而waitStatus<0表示有效状态。
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

#### enq(Node)

​    此方法用于将node加入队尾，采用终极自旋方式保证一定加入队尾。**CAS自旋volatile变量**，是一种很经典的用法。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
private Node enq(final Node node) {
     //CAS"自旋"，直到成功加入队尾
    for (;;) {
         Node t = tail;
         if (t == null) { // 队列为空，创建一个空的标志结点作为head结点，并将tail也指向它。
             if (compareAndSetHead(new Node()))
                 tail = head;
         } else {//正常流程，放入队尾
            node.prev = t;
             if (compareAndSetTail(t, node)) {
                 t.next = node;
                 return t;
             }
         }
     }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

#### acquireQueued(Node, int)

​     通过tryAcquire()和addWaiter()，该线程获取资源失败，已经被放入等待队列尾部了。该线程下一部进入等待状态休息，直到其他线程彻底释放资源后唤醒自己，自己再拿到资源，然后就可以去干自己想干的事了。**acquireQueued()就是干这件事：在等待队列中排队拿号（中间没其它事干可以休息），直到拿到号后再返回，这个函数非常关键。**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
 1 final boolean acquireQueued(final Node node, int arg) {
 2     boolean failed = true;//标记是否成功拿到资源
 3     try {
 4          boolean interrupted = false;//标记等待过程中是否被中断过
 5         //又是一个“自旋”！
 6         for (;;) {
 7             final Node p = node.predecessor();//拿到前驱
 8             //如果前驱是head，即该结点已成老二，那么便有资格去尝试获取资源（可能是老大释放完资源唤醒自己的，当然也可能被interrupt了）。
 9             if (p == head && tryAcquire(arg)) {
10                 setHead(node);//拿到资源后，将head指向该结点。所以head所指的标杆结点，就是当前获取到资源的那个结点或null。
11                 p.next = null;                  // setHead中node.prev已置为null，此处再将head.next置为null，就是为了方便GC回收以前的head结点。也就意味着之前拿完资源的结点出队了！
12                 failed = false;
13                 return interrupted;//返回等待过程中是否被中断过
14            }
15             
16              //如果自己可以休息了，就进入waiting状态，直到被unpark()
17              if (shouldParkAfterFailedAcquire(p, node) &&
18                  parkAndCheckInterrupt())
19                  interrupted = true;//如果等待过程中被中断过，哪怕只有那么一次，就将interrupted标记为true
20             }
21      } finally {
22          if (failed)
23              cancelAcquire(node);
24      }
25 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 到这里了，我们先看看shouldParkAfterFailedAcquire()和parkAndCheckInterrupt()具体干些什么。

#### shouldParkAfterFailedAcquire(Node, Node)

   此方法主要用于检查状态，看看自己是否真的可以去休息了，以免队列前边的线程都放弃了盲等。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
 1 private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
 2      int ws = pred.waitStatus;//拿到前驱的状态
 3       if (ws == Node.SIGNAL)
 4          //如果已经告诉前驱拿完号后通知自己一下，那就可以安心休息了
 5         return true;
 6         if (ws > 0) {
 7          /*
 8          * 如果前驱放弃了，那就一直往前找，直到找到最近一个正常等待的状态，并排在它的后边。
 9          * 注意：那些放弃的结点，由于被自己“加塞”到它们前边，它们相当于形成一个无引用链，
10          *           稍后就会被GC回收
11         */
12          do {
13              node.prev = pred = pred.prev;
14           } while (pred.waitStatus > 0);
15            pred.next = node;
16      } else {
17           //如果前驱正常，那就把前驱的状态设置成SIGNAL，告诉它拿完号后通知自己一下。
18          //有可能失败，前驱说不定刚刚释放完。
19          compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
20      }
21       return false;
22 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

​      整个流程中，如果前驱结点的状态不是SIGNAL，那么自己就不能安心去休息，需要去找个安心的休息点，同时可以再尝试下看有没有机会轮到自己拿号。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
1 parkAndCheckInterrupt()
2 　  如果线程找好安全休息点后，那就可以安心去休息了。此方法就是让线程去休息，真正进入等待状态。
3 private final boolean parkAndCheckInterrupt() {
4      LockSupport.park(this);//调用park()使线程进入waiting状态
5      return Thread.interrupted();//如果被唤醒，查看自己是不是被中断的。
6 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

​    park()会让当前线程进入waiting状态。在此状态下，有两种途径可以唤醒该线程：1）被unpark()；2）被interrupt()。需要注意的是，Thread.interrupted()会清除当前线程的中断标记位。 
​    至此，我们看一下前面的总函数就知道了整个流程了：
![img](https://img2018.cnblogs.com/blog/1157683/201810/1157683-20181028162211377-1552057530.png)

#### 2.1.2、release(int)

 　这里我们来讲一下acquire()的反操作release()。此方法是独占模式下线程释放共享资源的顶层入口。它会释放指定量的资源，如果彻底释放（即state=0）,则唤醒等待队列里的其他线程。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
1 public final boolean release(int arg) {
2     if (tryRelease(arg)) {
3          Node h = head;//找到头结点
4        if (h != null && h.waitStatus != 0)
5             unparkSuccessor(h);//唤醒等待队列里的下一个线程
6          return true;
7      }
8      return false;
9 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

   逻辑并不复杂。调用tryRelease()来释放资源。有一点需要注意的是，它是根据tryRelease()的返回值来判断该线程是否已经完成释放掉资源了，所以自定义同步器在设计tryRelease()的时候要明确这一点。

#### tryRelease(int)

```java
1 protected boolean tryRelease(int arg) {
2     throw new UnsupportedOperationException();
3 }
```

　　跟tryAcquire()一样，这个方法是需要独占模式的自定义同步器去实现。正常来说，tryRelease()都会成功的，因为这是独占模式，该线程来释放资源，那么它肯定已经拿到独占资源了，直接减掉相应量的资源即可(state-=arg)，也不需要考虑线程安全的问题。但要注意它的返回值，release()是根据tryRelease()的返回值来判断该线程是否已经完成释放掉资源了，所以自义定同步器在实现时，如果已经彻底释放资源(state=0)，要返回true，否则返回false。

#### unparkSuccessor(Node)

　　此方法用于唤醒等待队列中下一个线程。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
 1 private void unparkSuccessor(Node node) {
 2     //这里，node一般为当前线程所在的结点。
 3         int ws = node.waitStatus;
 4      if (ws < 0)//置零，当前线程所在的结点状态，允许失败。
 5          compareAndSetWaitStatus(node, ws, 0);
 6      Node s = node.next;//找到下一个需要唤醒的结点s
 7      if (s == null || s.waitStatus > 0) {//如果为空或已取消
 8         s = null;
 9         for (Node t = tail; t != null && t != node; t = t.prev)
10             if (t.waitStatus <= 0)//从这里可以看出，<=0的结点，都是还有效的结点。
11                  s = t;
12      }
13      if (s != null)
14           LockSupport.unpark(s.thread);//唤醒
15  }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

​     用unpark()唤醒等待队列中最前边的那个未放弃线程，这里我们也用s来表示吧。此时，再和acquireQueued()联系起来，s被唤醒后，进入if (p == head && tryAcquire(arg))的判断（即使p!=head也没关系，它会再进入shouldParkAfterFailedAcquire()寻找一个安全点。这里既然s已经是等待队列中最前边的那个未放弃线程了，那么通过shouldParkAfterFailedAcquire()的调整，s也必然会跑到head的next结点，下一次自旋p==head就成立啦），然后s把自己设置成head标杆结点，表示自己已经获取到资源了，acquire()也返回了。
　　release()是独占模式下线程释放共享资源的顶层入口。它会释放指定量的资源，如果彻底释放了（即state=0）,它会唤醒等待队列里的其他线程来获取资源。

![img](https://img2018.cnblogs.com/blog/1157683/201810/1157683-20181028171312416-1859849804.png)


    **同样的让我们再来看看对于共享锁的情况下，资源的获取和释放。**

#### 2.1.3、acquireShared(int)

　　此方法是共享模式下线程获取共享资源的顶层入口。它会获取指定量的资源，获取成功则直接返回，获取失败则进入等待队列，直到获取到资源为止，整个过程忽略中断。

```java
1 public final void acquireShared(int arg) {
2     if (tryAcquireShared(arg) < 0)
3         doAcquireShared(arg);
4     }
5 }
```

　　这里tryAcquireShared()依然需要自定义同步器去实现。但是AQS已经把其返回值的语义定义好了：负值代表获取失败；0代表获取成功，但没有剩余资源；正数表示获取成功，还有剩余资源，其他线程还可以去获取。所以这里acquireShared()的流程就是：
    tryAcquireShared()尝试获取资源，成功则直接返回；失败则通过doAcquireShared()进入等待队列，直到获取到资源为止才返回。

#### doAcquireShared(int)

　　此方法用于将当前线程加入等待队列尾部休息，直到其他线程释放资源唤醒自己，自己成功拿到相应量的资源后才返回。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
 1 private void doAcquireShared(int arg) {
 2     final Node node = addWaiter(Node.SHARED);//加入队列尾部
 3     boolean failed = true;//是否成功标志
 4     try {
 5          boolean interrupted = false;//等待过程中是否被中断过的标志
 6          for (;;) {
 7              final Node p = node.predecessor();//前驱
 8              if (p == head) {//如果到head的下一个，因为head是拿到资源的线程，此时node被唤醒，很可能是head用完资源来唤醒自己的
 9                  int r = tryAcquireShared(arg);//尝试获取资源
10                  if (r >= 0) {//成功
11                      setHeadAndPropagate(node, r);//将head指向自己，还有剩余资源可以再唤醒之后的线程
12                      p.next = null; // help GC
13                      if (interrupted)//如果等待过程中被打断过，此时将中断补上
14                          selfInterrupt();
15                      failed = false;
16                      return;
17                  }
18               }
19              //判断状态，寻找安全点，进入waiting状态，等着被unpark()或interrupt()
20              if (shouldParkAfterFailedAcquire(p, node) &&
21                  parkAndCheckInterrupt())
22                  interrupted = true;
23               }
24           }
25      } finally {
26          if (failed)
27              cancelAcquire(node);
28      }
29 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

​    其实和acquireQueued()流程并没有太大区别。只不过这里将补中断的selfInterrupt()放到doAcquireShared()里了，而独占模式是放到acquireQueued()之外。
　　跟独占模式比，还有一点需要注意的是，这里只有线程是head.next时（“老二”），才会去尝试获取资源，有剩余的话还会唤醒之后的队友。那么问题就来了，假如老大用完后释放了5个资源，而老二需要6个，老三需要1个，老四需要2个。老大先唤醒老二，老二一看资源不够，他是把资源让给老三呢，还是不让？答案是否定的！老二会继续park()等待其他线程释放资源，也更不会去唤醒老三和老四了。独占模式，同一时刻只有一个线程去执行，这样做未尝不可；但共享模式下，多个线程是可以同时执行的，现在因为老二的资源需求量大，而把后面量小的老三和老四也都卡住了。当然，这并不是问题，只是AQS保证严格按照入队顺序唤醒罢了（**保证公平，但降低了并发**）。

#### setHeadAndPropagate(Node, int)

​    此方法在setHead()的基础上多了一步，就是自己苏醒的同时，如果条件符合（比如还有剩余资源），还会去唤醒后继结点，毕竟是共享模式。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
 1 private void setHeadAndPropagate(Node node, int propagate) {
 2      Node h = head;
 3     setHead(node);//head指向自己
 4       //如果还有剩余量，继续唤醒下一个邻居线程
 5      if (propagate > 0 || h == null || h.waitStatus < 0) {
 6          Node s = node.next;
 7          if (s == null || s.isShared())
 8              doReleaseShared();
 9     }
10 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

#### 2.1.4、releaseShared()

　　我们来看acquireShared()的反操作releaseShared()，此方法是共享模式下线程释放共享资源的顶层入口。它会释放指定量的资源，如果成功释放且允许唤醒等待线程，它会唤醒等待队列里的其他线程来获取资源。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
1 public final boolean releaseShared(int arg) {
2     if (tryReleaseShared(arg)) {//尝试释放资源
3         doReleaseShared();//唤醒后继结点
4         return true;
5    }
6     return false;
7 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

​     此方法的流程也比较简单，释放掉资源后，唤醒后继。跟独占模式下的release()相似，但是独占模式下的tryRelease()在完全释放掉资源（state=0）后，才会返回true去唤醒其他线程，这主要是基于独占下可重入的考量；而共享模式下的releaseShared()则没有这种要求，共享模式实质就是控制一定量的线程并发执行，那么拥有资源的线程在释放掉部分资源时就可以唤醒后继等待结点。例如资源总量是13，A（5）和B（7）分别获取到资源并发运行，C（4）来时只剩1个资源就需要等待。A在运行过程中释放掉2个资源量，然后tryReleaseShared(2)返回true唤醒C，C一看只有3个仍不够继续等待；随后B又释放2个，tryReleaseShared(2)返回true唤醒C，C一看有5个够自己用了，然后C就可以跟A和B一起运行。而ReentrantReadWriteLock读锁的tryReleaseShared()只有在完全释放掉资源（state=0）才返回true，所以自定义同步器可以根据需要决定tryReleaseShared()的返回值。

#### doReleaseShared()

​    此方法用于唤醒后继。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
 1 private void doReleaseShared() {
 2     for (;;) {
 3        Node h = head;
 4         if (h != null && h != tail) {
 5            int ws = h.waitStatus;
 6            if (ws == Node.SIGNAL) {
 7                  if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
 8                      continue;
 9                  unparkSuccessor(h);//唤醒后继
10            }
11            else if (ws == 0 &&
12                       !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
13                  continue;
14          }
15          if (h == head)// head发生变化
16                break;
17      }
18 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

​    至此我们详解了独占和共享两种模式下获取-释放资源(acquire-release、acquireShared-releaseShared)的源码，值得注意的是，acquire()和acquireShared()两种方法下，线程在等待队列中都是忽略中断的。AQS也支持响应中断的，acquireInterruptibly()/acquireSharedInterruptibly()即是，这里相应的源码跟acquire()和acquireSahred()差不多。

![img](https://img2018.cnblogs.com/blog/1157683/201810/1157683-20181028171428719-2111852002.png)

 

#### 2.3、独占锁和共享锁在实现上的区别

​    独占锁的同步状态值为1，即同一时刻只能有一个线程成功获取同步状态。共享锁的同步状态>1，取值由上层同步组件确定。
​    独占锁队列中头节点运行完成后释放它的直接后继节点。共享锁队列中头节点运行完成后释放它后面的所有节点。
​    共享锁中会出现多个线程（即同步队列中的节点）同时成功获取同步状态的情况。

####  2.4、简单使用

   既然明白基本的操作机理，我们就可以实现自己的锁机制了，比如mutex这种不可重入的互斥锁。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
 1 class Mutex implements Lock, java.io.Serializable {
 2     // 自定义同步器
 3     private static class Sync extends AbstractQueuedSynchronizer {
 4         // 判断是否锁定状态
 5         protected boolean isHeldExclusively() {
 6             return getState() == 1;
 7         }
 8 
 9         // 尝试获取资源，立即返回。成功则返回true，否则false。
10         public boolean tryAcquire(int acquires) {
11             assert acquires == 1; // 这里限定只能为1个量
12             if (compareAndSetState(0, 1)) {//state为0才设置为1，不可重入！
13                 setExclusiveOwnerThread(Thread.currentThread());//设置为当前线程独占资源
14                 return true;
15             }
16             return false;
17         }
18 
19         // 尝试释放资源，立即返回。成功则为true，否则false。
20         protected boolean tryRelease(int releases) {
21             assert releases == 1; // 限定为1个量
22             if (getState() == 0)//既然来释放，那肯定就是已占有状态了。只是为了保险，多层判断！
23                 throw new IllegalMonitorStateException();
24             setExclusiveOwnerThread(null);
25             setState(0);//释放资源，放弃占有状态
26             return true;
27         }
28     }
29 
30     // 真正同步类的实现都依赖继承于AQS的自定义同步器！
31     private final Sync sync = new Sync();
32 
33     //lock<-->acquire。两者语义一样：获取资源，即便等待，直到成功才返回。
34     public void lock() {
35         sync.acquire(1);
36     }
37 
38     //tryLock<-->tryAcquire。两者语义一样：尝试获取资源，要求立即返回。成功则为true，失败则为false。
39     public boolean tryLock() {
40         return sync.tryAcquire(1);
41     }
42 
43     //unlock<-->release。两者语义一样：释放资源。
44     public void unlock() {
45         sync.release(1);
46     }
47 
48     //锁是否占有状态
49     public boolean isLocked() {
50         return sync.isHeldExclusively();
51     }
52 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

​    同步类在实现时一般都将自定义同步器（sync）定义为内部类，供自己使用；而同步类自己（Mutex）则实现某个接口，对外服务。当然，接口的实现要直接依赖sync，它们在语义上也存在某种对应关系，而sync只用实现资源state的获取-释放方式tryAcquire-tryRelelase，至于线程的排队、等待、唤醒等，上层的AQS都已经实现好了，我们不用关心。
　  除了Mutex，ReentrantLock/CountDownLatch/Semphore这些同步类的实现方式都差不多，不同的地方就在获取-释放资源的方式tryAcquire-tryRelelase。掌握了这点，AQS的核心便被攻破了。

####  2.5、重入锁

​    重入锁指的是当前线成功获取锁后，如果再次访问该临界区，则不会对自己产生互斥行为。Java中对ReentrantLock和synchronized都是可重入锁，**synchronized由jvm实现可重入机制，ReentrantLock的可重入性基于AQS实现**。同时，ReentrantLock还提供公平锁和非公平锁两种模式。

#### ReentrantLock重入锁  

​    重入锁的基本原理是判断上次获取锁的线程是否为当前线程，如果是则可再次进入临界区，如果不是，则阻塞。重入锁的最主要逻辑就锁判断上次获取锁的线程是否为当前线程。由于ReentrantLock是基于AQS实现的，底层通过操作同步状态来获取锁，下面看一下非公平锁的实现逻辑：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
 1 final boolean nonfairTryAcquire(int acquires) {
 2             //获取当前线程
 3             final Thread current = Thread.currentThread();
 4             //通过AQS获取同步状态
 5             int c = getState();
 6             //同步状态为0，说明临界区处于无锁状态，
 7             if (c == 0) {
 8                 //修改同步状态，即加锁
 9                 if (compareAndSetState(0, acquires)) {
10                     //将当前线程设置为锁的owner
11                     setExclusiveOwnerThread(current);
12                     return true;
13                 }
14             }
15             //如果临界区处于锁定状态，且上次获取锁的线程为当前线程
16             else if (current == getExclusiveOwnerThread()) {
17                  //则递增同步状态
18                 int nextc = c + acquires;
19                 if (nextc < 0) // overflow
20                     throw new Error("Maximum lock count exceeded");
21                 setState(nextc);
22                 return true;
23             }
24             return false;
25 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

#### 非公平锁

​    非公平锁是指当锁状态为可用时，不管在当前锁上是否有其他线程在等待，新近线程都有机会抢占锁。上述代码即为非公平锁和核心实现，可以看到只要同步状态为0，任何调用lock的线程都有可能获取到锁，而不是按照锁请求的FIFO原则来进行的。

#### 公平锁

​    公平锁是指当多个线程尝试获取锁时，成功获取锁的顺序与请求获取锁的顺序相同，下面看一个ReentrantLock的实现：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
 1 protected final boolean tryAcquire(int acquires) {
 2             final Thread current = Thread.currentThread();
 3             int c = getState();
 4             if (c == 0) {
 5                 //此处为公平锁的核心，即判断同步队列中当前节点是否有前驱节点
 6                 if (!hasQueuedPredecessors() &&
 7                     compareAndSetState(0, acquires)) {
 8                     setExclusiveOwnerThread(current);
 9                     return true;
10                 }
11             }
12             else if (current == getExclusiveOwnerThread()) {
13                 int nextc = c + acquires;
14                 if (nextc < 0)
15                     throw new Error("Maximum lock count exceeded");
16                 setState(nextc);
17                 return true;
18             }
19             return false;
20 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

​    从上面的代码中可以看出，公平锁与非公平锁的区别仅在于是否判断当前节点是否存在前驱节点!hasQueuedPredecessors() ，由AQS可知，如果当前线程获取锁失败就会被加入到AQS同步队列中，那么，如果同步队列中的节点存在前驱节点，也就表明存在线程比当前节点线程更早的获取锁，故只有等待前面的线程释放锁后才能获取锁。

####  2.6、读写锁

​    Java提供了一个基于AQS到读写锁实现ReentrantReadWriteLock，该读写锁到实现原理是：**将同步变量state按照高16位和低16位进行拆分，高16位表示读锁，低16位表示写锁。**

![img](https://img2018.cnblogs.com/blog/1157683/201810/1157683-20181028173945971-354373046.png)

 



```
1 一次只有一个线程可以占有写模式的读写锁, 但是可以有多个线程同时占有读模式的读写锁. 正是因为这个特性：
2 当读写锁是写加锁状态时, 在这个锁被解锁之前, 所有试图对这个锁加锁的线程都会被阻塞.
3 当读写锁在读加锁状态时, 所有试图以读模式对它进行加锁的线程都可以得到访问权, 但是如果线程希望以写模式对此锁进行加锁, 它必须直到所有的线程释放锁.
4 通常, 当读写锁处于读模式锁住状态时, 如果有另外线程试图以写模式加锁, 读写锁通常会阻塞随后的读模式锁请求, 这样可以避免读模式锁长期占用, 而等待的写模式锁请求长期阻塞.
5 读写锁适合于对数据结构的读次数比写次数多得多的情况. 因为, 读模式锁定时可以共享, 以写模式锁住时意味着独占, 所以读写锁又叫共享-独占锁.
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

#### 2.6.1、写锁的获取与释放

  写锁是一个独占锁，所以我们看一下ReentrantReadWriteLock中tryAcquire(arg)的实现：
**写锁的获取处理流程如下：**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1     获取同步状态，并从中分离出低16为的写锁状态
2     如果同步状态不为0，说明存在读锁或写锁
3     如果存在读锁（c ！=0 && w == 0），则不能获取写锁（保证写对读的可见性）
4     如果当前线程不是上次获取写锁的线程，则不能获取写锁（写锁为独占锁）
5     如果以上判断均通过，则在低16为写锁同步状态上利用CAS进行修改（增加写锁同步状态，实现可重入）
6     将当前线程设置为写锁的获取线程
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
 1 protected final boolean tryAcquire(int acquires) {
 2             Thread current = Thread.currentThread();
 3             int c = getState();
 4             int w = exclusiveCount(c);
 5             if (c != 0) {
 6                 if (w == 0 || current != getExclusiveOwnerThread())
 7                     return false;
 8                 if (w + exclusiveCount(acquires) > MAX_COUNT)
 9                     throw new Error("Maximum lock count exceeded");
10                 // Reentrant acquire
11                 setState(c + acquires);
12                 return true;
13             }
14             if (writerShouldBlock() ||
15                 !compareAndSetState(c, c + acquires))
16                 return false;
17             setExclusiveOwnerThread(current);
18             return true;
19 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**写锁的释放过程与独占锁基本相同：**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
 1 protected final boolean tryRelease(int releases) {
 2             if (!isHeldExclusively())
 3                 throw new IllegalMonitorStateException();
 4             int nextc = getState() - releases;
 5             boolean free = exclusiveCount(nextc) == 0;
 6             if (free)
 7                 setExclusiveOwnerThread(null);
 8             setState(nextc);
 9             return free;
10 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

   在释放的过程中，不断减少读锁同步状态，当同步状态为0时，写锁完全释放。

#### 2.6.2、读锁的获取与释放

​    读锁是一个共享锁，获取读锁的步骤如下：

```
1     获取当前同步状态
2     计算高16为读锁状态+1后的值
3     如果大于能够获取到的读锁的最大值，则抛出异常
4     如果存在写锁并且当前线程不是写锁的获取者，则获取读锁失败
5     如果上述判断都通过，则利用CAS重新设置读锁的同步状态
```

   读锁的获取步骤与写锁类似，即不断的释放写锁状态，直到为0时，表示没有线程获取读锁。

### 三、总结

​    通过对java中的AQS锁机制的剖析，我们理解了独占和共享两种基本的持有锁的方式，并且分析了Mutex、可重入锁、公平锁、非公平锁的实现，读写锁等特殊的锁。通过对这些所的理解，我们更加深刻地理解了AQS的本质，以及其中线程的状态和活动轨迹。

**AQS就是一个并发包的基础组件，用来实现各种锁，各种同步组件的。它包含了state变量、加锁线程、等待队列等并发中的核心组件。**