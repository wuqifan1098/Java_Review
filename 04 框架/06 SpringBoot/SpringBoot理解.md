# 面试题

## 1.SpringBoot的优缺点（贝贝、海康、思必驰）

优点：

1     独立运行的Spring项目：可以以jar包形式独立运行，通过java -jar xx.jar即可运行
2     内嵌Servlet容器：可以选择内嵌Tomcat、Jetty等
3     **提供starter简化maven配置**：一个maven项目，使用了spring-boot-starter-web时，会自动加载Spring Boot的依赖包
4     **自动配置Spring**：Spring Boot会根据在类路径中的jar包、类，为jar包中的类自动配置Bean
5     准生产的应用监控：提供基于http、ssh、telnet对运行时的项目进行监控
6     **无代码生成和xml配置**：主要通过条件注解来实现

缺点：

Spring Boot作为一个微框架，离微服务的实现还是有距离的。springboot 只是**为了提高开发效率**，是为了提升生产力的。

**没有提供相应的服务发现和注册的配套功能**，自身的acturator所提供的监控功能，也需要与现有的监控对接。**没有配套的安全管控方案**，对于REST的落地，还需要自行结合实际进行URI的规范化工作。

## 2. springboot的自动配置关键注解有哪些

每次我们直接直接启动这个**启动类，**SpringBoot就启动成功了，并且帮我们配置了好多自动配置类。

其中最重要是 **@SpringBootApplication** 这个注解

- @SpringBootConfiguration : Spring Boot的配置类，标注在某个类上，表示这是一个Spring Boot的配置类
- @EnableAutoConfiguration: 开启自动配置类，SpringBoot的精华所在。
- @ComponentScan包扫描

　　**以前我们需要配置的东西，Spring Boot帮我们自动配置；@EnableAutoConfiguration告诉SpringBoot开启自动配置功能；这样自动配置才能生效；**

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
```

**两个比较重要的注解**

- @AutoConfigurationPackage：自动配置包
- @Import: 导入自动配置的组件

https://www.cnblogs.com/wenbochang/p/9851314.html

## 3.SpringBoot与SpringMVC的区别（同程艺龙）

Spring MVC 是基于Spring的一个 MVC 框架 ；

Spring Boot 是基于Spring4的条件注册的一套快速开发整合包。

1. 注解和xml配置的使用上，SpringBoot**基本可以用注解实现所有功能。**

2. 启动方式变更，以前的大部分都是打包后放在诸如tomcat之类的容器运行，现在SpringBoot内置了容器，**通过Applicaiton的main函数可以直接运行。**

# 前言

​    关于spring boot，我们肯定听过了很多遍了，其实最本质的东西就是COC（convention over configuration），将各种框架组装起来，省去了我们配置各种框架的时间，是一种更高层次的封装和抽象，正如maven整合了很多的jar包一样，spring boot整合了很多的框架和操作，我们只需要简单的加载进来就可以使用了，这极大地方便了我们的程序开发效率。

# spring boot的使用

##  Spring Boot核心功能

​     独立运行的Spring项目：可以以jar包形式独立运行，通过java -jar xx.jar即可运行
​     内嵌Servlet容器：可以选择内嵌Tomcat、Jetty等
​     提供starter简化maven配置：一个maven项目，使用了spring-boot-starter-web时，会自动加载Spring Boot的依赖包
​     自动配置Spring：Spring Boot会根据在类路径中的jar包、类，为jar包中的类自动配置Bean
​     准生产的应用监控：提供基于http、ssh、telnet对运行时的项目进行监控
​     无代码生成和xml配置：主要通过条件注解来实现

##  Spring Boot项目搭建

　　**这里使用maven进行项目搭建，有几种搭建方式：**

1、http://start.spring.io/，填写相关的项目信息、jdk版本，需要的组件等，就会生成一个maven项目的压缩包，下载解压导入IDE（比如IntelliJ IDEA）就可以。

2、IDE下直接创建，IntelliJ IDEA均支持直接搭建，Spring Tool Suite：新建Spring Initializr项目,填写项目信息和选择技术，将项目设置成maven项目；IntelliJ IDEA：新建Spring Starter project,填写项目信息和选择技术完成maven工程创建。

3、Spring Boot CLI工具，使用命令创建。

4、手工构建maven项目，任意IDE新建空maven项目，修改pom.xml添加Spring Boot的父级依赖Spring-boot-starter-parent，添加之后这个项目就是一个Spring Boot项目了，这也是其他方式生成的本质，引入依赖。

下面我们通过前两种方式来创建工程，首先是第一种，从网址：http://start.spring.io/ 获取工程压缩文件：

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181116142516643-1322307642.png)

   选择使用更多的配置“switch to the full version”，我们可以进行更灵活的配置：

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181116142731458-1893158496.png)

​    **其中我们自己定义工程的名字，以及各种配置，最后我们可以加一点组件进去，比如我们加入web项目：**

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181116142856836-1282238824.png)

  **然后我们点击生成按钮即可，将生成的工程下载下来，然后解压到磁盘上：**

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181116143019406-384459740.png)

   **然后打开intellij IDEA，导入工程，这一步非常的重要，一定要选择pom.xml文件来打开：**

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181116143124013-1281819669.jpg)

​    **选择作为工程文件打开，之后就能自动生成工程了：**

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181116143129153-1075130917.jpg)

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181116143133624-2034714958.jpg)

   **然后我们看到之前已经生成了一个以项目名大写的文件“XXXApplication.java”，这个文件就是程序执行的入口点，我们可以在这个文件下面加一点简单的映射：**

```java
 package com.springboot.zyrboot;
 
 import org.springframework.boot.SpringApplication;
 import org.springframework.boot.autoconfigure.SpringBootApplication;
 import org.springframework.web.bind.annotation.RequestMapping;
 import org.springframework.web.bind.annotation.RestController;
 
 @SpringBootApplication
 @RestController
 public class ZyrbootApplication {
     @RequestMapping(path = {"/helloSpringBoot"})
     public String HelloSpring (){
        System.out.println("hello spring boot");
         return "hello spring boot";
     }
     @RequestMapping("/index")
     public String index (){
        System.out.println("欢迎使用springboot");
        return "欢迎使用springboot";
     }
     public static void main(String[] args) {
        SpringApplication.run(ZyrbootApplication.class, args);
    }
 }
```

  **然后运行main函数，就这样Tomcat就开始运行了，我们根本就没有配置呢，全靠spring boot帮我们做好了这些繁琐的工作：**

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181116143814468-1277962508.png)

  **我们在浏览器上访问,发现已经成功运行了，非常的方便：**

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181116144438964-307103082.png)

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181116144500806-1228780365.png)

**下面讲一下第二种创建工程的方法：**

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181116144643732-943405207.png)

   **可以看到本质上也是从spring的网址上面得到信息的：**

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181116144753853-892013740.png)

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181116144834776-680253714.png)

   **然后一直确定就生成了我们的spring boot工程：**

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181116144951322-652963982.png)

   **剩下的两种方式其实本质上和上面的两种一样的，最后都落实到了pom.xml文件的某些配置上。**

##  **Spring boot工程解析**

   **让我们看看自动生成的pom.xml文件吧，可以看到我们选择的web组件已经加载了，最重要的是<parent>标签里面的东西，这个标签是在配置 Spring Boot 的父级依赖，有了这个当前的项目才是 Spring Boot 项目，spring-boot-starter-parent 是一个特殊的 starter ，它用来提供相关的 Maven 默认依赖，使用它之后，常用的包依赖就可以省去 version 标签。**

```java
 <?xml version="1.0" encoding="UTF-8"?>
 <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
     <modelVersion>4.0.0</modelVersion>

     <groupId>com.example</groupId>
     <artifactId>zyrdemo</artifactId>
     <version>0.0.1-SNAPSHOT</version>
     <packaging>jar</packaging>
 
     <name>zyrdemo</name>
     <description>Demo project for Spring Boot</description>
 
     <parent>
         <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
         <version>2.1.0.RELEASE</version>
         <relativePath/> <!-- lookup parent from repository -->
    </parent>
 
     <properties>
         <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
         <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
         <java.version>1.8</java.version>
     </properties>
 
     <dependencies>
         <dependency>
             <groupId>org.springframework.boot</groupId>
             <artifactId>spring-boot-starter-web</artifactId>
         </dependency>

         <dependency>
             <groupId>org.springframework.boot</groupId>
             <artifactId>spring-boot-starter-test</artifactId>
             <scope>test</scope>
         </dependency>
     </dependencies>
 
     <build>
         <plugins>
             <plugin>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
         </plugins>
    </build>
 
 
 </project>
```

  **有了这个parent依赖和我们简单的web,test和maven插件配置之后，我们可以看到系统自动导入了非常多的外置包：**

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181116145636998-1632102403.png)

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181116145712278-1396876073.png)

应用入口类

​    ***Spring Boot 项目通常有一个名为 \*Application 的入口类，***入口类里有一个 main 方法， 这个 main 方法其实就是一个标准的 Java 应用的入口方法。
***@SpringBootApplication 是 Spring Boot 的核心注解，它是一个组合注解，该注解组合了：@Configuration、@EnableAutoConfiguration、@ComponentScan；*** 若不是用 @SpringBootApplication 注解也可以使用这三个注解代替。其中，***@EnableAutoConfiguration*** 让 Spring Boot 根据类路径中的 jar 包依赖为当前项目进行自动配置，例如，添加了 spring-boot-starter-web 依赖，会自动添加 Tomcat 和 Spring MVC 的依赖，那么 Spring Boot 会对 Tomcat 和 Spring MVC 进行自动配置。Spring Boot 还会自动扫描 @SpringBootApplication 所在类的同级包以及下级包里的 Bean ，所以入口类建议就配置在 grounpID + arctifactID 组合的包名下。

## Spring Boot 的配置文件

​    Spring Boot 使用一个全局的配置文件 application.properties 或 application.yml，放置在src/main/resources目录下。Spring Boot 不仅支持常规的 properties 配置文件，还支持 yaml 语言的配置文件。yaml 是以数据为中心的语言，在配置数据的时候具有面向对象的特征。Spring Boot 的全局配置文件的作用是对一些默认配置的配置值进行修改。比如我们对Tomcat的配置进行更改，然后运行：

```
 server.port=8081
 server.servlet.context-path=/hello
```

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181116151123018-1467602311.png)

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181116151248897-1034111959.png)

  **程序如下：**

```java
 1 package com.example.zyrdemo;
 2 
 3 import org.springframework.boot.SpringApplication;
 4 import org.springframework.boot.autoconfigure.SpringBootApplication;
 5 import org.springframework.web.bind.annotation.RequestMapping;
 6 import org.springframework.web.bind.annotation.RestController;
 7 
 8 @RestController
 9 @SpringBootApplication
10 public class ZyrdemoApplication {
11     @RequestMapping("/hello")
12     public String hello() {
13         return "Hello Spring Boot,I am zyr!";
14     }
15     public static void main(String[] args) {
16         SpringApplication.run(ZyrdemoApplication.class, args);
17     }
18 }
```

  **需要这样才能访问我们的程序了：**



 ![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181116151008783-671006090.png)

​    **另外，难道我们的程序，必须要写道这个“XXXApplication.java”的文件里面吗？肯定不是的，我们在同一包下再加一个文件：**

```java
 1 package com.example.zyrdemo;
 2 import org.springframework.web.bind.annotation.RequestMapping;
 3 import org.springframework.web.bind.annotation.RestController;
 4 
 5 @RestController
 6 public class OtherController {
 7     @RequestMapping("/hello1")
 8     public String hello1() {
 9         return "Hello Spring Boot,I am zyr!OtherController is running";
10     }
11 }
```

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181116151806319-1741084767.png)

  **然后运行一下，发现同样的可以访问了：**

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181116151732994-1165213007.png)

   **但是，如果我们新建一个包，在这个里面放入相应的文件就不能执行了，这一点我们要非常的注意，因为需要我们配置beans扫描了：**

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181116152319510-112751187.png)

![img](https://img2018.cnblogs.com/blog/1157683/201811/1157683-20181116152247909-861016147.png)
