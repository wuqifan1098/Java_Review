# 面试题

## 1.LinkedHashMap如何实现有序访问（阿里）



## [LinkedHashMap](https://cyc2018.github.io/CS-Notes/#/notes/Java 容器?id=linkedhashmap)

### [存储结构](https://cyc2018.github.io/CS-Notes/#/notes/Java 容器?id=存储结构)

继承自 HashMap，因此具有和 HashMap 一样的快速查找特性。

```java
public class LinkedHashMap<K,V> extends HashMap<K,V> implements Map<K,V>Copy to clipboardErrorCopied
```

内部维护了一个双向链表，用来维护插入顺序或者 LRU 顺序。

```java
/**
 * The head (eldest) of the doubly linked list.
 */
transient LinkedHashMap.Entry<K,V> head;

/**
 * The tail (youngest) of the doubly linked list.
 */
transient LinkedHashMap.Entry<K,V> tail;Copy to clipboardErrorCopied
```

accessOrder 决定了顺序，默认为 false，此时维护的是插入顺序。

```java
final boolean accessOrder;Copy to clipboardErrorCopied
```

LinkedHashMap 最重要的是以下用于维护顺序的函数，它们会在 put、get 等方法中调用。

```java
void afterNodeAccess(Node<K,V> p) { }
void afterNodeInsertion(boolean evict) { }
```

