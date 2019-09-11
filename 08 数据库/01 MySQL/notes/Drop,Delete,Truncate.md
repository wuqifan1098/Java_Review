# delete

1、delete是DML，执行delete操作时，每次从表中删除一行，并且同时将该行的的删除操作记录在redo和undo表空间中以便进行回滚（rollback）和重做操作，但要注意表空间要足够大，需要手动提交（commit）操作才能生效，可以通过rollback撤消操作。

2、delete可根据条件删除表中满足条件的数据，如果不指定where子句，那么删除表中所有记录。

3、delete语句不影响表所占用的extent，高水线(high watermark)保持原位置不变。

# truncate

1、truncate是DDL，会隐式提交，所以，不能回滚，不会触发触发器。

2、truncate会删除表中所有记录，并且将重新设置高水线和所有的索引，缺省情况下将空间释放到minextents个extent，除非使用reuse storage，。不会记录日志，所以执行速度很快，但不能通过rollback撤消操作（如果一不小心把一个表truncate掉，也是可以恢复的，只是不能通过rollback来恢复）。

3、对于外键（foreignkey ）约束引用的表，不能使用 truncate table，而应使用不带 where 子句的 delete 语句。

4、truncatetable不能用于参与了索引视图的表。

# drop

1、drop是DDL，会隐式提交，所以，不能回滚，不会触发触发器。

2、drop语句删除表结构及所有数据，并将表所占用的空间全部释放。

3、drop语句将删除表的结构所依赖的约束，触发器，索引，依赖于该表的存储过程/函数将保留,但是变为invalid状态。

 区别
1、表和索引所占空间：
　　当表被TRUNCATE 后，这个表和索引所占用的空间会恢复到初始大小；
　　DELETE操作不会减少表或索引所占用的空间；
　　DROP语句将表所占用的空间全释放掉。
　　
2、应用范围：
　　TRUNCATE 只能对table；
　　DELETE可以是table和view。

3、执行速度：
　　drop > truncate > delete

4、delete from删空表后，会保留一个空的页，truncate在表中不会留有任何页。

5、DELETE语句执行删除的过程是每次从表中删除一行，并且同时将该行的删除操作作为事务记录在日志中保存以便进行进行回滚操作。
　　TRUNCATE TABLE 则一次性地从表中删除所有的数据并不把单独的删除操作记录记入日志保存，删除行是不能恢复的。并且在删除的过程中不会激活与表有关的删除触发器。执行速度快。

6、当使用行锁执行 DELETE 语句时，将锁定表中各行以便删除。truncate始终锁定表和页，而不是锁定各行。

7、如果有identity产生的自增id列，delete from后仍然从上次的数开始增加，即种子不变；
　使用truncate删除之后，种子会恢复到初始值。

# 总结

1、在速度上，一般来说，drop> truncate > delete。

2、在使用drop和truncate时一定要注意，虽然可以恢复，但为了减少麻烦，还是要慎重。

3、如果想删除部分数据用delete，注意带上where子句，回滚段要足够大；

   如果想删除表，当然用drop； 

   如果想保留表而将所有数据删除，如果和事务无关，用truncate即可；

   如果和事务有关，或者想触发trigger，还是用delete；

   如果是整理表内部的碎片，可以用truncate跟上reuse stroage，再重新导入/插入数据。