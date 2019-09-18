# 面试题

## 1. 为什么equals()方法要重写？

判断两个对象在逻辑上是否相等，如根据类的成员变量来判断两个类的实例是否相等，而继承Object中的equals方法只能**判断两个引用变量是否是同一个对象**。这样我们往往需要重写equals()方法。

## 2. 怎样重写equals()方法？

重写equals方法的要求：

1、**自反性**：对于任何非空引用x，x.equals(x)应该返回true。

2、**对称性**：对于任何引用x和y，如果x.equals(y)返回true，那么y.equals(x)也应该返回true。

3、**传递性**：对于任何引用x、y和z，如果x.equals(y)返回true，y.equals(z)返回true，那么x.equals(z)也应该返回true。

4、**一致性**：如果x和y引用的对象没有发生变化，那么反复调用x.equals(y)应该返回同样的结果。

5、**非空性**：对于任意非空引用x，x.equals(null)应该返回false。

## 3. 重写了equals方法都要进而重写Hashcode方法呢？

当equals此方法被重写时，通常有必要重写 hashCode 方法，以维护 hashCode 方法的常规协定，该**协定声明相等对象必须具有相等的哈希码**。

(1)当obj1.equals(obj2)为true时，obj1.hashCode() == obj2.hashCode()必须为true 

(2)当obj1.hashCode() == obj2.hashCode()为false时，obj1.equals(obj2)必须为false

这样如果我们对一个对象重写了euqals，意思是只要对象的成员变量值都相等那么euqals就等于true，但不重写hashcode，那么我们再new一个新的对象，当原对象.equals（新对象）等于true时，**两者的hashcode却是不一样的**，由此将产生了理解的不一致。

## 4. 重写 equals 不重写 hashcode 会出现什么问题

在存储散列集合时(如 Set 类)，如果原对象.equals(新对象)，但没有对 hashCode 重写，即两个对象拥有不同的 hashCode，则在集合中将会存**储两个值相同的对象，从而导致混淆**。因此在重写 equals 方法时，必须重写 hashCode 方法。

## hashCode()

**hashCode() 返回散列值**，而 equals() 是用来判断两个对象是否等价。等价的两个对象散列值一定相同，但是散列值相同的两个对象不一定等价。

在覆盖 equals() 方法时应当总是覆盖 hashCode() 方法，保证等价的两个对象散列值也相等。

下面的代码中，新建了两个等价的对象，并将它们添加到 HashSet 中。我们希望将这两个对象当成一样的，只在集合中添加一个对象，但是因为 EqualExample 没有实现 hasCode() 方法，因此这两个对象的散列值是不同的，最终导致集合添加了两个等价的对象。

```java
EqualExample e1 = new EqualExample(1, 1, 1);
EqualExample e2 = new EqualExample(1, 1, 1);
System.out.println(e1.equals(e2)); // true
HashSet<EqualExample> set = new HashSet<>();
set.add(e1);
set.add(e2);
System.out.println(set.size());   // 2
```

理想的散列函数应当具有均匀性，即不相等的对象应当均匀分布到所有可能的散列值上。这就要求了散列函数要把所有域的值都考虑进来。可以将每个域都当成 R 进制的某一位，然后组成一个 R 进制的整数。R 一般取 31，因为它是一个奇素数，如果是偶数的话，当出现乘法溢出，信息就会丢失，因为与 2 相乘相当于向左移一位。

一个数与 31 相乘可以转换成移位和减法：`31*x == (x<<5)-x`，编译器会自动进行这个优化。

```java
@Override
public int hashCode() {
    int result = 17;
    result = 31 * result + x;
    result = 31 * result + y;
    result = 31 * result + z;
    return result;
}
```

## toString()

默认返回 ToStringExample@4554617c 这种形式，其中 @ 后面的数值为散列码的无符号十六进制表示。

```java
public class ToStringExample {

    private int number;

    public ToStringExample(int number) {
        this.number = number;
    }
}
ToStringExample example = new ToStringExample(123);
System.out.println(example.toString());
ToStringExample@4554617c
```