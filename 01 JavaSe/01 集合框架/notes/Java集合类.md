# 面试题

## 1.Java中Collection和Collections的区别（360企业安全）

java.util.Collection 是一个集合接口（集合类的一个顶级接口）。它提供了对集合对象进行基本操作的通用接口方法。Collection接口在Java 类库中有很多具体的实现。Collection接口的意义是为各种具体的集合提供了最大化的统一操作方式，其**直接继承接口有List与Set。**

java.util.Collections 是一个**包装类（工具类/帮助类**）。它包含有各种有关集合操作的静态多态方法。此类不能实例化，就像一个工具类，用于对集合中元素**进行排序、搜索以及线程安全等各种操作**，服务于Java的Collection框架。

## 2.List为什么是有序的,Set为什么都是无序的

set是基于哈希表实现的，通过哈希值来存取。LinkedListHashSet是有序的。

# 一、概述

ava集合框架提供了数据持有对象的方式，提供了对数据集合的操作。Java 集合框架位于 java.util 包下，主要有三个大类：**Collection(接口)、Map(接口)、集合工具类。**

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/%E9%9B%86%E5%90%88%E6%A1%86%E6%9E%B6.png)

## 1. List

List继承自Collection接口。List是一种**有序集合**，List中的元素可以根据索引（顺序号：元素在集合中处于的位置信息）进行取得/删除/插入操作。跟Set集合不同的是，List**允许有重复元素**。

- ArrayList：基于动态**数组**实现，支持随机访问；**线程不同步**。默认**初始容量为 10**，当数组大小不足时容量**扩大为 1.5 倍**。

- LinkedList：基于**双向循环链表**实现，只能顺序访问，但是可以快速地在链表中间插入和删除元素。不仅如此，LinkedList 还可以用作栈、队列和双端队列。**线程不同步**。

### 1.1使用场景

- 对于需要**快速插入，删除**元素，应该使用LinkedList。

- 对于需要**快速随机访问**元素，应该使用ArrayList。
- 对于“单线程环境” 或者 “多线程环境，但List仅仅只会被**单个线程操作**”，此时应该使用非同步的类(如ArrayList)。

  对于“**多线程环境**，且List可能同时被多个线程操作”，此时，应该使用同步的类(如Vector)。

## 2. Set

- HashSet：基于 **Hash** 实现，支持快速查找，但是失去**有序性**；

- TreeSet：基于**红黑树**实现，保持有序，但是查找效率不如 HashSet；

- LinkedListHashSet：具有 HashSet 的查找效率，且内部使用链表维护元素的插入顺序，因此具有**有序性**。

## 3. Queue

只有两个实现：LinkedList 和 PriorityQueue，其中 LinkedList 支持双向队列，PriorityQueue 是基于堆结构实现。

## 4. Map

- HashMap：**线程不同步**，基于 **Hash** 实现。默认**初始大小为 16**，每次扩大一倍。当发生 Hash 冲突时，**采用拉链法（链表）**。当单个桶中元素个数大于等于8时，链表实现改为红黑树实现；当元素个数小于6时，变回链表实现。由此来防止hashCode攻击。

- LinkedHashMap：使用链表来维护元素的顺序，顺序为插入顺序或者最近最少使用（LRU）顺序

- TreeMap：基于**红黑树**实现

- ConcurrentHashMap：线程安全 Map，不涉及类似于 HashTable 的同步加锁

- WeakHashMap：它的特殊之处在于 WeakHashMap 里的 entry 可能会被 GC 自动删除，即使程序员没有调用 remove() 或者 clear() 方法。

## 5. Java 1.0/1.1 容器

对于旧的容器，我们决不应该使用它们，只需要对它们进行了解。

- Vector：和 ArrayList 类似，但它是线程安全的,默认初始容量为 10，当数组大小不足时容量扩大为 2 倍。它的同步是通过 Iterator 方法加 synchronized 实现的。

- HashTable：和 HashMap 类似，但它是**线程安全的**。HashTable **不能存储 NULL 的 key 和 value**。


# 容器中的设计模式

## 1. 迭代器模式

从概览图可以看到，每个集合类都有一个 Iterator 对象，可以**通过这个迭代器对象来遍历集合中的元素**。

## 2. 适配器模式

java.util.Arrays#asList() 可以把数组类型转换为 List 类型。

```java
 	List list = Arrays.asList(1, 2, 3);
 	int[] arr = {1, 2, 3};
 	list = Arrays.asList(arr);
```



# 深入Java源码解析容器类List、Set、Map

## 1 常用容器继承关系图

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/20190524172615.png)


## 2 Collection和Map

Collection代表的是**单个元素对象的序列**，（可以有序/无序，可重复/不可重复 等，具体依据具体的子接口Set，List，Queue等）；Map代表的是**“键值对”对象的集合**（同样可以有序/无序 等依据具体实现）

### 2.1 Collection

接口定义：

```java
public interface Collection<E> extends Iterable<E> {

...

}
```

泛型<E>即该Collection中元素对象的类型，继承的Iterable是定义的**一个遍历操作接口，采用hasNext next的方式进行遍历**。具体实现还是放在具体类中去实现。

我们可以看下定义的几个重要的接口方法

```java
add(E e) 
```
 	Ensures that this collection contains the specified element

```java
clear()
```
 	Removes all of the elements from this collection (optional operation).

```java
contains(Object o)
```
 	Returns true if this collection contains the specified element.

```java
isEmpty()
```
 	Returns true if this collection contains no elements.

```java
iterator()
```
 	Returns an iterator over the elements in this collection.

```java
remove(Object o)
```
 	Removes a single instance of the specified element from this collection, if it is present (optional operation).

```java
retainAll(Collection<?> c)
```
 	Retains only the elements in this collection that are contained in the specified collection (optional operation).（**ps:这个平时倒是没注意，感觉挺好用的接口，保留指定的集合**）

```java
size()
```
 	Returns the number of elements in this collection.

```java
toArray()
```
 	Returns an array containing all of the elements in this collection.

```java
toArray(T[] a)
```
 	Returns an array containing all of the elements in this collection; the runtime type of the returned array is that of the specified array.（**ps:这个接口也可以mark下**）

 	...

### 2.2 Map

一个保存键值映射的对象。 映射Map中不能包含重复的key，每一个key最多对应一个value。

Map集合提供3种遍历访问方法，1.获得所有key的集合然后通过key访问value。2.获得value的集合。3.获得key-value键值对的集合（key-value键值对其实是一个对象，里面分别有key和value）。 Map的访问顺序取决于Map的遍历访问方法的遍历顺序。 有的Map，比如TreeMap可以保证访问顺序，但是有的比如HashMap，无法保证访问顺序。

接口定义如下：

```java
public interface Map<K,V> {

...

interface Entry<K,V> {
    K getKey();
    V getValue();
    ...
} 
}
```

## 3.List、Set和Queue

1. List是存储的元素容器是有个**有序**的可以索引到元素的容器，并且里面的元素可以重复。

1. Set里面和List最大的区别是Set里面的元素对象不可重复。
