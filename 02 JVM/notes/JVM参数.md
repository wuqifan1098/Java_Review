# JVM 参数

## GC 优化配置

| 配置 | 描述 |
| --- | --- |
| -Xms | 初始化堆内存大小 |
| -Xmx | 堆内存最大值 |
| -Xmn | 新生代大小 |
| -XX:PermSize | 初始化永久代大小 |
| -XX:MaxPermSize | 永久代最大容量 |

## GC 类型设置

| 配置 | 描述 |
| --- | --- |
| -XX:+UseSerialGC | 串行垃圾回收器 |
| -XX:+UseParallelGC | 并行垃圾回收器 |
| -XX:+UseConcMarkSweepGC | 并发标记扫描垃圾回收器 |
| -XX:ParallelCMSThreads= | 并发标记扫描垃圾回收器 = 为使用的线程数量 |
| -XX:+UseG1GC | G1 垃圾回收器 |

```java
java -Xmx12m -Xms3m -Xmn1m -XX:PermSize=20m -XX:MaxPermSize=20m -XX:+UseSerialGC -jar java-application.jar
```