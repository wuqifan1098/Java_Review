# 面试题

## 1. Bean的生命周期（涂鸦智能)

1. 通过构造器或工厂方法**创建Bean实例**
2. 为Bean的**属性设置值**和对其它Bean的引用
3. 如果Bean实现了BeanNameAware接口，工厂调用Bean的setBeanName()方法传递Bean的ID。（和下面的一条均属于检查Aware接口）
4. 如果Bean实现了BeanFactoryAware接口，工厂调用setBeanFactory()方法传入工厂自身
5. 将Bean实例**传递给Bean的前置处理器**的postProcessBeforeInitialization(Object bean, String beanname)方法
6. 调用Bean的**初始化方法**(自身的方法init-method)
7. 将Bean实例**传递给Bean的后置处理器**的postProcessAfterInitialization(Object bean, String beanname)方法
8. Bean可以使用了
9. 当容器关闭时，调用Bean的销毁方法

```java
public class Person
{
    String name;
    int age;
    String sex;
    public String getName()
    {
        return name;
    }
    public void setName(String name)
    {
        System.out.println("执行name属性");
        this.name = name;
    }
    public int getAge()
    {
        return age;
    }
    public void setAge(int age)
    {
        this.age = age;
    }
    public String getSex()
    {
        return sex;
    }
    public void setSex(String sex)
    {
        this.sex = sex;
    }
    @Override
    public String toString()
    {
        return "Person [name=" + name + ", age=" + age + ", sex=" + sex + "]";
    }
    public void init()
    {
        System.out.println("执行初始化方法！");
    }
    public Person()
    {
        System.out.println("执行构造函数");
    }
    public void destory()
    {
        System.out.println("执行销毁方法");
    }
    public void fun()
    {
        System.out.println("正在使用bean实例");
    }
}
```

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="person" class="com.test.Person" init-method="init" destroy-method="destory">
        <property name="name" value="xujian"></property>
        <property name="age" value="23"></property>
        <property name="sex" value="男"></property>
    </bean>
</beans>
```

　运行结果：

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/run%20result.png)

从上图可以看到，首先通过构造函数来创建bean的实例，然后通过setter方法设置属性，在使用bean实例之前，执行了init初始化方法，使用之后又执行了destroy方法。

https://www.cnblogs.com/xujian2014/p/5049483.html

## 2. 如何使用 BeanPostProcessor 和 BeanFactoryPostProcessor ?



## 3. BeanFactory和FactoryBean的区别

区别：BeanFactory是个Factory，也就是IOC容器或对象工厂，FactoryBean是个Bean。在Spring中，**所有的Bean都是由BeanFactory(也就是IOC容器)来进行管理的**。但对FactoryBean而言，**这个Bean不是简单的Bean，而是一个能生产或者修饰对象生成的工厂Bean,它的实现与设计模式中的工厂模式和修饰器模式类似** 

https://www.cnblogs.com/aspirant/p/9082858.html

BeanFactory是一个接口，`public interface BeanFactory`

提供如下方法：

```java
Object getBean(String name)
<T> T getBean(String name, Class<T> requiredType)
<T> T getBean(Class<T> requiredType)
Object getBean(String name, Object... args)
boolean containsBean(String name)
boolean isSingleton(String name)
boolean isPrototype(String name)
boolean isTypeMatch(String name, Class<?> targetType)
Class<?> getType(String name)
String[] getAliases(String name)
```

在 Spring 中，`BeanFactory`是 IoC 容器的核心接口。它的职责包括：**实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。**

`BeanFactory` 提供的高级配置机制，使得管理任何性质的对象成为可能。
 `ApplicationContext` 是 `BeanFactory` 的扩展，功能得到了进一步增强，比如更易与 **Spring AOP 集成、消息资源处理(国际化处理)、事件传递及各种不同应用层的 context 实现(如针对 web 应用的`WebApplicationContext`)。**

FactoryBean也是接口，为IOC容器中Bean的实现提供了更加灵活的方式，FactoryBean在IOC容器的基础上给Bean的实现加上了一个简单工厂模式和装饰模式(如果想了解装饰模式参考：[修饰者模式(装饰者模式，Decoration)](https://www.cnblogs.com/aspirant/p/9083082.html) 我们可以在getObject()方法中灵活配置。

## 4. Bean的注入方式

- 属性注入
- 构造器注入
- 工厂方法注入

## 5. 解决循环依赖的方式

不要使用**基于构造函数的依赖注入**，可以通过以下方式解决：

　　1.在字段上使用@Autowired注解，让Spring决定在合适的时机注入

　　2.用**基于setter方法的依赖注入。**

https://www.cnblogs.com/liuqing576598117/p/11227007.html

## 6. bean的初始化和销毁几种方式（有赞）

- init-method和destroy-method 通过**属性指定**[初始化](https://www.baidu.com/s?wd=初始化&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)之后 /销毁之前调用的操作方法
- InitializingBean 和DisposableBean  **实现接口**来定制[初始化](https://www.baidu.com/s?wd=初始化&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)之后/销毁之前的操作方法
- @PostConstruct或@PreDestroy 在指定方法上**加上注解来**制定方法是在[初始化](https://www.baidu.com/s?wd=初始化&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)之后还是销毁之前调用。

https://blog.csdn.net/alex_xfboy/article/details/51211054

https://www.jianshu.com/p/34692cdf7d5f

# 含义 #

Bean的含义是可重复使用的Java组件。

对于使用Spring框架的开发人员来说，我们主要做的主要有两件事情：**①开发Bean;②配置Bean;**

Spring帮我们做的就是根据配置文件来创建Bean实例，并调用Bean实例的方法来完成“依赖注入”，可以把Spring容器理解成一个大型工厂，Bean就是该工厂的产品，工厂(Spirng容器)里能生产出来什么样的产品（Bean），完全取决于我们在配置文件中的配置。

# 作用域 #

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/20190528162952.png)

 比较常用的是**singleton 和 prototype 两种作用域**，对于singleton作用域，每次请求该Bean都将获得相同的实例，Spring容器负责跟踪监视Bean实例的状态，负责维护Bean实例的生命周期行为，如果一个Bean被设置成prototype作用域，程序每次请求该id的Bean，Spring都会创建一个新的Bean实例，然后返回给程序，在这种情况下，**Spring容器仅仅使用new 关键字创建Bean实例，一旦创建成功，容器Spring不再对Bean的生命周期负责，也不会维护Bean实例的状态。**

 如果不指定Bean的作用域，Spring**默认使用singleton作用域**。Java在常见Java实例时，需要进行内存申请，销毁实例是，需要完成垃圾回收，这些工作都会导致系统开销的增加。因此prototype作用域Bean的创建销毁代价比较大。而singleton作用域的Bean 实例一旦创建成功，可以重复使用，因此，除非必要，否则避免将Bean作用域设置成prototype。

# 初始化过程

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/bean%E7%9A%84%E5%88%9D%E5%A7%8B%E5%8C%96.png)

简要总结：

- BeanDefinitionReader**读取Resource所指向的配置文件资源**，然后解析配置文件。配置文件中每一个`<bean>`解析成一个**BeanDefinition对象**，并**保存**到BeanDefinitionRegistry中；
- 容器扫描BeanDefinitionRegistry中的BeanDefinition；调用InstantiationStrategy**进行Bean实例化的工作**；使用**BeanWrapper完成Bean属性的设置**工作；
- 单例Bean缓存池：Spring 在DefaultSingletonBeanRegistry类中提供了一个用于缓存单实例 Bean 的**缓存器**，它是一个用HashMap实现的缓存器，单实例的Bean**以beanName为键保存在这个HashMap**中。

<https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484247&idx=1&sn=e228e29e344559e469ac3ecfa9715217&chksm=ebd74256dca0cb40059f3f627fc9450f916c1e1b39ba741842d91774f5bb7f518063e5acf5a0&scene=21##wechat_redirect>

# 生命周期 #

getBean() 只是 **bean 实例化进程的入口**，真正的实现逻辑其实是在 AbstractAutowireCapableBeanFactory 的 doCreateBean() 实现，实例化过程如下图：

![img](https://pic4.zhimg.com/80/v2-1db48a25c20905cd7518da5d257ef937_hd.jpg)

## bean实例化，调用bean的构造参数

在 doCreateBean() 中首先进行 bean 实例化工作，**主要由 createBeanInstance() 实现，该方法返回一个 BeanWrapper 对象。**BeanWrapper 对象是 Spring 的一个低级 Bean 基础结构的核心接口，为什么说是低级呢？因为这个时候的 Bean 还不能够被我们使用，连最基本的属性都没有设置。而且在我们实际开发过程中一般都不会直接使用该类，而是通过 BeanFactory 隐式使用。

BeanWrapper 接口有一个默认实现类 BeanWrapperImpl，其主要作用是对 Bean 进行“包裹”，然后对这个包裹的 bean 进行操作，比如后续注入 bean 属性。

**在实例化 bean 过程中，Spring 采用“策略模式”来决定采用哪种方式来实例化 bean，一般有反射和 CGLIB 动态字节码两种方式。**

InstantiationStrategy 定义了 Bean 实例化策略的抽象接口，其子类 SimpleInstantiationStrategy 提供了基于反射来实例化对象的功能，但是不支持方法注入方式的对象实例化。CglibSubclassingInstantiationStrategy 继承 SimpleInstantiationStrategy，他除了拥有父类以反射实例化对象的功能外，还提供了通过 CGLIB 的动态字节码的功能进而支持方法注入所需的对象实例化需求。默认情况下，Spring 采用 CglibSubclassingInstantiationStrategy。

## 设置对象属性

调用bean的set方法，将**属性注入到bean的属性中**

## 激活 Aware

检查bean是否实现**BeanNameAware、BeanFactoryAware、ApplicationContextAware**接口，如果实现了这几个接口Spring会分别调用其中实现的方法。

当 Spring 完成 bean 对象实例化并且设置完相关属性和依赖后，则会开始 bean 的初始化进程（initializeBean()），**初始化第一个阶段是检查当前 bean 对象是否实现了一系列以 Aware 结尾的的接口。**

BeanNameAware：setBeanName(String name)方法，对该 **bean 对象定义的 beanName 设置到当前对象实例中**

BeanFactoryAware：setBeanFactory(BeanFactory bf)方法，**会将自身注入到当前对象实例中，这样当前对象就会拥有一**

**个 BeanFactory 容器的引用。**

BeanClassLoaderAware：将当前 bean 对象相应的 ClassLoader 注入到当前对象实例中

ApplicationContextAware：setApplicationContext(ApplicationContext context)方法，参数是bean所在的引用的上下文，

如果是用Bean工厂创建bean，那就可以忽略ApplicationContextAware**。

```java
private void invokeAwareMethods(final String beanName, final Object bean) {
    if (bean instanceof Aware) {
        if (bean instanceof BeanNameAware) {
            ((BeanNameAware) bean).setBeanName(beanName);
        }
        if (bean instanceof BeanClassLoaderAware) {
            ClassLoader bcl = getBeanClassLoader();
            if (bcl != null) {
                ((BeanClassLoaderAware) bean).setBeanClassLoader(bcl);
            }
        }
        if (bean instanceof BeanFactoryAware) {
            ((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);
        }
    }
}
```

## BeanPostProcessor

初始化第二个阶段则是 BeanPostProcessor 增强处理，在该阶段 BeanPostProcessor 会处理当前容器内所有符合条件的实例化后的 bean 对象。它主要是对 Spring 容器提供的 bean 实例对象进行有效的扩展，允许 Spring 在初始化 bean 阶段对其进行定制化修改，如处理标记接口或者为其提供代理实现。

BeanPostProcessor 接口提供了两个方法，在不同的时机执行，分别对应上图的前置处理和后置处理。

```java
public interface BeanPostProcessor {
    @Nullable
     default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
    @Nullable
     default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
}
```

## InitializingBean 和 init-method

InitializingBean 是一个接口，它为 Spring Bean 的初始化提供了一种方式，它有一个 afterPropertiesSet() 方法，**在 bean 的初始化进程中会判断当前 bean 是否实现了 InitializingBean，如果实现了则调用 afterPropertiesSet() 进行初始化工作。**然后再检查是否也指定了 init-method()，如果指定了则**通过反射机制调用指定的 init-method()。**

```java
protected void invokeInitMethods(String beanName, final Object bean, @Nullable RootBeanDefinition mbd)
   throws Throwable {
    Boolean isInitializingBean = (bean instanceof InitializingBean);
    if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
        if (logger.isDebugEnabled()) {
            logger.debug("Invoking afterPropertiesSet() on bean with name '" + beanName + "'");
        }
        if (System.getSecurityManager() != null) {
            try {
                AccessController.doPrivileged((PrivilegedExceptionAction<Object>) () -> {
                    ((InitializingBean) bean).afterPropertiesSet();
                    return null;
                }
                , getAccessControlContext());
            }
            catch (PrivilegedActionException pae) {
                throw pae.getException();
            }
        } else {
            ((InitializingBean) bean).afterPropertiesSet();
        }
    }
    if (mbd != null && bean.getClass() != NullBean.class) {
        String initMethodName = mbd.getInitMethodName();
        if (StringUtils.hasLength(initMethodName) &&
             !(isInitializingBean && "afterPropertiesSet".equals(initMethodName)) &&
             !mbd.isExternallyManagedInitMethod(initMethodName)) {
            invokeCustomInitMethod(beanName, bean, mbd);
        }
    }
}
```

## 使用bean

bean将会一直保留在应用的上下文中，直到该应用上下文被销毁。

## DisposableBean 和 destroy-method

检查**bean是否实现DisposableBean接口**，Spring会调用它们的destory方法

当一个 bean 对象经历了实例化、设置属性、初始化阶段,那么该 bean 对象就可以供容器使用了（调用的过程）。当完成调用后，如果是 singleton 类型的 bean ，则会看当前 bean 是否应实现了 DisposableBean 接口或者配置了 destroy-method 属性，如果是的话，则会为该实例注册一个用于对象销毁的回调方法，便于在这些 singleton 类型的 bean 对象销毁之前执行销毁逻辑。

但是，并不是对象完成调用后就会立刻执行销毁方法，因为这个时候 Spring 容器还处于运行阶段，只有当 Spring 容器关闭的时候才会去调用。但是， Spring 容器不会这么聪明会自动去调用这些销毁方法，而是需要我们**主动去告知 Spring 容器。**

- 对于 BeanFactory 容器而言，我们需要**主动调用 destroySingletons()** 通知 BeanFactory 容器去执行相应的销毁方法。
- 对于 ApplicationContext 容器而言调用 registerShutdownHook() 方法。

参考：https://zhuanlan.zhihu.com/p/64143194

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/SpringBean%20lifeCycle.png)

上面流程图当中涉及调用很多的方法，可能我们直接去理解和记忆比较困难，其实对于这么一大堆方法我们可以根据它们的特点对他们进行整理分类，下面提供一种可供大家参考的分类模型：

| 分类类型               | 所包含方法                                                   |
| ---------------------- | ------------------------------------------------------------ |
| Bean自身的方法         | 配置文件中的init-method和destroy-method配置的方法、Bean对象自己调用的方法 |
| Bean级生命周期接口方法 | BeanNameAware、BeanFactoryAware、InitializingBean、DiposableBean等接口中的方法 |
| 容器级生命周期接口方法 | InstantiationAwareBeanPostProcessor、BeanPostProcessor等后置处理器实现类中重写的方法 |

  这时回过头再来看这些方法轮廓上就比较清晰了，记忆的时候我们可通过记忆分类的类型来理解性的记忆所涉及的方法。

https://www.cnblogs.com/jasonZh/p/8762855.html

# Bean 实例的创建方式 #

常用的创建方式有以下四种：

1) setter 方法

2) 构造函数

3) 静态工厂

4) 实例工厂

**一、用 setter 方式**

```java
public interface IUserDao {
                void addUser();
                void delUser();
                void updateUser();
            }
            
            public class UserDaoImpl implements IUserDao {
                public void addUser() {
                    System.out.println("addUser方法被调用了");
                }        
                public void delUser() {
                    System.out.println("delUser方法被调用了");
                }        
                public void updateUser() {
                    System.out.println("updateUser方法被调用了");
                }
            }
            
            public class UserAction {
                    private IUserDao dao; //dao是一个依赖对象,要由springg进行管理,要生成 get set 方法
                            public void execute(){
                            dao.addUser();
                            dao.updateUser();
                            dao.delUser();
                    }
                }
```

```java
//配置文件
<bean name="userAction_name" class="cat.action.UserAction" >
<property name="dao" ref="userDao_name" />  //引用的是下面的名称
</bean>    
<bean name="userDao_name" class="cat.dao.UserDaoImpl" />
 //测试
ClassPathXmlApplicationContext ctx=new ClassPathXmlApplicationContext("beans.xml");
UserAction action=(UserAction)ctx.getBean("userAction_name");
action.execute(); 
```

**二、构造函数**

```java
public class UserAction {
       //public UserAction(){} 可以保保留一个无参的构造函数
                
       //这是几个依赖对象,不用生成get set方法了
       private UserInfo user;
       private String school;
       private IUserDao dao;     
            
       //希望Spring 由构造函数注入依赖对象
       public UserAction(IUserDao dao,UserInfo user,String school){
              this.dao=dao;
              this.school=school;
              this.user=user;
              }
                
            
       public void execute(){
              dao.addUser();
              dao.updateUser();
              dao.delUser();
                    
              System.out.println(user);
              System.out.println(school);
}
```

```java
//配置文件
<bean name="userInfo_name" class="cat.beans.UserInfo" >
      <property name="id" value="1" />
      <property name="userName" value="周周" />
      <property name="password" value="123" />
      <property name="note" value="这是备注" />
</bean>
                    
<bean name="userAction_name" class="cat.action.UserAction" >
      <constructor-arg ref="userDao_name" />
      <constructor-arg ref="userInfo_name" />
      <constructor-arg value="哈尔滨师范大学" />
</bean>
            
/*
也可以指定 索引和 type 属性 , 索引和type 都可以不指定
<bean name="userAction_name" class="cat.action.UserAction" >
<constructor-arg index="0" ref="userDao_name" type="cat.dao.IUserDao" />  如果是接口,就不能指定是实现类的类型
<constructor-arg index="1" ref="userInfo_name" type="cat.beans.UserInfo" />
<constructor-arg index="2" value="哈尔滨师范大学"  />
</bean>
*/
                
<bean name="userDao_name" class="cat.dao.UserDaoImpl" />
```

```java
//测试
ClassPathXmlApplicationContext ctx=new ClassPathXmlApplicationContext("beans.xml");
UserAction action=(UserAction)ctx.getBean("userAction_name");
action.execute(); 
```

**三、静态工厂方式**

```java
//工厂,用来生成dao的实现类
public class UserDaoFactory {
public static IUserDao createUserDaoInstance(){
       return new UserDaoOracleImpl();
       }
}

           
public class UserAction {
       private IUserDao dao;//使用工厂方式注值,也要生成set方法
       public void execute(){
              dao.addUser();
              dao.updateUser();
              dao.delUser();
}
                public void setDao(IUserDao dao) {
              this.dao = dao;
              }    
}
```

```java
//配置文件 
<bean name="userAction_name" class="cat.action.UserAction" >
<property name="dao"  ref="userDao_name" />
</bean>
              
<bean name="userDao_name" class="cat.dao.UserDaoFactory" factory-method="createUserDaoInstance" />
```

```java
//测试
ClassPathXmlApplicationContext ctx=new ClassPathXmlApplicationContext("beans.xml");
UserAction action=(UserAction)ctx.getBean("userAction_name");
action.execute(); 
```

**四、实例工厂**

```java
//工厂 =>
public class UserDaoFactory {
//这个方法不是静态的
public  IUserDao createUserDaoInstance(){
        return new UserDaoOracleImpl();
        }
}
```

```java
//配置文件 
<bean name="userAction_name" class="cat.action.UserAction" >
<property name="dao"  ref="userDao_name" />
</bean>
              
<bean  name="userDaoFactory_name" class="cat.dao.UserDaoFactory" />
<bean name="userDao_name" factory-bean="userDaoFactory_name" factory-method="createUserDaoInstance" />
```

https://www.cnblogs.com/1693977889zz/p/8146563.html

