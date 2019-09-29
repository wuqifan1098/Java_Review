spring的启动是建筑在servlet容器之上的，所有web工程的初始位置就是web.xml,它配置了servlet的上下文（context）和监听器（Listener），下面就来看看web.xml里面的配置：

```xml
<!--上下文监听器，用于监听servlet的启动过程-->
<listener>
        <description>ServletContextListener</description>
      <!--这里是自定义监听器，个性化定制项目启动提示-->
        <listener-class>com.trace.app.framework.listeners.ApplicationListener</listener-class>
    </listener>

<!--dispatcherServlet的配置，这个servlet主要用于前端控制，这是springMVC的基础-->
    <servlet>
        <servlet-name>service_dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/spring/services/service_dispatcher-servlet.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

<!--spring资源上下文定义，在指定地址找到spring的xml配置文件-->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring/application_context.xml</param-value>
    </context-param>
<!--spring的上下文监听器-->
    <listener>
        <listener-class>
            org.springframework.web.context.ContextLoaderListener
        </listener-class>
    </listener>

<!--Session监听器，Session作为公共资源存在上下文资源当中，这里也是自定义监听器-->
    <listener>
        <listener-class>
            com.trace.app.framework.listeners.MySessionListener
        </listener-class>
    </listener>
```

接下来就一点的来解析这样一个启动过程。

# 上下文监听器

代码如下：

```xml
<context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring/application_context.xml</param-value>
</context-param>

<listener>
        <listener-class>
            org.springframework.web.context.ContextLoaderListener
        </listener-class>
 </listener>
```

spring的启动其实就是IOC容器的启动过程，通过上述的第一段配置`<context-param>`是初始化上下文，然后通过后一段的的<listener>来加载配置文件，其中调用的spring包中的`ContextLoaderListener`这个上下文监听器，`ContextLoaderListener`是一个实现了`ServletContextListener`接口的监听器，他的父类是 `ContextLoader`，在启动项目时会触发`contextInitialized`上下文初始化方法。下面我们来看看这个方法：

```java
public void contextInitialized(ServletContextEvent event) {
        initWebApplicationContext(event.getServletContext());
}
```

可以看到，这里是调用了父类`ContextLoader`的`initWebApplicationContext(event.getServletContext());`方法，很显然，这是对ApplicationContext的初始化方法，也就是到这里正是进入了springIOC的初始化。

接下来再来看看`initWebApplicationContext`又做了什么工作，先看看代码：

```java
if (servletContext.getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE) != null) {
            throw new IllegalStateException(
                    "Cannot initialize context because there is already a root application context present - " +
                    "check whether you have multiple ContextLoader* definitions in your web.xml!");
        }

        Log logger = LogFactory.getLog(ContextLoader.class);
        servletContext.log("Initializing Spring root WebApplicationContext");
        if (logger.isInfoEnabled()) {
            logger.info("Root WebApplicationContext: initialization started");
        }
        long startTime = System.currentTimeMillis();

        try {
            // Store context in local instance variable, to guarantee that
            // it is available on ServletContext shutdown.
            if (this.context == null) {
                this.context = createWebApplicationContext(servletContext);
            }
            if (this.context instanceof ConfigurableWebApplicationContext) {
                ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) this.context;
                if (!cwac.isActive()) {
                    // The context has not yet been refreshed -> provide services such as
                    // setting the parent context, setting the application context id, etc
                    if (cwac.getParent() == null) {
                        // The context instance was injected without an explicit parent ->
                        // determine parent for root web application context, if any.
                        ApplicationContext parent = loadParentContext(servletContext);
                        cwac.setParent(parent);
                    }
                    configureAndRefreshWebApplicationContext(cwac, servletContext);
                }
            }
            servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);

            ClassLoader ccl = Thread.currentThread().getContextClassLoader();
            if (ccl == ContextLoader.class.getClassLoader()) {
                currentContext = this.context;
            }
            else if (ccl != null) {
                currentContextPerThread.put(ccl, this.context);
            }

            if (logger.isDebugEnabled()) {
                logger.debug("Published root WebApplicationContext as ServletContext attribute with name [" +
                        WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE + "]");
            }
            if (logger.isInfoEnabled()) {
                long elapsedTime = System.currentTimeMillis() - startTime;
                logger.info("Root WebApplicationContext: initialization completed in " + elapsedTime + " ms");
            }

            return this.context;
        }
        catch (RuntimeException ex) {
            logger.error("Context initialization failed", ex);
            servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, ex);
            throw ex;
        }
        catch (Error err) {
            logger.error("Context initialization failed", err);
            servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, err);
            throw err;
        }
```

这个方法还是有点长的，其实仔细看看，出去异常错误处理，这个方法主要做了三件事：

1. 创建WebApplicationContext。
2. 加载对应的spring配置文件中的Bean。
3. 将WebApplicationContext放入ServletContext（Java Web的全局变量）中。

上述代码中`createWebApplicationContext(servletContext)`方法即是完成创建WebApplicationContext工作，也就是说这个方法创建爱你了上下文对象，支持用户自定义上下文对象，但必须继承ConfigurableWebApplicationContext，而Spring MVC默认使用ConfigurableWebApplicationContext作为ApplicationContext（它仅仅是一个接口）的实现。

再往下走，有一个方法`configureAndRefreshWebApplicationContext`就是用来加载spring配置文件中的Bean实例的。这个方法于封装ApplicationContext数据并且初始化所有相关Bean对象。它会从web.xml中读取名为 contextConfigLocation的配置，这就是spring xml数据源设置，然后放到ApplicationContext中，最后调用**传说中的**`refresh`方法执行所有Java对象的创建。

最后完成ApplicationContext创建之后就是将其放入ServletContext中，注意它存储的key值常量。

```css
servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);
```

**总结来说如下图：**

![img](https://upload-images.jianshu.io/upload_images/11798292-651b056425ce045f.JPG?imageMogr2/auto-orient/strip|imageView2/2/w/637/format/webp)

# SpringMVC的启动过程

web.xml的相关配置：

```xml
<servlet>
        <servlet-name>service_dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/spring/services/service_dispatcher-servlet.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
```

这里采用这种自定义初始化参数的配置方式，当然也可以使用默认的。这里Spring Web MVC框架将加载“classpath:service_dispatcher-servlet.xml”来进行初始化上下文而不是“/WEB-INF/[servlet名字]-servlet.xml”。

通过上述配置文件很明显可以看出，springMVC的起始位置是DispatcherServlet（还是spring提供的）:

```java
public class DispatcherServlet extends FrameworkServlet {
          ... ...
}
```

这个类的父类是`FrameworkServlet`，`FrameworkServlet`又继承了`HttpServletBean`类，`HttpServletBean`又继承了`HttpServlet`，`HttpServlet`继承了`GenericServlet`。

```java
public abstract class FrameworkServlet extends HttpServletBean implements ApplicationContextAware {
        ... ...
}
public abstract class HttpServletBean extends HttpServlet
        implements EnvironmentCapable, EnvironmentAware {
      ... ...
}
public abstract class HttpServlet extends GenericServlet
    implements java.io.Serializable
{
          ... ...
}
```

所以在这样一个web容器启动的时候会调用`HttpServletBean`的init方法，这个方法覆盖了GenericServlet中的init方法。让我我们来看看代码：

```java
    @Override
    public final void init() throws ServletException {
        if (logger.isDebugEnabled()) {
            logger.debug("Initializing servlet '" + getServletName() + "'");
        }

        // Set bean properties from init parameters.
        try {
            PropertyValues pvs = new ServletConfigPropertyValues(getServletConfig(), this.requiredProperties);
            BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
            ResourceLoader resourceLoader = new ServletContextResourceLoader(getServletContext());
            bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, getEnvironment()));
            initBeanWrapper(bw);
            bw.setPropertyValues(pvs, true);
        }
        catch (BeansException ex) {
            logger.error("Failed to set bean properties on servlet '" + getServletName() + "'", ex);
            throw ex;
        }

        // Let subclasses do whatever initialization they like.
        initServletBean();

        if (logger.isDebugEnabled()) {
            logger.debug("Servlet '" + getServletName() + "' configured successfully");
        }
    }
```

该初始化方法的主要作用：将Servlet初始化参数（init-param）设置到该组件上（如contextAttribute、contextClass、namespace、contextConfigLocation），通过BeanWrapper简化设值过程，方便后续使用；提供给子类初始化扩展点，`initServletBean()`，该方法由`FrameworkServlet`覆盖。

`FrameworkServlet`继承`HttpServletBean`，通过initServletBean()进行Web上下文初始化，该方法主要覆盖一下两件事情：初始化web上下文；提供给子类初始化扩展点。

```java
@Override
    protected final void initServletBean() throws ServletException {
        getServletContext().log("Initializing Spring FrameworkServlet '" + getServletName() + "'");
        if (this.logger.isInfoEnabled()) {
            this.logger.info("FrameworkServlet '" + getServletName() + "': initialization started");
        }
        long startTime = System.currentTimeMillis();

        try {
            this.webApplicationContext = initWebApplicationContext();
            initFrameworkServlet();
        }
        catch (ServletException ex) {
            this.logger.error("Context initialization failed", ex);
            throw ex;
        }
        catch (RuntimeException ex) {
            this.logger.error("Context initialization failed", ex);
            throw ex;
        }

        if (this.logger.isInfoEnabled()) {
            long elapsedTime = System.currentTimeMillis() - startTime;
            this.logger.info("FrameworkServlet '" + getServletName() + "': initialization completed in " +
                    elapsedTime + " ms");
        }
    }
```

DispatcherServlet继承FrameworkServlet，并实现了onRefresh()方法提供一些前端控制器相关的配置。

整个DispatcherServlet初始化的过程和做了些什么事情，具体主要做了如下两件事情：

1、初始化Spring Web MVC使用的Web上下文，并且指定父容器为WebApplicationContext（ContextLoaderListener加载了的根上下文）；

2、初始化DispatcherServlet使用的策略，如HandlerMapping、HandlerAdapter等。

onRefresh方法代码如下：

```java
@Override
    protected void onRefresh(ApplicationContext context) {
        initStrategies(context);
    }

    /**
     * Initialize the strategy objects that this servlet uses.
     * <p>May be overridden in subclasses in order to initialize further strategy objects.
     */
    protected void initStrategies(ApplicationContext context) {
        initMultipartResolver(context);
        initLocaleResolver(context);
        initThemeResolver(context);
        initHandlerMappings(context);
        initHandlerAdapters(context);
        initHandlerExceptionResolvers(context);
        initRequestToViewNameTranslator(context);
        initViewResolvers(context);
        initFlashMapManager(context);
    }
```

# 总结

1.首先，对于一个web应用，其部署在web容器中，web容器提供其一个全局的上下文环境，这个上下文就是ServletContext，其为后面的spring IoC容器提供宿主环境；

2.其 次，在web.xml中会提供有`contextLoaderListener`。在web容器启动时，会触发容器初始化事件，此时 `contextLoaderListener`会监听到这个事件，其`contextInitialized`方法会被调用，在这个方法中，spring会初始 化一个启动上下文，这个上下文被称为根上下文，即`WebApplicationContext`，这是一个接口类，确切的说，其实际的实现类是 `XmlWebApplicationContext`。这个就是spring的IoC容器，其对应的Bean定义的配置由web.xml中的 context-param标签指定。在这个IoC容器初始化完毕后，spring以`WebApplicationContext.ROOTWEBAPPLICATIONCONTEXTATTRIBUTE`为属性Key，将其存储到ServletContext中，便于获取；

3.再 次，`contextLoaderListener`监听器初始化完毕后，开始初始化web.xml中配置的Servlet，这里是DispatcherServlet，这个servlet实际上是一个标准的前端控制器，用以转发、匹配、处理每个servlet请 求。DispatcherServlet上下文在初始化的时候会建立自己的IoC上下文，用以持有spring mvc相关的bean。在建立DispatcherServlet自己的IoC上下文时，会利用`WebApplicationContext.ROOTWEBAPPLICATIONCONTEXTATTRIBUTE`先从ServletContext中获取之前的根上下文(即WebApplicationContext)作为自己上下文的parent上下文。有了这个 parent上下文之后，再初始化自己持有的上下文。这个DispatcherServlet初始化自己上下文的工作在其initStrategies方 法中可以看到，大概的工作就是初始化处理器映射、视图解析等。这个servlet自己持有的上下文默认实现类也是 `XmlWebApplicationContext`。初始化完毕后，spring以与servlet的名字相关(此处不是简单的以servlet名为 Key，而是通过一些转换，具体可自行查看源码)的属性为属性Key，也将其存到ServletContext中，以便后续使用。这样每个servlet 就持有自己的上下文，即拥有自己独立的bean空间，同时各个servlet共享相同的bean，即根上下文(第2步中初始化的上下文)定义的那些 bean。

https://www.jianshu.com/p/280c7e720d0c