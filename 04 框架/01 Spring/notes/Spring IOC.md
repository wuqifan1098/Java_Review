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

## BeanFactory 简介

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

# 参考

[1]: https://yikun.github.io/2015/05/29/Spring-IOC%E6%A0%B8%E5%BF%83%E6%BA%90%E7%A0%81%E5%AD%A6%E4%B9%A0/	"Spring-IOC核心源码学习"

