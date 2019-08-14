# 面试题

## 1. 谈谈IOC的理解（华数）

IOC就是控制反转，一种思想。**借助Spring容器实现具有依赖关系的对象之间的解耦。**

软件系统是由很多N个对象组成的，各个对象之间相互合作，实现业务逻辑。比如说A依赖于B，那么A在初始化的时候或运行到某一点的时候，需要自己主动创建B或使用已经创建的对象B。无论是创建还是使用对象B，控制权都在自己手上。

引入IOC之后，**对象A和对象B就失去了联系，当对象A运行到需要B的时候，IOC会主动创建一个对象B注入到对象A需要的地方。**

把对象的创建、初始化、销毁等工作交给spring容器来做。由spring容器控制对象的生命周期。

传统Java程序设计，直接在内部通过new来创建对象，是主动创建依赖对象；而IOC是专门有一个容器来创建这些对象，由IOC容器来控制对象的创建。

**通俗的讲就是如果在什么地方需要一个对象，你自己不用去通过new 生成你需要的对象，而是通过spring的bean工厂为你创建这样一个对象。**

## 2. IOC的实现（华数）

DI 就是**将spring管理的对象通过 AutoWrite 注解注入到我们需要的地方**.注入过来的对象是由spring管理的.

**注入方式：**

- 接口注入
- setter
- 构造器注入

## 3. BeanFactory和ApplicationContext的区别

- BeanFactory和ApplicationContext是Spring的两大核心接口，而其中ApplicationContext是BeanFactory的子接口。它们都可以当做Spring的容器，Spring容器是生成Bean实例的工厂，并管理容器中的Bean。
- Spring的核心是容器，而容器并不唯一，框架本身就提供了很多个容器的实现，大概分为两种类型：
  一种是不常用的BeanFactory，这是**最简单的容器，只能提供基本的DI功能；**
  一种就是继承了BeanFactory后派生而来的ApplicationContext(应用上下文)，它能提供更多企业级的服务，**例如解析配置文本信息等等，这也是ApplicationContext实例对象最常见的应用场景。**
- BeanFactroy采用的是**延迟加载形式来注入Bean的**，即**只有在使用到某个Bean时(调用getBean())，才对该Bean进行加载实例化**。这样，我们就不能发现一些存在的Spring的配置问题。如果Bean的某一个属性没有注入，BeanFacotry加载后，**直至第一次使用调用getBean方法才会抛出异常。**实现 BeanFactory 最常用的 API 是 XMLBeanFactory 这里是如何通过 BeanFactory 获取一个 bean 的例子：

```java
package com.zoltanraffai;  
import org.springframework.core.io.ClassPathResource;  
import org.springframework.beans.factory.InitializingBean; 
import org.springframework.beans.factory.xml.XmlBeanFactory; 
public class HelloWorldApp{ 
   public static void main(String[] args) { 
      XmlBeanFactory factory = new XmlBeanFactory (new ClassPathResource("beans.xml")); 
      HelloWorld obj = (HelloWorld) factory.getBean("helloWorld");    
      obj.getMessage();    
   }
}

作者：日拱一兵
链接：https://juejin.im/post/5d195530f265da1bb80c4560
```

- BeanFactory和ApplicationContext都支持BeanPostProcessor、BeanFactoryPostProcessor的使用，但两者之间的区别是：**BeanFactory需要手动注册，而ApplicationContext则是自动注册。**

# 引言

先看下最基本的启动 Spring 容器的例子：

```java
public static void main(String[] args) {
    ApplicationContext context = new ClassPathXmlApplicationContext("classpath:applicationfile.xml");
}
```

以上代码就可以利用配置文件来启动一个 Spring 容器了，请使用 maven 的小伙伴直接在 dependencies 中加上以下依赖即可，个人比较反对那些不知道要添加什么依赖，然后把 Spring 的所有相关的东西都加进来的方式。

```java
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-context</artifactId>
  <version>4.3.11.RELEASE</version>
</dependency>
```

> spring-context 会自动将 spring-core、spring-beans、spring-aop、spring-expression 这几个基础 jar 包带进来。

`ApplicationContext context = new ClassPathXmlApplicationContext(...)` 其实很好理解，从名字上就可以猜出一二，就是在 ClassPath 中寻找 xml 配置文件，根据 xml 文件内容来构建 ApplicationContext。当然，除了 ClassPathXmlApplicationContext 以外，我们也还有其他构建 ApplicationContext 的方案可供选择，我们先来看看大体的继承结构是怎么样的：

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/20190528210235.png)

ClassPathXmlApplicationContext 兜兜转转了好久才到 ApplicationContext 接口，同样的，我们也可以使用绿颜色的 **FileSystemXmlApplicationContext** 和 **AnnotationConfigApplicationContext** 这两个类。

**FileSystemXmlApplicationContext** 的构造函数需要一个 xml 配置文件在系统中的路径，其他和 ClassPathXmlApplicationContext 基本上一样。

**AnnotationConfigApplicationContext** 是基于注解来使用的，它不需要配置文件，采用 java 配置类和各种注解来配置，是比较简单的方式，也是大势所趋吧。

BeanFactory 简介

BeanFactory，从名字上也很好理解，生产 bean 的工厂，它负责生产和管理各个 bean 实例。

初学者可别以为我之前说那么多和 BeanFactory 无关，前面说的 ApplicationContext 其实就是一个 BeanFactory。我们来看下和 BeanFactory 接口相关的主要的继承结构：

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/20190528210412.png)

# 1.什么是IOC

IoC(Inversion of Control)，意为控制反转，不是什么技术，而是一种设计思想。Ioc意味着**将你设计好的对象交给容器控制，而不是传统的在你的对象内部直接控制**。

- **谁控制谁，控制什么**：传统Java SE程序设计，我们直接在对象内部通过new进行创建对象，是**程序主动去创建依赖对象**；而IoC是有**专门一个容器来创建这些对象**，即由Ioc容器来控制对象的创建；谁控制谁？当然是IoC 容器控制了对象；控制什么？那就是主要控制了外部资源获取（不只是对象包括比如文件等）。
- **为何是反转，哪些方面反转了**：有反转就有正转，传统应用程序是由我们**自己在对象中主动控制去直接获取依赖对象**，也就是正转；而反转则是**由容器来帮忙创建及注入依赖对象**；为何是反转？因为由容器帮我们查找及注入依赖对象，对象只是被动的接受依赖对象，所以是反转；哪些方面反转了？依赖对象的获取被反转了

**简单来说**

> 正转：比如有一个类，在类里面有方法（不是静态的方法），调用类里面的方法，创建类的对象，使用对象调用方法，创建类对象的过程，需要new出来对象
>
> 反转：把对象的创建不是通过new方式实现，而是交给Spring配置创建类对象

# 2.IoC和DI

**DI—Dependency Injection，即“依赖注入”**：组件之间依赖关系由容器在运行期决定，形象的说，即由容器动态的将某个依赖关系注入到组件之中。

**DI(Dependecy Inject,依赖注入)是实现控制反转的一种设计模式，依赖注入就是将实例变量传入到一个对象中去。**

理解DI的关键是：“**谁依赖谁，为什么需要依赖，谁注入谁，注入了什么**”，那我们来深入分析一下：

- **谁依赖于谁：** 当然是应用程序依赖于IoC容器；
- **为什么需要依赖：** 应用程序需要IoC容器来提供对象需要的外部资源；
- **谁注入谁：** 很明显是IoC容器注入应用程序某个对象，应用程序依赖的对象；
- **注入了什么：** 就是注入某个对象所需要的外部资源（包括对象、资源、常量数据）

##  IoC 设计支持以下功能：

1. 依赖注入
2. 依赖检查
3. 自动装配
4. 支持集合
5. 指定初始化方法和销毁方法
6. 支持回调某些方法（但是需要实现 Spring 接口，略有侵入）

其中，最重要的就是**依赖注入**，从 XML 的配置上说， 即 ref 标签。对应 Spring RuntimeBeanReference 对象。

对于 IoC 来说，最重要的就是容器。容器管理着 Bean 的生命周期，控制着 Bean 的依赖注入。

那么， Spring 如何设计容器的呢？

Spring 作者 Rod Johnson 设计了两个接口用以表示容器。

1. BeanFactory
2. ApplicationContext

BeanFactory 粗暴简单，可以理解为就是个 HashMap，Key 是 BeanName，Value 是 Bean 实例。**通常只提供注册（put），获取（get）这两个功能。我们可以称之为 “低级容器”**。

ApplicationContext 可以称之为 “高级容器”。因为他比 BeanFactory 多了更多的功能。**他继承了多个接口。因此具备了更多的功能。**

该接口定义了一个 refresh 方法，此方法是所有阅读 Spring 源码的人的最熟悉的方法，用于刷新整个容器，**即重新加载/刷新所有的 bean。**

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/Spring%20%E9%AB%98%E7%BA%A7%20%E4%BD%8E%E7%BA%A7%E5%AE%B9%E5%99%A8%20UML.jpg)

看下面的隶属 ApplicationContext 粉红色的 “高级容器”，依赖着 “低级容器”，这里说的是依赖，不是继承。**他依赖着 “低级容器” 的 getBean 功能**。而高级容器有更多的功能：**支持不同的信息源头，可以访问文件资源，支持应用事件（Observer 模式）。**

通常用户看到的就是 “高级容器”。但 BeanFactory 也非常够用。

左边灰色区域的是 “低级容器”， 只负载加载 Bean，获取 Bean。容器其他的高级功能是没有的。例如上图画的 refresh 刷新 Bean 工厂所有配置。生命周期事件回调等。

下图是 ClassPathXmlApplicationContext 的构造过程，实际就是 Spring IoC 的初始化过程。

1. 用户构造 ClassPathXmlApplicationContext（简称 CPAC）
2. CPAC 首先访问了 “抽象高级容器” 的 final 的 refresh 方法，这个方法是模板方法。所以要回调子类（低级容器）的 refreshBeanFactory 方法，这个方法的作用是使用低级容器加载所有 BeanDefinition 和 Properties 到容器中。
3. 低级容器加载成功后，高级容器开始处理一些回调，例如 Bean 后置处理器。回调 setBeanFactory 方法。或者注册监听器等，发布事件，实例化单例 Bean 等等功能。

简单说就是：

1. **低级容器 加载配置文件（从 XML，数据库，Applet）**，并解析成 BeanDefinition 到低级容器中。
2. 加载成功后，**高级容器启动高级功能**，例如接口回调，监听器，自动实例化单例，发布事件等等功能。

# BeanFactory

**BeanFactory接口：**
  是Spring bean容器的根接口，提供获取bean，是否包含bean,是否单例与原型，获取bean类型，bean 别名的方法 。它最主要的方法就是getBean(String beanName)。
  
 **BeanFactory的三个子接口：**

  * HierarchicalBeanFactory：提供父容器的访问功能

  * ListableBeanFactory：提供了批量获取Bean的方法

  * AutowireCapableBeanFactory：在BeanFactory基础上实现对已存在实例的管理

**ConfigurableBeanFactory：**

 主要单例bean的注册，生成实例，以及统计单例bean

**ConfigurableListableBeanFactory：**

 继承了上述的所有接口，增加了其他功能：比如类加载器,类型转化,属性编辑器,BeanPostProcessor,作用域,bean定义,处理bean依赖关系, bean如何销毁…

 **实现类DefaultListableBeanFactory详细介绍：**

 实现了ConfigurableListableBeanFactory，实现上述BeanFactory所有功能。它还可以注册BeanDefinition
  接口详细介绍请参考:[揭秘BeanFactory](https://blog.csdn.net/u011179993/article/details/51636742)

# ApplicationContext

![img](https:////upload-images.jianshu.io/upload_images/12234310-a14ad5a594b524fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

​                                                                        ApplicationContext结构图

![img](https:////upload-images.jianshu.io/upload_images/12234310-be1edded652cea7b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

​                                                                       ApplicationContext类结构树

|     ApplicationContext常用实现类      |                             作用                             |
| :-----------------------------------: | :----------------------------------------------------------: |
|  AnnotationConfigApplicationContext   | 从一个或多个基于java的配置类中加载上下文定义，适用于java注解的方式。 |
|    ClassPathXmlApplicationContext     | 从类路径下的一个或多个xml配置文件中加载上下文定义，适用于xml配置的方式。 |
|    FileSystemXmlApplicationContext    | 从文件系统下的一个或多个xml配置文件中加载上下文定义，也就是说系统盘符中加载xml配置文件。 |
| AnnotationConfigWebApplicationContext |            专门为web应用准备的，适用于注解方式。             |
|       XmlWebApplicationContext        | 从web应用下的一个或多个xml配置文件加载上下文定义，适用于xml配置方式。 |

Spring具有非常大的灵活性，它提供了三种主要的装配机制：

- 在XMl中进行显示配置
- 在Java中进行显示配置
- 隐式的bean发现机制和自动装配
   *组件扫描（component scanning）：Spring会自动发现应用上下文中所创建的bean。
   *自动装配（autowiring）：Spring自动满足bean之间的依赖。

（使用的优先性: 3>2>1）尽可能地使用自动配置的机制，显示配置越少越好。当必须使用显示配置bean的时候（如：有些源码不是由你来维护的，而当你需要为这些代码配置bean的时候），推荐使用类型安全比XML更加强大的JavaConfig。最后只有当你想要使用便利的XML命名空间，并且在JavaConfig中没有同样的实现时，才使用XML。

代码示例：

通过**xml文件将配置**加载到IOC容器中

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
     <!--若没写id，则默认为com.test.Man#0,#0为一个计数形式-->
    <bean id="man" class="com.test.Man"></bean>
</beans>
public class Test {
    public static void main(String[] args) {
        //加载项目中的spring配置文件到容器
        //ApplicationContext context = new ClassPathXmlApplicationContext("resouces/applicationContext.xml");
        //加载系统盘中的配置文件到容器
        ApplicationContext context = new FileSystemXmlApplicationContext("E:/Spring/applicationContext.xml");
        //从容器中获取对象实例
        Man man = context.getBean(Man.class);
        man.driveCar();
    }
}
```

通过**java注解的方式**将配置加载到IOC容器

```java
//同xml一样描述bean以及bean之间的依赖关系
@Configuration
public class ManConfig {
    @Bean
    public Man man() {
        return new Man(car());
    }
    @Bean
    public Car car() {
        return new QQCar();
    }
}
public class Test {
    public static void main(String[] args) {
        //从java注解的配置中加载配置到容器
        ApplicationContext context = new AnnotationConfigApplicationContext(ManConfig.class);
        //从容器中获取对象实例
        Man man = context.getBean(Man.class);
        man.driveCar();
    }
}
```

隐式的bean**发现机制和自动装配**

```java
/**
 * 这是一个游戏光盘的实现
 */
//这个简单的注解表明该类回作为组件类，并告知Spring要为这个创建bean。
@Component
public class GameDisc implements Disc{
    @Override
    public void play() {
        System.out.println("我是马里奥游戏光盘。");
    }
}
```

不过，组件扫描默认是不启用的。我们还需要显示配置一下Spring，从而命令它去寻找@Component注解的类，并为其创建bean。

```java
@Configuration
@ComponentScan
public class DiscConfig {
}
```

我们在DiscConfig上加了一个@ComponentScan注解表示在Spring中开启了组件扫描，默认扫描与配置类相同的包，就可以扫描到这个GameDisc的Bean了。这就是Spring的自动装配机制

**除了提供BeanFactory所支持的所有功能外ApplicationContext还有额外的功能**

- 默认初始化所有的Singleton，也可以通过配置取消预初始化。
- 继承MessageSource，因此支持国际化。
- 资源访问，比如访问URL和文件。
- 事件机制。
- 同时加载多个配置文件。
- 以声明式方式启动并创建Spring容器。

https://www.jianshu.com/p/2854d8984df

# 底层原理 (降低类之间的耦合度)

- 底层原理使用技术
  - xml配置文件
  - dom4j解决xml
  - 工厂设计模式
  - 反射
- 原理

```java
//伪代码
//需要实例化的类
public class UserService{
}

public class UserServlet{
    //得到UserService的对象
    //原始的做法：new 对象();  来创建
    
    //经过spring后
    UserFactory.getService();   //(下面两步的代码调用的)
}
```

**第一步：创建xml配置文件，配置要创建的对象类**

```java
<bean id="userService" class="cn.blinkit.UserService"/>
```

**第二步：创建工厂类，使用dom4j解析配置文件+反射**

```java
public class Factory {
    //返回UserService对象的方法
    public static UserService getService() {
        //1.使用dom4j来解析xml文件  
        //根据id值userService，得到id值对应的class属性值
        String classValue = "class属性值";
        //2.使用反射来创建类对象
        Class clazz = Class.forName(classValue)；
        //创建类的对象
        UserService service = clazz.newInstance();
        return service;
    }
}
```

# 3.初始化

整个脉络很庞大，初始化的过程主要就是**读取XML资源，并解析，最终注册到Bean Factory中**：

BeanDefinition的**Resouce定位，BeanDefinition的载入和注册**三个基本的过程。

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/20190528203916.png)

```java
<bean id="XiaoWang" class="com.springstudy.talentshow.SuperInstrumentalist">
    <property name="instruments">
        <list>
            <ref bean="piano"/>
            <ref bean="saxophone"/>
        </list>
    </property>
</bean>
```

加载时需要读取、解析、注册bean，这个过程具体的调用栈如下所示：

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/20190528204142.png)

## 3.1准备

**保存配置位置，并刷新**
在调用ClassPathXmlApplicationContext后，先会将配置位置信息保存到configLocations，供后面解析使用，之后，会调用`AbstractApplicationContext`的**refresh方法**进行刷新：

```java
public ClassPathXmlApplicationContext(String[] configLocations, boolean refresh, 
        ApplicationContext parent) throws BeansException {

    super(parent);
    // 保存位置信息，比如`com/springstudy/talentshow/talent-show.xml`
    setConfigLocations(configLocations);
    if (refresh) {
        // 刷新
        refresh();
    }
}

public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // Prepare this context for refreshing.
        prepareRefresh();
        // Tell the subclass to refresh the internal bean factory.
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
        // Prepare the bean factory for use in this context.
        prepareBeanFactory(beanFactory);
        try {
            // Allows post-processing of the bean factory in context subclasses.
            postProcessBeanFactory(beanFactory);
            // Invoke factory processors registered as beans in the context.
            invokeBeanFactoryPostProcessors(beanFactory);
            // Register bean processors that intercept bean creation.
            registerBeanPostProcessors(beanFactory);
            // Initialize message source for this context.
            initMessageSource();
            // Initialize event multicaster for this context.
            initApplicationEventMulticaster();
            // Initialize other special beans in specific context subclasses.
            onRefresh();
            // Check for listener beans and register them.
            registerListeners();
            // Instantiate all remaining (non-lazy-init) singletons.
            finishBeanFactoryInitialization(beanFactory);
            // Last step: publish corresponding event.
            finishRefresh();
        }
        catch (BeansException ex) {
            // Destroy already created singletons to avoid dangling resources.
            destroyBeans();
            // Reset 'active' flag.
            cancelRefresh(ex);
            // Propagate exception to caller.
            throw ex;
        }
    }
}
```

**创建载入BeanFactory**

```java
protected final void refreshBeanFactory() throws BeansException {
    // ... ...
    DefaultListableBeanFactory beanFactory = createBeanFactory();
    // ... ...
    loadBeanDefinitions(beanFactory);
    // ... ...
}
```

**创建XMLBeanDefinitionReader**

```java
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory)
     throws BeansException, IOException {
    // Create a new XmlBeanDefinitionReader for the given BeanFactory.
    XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);
    // ... ...
    // Allow a subclass to provide custom initialization of the reader,
    // then proceed with actually loading the bean definitions.
    initBeanDefinitionReader(beanDefinitionReader);
    loadBeanDefinitions(beanDefinitionReader);
}
```

## 3.2读取

**创建处理每一个resource**

```java
public int loadBeanDefinitions(String location, Set<Resource> actualResources)
     throws BeanDefinitionStoreException {
    // ... ...
    // 通过Location来读取Resource
    Resource[] resources = ((ResourcePatternResolver) resourceLoader).getResources(location);
    int loadCount = loadBeanDefinitions(resources);
    // ... ...
}

public int loadBeanDefinitions(Resource... resources) throws BeanDefinitionStoreException {
    Assert.notNull(resources, "Resource array must not be null");
    int counter = 0;
    for (Resource resource : resources) {
        // 载入每一个resource
        counter += loadBeanDefinitions(resource);
    }
    return counter;
}
```

**处理XML每个元素**

```java
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
    // ... ...
    NodeList nl = root.getChildNodes();
    for (int i = 0; i < nl.getLength(); i++) {
        Node node = nl.item(i);
        if (node instanceof Element) {
            Element ele = (Element) node;
            if (delegate.isDefaultNamespace(ele)) {
                // 处理每个xml中的元素，可能是import、alias、bean
                parseDefaultElement(ele, delegate);
            }
            else {
                delegate.parseCustomElement(ele);
            }
        }
    }
    // ... ...
}
```

**解析和注册bean**

```java
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
    // 解析
    BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
    if (bdHolder != null) {
        bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
        try {
            // 注册
            // Register the final decorated instance.
            BeanDefinitionReaderUtils.registerBeanDefinition(
                bdHolder, getReaderContext().getRegistry());
        }
        catch (BeanDefinitionStoreException ex) {
            getReaderContext().error("Failed to register bean definition with name '" +
                    bdHolder.getBeanName() + "'", ele, ex);
        }
        // Send registration event.
        getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
    }
}
```

本步骤中，通过`parseBeanDefinitionElement`将XML的元素解析为`BeanDefinition`，然后存在`BeanDefinitionHolder`中，然后再利用`BeanDefinitionHolder`将`BeanDefinition`注册，实质就是把`BeanDefinition`的实例put进`BeanFactory`中，后面将详细的介绍解析和注册过程。

## 3.3解析

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/20190528204945.png)

**处理每个Bean的元素**

```java
public AbstractBeanDefinition parseBeanDefinitionElement(
        Element ele, String beanName, BeanDefinition containingBean) {

    // ... ...
    // 创建beandefinition
    AbstractBeanDefinition bd = createBeanDefinition(className, parent);

    parseBeanDefinitionAttributes(ele, beanName, containingBean, bd);
    bd.setDescription(DomUtils.getChildElementValueByTagName(ele, DESCRIPTION_ELEMENT));

    parseMetaElements(ele, bd);
    parseLookupOverrideSubElements(ele, bd.getMethodOverrides());
    parseReplacedMethodSubElements(ele, bd.getMethodOverrides());
    // 处理“Constructor”
    parseConstructorArgElements(ele, bd);
    // 处理“Preperty”
    parsePropertyElements(ele, bd);
    parseQualifierElements(ele, bd);
    // ... ...
}
```

**处理属性的值**

```java
public Object parsePropertyValue(Element ele, BeanDefinition bd, String propertyName) {
    String elementName = (propertyName != null) ?
                    "<property> element for property '" + propertyName + "'" :
                    "<constructor-arg> element";

    // ... ...
    if (hasRefAttribute) {
    // 处理引用
        String refName = ele.getAttribute(REF_ATTRIBUTE);
        if (!StringUtils.hasText(refName)) {
            error(elementName + " contains empty 'ref' attribute", ele);
        }
        RuntimeBeanReference ref = new RuntimeBeanReference(refName);
        ref.setSource(extractSource(ele));
        return ref;
    }
    else if (hasValueAttribute) {
    // 处理值
        TypedStringValue valueHolder = new TypedStringValue(ele.getAttribute(VALUE_ATTRIBUTE));
        valueHolder.setSource(extractSource(ele));
        return valueHolder;
    }
    else if (subElement != null) {
    // 处理子类型（比如list、map等）
        return parsePropertySubElement(subElement, bd);
    }
    // ... ...
}
```

## 3.4注册

```java
public static void registerBeanDefinition(
        BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
        throws BeanDefinitionStoreException {

    // Register bean definition under primary name.
    String beanName = definitionHolder.getBeanName();
    registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());

    // Register aliases for bean name, if any.
    String[] aliases = definitionHolder.getAliases();
    if (aliases != null) {
        for (String alias : aliases) {
            registry.registerAlias(beanName, alias);
        }
    }
}

public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
        throws BeanDefinitionStoreException {

    // ......

    // 将beanDefinition注册
    this.beanDefinitionMap.put(beanName, beanDefinition);

    // ......
}
```

注册过程中，最核心的一句就是：`this.beanDefinitionMap.put(beanName, beanDefinition)`，也就是说注册的实质**就是以beanName为key，以beanDefinition为value，将其put到HashMap中。**

# 4.注入依赖

当完成初始化IOC容器后，如果**bean没有设置lazy-init(延迟加载)属性**，那么bean的实例就会在初始化IOC完成之后，及时地进行初始化。初始化时会先建立实例，然后根据配置利用反射对实例进行进一步操作，具体流程如下所示：

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/20190528205330.png)

**创建bean的实例**
创建bean的实例过程函数调用栈如下所示：

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/20190528205426.png)

**注入bean的属性**
注入bean的属性过程函数调用栈如下所示：

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/20190528205533.png)

在创建bean和注入bean的属性时，都是在doCreateBean函数中进行的，我们重点看下：

```java
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, 
        final Object[] args) {
    // Instantiate the bean.
    BeanWrapper instanceWrapper = null;
    if (mbd.isSingleton()) {
        instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
    }
    if (instanceWrapper == null) {
        // 创建bean的实例
        instanceWrapper = createBeanInstance(beanName, mbd, args);
    }

    // ... ...

    // Initialize the bean instance.
    Object exposedObject = bean;
    try {
        // 初始化bean的实例，如注入属性
        populateBean(beanName, mbd, instanceWrapper);
        if (exposedObject != null) {
            exposedObject = initializeBean(beanName, exposedObject, mbd);
        }
    }

    // ... ...
}
```

理解了以上两个过程，我们就可以自己实现一个简单的Spring框架了。于是，我根据自己的理解实现了一个简单的IOC框架[Simple Spring](https://github.com/Yikun/simple-spring)，有兴趣可以看看。

# 总结

IoC 在 Spring 里，只需要低级容器就可以实现，2 个步骤：

1. 加载配置文件，解析成 BeanDefinition 放在 Map 里。
2. 调用 getBean 的时候，从 BeanDefinition 所属的 Map 里，拿出 Class 对象进行实例化，同时，如果有依赖关系，将递归调用 getBean 方法 —— 完成依赖注入。

上面就是 Spring 低级容器（BeanFactory）的 IoC。

至于高级容器 ApplicationContext，他包含了低级容器的功能，当他执行 refresh 模板方法的时候，将刷新整个容器的 Bean。同时其作为高级容器，包含了太多的功能。一句话，他不仅仅是 IoC。他支持不同信息源头，支持 BeanFactory 工具类，支持层级容器，支持访问文件资源，支持事件发布通知，支持接口回调等等。

# 参考

[1]: https://yikun.github.io/2015/05/29/Spring-IOC%E6%A0%B8%E5%BF%83%E6%BA%90%E7%A0%81%E5%AD%A6%E4%B9%A0/	"Spring-IOC核心源码学习"

必问的 Spring IOC，要看看了！！！ http://thinkinjava.cn 作者：莫那·鲁道

