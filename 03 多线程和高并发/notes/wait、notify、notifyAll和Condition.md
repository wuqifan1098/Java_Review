# 面试题

## 1. sleep()和wait()区别

- 这两个方法来自不同的类分别是，**sleep来自Thread类，wait来自Object类。**


- 最主要是sleep方法**没有释放锁**，而wait方法**释放了锁**，使得其他线程可以使用同步控制块或者方法。


- 使用范围：wait，notify和notifyAll**只能在同步控制方法或者同步控制块里面使用**，而sleep可以**在任何地方使用**


- sleep**必须捕获异常**，而wait，notify和notifyAll**不需要捕获异常**

## 2.notify和notifyAll有什么区别？

- 如果线程调用了对象的 wait()方法，那么线程便会处于该对象的等待池中，等待池中的线程不会去竞争该对象的锁。
- 当有线程调用了对象的 notifyAll()方法**（唤醒所有 wait 线程）**或 notify()方法**（只随机唤醒一个 wait 线程）**，被唤醒的的线程便会进入该对象的锁池中，锁池中的线程会去竞争该对象锁。也就是说，调用了notify后只有一个线程会由等待池进入锁池，而notifyAll会将该对象等待池内的所有线程移动到锁池中，等待锁竞争
- **优先级高的线程竞争到对象锁的概率大**，假若某线程没有竞争到该对象锁，它还会留在锁池中，唯有线程再次调用 wait()方法，它才会重新回到等待池中。而竞争到对象锁的线程则继续往下执行，直到执行完了 synchronized 代码块，它会释放掉该对象锁，这时锁池中的线程会继续竞争该对象锁。
- 尽量使用 notifyAll()，notify()**可能会导致死锁**

## 3. notify()发生死锁的原因，如何正确的使用notify()

导致死锁的原因是notify()随机唤醒了一条线程，但它既无法正确改变条件，也不叫醒另一个兄弟来搞，就会产生一个情况：**锁池中的队列空了，等待池中有一堆线程，但不会再被唤醒永远等待。**

正确的场景应该是 WaitSet中**等待的是相同的条件，唤醒任一个都能正确处理接下来的事项**，如果唤醒的线程无法正确处理，务必确保继续notify()下一个线程，并且自身需要重新回到WaitSet中（参见下一条）

## 4. 为什么Object.wait(),Object.notify(),Object.notifyAll()必须在同步块中执行呢?

- 调用wait()就是释放锁，**释放锁的前提是必须要先获得锁**，先获得锁才能释放锁。
- notify(),notifyAll()是**将锁交给含有wait()方法的线程**，让其继续执行下去，如果自身没有锁，怎么叫把锁交给其他线程呢；（本质是让处于入口队列的线程竞争锁）

<https://blog.csdn.net/qq_42145871/article/details/81950949>

## 5.为什么wait(),notify()是Object方法？

**一个很明显的原因是JAVA提供的锁是对象级的而不是线程级的，每个对象都有锁，通过线程获得。**如果线程需要等待某些锁那么调用对象中的wait()方法就有意义了。如果wait()方法定义在Thread类中，线程正在等待的是哪个锁就不明显了。

# wait()、notify()和notifyAll()

wait()、notify()和notifyAll()是Object**类中的方法**：

```java
public final native void notify();

public final native void notifyAll();
 
public final native void wait(long timeout) throws InterruptedException;
```

1）wait()、notify()和notifyAll()方法是本地方法，**并且为final方法，无法被重写**。

2）调用某个对象的wait()方法能让**当前线程阻塞**，并且**当前线程必须拥有此对象的monitor**（即锁）

3）调用某个对象的notify()方法能够唤醒一个正在等待这个对象的monitor的线程，如果有多个线程都在等待这个对象的monitor，则只能唤醒其中一个线程；

4）调用notifyAll()方法能够唤醒所有正在等待这个对象的monitor的线程；

## 锁池和等待池

- 锁池:假设线程A已经拥有了某个对象(注意:不是类)的锁，而其它的线程想要调用这个对象的某个synchronized方法(或者synchronized块)，由于这些线程在进入对象的synchronized方法之前**必须先获得该对象的锁的拥有权**，但是该对象的锁目前正被线程A拥有，所以这些**线程就进入了该对象的锁池中**。
- 等待池:假设一个线程A**调用了某个对象的wait()方法**，线程A就会释放该对象的锁后，**进入该对象的等待池中**

> Reference：[java中的锁池和等待池 ](https://link.zhihu.com/?target=http%3A//blog.csdn.net/emailed/article/details/4689220)

## notify和notifyAll的区别

- 如果线程调用了对象的 wait()方法，那么线程便会处于该对象的**等待池**中，等待池中的线程**不会去竞争该对象的锁**。
- 当有线程调用了对象的 **notifyAll**()方法（唤醒所有 wait 线程）或 **notify**()方法（**只随机唤醒一个 wait 线程**），被唤醒的的线程便会进入该对象的锁池中，锁池中的线程会去竞争该对象锁。也就是说，调用了notify后只有一个线程会由等待池进入锁池，而notifyAll会将该对象等待池内的所有线程移动到锁池中，等待锁竞争
- **优先级高的线程竞争到对象锁的概率大**，假若某线程没有竞争到该对象锁，它**还会留在锁池中**，唯有线程再次调用 wait()方法，它才会重新回到等待池中。而竞争到对象锁的线程则继续往下执行，直到执行完了 synchronized 代码块，它会释放掉该对象锁，这时锁池中的线程会继续竞争该对象锁。

notifyAll()将所有WaitSet中的线程从等待池唤醒，全部进入锁池竞争去sync锁，最终也只有一个线程能获取锁去执行，**唤醒+竞争锁池本身是线程上下文的重操作，对性能产生不良影响。**滥用notifyAll()有可能导致“惊群效应”。

notify() 是对notifyAll()的一个优化，但它有很精确的应用场景，**并且要求正确使用。不然可能导致死锁。**正确的场景应该是 WaitSet中**等待的是相同的条件，唤醒任一个都能正确处理接下来的事项**，如果唤醒的线程无法正确处理，务必确保继续notify()下一个线程，并且自身需要重新回到WaitSet中（参见下一条）。导致死锁的原因是**notify()随机唤醒了一条线程，但它既无法正确改变条件，也不叫醒另一个兄弟来搞，就会产生一个情况：锁池中的队列空了，等待池中有一堆线程，但不会再被唤醒永远等待。**

**wait() 应配合while循环使用，不应使用if，务必在wait()调用前后都检查条件，如果不满足，必须调用notify()唤醒另外的线程来处理，自己继续wait()直至条件满足再往下执行。**

![img](https://pic2.zhimg.com/80/v2-eaf7782a87e68dd1e3630cf1e900ac97_hd.jpg)

**总结**

- 如果线程调用了对象的 wait()方法，那么线程便会处于该对象的等待池中，等待池中的线程不会去竞争该对象的锁。
- 当有线程调用了对象的 notifyAll()方法（唤醒所有 wait 线程）或 notify()方法（只随机唤醒一个 wait 线程），被唤醒的的线程便会进入该对象的锁池中，锁池中的线程会去竞争该对象锁。也就是说，调用了notify后只要一个线程会由等待池进入锁池，而notifyAll会将该对象等待池内的所有线程移动到锁池中，等待锁竞争
- 优先级高的线程竞争到对象锁的概率大，假若某线程没有竞争到该对象锁，它还会留在锁池中，唯有线程再次调用 wait()方法，它才会重新回到等待池中。而竞争到对象锁的线程则继续往下执行，直到执行完了 synchronized 代码块，它会释放掉该对象锁，这时锁池中的线程会继续竞争该对象锁。
- 尽量使用 notifyAll()，notify()可能会导致死锁

作者：大王叫我来巡山

## Demo

```java
public class waitAndNotifyDemo(){
    public static void main(String[] args){
        waitAndNotifyDemo test = new waitAndNotifyDemo();
        test.testWait();
    }
    Object obj = new Object();//创建一个全局变量
    ThreadLocal<AtomicInteger> num = new ThreadLocal<AtomicInteger>();//设置一个线程wait和notify的触发条件
    class MyRunner implements Runnable{
        @Override
        public void run(){
            num.set(new AtomicInteger(0));
            while(true){
                
            }
        }
    }
}
```

https://www.cnblogs.com/PerkinsZhu/p/7439330.html

# Condition

Condition是在java 1.5中才出现的，它用来替代传统的Object的wait()、notify()实现线程间的协作，相比使用Object的wait()、notify()，使用Condition1的**await()、signal()这种方式实现线程间协作更加安全和高效**。因此通常来说比较推荐使用Condition，在阻塞队列那一篇博文中就讲述到了，阻塞队列实际上是使用了Condition来模拟线程间协作。

- Condition是个**接口**，基本的方法就是await()和signal()方法；
- Condition**依赖于Lock接口**，生成一个Condition的基本代码是lock.newCondition() 
-  调用Condition的**await()和signal()方法**，都必须在lock保护之内，就是说必须在lock.lock()和lock.unlock之间才可以使用

　　Conditon中的await()对应Object的wait()；

　　Condition中的signal()对应Object的notify()；

　　Condition中的signalAll()对应Object的notifyAll()。

# 生产者-消费者模型

