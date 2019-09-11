# 面试题

## 1.过滤器与拦截器的区别

- 拦截器是基于java反射机制来实现的，而过滤器是基于函数回调来实现的。（有人说，拦截器是基于动态代理来实现的）
- 拦截器**不依赖servlet容器**，过滤器**依赖于servlet容器。**
- 拦截器只对Action起作用，过滤器可以对所有请求起作用。
- 拦截器可以访问Action上下文和值栈中的对象，过滤器不能。
- 在Action的生命周期中，拦截器可以多次调用，而过滤器只能在容器初始化时调用一次。

# 什么是Spring中的处理程序拦截器？

要了解Spring拦截器的作用，我们需要先解释一下HTTP请求的执行链。DispatcherServlet捕获每个请求。调度员做的第一件事就是将接收到的URL和相应的controller进行映射(controller必须恰到好处地处理当前的请求)。但是，在到达对应的controller之前，请求可以被拦截器处理。这些拦截器就像过滤器。只有当URL找到对应于它们的映射时才调用它们。在通过拦截器(拦截器预处理，其实也可以说前置处理)进行前置处理后，请求最终到达controller。之后，发送请求生成视图。但是在这之前，拦截器还是有可能来再次处理它(拦截器后置处理)。只有在最后一次操作之后，视图解析器才能捕获数据并输出视图。

处理程序映射拦截器基于**org.springframework.web.servlet.HandlerInterceptor**接口。和之前简要描述的那样，它们可以在将其发送到控制器(方法前使用**preHandle**)之前或之后(方法后使用**postHandle**)拦截请求。`preHandle`方法返回一个`布尔值`，如果返回`false`，则可以在执行链中`执行中断请求处理`。此接口中还有一个方法**afterCompletion**，只有在`preHandler`方法发送为true时才会在渲染视图后调用它(完成请求处理后的回调，即渲染视图后)。

拦截器也可以在新线程中启动。在这种情况下，拦截器必须实现**org.springframework.web.servlet.AsyncHandlerInterceptor**接口。它继承`HandlerInterceptor`并提供一个方法**afterConcurrentHandlingStarted**。每次处理程序得到正确执行时，都会调用此方法而不是调用`postHandler()`和`afterCompletion()`。它也可以对发送请求进行异步处理。通过Spring源码此方法注释可以知道，这个方法的典型的应用是可以用来清理本地线程变量

# 拦截器和过滤器之间的区别

拦截器看起来很像servlet过滤器，为什么Spring不采用默认的Java解决方案？这其中主要区别就是`两者的作用域`的问题。**过滤器只能在servlet容器下使用**。而我们的**Spring容器不一定运行在web环境中**，在这种情况下过滤器就不好使了，而拦截器依然可以在Spring容器中调用。

Spring通过拦截器为请求提供了一个更细粒度的控制。就像我们之前看到的那样，它们可以在controller对请求处理之前或之后被调用，也可以在将渲染视图呈现给用户之后被调用。如果是过滤器的话，只能在将响应返回给最终用户之前使用它们。

下一个不同之处在于中断链执行的难易程度。拦截器可以通过在`preHandler()`方法内返回`false`来简单实现。而在过滤器的情况下，它就变得复杂了，因为它必须处理请求和响应对象来引发中断，需要一些额外的动作，比如如将用户重定向到错误页面。

# 什么是默认的Spring拦截器？

Spring主要将拦截器用于切换操作。比如我们最常用的功能之一是区域设置更改(也就是本地化更改)。请查看**org.springframework.web.servlet.i18n.LocaleChangeInterceptor**类中源码，可以通过我们所定义的语言环境解析器来对HTTP请求进行分析来实现。所有区域设置解析器都会分析请求元素(headers，Cookie)，以确定向用户提供哪种本地化语言设置。

另一个本地拦截器是**org.springframework.web.servlet.theme.ThemeChangeInterceptor**，它允许更改视图的主题(见此类的注释)。它还使用主题解析器更精确地来知道要使用的主题(参照下面**preHandle**方法)。它的解析器也基于请求分析(cookie，会话或参数)。

```java
/**
 * Interceptor that allows for changing the current theme on every request,
 * via a configurable request parameter (default parameter name: "theme").
 *
 * @author Juergen Hoeller
 * @since 20.06.2003
 * @see org.springframework.web.servlet.ThemeResolver
 */
public class ThemeChangeInterceptor extends HandlerInterceptorAdapter {

	/**
	 * Default name of the theme specification parameter: "theme".
	 */
	public static final String DEFAULT_PARAM_NAME = "theme";

	private String paramName = DEFAULT_PARAM_NAME;


	/**
	 * Set the name of the parameter that contains a theme specification
	 * in a theme change request. Default is "theme".
	 */
	public void setParamName(String paramName) {
		this.paramName = paramName;
	}

	/**
	 * Return the name of the parameter that contains a theme specification
	 * in a theme change request.
	 */
	public String getParamName() {
		return this.paramName;
	}


	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws ServletException {

		String newTheme = request.getParameter(this.paramName);
		if (newTheme != null) {
			ThemeResolver themeResolver = RequestContextUtils.getThemeResolver(request);
			if (themeResolver == null) {
				throw new IllegalStateException("No ThemeResolver found: not in a DispatcherServlet request?");
			}
			themeResolver.setThemeName(request, response, newTheme);
		}
		// Proceed in any case.
		return true;
	}

}
```

# 在Spring中自定义处理程序拦截器

我们如果在项目中使用了Spring框架，那么，我们可以直接继承HandlerInterceptorAdapter.java这个抽象类，来实现我们自己的拦截器。

**preHandle**：预处理回调方法，实现处理器的预处理（如登录检查），第三个参数为响应的处理器（如我们上一章的Controller实现）；

`    ` 返回值：true表示继续流程（如调用下一个拦截器或处理器）；

​             false表示流程中断（如登录检查失败），不会继续调用其他的拦截器或处理器，此时我们需要通过response来产生响应；

**postHandle**：后处理回调方法，实现处理器的后处理（但在渲染视图之前），此时我们可以通过modelAndView（模型和视图对象）对模型数据进行处理或对视图进行处理，modelAndView也可能为null。

**afterCompletion**：整个请求处理完毕回调方法，即在视图渲染完毕时回调，如性能监控中我们可以在此记录结束时间并输出消耗时间，还可以进行一些资源清理，类似于try-catch-finally中的finally，但仅调用处理器执行链中**preHandle**返回true的拦截器的afterCompletion。

`有时候我们可能只需要实现三个回调方法中的某一个，如果实现`HandlerInterceptor接口的话，三个方法必须实现，不管你需不需要，此时**spring提供了一个HandlerInterceptorAdapter适配器（一种适配器设计模式的实现），按需覆写里面的preHandle和postHandle方法即可。**

```java
public abstract class HandlerInterceptorAdapter implements HandlerInterceptor {  
     //省略代码 此处所以三个回调方法都是空实现，preHandle返回true。  
}   
```

```java
package org.springframework.web.servlet.handler;  
import javax.servlet.http.HttpServletRequest;  
import javax.servlet.http.HttpServletResponse;  
import org.springframework.web.servlet.HandlerInterceptor;  
import org.springframework.web.servlet.ModelAndView;  
public abstract class HandlerInterceptorAdapter implements HandlerInterceptor{  
    // 在业务处理器处理请求之前被调用  
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception{  
        return true;  
    }  
    // 在业务处理器处理请求完成之后，生成视图之前执行  
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView)  
      throws Exception{  
    }  
    // 在DispatcherServlet完全处理完请求之后被调用，可用于清理资源  
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)  
      throws Exception{  
    }  
}  
```

接下来我们看一下Spring框架实现的一个简单的拦截器UserRoleAuthorizationInterceptor，UserRoleAuthorizationInterceptor继承了抽象类HandlerInterceptorAdapter，实现了用户登录认证的拦截功能，如果当前用户没有通过认证，会报403错误。

```java
package org.springframework.web.servlet.handler;  
import java.io.IOException;  
import javax.servlet.ServletException;  
import javax.servlet.http.HttpServletRequest;  
import javax.servlet.http.HttpServletResponse;  
public class UserRoleAuthorizationInterceptor extends HandlerInterceptorAdapter{  
    // 字符串数组，用来存放用户角色信息  
    private String[] authorizedRoles;  
    public final void setAuthorizedRoles(String[] authorizedRoles){  
        this.authorizedRoles = authorizedRoles;  
    }  
    public final boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)  
      throws ServletException, IOException{  
        if (this.authorizedRoles != null) {  
            for (int i = 0; i < this.authorizedRoles.length; ++i) {  
                if (request.isUserInRole(this.authorizedRoles[i])) {  
                    return true;  
                }  
            }  
        }  
        handleNotAuthorized(request, response, handler);  
        return false;  
    }  
    protected void handleNotAuthorized(HttpServletRequest request, HttpServletResponse response, Object handler)  
      throws ServletException, IOException{  
          // 403表示资源不可用。服务器理解用户的请求，但是拒绝处理它，通常是由于权限的问题  
          response.sendError(403);  
    }  
}  
```

 下面，我们利用Spring框架提供的HandlerInterceptorAdapter抽过类，来实现一个自定义的拦截器。我们这个拦截器叫做
UserLoginInterceptorBySpring，进行登录拦截控制。工作流程是这样的：如果当前用户没有登录，则跳转到登录页面；登录成功后，跳转到之前访问的URL页面。

```java
import java.util.HashMap;  
import java.util.Map;  
import javax.servlet.http.HttpServletRequest;  
import javax.servlet.http.HttpServletResponse;  
import org.springframework.web.servlet.ModelAndView;  
import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;  
/** 
 * @description 利用spring框架提供的HandlerInterceptorAdapter，实现自定义拦截器 
 */  
public class UserLoginInterceptorBySpring extends HandlerInterceptorAdapter{  
    // 在业务处理器处理请求之前被调用  
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception{  
        // equalsIgnoreCase 与 equals的区别？  
        if("GET".equalsIgnoreCase(request.getMethod())){  
            //RequestUtil.saveRequest();  
        }  
        System.out.println("preHandle...");  
        String requestUri = request.getRequestURI();  
        String contextPath = request.getContextPath();  
        String url = requestUri.substring(contextPath.length());  
        System.out.println("requestUri" + requestUri);  
        System.out.println("contextPath" + contextPath);  
        System.out.println("url" + url);  
        String username = (String) request.getSession().getAttribute("username");  
        if(null == username){  
            // 跳转到登录页面  
            request.getRequestDispatcher("/WEB-INF/login.jsp").forward(request, response);  
            return false;  
        }  
        else{  
            return true;  
        }  
    }  
    // 在业务处理器处理请求完成之后，生成视图之前执行  
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception{  
        System.out.println("postHandle...");  
        if(modelAndView != null){  
            Map<String, String> map = new HashMap<String, String>();  
            modelAndView.addAllObjects(map);  
        }  
    }  
    // 在DispatcherServlet完全处理完请求之后被调用，可用于清理资源  
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception{  
        System.out.println("afterCompletion...");  
    }  
```

 拦截器是依赖Java反射机制来实现的。拦截器的实现，用到的是JDK实现的动态代理，我们都知道，JDK实现的动态代理，需要依赖接口。拦截器是在面向切面编程中应用的，就是在你的service或者一个方法前调用一个方法，或者在方法后调用一个方法。

https://www.cnblogs.com/jpfss/p/8072724.html