# 面试题

## 1.final修饰的String不可继承，final修饰的StringBuilder还可以append吗 （360企业安全）

final修饰的引用地址不可变，引用所指向的对象的内容是可以发生变化的。

**StringBuilder.append方法效率比String + String 高。因为String + String会创建一个新的对象，而创建新的对象时，有可能会触发GC。**

StringBuilder直到最后sb.toString()才会创建String对象，之前都没有创建新对象，但是如果你append的**总长度超过一定范围——默认是16**——就会创建一个新的数组，来装下更多的String。

StringBuilder和StringBuffer，**字符串是存放在char[]中的，char[]是存放在堆中的。**

相比String每次+都重新创建一个String对象，重新开辟一段内存不同，StringBuilder和StringBuffer的append都是直接把String对象中的char[]的字符直接拷贝到StringBuilder和StringBuffer的char[]上，效率比String的+高得多。

## 2.String类为什么要设计成final？是否真的不可变？（头条）

1. 不可变性**支持线程安全**。
1. 不可变性**支持字符串常量池，提升性能**。
1. String字符串作为最常用数据类型之一，不可变**防止了随意修改，保证了数据的安全性**。
1. 可以**缓存hash值**

从上文可知String的成员变量是private final 的，也就是初始化之后不可改变。那么在这几个成员中， value比较特殊，因为他是一个引用变量，而不是真正的对象。value是final修饰的，也就是说final不能再指向其他数组对象，那么我能改变value指向的数组吗？ 比如将数组中的某个位置上的字符变为下划线“_”。 至少在我们自己写的普通代码中不能够做到，因为我们根本不能够访问到这个value引用，更不能通过这个引用去修改数组。
那么用什么方式可以访问私有成员呢？ 没错，用反射， 可以反射出String对象中的value属性， 进而改变通过获得的value引用改变数组的结构
Java中**String是不可变的，但是可以通过反射修改其内容**。

## 2.定义和特点

String：

（1）Java 中 String底层是**采用char数组实现的**，存放字符的数组被声明为 final 的 ，因此是不可变的。

（2）String的值是不可变的（ immutable），导致每次对String的操作都会生成新的String对象。

（3）String是**字符串常量，因此是线程安全的。**

StringBuffer：

（1） StringBuffer继承自AbstractStringBuilder类，在AbstractStringBuilder中也是使用字符数组保存字符串。

（2）解决了String拼接过程中产生很多中间对象的问题，StringBuffer类的对象能够被多次修改。

（3）StringBuffer 是**基于Synchronized实现线程安全的可变字符序列，但带来一定的开销。**

StringBuilder：

（1）StringBuilder 类在 JDK 1.5 中被提出，同样继承自AbstractStringBuilder类，使用字符数组保存字符串。

（2）StringBuilder 是一个可变的字符序列，提供一个与 StringBuffer 兼容的 API。

（3）StringBuilder**虽然在效率上较StringBuffer更高，但是不保证线程安全。**

## 3.使用策略

（1）如果要操作**少量的数据，用String** ；单线程操作**大量数据，用StringBuilder **；多线程操作大量数据，用StringBuffer。

（2）不要使用String类的"+"来进行频繁的拼接，**都会生成一个新的 String 对象，**然后将指针指向新的 String 对象，当内存中无引用对象多了以后， JVM 的 GC 就会开始工作，性能就会降低。

（3）在构造 StringBuffer 或 StringBuilder 时应尽可能按照需求指定它们的容量，相较于不指定容量（默认为16），有利于提升性能。

（4）如果不是多线程环境，或者不考虑线程安全问题，StringBuilder 相比使用 StringBuffer 能获得 10%~15% 左右的性能提升。

### StringBuilder()源码

```java
public final class StringBuilder
extends AbstractStringBuilder
implements java.io.Serializable, CharSequence
{
 
static final long serialVersionUID = 4383685877147921099L;
 
public StringBuilder() {
    super(16);
}
 
public StringBuilder(int capacity) {
    super(capacity);
}
}
```

父类AbstractStringBuilder 中的AbstractStringBuilder(int capacity)构造方法

```java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
 
char[] value;
 
int count;
 
AbstractStringBuilder() {
}
 
AbstractStringBuilder(int capacity) {
    value = new char[capacity];
}
}
```

StringBuilder的内部有一个char[]， 在调用StringBuilder的无参构造方法时其内部char[]的默认长度是16。当我们调用StringBuilder的append方法时，其实就是不断的往char[]里填东西的过程。

```java
public StringBuilder append(String str) {
super.append(str);
return this;
```

其中，super.append是调用AbstractStringBuilder 的append(String str)方法，如下：

```java
public AbstractStringBuilder append(String str) {
if (str == null) str = "null";
int len = str.length();
ensureCapacityInternal(count + len);
str.getChars(0, len, value, count);
count += len;
return this;
}
```

StringBuilder默认长度是16，然后，如果要append第17个字符，怎么办？ 
答案是**采用 Arrays.copyOf()成倍复制扩容！**。和ArrayList类似。 

# String

## 1.源码

```java
public final class String
implements java.io.Serializable, Comparable<String>, CharSequence
{
/** The value is used for character storage. */
private final char value[];
 
/** The offset is the first index of the storage that is used. */
private final int offset;
 
/** The count is the number of characters in the String. */
private final int count;
 
/** Cache the hash code for the string */
private int hash; // Default to 0
 
/** use serialVersionUID from JDK 1.0.2 for interoperability */
private static final long serialVersionUID = -6849794470754667710L;
 
......
 
}
```

从上面可以看出几点：

1. String类是final类，也即意味着**String类不能被继承，并且它的成员方法都默认为final方法**。

2. 在Java中，被final修饰的类是不允许被继承的，并且该类中的成员方法都默认为final方法。在早期的JVM实现版本中，被final修饰的方法会被转为内嵌调用以提升执行效率。而从Java SE5/6开始，就渐渐摈弃这种方式了。因此在现在的Java SE版本中，不需要考虑用final去提升方法调用效率。只有在确定不想让该方法被覆盖时，才将方法设置为final。

1. 上面列举出了String类中所有的成员属性，从上面可以看出String类其实是通过**char数组来保存字符串**的。

## 2.实现方法

```java
public String substring(int beginIndex, int endIndex) {
if (beginIndex < 0) {
    throw new StringIndexOutOfBoundsException(beginIndex);
}
if (endIndex > count) {
    throw new StringIndexOutOfBoundsException(endIndex);
}
if (beginIndex > endIndex) {
    throw new StringIndexOutOfBoundsException(endIndex - beginIndex);
}
return ((beginIndex == 0) && (endIndex == count)) ? this :
    new String(offset + beginIndex, endIndex - beginIndex, value);
}
 
public String concat(String str) {
int otherLen = str.length();
if (otherLen == 0) {
    return this;
}
char buf[] = new char[count + otherLen];
getChars(0, count, buf, 0);
str.getChars(0, otherLen, buf, count);
return new String(0, count + otherLen, buf);
}
 
public String replace(char oldChar, char newChar) {
if (oldChar != newChar) {
    int len = count;
    int i = -1;
    char[] val = value; /* avoid getfield opcode */
    int off = offset;   /* avoid getfield opcode */
 
    while (++i < len) {
    if (val[off + i] == oldChar) {
        break;
    }
    }
    if (i < len) {
    char buf[] = new char[len];
    for (int j = 0 ; j < i ; j++) {
        buf[j] = val[off+j];
    }
    while (i < len) {
        char c = val[off + i];
        buf[i] = (c == oldChar) ? newChar : c;
        i++;
    }
    return new String(0, len, buf);
    }
}
return this;
```

无论是sub操、concat还是replace操作都**不是在原有的字符串上进行的，而是重新生成了一个新的字符串对象**。也就是说进行这些操作后，最原始的字符串并没有被改变。

**“对String对象的任何改变都不影响到原对象，相关的任何change操作都会生成新的对象”。**

# StringBuffer

StringBuffer与StringBuilder都是继承于AbstractStringBuilder，唯一的区别就是**StringBuffer的函数上都有synchronized关键字。**

## 源码

```java
public final class StringBuffer
extends AbstractStringBuilder
implements java.io.Serializable, CharSequence {
 
/** use serialVersionUID from JDK 1.0.2 for interoperability */
static final long serialVersionUID = 3388685877147921107L;
 
public StringBuffer() {
    super(16);
}
 
public StringBuffer(int capacity) {
    super(capacity);
}
 
public StringBuffer(String str) {
    super(str.length() + 16);
    append(str);
}
 
public StringBuffer(CharSequence seq) {
    this(seq.length() + 16);
    append(seq);
}
 
public synchronized int length() {
    return count;
}
 
public synchronized int capacity() {
    return value.length;
}
 
public synchronized void ensureCapacity(int minimumCapacity) {
    if (minimumCapacity > value.length) {
        expandCapacity(minimumCapacity);
    }
}
 
public synchronized void setLength(int newLength) {
    super.setLength(newLength);
}
 
public synchronized StringBuffer append(Object obj) {
    super.append(String.valueOf(obj));
    return this;
}
 
public synchronized StringBuffer append(String str) {
    super.append(str);
    return this;
}
......
}
```

# 深入理解String、StringBuffer、StringBuilder

## 1.既然在Java中已经存在了String类，那为什么还需要StringBuilder和StringBuffer类呢？

因为string使用final char[] value数组存储字符串内容，每次修改是return new String**返回一个新的字符串**，需要重新生成一个字符串对象，申请内存空间，这花了时间，而stringbuilder类的value数组**不是final的，是可变的，不需要重新生成新的对象**。

StringBuilder和StringBuffer类拥有的成员属性以及成员方法基本相同，区别是StringBuffer类的成员方法前面多了一个关键字：synchronized，不用多说，这个关键字是在**多线程访问时起到安全保护作用的,也就是说StringBuffer是线程安全的**。

## 2.区别小结

**是否可变**

String 不可变，StringBuffer 和 StringBuilder 可变。

**是否线程安全**

String 不可变，因此是**线程安全**的。

StringBuilder **不是线程安全**的；StringBuffer 是线程安全的，使用 synchronized 来同步。

## 3.使用场景

String：适用于少量的字符串操作的情况

StringBuilder：适用于**单线程下**在字符缓冲区进行大量操作的情况

StringBuffer：适用**多线程下**在字符缓冲区进行大量操作的情况

# String不变性的理解

- String 类是被 final 进行修饰的，不能被继承
- 在用 + 号链接字符串的时候会创建新的字符串
- String s = new String("Hello world"); **可能创建两个对象也可能创建一个对象**。如果静态区中有 “Hello world” 字符串常量对象的话，则仅仅在堆中创建一个对象。如果静态区中没有 “Hello world” 对象，则堆上和静态区中都需要创建对象。
- 在 Java 中, 通过**使用 "+" 符号来串联字符串的时候**,，实际上底层会**转成通过 StringBuilder 实例的 append() 方法**来实现。

# String有重写Object的hashcode和toString吗？

- String 重写了 Object 类的 hashcode 和 toString 方法。

# 如果重写equals不重写hashcode会出现什么问题？

- 在存储散列集合时(如 Set 类)，如果原对象.equals(新对象)，但没有对 hashCode 重写，即两个对象拥有不同的 hashCode，**则在集合中将会存储两个值相同的对象，从而导致混淆。**因此在重写 equals 方法时，必须重写 hashCode 方法。