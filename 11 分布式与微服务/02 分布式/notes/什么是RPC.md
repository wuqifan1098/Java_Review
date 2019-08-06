# 如何科学的解释RPC

说起RPC，就不能不提到**分布式**，这个促使RPC诞生的领域。

假设你有一个计算器接口，Calculator，以及它的实现类CalculatorImpl，那么在系统还是**单体应用**时，你要调用Calculator的add方法来执行一个加运算，直接new一个CalculatorImpl，然后调用add方法就行了，这其实就是非常普通的**本地函数调用**，因为在**同一个地址空间**，或者说在同一块内存，所以通过方法栈和参数栈就可以实现。
 



![img](https:////upload-images.jianshu.io/upload_images/7143349-443a8cbcf6136ef5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/540/format/webp)





现在，基于高性能和高可靠等因素的考虑，你决定将系统改造为分布式应用，将很多可以共享的功能都单独拎出来，比如上面说到的计算器，你单独把它放到一个服务里头，让别的服务去调用它。



![img](https:////upload-images.jianshu.io/upload_images/7143349-9b1fbd80b8db018b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/901/format/webp)



这下问题来了，服务A里头并没有CalculatorImpl这个类，那它要怎样调用服务B的CalculatorImpl的add方法呢？

有同学会说，可以模仿B/S架构的调用方式呀，在B服务暴露一个Restful接口，然后A服务通过调用这个Restful接口来间接调用CalculatorImpl的add方法。

很好，这已经很接近RPC了，不过如果是这样，那每次调用时，是不是都需要写一串发起http请求的代码呢？比如httpClient.sendRequest...之类的，能不能像本地调用一样，去发起远程调用，让使用者感知不到远程调用的过程呢，像这样：

```
@Reference
private Calculator calculator;

...

calculator.add(1,2);

...
```

这时候，有同学就会说，用**代理模式**呀！而且最好是结合Spring IoC一起使用，通过Spring注入calculator对象，注入时，如果扫描到对象加了@Reference注解，那么就给它生成一个代理对象，将这个代理对象放进容器中。而这个代理对象的内部，就是通过httpClient来实现RPC远程过程调用的。

可能上面这段描述比较抽象，不过这就是很多RPC框架要解决的问题和解决的思路，比如阿里的Dubbo。

总结一下，**RPC要解决的两个问题：**

1. **解决分布式系统中，服务之间的调用问题。**
2. **远程调用时，要能够像本地调用一样方便，让调用者感知不到远程调用的逻辑。**

# 如何实现一个RPC

实际情况下，RPC很少用到http协议来进行数据传输，毕竟我只是想传输一下数据而已，何必动用到一个文本传输的应用层协议呢，我为什么不直接使用**二进制传输**？比如直接用Java的Socket协议进行传输？

不管你用何种协议进行数据传输，**一个完整的RPC过程，都可以用下面这张图来描述**：



![img](https:////upload-images.jianshu.io/upload_images/7143349-9e00bb104b9e3867.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/263/format/webp)



以左边的Client端为例，Application就是rpc的调用方，Client Stub就是我们上面说到的代理对象，也就是那个看起来像是Calculator的实现类，其实内部是通过rpc方式来进行远程调用的代理对象，至于Client Run-time Library，则是实现远程调用的工具包，比如jdk的Socket，最后通过底层网络实现实现数据的传输。

这个过程中最重要的就是**序列化**和**反序列化**了，因为数据传输的数据包必须是二进制的，你直接丢一个Java对象过去，人家可不认识，你必须把Java对象序列化为二进制格式，传给Server端，Server端接收到之后，再反序列化为Java对象。

下一次我也将通过代码，给大家演示一下，如何实现一个简单的RPC。

# RPC vs Restful

其实这两者并不是一个维度的概念，总得来说RPC涉及的维度更广。

如果硬要比较，那么可以从RPC风格的url和Restful风格的url上进行比较。

比如你提供一个查询订单的接口，用RPC风格，你可能会这样写：

```
/queryOrder?orderId=123
```

用Restful风格呢？

```
Get  
/order?orderId=123
```

**RPC是面向过程，Restful是面向资源**，并且使用了Http动词。从这个维度上看，Restful风格的url在表述的精简性、可读性上都要更好。

# RPC vs RMI

严格来说这两者也不是一个维度的。

RMI是Java提供的一种访问远程对象的协议，是已经实现好了的，可以直接用了。

而RPC呢？人家只是一种编程模型，并没有规定你具体要怎样实现，**你甚至都可以在你的RPC框架里面使用RMI来实现数据的传输**，比如Dubbo：[Dubbo - rmi协议](https://link.jianshu.com?t=http%3A%2F%2Fdubbo.apache.org%2Fbooks%2Fdubbo-user-book%2Freferences%2Fprotocol%2Frmi.html)

# RPC没那么简单

**要实现一个RPC不算难，难的是实现一个高性能高可靠的RPC框架。**

比如，既然是分布式了，那么一个服务可能有多个实例，你在调用时，要如何获取这些实例的地址呢？

这时候就需要一个服务注册中心，比如在Dubbo里头，就可以使用Zookeeper作为注册中心，在调用时，从Zookeeper获取服务的实例列表，再从中选择一个进行调用。

那么选哪个调用好呢？这时候就需要负载均衡了，于是你又得考虑如何实现复杂均衡，比如Dubbo就提供了好几种负载均衡策略。

这还没完，总不能每次调用时都去注册中心查询实例列表吧，这样效率多低呀，于是又有了缓存，有了缓存，就要考虑缓存的更新问题，blablabla......

你以为就这样结束了，没呢，还有这些：

- 客户端总不能每次调用完都干等着服务端返回数据吧，于是就要支持异步调用；
- 服务端的接口修改了，老的接口还有人在用，怎么办？总不能让他们都改了吧？这就需要版本控制了；
- 服务端总不能每次接到请求都马上启动一个线程去处理吧？于是就需要线程池；
- 服务端关闭时，还没处理完的请求怎么办？是直接结束呢，还是等全部请求处理完再关闭呢？
- ......

如此种种，都是一个优秀的RPC框架需要考虑的问题。

当然，接下来我们还是先实现一个简单的RPC，再在上面一步步优化！

传送门： [如何实现一个简单的RPC](https://www.jianshu.com/p/5b90a4e70783)

转载 ：https://www.jianshu.com/p/2accc2840a1b

