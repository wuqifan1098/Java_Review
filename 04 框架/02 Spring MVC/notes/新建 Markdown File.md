# Spring MVC

Spring MVC是Spring中的基础 Web 框架，基于模型-视图-控制器（Model-View-Controller，[MVC](https://zh.wikipedia.org/wiki/MVC)）模式实现，它能够帮你构建像Spring框架那样灵活和松耦合的Web应用程序。
在该框架下，一次web请求大致可以分为如下图几个步骤，这些划分分离了职责，使得代码灵活、维护性更好。

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/DispatchServlet.png)

为了使用该框架，我们**首先要配置DispatchServlet，也就是前端控制器，然后启用Spring MVC，并编写控制器，视图，模型等等。**
其中，DispatcherServlet是Spring MVC的核心，DispatcherServlet启动的时候，它会创建Spring应用上下文，并加载配置文件或配置类中所声明的bean或者自动扫描的bean，但是在Spring Web应用中，通常还会有另外一个应用上下文，这个应用上下文是由ContextLoaderListener创建的。DispatcherServlet加载包含Web组件的bean，如控制器、视图解析器以及处理器映射，而ContextLoaderListener要加载应用中的其他bean，通常是驱动应用后端的中间层和数据层组件。

Spring MVC是一个强大灵活的Web框架。借助于注解，Spring MVC提供了近似于POJO的开发模式，这使得开发处理请求的控制器变得非常简单，同时也易于测试。而且Spring MVC还支持多种视图解析器如JSP，Tiles，Thymeleaf，使得前端界面的功能更强大，编写更容易。