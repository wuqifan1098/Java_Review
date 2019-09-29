# 面试题

## 1. 讲讲tomcat（大华）

# Tomcat的基本原理

##  Tomcat的架构

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181111152014980-134475588.png)

   **Tomcat主要组件：服务器Server，服务Service，连接器Connector、容器Container。**
   **连接器Connector和容器Container是Tomcat的核心。**

- Server：服务器的意思，代表整个tomcat服务器，一个tomcat只有一个Server；
- Service：Server中的一个逻辑功能层， **一个Server可以包含多个Service**；
- Connector：称作连接器，是Service的核心组件之一，一个Service可以有多个Connector，主要是**连接客户端请求**；
- Container：Service的另一个核心组件，按照层级有Engine，Host，Context，Wrapper四种，一个Service只有一个
- Engine，其主要作用是**执行业务逻辑**；
- Jasper：JSP引擎；
- Session：会话管理；
  

   **Tomcat 还有其它重要的组件，如安全组件 security、logger 日志组件、session、mbeans、naming 等其它组件。这些组件共同为 Connector 和 Container 提供必要的服务。**
   一个Container容器和一个或多个Connector组合在一起，加上其他一些支持的组件共同组成一个Service服务，有了Service服务便可以对外提供能力了，但是Service服务的生存需要一个环境，这个环境便是Server，Server组件为Service服务的正常使用提供了生存环境，Server组件可以同时管理一个或多个Service服务。

```java
<?xml version="1.0" encoding="UTF-8"?>
<Server port="8005" shutdown="SHUTDOWN">
    <Listener className="org.apache.catalina.startup.VersionLoggerListener"/>
    <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on"/>
    <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener"/>
    <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener"/>
    <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener"/>

    <GlobalNamingResources>
        <Resource name="UserDatabase" auth="Container"
                  type="org.apache.catalina.UserDatabase"
                  description="User database that can be updated and saved"
                  factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
                  pathname="conf/tomcat-users.xml"/>
    </GlobalNamingResources>

    <Service name="Catalina">
        <Connector port="8080" protocol="HTTP/1.1"
                   connectionTimeout="20000"
                   redirectPort="8443"/>
        <Connector port="8009" protocol="AJP/1.3" redirectPort="8443"/>
        <Engine name="Catalina" defaultHost="localhost">
            <Realm className="org.apache.catalina.realm.LockOutRealm">
                <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
                       resourceName="UserDatabase"/>
            </Realm>

            <Host name="localhost" appBase="webapps"
                  unpackWARs="true" autoDeploy="true">
                <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
                       prefix="localhost_access_log" suffix=".txt"
                       pattern="%h %l %u %t &quot;%r&quot; %s %b"/>
            </Host>
        </Engine>
    </Service>
</Server>

```

## Service

可以想象，一个Server服务器，它最基本的功能肯定是：

> 接收客户端的请求，然后解析请求，完成相应的业务逻辑，然后把处理后的结果返回给客户端，一般会提供两个节本方法，一个start打开服务Socket连接，监听服务端口，一个stop停止服务释放网络资源。

这时的服务器就是一个Server类：

但如果将请求监听和请求处理放在一起，扩展性会变差，毕竟网络协议不止HTTP一种，如果想适配多种网络协议，请求处理又相同，这时就无能为力了，tomcat的设计大师不会采取这种做法，而是将请求监听和请求处理分开为两个模块，分别是Connector和Container，**Connector负责处理请求监听，Container负责处理请求处理。**

但显然tomcat可以有多个Connector，同时Container也可以有多个。那这就存在一个问题，哪个Connector对应哪个Container，提供复杂的映射吗？相信看过server.xml文件的人已经知道了tomcat是怎么处理的了。

没错，Service就是这样来的。在conf/server.xml文件中，可以看到Service组件包含了Connector组件和Engine组件（前面有提过，Engine就是一种容器），即Service相当于Connector和Engine组件的包装器，将一个或者多个Connector和一个Engine建立关联关系。在默认的配置文件中，定义了一个叫Catalina 的服务，它将HTTP/1.1和AJP/1.3这两个Connector与一个名为Catalina 的Engine关联起来。

**一个Server可以包含多个Service（它们相互独立，只是公用一个JVM及类库），一个Service负责维护多个Connector和一个Container。**

其标准实现是StandardService，UML类图如下：

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/20190929171259.png)

这时tomcat就是这样了：

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/20190929171228.png)

## Connector

前面介绍过Connector是连接器，用于接受请求并将请求封装成Request和Response，然后交给Container进行处理，Container处理完之后在交给Connector返回给客户端。

一个Connector会**监听一个独立的端口来处理来自客户端的请求**。server.xml默认配置了两个Connector：

- <Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443"/>，它监听端口8080,这个端口值可以修改，connectionTimeout定义了连接超时时间，单位是毫秒，redirectPort 定义了ssl的重定向接口，根据上述配置，Connector会将ssl请求转发到8443端口。
- <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />, AJP表示Apache Jserv Protocol，它将处理Tomcat和Apache http服务器之间的交互，此连接器用于处理我们将Tomcat和Apache http服务器结合使用的情况，如在同一台物理Server上部署一个Apache http服务器和多台Tomcat服务器，通过Apache服务器来处理静态资源以及负载均衡时，针对不同的Tomcat实例需要AJP监听不同的端口。

Connector在tomcat中的设计比较复杂，先大致列上一个图：
![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/20190929171435.png)

Connector使用ProtocolHandler来处理请求的，不同的ProtocolHandler代表不同的连接类型，比如：Http11Protocol使用的是普通Socket来连接的（tomcat9已经删除了这个类，不再采用BIO的方式），Http11NioProtocol使用的是NioSocket来连接的。

其中ProtocolHandler由包含了三个部件：Endpoint、Processor、Adapter。

1. Endpoint用来处理底层Socket的网络连接，Processor用于将Endpoint接收到的Socket封装成Request（这个Request和ServletRequest无关），Adapter充当适配器，用于将Request转换为ServletRequest交给Container进行具体的处理。
2. Endpoint由于是处理底层的Socket网络连接，因此Endpoint是用来实现TCP/IP协议的，而Processor用来实现HTTP协议的，Adapter将请求适配到Servlet容器进行具体的处理。
3. Endpoint的抽象实现AbstractEndpoint里面定义的Acceptor和AsyncTimeout两个内部类和一个Handler接口。Acceptor用于监听请求，AsyncTimeout用于检查异步Request的超时，Handler用于处理接收到的Socket，在内部调用Processor进行处理。
   

##  Container

​    **Container是容器的父接口，**该容器的设计用的是典型的**责任链的设计模式**，它由四个自容器组件构成，分别是**Engine、Host、Context、Wrapper**。这四个组件是负责关系，存在包含关系。***通常一个Servlet class对应一个Wrapper，如果有多个Servlet定义多个Wrapper，***如果有多个Wrapper就要定义一个更高的Container，如Context。 Context 还可以定义在父容器 Host 中，一个Host可以对应多个Context，Host 不是必须的，但是要运行 war 程序，就必须要 Host，因为 war 中必有 web.xml 文件，这个文件的解析就需要 Host 了，***如果要有多个 Host 就要定义一个 top 容器 Engine 了***。而 Engine 没有父容器了，一个 Engine 代表一个完整的 Servlet 引擎。

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181111164510371-471190422.png)
    **Engine 容器比较简单，它只定义了一些基本的关联关系。**
    **Host 容器**是Engine 的子容器，一个Host在 Engine 中代表一个**虚拟主机**，这个虚拟主机的作用就是运行多个应用，它负责安装和展开这些应用，并且标识这个应用以便能够区分它们。它的子容器通常是 Context，它除了关联子容器外，**还有就是保存一个主机应该有的信息**。
    **Context 容器**代表 Servlet 的 Context，它具备了 Servlet 运行的基本环境，理论上只要有 Context 就能运行 Servlet 了。简单的 Tomcat 可以没有 Engine 和 Host。Context 最重要的功能就是管理它里面的 Servlet 实例，***Servlet 实例在 Context 中是以 Wrapper 出现的，***还有一点就是 Context 如何找到正确的 Servlet 来执行它呢，Tomcat5 以前是通过一个 Mapper 类来管理的，Tomcat5 以后这个功能被移到了 **request** 中。
    **Wrapper容器**代表一个 Servlet，负责管理一个Servlet，包括的 Servlet 的装载、初始化、执行以及资源回收。Wrapper是最底层的容器，它没有子容器，所以调用它的addChild 将会报错。 
    Wrapper 的实现类是StandardWrapper，StandardWrapper还实现了拥有一个Servlet 初始化信息的 ServletConfig，由此看出 StandardWrapper将直接和 Servlet 的各种信息打交道。

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181111152019164-1364893646.png)

## HTTP请求过程

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181111152024363-1479165699.png)

​    **Tomcat Server处理一个HTTP请求的过程：**

 1 用户点击网页内容，请求被发送到本机端口8080，被在那里监听的Coyote HTTP/1.1 Connector获得。
 2 Connector把该请求交给**它所在的Service的Engine来处理，并等待Engine的回应**。
 3 Engine获得请求localhost/test/index.jsp，匹配所有的虚拟主机Host。
 4 Engine匹配到名为localhost的Host（即使匹配不到也把请求交给该Host处理，因为该Host被定义为该Engine的默认主机），名为localhost的Host获得请求/test/index.jsp，匹配它所拥有的所有的Context。Host匹配到路径为/test的Context（如果匹配不到就把该请求交给路径名为“ ”的Context去处理）。
 5 path=“/test”的Context获得请求/index.jsp，在它的mapping table中寻找出对应的Servlet。Context匹配到URL PATTERN为*.jsp的Servlet,对应于Jsp的Servlet类。
 6 构造HttpServletRequest对象和HttpServletResponse对象，作为参数调用Servlet的doGet（）或doPost（），执行业务逻辑、数据存储等程序。
 7 Context把执行完之后的HttpServletResponse对象返回给Host。
 8 Host把HttpServletResponse对象返回给Engine。
 9 Engine把HttpServletResponse对象返回Connector。
10 Connector把HttpServletResponse对象返回给客户Browser。

## Tomncat启动流程

tomcat的启动流程很标准化，入口是BootStrap，统一按照生命周期管理接口Lifecycle的定义进行启动。首先，调用init()方法逐级初始化，接着调用start()方法进行启动，同时，每次调用伴随着生命周期状态变更事件的触发。

每一级组件除完成自身的处理外，还有负责调用子组件的相关调用，组件和组件之间是松耦合的，可以通过配置进行修改。
![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/20190929171745.png)

##  server.xml配置

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181111155319826-1701278516.png)

```
 1  1、Server：server.xml的最外层元素。常用属性：
 2  　　port：Tomcat监听shutdown命令的端口。
 3  　　shutdown：通过指定的端口（port）关闭Tomcat所需的字符串。修改shutdown的值，对shutdown.bat无影响
 4  2. Listener：Listener即监听器，负责监听特定的事件，当特定事件触发时，Listener会捕捉到该事件，并做出相应处理。Listener通常用在Tomcat的启动和关闭过程。Listener可嵌在Server、Engine、Host、Context内。常用属性：
 5  　　className：指定实现org.apache.catalina.LifecycleListener接口的类
 6  3. GlobalNamingResources：用于配置JNDI。
 7  4. Service：包装Executor、Connector、Engine，以组成一个完整的服务， Server可以包含多个Service组件。常用属性：
 8     className：指定实现org.apache.catalina. Service接口的类，默认值为org.apache.catalina.core.StandardService
 9     name：Service的名字；
10  5. Executor：即Service提供的线程池，供Service内各组件使用，常用属性：
11         className：指定实现org.apache.catalina. Executor接口的类，默认值为org.apache.catalina.core. StandardThreadExecutor
12         name：线程池的名字
13         daemon：是否为守护线程，默认值为true
14         maxIdleTime：总线程数高于minSpareThreads时，空闲线程的存活时间（单位：ms），默认值为60000，即1min
15         maxQueueSize：任务队列上限，默认值为Integer.MAX_VALUE(（2147483647），超过此值，将拒绝
16         maxThreads：线程池内线程数上限，默认值为200
17         minSpareThreads：线程池内线程数下限，默认值为25
18         namePrefix：线程名字的前缀。线程名字通常为namePrefix+ threadNumber
19         prestartminSpareThreads：是否在Executor启动时，就生成minSpareThreads个线程。默认为false
20         threadPriority：Executor内线程的优先级，默认值为5（Thread.NORM_PRIORITY）
21         threadRenewalDelay：重建线程的时间间隔。重建线程池内的线程时，为了避免线程同时重建，每隔threadRenewalDelay（单位：ms）重建一个线程。默认值为1000，设置为负则不重建
22 
23 6. Connector是Tomcat接收请求的入口，每个Connector有自己专属的监听端口；Connector有两种：HTTP Connector和AJP Connector；常用属性：
24     port：Connector接收请求的端口
25     protocol：Connector使用的协议（HTTP/1.1或AJP/1.3）
26 　　connectionTimeout：每个请求的最长连接时间（单位：ms）
27 　　redirectPort：处理http请求时，收到一个SSL传输请求，该SSL传输请求将转移到此端口处理
28 　　executor：指定线程池，如果没设置executor，可在Connector标签内设置maxThreads（默认200）、minSpareThreads（默认10）
29 　　acceptCount：Connector请求队列的上限。默认为100。当该Connector的请求队列超过acceptCount时，将拒绝接收请求
30 
31 7. Engine负责处理Service内的所有请求。它接收来自Connector的请求，并决定传给哪个Host来处理，Host处理完请求后，将结果返回给Engine，Engine再将结果返回给Connector。常用属性：
32 　　name：Engine的名字
33 　　defaultHost：指定默认Host。Engine接收来自Connector的请求，然后将请求传递给defaultHost，defaultHost 负责处理请求
34 　　className：指定实现org.apache.catalina. Engine接口的类，默认值为org.apache.catalina.core. StandardEngine
35 　　backgroundProcessorDelay：Engine及其部分子组件（Host、Context）调用backgroundProcessor方法的时间间隔。若为负，将不调用backgroundProcessor。backgroundProcessorDelay的默认值为10。 Tomcat启动后，Engine、Host、Context会启动一个后台线程，定期调用backgroundProcessor方法。backgroundProcessor方法主要用于重新加载Web应用程序的类文件和资源、扫描Session过期。
36    jvmRoute：Tomcat集群节点的id。部署Tomcat集群时会用到该属性，Service内必须包含一个Engine组件；Service包含一个或多个Connector组件，Service内的Connector共享一个Engine
37 
38 8. Host
39         Host负责管理一个或多个Web项目，常用属性：
40         name：Host的名字
41         appBase：存放Web项目的目录（绝对路径、相对路径均可）
42         unpackWARs：当appBase下有WAR格式的项目时，是否将其解压（解成目录结构的Web项目）。设成false，则直接从WAR文件运行Web项目
43         autoDeploy：是否开启自动部署。设为true，Tomcat检测到appBase有新添加的Web项目时，会自动将其部署
44         startStopThreads：线程池内的线程数量。Tomcat启动时，Host提供一个线程池，用于部署Web项目，startStopThreads为0，并行线程数=系统CPU核数；startStopThreads为负数，并行线程数=系统CPU核数+startStopThreads，如果（系统CPU核数+startStopThreads）小于1，并行线程数设为1；startStopThreads为正数，并行线程数= startStopThreads，startStopThreads默认值为1
45         startStopThreads为默认值时，Host只提供一个线程，用于部署Host下的所有Web项目。如果Host下的Web项目较多，由于只有一个线程负责部署这些项目，因此这些项目将依次部署，最终导致Tomcat的启动时间较长。此时，修改startStopThreads值，增加Host部署Web项目的并行线程数，可降低Tomcat的启动时间
46 9. Context
47    Context代表一个运行在Host上的Web项目。一个Host上可以有多个Context。将一个Web项目（D:\MyApp）添加到Tomcat，在Host标签内，添加Context标签
48 <Context path="" docBase="D:\MyApp"  reloadable="true" crossContext="true">
49 </Context>
50   常用属性：
51    path：该Web项目的URL入口。path设置为””，输入http://localhost:8080即可访问MyApp；path设置为”/test/MyApp”，输入http://localhost:8080/test/MyApp才能访问MyApp
52    docBase：Web项目的路径，绝对路径、相对路径均可（相对路径是相对于CATALINA_HOME\webapps）
53    reloadable：设置为true，Tomcat会自动监控Web项目的/WEB-INF/classes/和/WEB-INF/lib变化，当检测到变化时，会重新部署Web项目。reloadable默认值为false。通常项目开发过程中设为true，项目发布的则设为false
54    crossContext：设置为true，该Web项目的Session信息可以共享给同一host下的其他Web项目。默认为false
55 
56 10. Cluster
57         Tomcat集群配置。
58 11. Realm可以理解为包含用户、密码、角色的”数据库”。Tomcat定义了多种Realm实现：JDBC Database Realm、DataSource Database Realm、JNDI Directory Realm、UserDatabase Realm等
59 12. Valve可以理解为Tomcat的拦截器，而我们常用filter为项目内的拦截器。Valve可以用于Tomcat的日志、权限等。Valve可嵌在Engine、Host、Context内。
```

   **下面看一个server.xml配置实例：**

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) server.xml配置实例

##  Tomcat 日志概述

​    **日志分为两种，系统日志和控制台日志。**
​    系统日志主要包含运行中日志和访问日志，分为5类：**catalina、localhost、manager、localhost_access、host-manager**，在logging.properties文件中进行配置。控制台日志包含了catalina日志，另外包含了程序输出的日志（System打印，Console），可以将其日志配置输出到文件catalina.out中。

### 日志级别分为如下 7 种

```
    SEVERE (highestvalue) > WARNING > INFO > CONFIG > FINE > FINER > FINEST (lowest value)
```

### 系统日志

```
1     1、catalina日志
2         Catalina引擎的日志文件，文件名catalina.日期.log
3     2、localhost日志
4         Tomcat下内部代码抛出异常的日志，文件名localhost.日期.log
5     3、localhost_access
6         默认 tomcat 不记录访问日志，server.xml文件中配置Valve可以使 tomcat 记录访问日志
7     4、manager、host-manager
8         Tomcat下默认manager应用日志
```

### 控制台日志

​    在Linux系统中，Tomcat 启动后默认将很多信息都写入到 catalina.out 文件中，我们可以通过tail  -f  catalina.out 来跟踪Tomcat 和相关应用运行的情况。 在windows下，我们使用startup.bat启动Tomcat以后，会发现catalina日志与Linux记录的内容有很大区别，大多信息只输出到屏幕而没有记录到catalina.out里面。，可以通过设置来做到这一点。

## 总结

​     在这里我们了解了Tomcat的基本框架和运行的机制，以及相关文件的配置和日志的配置，对于Tomcat的源码如果我们仔细研读的话一定会受益匪浅的。

https://blog.csdn.net/w1992wishes/article/details/79242797