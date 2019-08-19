# 面试题

## 1. 主线程可以捕获到子线程的异常吗？（360企业安全）

正常情况下，如果不做特殊的处理，在主线程中是**不能**够捕获到子线程中的异常的。线程方法的异常只能自己来处理。可以给某个thread设置一个UncaughtExceptionHandler。

## 2. Runnable和Thread哪一种比较好,哪一种比较安全。（360企业安全）

（1）由于Java不允许多继承，因此实现了**Runnable接口可以再继承其他类，但是Thread明显不可以**

（2）Runnable可以实现多个相同的程序代码的**线程去共享同一个资源**。

如果每个线程执行的代码相同，可以使用同一个Runnable对象，然后将**共享的数据**放在Runnable里面，来实现数据的共享。 

如果每个线程执行的代码不同， 那么就需要不同的Runnable对象

https://www.cnblogs.com/E-star/p/3482170.html

其实在实现一个任务用多个线程来做也可以用继承Thread类来实现只是比较麻烦，一般我们用实现Runnable接口来实现，简洁明了。

Thread比较安全，每个线程执行自己的Thread对象中的代码

Runnable由几个对象共同执行一个Runnable对象，可能会造成线程的不安全。如果不对多线程代码进行同步的话，还是会造成数据不一致问题。

## 3. Futuretask 怎么返回计算值的 线程怎么拿到另一个线程的计算结果

1）FutureTask通过MyCallable创建；

2）new Thread()创建线程，通过start()方法启动线程；

3）执行FutureTask中的run()方法；

4）run()方法中调用了MyCallable中的call()方法，拿到返回值set到FutureTask的outcome成员变量中；

5）FutureTask通过get方法获取outcome对象值，从而成功拿到线程执行的返回值；

# Callable、Future和FutureTask

创建线程的2种方式，一种是**直接继承Thread**，另外一种就是**实现Runnable接口**。

这2种方式都有一个缺陷就是：在执行完任务之后**无法获取执行结果**。

如果需要获取执行结果，就必须通过**共享变量或者使用线程通信**的方式来达到效果，这样使用起来就比较麻烦。

而自从Java 1.5开始，就提供了Callable和Future，通过它们可以在任务执行完毕之后得到任务执行结果。

## Callable与Runnable

先说一下java.lang.Runnable吧，它是一个接口，在它里面**只声明了一个run()方法**：

```java
public interface Runnable {
public abstract void run();
}
```
由于run()方法返回值为void类型，所以在**执行完任务之后无法返回任何结果**。

Callable位于java.util.concurrent包下，它也是一个接口，在它里面也**只声明了一个方法，只不过这个方法叫做call()**：

```java
public interface Callable<V> {
/**
	* Computes a result, or throws an exception if unable to do so.
 	*
 	* @return computed result
 	* @throws Exception if unable to compute a result
 */
V call() throws Exception;
}
```

一个泛型接口，call()函数返回的类型就是传递进来的V类型。

那么怎么使用Callable呢？一般情况下是配合**ExecutorService**来使用的，在ExecutorService接口中声明了若干个submit方法的重载版本：

```java
<T> Future<T> submit(Callable<T> task);
<T> Future<T> submit(Runnable task, T result);
Future<?> submit(Runnable task);
```

第一个submit方法里面的参数类型就是Callable。

暂时只需要知道Callable一般是和ExecutorService配合来使用的，具体的使用方法讲在后面讲述。

一般情况下我们使用第一个submit方法和第三个submit方法，第二个submit方法很少使用。

## Future

Future就是对于具体的Runnable或者Callable任务的**执行结果进行取消、查询是否完成、获取结果**。必要时可以通过get方法获取执行结果，该方法**会阻塞直到任务返回结果。**

Future类位于java.util.concurrent包下，它是一个接口：

```java
public interface Future<V> {
boolean cancel(boolean mayInterruptIfRunning);
boolean isCancelled();
boolean isDone();
V get() throws InterruptedException, ExecutionException;
V get(long timeout, TimeUnit unit)
    throws InterruptedException, ExecutionException, TimeoutException;
}
```

在Future接口中声明了5个方法，下面依次解释每个方法的作用：

- cancel方法用来取消任务，如果取消任务成功则返回true，如果取消任务失败则返回false。参数mayInterruptIfRunning表示是否允许取消正在执行却没有执行完毕的任务，如果设置true，则表示可以取消正在执行过程中的任务。如果任务已经完成，则无论mayInterruptIfRunning为true还是false，此方法肯定返回false，即如果取消已经完成的任务会返回false；如果任务正在执行，若mayInterruptIfRunning设置为true，则返回true，若mayInterruptIfRunning设置为false，则返回false；如果任务还没有执行，则无论mayInterruptIfRunning为true还是false，肯定返回true。

- isCancelled方法表示任务是否被取消成功，如果在任务正常完成前被取消成功，则返回 true。

- isDone方法表示任务是否已经完成，若任务完成，则返回true；

- get()方法用来获取执行结果，这个方法会产生阻塞，会一直等到任务执行完毕才返回；

- get(long timeout, TimeUnit unit)用来获取执行结果，如果在指定时间内，还没获取到结果，就直接返回null。

　也就是说Future提供了三种功能：

　　1）判断任务是否完成；

　　2）能够中断任务；

　　3）能够获取任务执行结果。

　　因为Future只是一个接口，所以是**无法直接用来创建对象使用的**，因此就有了下面的FutureTask。

## FutureTask

先来看一下FutureTask的实现：

```java
public class FutureTask<V> implements RunnableFuture<V>
```

FutureTask类实现了RunnableFuture接口，我们看一下RunnableFuture接口的实现：

```java
public interface RunnableFuture<V> extends Runnable, Future<V> {
void run();
}
```

可以看出RunnableFuture继承了Runnable接口和Future接口，而FutureTask实现了RunnableFuture接口。**所以它既可以作为Runnable被线程执行，又可以作为Future得到Callable的返回值。**

FutureTask提供了2个构造器：

```java
public FutureTask(Callable<V> callable) {
}
public FutureTask(Runnable runnable, V result) {
}
```
FutureTask是Future接口的一个唯一实现类。

## 获取线程返回值

```java
package com.lanhuigu.demo.createthread;
 
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;
 
public class MyCallable implements Callable<String> {
 
    /**
     * 实现Callable中的call方法
     * @author yihonglei
     * @date 2018/9/12 17:01
     */
    public String call() throws Exception {
        return "Test Callable";
    }
 
    public static void main(String[] args) {
        /** 根据MyCallable创建FutureTask对象 */
        FutureTask<String> futureTask = new FutureTask<>(new MyCallable());
        try {
            /** 启动线程 */
            new Thread(futureTask).start();
            /** 获取线程执行返回值 */
            String s = futureTask.get();
            /** 打印返回值 */
            System.out.println(s);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}
```

结果

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/20190819154933.png)

成功拿到了线程执行的返回值。

创建线程对象，通过start()方法启动线程：

```java
/** 启动线程 */
new Thread(futureTask).start();
```

start()方法源码如下：

```java
public synchronized void start() {
    if (threadStatus != 0)
        throw new IllegalThreadStateException();
 
    group.add(this);
 
    boolean started = false;
    try {
        start0();
        started = true;
    } finally {
        try {
            if (!started) {
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {
            /* do nothing. If start0 threw a Throwable then
            it will be passed up the call stack */
        }
    }
}
// 本地方法
private native void start0();
```

start()方法最后会**调用本地方法，由JVM通知操作系统，创建线程，最后线程通过JVM访问到Runnable中的run()方法。**

而FutureTask实现了Runnable的run()方法，看下FutureTask中的run()方法源码：

```java
public void run() {
    if (state != NEW ||
        !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                     null, Thread.currentThread()))
        return;
    try {
        /** 
          这里的callable就是我们创建FutureTask的时候传进来的MyCallable对象，
          该对象实现了Callable接口的call()方法。
        */
        Callable<V> c = callable;
        if (c != null && state == NEW) {
            V result;
            boolean ran;
            try {
                /** 
                  调用Callable的call方法，即调用实现类MyCallable的call()方法，执行完会拿到MyCallable的call()方法的返回值“Test Callable”。
                */
                result = c.call();
                ran = true;
            } catch (Throwable ex) {
                result = null;
                ran = false;
                setException(ex);
            }
            if (ran)
                /** 将返回值传入到set方法中，这里是能获取线程执行返回值的关键 */
                set(result);
        }
    } finally {
        // runner must be non-null until state is settled to
        // prevent concurrent calls to run()
        runner = null;
        // state must be re-read after nulling runner to prevent
        // leaked interrupts
        int s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
}
```

**从run()方法源码可以知道，MyCallabel执行call()方法的返回值被传入到了一个set()方法中，能拿到线程返回值最关键的**

**就是这个FutureTask的set()方法源码**：

```java
protected void set(V v) {
    if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
        /** 
          将MyCallable执行call()方法的返回值传进来赋值给了outcome,
          这个outcome是FutureTask的一个成员变量。
          该变量用于存储线程执行返回值或异常堆栈，通过对应的get()方法获取值。
          private Object outcome;
        */
        outcome = v;
        UNSAFE.putOrderedInt(this, stateOffset, NORMAL); // final state
        finishCompletion();
    }
}
```

到这里，得出一个结论就是，MyCallable执行的call()方法结果通过FutureTask的set()方法存到了成员变量outcome中。

**FutureTask中get()方法的源码：**

```java
public V get() throws InterruptedException, ExecutionException {
    int s = state;
    if (s <= COMPLETING)
        s = awaitDone(false, 0L);
    /** 调用report方法 */
    return report(s);
}
 
private V report(int s) throws ExecutionException {
    /** outcome赋值给Object x */
    Object x = outcome;
    if (s == NORMAL)
       /** 返回outcome的值，也就是线程执行run()方法时通过set()方法放进去的MyCallable的call()执行的返回值 */
       return (V)x;
    if (s >= CANCELLED)
        throw new CancellationException();
    throw new ExecutionException((Throwable)x);
}
```

get()方法调用report()方法，report()方法会将outcome赋值并返回，set方法成功拿到返回的outcome，

也就是MyCallable()的call()方法执行结果。

到这里，我们大概理解了FutureTask.get()能拿到线程执行返回值的本质原理，也就基于FutureTask的成员变量

outcome进行的set赋值和get取值的过程。

### 总结

下面简单总结一下过程：

1）FutureTask通过MyCallable创建；

2）new Thread()创建线程，通过start()方法启动线程；

3）执行FutureTask中的run()方法；

4）run()方法中调用了MyCallable中的call()方法，拿到返回值set到FutureTask的outcome成员变量中；

5）FutureTask通过get方法获取outcome对象值，从而成功拿到线程执行的返回值；

其实，本质上就是一个基于FutureTask成员变量outcome进行的set和get的过程，饶了一圈而已。
原文链接：https://blog.csdn.net/yhl_jxy/article/details/82664829