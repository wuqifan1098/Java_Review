# 面试题

## 1. Java类初始化顺序

https://www.cnblogs.com/fly-piglet/p/8766226.html

# static

**无需创建对象就可以调用（方法、属性）**。

1.静态变量：static修饰的属性，称为类属性，即全局变量。前面已经有提及。

(1).静态变量可以使用类名直接访问，也可以使用对象名进行访问。

```java
 class Number
  {
      int a;
     static int b;
  }
  public class T01
  {
      public static void main(String[] args)
      {
         Number.a=5;//错误的
         Number.b=6;//正确的，b是static修饰的静态变量可以直接用类名调用
     }
 }
```

(2).static不允许修饰局部变量：

```java
public static void max(int a,int b)
 {
        static int result;//Java中不被允许   
}
```

(3).非静态和静态方法中均可以访问static变量。

2.静态方法：static修饰的方法，称为类方法。

(1).静态方法中可以调用类中的静态变量，但不能调用非静态变量。

```java
  class Number
  {
      int a;
      static int b;
      public static void max(int c,int d)
        {
             int result;
             result=a+b;//a是非静态变量不能被静态方法直接调用
             
       }
 }
```

那么如何在静态方法中调用非静态变量呢?

```java
  class Number
  {
      int a;
      static int b;
      public static void max()
        {
             int result;
            Number num = new Number();
             num.a=5;
            result=num.a+b;//借助对象调用非静态变量a
            System.out.println(result);
       }
 }
 
 public class T01
 {
     public static void main(String[] args)
    {
         Number.b=6;
         Number.max();
     }
 }
```

(2).static方法没有this—不依附于任何对象。

```java
  class Person
 {
     static String name;
     static int age;
     public static void print(String n,int a)
     {
         this.name=n;//错误的，静态方法中无法使用this
         age=a;
     }
 }
```

3.静态初始化块—**类加载时执行，且只会执行一次，只能给静态变量赋值**，不能接受任何参数。

```java
  class Number
  {
     static int a;
     static int b;
     public static void print()
     {
         int result;
         result=a+b;
         System.out.println(result);
     }
     static//static语句块用于初始化static变量，是最先运行的语句块
     {
         a=5;
         b=8;
     }
 }
 public class T01
 {
     public static void main(String[] args)
     {
         Number.print();
     }
 }
```

因为static语句块只会在类加载时执行一次的特性，经常用来优化程序性能。 

# final

“只读”修饰符

1.final和static一起使用，声明一个常量。
**public static final int Age=10;//常量名大写**
final成员变量必须在声明的时候初始化或在构造方法中初始化，不能再次赋值。
final局部变量必须在声明时赋值。
2.final方法，这个方法不可以被子类方法重写。

```java
 class Person
 {
    public final String getName
    {
         return name;
    }
 }
```

3.final类，这种类无法被继承。

```java
final class Person
{
}
```

4.final类不可能是abstract的。
5.final和static一样，合理的使用都可以提高程序性能。

# this

**代表当前对象，封装对象属性**

1.使用this调用本类中的成员变量。

```java
public void setName(String name)
{
     this.name=name;//类似于指针的作用
}
```

2.当成员变量和方法中的局部变量名字相同时，使用this调用成员变量。

3.this在方法内部调用本类中的其他方法。

```java
 class Person
 {
     int age;
     public void setAge(int age)
     {
          this.age=age;
          this.print();//调用本类中的print()方法
     }
     public int getAge()
     {
         return age;
     }
     public void print()
     {
         System.out.println("我今年是："+age+"岁");
     }
 }
 public class T01
 {
     public static void main(String[] args)
     {
         Person p = new Person();
         p.setAge(12);
     }
 }
```

4.this在方法中调用类的构造函数，必须位于第一行。

this(12,"Mary");//位于第一行


