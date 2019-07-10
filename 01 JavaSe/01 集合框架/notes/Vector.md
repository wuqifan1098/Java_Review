# 面试题

## 1. Vector线程安全问题

虽然源代码注释里面说这个是线程安全的，因为确实很多方法都加上了同步关键字synchronized，但是对于符合操作而言，只是同步方法并没有解决线程安全的问题。

要真正达成线程安全，还需要以vector对象为锁，来进行操作。

所以，如果是这样的话，那么用vector和ArrayList就没有区别了，所以，不推荐使用vector。

当然会有安全问题，比如说另一个线程持有一个迭代器对象，那么会导致迭代器状态无效。你有两个办法，一个是锁住向量变量，一个是查询的时候先复制一个vector的副本。关键看你对同步的要求和为读还是写优化（程序里查询的多还是修改的多）

# Vector

## 概览

实现了**List接口，继承了AbstractList**。

实现了 **RandomAccess **接口，因此**支持随机访问**。Vector底层实现是**数组**。

实现了**Serializable**接口，因此它**支持序列化**。

实现了**Cloneable**接口，能被**克隆**。
		
```java
public class Vector<E> extends AbstractList<E>
implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

## 1.同步

它的实现与 ArrayList 类似，但是**使用了 synchronized 进行同步**,这就是Vector为什么是线程安全的原因。
这个类的线程安全，是建立在几乎**所有修改集合操作**的方法都加了synchronized关键字，例如：

```java
public synchronized boolean add(E e) {
modCount++;
ensureCapacityHelper(elementCount + 1);
elementData[elementCount++] = e;
return true;
}

public synchronized E get(int index) {
if (index >= elementCount)
    throw new ArrayIndexOutOfBoundsException(index);
return elementData(index);
}
```

## 2.重要成员变量

```java
protected Object[] elementData; //Object数组，说明Vector底层实现是数组
protected int elementCount; //实际大小
protected int capacityIncrement; //动态数组的增长系数，扩容时增加大小为capacityIncrement
```

capacityIncrement 是我们可控的一个参数，在构造方法中传入，即每次容量具体增加多少，如果我们在创建 Vector 对象是没有指定 capacityIncrement，则**默认每次把容量增加为原来的二倍**（不越界的情况下）。

## 3.扩容

```java
private void ensureCapacityHelper(int minCapacity) {
if (minCapacity - elementData.length > 0)
grow(minCapacity);
```

通过grow()方法扩容。

```java
private void grow(int minCapacity) {
int oldCapacity = elementData.length;
int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                       capacityIncrement : oldCapacity);
      if (newCapacity - minCapacity < 0)
          newCapacity = minCapacity;
      if (newCapacity - MAX_ARRAY_SIZE > 0)
          newCapacity = hugeCapacity(minCapacity);
     elementData = Arrays.copyOf(elementData, newCapacity);
 }
```

Vector与ArrayList比较明显不同就是grow方法数组容量的扩增算法，oldCapacity + ((capacityIncrement > 0) ?capacityIncrement : oldCapacity)。 

　　如果是通过new Vector()实例化对象，**此时capacityIncrement变量的值就会默认为0，那每次容量就只是扩增一倍（100%）**。 
　　如果是通过Vector(int initialCapacity, int capacityIncrement)实例化Vector，只要你传入的capacityIncrement不小于0，那么数组的容量就能按照你设置的值来扩增。其它代码部分与ArrayList差不多。


## 4.为什么是一个遗弃的类

Vector中对**每一个独立操作都实现了同步**，这通常不是我们想要的做法。对单一操作实现同步通常不是线程安全的（举个例子，比如你想遍历一个Vector实例。你仍然需要申明一个锁来防止其他线程在同一时刻修改这个Vector实例。如果不添加锁的话通常会在遍历实例的这个线程中导致一个ConcurrentModificationException）同时这个操作也是十分慢的(在创建了一个锁就已经足够的前提下，为什么还需要重复的创建锁)当然，即使你不需要同步，Vector也是有锁的资源开销的。

