# 面试题

## 1. varchar和char的区别

区别一，定长和变长

char 表示定长，长度固定，varchar表示变长，即长度可变。char如果插入的长度小于定义长度时，则用空格填充；varchar小于定义长度时，还是按实际长度存储，插入多长就存多长。

因为其长度固定，char的存取速度还是要比varchar要快得多，方便程序的存储与查找；但是char也为此付出的是空间的代价，因为其长度固定，所以会占据多余的空间，可谓是以空间换取时间效率。varchar则刚好相反，以时间换空间。

区别之二，存储的容量不同

对 char 来说，**最多能存放的字符个数 255**，和编码无关。
而 varchar 呢，**最多能存放 65532 个字符**。varchar的最大有效长度由最大行大小和使用的字符集确定。整体最大长度是 65,532字节。

原文链接：https://blog.csdn.net/qq_20264581/article/details/83755789

## 2. groupby where having join 执行的顺序（CVTE)

- 先连接from后的数据源(若有join，则先执行on后条件，再连接数据源)。
- 执行where条件
- 执行group by
- 执行having 过滤
- 执行order by
- 输出结果

having是分组（group by）后的筛选条件，分组后的数据组内再筛选
where则是在分组前筛选

where子句中不能使用聚集函数，而having子句中可以，所以在集合函数中加上了HAVING来起到测试查询结果是否符合条件的作用。
即having子句的适用场景是可以使用聚合函数

having 子句限制的是组，而不是行
having 子句中的每一个元素也必须出现在select列表中。有些数据库例外，如oracle

当同时含有 where 子句、group by 子句 、having 子句及聚集函数时，执行顺序如下：
执行where子句查找符合条件的数据；
使用group by 子句对数据进行分组；对group by 子句形成的组运行聚集函数计算每一组的值；最后用having 子句去掉不符合条件的组

https://blog.csdn.net/alice_tl/article/details/88764591

