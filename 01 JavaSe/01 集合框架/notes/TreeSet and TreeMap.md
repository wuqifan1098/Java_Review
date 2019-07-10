# 面试题

## 1.TreeMap 最大的特点是什么？

TreeMap 在一个“红-黑”树的基础上实现。查看键或者“键-值”对时，它们会按固定的顺序排列(取决于Comparable或Comparator，稍后即会讲到)。TreeMap最大的好处就是我们得到的是**已排好序的结果**。

在需要排序的Map时候才用TreeMap。

# TreeSet and TreeMap

# 总体介绍

之所以把TreeSet和TreeMap放在一起讲解，是因为二者在Java里有着相同的实现，前者仅仅是对后者做了一层包装，也就是说TreeSet里面有一个**TreeMap（适配器模式）**。因此本文将重点分析TreeMap。

TreeMap底层通过**红黑树**（Red-Black tree）实现，也就意味着containsKey(), get(), put(), remove()都有着**log(n)**的时间复杂度。

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/TreeMap_base.png)

**红黑树是一种近似平衡的二叉查找树，它能够确保任何一个节点的左右子树的高度差不会超过二者中较低那个的一陪。**

- 每个节点要么是红色，要么是黑色。
- **根节点必须是黑色**
- **红色节点不能连续**（也即是，红色节点的孩子和父亲都不能是红色）。
- 对于每个节点，从该点至null（树尾端）的任何路径，**都含有相同个数的黑色节点**。

# 预备知识

当查找树的结构发生改变时，红黑树的约束条件可能被破坏，**需要通过调整使得查找树重新满足红黑树的约束条件。**调整可以分为两类：一类是颜色调整，即**改变某个节点的颜色**；另一类是结构调整，集改变检索树的结构关系。结构调整过程包含两个基本操作：**左旋（Rotate Left），右旋（RotateRight）。**

## 左旋

左旋的过程是将x的**右子树绕x逆时针旋转**，**使得x的右子树成为x的父亲**，同时修改相关节点的引用。旋转之后，二叉查找树的属性仍然满足。

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/TreeMap_rotateLeft.png)

## 右旋

右旋的过程是将x的**左子树绕x顺时针旋转，使得x的左子树成为x的父亲**，同时修改相关节点的引用。旋转之后，二叉查找树的属性仍然满足。

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/TreeMap_rotateRight.png)

# TreeSet

TreeSet是对TreeMap的简单包装，对TreeSet的函数调用都会转换成合适的TreeMap方法，因此TreeSet的实现非常简单。

```java
// TreeSet是对TreeMap的简单包装
public class TreeSet<E> extends AbstractSet<E>
implements NavigableSet<E>, Cloneable, java.io.Serializable
{
......
private transient NavigableMap<E,Object> m;
// Dummy value to associate with an Object in the backing Map
private static final Object PRESENT = new Object();
public TreeSet() {
    this.m = new TreeMap<E,Object>();// TreeSet里面有一个TreeMap
}
......
public boolean add(E e) {
    return m.put(e, PRESENT)==null;
}
......
}
```