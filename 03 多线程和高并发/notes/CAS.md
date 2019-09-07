# 面试题 #

## 1. 手写实现CAS(有赞) ##

```java
import java.lang.reflect.Field;
import sun.misc.Unsafe;
public class DemoCas {
 
	int i = 0;
	//Unsafe unsafe = Unsafe.getUnsafe()//不能直接使用，需要通过反射来获取
	static Unsafe unsafe; // java 中操作内存的一个类
	static long valueOffset; //内存中的地址(偏移量)
	static {
		try{//通过反射机制去拿unsafe
			Field filed = Unsafe.class.getDeclaredField("theUnsafe"); //拿到theUnsafe
			filed.setAccessible(true);  //因为是私有，设置成可访问
            //拿到unsafe  参数是要传一个对象，因为是静态的，没有对象所以传Null
			unsafe = (Unsafe) filed.get(null); 
			//通过数据去拿到内存中的i地址（偏移量）
			//直接操作内存，获取属性的偏移量（同它来定位对象内具体属性的内存地址）
			valueOffset = unsafe.objectFieldOffset(DemoCas.class.getDeclaredField("i"));
		}catch(Exception e){
			e.printStackTrace();
		}
	}
	public void incr(){
		//比较和替换
		int current =0;//内存中的值,当前值
		do{
		current = unsafe.getIntVolatile(this,valueOffset); //获取当前内存中的值
		}while(!unsafe.compareAndSwapInt(this,valueOffset,current,current+1));
		
	}
	public static void main(String[] args) {
		DemoCas demo = new DemoCas();
		for(int i =0;i<2;i++){
			new Thread(()->{
				for(int j=0;j<10000;j++){
					demo.incr();
				}
			}).start();
		}
		try {
			Thread.sleep(2000);
			System.out.println("i="+demo.i);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
	}
}
```

通过Unsafe 来获取 内存中的属性i，然后通过方法：

Unsafe.compareAndSwapInt(Object var1, long var2, int var4, int var5) 来进行比较交换实现cas机制。var1 表示当前操作的对象，var2 表示操作的属性在对象中的位置(偏移量)，var4表示内存中的属性值(上图的A)，var5表示修改后的属性值(上图的B)

主要是do，while轮询

原文链接：https://blog.csdn.net/qq_28332595/article/details/87967742



## 前言

​    在多并发程序设计之中，我们不得不面对并发、互斥、竞争、死锁、资源抢占等等问题，归根到底就是读写的问题，有了读写才有了增删改查，才有了所有的一切，同样的也有了谁读谁写，这样的顺序和主次问题，于是就有了上锁，乐观锁和悲观锁，同步和异步，睡眠和换入换出等问题，归根到底就是模拟了社会上的分工协作与资源共享和抢占，要理解好这些现象的本质，我们需要更加深刻地进行类比和辨析，要知道这些内容的本质就是内存和CPU之间的故事，有的时候还会有一些外存或者其他缓存。只有我们深刻的理解了这些内容背后的原理和本质，我们才能算是真正的有所感悟，于是我们就需要理解操作系统的保护模式和进程线程的运行机理，需要理解计算机组成的基本原理，明白硬件的基本结构，运行的基本单位，存储和寄存器等多种电器元件，在理解了软件和硬件的基础之上，我们还要有一些编译原理方面的知识，因为编译器将我们能看到的程序语言翻译成了一个个中间代码直到机器码，只有明白了这一点才会知道i++，这样的简单的代码其实是有两三条指令才能完成的，并不是原子操作，明白了这些我们才能够真正的理解多并发机制。

## CAS原理

###   本质

​    java的CAS是Compare And Swap的缩写，先进行比较再进行交换，是**实现java乐观锁的一种机制**。java.util.concurrent包完全建立在CAS之上的，借助CAS实现了区别于synchronouse同步锁的一种乐观锁。但是这种机制是有一定的问题的，会造成ABA问题，因此需要**加时间戳（版本号）**这样的机制，为什么会有悲观锁和乐观锁呢，本质上如果使用一个进程把一个资源完全锁住就称为悲观锁，直到这个进程把资源使用完之后才能解锁，这样的上锁会导致其他进程在这段时间之内一直等不到这个内存资源，但是在实际的运行之中很多进程使用内存资源都是用来读操作的，并不会修改这个内存的内容，基于这样的实际情况，如果全部使用悲观锁，在高并发的环境下必定是**非常浪费时间和CPU**，内存资源的，因此类似CAS这样的乐观锁就出现了，主要是满足大部分进程用来读，少部分进程用来写这样的需求的。

![img](https://img2018.cnblogs.com/blog/1157683/201810/1157683-20181028142547162-1838190944.png)

​    CAS 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值(B)。如果**内存地址里面的值和A的值是一样的，那么就将内存里面的值更新成B。**CAS是通过**无限循环**来获取数据的，若果在第一轮循环中，a线程获取地址里面的值被b线程修改了，那么a线程需要自旋，到下次循环才有可能机会执行。

​    也就是什么意思呢？？让我们想一下，一个内存地址上可以存一个变量，如果不对这个变量进行悲观锁（同步）的加锁方式，我们在面对着很多进程的读操作的时候肯定是没问题的，毕竟内存号称“一次写入，无数次读取”的，但是如果是写操作的时候，我们就要采取一定的办法保证如果之前已经有进程修改过这块空间的值了，这次我们的修改就不能继续下去，不然的话就会造成混乱了，因此我们需要自旋等待下次的操作时机，正是因为这样的操作机制，我们对内存的使用效率也有了很大的提高。那么我们怎么判断呢？我们知道内存的地址V,并且**采取的轮询的机制**，在这个机制之下，我们首先读取一次V的值，记为A，之后我们运行CAS算法，这个算法是一个原子操作，为什么不直接把我们需要写入的数值B写入内存呢？！原因很简单，那就是在我们读取了V的值A之后，假如此时发生了进程切换，这个内存空间被另一个进程修改了，如果此时我们直接写入B，就把上一个进程的值给取代了，上一个进程就丢失了该信息。因此，我们在下一步需要判断一下是不是有这种操作发生，如果有的话什么都不做，继续下一轮的读V的值A1,继续比较；如果值没有变，就姑且认为一切状态没发生变化，使用原子操作，比较并且替代这个内存的值。在这种机理之下，就要求这个内存V是唯一的，也就是volatile（易变）的，只在内存中有一份，在其他地方不能被缓存。这就是CAS的本质思想。

###   CAS的问题

​    1、CAS容易造成ABA问题。ABA：一个线程将某一内存地址中的数值A改成了B，接着又改成了A，此时CAS认为是没有变化，其实是已经变化过了，而这个问题的解决方案可以使用版本号标识，每操作一次version加1。在java5中，已经提供了**AtomicStampedReference**来解决问题。
​    CAS操作容易导致ABA问题,也就是在做i++之间，i可能被多个线程修改过了，只不过回到了最初的值，这时CAS会认为i的值没有变。i在外面逛了一圈回来，你能保证它没有做任何坏事，不能！！也许它把b的值减了一下，把c的值加了一下等等，更有甚者如果i是一个对象，这个对象有可能是新创建出来的，i是一个引用情况又如何，所以这里面还是存在着很多问题的，解决ABA问题的方法有很多，可以考虑增加一个修改计数，只有修改计数不变的且i值不变的情况下才做i++，也可以考虑引入版本号，当版本号相同时才做i++操作等，这和事务原子性处理有点类似。
​    2、CAS造成CPU利用率增加。之前说过了CAS里面是**一个循环判断的过程，如果线程一直没有获取到状态**，cpu资源会一直被占用。
​    3、会增加程序测试的复杂度，稍不注意就会出现问题。

​	4、只能保证一个共享变量的原子操作。当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁，或者有一个取巧的办法，就是把多个共享变量合并成一个共享变量来操作。从Java1.5开始JDK提供了AtomicReference类来保证引用对象之间的原子性，可以**把多个变量放在一个对象**里来进行CAS操作。

###   ABA的解决办法

​    如果一开始位置V得到的旧值是A，当进行赋值操作时再次读取发现仍然是A，并不能说明变量没有被其它线程改变过。有可能是其它线程将变量改为了B，后来又改回了A。大部分情况下ABA问题不会影响程序并发的正确性，如果要解决ABA问题，用传统的互斥同步可能比原子类更高效。可以用CAS在无锁的情况下实现原子操作，但要明确应用场合，非常简单的操作且又不想引入锁可以考虑使用CAS操作，当想要非阻塞地完成某一操作也可以考虑CAS。不推荐在复杂操作中引入CAS，会使程序可读性变差，且难以测试，同时会出现ABA问题。
​    ABA问题的解决办法：
​    1.在变量前面追加版本号：每次变量更新就把版本号加1，则A-B-A就变成1A-2B-3A。
​    2.atomic包下的AtomicStampedReference类：其compareAndSet方法**首先检查当前引用是否等于预期引用**，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用的该标志的值设置为给定的更新值。

## 具体例子

####   JAVA中CAS的实现

   JAVA中的CAS主要使用的是Unsafe方法，Unsafe的CAS操作主要是基于硬件平台的汇编指令，目前的处理器基本都支持CAS，只不过不同的厂家的实现不一样罢了。
   Unsafe提供了三个方法用于CAS操作，分别是:

```java
 public final native boolean compareAndSwapObject(Object value, long valueOffset, Object expect, Object update);
 public final native boolean compareAndSwapInt(Object value, long valueOffset, int expect, int update);  
 public final native boolean compareAndSwapLong(Object value, long valueOffset, long expect, long update);
```

```
    value 表示 需要操作的对象
    valueOffset 表示 对象(value)的地址的偏移量（通过Unsafe.objectFieldOffset(Field valueField)获取）
    expect 表示更新时value的期待值
    update 表示将要更新的值
```

​    具体过程为每次在执行CAS操作时，线程会根据valueOffset去内存中获取当前值去跟expect的值做对比如果一致则修改并返回true，如果不一致说明有别的线程也在修改此对象的值，则返回false。
​    Unsafe类中compareAndSwapInt的具体实现：

```
UNSAFE_ENTRY(jboolean, Unsafe_CompareAndSwapInt(JNIEnv *env, jobject unsafe, jobject obj, jlong offset, jint e, jint x)) UnsafeWrapper("Unsafe_CompareAndSwapInt"); oop p = JNIHandles::resolve(obj); jint* addr = (jint *) index_oop_from_field_offset_long(p, offset);return (jint)(Atomic::cmpxchg(x, addr, e)) == e; UNSAFE_END
```

####   AtomicInteger类中实现CAS的方法

```java
 public final int incrementAndGet() {
     for (;;) {
        int current = get();
         int next = current + 1;
          if (compareAndSet(current, next))
            return next;
      }
  }
  public final int decrementAndGet() {
     for (;;) {
         int current = get();
         int next = current - 1;
         if (compareAndSet(current, next))
             return next;
     }
 }
```

![img](https://img2018.cnblogs.com/blog/1157683/201810/1157683-20181028141542492-1913253203.png)

![img](https://img2018.cnblogs.com/blog/1157683/201810/1157683-20181028141550122-712491526.png)

   但是上面的操作会造成CAS的ABA问题，基本是这个样子：

```
     进程P1在共享变量中读到值为A
     P1被抢占了，进程P2执行
    P2把共享变量里的值从A改成了B，再改回到A，此时被P1抢占。
     P1回来看到共享变量里的值没有被改变，于是继续执行。
```

​      虽然P1以为变量值没有改变，继续执行了，但是这个会引发一些潜在的问题。ABA问题最容易发生在lock free 的算法中的，CAS首当其冲，因为CAS判断的是指针的地址。如果这个地址被重用了呢，问题就很大了。（地址被重用是很经常发生的，一个内存分配后释放了，再分配，很有可能还是原来的地址）。比如上述的DeQueue()函数，因为我们要让head和tail分开，所以我们引入了一个dummy指针给head，当我们做CAS的之前，如果head的那块内存被回收并被重用了，而重用的内存又被EnQueue()进来了，这会有很大的问题。（内存管理中重用内存基本上是一种很常见的行为）
​     **具体案例：**

```java
  package com.cas.aba;
  
  import java.util.concurrent.atomic.AtomicInteger;
  
  public class AtomicIntegerTest {
      public static AtomicInteger a = new AtomicInteger(1);
  
      public static void main(String[] args) {
          Thread main = new Thread(() -> {
             System.out.println("操作线程" + Thread.currentThread() + ",初始值 = " + a);
            // 定义变量 a = 1
             try {
                 Thread.sleep(1000);
                 // 等待1秒 ，以便让干扰线程执行
             } catch (InterruptedException e) {
                 e.printStackTrace();
             }
             boolean isCASSuccess = a.compareAndSet(1, 2);
             // CAS操作
             System.out.println("操作线程" + Thread.currentThread() + ",CAS操作结果: " + isCASSuccess);
         }, "主操作线程");
 
        Thread other = new Thread(() -> {
             Thread.yield();
            // 确保thread-main线程优先执行
            a.incrementAndGet(); // a 加 1, a + 1 = 1 + 1 = 2
             System.out.println("操作线程" + Thread.currentThread() + ",【increment】 ,值 = " + a);
             a.decrementAndGet(); // a 减 1, a - 1 = 2 - 1 = 1
             System.out.println("操作线程" + Thread.currentThread() + ",【decrement】 ,值 = " + a);
         }, "干扰线程");
         main.start();
        other.start();
     }
 }
```

![img](https://img2018.cnblogs.com/blog/1157683/201810/1157683-20181028140547071-1366868158.png)

​    可以看到发生了ABA，但是程序还是执行了CAS。

####  AtomicStampedReference类解决ABA问题：

代码

![img](https://img2018.cnblogs.com/blog/1157683/201810/1157683-20181028141424361-1908623206.png)

​    AtomicStampedReference主要维护包含一个对象引用以及一个可以自动更新的整数"stamp"的pair对象来解决ABA问题。

```java
 /**
  * 原子更新带有版本号的引用类型。
  * 该类将整数值与引用关联起来，可用于原子的更数据和数据的版本号。
  * 可以解决使用CAS进行原子更新时，可能出现的ABA问题。
  */
 public class AtomicStampedReference<V> {
  
     private static class Pair<T> {
         final T reference;
         //最好不要重复的一个数据,决定数据是否能设置成功
         final int stamp;
         private Pair(T reference, int stamp) {
             this.reference = reference;
             this.stamp = stamp;
         }
         //根据reference和stamp来生成一个Pair的实例
         static <T> Pair<T> of(T reference, int stamp) {
             return new Pair<T>(reference, stamp);
         }
     }
     //对Value 进行封装,放入Pair里面
      private volatile Pair<V> pair;
  
     /**
      * 返回当前的对象
      */
     public V getReference() {
         return pair.reference;
     }
  
     /**
      * 返回当前的stamp
      */
     public int getStamp() {
         return pair.stamp;
     }
  
     /**
      * 当当前的引用数据和期望的引用数据相等并且当前stamp和期望的stamp也相等
      * 并且
      * (当前的引用数据和新的引用数据相等并且当前stamp和新的stamp也相等
      * 或者cas操作成功
      * )
      * @param 期望(老的)的引用数据
      * @param 新的引用数据
      * @param 期望(老的)的stamp值
      * @param 新的stamp值
      * @return 
      */
     public boolean compareAndSet(V   expectedReference,
                                  V   newReference,
                                  int expectedStamp,
                                  int newStamp) {
         Pair<V> current = pair;
         return
             expectedReference == current.reference &&
             expectedStamp == current.stamp &&
             ((newReference == current.reference &&
               newStamp == current.stamp) ||
              casPair(current, Pair.of(newReference, newStamp)));
     }
  
     /**
      * 当前值和期望值比较设置
      */
     private boolean casPair(Pair<V> cmp, Pair<V> val) {
         return UNSAFE.compareAndSwapObject(this, pairOffset, cmp, val);
     }
 }
```

   **实验案例：**



```java
  package com.cas.aba;
  
  import java.util.concurrent.atomic.AtomicStampedReference;
  
  public class AtomicStampedReferenceTest {
      private static AtomicStampedReference<Integer> atomicStampedRef = new AtomicStampedReference<>(1, 0); 
      public static void main(String[] args){ 
          Thread main = new Thread(() -> {
              System.out.println("操作线程" + Thread.currentThread() +",初始值 a = " + atomicStampedRef.getReference()); 
             int stamp = atomicStampedRef.getStamp(); 
             //获取当前标识别 
             try { 
                 Thread.sleep(1000); //等待1秒 ，以便让干扰线程执行 
             } catch (InterruptedException e) {
                 e.printStackTrace();
             } 
             boolean isCASSuccess = atomicStampedRef.compareAndSet(1,2,stamp,stamp +1); 
             //此时expectedReference未发生改变，但是stamp已经被修改了,所以CAS失败 
             System.out.println("操作线程" + Thread.currentThread() +",CAS操作结果: " + isCASSuccess); 
          },"主操作线程"); 
         Thread other = new Thread(() -> { 
             Thread.yield(); // 确保thread-main 优先执行 
             atomicStampedRef.compareAndSet(1,2,atomicStampedRef.getStamp(),atomicStampedRef.getStamp() +1); 
             System.out.println("操作线程" + Thread.currentThread() +",【increment】 ,值 = "+ atomicStampedRef.getReference()); 
             atomicStampedRef.compareAndSet(2,1,atomicStampedRef.getStamp(),atomicStampedRef.getStamp() +1); 
             System.out.println("操作线程" + Thread.currentThread() +",【decrement】 ,值 = "+ atomicStampedRef.getReference()); 
         },"干扰线程");
         main.start(); 
         other.start(); 
     }
 }
```



![img](https://img2018.cnblogs.com/blog/1157683/201810/1157683-20181028141226106-1702019423.png)

##  总结

​     通过对CAS的理解和实验，我们更加深刻的理解了CAS的乐观锁，以及在java中的相应实现和对应ABA补丁的实现。

转载 ：https://www.cnblogs.com/zyrblog/p/9864932.html