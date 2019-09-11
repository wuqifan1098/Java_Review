## 什么是bean post processor？

bean生命周期始于加载bean的定义。通过拿到的这个定义，Spring可以构造出(`construct`嘛)bean并注入组件(因为我们常用的就是在controller里 service里使用)。之后，所有的bean都可以进行**后置处理**。这意味着我们可以实现一些自定义逻辑并调用它。并在调用bean的初始化方法(xml配置所定义的init-method 属性)之前和/或之后进行调用(当然默认的上下文环境是Spring容器)。

你不能为给定的bean类型明确指定一个bean后置处理器。每个定义的后处理器可以应用于`application context`中的所有定义的bean。后置处理器bean必须实现**org.springframework.beans.factory.config.BeanPostProcessor**接口并定义`postProcessBeforeInitialization`和`postProcessAfterInitialization`方法。第一个在调用初始化方法(init-method所指定的方法)之前被调用，第二个在调用初始化方法之后被调用。这两个方法都有两个参数：

- Object：表示已处理的bean的实例。
- 字符串：包含已处理的bean的名称。

http://www.iocoder.cn/Spring/Bean-post-processors/