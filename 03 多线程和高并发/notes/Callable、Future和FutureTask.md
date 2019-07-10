# 面试题

## 1.主线程可以捕获到子线程的异常吗？（360企业安全）

正常情况下，如果不做特殊的处理，在主线程中是**不能**够捕获到子线程中的异常的。线程方法的异常只能自己来处理。可以给某个thread设置一个UncaughtExceptionHandler。

## 2.Runnable和Thread哪一种比较好,哪一种比较安全。（360企业安全）

（1）由于Java不允许多继承，因此实现了Runnable接口可以再继承其他类，但是Thread明显不可以

（2）Runnable可以实现多个相同的程序代码的**线程去共享同一个资源**，而Thread并不是不可以，而是相比于Runnable来说，不太适合，具体原因文章中有。

# Callable、Future和FutureTask

创建线程的2种方式，一种是**直接继承Thread**，另外一种就是**实现Runnable接口**。

这2种方式都有一个缺陷就是：在执行完任务之后**无法获取执行结果**。

如果需要获取执行结果，就必须通过**共享变量或者使用线程通信**的方式来达到效果，这样使用起来就比较麻烦。

而自从Java 1.5开始，就提供了Callable和Future，通过它们可以在任务执行完毕之后得到任务执行结果。

## 一.Callable与Runnable

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

## 二.Future

Future就是对于具体的Runnable或者Callable任务的**执行结果进行取消、查询是否完成、获取结果**。必要时可以通过get方法获取执行结果，该方法会阻塞直到任务返回结果。

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

## 三.FutureTask

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

可以看出RunnableFuture继承了Runnable接口和Future接口，而FutureTask实现了RunnableFuture接口。所以它既可以作为Runnable被线程执行，又可以作为Future得到Callable的返回值。

FutureTask提供了2个构造器：

```java
public FutureTask(Callable<V> callable) {
}
public FutureTask(Runnable runnable, V result) {
}
```
FutureTask是Future接口的一个唯一实现类。

