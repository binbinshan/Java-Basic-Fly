# MySQL的 Undo Log(七)
大家应该都知道事务的ACID,即原子性、持久性、隔离性、一致性。
那么MySQL中的原子性是怎么实现的呢？答案：MySQL 使用回滚日志（undo log）实现事务的原子性。

例如执行的是一个INSERT语句，那么undo log中就记录了一条 DELETE;
例如执行的是一个UPDATE语句，那么undo log中就记录了一条 UPDATE之前的值;
例如执行的是一个DELETE语句，那么undo log中就记录了一条 INSERT;

UNDO LOG中分为两种类型:
* 一种是 INSERT_UNDO（INSERT操作），记录插入的唯一键值；
* 一种是 UPDATE_UNDO（包含UPDATE及DELETE操作），记录修改的唯一键值以及old column记录。


### INSERT语句的undo log日志
INSERT语句的undo log的类型是TRX_UNDO_INSERT_REC，这个undo log里包含了以下一些东西：
* 这条日志的开始位置 : undo log日志开始的位置

* 主键的各列长度和值 ： 主键/联合主键
* 表id ： 要插入到哪张表
* undo log日志编号 ：一个事务里会有多个SQL语句，就会有多个undo log日志，在每个事务里的undo log日志的编号都 是从0开始的，然后依次递增。
* undo log日志类型 : TRX_UNDO_INSERT_REC
* 这条日志的结束位置 : undo log日志结束的位置

假设在buﬀer pool的一个缓存页里插入了一条数据了，执行了insert语句，然后写入了undo log，现在事务要求回滚了，直接找到这条insert语句的undo log。 然后在undo log里就知道在哪个表里插入的数据，主键是什么，直接定位到那个表和主键对应的缓存页，从里面删除掉之前insert语句插入进去的数据就可以了，这样就可以实现事务回滚的效果了。


### undo log 不落盘
驻留在全局临时表空间中的撤消日志用于修改用户定义临时表中数据的事务。这些撤消日志不会被重做日志，因为它们不是崩溃恢复所必需的。它们仅用于在服务器运行时回滚。这种类型的撤消日志通过避免重做日志记录 I/O 来提高性能。