# 一条update sql的执行


一条update sql的执行：

## 缓冲池(Buffer Poll)
Buffer Poll是InnoDB引擎中放在内存中的一个重要组件，会缓存很多数据，便于查询时使用。
![-w281](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-05-31/16224445118117.jpg)
引擎要执行更新语句的时候 ，比如对“id=10”这一行数据进行更新，他会先判断“id=10”这一行数据看看是否在缓冲池(Buffer Poll)里，如果不在的话，那么会直接从磁盘里加载到Buffer Poll里来，而且接着还会对Buffer Poll中的这行记录加独占锁。


## 回滚日志(undo Log）
当进行一次事务时，如果需要回滚的时候，就需要undo Log。比如对”id=10“的这一行数据将name = "z" 改为 name ="l" ,那么在更新前就会把原来的值“z”和“id=10”这些信息，写入到undo日志文件中去。
![-w510](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-05-31/16224455126311.jpg)


## 重做日志缓冲(Redo Log Buffer)
当修改了Buffer Poll中的数据，但是磁盘还未修改，此时宕机的情况。就必须把对内存的修改放到Redo Log Buffer中，也是内存的一个缓冲区，用于存放Redo日志。
所谓的redo日志，就是记录了对数据做了什么修改，比如对“id=10这行记录修改了name字段的值为xxx”
![-w607](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-05-31/16224464882908.jpg)

## 提交事务
当执行后需要提交事务的时候，此时就会根据一定的策略把redo日志从redo log buffer里刷入到磁盘文件里去。

innodb_flush_log_at_trx_commit:
* 0 : 提交事务的时候，不会把redo log buffer里的数据刷入磁盘文件中，如果提交事务时mysql宕机了，那么此时内存里的redo log数据全部丢失。
* 1 : 提交事务的时候，强制把redo log从redo log buffer刷入到磁盘文件里去。(通常设置)
* 2 : 提交事务的时候，把redo日志写入磁盘文件对应的os cache缓存里去，而不是直接进入磁盘文件，可 能1秒后才会把os cache里的数据写入到磁盘文件里去。 

![-w605](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-05-31/16224471756626.jpg)
假设mysql处于上述步骤的时候崩溃了，那么也不影响，因为redo log已经记录了更新做了什么操作，重启后会自动刷redo log进行恢复。


### 归档日志（binlog）
binlog叫做归档日志，记录的是偏向于逻辑性的日志，类似于“对users表中的id=10的一行数据做了更新操作，更新以后的值是什么”，binlog不是InnoDB存储引擎特有的日志文件，是属于mysql server自己的日志文件。
binlog日志的刷盘策略：sync_binlog
* 0 ： 默认值，不是直接进入磁盘文件，而是进入os cache内存缓存，可能过一会儿才能进入磁盘。
* 1 ： 强制在提交事务的时候，把binlog直接写入到磁盘文件里。
![-w811](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-05-31/16224477514277.jpg)




redo日志中写入commit，就是用来保持redo log日志与binlog日志一致的
![-w767](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-05-31/16224492482291.jpg)


## 刷新磁盘文件
MySQL有一个后台的IO线程，会在之后某个时间里，随机的把内存buffer pool中的修改后的数据给刷回到磁盘上的数据文件里去。在数据刷回磁盘之前，哪怕mysql宕机崩溃也没关系，因为重启之后，会根据redo日志恢复之前提交事务做过的修改到内存里去，等适当时机，IO线程自然还是会把这个修改 后的数据刷到磁盘上的数据文件里去的
![-w795](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-05-31/16224495940485.jpg)

