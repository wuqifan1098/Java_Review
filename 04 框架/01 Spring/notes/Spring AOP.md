# 面试题

## 1.AOP配置（华数）

注解配置AOP（使用 AspectJ 类库实现的），大致分为三步： 
1. 使用注解@Aspect来定义一个切面，在切面中定义切入点(@Pointcut),通知类型(@Before, @AfterReturning,@After,@AfterThrowing,@Around). 

2. 开发需要被拦截的类。 

3. 将切面配置到xml中，当然，我们也可以使用自动扫描Bean的方式。这样的话，那就交由Spring AoP容器管理。 

   注意：

   - @Aspect：意思是这个类为切面类  
- @Componet：因为作为切面类需要 Spring 管理起来，所以在初始化时就需要将这个类初始化加入 Spring 的管理；  
   - @Befoe：切入点的逻辑(Advice)
   - execution…:切入点语法 

## 2. 什么是AOP（华数）

我们要关注的方法散布**在不同类中，如果想要在这些方法中加入相同的处理逻辑**，过去我们采用的方法只能是硬编码，但是这样会造成大量的重复代码，不便于维护，AOP的出现就是解决针对这种问题的一种实现模式。

aop就是面向切面的编程。比如说你每做一次对数据库操作，都要生成一句日志。如果，你对数据库的操作有很多类，那你每一类中都要写关于日志的方法。但是如果你用aop，那么你可以写一个方法，在这个方法中**有关于数据库操作的方法，每一次调用这个方法的时候，**就加上生成日志的操作。

使用"横切"技术，AOP把软件系统分为两个部分：核心关注点和横切关注点。***业务处理的主要流程是核心关注点，与之关系不大的部分是横切关注点。***横切关注点的一个特点是，它们经常发生在核心关注点的多处，而各处基本相似，比如权限认证、日志、事务管理、性能监控等。AOP的作用在于分离系统中的各种关注点，将核心关注点和横切关注点分离开来。

## 3. CGLib实现（涂鸦智能）

通过“**继承”可以继承父类所有的公开方法，然后可以重写这些方法**，在重写时对这些方法增强，这就是cglib的思想。

https://www.cnblogs.com/xrq730/p/6661692.html

## 4. Spring AOP和AspectJ AOP的区别

Spring AOP属于**运行时增强**，而AspectJ是**编译时增强**。

Spring AOP**基于代理（Proxying）**，而AspectJ**基于字节码操作（Bytecode Manipulation）**。

AspectJ相比于Spring AOP功能更加强大，但是Spring AOP相对来说更简单。如果切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择AspectJ，它比SpringAOP快很多。

# AOP中的基本概念

不得不说，AOP的概念是真的多且难以理解，不过不用担心，聪明的你已经准备好战胜它们了。

- 通知(Adivce)

通知有5种类型：

- `Before` 在方法被调用之前调用

- `After` 在方法完成后调用通知，无论方法是否执行成功

- `After-returning` 在方法**成功执行之后调用通知**

- `After-throwing` 在方法**抛出异常后调用通知**

- `Around` 通知了好、包含了被通知的方法，在被通知的方法调用之前后调用之后执行自定义的行为

  我们可能会问，那通知对应系统中的代码是一个方法、对象、类、还是接口什么的呢？我想说一点，其实都不是，你可以理解通知就是对应我们日常生活中所说的通知，比如‘某某人，你2019年9月1号来学校报个到’，通知更多地体现一种告诉我们（告诉系统何）何时执行，规定一个时间，在系统运行中的某个时间点（比如抛异常啦！方法执行前啦！），`并非对应代码中的方法！并非对应代码中的方法！并非对应代码中的方法！`

- 切点（Pointcut）

  哈哈，这个你可能就比较容易理解了，**切点在Spring AOP中确实是对应系统中的方法**。但是这个方法是定义在切面中的方法，一般和通知一起使用，一起组成了切面。

- 连接点（Join point）

  比如：方法调用、方法执行、字段设置/获取、异常处理执行、类初始化、甚至是 for 循环中的某个点

  理论上, 程序执行过程中的任何时点都可以作为作为织入点, 而所有这些执行时点都是 Joint point

  但 Spring AOP 目前仅支持方法执行 (method execution) 也可以这样理解，连接点就是你准备在系统中执行切点和切入通知的地方（一般是一个方法，一个字段）

- 切面（Aspect）

  切面是切点和通知的集合，一般单独作为一个类。通知和切点共同定义了关于切面的全部内容，它是什么时候，在何时和何处完成功能。

- 引入（Introduction）

  引用允许我们向现有的类添加新的方法或者属性

- 织入（Weaving）

  组装方面来创建一个被通知对象。这可以在编译时完成（例如使用AspectJ编译器），也可以在运行时完成。Spring和其他纯Java AOP框架一样，在运行时完成织入。

作者：拥抱心中的梦想

链接：https://juejin.im/post/5aa7818af265da23844040c6

```java
@Aspect
@Component // for auto scan
//@Order(2)
public class LogInterceptor {
	@Pointcut("execution(public * net.aazj.service..*.getUser(..))")
	public void myMethod(){};
	@Before("myMethod()")
	public void before() {
		System.out.println("method start");
	}
	@After("myMethod()")
	public void after() {
		System.out.println("method after");
	}
	@AfterReturning("execution(public * net.aazj.mapper..*.*(..))")
	public void AfterReturning() {
		System.out.println("method AfterReturning");
	}
	@AfterThrowing("execution(public * net.aazj.mapper..*.*(..))")
//  @Around("execution(public * net.aazj.mapper..*.*(..))")
	public void AfterThrowing() {
		System.out.println("method AfterThrowing");
	}
	@Around("execution(public * net.aazj.mapper..*.*(..))")
	public Object Around(ProceedingJoinPoint jp) throws Throwable {
		System.out.println("method Around");
		SourceLocation sl = jp.getSourceLocation();
		Object ret = jp.proceed();
		System.out.println(jp.getTarget());
		return ret;
	}
	@Before("execution(public * net.aazj.service..*.getUser(..)) && args(userId,..)")
	public void before3(int userId) {
		System.out.println("userId-----" + userId);
	}
	@Before("myMethod()")
	public void before2(JoinPoint jp) {
		Object[] args = jp.getArgs();
		System.out.println("userId11111: " + (Integer)args[0]);
		System.out.println(jp.getTarget());
		System.out.println(jp.getThis());
		System.out.println(jp.getSignature());
		System.out.println("method start");
	}
}
```



## 特点

1、降低模块之间的耦合度

2、使系统容易扩展

3、更好的代码复用。

# 实现

##   代理模式（静态代理）

​          静态代理由 **业务实现类、业务代理类** 两部分组成。业务实现类 负责实现主要的业务方法，业务代理类负责对调用的业务方法作**拦截、过滤、预处理**，主要是在方法中首先进行预处理动作，然后调用业务实现类的方法，还可以规定调用后的操作。我们在需要调用业务时，不是直接通过业务实现类来调用的，而是通过**业务代理类的同名方法来调用被代理类处理过的业务方法。**

​          静态代理的实现：

​          1：首先定义一个接口，说明业务逻辑。          

```java
    package net.battier.dao;      
    /** 
     * 定义一个账户接口 
     * @author Administrator
     */  
    public interface Count {  
        // 查询账户
        public void queryCount();  
      
        // 修改账户  
        public void updateCount();  
      
    }  
```

​           2：然后，定义业务实现类，实现业务逻辑接口

```java
import net.battier.dao.Count;    
/** 
 * 委托类(包含业务逻辑) 
 *  
 * @author Administrator 
 *  
 */  
public class CountImpl implements Count {  
  
    @Override  
    public void queryCount() {  
        System.out.println("查看账户...");  
  
    }  
  
    @Override  
    public void updateCount() {  
        System.out.println("修改账户...");  
  
    }  
  
}  
```

​       3：定义业务代理类：通过组合，在代理类中创建一个**业务实现类对象来调用具体的业务方法**；通过实现业务逻辑接口，来统一业务方法；在代理类中实现业务逻辑接口中的方法时，进行预处理操作、通过业务实现类对象调用真正的业务方法、进行调用后操作的定义。

```java
public class CountProxy implements Count {  
    private CountImpl countImpl;  //组合一个业务实现类对象来进行真正的业务方法的调用
  
    /** 
     * 覆盖默认构造器 
     *  
     * @param countImpl 
     */  
    public CountProxy(CountImpl countImpl) {  
        this.countImpl = countImpl;  
    }  
  
    @Override  
    public void queryCount() {  
        System.out.println("查询账户的预处理——————");  
        // 调用真正的查询账户方法
        countImpl.queryCount();  
        System.out.println("查询账户之后————————");  
    }  
  
    @Override  
    public void updateCount() {  
        System.out.println("修改账户之前的预处理——————");  
        // 调用真正的修改账户操作
        countImpl.updateCount();  
        System.out.println("修改账户之后——————————");  
  
    }  
  
}  
```

​       4：在使用时，首先创建业务实现类对象，然后把**业务实现类对象作构造参数创建一个代理类对象**，最后通过代理类对象进行业务方法的调用。

```java
 public static void main(String[] args) {  
        CountImpl countImpl = new CountImpl();  
        CountProxy countProxy = new CountProxy(countImpl);  
        countProxy.updateCount();  
        countProxy.queryCount();  
  
    }  
```

​       静态代理的缺点很明显：**一个代理类只能对一个业务接口的实现类进行包装，如果有多个业务接口的话就要定义很多实现类和代理类才行。**而且，如果代理类对业务方法的预处理、调用后操作都是一样的（比如：调用前输出提示、调用后自动关闭连接），则多个代理类就会有很多重复代码。这时我们可以定义这样一个代理类，它能代理所有实现类的方法调用：根据传进来的业务实现类和方法名进行具体调用。——那就是动态代理。

  

##     动态代理的第一种实现——JDK动态代理

​    JDK动态代理所用到的代理类在程序调用到代理类对象时才由JVM真正创建，JVM根据传进来的 业务实现类对象 以及 方法名 ，**动态地创建了一个代理类的class文件并被字节码引擎执行，然后通过该代理类对象进行方法调用**。我们需要做的，只需指定代理类的预处理、调用后操作即可。

​       1：首先，定义业务逻辑接口

```java
public interface BookFacade {  
    public void addBook();  
} 
```

​       2：然后，实现业务逻辑接口创建业务实现类

```java
public class BookFacadeImpl implements BookFacade {   
    @Override  
    public void addBook() {  
        System.out.println("增加图书方法。。。");  
    }  
  
} 
```

​       3：最后，实现 **调用管理接口InvocationHandle**r  创建动态代理类

```java
public class BookFacadeProxy implements InvocationHandler {  
    private Object target;//这其实业务实现类对象，用来调用具体的业务方法 
    /** 
     * 绑定业务对象并返回一个代理类  
     */  
    public Object bind(Object target) {  
        this.target = target;  //接收业务实现类对象参数

       //通过反射机制，创建一个代理类对象实例并返回。用户进行方法调用时使用
       //创建代理对象时，需要传递该业务类的类加载器（用来获取业务实现类的元数据，在包装方法是调用真正的业务方法）、接口、handler实现类
        return Proxy.newProxyInstance(target.getClass().getClassLoader(),  
                target.getClass().getInterfaces(), this); }  
    /** 
     * 包装调用方法：进行预处理、调用后处理 
     */  
    public Object invoke(Object proxy, Method method, Object[] args)  
            throws Throwable {  
        Object result=null;  
        System.out.println("预处理操作——————");  
        //调用真正的业务方法  
        result=method.invoke(target, args);  
        System.out.println("调用后处理——————");  
        return result;  
    }  
  
}  
```

​       4：在使用时，首先**创建一个业务实现类对象和一个代理类对象，然后定义接口引用**（这里使用向上转型）并用代理对象.bind(业务实现类对象)的返回值进行赋值。最后**通过接口引用调用业务方法即可**。（接口引用真正指向的是一个绑定了业务类的代理类对象，所以通过接口方法名调用的是被代理的方法们）

```java
public static void main(String[] args) {  
        BookFacadeImpl bookFacadeImpl = new BookFacadeImpl();
        BookFacadeProxy proxy = new BookFacadeProxy();  
        BookFacade bookfacade = (BookFacade) proxy.bind(bookFacadeImpl);  
        bookfacade.addBook();  
    }  
```

​        JDK动态代理的代理对象在创建时，需要**使用业务实现类所实现的接口作为参数**（因为在后面代理方法时需要根据接口内的方法名进行调用）。如果业务实现类是没有实现接口而是直接定义业务方法的话，就无法使用JDK动态代理了。并且，如果业务实现类中新增了接口中没有的方法，这些方法是无法被代理的（因为无法被调用）。

##    动态代理的第二种实现——CGlib

​       **cglib是针对类来实现代理的，原理是对指定的业务类生成一个子类，并覆盖其中业务方法实现代理。因为采用的是继承，所以不能对final修饰的类进行代理。** 

​       1：首先定义业务类，无需实现接口（当然，实现接口也可以，不影响的）

```java
public class BookFacadeImpl1 {  
    public void addBook() {  
        System.out.println("新增图书...");  
    }  
}  
```

​       2：实现 MethodInterceptor方法代理接口，创建代理类

```java
public class BookFacadeCglib implements MethodInterceptor {  
    private Object target;//业务类对象，供代理方法中进行真正的业务方法调用
  
    //相当于JDK动态代理中的绑定
    public Object getInstance(Object target) {  
        this.target = target;  //给业务对象赋值
        Enhancer enhancer = new Enhancer(); //创建加强器，用来创建动态代理类
        enhancer.setSuperclass(this.target.getClass());  //为加强器指定要代理的业务类（即：为下面生成的代理类指定父类）
        //设置回调：对于代理类上所有方法的调用，都会调用CallBack，而Callback则需要实现intercept()方法进行拦        enhancer.setCallback(this);        // 创建动态代理类对象并返回         return enhancer.create();     }    // 实现回调方法     public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {         System.out.println("预处理——————");        proxy.invokeSuper(obj, args); //调用业务类（父类中）的方法        System.out.println("调用后操作——————");        return null;     } 
```

​       3：**创建业务类和代理类对象，然后通过代理类对象.getInstance(业务类对象)  返回一个动态代理类对象**（它是业务类的子类，可以用业务类引用指向它）。**最后通过动态代理类对象进行方法调用**。

```java
public static void main(String[] args) {      
        BookFacadeImpl1 bookFacade=new BookFacadeImpl1()；
        BookFacadeCglib  cglib=new BookFacadeCglib();  
        BookFacadeImpl1 bookCglib=(BookFacadeImpl1)cglib.getInstance(bookFacade);  
        bookCglib.addBook();  
    }  
```

##     比较

​    静态代理是通过在代码中显式定义一个业务实现类一个代理，**在代理类中对同名的业务方法进行包装**，用户通过代理类调用被包装过的业务方法；

​    JDK动态代理是**通过接口中的方法名，在动态生成的代理类中调用业务实现类**的同名方法；

​    CGlib动态代理是**通过继承业务类，生成的动态代理类是业务类的子类，通过重写业务方法**进行代理；

 

​    参考资料：http://www.cnblogs.com/jqyp/archive/2010/08/20/1805041.html

​                  http://www.360doc.com/content/14/0801/14/1073512_398598312.shtml