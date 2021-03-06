

# 面试题

## 1.栈和堆的区别（贝贝）

- 存储数据 

**栈中主要存放一些基本数据类型的变量（byte，short，int，long，float，double，boolean，char）和对象的引用**，每个线程都由一个独立的栈空间，线程是**不共享数据的**。

堆主要存储**实例化的对象，数组**，由**JVM动态分配空间**，线程可以**共享数据**。

- 存取速度

栈内存的**存取速度要快**于堆内存，但栈中的数据大小和生存周期是确定的，缺乏灵活性。堆内存可以动态的分配内存大小，JVM也不会自动的收走这些数据，但动态分配内存，**存取速度比较慢。**

- 是否共享

栈是私有的，不共享数据。堆是共享的

- 异常错误

如果栈内存没有可用的空间存储方法调用和局部变量，JVM会抛出java.lang.StackOverFlowError。
而如果是堆内存没有可用的空间存储生成的对象，JVM会抛出java.lang.OutOfMemoryError。

- 空间大小

**栈的内存要远远小于堆内存**，如果你使用递归的话，那么你的栈很快就会充满。如果递归没有及时跳出，很可能发生StackOverFlowError问题。

你可以通过-Xss选项设置栈内存的大小。-Xms选项可以设置堆的开始时的大小，-Xmx选项可以设置堆的最大值。

 https://segmentfault.com/a/1190000014960714

## 2. 本地方法栈和虚拟机栈

虚拟机栈描述的Java方法执行的内存模型：每个方法执行的同时会创建一个栈帧。 栈帧用于存储**局部变量表、操作数栈、动态链接、方法返回**等信息。 每个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中入栈到出栈的过程。 

在活动线程中，只有位于栈顶的栈帧才是有效的，称为当前栈帧，与这个栈帧相关联的方法称为当前方法。执行引擎运行的所有字节码指令都只针对当前栈帧进行操作。

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/stack-Frame.png)

本地方法栈它们之间的区别不过是虚拟机栈为虚拟机执行Java方法服务（也就是字节码）服务，而本地方法栈**为虚拟机使用到的Native方法服务。**

# 运行时数据区域

JDK 1.8之前：

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/jdk1.8%E5%89%8D%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84.png)

JDK 1.8 ：

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/jdk1.8%E5%90%8E%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84.png)

## 1.程序计数器（线程私有）

字节码解释器工作时通过改变**这个计数器的值**来选**取下一条需要执行的字节码指令**，分支、循环、跳转、异常处理、线程恢复等功能都需要依赖这个计数器来完。

为了线程切换后能恢复到正确的执行位置，**每条线程都需要有一个独立的程序计数器，各线程之间计数器互不影响，独立存储**，我们称这类内存区域为“线程私有”的内存。

从上面的介绍中我们知道程序计数器主要有两个作用：

1. 字节码解释器通过**改变程序计数器来依次读取指令**，从而实现代码的流程控制，如：顺序执行、选择、循环、异常处理。
1. 在多线程的情况下，程序计数器用于**记录当前线程执行的位置**，从而当线程被切换回来的时候能够知道该线程上次运行到哪儿了。

程序计数器是唯一一个**不会出现 OutOfMemoryError 的内存区域**，它的生命周期随着线程的创建而创建，随着线程的结束而死亡。

## 2.虚拟机栈（线程私有）

与程序计数器一样，Java虚拟机栈也是**线程私有的**，它的生命周期和线程相同，描述的是 Java 方法执行的内存模型，每次方法调用的数据都是通过栈传递的。

Java 内存可以粗糙的区分为堆内存（Heap）和栈内存(Stack),其中栈就是现在说的虚拟机栈，或者说是虚拟机栈中局部变量表部分。 （实际上，Java虚拟机栈是由一个个栈帧组成，而每个栈帧中都拥有：**局部变量表、操作数栈、动态链接、方法出口**信息。）

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/%E8%99%9A%E6%8B%9F%E6%9C%BA%E6%A0%88%E5%B8%A7.png)

局部变量表主要存放了编译器可知的各种数据类型（boolean、byte、char、short、int、float、long、double）、对象引用（reference类型，它不同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置）。

Java 虚拟机栈会出现两种异常：**StackOverFlowError 和 OutOfMemoryError**。

- StackOverFlowError： **若Java虚拟机栈的内存大小不允许动态扩展，那么当线程请求栈的深度超过当前Java虚拟机栈的最大深度的时候**，就抛出StackOverFlowError异常。
- OutOfMemoryError： **若 Java 虚拟机栈的内存大小允许动态扩展，且当线程请求栈时内存用完了，无法再动态扩展了**，此时抛出OutOfMemoryError异常。

可以通过 -Xss 这个虚拟机参数来指定一个程序的 Java 虚拟机栈内存大小：

    java -Xss=512M HackTheJava

## 3.本地方法栈（线程私有）

本地方法一般是用其它语言**（C、C++ 或汇编语言等）**编写的，并且被编译为基于本机硬件和操作系统的程序，对待这些方法需要特别处理。

和虚拟机栈所发挥的作用非常相似，区别是： 虚拟机栈为虚拟机执行 Java 方法 （也就是字节码）服务，而**本地方法栈则为虚拟机使用到的 Native 方法服务**。 在 HotSpot 虚拟机中和 Java 虚拟机栈合二为一。

## 4.堆（线程共享）

Java 虚拟机所管理的内存中最大的一块，Java 堆是所有线程共享的一块内存区域，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数组都在这里分配内存。

Java 堆是垃圾收集器管理的主要区域，因此也被称作GC堆（Garbage Collected Heap）.从垃圾回收的角度，由于现在收集器基本都采用分代垃圾收集算法，所以Java堆还可以细分为：新生代和老年代：再细致一点有：Eden空间、From Survivor、To Survivor空间等。**进一步划分的目的是更好地回收内存，或者更快地分配内存**。

**新生代 （Young Generation）**
在方法中去 new 一个对象，那这方法调用完毕后，对象就会被回收，这就是一个典型的新生代对象。

**老年代 （Old Generation）**

在新生代中经历了 N 次垃圾回收后仍然存活的对象就会被放到老年代中。而且大对象直接进入老年代
当 Survivor 空间不够用时，需要依赖于老年代进行分配担保，所以大对象直接进入老年代

**永久代 （Permanent Generation）**

即方法区。
当一个对象被创建时，它首先进入新生代，之后有可能被转移到老年代中。

可以通过 -Xms 和 -Xmx 两个虚拟机参数来指定一个程序的 Java 堆内存大小，第一个参数设置初始值，第二个参数设置最大值。

    java -Xms=1M -Xmx=2M HackTheJava

## 5.方法区 （线程共享）

方法区与 Java 堆一样，是各个线程共享的内存区域，它用于存储已被**虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码**等数据。虽然Java虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫做 Non-Heap（非堆），目的应该是与 Java 堆区分开来。

**方法区和永久代的关系**

方法区和永久代的关系很像Java中接口和类的关系，类实现了接口，而永久代就是HotSpot虚拟机对虚拟机规范中方法区的一种实现方式。

**为什么要将永久代(PermGen)替换为元空间(MetaSpace)呢?**

## 6.运行时常量池

JDK1.7及之后版本的 JVM 已经将运行时常量池从**方法区中移了出来**，在 Java **堆（Heap）中**开辟了一块区域存放运行时常量池。

## 7.直接内存

直接内存并不是虚拟机运行时数据区的一部分，也不是虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用。而且也可能导致 OutOfMemoryError 异常出现。

JDK1.4 中新加入的 NIO(New Input/Output) 类，引入了一种基于通道（Channel） 与缓存区（Buffer） 的 I/O 方式，它可以直接使用 Native 函数库直接分配堆外内存，然后通过一个存储在 Java 堆中的 DirectByteBuffer 对象作为这块内存的引用进行操作。这样就能在一些场景中显著提高性能，因为避免了在 **Java 堆和 Native 堆之间来回复制数据。**

