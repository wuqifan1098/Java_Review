# LinkedList

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/LinkedList.png)

## 1. 概览

LinkedList 底层是基于**双向链表**实现的。**顺序访问**会非常高效，而**随机访问效率比较低**。

LinkedList 是一个**继承于AbstractSequentialList**的双向链表。它也可以被当作堆栈、队列或双端队列进行操作。

LinkedList **实现 List 接口**，能对它进行**队列**操作。

LinkedList **实现 Deque 接口**，即能将LinkedList当作**双端队列**使用。

LinkedList 实现了Cloneable接口，即覆盖了函数clone()，能克隆。

LinkedList 实现java.io.Serializable接口，这意味着LinkedList支持序列化，能通过序列化去传输。

LinkedList 是非同步的。

```java
public class LinkedList<E>
 extends AbstractSequentialList<E>
 implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```

类图关系

```java
java.lang.Object
↳     java.util.AbstractCollection<E>
     ↳     java.util.AbstractList<E>
           ↳     java.util.AbstractSequentialList<E>
                 ↳     java.util.LinkedList<E>
```

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/LinkedList%E7%B1%BB%E5%9B%BE%E5%85%B3%E7%B3%BB.png)

构造函数

```java
// 默认构造函数
transient Node<E> first;
transient Node<E> last;
public LinkedList() { 
}
```


```java
// 创建一个LinkedList，保护Collection中的全部元素。
LinkedList(Collection<? extends E> collection)
```

重要成员

header 和 size。
　　
header是双向链表的表头，它是双向链表节点所对应的类Entry的实例。Entry中包含成员变量： previous, next, element。
其中，**previous是该节点的上一个节点，next是该节点的下一个节点，element是该节点所包含的值。 **
　　
size是双向链表中节点的个数。

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

明显看出这是一个双向链表节点，**item是用来存放节点值，next是尾指针指向下一个节点，prev是头指针指向前一个节点。**

## 2.方法剖析

### add()

add()方法有两个版本，一个是add(E e)，该方法在LinkedList的**末尾插入元素**，因为有last指向链表末尾，在末尾插入元素的花费是常数时间。只需要简单修改几个相关引用即可；另一个是add(int index, E element)，该方法是**在指定下表处插入元素**，需要先通过线性查找找到具体位置，然后修改相关引用完成插入操作。

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/LinkedList_add(E%20e).png)


```java
//add(E e)
public boolean add(E e) {
final Node<E> l = last;
final Node<E> newNode = new Node<>(l, e, null);
last = newNode;
if (l == null)
    first = newNode;//原来链表为空，这是插入的第一个元素
else
    l.next = newNode;
size++;
return true;
}
```

add(int index, E element)的逻辑稍显复杂，可以分成两部分，**1.先根据index找到要插入的位置；2.修改引用，完成插入操作。**

```java
//add(int index, E element)
public void add(int index, E element) {
checkPositionIndex(index);//index >= 0 && index <= size;
if (index == size)//插入位置是末尾，包括列表为空的情况
    add(element);
else{
	Node<E> succ = node(index);//1.先根据index找到要插入的位置
    //2.修改引用，完成插入操作。
    final Node<E> pred = succ.prev;
    final Node<E> newNode = new Node<>(pred, e, succ);
    succ.prev = newNode;
    if (pred == null)//插入位置为0
        first = newNode;
    else
        pred.next = newNode;
    size++;
}
}
```

根据下标插入数据

```java
public void add(int index, E element) {
      checkPositionIndex(index);
      if (index == size)
          linkLast(element);
      else
          linkBefore(element, node(index));
  }
```

先是使用checkPositionIndex检查index下标是否越界。有的话则抛出异常。没有继续往下走，如果插入下标恰好位于数组的末尾，直接通过linkLast方法将节点插入到链表末尾。如果不是，**先用node方法寻找链表中index位置的节点**。再通**过linkBefore方法将节点插入到链表中**。咋们先看看node方法的源码

```java
Node<E> node(int index) {
    // assert isElementIndex(index);

    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

这个node方法第一眼看上去有点懵逼，细看之后知道了大概思路。(size >> 1)这个前面就说过，简单来说就是**size除以2的值。接着判断index下标是在整个链表前半段还是后半段。 **
　　如果是前半段，x指向链表的头指针first，从头部遍历循环到index的位置，返回index的节点。 
　　如果是后半段，x指向链表的尾指针last， 从尾部遍历循环到index的位置，返回index的节点。

```java
void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    final Node<E> pred = succ.prev;
    final Node<E> newNode = new Node<>(pred, e, succ);
    succ.prev = newNode;
    if (pred == null)
        first = newNode;
    else
        pred.next = newNode;
    size++;
    modCount++;
}
```

如果succ是链表的头结点（第一个节点），pred=null（succ是头结点，没有前驱节点）,然后通过new Node<>(pred, e, succ)创建一个新节点（这个新节点的头指针指向pred，刚才说过pred=null，尾指针指向succ）。接着succ的头指针指向新节点newNode。最后由于pred为空null，直接让first指针重新指向newNode，此时newNode变成了头结点。

如果succ是链表的非头结点，pred指针指向succ的前驱节点。然后通过new Node<>(pred, e, succ)创建一个新节点（这个新节点的头指针指向pred，尾指针指向succ）。接着让succ节点的头指针指向新节点newNode。最后由于pred不为空，让pred的尾指针指向newNode。newNode就和其它节点完成双向关联，形成一个双向链表。

### remove()

remove()方法也有两个版本，一个是删除跟**指定元素相等的**第一个元素remove(Object o)，另一个是删除**指定下标处的元素**remove(int index)。

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/LinkedList_remove.png)

两个删除操作都要1.先找到要删除元素的引用，2.修改相关引用，完成删除操作。在寻找被删元素引用的时候remove(Object o)调用的是元素的equals方法，而remove(int index)使用的是下标计数，两种方式都是**线性时间复杂度**。在步骤2中，两个revome()方法都是通过unlink(Node<E> x)方法完成的。这里需要考虑删除元素是第一个或者最后一个时的边界情况。

根据下标移除数据

```java
 public E remove(int index) {
      checkElementIndex(index);
      return unlink(node(index));
}
E unlink(Node<E> x) {
      // assert x != null;
      final E element = x.item;
      final Node<E> next = x.next;
      final Node<E> prev = x.prev;
      if (prev == null) {
          first = next;
      } else {
          prev.next = next;
          x.prev = null;
      }
      if (next == null) {
          last = prev;
      } else {
          next.prev = prev;
          x.next = null;
      }
      x.item = null;
      size--;
      modCount++;
      return element;
  }
```

remove方法中根据下标通过node方法找到对应的node节点，进入unlink方法进行删除操作。Node x表示将要删除的节点，next指针指向x的后继节点。prev指针指向x的前驱节点。接下来要分三种情况来删除节点。 

第一种情况，x是头结点。判断prev指针为空，first指向next(也就是第二个节点)。此时next不为空，next节点的prev指针指向prev（prev是一个空值）。最后x节点尾指针切断与后节点的关联（x.next=null）。

第二种情况，x是尾结点。判断prev指针不为空，prev节点的next指针指向next（next是一个空值），x节点prev指针切断与前节点的关联（x.prev=null）。此时next为空，让last指针重新指向prev（也就是倒数第二个节点）。

第三种情况，x是中间节点。判读那prev不为空，prev的尾指针next指向next后继节点，x切断与前驱节点的关联（x.prev=null）。判断next不为空，next的头指针prev指向prev前驱节点，x切断与后继节点的关联（x.next = null）。


### get()

```java
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}
```

一个简单的方法，检查下标是否越界，接着进入node方法（node的源码上面已经分析过）遍历链表，返回找到的节点的值。

## 3.遍历方法

LinkedList可以通过**迭代器、foreach语句、for循环语句**等方法遍历集合的所有元素。