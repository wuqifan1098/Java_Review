# 面试题

## 1.CountDownLatch理解

CountDownLatch可以理解为一个计数器在初始化时设置初始值，**当一个线程需要等待某些操作先完成时，需要调用await()方法**。这个方法让线程进入休眠状态直到等待的所有线程都执行完成。每调用一次countDown()方法，内部计数器减1，直到计数器为0时唤醒。这个可以理解为特殊的CyclicBarrier。

## 2.CountDownLatch方法

核心方法两个：countDown()和await()

```java
countDown():使CountDownLatch维护的内部计数器减1,每个被等待的线程完成的时候调用
await():线程在执行到CountDownLatch的时候会将此线程置于休眠
```

案例场景：视频会议室里等与会人员到齐了会议才能开始。

## 3.CountDownLatch 和CyclicBarrier的不同之处?

| CountDownLatch                                               | CyclicBarrier                                                |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 减计数方式                                                   | 加计数方式                                                   |
| 计算为0时释放所有等待的线程                                  | 计数达到指定值时释放所有等待线程                             |
| 计数为0时，无法重置                                          | 计数达到指定值时，计数置为0重新开始                          |
| **调用countDown()方法计数减一，调用await()方法只进行阻塞，对计数没任何影响** | **调用await()方法计数加1，若加1后的值不等于构造方法的值，则线程阻塞** |
| 不可重复利用                                                 | 可重复利用                                                   |

适用场景：

CountDownLatch ：一个线程**需要等待其它线程完成操作后**，才能进行后续的操作；

CyclicBarrier ：需要**所有的子任务都完成时**，才执行主任务。

## 4.多线程情况下怎么实现线程等待？（美团，海康威视）

CountDownLatch

## 5.CountDownLatch在实现过程中有什么需要注意的？（美团）

1. 批次请求之间不能有执行顺序要求，否则多个线程并发处理无法保证请求执行顺序
1. 各线程都要操作的结果列表必须是**线程安全的**，比如上面代码范例的countDownResultList
1. 各子线程的**countDown操作要在finally中执行**，确保一定可以执行
1. 主线程的await**操作需要设置超时时间**，避免因子线程处理异常而长时间一直等待，如果**中断需要抛出异常或返回错误结果**

## 6.实现CountDownLatch需要捕捉异常，为什么需要捕捉异常？（美团）

## 7. Semaphore拿到执行权的线程之间是否互斥

Semaphore可以有多个锁，可允许多个线程拥有执行权，如果并发访问同一个对象，会产生线程安全问题。

# CountDownLatch

## 1、简介

　　CountDownLatch是Java1.5之后引入的Java并发工具类，放在java.util.concurrent包下面 http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/package-summary.html 官方API。

　　CountDownLatch**能够使一个或多个线程等待其他线程完成各自的工作后再执行**；CountDownLatch是JDK 5+里面闭锁的一个实现。

　　**闭锁（Latch）**：一种同步方法，可以延迟线程的进度直到线程到达某个终点状态。通俗的讲就是，一个闭锁相当于一扇大门，在大门打开之前所有线程都被阻断，一旦大门打开所有线程都将通过，但是一旦大门打开，所有线程都通过了，那么这个闭锁的状态就失效了，门的状态也就不能变了，只能是打开状态。也就是说闭锁的状态是一次性的，它确保在闭锁打开之前所有特定的活动都需要在闭锁打开之后才能完成。

　　与CountDownLatch第一次交互是主线程等待其它的线程，**主线程必须在启动其它线程后立即调用await方法，这样主线程的操作就会在这个方法上阻塞，直到其他线程完成各自的任务。**

　　其他的N个线程必须引用闭锁对象，因为他们需要通知CountDownLatch对象，他们已经完成了各自的任务，这种机制就是通过countDown()方法来完成的。**每调用一次这个方法，在构造函数中初始化的count值就减1，所以当N个线程都调用了这个方法count的值等于0，然后主线程就能通过await方法，恢复自己的任务**。

　　这里的主线程是相对的概念，需要根据CountDownLatch创建的场景分析。

## 2、实现原理

同步功能是**基于 AQS 实现的**，CountDownLatch 使用 AQS 中的 **state 成员变量**作为计数器，在 state 不为0的情况下，凡是调用 await 方法的线程将会被阻塞，并被放入 AQS 所维护的同步队列中进行等待。

## 3、主要方法

```java
特有方法： 
public CountDownLatch(int count); //指定计数的次数，只能被设置1次
public void countDown();  //调用此方法则计数减1
public void await() throws InterruptedException   //调用此方法会一直阻塞当前线程，直到计时器的值为0，除非线程被中断。
Public Long getCount();   //得到当前的计数
Public boolean await(long timeout, TimeUnit unit) //调用此方法会一直阻塞当前线程，直到计时器的值为0，除非线程被中断或者计数器超时，返回false代表计数器超时。
From Object Inherited：
Clone、equals、hashCode、notify、notifyALL、wait等。
```

## 4、使用场景

（1）开启多个线程分块下载一个大文件，每个线程只下载固定的一截，最后由另外一个线程来拼接所有的分段。

（2）应用程序的主线程希望在负责启动框架服务的线程已经启动所有的框架服务之后再执行。

（3）确保一个计算不会执行，直到所需要的资源被初始化。

# 5、Demo

```java
public class CountDownLatchDemo{
    
    public static void main(String[] args) throws Exception{
        CountDownLatch countDownLatch = new CountDownLatch(6);
        for(int i = 1; i <=6 ; i++){
            new Thread(() ->{
                System.out.printIn(Thread.currentThread().getName()+"\t 上完自习，离开教室");
                countDownLatch.countDown();//走一个就减1
            },String.valueOf(i).start();)
                                countDownLatch.await();//减到0就不阻塞
                System.out.printIn(Thread.currentThread().getName()+"\t ****班长关门走人");

    }
}
```



# CyclicBarrier

## 1.简介

CyclicBarrier 的字面意思是可循环使用(Cyclic)的屏障(Barrier)。它要做的事情是，**让一组线程到达一个屏障(也可以叫同步点)时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。**CyclicBarrier默认的构造方法是CyclicBarrier(int parties)，其参数表示屏障拦截的线程数量，每个线程调用await方法告诉CyclicBarrier我已经到达了屏障，然后当前线程被阻塞。

当一个线程到达集合点时，它将调用await()方法等待其它的线程。线程调用await()方法后，CyclicBarrier将阻塞这个线程，并将它置入休眠状态等待其它线程的到来。等最后一个线程调用await()方法时，CyclicBarrier将唤醒所有等待的线程，然后这些线程将继续执行。

## 2.原理

基于**重入锁 ReentrantLock 实现**，线程调用 await 方法需要先获取锁才能访问。在最后一个线程访问 await 方法前，其他线程进入 await 方法中后，会调用 Condition 的 await 方法进入等待状态；在最后一个线程进入 CyclicBarrier的 await 方法后，该线程将会调用 Condition 的 signalAll 方法唤醒所有处于等待状态中的线程；最后一个进入 await 的线程还会重置 CyclicBarrier 的状态，使其可以重复使用。

## 3.方法

```java
await()：使线程置入休眠直到最后一个线程的到来之后唤醒所有休眠的线程
```

CyclicBarrier类有两个常用的构造方法：

（1）CyclicBarrier(int parties)
这里的parties也是一个计数器，例如，初始化时parties里的计数是3，于是拥有该CyclicBarrier对象的线程当parties的计数为3时就唤醒，注意：这里parties里的计数在运行时当调用CyclicBarrier:await()时,计数就加1，一直加到初始的值。

（2）CyclicBarrier(int parties, Runnable barrierAction)
这里的parties与上一个构造方法的解释是一样的，这里需要解释的是第二个入参(Runnable barrierAction),这个参数是一个实现Runnable接口的类的对象，也就是说当parties加到初始值时就触发barrierAction的内容。

案例场景：有4个游戏玩家玩游戏，游戏有三个关卡，每个关卡必须要所有玩家都到达后才能允许通过。其实这个场景里的玩家中如果有玩家A先到了关卡1，他必须等到其他所有玩家都到达关卡1时才能通过，也就是说线程之间需要相互等待。这和CountDownLatch的应用场景有区别，CountDownLatch里的线程是到了运行的目标后继续干自己的其他事情，而这里的线程需要等待其他线程后才能继续完成后面的工作。 

# 4.Demo

```java
public class CyclicBarrierDemo{
    
    public static void main(String[] args){
        
        CyclicBarrier cyclicBarrier = new CyclicBarrier(7,() - > {System.out.printIn("****召唤神龙");});
        
        for(int i = 1; i <=7 ; i++){
            final int tempInt = i;
            new Thread(() ->{
                System.out.printIn(Thread.currentThread().getName()+"\t 收集到第： " + tempInt + "龙珠");
                try{
                    cyclicBarrier.await();//先到的被阻塞
                }catch(InterruptedException e){
                    e.printStackTrace();
                }catch(BrokenBarrierException e){
                    e.printStackTrace();
                }
    },String.valueOf(i)).start();
}
```


# Semaphore

## 1.简介

信号量就是可以声明多把锁（包括一把锁，此时为互斥信号量）。
举个例子：一个房间如果只能容纳5个人，多出来的人必须在门外面等着。如何去做呢？一个解决办法就是：房间外面挂着五把钥匙，每进去一个人就取走一把钥匙，没有钥匙的不能进入该房间，而是在外面等待。每出来一个人就把钥匙放回原处以方便别人再次进入。 相当于抢车位，可以复用。

## 2.Demo

```java
public class SemaphoreDemo
{
    public static void main(String[] args){
        Semaphore semaphore = new Semaphore(3);//设定3个车位
        
        for(int i = 1; i <= 6; i++){
            try{
                    semaphore.acquire();//先到的被阻塞
                                System.out.printIn(Thread.currentThread().getName()+"\t抢到车位");
                }catch(InterruptedException e){
                    e.printStackTrace();
                }finally{
                	semaphore.release();
            }
        },String.valueOf(i)).start();
    }
```

