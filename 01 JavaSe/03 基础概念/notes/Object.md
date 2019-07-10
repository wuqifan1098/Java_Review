## Object类简介

Object类是所有类的父类，在Java中只有基本数据类型不是对象。对于所有数组类型（对象类型&&基本数据类型数组）都继承于Object类；

## equals方法

Object类中通过判断两个对象是否具有相同引用，从而判断两个对象是否相同；
**子类只要重写equals方法，就必须重写hashCode方法**

```java
// in java.lang.Object
public boolean equals(Object obj) {
    return (this == obj);
}
```

### 重写equals方法原则：

- **自反性**：A.equals(A)返回true；
- **对称性**：A.equals(B)结果和B.equals(A)相同；
- **传递性**：A.equals(B)为true，B.equals(C)为true，则A.equals(C)为true
- **一致性**：对于非null引用A,B， 只要equals()比较操作所用到对象信息不变，多次调用A.equals(B)，结果一致；
- 对于任何非null引用，x.equals(null)必须返回false；
- **重写equals方法时，参数类型必须为Object类型**

### 重写equals方法示例

```java
class myObject {
    private String name;
    private int age;
    ...
    public getName() {
        return this.name;
    }
    public getAge() {
        return this.age;
    }
}

/**
 * 重写equals方法demo步骤
 * Effective Java中推荐方式
 */
public boolean equals(Object x) {
    // 1. 检查x和this是否引用同一个对象
    if (x == this) {
        return true;
    }
    
    // 2. 检查x对象类型是否是myObject派生
    if (!(x instanceof myObject)) {
        return false;
    }

    // 3. 比较数据域
    // 经过1，2检查，将参数转换为正确类型
    myObject o = (myObject)(x);
    return this.name.equals(x.getName()) && this.age == (x.getAge());
}
```

## hashCode()方法

`hashCode方法`返回对象的散列码，**相等对象必须返回相等的hashCode，不同对象的hashCode尽可能不相等**；

```java
// in java.lang.Object
public native int hashCode();
```

## 重写equals时总要重写hashCode

- 重写equals不重写hashCode，会导致**“不相等对象拥有相同的hashCode”**，导致集合类`HashMap`，`HashSet`和`Hashtable`无法工作；极端情况下，在散列表中使所有对象的hashCode都相等，所有对象都被映射到同一个桶中，散列表退化成链表；
- 当两个对象调用equal返回**true**，则两个对象各自调用hashCode()返回**相同**hashCode；
- 当两个对象调用equal返回false， 两个对象各自调用hashCode()返回的hashCode可以相同（**散列冲突不能完全避免**）

## toString()方法

Object类中toString方法，输出对象的“对象类名@散列码”；