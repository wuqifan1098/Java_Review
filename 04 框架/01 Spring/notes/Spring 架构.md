# 面试题

## 1. Spring结构 （涂鸦智能，海康）

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/Spring%20constructure.png)

- **Spring Core**：核心容器提供 Spring 框架的基本功能。核心容器的主要组件是 `BeanFactory`，它是工厂模式的实现。`BeanFactory` 使用*控制反转*（IOC） 模式将应用程序的配置和依赖性规范与实际的应用程序代码分开。
- **Spring Context**：Spring 上下文是一个配置文件，向 Spring 框架提供上下文信息。Spring 上下文包括企业服务，例如 JNDI、EJB、电子邮件、国际化、校验和调度功能。
- **Spring AOP**：通过配置管理特性，Spring AOP 模块直接将面向方面的编程功能集成到了 Spring 框架中。所以，可以很容易地使 Spring 框架管理的任何对象支持 AOP。Spring AOP 模块为基于 Spring 的应用程序中的对象提供了事务管理服务。通过使用 Spring AOP，不用依赖 EJB 组件，就可以将声明性事务管理集成到应用程序中。
- **Spring DAO**：JDBC DAO 抽象层提供了有意义的异常层次结构，可用该结构来管理异常处理和不同数据库供应商抛出的错误消息。异常层次结构简化了错误处理，并且极大地降低了需要编写的异常代码数量（例如打开和关闭连接）。Spring DAO 的面向 JDBC 的异常遵从通用的 DAO 异常层次结构。
- **Spring ORM**：Spring 框架插入了若干个 ORM 框架，从而提供了 ORM 的对象关系工具，其中包括 JDO、Hibernate 和 iBatis SQL Map。所有这些都遵从 Spring 的通用事务和 DAO 异常层次结构。
- **Spring Web 模块**：Web 上下文模块建立在应用程序上下文模块之上，为基于 Web 的应用程序提供了上下文。所以，Spring 框架支持与 Jakarta Struts 的集成。Web 模块还简化了处理多部分请求以及将请求参数绑定到域对象的工作。
- **Spring MVC 框架**：MVC 框架是一个全功能的构建 Web 应用程序的 MVC 实现。通过策略接口，MVC 框架变成为高度可配置的，MVC 容纳了大量视图技术，其中包括 JSP、Velocity、Tiles、iText 和 POI。

## Spring的架构理解

### 一、前言

​    在Spring之前使用的EJB框架太庞大和重量级了，开发成本很高，由此spring应运而生。关于Spring，学过java的人基本上都会慢慢接触到，并且在面试的时候也是经常遇到的，因为这个技术极大地方便了我们的开发和部署，并且由此衍生出来的框架和思想在很多地方都有使用，比如Spring mvc，Spring boot，Spring cloud等等框架以及IoC和AOP这两个Spring最本质的思想。可以说一切的本质都是为了高内聚，松耦合，将不同的事物之间尽量的分割清楚，通过某一种额外的比较好修改的文档来记录这两者的联系，然后通过框架来将两者在联系起来，这样做看似麻烦实则便于我们扩展、维护和变更，非常的方便，并且把冗余的代码动态的装饰接入点，更加方便了代码的扩展、可维护性和可读性，只需要一个代理即可。在这两种思想之上，spring又构建了一些列的实现方式，通过IoC的Beans组件，以及Context这个让Beans运行的上下文，以及Core这个Beans的常用工具，使得一切都以Beans为核心，搭建出了最本质的框架，然后由这三者支撑了更加顶层的、复杂的技术，极大地方便了框架的扩展和维护。这也是spring长盛不衰的原因了。

### 二、Spring的框架

​    Spring是一个开源的轻量级Java SE/Java EE开发**应用框架**，其目的是用于简化企业级应用程序开发。在传统应用程序开发中，一个完整的应用是由**一组相互协作的对象**组成。所以开发一个应用除了要开发业务逻辑之外，最多的是关注如何使这些对象协作来完成所需功能，而且要低耦合、高内聚。业务逻辑开发是不可避免的，那如果有个框架出来***帮我们来创建对象及管理这些对象之间的依赖关系。***虽然“抽象工厂、工厂方法设计模式”可以帮我们创建对象，“生成器模式”可以帮我们处理对象间的依赖关系，但是这些又需要我们创建另一些工厂类、生成器类，另外管理这些类，增加了我们的负担，如果能通过配置方式来创建对象，管理对象之间依赖关系，我们不需要通过工厂和生成器来创建及管理对象之间的依赖关系，这样我们就能减少许多工作，加速开发，能节省出很多时间来干其他事。Spring框架刚出来时主要就是来完成这个功能。
​    Spring框架除了帮我们管理对象及其依赖关系，还提供像通用日志记录、性能统计、安全控制、异常处理等***面向切面编程***的能力，还能帮我们管理最头疼的数据库事务，它本身提供了一套简单的JDBC访问实现，还能与第三方数据库访问框架集成（如Hibernate、JPA），与各种Java EE技术整合（如Java Mail、任务调度等等），提供一套自己的web层框架Spring MVC、而且还能非常简单的与第三方web框架集成。从这里我们可以认为Spring是一个***超级粘合平台***，除了自己提供功能外，还提供粘合其他技术和框架的能力，从而使我们可以更自由的选择到底使用什么技术进行开发。而且不管是JAVA SE（C/S架构）应用程序还是JAVA EE（B/S架构）应用程序都可以使用这个平台进行开发。

####  2.1、框架的本质

​    Spring 框架中的核心组件只有三个：***Core、Context 和 Beans。***它们构建起了整个 Spring 的骨骼架构。没有它们就不可能有 AOP、Web 等上层的特性功能。如果在它们三个中选出核心的话，那就非 Beans 组件莫属了，其实 Spring 就是面向 Bean 的编程（BOP,Bean Oriented Programming），Bean 在 Spring 中才是真正的主角。Bean 在 Spring 中作用就像 Object 对 OOP 的意义一样，没有对象的概念就像没有面向对象编程，Spring 中没有 Bean 也就没有 Spring 存在的意义。就像一次演出舞台都准备好了但是却没有演员一样。为什么要 Bean 这种角色 Bean 或者为何在 Spring 如此重要，这由 Spring 框架的设计目标决定，Spring 为何如此流行，我们用 Spring 的原因是什么，想想会发现原来 Spring 解决了一个非常关键的问题，它可以让你把对象之间的依赖关系转而用配置文件来管理，也就是依赖注入机制。而这个注入关系在一个叫 IoC 容器中管理，那 IoC 容器就是被 Bean 包裹的对象。Spring 正是通过把对象包装在 Bean 中而达到对这些对象的管理以及一些列额外操作的目的。它这种设计策略完全类似于 Java 实现 OOP 的设计理念，当然了 Java 本身的设计要比 Spring 复杂太多太多，但是都是构建一个数据结构，然后根据这个数据结构设计他的生存环境，并让它在这个环境中按照一定的规律在不停的运动，在它们的不停运动中设计一系列与环境或者与其他个体完成信息交换。这样想来我们用到的其他框架都是大慨类似的设计理念。

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181119102630992-1039511650.gif)

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181119141338889-768448597.png)

####  2.2、核心组件的协同工作

​    ***把 Beans 比作一场演出中的演员的话，那 Context 就是这场演出的舞台背景，而 Core 应该就是演出的道具了。只有它们在一起才能具备演出一场好戏的最基本条件。当然有最基本的条件还不能使这场演出脱颖而出，还要表演的节目足够的精彩，这些节目就是 Spring 能提供的特色功能了。***我们知道 Bean 包装的是 Object，而 Object 必然有数据，如何给这些数据提供生存环境就是 Context 要解决的问题，对 Context 来说就是要发现每个 Bean 之间的关系，为它们建立这种关系并且要维护好这种关系。所以 Context 就是一个 Bean 关系的集合，这个关系集合又叫 Ioc 容器，一旦建立起这个 Ioc 容器后 Spring 就可以工作了。那 Core 组件又有什么用武之地呢？其实 Core 就是发现、建立和维护每个 Bean 之间的关系所需要的一些列的工具，从这个角度看来，Core 这个组件叫 Util 更能理解。

 ![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181119103618503-838184056.gif)

####  2.3、Bean 组件

   Bean 组件在 Spring 的 org.springframework.beans 包下。***这个包下的所有类主要解决了三件事：Bean 的定义、Bean 的创建以及对 Bean 的解析。***对 Spring 的使用者来说唯一需要关心的就是 Bean 的创建，其他两个由 Spring 在内部帮你完成了，对你来说是透明的。Spring Bean 的创建时典型的工厂模式，**它的顶级接口是 BeanFactory**，下图是这个工厂的继承层次关系：

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181119103717048-782827587.png)

​     ***BeanFactory 有三个子类：ListableBeanFactory、HierarchicalBeanFactory 和 AutowireCapableBeanFactory。***但是我们可以发现最终的默认实现类是 DefaultListableBeanFactory，实现了所有的接口。那为何要定义这么多层次的接口呢？查阅这些接口的源码和说明发现，每个接口都有使用的场合，它主要是为了区分在 Spring 内部对象的传递和转化过程中，对对象的数据访问所做的限制。例如 ListableBeanFactory 接口表示这些 Bean 是可列表的，而 HierarchicalBeanFactory 表示的这些 Bean 是有继承关系的，也就是每个 Bean 有可能有父 Bean。AutowireCapableBeanFactory 接口定义 Bean 的自动装配规则。***这四个接口共同定义了 Bean 的集合、Bean 之间的关系以及 Bean 行为。***Bean 的定义主要有 BeanDefinition 描述。

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181119140518606-1310179509.png)

#### Bean的作用域

  **Spring定义了多种Bean作用域，可以基于这些作用域创建bean，包括：**

```
`单例（Singleton）：在整个应用中，只创建bean的一个实例。``原型（Prototype）：每次注入或者通过Spring应用上下文获取的时候，都会创建一个新的bean实例。``会话（Session）：在Web应用中，为每个会话创建一个bean实例。``请求（Rquest）：在Web应用中，为每个请求创建一个bean实例。`
```

####  声明Bean

```
`@Component 组件，没有明确的角色``@Service 在业务逻辑层使用``@Repository 在数据访问层使用``@Controller 在展现层使用(MVC -> Spring MVC)使用``在这里，可以指定bean的id名：Component(“yourBeanName”)``同时，Spring支持将@Named作为@Component注解的替代方案。<br>两者之间有一些细微的差异，但是在大多数场景中，它们是可以互相替换的。`
```

####  2.4、Context 组件

​    ***Context 作为 Spring 的 Ioc 容器，基本上整合了 Spring 的大部分功能，或者说是大部分功能的基础。***Context 在 Spring 的 org.springframework.context 包下， Context 组件实际上就是给 Spring 提供一个运行时的环境，用以保存各个对象的状态。下面看一下这个环境是如何构建的。ApplicationContext 是 Context 的顶级父类，除了能标识一个应用环境的基本信息外，还继承了五个接口，这五个接口主要是扩展了 Context 的功能。

 ![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181119104140806-1002090366.png)

​    **ApplicationContext 继承了 BeanFactory，这也说明了 Spring 容器中运行的主体对象是 Beans，另外 ApplicationContext 继承了 ResourceLoader 接口，使得 ApplicationContext 可以访问到任何外部资源。**
​    **ApplicationContext 的子类主要包含两个方面：**

```
1  ConfigurableApplicationContext 表示该 Context 是可修改的，也就是在构建 Context 中用户可以动态添加或修改已有的配置信息，它下面又有多个子类，其中经常使用的是可更新的 Context，即 AbstractRefreshableApplicationContext 类。
2  WebApplicationContext 顾名思义，就是为 web 准备的 Context ，可以直接访问到 ServletContext，通常情况下，这个接口使用的少。
```

   **再往下分就是按照构建 Context 的文件类型，接着就是访问 Context 的方式。这样一级一级构成了完整的 Context 等级层次。**
   **总体来说 ApplicationContext 必须要完成以下几件事：**

```
1   标识一个应用环境
2   利用 BeanFactory 创建 Bean 对象
3   保存对象关系表
4   能够捕获各种事件
```

####  **2.5、Core 组件**

​    Core 组件作为 Spring 的核心组件，其中包含了很多的关键类，***其中一个重要组成部分就是定义了资源的访问方式。***这种***把所有资源都抽象成一个接口***的方式很值得在以后的设计中拿来学习。下面就重要看一下这个部分在 Spring 的作用。

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181119105104038-1109969312.png)
    Resource 接口封装了各种可能的资源类型，也就是对使用者来说屏蔽了文件类型的不同。对资源的提供者来说，如何***把资源包装起来交给其他人用***这也是一个问题，我们看到 Resource 接口继承了 InputStreamSource 接口，这个接口中有个 getInputStream 方法，返回的是 InputStream 类。这样所有的资源都被可以通过 InputStream 这个类来获取，所以也屏蔽了资源的提供者。另外还有一个问题就是加载资源的问题，也就是***资******源的加载者要统一***，这个任务是由 ResourceLoader 接口完成，它屏蔽了所有的资源加载者的差异，只需要实现这个接口就可以加载所有的资源，默认实现是 DefaultResourceLoader。

   **Context 和 Resource 建立关系：Context 是把资源的加载、解析和描述工作委托给了 ResourcePatternResolver 类来完成**，相当于一个接头人，把资源的加载、解析和资源的定义整合在一起便于其他组件使用。Core 组件中还有很多类似的方式。

####  2.6、IoC 容器如何工作

  前面介绍了 Core 组件、Bean 组件和 Context 组件的结构与相互关系，下面这里从使用者角度看一下它们是如何运行的，以及我们如何让 Spring 完成各种功能。

####  如何创建 BeanFactory 工厂

   IoC容器实际上就是Context组件结合其他两个组件共同构建了一个 Bean 关系网，构建的入口就在 AbstractApplicationContext 类的 refresh 方法中。这个方法的代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public void refresh() throws BeansException, IllegalStateException {
 2     synchronized (this.startupShutdownMonitor) {
 3         // Prepare this context for refreshing.
 4         prepareRefresh();
 5         // Tell the subclass to refresh the internal bean factory.
 6         ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
 7         // Prepare the bean factory for use in this context.
 8         prepareBeanFactory(beanFactory);
 9         try {
10             // Allows post-processing of the bean factory in context subclasses.
11             postProcessBeanFactory(beanFactory);
12             // Invoke factory processors registered as beans in the context.
13             invokeBeanFactoryPostProcessors(beanFactory);
14             // Register bean processors that intercept bean creation.
15             registerBeanPostProcessors(beanFactory);
16             // Initialize message source for this context.
17             initMessageSource();
18             // Initialize event multicaster for this context.
19             initApplicationEventMulticaster();
20             // Initialize other special beans in specific context subclasses.
21             onRefresh();
22             // Check for listener beans and register them.
23             registerListeners();
24             // Instantiate all remaining (non-lazy-init) singletons.
25             finishBeanFactoryInitialization(beanFactory);
26             // Last step: publish corresponding event.
27             finishRefresh();
28         }
29         catch (BeansException ex) {
30             // Destroy already created singletons to avoid dangling resources.
31             destroyBeans();
32             // Reset 'active' flag.
33             cancelRefresh(ex);
34             // Propagate exception to caller.
35             throw ex;
36         }
37     }
38 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

​    **这个方法就是构建整个 Ioc 容器过程的完整的代码，了解里面的每一行代码基本上就了解大部分 Spring 的原理和功能了。主要包含这样几个步骤：**

```
1  构建 BeanFactory，以便于产生所需的“演员”
2  注册可能感兴趣的事件
3  创建 Bean 实例对象
4  触发被监听的事件
```

​     创建和配置 BeanFactory，这里 refresh 也就是刷新配置，前面介绍了 Context 有可更新的子类，这里正是实现这个功能，***当 BeanFactory 已存在是就更新，如果没有就新创建。***下面是更新 BeanFactory 的方法代码：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 protected final void refreshBeanFactory() throws BeansException {
 2     if (hasBeanFactory()) {
 3         destroyBeans();
 4         closeBeanFactory();
 5     }
 6     try {
 7         DefaultListableBeanFactory beanFactory = createBeanFactory();
 8         beanFactory.setSerializationId(getId());
 9         customizeBeanFactory(beanFactory);
10         loadBeanDefinitions(beanFactory);
11         synchronized (this.beanFactoryMonitor) {
12             this.beanFactory = beanFactory;
13         }
14     }
15     catch (IOException ex) {
16         throw new ApplicationContextException(
17             "I/O error parsing bean definition source for "
18             + getDisplayName(), ex);
19     }
20 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

​     这个方法实现了 AbstractApplicationContext 的抽象方法 refreshBeanFactory，这段代码清楚的说明了 BeanFactory 的创建过程。注意 BeanFactory 对象的类型的变化，前面介绍了它有很多子类，在什么情况下使用不同的子类这非常关键。BeanFactory 的原始对象是***DefaultListableBeanFactory***，设计到后面对这个对象的多种操作。除了 BeanFactory 相关的类外，还发现了与 Bean 的 register 相关。这在 refreshBeanFactory 方法中有一行 loadBeanDefinitions(beanFactory) 将找到答案，这个方法将开始加载、解析 Bean 的定义，也就是把用户定义的数据结构转化为 IoC 容器中的特定数据结构。
![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181119110418619-502987707.png)

***创建 BeanFactory 时序图:***

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181119110511989-629063264.png)

***Bean 的解析和登记流程时序图:***
![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181119110701356-2042487579.png)

​    创建好 BeanFactory 后，接下去添加一些 Spring 本身需要的一些工具类，这个操作在 AbstractApplicationContext 的 prepareBeanFactory 方法完成。AbstractApplicationContext 中接下来的三行代码对 Spring 的功能扩展性起了至关重要的作用。***前两行主要是让我们可以对已经构建的 BeanFactory 的配置做修改，后面一行就是让我们对以后再创建 Bean 的实例对象时添加一些自定义的操作。***所以它们都是扩展了 Spring 的功能，所以我们要学习使用 Spring 必须对这一部分搞清楚。

####  如何创建 Bean 实例并构建 Bean 的关系网

​    下面是 Bean 的实例化代码，是从 finishBeanFactoryInitialization 方法开始的。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 protected void finishBeanFactoryInitialization(
 2     ConfigurableListableBeanFactory beanFactory) {
 3  
 4     // Stop using the temporary ClassLoader for type matching.
 5     beanFactory.setTempClassLoader(null);
 6  
 7     // Allow for caching all bean definition metadata, not expecting further changes.
 8     beanFactory.freezeConfiguration();
 9  
10     // Instantiate all remaining (non-lazy-init) singletons.
11     beanFactory.preInstantiateSingletons();
12 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

​    从上面代码中可以发现 Bean 的实例化是在 BeanFactory 中发生的。preInstantiateSingletons 方法的代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public void preInstantiateSingletons() throws BeansException {
 2     if (this.logger.isInfoEnabled()) {
 3         this.logger.info("Pre-instantiating singletons in " + this);
 4     }
 5     synchronized (this.beanDefinitionMap) {
 6         for (String beanName : this.beanDefinitionNames) {
 7             RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
 8             if (!bd.isAbstract() && bd.isSingleton()
 9                 && !bd.isLazyInit()) {
10                 if (isFactoryBean(beanName)) {
11                     final FactoryBean factory =
12                         (FactoryBean) getBean(FACTORY_BEAN_PREFIX+ beanName);
13                     boolean isEagerInit;
14                     if (System.getSecurityManager() != null
15                         && factory instanceof SmartFactoryBean) {
16                         isEagerInit = AccessController.doPrivileged(
17                             new PrivilegedAction<Boolean>() {
18                             public Boolean run() {
19                                 return ((SmartFactoryBean) factory).isEagerInit();
20                             }
21                         }, getAccessControlContext());
22                     }
23                     else {
24                         isEagerInit = factory instanceof SmartFactoryBean
25                             && ((SmartFactoryBean) factory).isEagerInit();
26                     }
27                     if (isEagerInit) {
28                         getBean(beanName);
29                     }
30                 }
31                 else {
32                     getBean(beanName);
33                 }
34             }
35         }
36     }
37 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

   ***这里出现了一个非常重要的 Bean—— FactoryBean，可以说 Spring 一大半的扩展的功能都与这个 Bean 有关，这是个特殊的 Bean 是一个工厂 Bean，可以产生 Bean 的 Bean，这里的产生 Bean 是指 Bean 的实例，***如果一个类继承 FactoryBean 用户只要实现它的 getObject 方法，就可以自己定义产生实例对象的方法。然而在 Spring 内部这个 Bean 的实例对象是 FactoryBean，通过调用这个对象的 getObject 方法就能获取用户自定义产生的对象，从而为 Spring 提供了很好的扩展性。Spring 获取 FactoryBean 本身的对象是在前面加上 & 来完成的。如何创建 Bean 的实例对象以及如何构建 Bean 实例对象之间的关联关系式 Spring 中的一个核心关键，下面是这个过程的流程图。

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181119111937668-2011648357.png)

   ***如果是普通的 Bean 就直接创建它的实例，是通过调用 getBean 方法。下面是创建 Bean 实例的时序图：***
![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181119112746161-607139309.png)

  还有一个非常重要的部分就是建立 Bean 对象实例之间的关系，这也是 Spring 框架的核心竞争力，何时、如何建立它们之间的关系：

 ![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181119112832223-318803665.png)

####  IoC 容器的扩展点

​    现在还有一个问题就是如何让这些 Bean 对象有一定的扩展性，就是可以加入用户的一些操作。那么有哪些扩展点呢？ Spring 又是如何调用到这些扩展点的？对 Spring 的 Ioc 容器来说，主要有这么几个。BeanFactoryPostProcessor， BeanPostProcessor。它们分别是在构建 BeanFactory 和构建 Bean 对象时调用。还有就是 InitializingBean 和 DisposableBean，他们分别是在 Bean 实例创建和销毁时被调用。用户可以实现这些接口中定义的方法，Spring 就会在适当的时候调用他们。还有一个是 FactoryBean 他是个特殊的 Bean，这个 Bean 可以被用户更多的控制。这些扩展点通常也是我们使用 Spring 来完成我们特定任务的地方，如何精通 Spring 就看你有没有掌握好 Spring 有哪些扩展点，并且如何使用他们，要知道如何使用他们就必须了解他们内在的机理。可以用下面一个比喻来解释。
​    我们把 Ioc 容器比作一个箱子，这个箱子里有若干个球的模子，可以用这些模子来造很多种不同的球，还有一个造这些球模的机器，这个机器可以产生球模。那么他们的对应关系就是：BeanFactory 是那个造球模的机器，球模就是 Bean，而球模造出来的球就是 Bean 的实例。那前面所说的几个扩展点又在什么地方呢？ BeanFactoryPostProcessor 对应到当造球模被造出来时，你将有机会可以对其做出适当的修正，也就是他可以帮你修改球模。而 InitializingBean 和 DisposableBean 是在球模造球的开始和结束阶段，你可以完成一些预备和扫尾工作。BeanPostProcessor 就可以让你对球模造出来的球做出适当的修正。最后还有一个 FactoryBean，它可是一个神奇的球模。这个球模不是预先就定型了，而是由你来给他确定它的形状，既然你可以确定这个球模型的形状，当然他造出来的球肯定就是你想要的球了，这样在这个箱子里你可以发现所有你想要的球。

#### Ioc 容器如何使用

​    我们使用 Spring 必须要首先构建 Ioc 容器，没有它 Spring 无法工作，***ApplicatonContext.xml 就是 IoC 容器的默认配置文件，***Spring 的所有特性功能都是基于这个 Ioc 容器工作的，比如AOP。IoC 它实际上就是为你构建了一个魔方，Spring 为你搭好了骨骼架构，这个魔方到底能变出什么好的东西出来，这必须要有你的参与。那我们怎么参与？这就是前面说的要了解 Spring 中有哪些扩展点，我们通过实现那些扩展点来改变 Spring 的通用行为。至于如何实现扩展点来得到我们想要的个性结果，Spring 中有很多例子，其中 AOP 的实现就是 Spring 本身实现了其扩展点来达到了它想要的特性功能。

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181119113235986-1683466028.png)

​    通过从 Spring 的几个核心组件入手，试图找出构建 Spring 框架的骨骼架构，进而分析 Spring 在设计时的一些设计理念，是否从中找出一些好的设计思想，对我们以后程序设计能提供一些思路。接着再详细分析了 Spring 中是如何实现这些理念的，以及在设计模式上是如何使用的。通过分析 Spring 给我一个很大的启示就是这套设计理念其实对我们有很强的借鉴意义，它通过抽象复杂多变的对象，进一步做规范，然后根据它定义的这套规范设计出一个容器，容器中构建它们的复杂关系，其实现在有很多情况都可以用这种类似的处理方法。

### 三、Spring的特性

####  **3.1、一些概念**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1   应用程序：是能完成我们所需要功能的成品，比如购物网站、OA系统。
2   框架：是能完成一定功能的半成品，比如我们可以使用框架进行购物网站开发；框架做一部分功能，我们自己做一部分功能，这样应用程序就创建出来了。而且框架规定了你在开发应用程序时的整体架构，提供了一些基础功能，还规定了类和对象的如何创建、如何协作等，从而简化我们开发，让我们专注于业务逻辑开发。
3   非侵入式设计：从框架角度可以这样理解，无需继承框架提供的类，这种设计就可以看作是非侵入式设计，如果继承了这些框架类，就是侵入设计，如果以后想更换框架，之前写过的代码几乎无法重用，如果非侵入式设计则之前写过的代码仍然可以继续使用。
4   轻量级及重量级：轻量级是相对于重量级而言的，轻量级一般就是非入侵性的、所依赖的东西非常少、资源占用非常少、部署简单等等，其实就是比较容易使用，而重量级正好相反。
5   POJO：POJO（Plain Old Java Objects）简单的Java对象，它可以包含业务逻辑或持久化逻辑，但不担当任何特殊角色且不继承或不实现任何其它Java框架的类或接口。
6   容器：在日常生活中容器就是一种盛放东西的器具，从程序设计角度看就是装对象的的对象，因为存在放入、拿出等操作，所以容器还要管理对象的生命周期。
7   控制反转：即Inversion of Control，缩写为IoC，控制反转还有一个名字叫做依赖注入（Dependency Injection），就是由容器控制程序之间的关系，而非传统实现中，由程序代码直接操控。
8   Bean：一般指容器管理的对象，在Spring中指Spring IoC容器管理的对象。
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####  3.2、Spring的优点

```
`非常轻量级的容器：以集中的、自动化的方式进行应用程序对象创建和装配，负责对象创建和装配，管理对象生命周期，能组合成复杂的应用程序。Spring容器是非侵入式的（不需要依赖任何Spring特定类），而且完全采用POJOs进行开发，使应用程序更容易测试、更容易管理。而且核心JAR包非常小，Spring3.0.5不到1M，而且不需要依赖任何应用服务器，可以部署在任何环境（Java SE或Java EE）。``AOP：面向切面编程提供从另一个角度来考虑程序结构以完善面向对象编程（OOP），即可以通过在编译期间、装载期间或运行期间实现在不修改源代码的情况下给程序动态添加功能的一种技术。通俗点说就是把可重用的功能提取出来，然后将这些通用功能在合适的时候织入到应用程序中；比如安全，日志记录，这些都是通用的功能，我们可以把它们提取出来，然后在程序执行的合适地方织入这些代码并执行它们，从而完成需要的功能并复用了这些功能。``简单的数据库事务管理：在使用数据库的应用程序当中，自己管理数据库事务是一项很让人头疼的事，而且很容易出现错误，Spring支持可插入的事务管理支持，而且无需JavaEE环境支持，通过Spring管理事务可以把我们从事务管理中解放出来来专注业务逻辑。``JDBC抽象及ORM框架支持：Spring使JDBC更加容易使用；提供DAO（数据访问对象）支持，非常方便集成第三方ORM框架，比如Hibernate，Mybatis等；并且完全支持Spring事务和使用Spring提供的一致的异常体系。``灵活的Web层支持：Spring本身提供一套非常强大的MVC框架，而且可以非常容易的与第三方MVC框架集成，比如Struts等。``简化各种技术集成：提供对Java Mail、任务调度、JMX、JMS、JNDI、EJB、动态语言、远程访问、Web Service等的集成。`
```

### 四、总结

   **在Spring中，我们知道了其中的本质——‘道’，然后在‘术’上进行实现，并且发现spring中提供了很多的扩展，从此对spring有了更深的理解，同时随着版本的递增，其中的各种功能也进行了很多的优化和提升，我们最好多阅读一下spring的源代码，从中明白很多程序背后的机理。**

https://www.cnblogs.com/zyrblog/p/9944290.html