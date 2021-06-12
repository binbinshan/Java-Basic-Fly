# MySQL中的RR是否解决了幻读（十）

之前有人反馈过这个问题 https://bugs.mysql.com/bug.php?id=63870
这里是Mysql官方给出的回复https://dev.mysql.com/doc/refman/8.0/en/innodb-consistent-read.html
从回复中Mysql官方人员认为连续的快照读或者连续的当前读出现数据不一致才符合幻读的定义，而这里出现问题的是先快照读然后当前读，所以他们是这么说的：To prevent phantoms, InnoDB uses an algorithm called next-key locking that combines index-row locking with gap locking。(为了防止出现“幻读”，InnoDB使用了一种名为next-key locking的算法，它结合了索引行锁和间隙锁。)

所以情况如下：
* 对于连续的快照读，mvcc会保证其他事务的修改在当前事务不可见；

* 对于连续的当前读，第一个当前读会加间隙锁，别的事务要修改直接就阻塞了。
* 在一个事务里先快照读再当前读，由于第一个快照读mvcc没有加锁，其他事务可以修改并提交，后面的当前读在设计上就可以读到已提交的事务，update之后状态变成了自己的修改，mvcc里自己的修改是可见的，这条记录就完全可见了。

所以最起码MySQL官方是认为InnoDB中的RR是解决了幻读的。但是需要注意的是MySQL 是对标准sql事务隔离级别进行实现。「当前读」和「快照读」是 MySQL 用于实现事务隔离级别 MVCC 机制中衍生出的概念，而非标准定义的概念。


### 概念
在进行进一步分析的时候，我们先理清几个概念：

##### 幻读和不可重复读区别
幻读和不可重复读的概念很容易弄混，因为两者很相似，但不可重复读重点在于update和delete，而幻读的重点在于insert。

如果使用锁机制来实现这两种隔离级别，在可重复读中，该sql第一次读取到数据后，就将这些数据加锁，其它事务无法修改这些数据，就可以实现可重复读了。
但这种方法却无法锁住insert的数据，所以当事务A先前读取了数据，或者修改了全部数据，事务B还是可以insert数据提交，这时事务A就会发现莫名其妙多了一条之前没有的数据，这就是幻读，不能通过行锁来避免。需要Serializable隔离级别 ，读用读锁，写用写锁，读锁和写锁互斥，这么做可以有效的避免幻读、不可重复读、脏读等问题，但会极大的降低数据库的并发能力。

所以说不可重复读和幻读最大的区别，就在于如何通过锁机制来解决他们产生的问题。

上面说的是使用悲观锁机制来处理这两种问题，但是MySQL、ORACLE、PostgreSQL等成熟的数据库，出于性能考虑，都是使用了以乐观锁为理论基础的MVCC（多版本并发控制）来避免这两种问题。

##### 当前读和快照读 
快照读：在RR级别中，通过MVCC机制，虽然让数据变得可重复读，但我们读到的数据可能是历史数据，这就是快照读。
当前读：读到的都是数据库最新的数据。

快照读：就是select
* select * from table ….;

当前读：特殊的读操作，插入/更新/删除操作，属于当前读，处理的都是当前的数据，需要加锁。
* select * from table where ? lock in share mode;
* select * from table where ? for update;
* insert;
* update ;
* delete;

### 深入解析是否解决

再了解了基本概念之后，再来看下，为什么要区分当前读和快照读呢？

情况1：事务A开启了一个事务后，进行了两次select，这个时候都是使用快照读，那么通过结果我们可以看出确实没有查出修改后的数据，这说明RR级别下，避免了**不可重复读**问题。

| 事务A | 事务B |
| --- | --- |
| BEGIN; | BEGIN; |
| SELECT * from `user`; 结果：id=13 nick_name == ‘aa' |  |
|  | UPDATE `user` set nick_name = 'bb' where id = 13; |
|  | COMMIT; |
| SELECT * from `user`; 结果：id=13 nick_name == ‘aa' |  |
| COMMIT; |  |

情况2：事务A开启了一个事务后，进行了两次select，这个时候都是使用快照读，那么通过结果我们可以看出确实没有查出修改后的数据，这说明InnoDB下RR级别下快照读，避免了**幻读**问题。

| 事务A | 事务B |
| --- | --- |
| BEGIN; | BEGIN; |
| SELECT * from `user`; 结果：id=13 nick_name == ‘aa' |  |
|  | INSERT INTO `user` (`id`,`nick_name`, ) VALUES (15, 'cc', ); |
|  | COMMIT; |
| SELECT * from `user`; 结果：id=13 nick_name == ‘aa' ROW=1 |  |
| COMMIT; |  |

情况3：事务A开启了一个时候后，进行了一次select,进行了一次update,这个时候select使用快照读,而update则使用当前读，那么这种情况（「快照读」和「当前读」一起使用）下就会出现**幻读**。

| 事务A | 事务B |
| --- | --- |
| BEGIN; | BEGIN; |
| SELECT * from `user`; 结果：id=13 nick_name == ‘aa' |  |
|  | INSERT INTO `user` (`id`,`nick_name`, ) VALUES (15, 'cc', ); |
|  | COMMIT; |
|  update `user` set nick_name = 'dd’ ; 结果：Affected rows: 2|  |
| SELECT * from `user`; 结果：id=13 nick_name = ‘aa’ id = 15 nick_name = 15 |  |
| COMMIT; |  |

情况4：针对上面的情况，我们调整下代码，将事务B的插入放在事务UPDATE后面，那么这种情况就没有发生幻读，这是因为当前读加了加 next-key lock，这样事务B就会一直阻塞到事务A提交。

| 事务A | 事务B |
| --- | --- |
| BEGIN; |  |
| SELECT * from `user`; 结果：id=13 nick_name == ‘aa' |  |
| update user set nick_name = 'dd’ ; 结果：Affected rows: 1 |  |
|  | BEGIN; |
|  | INSERT INTO user (id,nick_name, ) VALUES (15, 'cc', ); |
| SELECT * from `user`; 结果：id=13 nick_name = ‘dd’  | Wait  |
| COMMIT; |  |
|  | COMMIT; |

情况5：上面情况说的是当前读加了Next-Key锁，我们也可以自己手动给select加next-key锁，这样也不会出现幻读；

| 事务A | 事务B |
| --- | --- |
| BEGIN; |  |
| SELECT * from `user` for update; 结果：id=13 nick_name == ‘aa' |  |
|  | BEGIN; |
|  | INSERT INTO user (id,nick_name, ) VALUES (15, 'cc', );  |
| SELECT * from `user`; 结果：id=13 nick_name = ‘aa’  | Wait |
| COMMIT; |  |
|  | COMMIT; |

总结一下：快照读的时候无需任何操作即可避免幻读，当快照读和当前读混合使用的使用就需要按照实际情况显式加锁去解决幻读。
