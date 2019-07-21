# 面试题

## 1.请介绍一下Mybatis的两级缓存（cvte）



## 2. 请说出Mybatis的${}和#{}的区别（cvte）



## 3. 什么是SQL注入。（cvte）

# 一、Mybatis介绍

MyBatis是一个支持**普通SQL查询**，**存储过程**和**高级映射**的优秀**持久层框架**。MyBatis消除了几乎所有的JDBC代码和参数的手工设置以及对结果集的检索封装。MyBatis可以使用简单的**XML或注解**用于配置和原始映射，将接口和Java的POJO（Plain Old Java Objects，普通的Java对象）映射成数据库中的记录。

## MyBatis的缓存

MyBatis的缓存分为一级缓存和二级缓存,
一级缓存放在session里面,默认就有,二级缓存放在它的命名空间里,默认是打开的,
使用二级缓存属性类需要实现Serializable序列化接口(可用来保存对象的状态),可在它的映射文件中配置

