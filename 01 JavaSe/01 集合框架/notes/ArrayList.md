# 面试题

## 1.[向数组的中间插入数据，ArrayList和LinkedList哪个快](https://www.cnblogs.com/liqiu/archive/2013/05/07/3065203.html)（涂鸦智能）

ArrayList

插入位置越往中间，LinkedList效率越低，**因为它遍历获取插入位置是从两头往中间搜，index越往中间遍历越久**，因此ArrayList的插入效率可能会比LinkedList高。

插入位置的选取对LinkedList有很大的影响，因为LinkedList在插入时需要向移动指针到指定节点， 才能开始插入,，一旦要插入的位置比较远，LinkedList就需要一步一步的移动指针， 直到移动到插入位置，这就解释了， 为什么节点值越大， 时间越长， 因为指针移动需要时间。

而ArrayList是数据结构， **可以根据下标直接获得位置， 这就省去了查找特定节点的时间**，所以对ArrayList的影响不是特别大。

Demo  

```java
public class ArrayList_LinkedList_compare_demo {
    public static void main(String[] args){
        arrayAndLink();
    }
    public static void  arrayAndLink(){
        List<String> arrayList = new ArrayList<String>();
        List<String> linkedList = new LinkedList<String>();
        //生产数据
            for(int i = 0; i < 10000; i++){
                arrayList.add("arrayList" + i);
                linkedList.add("linkedList" + i);
            }
        //标记开始时间
        long startTime1 = System.currentTimeMillis();
        //在list中间插入数据
        for(int i = 0; i < 10000; i++){
            arrayList.add((5000 + i), "arrayList");

        }
        //标记结束时间
        long endTime1 = System.currentTimeMillis();
        System.out.println("arrayList time ---- " + (endTime1 - startTime1));

        //标记开始时间
        long startTime2 = System.currentTimeMillis();
        //在link中间插入数据
        for(int i = 0; i < 10000; i ++){
            linkedList.add((5000+i), "linkedList");
        }
        //标记结束时间
        long endTime2 = System.currentTimeMillis();
        System.out.println("linkedList time -----"+(endTime2 - startTime2));

    }
}
```

插入中间结果

```java
arrayList time ---- 27
linkedList time -----687
```

ArrayList明显快于LinkedList

插入开头结果

```java
arrayList time ---- 27
linkedList time -----141
```

ArrayList依然快于LinkedList

## 2.ArrayList怎么扩容（中兴）

添加元素时使用 ensureCapacityInternal() 方法来保证容量足够，如果不够时，需要使用 grow() 方法进行扩容，新容量的大小为 oldCapacity + (oldCapacity >> 1)，oldCapacity >> 1 为位运算的右移操作，右移一位相当于除以2，也就是**旧容量的 1.5 倍**。

获取newCapacity后再对newCapacity的大小进行判断，**如果仍然小于minCapacity，则直接让newCapacity 等于minCapacity**，而不再计算1.5倍的扩容。**实际扩容可能大于1.5倍**。

**①、当通过 ArrayList() 构造一个空集合，初始长度是为0的，第 1 次添加元素，会创建一个长度为10的数组，并将该元素赋值到数组的第一个位置。**

　　**②、第 2 次添加元素，集合不为空，而且由于集合的长度size+1是小于数组的长度10，所以直接添加元素到数组的第二个位置，不用扩容。**

　　**③、第 11 次添加元素，此时 size+1 = 11，而数组长度是10，这时候创建一个长度为10+10\*0.5 = 15 的数组（扩容1.5倍），然后将原数组元素引用拷贝到新数组。并将第 11 次添加的元素赋值到新数组下标为10的位置。**

　　**④、第 Integer.MAX_VALUE - 8 = 2147483639，然后 2147483639%1.5=1431655759（这个数是要进行扩容） 次添加元素，为了防止溢出，此时会直接创建一个 1431655759+1 大小的数组，这样一直，每次添加一个元素，都只扩大一个范围。**

　　**⑤、第 Integer.MAX_VALUE - 7 次添加元素时，创建一个大小为 Integer.MAX_VALUE 的数组，在进行元素添加。**

　　**⑥、第 Integer.MAX_VALUE + 1 次添加元素时，抛出 OutOfMemoryError 异常。**

　　**注意：能向集合中添加 null 的，因为数组可以有 null 值存在。**

https://blog.csdn.net/zymx14/article/details/78324464

# ArrayList

## 1.概览

**ArrayList 是一个用数组实现的集合，支持随机访问，元素有序且可以重复。**

实现了**List接口，继承了AbstractList**，底层是数组实现的，一般我们把它认为是可以自增扩容的数组。

实现了 **RandomAccess **接口，因此**支持随机访问**，这是理所当然的，因为 ArrayList 是基于数组实现的。

实现了**Serializable**接口，因此它**支持序列化**。

实现了**Cloneable**接口，能被**克隆**。

```java
public class ArrayList<E> extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

数组的**默认大小为 10**。

```java
private static final int DEFAULT_CAPACITY = 10;
```

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/ArrayList%E5%BA%95%E5%B1%82%E5%AE%9E%E7%8E%B0.png)

继承关系

```java
java.lang.Object
   	↳     java.util.AbstractCollection<E>
         ↳     java.util.AbstractList<E>
               ↳     java.util.ArrayList<E>
```

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/Arrylist%E4%B8%8ECollection%E5%85%B3%E7%B3%BB.jpg)
## 2.序列化

基于数组实现，保存元素的数组使用 **transient 修饰**，该关键字声明数组默认**不会被序列化**。ArrayList 具有动态扩容特性，因此保存元素的数组不一定都会被使用，那么就没必要全部进行序列化。ArrayList 重写了 writeObject() 和 readObject() 来控制只序列化数组中有元素填充那部分内容。

## 3.扩容

添加元素时使用 ensureCapacityInternal() 方法来保证容量足够，如果不够时，需要使用 grow() 方法进行扩容，新容量的大小为 oldCapacity + (oldCapacity >> 1)，oldCapacity >> 1 为位运算的右移操作，右移一位相当于除以2，也就是**旧容量的 1.5 倍**。

获取newCapacity后再对newCapacity的大小进行判断，**如果仍然小于minCapacity，则直接让newCapacity 等于minCapacity**，而不再计算1.5倍的扩容。**实际扩容可能大于1.5倍**。

扩容操作需要调用 Arrays.copyOf() 把**原数组整个复制到新数组**中，这个**操作代价很高**，因此最好在创建 ArrayList 对象时就指定大概的容量大小，减少扩容操作的次数。

```java
ArrayList构造方法初始化一个空数组

private static final int DEFAULT_CAPACITY = 10;
	transient Object[] elementData; // non-private to simplify nested class access
	private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
	public ArrayList() {
   	this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
	}	

调用add方法插入数据

// JDK 1.8
	private int size;
    public boolean add(E e) {
	ensureCapacityInternal(size + 1);  // Increments modCount!!
	elementData[size++] = e;
	return true;
	}
```

size变量是用来记录数组的元素总数。当使用add方法的时候**首先调用ensureCapacityInternal方法，传入size+1进去**，检查是否需要扩充elementData数组的大小。检查完毕之后再将e赋值给elementData数组 ，size再自增1。ensureCapacityInternal源码如下。
    
```java
// 判断数组是否越界
private void ensureCapacityInternal(int minCapacity) {
if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
    minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
}
ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
modCount++;
// overflow-conscious code
if (minCapacity - elementData.length > 0)
    grow(minCapacity);
}
```

如果elementData数组为空，则在DEFAULT_CAPACITY和minCapacity（刚才的size+1）选取最大值，赋值给minCapacity。接着进入ensureExplicitCapacity方法，**如果需要的最小容量（minCapacity）比当前数组长度还要大，则进入grow方法扩增数组。**

```java
// 扩容
private void grow(int minCapacity) {
// overflow-conscious code
int oldCapacity = elementData.length;
int newCapacity = oldCapacity + (oldCapacity >> 1); // 1.5倍
if (newCapacity - minCapacity < 0)
    newCapacity = minCapacity;
if (newCapacity - MAX_ARRAY_SIZE > 0)
    newCapacity = hugeCapacity(minCapacity);
// minCapacity is usually close to size, so this is a win:
elementData = Arrays.copyOf(elementData, newCapacity);
}
```

接下来重点讲讲这个grow方法，先通过elementData.length获取当前数组的容量大小赋值给oldCapacity。接着oldCapacity + (oldCapacity >> 1)，其中(oldCapacity >> 1)表示oldCapacity右移1位，它的值可以理解为oldCapacity除以2（至于为什么请百度二进制、移位操作等关键字）。总结起来就是在oldCapacity基础上扩增50%容量再赋值给newCapacity。最后使用Arrays工具类扩增elementData数组。

```java
public static <T> T[] copyOf(T[] original, int newLength) {
    return (T[]) copyOf(original, newLength, original.getClass());
}
public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
    @SuppressWarnings("unchecked")
    T[] copy = ((Object)newType == (Object)Object[].class)
        ? (T[]) new Object[newLength]
        : (T[]) Array.newInstance(newType.getComponentType(), newLength);
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));
    return copy;
}
```

整个核心就是System.arraycopy方法，**将original数组从的所有数据复制到copy数组中，返回给调用方法**。整个add方法添加数据流程就走完了。

```java
private static int hugeCapacity(int minCapacity) {
if (minCapacity < 0) // overflow
    throw new OutOfMemoryError();
return (minCapacity > MAX_ARRAY_SIZE) ? Integer.MAX_VALUE : MAX_ARRAY_SIZE;
}
```

### 总结

对于 ArrayList 集合添加元素，我们总结一下：

　　**①、当通过 ArrayList() 构造一个空集合，初始长度是为0的，第 1 次添加元素，会创建一个长度为10的数组，并将该元素赋值到数组的第一个位置。**

　　**②、第 2 次添加元素，集合不为空，而且由于集合的长度size+1是小于数组的长度10，所以直接添加元素到数组的第二个位置，不用扩容。**

　　**③、第 11 次添加元素，此时 size+1 = 11，而数组长度是10，这时候创建一个长度为10+10\*0.5 = 15 的数组（扩容1.5倍），然后将原数组元素引用拷贝到新数组。并将第 11 次添加的元素赋值到新数组下标为10的位置。**

　　**④、第 Integer.MAX_VALUE - 8 = 2147483639，然后 2147483639%1.5=1431655759（这个数是要进行扩容） 次添加元素，为了防止溢出，此时会直接创建一个 1431655759+1 大小的数组，这样一直，每次添加一个元素，都只扩大一个范围。**

　　**⑤、第 Integer.MAX_VALUE - 7 次添加元素时，创建一个大小为 Integer.MAX_VALUE 的数组，在进行元素添加。**

　　**⑥、第 Integer.MAX_VALUE + 1 次添加元素时，抛出 OutOfMemoryError 异常。**

　　**注意：能向集合中添加 null 的，因为数组可以有 null 值存在。**

如果是指定位置插入数据是怎样的呢？

```java
public void add(int index, E element) {
    if (index > size || index < 0)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    ensureCapacityInternal(size + 1); // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}
```



index下标表示插入位置，**先检查下标是否越界，如果是则抛出异常。**之后调用ensureCapacityInternal方法判断是否需要扩增数组，这个方法上面已经分析过，略。最重要的是System.arraycopy方法，为了空出index的位置，将elementData数组从index开始到（size - index）位置，都后移1位。此时数组的index位置已经空出来了，最后再将新的元素放进去，完成数据插入操作。

## 4.删除元素

需要调用 System.arraycopy() 将 index+1 后面的元素都复制到 index 位置上。

```java
public E remove(int index) {
rangeCheck(index);
modCount++;
E oldValue = elementData(index);
int numMoved = size - index - 1;
if (numMoved > 0)
    System.arraycopy(elementData, index+1, elementData, index, numMoved);
elementData[--size] = null; // clear to let GC do its work
return oldValue;
}
```

## 5.查找元素

```java
public E get(int index) {
    if (index >= size)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    return (E) elementData[index];
}
```

这是一个很简单的方法，相信很多人都看得懂，但是仔细想深一层为什么数组通过一个下标就能够查找出元素？**我们在实例化ArrayList的时候，elementData已经完成了初始化。**此时JVM虚拟机中，Java堆中则为elementData数组对象开辟一片连续的内存空间，**虚拟机栈则存储了elementData数组的引用声明，并且引用指向了Java堆的数组首地址。**因此在内存中可以通过：首地址+（元素存储单元×数组下标）=元素地址，快速的寻找对应下标的元素值。

## 6.Fail-Fast

fail-fast 机制，即快速失败机制，是java集合(Collection)中的一种**错误检测机制**。当在迭代集合的过程中该集合在结构上发生改变的时候，就有可能会发生fail-fast，即**抛出ConcurrentModificationException异常**。fail-fast机制并不保证在不同步的修改下一定会抛出异常，它只是尽最大努力去抛出，所以这种机制一般仅用于检测bug。

## 7.SubList

```java
   public List<E> subList(int fromIndex, int toIndex) {
        subListRangeCheck(fromIndex, toIndex, size);
        return new SubList(this, 0, fromIndex, toIndex);
    }
```

作用是返回从 fromIndex(包括) 开始的下标，到 toIndex(不包括) 结束的下标之间的元素**视图**。如下：

```java
ArrayList<String> list = new ArrayList<>();
 list.add("a");
 list.add("b");
 list.add("c");
 
 List<String> subList = list.subList(0, 1);
 for(String str : subList){
     System.out.print(str + " ");//a
 }
```

**返回的是原集合的视图，也就是说，如果对 subList 出来的集合进行修改或新增操作，那么原始集合也会发生同样的操作。**

## 8.Demo

```java
ArrayList<String> list = new ArrayList();
for(int i = 1; i <= 11; i++){
    list.add("value" + i);
}
Integer length = getCapacity(list);
int size = list.size();
System.out.printIn("容量： " + length);
System.out.printIn("大小 " + size);
```

