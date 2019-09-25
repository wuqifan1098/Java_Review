# 面试题

## 1. Java类初始化顺序

https://www.cnblogs.com/fly-piglet/p/8766226.html

## 2. static有什么用途？（华为）

static对象可以在它的任何对象创建之前访问，无需引用任何对象。

就是在调用类的成员变量或方法时，可以**直接使用类名.方法名**，而**无需再去使用new一个类**，然后再使用对象引用来调用，使用起来更便捷。而用static修饰的成员变量和方法跟上述类似。故使用static提高了共享性。

非static方法**必须等对象被new出来以后才能使用**，因而不能在main中直接调用。

static主要是**减少成员变量和方法的多次创建**，一旦声明为静态，该成员变量或方法就属于这个类，可以**被该类所创建的所有对象共享**，就能直接用类名来调用

## 3. 静态内部类 和 成员类的区别（招银网络科技）

（1）内部静态类**不需要有指向外部类的引用**。但非静态内部类**需要持有对外部类的引用**。

（2）非静态内部类能**够访问外部类的静态和非静态成员**。静态类**不能访问外部类的非静态成员**。他只能访问外部类的静态成员。

（3）一个非静态内部类**不能脱离外部类实体被创建**，一个非静态内部类**可以访问外部类的数据和方法**，因为他就在外部类里面。

https://blog.csdn.net/qq_21918021/article/details/88052728

https://www.jianshu.com/p/ed3f8b0f0471

# static

## 静态变量

static修饰的属性，称为类属性，即全局变量。前面已经有提及。

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

按照是否静态的对类成员变量进行分类可分两种：一种是被static修饰的变量，叫静态变量或类变量；另一种是没有被static修饰的变量，叫实例变量。

两者的区别是：

- 对于静态变量在内存中只有一个拷贝（节省内存），JVM只为静态分配一次内存，**在加载类的过程中完成静态变量的内存分配**，可用类名直接访问（方便），当然也可以通过对象来访问（但是这是不推荐的）。
- 对于实例变量，**每创建一个实例，就会为实例变量分配一次内存**，实例变量可以在内存中有多个拷贝，互不影响（灵活）。

所以一般在需要实现以下两个功能时使用静态变量： - 在对象之间共享值时 - 方便访问变量时

## 静态方法

static修饰的方法，称为类方法。

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

new 类之后 借助对象调用非静态变量

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

静态方法可以直接通过类名调用，任何的实例也都可以调用， **因此静态方法中不能用this和super关键字，不能直接访问所属类的实例变量和实例方法(就是不带static的成员变量和成员成员方法)，只能访问所属类的静态成员变量和成员方法。 因为实例成员与特定的对象关联！！**

因为static方法独立于任何实例，因此static方法必须被实现，而不能是抽象的abstract。

## 静态初始化块

**类加载时执行，且只会执行一次，只能给静态变量赋值**，不能接受任何参数。

static代码块也叫静态代码块，是在类中独立于类成员的static语句块，可以有多个，位置可以随便放，它不在任何的方法体内，JVM加载类时会执行这些静态的代码块，如果static代码块有多个，JVM将按照它们在类中出现的先后顺序依次执行它们，每个代码块只会被执行一次

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

## static和final

static final用来修饰成员变量和成员方法，可简单理解为“全局常量”！

**对于变量，表示一旦给值就不可修改，并且通过类名可以访问。对于方法，表示不可覆盖，并且可以通过类名直接访问。**

https://zhuanlan.zhihu.com/p/42961231

## 初始化块、静态初始化块、构造函数的执行顺序

## 执行顺序

首先定义A, B, C三个类用作测试，其中B继承了A，C又继承了B，并分别给它们加上静态初始化块、非静态初始化块和构造函数，里面都是一句简单的输出。主类Main里面也如法炮制。 测试代码

```java
class A {
    static {
        System.out.println("Static init A.");
    }

    {
        System.out.println("Instance init A.");
    }

    A() {
        System.out.println("Constructor A.");
    }
}

class B extends A {
    static {
        System.out.println("Static init B.");
    }

    {
        System.out.println("Instance init B.");
    }

    B() {
        System.out.println("Constructor B.");
    }
}

class C extends B {

    static {
        System.out.println("Static init C.");
    }

    {
        System.out.println("Instance init C.");
    }

    C() {
        System.out.println("Constructor C.");
    }
}

public class Main {

    static {
        System.out.println("Static init Main.");
    }

    {
        System.out.println("Instance init Main.");
    }

    public Main() {
        System.out.println("Constructor Main.");
    }

    public static void main(String[] args) {
        C c = new C();
        //B b = new B();
    }
}
```

当然这里不使用内部类，因为内部类不能使用静态的定义；而用静态内部类就失去了一般性。那么可以看到，当程序进入了main函数，并创建了一个类C的对象之后，输出是这样子的：

```text
Static init Main.
Static init A.
Static init B.
Static init C.
Instance init A.
Constructor A.
Instance init B.
Constructor B.
Instance init C.
Constructor C.
```

观察上面的输出，可以观察到两个有趣的现象：

1)Main类是肯定没有被实例化过的，但是由于执行main入口函数用到了Main类，于是static初始化块也被执行了；

2)所有的静态初始化块都优先执行，其次才是非静态的初始化块和构造函数，它们的执行顺序是：

- 父类的静态初始化块
- 子类的静态初始化块
- 父类的初始化块
- 父类的构造函数
- 子类的初始化块
- 子类的构造函数

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


