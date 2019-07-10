# 面试题

## 1. Bean的生命周期（涂鸦智能)

1. 通过构造器或工厂方法**创建Bean实例**
2. 为Bean的属性设置值和对其它Bean的引用
3. 如果Bean实现了BeanNameAware接口，工厂调用Bean的setBeanName()方法传递Bean的ID。（和下面的一条均属于检查Aware接口）
4. 如果Bean实现了BeanFactoryAware接口，工厂调用setBeanFactory()方法传入工厂自身
5. 将Bean实例传递给Bean的前置处理器的postProcessBeforeInitialization(Object bean, String beanname)方法
6. 调用Bean的初始化方法(自身的方法init-method)
7. 将Bean实例传递给Bean的后置处理器的postProcessAfterInitialization(Object bean, String beanname)方法
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

# 含义 #

Bean的含义是可重复使用的Java组件。

对于使用Spring框架的开发人员来说，我们主要做的主要有两件事情：**①开发Bean;②配置Bean;**

Spring帮我们做的就是根据配置文件来创建Bean实例，并调用Bean实例的方法来完成“依赖注入”，可以把Spring容器理解成一个大型工厂，Bean就是该工厂的产品，工厂(Spirng容器)里能生产出来什么样的产品（Bean），完全取决于我们在配置文件中的配置。

# 作用域 #

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/20190528162952.png)

 比较常用的是**singleton 和 prototype 两种作用域**，对于singleton作用域，每次请求该Bean都将获得相同的实例，Spring容器负责跟踪监视Bean实例的状态，负责维护Bean实例的生命周期行为，如果一个Bean被设置成prototype作用域，程序每次请求该id的Bean，Spring都会创建一个新的Bean实例，然后返回给程序，在这种情况下，**Spring容器仅仅使用new 关键字创建Bean实例，一旦创建成功，容器Spring不再对Bean的生命周期负责，也不会维护Bean实例的状态。**

 如果不指定Bean的作用域，Spring**默认使用singleton作用域**。Java在常见Java实例时，需要进行内存申请，销毁实例是，需要完成垃圾回收，这些工作都会导致系统开销的增加。因此prototype作用域Bean的创建销毁代价比较大。而singleton作用域的Bean 实例一旦创建成功，可以重复使用，因此，除非必要，否则避免将Bean作用域设置成prototype。

# 生命周期 #

 1、实例化一个Bean－－也就是我们常说的new；

 2、按照Spring上下文对实例化的Bean进行配置－－也就是IOC注入；

 3、如果这个Bean已经实现了BeanNameAware接口，会调用它实现的setBeanName(String)方法，此处传递的就是**Spring配置文件中Bean的id值**

 4、如果这个Bean已经实现了BeanFactoryAware接口，会调用它实现的setBeanFactory(setBeanFactory(BeanFactory)**传递的是Spring工厂自身**（可以用这个方式来获取其它Bean，只需在Spring配置文件中配置一个普通的Bean就可以）；

  5、如果这个Bean已经实现了ApplicationContextAware接口，会调用setApplicationContext(ApplicationContext)方法，**传入Spring上下文**（同样这个方式也可以实现步骤4的内容，但比4更好，因为ApplicationContext是BeanFactory的子接口，有更多的实现方法）；

  6、如果这个Bean关联了BeanPostProcessor接口，将会调用postProcessBeforeInitialization(Object obj, String s)方法，BeanPostProcessor经常被用作是Bean内容的更改，并且由于这个是在Bean初始化结束时调用那个的方法，也可以被应用于内存或缓存技术；

  7、如果Bean在Spring配置文件中配置了init-method属性会自动调用其配置的初始化方法。

  8、如果这个Bean关联了BeanPostProcessor接口，将会调用postProcessAfterInitialization(Object obj, String s)方法、；

  注：以上工作完成以后就可以应用这个Bean了，那这个Bean是一个Singleton的，所以一般情况下我们调用同一个id的Bean会是在内容地址相同的实例，当然在Spring配置文件中也可以配置非Singleton，这里我们不做赘述。

  9、当Bean不再需要时，会经过清理阶段，如果Bean实现了DisposableBean这个接口，会调用那个其实现的destroy()方法；

  10、最后，如果这个Bean的Spring配置中配置了destroy-method属性，会自动调用其配置的销毁方法。

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

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
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

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
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

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
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

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
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

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//测试
ClassPathXmlApplicationContext ctx=new ClassPathXmlApplicationContext("beans.xml");
UserAction action=(UserAction)ctx.getBean("userAction_name");
action.execute(); 
```

**三、静态工厂方式**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
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

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//配置文件 
<bean name="userAction_name" class="cat.action.UserAction" >
<property name="dao"  ref="userDao_name" />
</bean>
              
<bean name="userDao_name" class="cat.dao.UserDaoFactory" factory-method="createUserDaoInstance" />
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//测试
ClassPathXmlApplicationContext ctx=new ClassPathXmlApplicationContext("beans.xml");
UserAction action=(UserAction)ctx.getBean("userAction_name");
action.execute(); 
```

**四、实例工厂**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//工厂 =>
public class UserDaoFactory {
//这个方法不是静态的
public  IUserDao createUserDaoInstance(){
        return new UserDaoOracleImpl();
        }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//配置文件 
<bean name="userAction_name" class="cat.action.UserAction" >
<property name="dao"  ref="userDao_name" />
</bean>
              
<bean  name="userDaoFactory_name" class="cat.dao.UserDaoFactory" />
<bean name="userDao_name" factory-bean="userDaoFactory_name" factory-method="createUserDaoInstance" />
```

https://www.cnblogs.com/1693977889zz/p/8146563.html

