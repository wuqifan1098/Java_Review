# 面试题

## 1. sleep()和wait()区别

1、这两个方法来自不同的类分别是，**sleep来自Thread类，wait来自Object类。**

2、最主要是sleep方法**没有释放锁**，而wait方法**释放了锁**，使得其他线程可以使用同步控制块或者方法。

3、使用范围：wait，notify和notifyAll**只能在同步控制方法或者同步控制块里面使用**，而sleep可以**在任何地方使用**

4、sleep必须捕获异常，而wait，notify和notifyAll不需要捕获异常

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

# 三.生产者-消费者模型的实现

