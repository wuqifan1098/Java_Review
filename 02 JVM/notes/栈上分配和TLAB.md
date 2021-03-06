JVM提供了一种叫做栈上分配的概念，针对那些**作用域不会逃逸出方法的对象，在分配内存时不在将对象分配在堆内存中，而是将对象属性打散后分配在栈**（线程私有的，属于栈内存）上，这样，随着方法的调用结束，栈空间的回收就会随着将栈上分配的打散后的对象回收掉，不再给gc增加额外的无用负担，从而提升应用程序整体的性能

### 线程私有分配区TLAB

对象分配在堆上，而堆是一个全局共享的区域，**当多个线程同一时刻操作堆内存分配对象空间时，就需要通过锁机制或者指针碰撞的方式确保不会申请到同一块内存**，而这带来的效果就是对象分配效率变差（尽管JVM采用了CAS的形式处理分配失败的情况），但是对于存在竞争激烈的分配场合仍然会导致效率变差。因此，在Hotspot 1.6的实现中引入了TLAB技术。

TLAB全称ThreadLocalAllocBuffer，是线程的一块私有内存，如果设置了虚拟机参数 -XX:UseTLAB，在线程初始化时，**同时也会申请一块指定大小的内存，只给当前线程使用，这样每个线程都单独拥有一个Buffer，**如果需要分配内存，就在自己的Buffer上分配，这样就不存在竞争的情况，可以大大提升分配效率。