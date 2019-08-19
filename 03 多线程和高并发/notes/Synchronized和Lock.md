# 面试题

## 1.什么时候会出现线程安全问题？

在单线程中不会出现线程安全问题，而在多线程编程中，有可能会出现**同时访问同一个资源的情况**，这种资源可以是各种类型的的资源：一个变量、一个对象、一个文件、一个数据库表等，而当多个线程同时访问同一个资源的时候，就会存在一个问题：

这个就是线程安全问题，即**多个线程同时访问一个资源时，会导致程序运行结果并不是想看到的结果**。

这个资源被称为：**临界资源**（也有称为共享资源）。

## 2.如何解决线程安全问题？

基本上所有的并发模式在解决线程安全问题时，都采用“序列化访问临界资源”的方案，即在**同一时刻，只能有一个线程访问临界资源**，也称作同步互斥访问。

是在访问临界资源的代码前面加上一个锁，当访问完临界资源后释放锁，让其他线程继续访问。

在Java中，提供了两种方式来实现同步互斥访问：**synchronized和Lock**。


## 3.synchronized同步方法或者同步块

在Java中，**每一个对象都拥有一个锁标记（monitor）**，也称为监视器，多线程同时访问某个对象时，线程只有获取了该对象的锁才能访问。

在Java中，可以使用synchronized关键字来标记一个方法或者代码块，当某个线程调用该对象的synchronized方法或者访问synchronized代码块时，**这个线程便获得了该对象的锁，其他线程暂时无法访问这个方法**，只有等待这个方法执行完毕或者代码块执行完毕，这个线程才会释放该对象的锁，其他线程才能执行这个方法或者代码块。

## 4.不使用Synchronized和ReentrantLock，如何实现一个线程安全的单例模式？

答：可有三种方案：

（1）使用饿汉式模式实现；

（2）使用static定义静态代码块，借助类加载机制实现；

（3）使用静态内部类实现，相比较前两种方式，有所优化，具有lazy-loading的特点；

（4）使用枚举的方式实现，不仅能够避免多线程同步问题，还能防止反序列化重新创建对象。

## 5.synchronized怎么升级成重量级锁

​	偏向锁，轻量级锁都是乐观锁，重量级锁是悲观锁。

​     一个对象刚开始实例化的时候，没有任何线程来访问它的时候。它是可偏向的，意味着，它现在认为只可能有一个线程来访问它，所以当第一个线程来访问它的时候，它会偏向这个线程，此时，对象持有偏向锁。偏向第一个线程，这个线程在修改对象头成为偏向锁的时候使用CAS操作，并将对象头中的ThreadID改成自己的ID，之后再次访问这个对象时，只需要对比ID，不需要再使用CAS在进行操作。

​	一旦有第二个线程访问这个对象，因为偏向锁不会主动释放，所以第二个线程可以看到对象时偏向状态，这时表明在这个对象上已经存在竞争了，检查原来持有该对象锁的线程是否依然存活，如果挂了，则可以将对象变为无锁状态，然后重新偏向新的线程，如果原来的线程依然存活，则马上执行那个线程的操作栈，检查该对象的使用情况，如果仍然需要持有偏向锁，则偏向锁升级为轻量级锁，（**偏向锁就是这个时候升级为轻量级锁的**）。如果不存在使用了，则可以将对象回复成无锁状态，然后重新偏向。

​	轻量级锁认为竞争存在，但是竞争的程度很轻，一般两个线程对于同一个锁的操作都会错开，或者说稍微等待一下（自旋），另一个线程就会释放锁。 但是当自旋超过一定的次数，或者一个线程在持有锁，一个在自旋，又有第三个来访时，轻量级锁膨胀为重量级锁，重量级锁使除了拥有锁的线程以外的线程都阻塞，防止CPU空转。
作者：EnjoyMoving链接：https://www.zhihu.com/question/53826114/answer/236363126

## 6. sychronized锁住了什么

synchronized锁的是**对象**，并且有三种情况：

第一种是对于同步方法，即修饰某个函数，这个时候锁对象是**当前类的实例化对象。**

第二种是对于同步静态方法，这个时候锁是当前**类的Class对象。**

第三种就是后面跟括号，对于同步代码块,锁是括号里的对象。

线程发现加锁后，找到加锁的对象，对象上有几个字节标志哪个线程得到了锁，标志位。

每个object有一个monitor，这东西其实就是个recursive lock，调用synchronized方法或者进入synchronized block的时候，当前thread就会尝试获取这个recursive lock，如果这个lock当前没有owner或者owner就是当前thread，则monitor的owner设为当前thread并且的锁定计数加一，否则当前thread就会一直等待到锁被释放，当退出sychronized block的时候锁定计数减一，减至0则释放锁。

作者：徐辰链接：https://www.zhihu.com/question/57794716/answer/614257890


# synchronized的缺陷

如果一个代码块被synchronized修饰了，当一个线程获取了对应的锁，并执行该代码块时，**其他线程便只能一直等待，等待获取锁的线程释放锁**，而这里获取锁的线程释放锁只会有两种情况：

1. 获取锁的线程执行完了该代码块，然后线程释放对锁的占有；

1. 线程执行发生异常，此时JVM会让线程自动释放锁。

采用synchronized关键字来实现同步的话，就会导致一个问题：

如果多个线程都只是进行读操作，所以当一个线程在进行读操作时，其他线程**只能等待无法进行读操作**。

因此就需要一种机制来使**得多个线程都只是进行读操作时**，线程之间不会发生冲突，通过Lock就可以办到。

# java.util.concurrent.locks包下常用的类

## 1.Lock

Lock是一个接口：

```java
public interface Lock {
void lock();
void lockInterruptibly() throws InterruptedException;
boolean tryLock();
boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
void unlock();
Condition newCondition();
}
```

下面来逐个讲述Lock接口中每个方法的使用，lock()、tryLock()、tryLock(long time, TimeUnit unit)和lockInterruptibly()是用来获取锁的。unLock()方法是用来释放锁的。

由于在前面讲到如果采用Lock，**必须主动去释放锁**，并且在发生异常时，**不会自动释放锁**。因此一般来说，使用Lock必须**在try{}catch{}块中进行**，并且将释放锁的操作放在finally块中进行，以保证锁一定被被释放，防止死锁的发生。通常使用Lock来进行同步的话，是以下面这种形式去使用的：
    
```java
Lock lock = ...;
lock.lock();
try{
//处理任务
}catch(Exception ex){
 
}finally{
lock.unlock();   //释放锁
}
```

tryLock()方法是有返回值的，它表示用来尝试获取锁，如果获取成功，则返回true，如果获取失败（即锁已被其他线程获取），则返回false，也就说这个方法无论如何都会立即返回。在拿不到锁时不会一直在那等待。

tryLock(long time, TimeUnit unit)方法和tryLock()方法是类似的，只不过区别在于这个方法在拿不到锁时会等待一定的时间，在时间期限之内如果还拿不到锁，就返回false。如果如果一开始拿到锁或者在等待期间内拿到了锁，则返回true。

所以，一般情况下通过tryLock来获取锁时是这样使用的：

```java
Lock lock = ...;
if(lock.tryLock()) {
 try{
     //处理任务
 }catch(Exception ex){
     
 }finally{
     lock.unlock();   //释放锁
 } 
}else {
//如果不能获取锁，则直接做其他事情
}
```

lockInterruptibly()方法比较特殊，当通过这个方法去获取锁时，如果线程正在等待获取锁，则这个线程能够响应中断，即中断线程的等待状态。也就使说，当两个线程同时通过lock.lockInterruptibly()想获取某个锁时，假若此时线程A获取到了锁，而线程B只有在等待，那么对线程B调用threadB.interrupt()方法能够中断线程B的等待过程。

由于lockInterruptibly()的声明中抛出了异常，所以lock.lockInterruptibly()必须放**在try块中或者在调用lockInterruptibly()的方法外声明抛出InterruptedException。**

一般的使用形式如下：

```java
public void method() throws InterruptedException {
lock.lockInterruptibly();
try {  
 //.....
}
finally {
    lock.unlock();
}  
}
```

用synchronized修饰的话，当一个线程处于等待某个锁的状态，是**无法被中断的，只有一直等待下**去。

## 2.ReentrantLock

ReentrantLock，意思是“可重入锁”。ReentrantLock是唯一实现了Lock接口的类。

相比于 synchronized，它多了以下高级功能：

- 等待可中断

当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。

- 可实现公平锁

公平锁是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁。

synchronized 中的锁是非公平的，ReentrantLock 默认情况下也是非公平的，但可以通过带布尔值的构造函数要求使用公平锁。

- 锁绑定多个条件

一个 ReentrantLock 对象可以同时绑定多个 Condition 对象。

## 3.ReadWriteLock

ReadWriteLock也是一个接口，在它里面只定义了两个方法：

```java
public interface ReadWriteLock {
/**
 * Returns the lock used for reading.
 *
 * @return the lock used for reading.
 */
Lock readLock();
 
/**
 * Returns the lock used for writing.
 *
 * @return the lock used for writing.
 */
Lock writeLock();
}
```

也就是说将文件的读写操作分开，**分成2个锁来分配给线程**，从而使得**多个线程可以同时进行读操作**。

## 4.ReentrantReadWriteLock

ReentrantReadWriteLock里面提供了很多丰富的方法，不过最主要的有两个方法：readLock()和writeLock()用来获取读锁和写锁。

如果有一个线程已经占用了读锁，则此时其他线程如果要申请写锁，则申请写锁的线程会**一直等待释放读锁**。

如果有一个线程已经占用了写锁，则此时其他线程如果申请写锁或者读锁，则申请的线程会**一直等待释放写锁**。

## 5.synchronized与lock的区别

- (用法）synchronized（隐式锁）：在需要同步的对象中加入此控制，synchronized 可以加在方法上，也可以加在特定代码块中，括号中表示需要锁的对象，不会发生死锁。
- （用法）lock（显示锁）：需要显示指定起始位置和终止位置。一般使用 ReentrantLock 类做为锁，多个线程中必须要使用一个 ReentrantLock 类做为对象才能保证锁的生效。且在加锁和解锁处需要通过 lock() 和 unlock() 显示指出。所以一般会在 finally 块中写 unlock() 以防死锁。
- （性能）synchronized 是托管给 JVM 执行的，而 lock 是 Java 写的控制锁的代码。在 Java1.5 中，synchronize 是性能低效的。因为这是一个**重量级操作**，需要调用操作接口，导致有可能加锁消耗的系统时间比加锁以外的操作还多。相比之下使用 Java 提供的 Lock 对象，性能更高一些。但是到了 Java1.6 ，发生了变化。synchronize 在语义上很清晰，可以进行很多优化，**有适应自旋，锁消除，锁粗化，轻量级锁，偏向锁等等**。导致 在 Java1.6 上 synchronize 的性能并不比 Lock 差。
- （机制）synchronized 原始采用的是 **CPU 悲观锁机制，即线程获得的是独占锁**。独占锁意味着其他线程只能依靠阻塞来等待线程释放锁。Lock 用的是**乐观锁**方式。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。**乐观锁实现的机制就是 CAS 操作**（Compare and Swap）。
- synchronized不可中断，除非抛出异常或正常完成，Lock可以响应中断，设置超时时间
- 锁绑定了多个条件condition，syc不能精确唤醒线程，要么唤醒一个要么全部唤醒。

## 6.Lock和synchronized的选择

1. Lock是一个**接口**，而synchronized是Java中的**关键字**，synchronized是内置的语言实现；

1. synchronized**在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生**；而Lock**在发生异常时，如果没有主动通过unLock()去释放锁，则很可能造成死锁现象，因此使用Lock时需要在finally块中释放锁**；

1. Lock可以**让等待锁的线程响应中断**，而synchronized却不行，使用synchronized时，等待的线程会一直等待下去，**不能够响应中断**；

1. 通过Lock可以知道**有没有成功获取锁**，而synchronized却无法办到。

1. Lock可以**提高多个线程进行读操作的效率**。

# synchronized的优化

- 偏向锁

- 轻量级锁

- 自旋锁

- 锁消除

- 锁粗化

# synchronized底层原理

**synchronized 关键字底层原理属于 JVM 层面。**

**① synchronized 同步语句块的情况**

```java
public class SynchronizedDemo {
	public void method() {
		synchronized (this) {
			System.out.println("synchronized 代码块");
		}
	}
}
```

通过 JDK 自带的 javap 命令查看 SynchronizedDemo 类的相关字节码信息：首先切换到类的对应目录执行 `javac SynchronizedDemo.java` 命令生成编译后的 .class 文件，然后执行`javap -c -s -v -l SynchronizedDemo.class`。

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/Syc%20code.png)

synchronized 同步语句块的**实现使用的是 monitorenter 和 monitorexit 指令，其中 monitorenter 指令指向同步代码块的开始位置，monitorexit 指令则指明同步代码块的结束位置。**当执行 monitorenter 指令时，**线程试图获取锁也就是获取 monitor**(monitor对象存在于每个Java对象的对象头中，synchronized 锁便是通过这种方式获取锁的，也是为什么Java中任意对象可以作为锁的原因) 的持有权。当计数器为0则可以成功获取，获取后将锁计数器设为1也就是加1。相应的在执行 monitorexit 指令后，将锁计数器设为0，表明锁被释放。如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止。

**② synchronized 修饰方法的的情况**

```java
public class SynchronizedDemo2 {
	public synchronized void method() {
		System.out.println("synchronized 方法");
	}
}
```

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/Syc%20method.png)

synchronized 修饰的方法并没有 monitorenter 指令和 monitorexit 指令，取得代之的确实是 **ACC_SYNCHRONIZED 标识，该标识指明了该方法是一个同步方法，JVM 通过该 ACC_SYNCHRONIZED 访问标志来辨别一个方法是否声明为同步方法，从而执行相应的同步调用。**