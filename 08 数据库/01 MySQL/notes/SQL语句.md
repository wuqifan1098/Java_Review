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

https://blog.csdn.net/alice_tl/article/details/88764591

