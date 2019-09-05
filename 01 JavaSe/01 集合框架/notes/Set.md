# 面试题

## 1. 使用set需要注意什么（壹钱包）

- 为Set集合里的元素的实现类**重写equals()和hashCode()方法()**
- 若传入重复的元素,**重复元素会被忽略**(可以用于做集合的去重)

## 2.讲一下HashSet和TreeSet（壹钱包）

相同点： 
**都是唯一不重复的Set集合。**

不同点： 
底层来说，HashSet是用**Hash表来存储数据，而TreeSet是用二叉平衡树来存储数据**。 功能上来说，由于TreeSet是有序的Set，可以使用SortedSet。接口的first()、last()等方法。但由于要排序，势必要影响速度。所以，如果不需要顺序的话，还是使用HashSet吧，使用Hash表存储数据的HashSet在速度上更胜一筹。如果需要顺序则TreeSet更为明智。 
底层来说，HashSet是用Hash表来存储数据，而TreeSet是用二叉平衡树来存储数据。

## 3. Set实现（中兴）

Set的内部是通过Map来实现的。

https://blog.csdn.net/qq_21870555/article/details/82956199

## 4. HashSet去重原理

是通过**调用元素内部的hashCode和equals方法实现去重**，首先调用hashCode方法，比较两个元素的哈希值，如果哈希值不同，直接 
* 认为是两个对象，停止比较。如果哈希值相同，再去调用equals方法，返回true，认为是一个对象。返回false，认为是两个对象 
---------------------
作者：ThreeCQscbRONLY1 
原文：https://blog.csdn.net/ThreeCQscbRONLY1/article/details/81541510 

# Set

Set是一个**存储无序且不重复元素**的集合。
在使用Set集合的时候，应该注意两点

- 为Set集合里的元素的实现类**重写equals()和hashCode()方法()**
- 若传入重复的元素,**重复元素会被忽略**(可以用于做集合的去重)

`扩展`
判断两个元素相等的标准：**两个对象通过equals()方法比较相等，并且两个对象的hashCode()方法返回值也相等。**

# HashSet

HashSet是Set接口的典型实现,是哈希表结构，主要**利用HashMap的key来存储元素，计算插入元素的hashCode来获取元素在集合中的位置,**因此具有很好的存取和查找性能。

　HashSet实现Set接口，由哈希表（实际上是一个HashMap实例）支持。它不保证set 的迭代顺序；特别是它不保证该顺序恒久不变。此类允许使用null元素。HashSet中不允许有重复元素，这是因为HashSet是基于HashMap实现的，HashSet中的元素都存放在HashMap的key上面，而value中的值都是统一的一个private static final Object PRESENT = new Object();。HashSet跟HashMap一样，都是一个存放链表的数组。

　　HashSet中add方法调用的是底层HashMap中的put()方法，而如果是在HashMap中调用put，首先会判断key是否存在，如果key存在则修改value值，如果key不存在这插入这个key-value。而在set中，因为value值没有用，也就不存在修改value值的说法，因此往HashSet中添加元素，首先判断元素（也就是key）是否存在，如果不存在这插入，如果存在着不插入，这样HashSet中就不存在重复值。

主要特点
1.不允许出现重复元素
2.存储的元素是无序的
3.不是同步的，如果多个线程同时访问一个HashSet，则必须通过代码来保证其同步。
4.**集合元素值可以是null。**

## 源码

对于HashSet而言，它是基于HashMap实现的，HashSet底层使用HashMap来保存所有元素，更确切的说，HashSet中的元素，只是存放在了底层HashMap的key上， 而value使用一个static final的Object对象标识。因此HashSet 的实现比较简单，相关HashSet的操作，基本上都是直接调用底层HashMap的相关方法来完成，HashSet的源代码如下：

```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
    static final long serialVersionUID = -5024744406713321676L;

    // 底层使用HashMap来保存HashSet中所有元素。
    private transient HashMap<E,Object> map;
    
    // 定义一个虚拟的Object对象作为HashMap的value，将此对象定义为static final。
    private static final Object PRESENT = new Object();

    /**
     * 默认的无参构造器，构造一个空的HashSet。
      * 
     * 实际底层会初始化一个空的HashMap，并使用默认初始容量为16和加载因子0.75。
      */
    public HashSet() {
        map = new HashMap<E,Object>();
    }

    /**
     * 构造一个包含指定collection中的元素的新set。
      *
     * 实际底层使用默认的加载因子0.75和足以包含指定
     * collection中所有元素的初始容量来创建一个HashMap。
     * @param c 其中的元素将存放在此set中的collection。
     */
    public HashSet(Collection<? extends E> c) {
        map = new HashMap<E,Object>(Math.max((int) (c.size()/.75f) + 1, 16));
        addAll(c);
    }

    /**
     * 以指定的initialCapacity和loadFactor构造一个空的HashSet。
     *
     * 实际底层以相应的参数构造一个空的HashMap。
     * @param initialCapacity 初始容量。
     * @param loadFactor 加载因子。
     */
    public HashSet(int initialCapacity, float loadFactor) {
        map = new HashMap<E,Object>(initialCapacity, loadFactor);
    }

    /**
     * 以指定的initialCapacity构造一个空的HashSet。
     *
     * 实际底层以相应的参数及加载因子loadFactor为0.75构造一个空的HashMap。
     * @param initialCapacity 初始容量。
     */
    public HashSet(int initialCapacity) {
        map = new HashMap<E,Object>(initialCapacity);
    }

    /**
     * 以指定的initialCapacity和loadFactor构造一个新的空链接哈希集合。
     * 此构造函数为包访问权限，不对外公开，实际只是是对LinkedHashSet的支持。
     *
     * 实际底层会以指定的参数构造一个空LinkedHashMap实例来实现。
     * @param initialCapacity 初始容量。
     * @param loadFactor 加载因子。
     * @param dummy 标记。
     */
    HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        map = new LinkedHashMap<E,Object>(initialCapacity, loadFactor);
    }

    /**
     * 返回对此set中元素进行迭代的迭代器。返回元素的顺序并不是特定的。
     * 
     * 底层实际调用底层HashMap的keySet来返回所有的key。
     * 可见HashSet中的元素，只是存放在了底层HashMap的key上，
     * value使用一个static final的Object对象标识。
     * @return 对此set中元素进行迭代的Iterator。
     */
    public Iterator<E> iterator() {
        return map.keySet().iterator();
    }

    /**
     * 返回此set中的元素的数量（set的容量）。
     *
     * 底层实际调用HashMap的size()方法返回Entry的数量，就得到该Set中元素的个数。
     * @return 此set中的元素的数量（set的容量）。
     */
    public int size() {
        return map.size();
    }

    /**
     * 如果此set不包含任何元素，则返回true。
     *
     * 底层实际调用HashMap的isEmpty()判断该HashSet是否为空。
     * @return 如果此set不包含任何元素，则返回true。
     */
    public boolean isEmpty() {
        return map.isEmpty();
    }

    /**
     * 如果此set包含指定元素，则返回true。
     * 更确切地讲，当且仅当此set包含一个满足(o==null ? e==null : o.equals(e))
     * 的e元素时，返回true。
     *
     * 底层实际调用HashMap的containsKey判断是否包含指定key。
     * @param o 在此set中的存在已得到测试的元素。
     * @return 如果此set包含指定元素，则返回true。
     */
    public boolean contains(Object o) {
        return map.containsKey(o);
    }

    /**
     * 如果此set中尚未包含指定元素，则添加指定元素。
     * 更确切地讲，如果此 set 没有包含满足(e==null ? e2==null : e.equals(e2))
     * 的元素e2，则向此set 添加指定的元素e。
     * 如果此set已包含该元素，则该调用不更改set并返回false。
     *
     * 底层实际将将该元素作为key放入HashMap。
     * 由于HashMap的put()方法添加key-value对时，当新放入HashMap的Entry中key
     * 与集合中原有Entry的key相同（hashCode()返回值相等，通过equals比较也返回true），
     * 新添加的Entry的value会将覆盖原来Entry的value，但key不会有任何改变，
     * 因此如果向HashSet中添加一个已经存在的元素时，新添加的集合元素将不会被放入HashMap中，
     * 原来的元素也不会有任何改变，这也就满足了Set中元素不重复的特性。
     * @param e 将添加到此set中的元素。
     * @return 如果此set尚未包含指定元素，则返回true。
     */
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }

    /**
     * 如果指定元素存在于此set中，则将其移除。
     * 更确切地讲，如果此set包含一个满足(o==null ? e==null : o.equals(e))的元素e，
     * 则将其移除。如果此set已包含该元素，则返回true
     * （或者：如果此set因调用而发生更改，则返回true）。（一旦调用返回，则此set不再包含该元素）。
     *
     * 底层实际调用HashMap的remove方法删除指定Entry。
     * @param o 如果存在于此set中则需要将其移除的对象。
     * @return 如果set包含指定元素，则返回true。
     */
    public boolean remove(Object o) {
        return map.remove(o)==PRESENT;
    }

    /**
     * 从此set中移除所有元素。此调用返回后，该set将为空。
     *
     * 底层实际调用HashMap的clear方法清空Entry中所有元素。
     */
    public void clear() {
        map.clear();
    }

    /**
     * 返回此HashSet实例的浅表副本：并没有复制这些元素本身。
     *
     * 底层实际调用HashMap的clone()方法，获取HashMap的浅表副本，并设置到HashSet中。
     */
    public Object clone() {
        try {
            HashSet<E> newSet = (HashSet<E>) super.clone();
            newSet.map = (HashMap<E, Object>) map.clone();
            return newSet;
        } catch (CloneNotSupportedException e) {
            throw new InternalError();
        }
    }
}
```

## 重要方法

```java
//无参构造方法，完成map的创建
public HashSet() {
    map = new HashMap<>();
}
//指定集合转化为HashSet, 完成map的创建
public HashSet(Collection<? extends E> c) {
   map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
   addAll(c);
}
//指定初始化大小，和负载因子
public HashSet(int initialCapacity, float loadFactor) {
    map = new HashMap<>(initialCapacity, loadFactor);
}
//指定初始化大小
public HashSet(int initialCapacity) {
    map = new HashMap<>(initialCapacity);
}
//指定初始化大小和负载因子，dummy 无实际意义
HashSet(int initialCapacity, float loadFactor, boolean dummy) {
    map = new LinkedHashMap<>(initialCapacity, loadFactor);
}
```

通过构造函数，不难发现，HashSet的底层是采用HashMap实现的。

```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
```

PRESENT为HashSet类中定义的一个使用static final 修饰的常量，并无实际的意义，HashSet的add方法调用HashMap的put()方法实现，如果键已经存在，map.put()放回的是旧值，添加失败**，如果添加成功map.put()方法返回的是null ,HashSet.add()方法返回true,要添加的元素可作为map中的key 。**

```java
public boolean remove(Object o) {
    return map.remove(o)==PRESENT;
}
```

删除方法，调用map.remove()方法实现，map.remove()能找到指定的key,则返回key对应的value,对于Hashset而言，它所有的key对应的值都是PRESENT。

## Demo

```java
public class HashSetDemo{
	public static void main(String[] args){
		//创建HashSet集合
        Set<String> hashSet = new HashSet<String>();
        
        //添加元素
        hashSet.add("my");
        hashSet.add("name");
        hashSet.add("is");
        hashSet.add("ken");
        hashSet.add("hello");
        hashSet.add("everyone");
        System.out.println("HashSet容量大小："+hashSet.size());
        
        //迭代器遍历
        
        Iterator<String> it = hashSet.iterator();
        while(it.hasNext()){
            String str = it.next();
            System.out.printIn(str + " ");   
        }
        System.out.print("\n");
        //元素删除
        hashSet.remove("ken");
        System.out.println("HashSet元素大小：" + hashSet.size());
        hashSet.clear();
        System.out.println("HashSet元素大小：" + hashSet.size());
        
        //集合判断
        boolean isEmpty = hashSet.isEmpty();
        System.out.printIn("HashSet是否为空：" + isEmpty);
        boolean isContains = hashSet.contains("hello");
        System.out.println("HashSet是否为空：" + isContains);
	}
}

//out:
HashSet初始容量大小：0
HashSet容量大小：6
ken everyone name is hello my 
HashSet元素大小：5
HashSet元素大小：0
HashSet是否为空：true
HashSet是否为空：false
```

```JAVA
//重写类A的equals方法总是返回true,但没有重写其hashCode()方法。
//不能保证当前对象是HashSet中的唯一对象
class A {
    @Override
    public boolean equals(Object obj) {
        return true;
    }
}

//重写类B的hashCode()方法总是返回1,但没有重写其equals()方法。
//不能保证当前对象是HashSet中的唯一对象
class B {
    @Override
    public int hashCode() {
        return 1;
    }
}

//重写重写类C的hashCode()方法总是返回2,且有重写其equals()方法
class C {
    @Override
    public int hashCode() {
        return 2;
    }

    @Override
    public boolean equals(Object obj) {
        return true;
    }
}

public class HashSetTest {
    public static void main(String[] args) {
        HashSet books = new HashSet();
        //分别向books集合中添加两个A对象，两个B对象，两个C对象
        books.add(new A());
        books.add(new A());

        books.add(new B());
        books.add(new B());

        books.add(new C());
        books.add(new C());
        System.out.println(books);
    }
}
//out
[B@1, B@1, C@2, A@3bc257, A@785d65]
```

可以看出,只有当同时重写了equals方法和hashCode方法时,才能按照自己的意图判断对象是否相等,否则均判定为不相等,可加入Set集合中。

## 总结

（1）HashSet 的实现其实非常简单，它只是**封装了一个 HashMap 对象来存储所有的集合元素**，所有放入 HashSet 中的集合元素实际上由 HashMap 的 key 来保存，而 HashMap 的 value 则存储了一个 PRESENT，它是一个静态的 Object 对象。

（2）对于HashSet中保存的对象，请注意**正确重写其equals和hashCode方法，以保证放入的对象的唯一性。**

- HashSet的底层通过HashMap实现的。而HashMap在1.7之前使用的是数组+链表实现，在1.8+使用的数组+链表+红黑树实现。其实也可以这样理解，**HashSet的底层实现和HashMap使用的是相同的方式，因为Map是无序的，因此HashSet也无法保证顺序。**

- HashSet的方法，也是借助HashMap的方法来实现的。

  https://my.oschina.net/90888/blog/1625854

# TreeSet

与HashSet集合类似，TreeSet也是基于Map来实现，其底层结构为红黑树（特殊的二叉查找树）
与HashSet不同的是，TreeSet具有排序功能，分为**自然排序(123456)和自定义排序两类**，默认是自然排序

具有如下特点：

- 对插入的元素进行排序，是**一个有序的集合**（主要与HashSet的区别）
- 底层使用**红黑树结构**，而不是哈希表结构
- 允许插入Null值
- 不允许插入重复元素
- 线程不安全

## 源码

```java
private transient NavigableMap<E,Object> m;

TreeSet(NavigableMap<E,Object> m) {
    this.m = m;
}

 // 以自然排序方式创建一个新的 TreeMap，
        // 根据该 TreeSet 创建一个 TreeSet，
        // 使用该 TreeMap 的 key 来保存 Set 集合的元素

    public TreeSet() {
        this(new TreeMap<E,Object>());
    }
// 以定制排序方式创建一个新的 TreeMap，
        // 根据该 TreeSet 创建一个 TreeSet，
        // 使用该 TreeMap 的 key 来保存 Set 集合的元素

  public TreeSet(Comparator<? super E> comparator) {
        this(new TreeMap<>(comparator));
    }
public TreeSet(Collection<? extends E> c) {
        // 调用方式一构造器创建一个 TreeSet，底层以 TreeMap 保存集合元素
        this(); 
        // 向 TreeSet 中添加 Collection 集合 c 里的所有元素
        addAll(c); 
    }

```

从上述可以看出，TreeSet的构造函数都是通过新建一个TreeMap作为实际存储Set元素的容器。因此得出结论: TreeSet的底层实际使用的存储容器就是TreeMap。TreeSet 里绝大部分方法都是直接调用 TreeMap 的方法来实现的。 

列举一个，大家也可以自行查看源码。

https://blog.csdn.net/u010176014/article/details/52096398

# 对比：

## TreeSet和TreeMap

相同点： 
TreeMap和TreeSet都是有序的集合。 
TreeMap和TreeSet都是非同步集合，因此他们不能在多线程之间共享，不过可以使用方法Collections.synchroinzedMap()来实现同步。 
运行速度都要比Hash集合慢，他们内部对元素的操作时间复杂度为O(logN)，而HashMap/HashSet则为O(1)。

不同点： 
最主要的区别就是TreeSet和TreeMap分别实现Set和Map接口 
TreeSet**只存储一个对象，而TreeMap存储两个对象Key和Value**（仅仅key对象有序） 
TreeSet**中不能有重复对象，而TreeMap中可以存在。**

## TreeSet和HashSet

相同点： 
**都是唯一不重复的Set集合。**

不同点： 
底层来说，HashSet是用**Hash表来存储数据，而TreeSet是用二叉平衡树来存储数据**。 功能上来说，由于TreeSet是有序的Set，可以使用SortedSet。接口的first()、last()等方法。但由于要排序，势必要影响速度。所以，如果不需要顺序的话，还是使用HashSet吧，使用Hash表存储数据的HashSet在速度上更胜一筹。如果需要顺序则TreeSet更为明智。 
底层来说，HashSet是用Hash表来存储数据，而TreeSet是用二叉平衡树来存储数据。

## 总结

1、不能有重复的元素； 
2、具有排序功能； 
3、TreeSet中的元素必须实现Comparable接口并重写compareTo()方法，TreeSet判断元素是否重复 、以及确定元素的顺序 靠的都是这个方法； 
①对于java类库中定义的类，TreeSet可以直接对其进行存储，如String，Integer等,因为这些类已经实现了Comparable接口); 
②对于自定义类，如果不做适当的处理，TreeSet中只能存储一个该类型的对象实例，否则无法判断是否重复。 
4、依赖TreeMap。 

5、相对HashSet,TreeSet的优势是有序，劣势是相对读取慢。根据不同的场景选择不同的集合。

作者：青年小篆 
原文：https://blog.csdn.net/u010176014/article/details/52096398 


