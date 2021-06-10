# MySQL的 Redo Log(六)
大家应该都知道事务的ACID,即原子性、持久性、隔离性、一致性。
那么MySQL中的持久性是怎么实现的呢？答案：MySQL 使用重做日志（redo log）实现事务的持久性。

### 为什么需要Redo Log?
![-w795](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-06-10/16230491866161.jpg)

通过图我们可以知道，为了取得更好的读写性能，Mysql对数据的增删改查是在Buffer Poll中操作的，那么假设事务提交了，但是还未来得及更新磁盘文件，Mysql便宕机了，那么为了保证持久性，就需要一个能记录修改内容的东西，当故障发生导致内存数据丢失后，InnoDB会在重启时，通过重放Redo，将Page恢复到崩溃前的状态。这个就是Redo Log。

为什么不直接将修改内容写入磁盘，而是采用Redo Log呢？
1. 空间：
    * 因为直接将修改内容写入磁盘的话，就需要将一个个的数据页写入磁盘，每个数据页为16k,数据比较大。
    * redo log可能就占据几十个字节，写入速度快。
2. 速度：
    * 将数据页落入磁盘，需要的是随机读写，比较慢
    * Redo Log则采用顺序读写的方式，速度快。

    
### Redo log 
redo log的格式：
日志类型（就是类似MLOG_1BYTE之类的），表空间ID，数据页号，数据页中的偏移量，具体修改的数据

日志类型（就是类似MLOG_WRITE_STRING之类的），表空间ID，数据页号，数据页中的偏移量，修改数据长度，具体修改的数据


### Redo log block
Redo Log中也是有块的概念的，redo log不是一行行单独的直接写入日志文件中的，而是采用了redo log block来存放多个单行日志的。
一个redo log block是512字节，这个redo log block的512字节分为3个部分，一个是12字节的header块头，一个是 496字节的body块体，一个是4字节的trailer块尾。

![-w649](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-06-10/16230513960928.jpg)
所以一个个redo log先写入在内存中redo log block,然后redo log block写满之后，再将其刷新到磁盘上的日志文件中。

### redo log buffer
上面讨论了redo log会写入到redo log block后再把redo log block刷入到磁盘上，那么这个redo log block就是在redo log buffer上。

redo log buffer 会申请一片连续的空间(默认是16MB)，然后在里面划分出N多个redo log block。当你要写一条redo log的时候，就会先从第一个redo log block开 始写入，写满了一个redo log block，就会继续写下一个redo log block。

另外执行一个事务的过程中，每个事务会有多个增删改操作，那么就会有多个redo log，这多个redo log就是一组redo log，其实每次一组redo log都是先在别的地方暂存，然后都执行完了，再把一组redo log给写入到redo log buffer的block里去的。

##### 那什么时候redo log buffer里的redo log block写入磁盘呢？
1. 如果写入redo log buﬀer的日志已经占据了redo log buﬀer总容量的一半了，也就是超过了8MB 的redo log在缓冲里了，此时就会刷入到磁盘文件里去。

2. 一个事务提交的时候，必须把他的那些redo log所在的redo log block都刷入到磁盘文件里去。另外要注意的，是否提交事务就强行把redo log刷入物理磁盘文件中，这个需要设置对应的参数，也有可能刷入os cache。
3. 后台线程定时刷新，有一个后台线程每隔1秒就会把redo log buﬀer里的redo log block刷到磁盘文件里去。
4. MySQL关闭的时候，redo log block都会刷入到磁盘里去

##### 磁盘上的redo log怎么存储？
实际上默认情况下，redo log都会写入一个目录中的文件里，这个目录 可以通过show variables like 'datadir'来查看，可以通过innodb_log_group_home_dir参数来设置这个目录的。

通过innodb_log_ﬁle_size可以指定每个redo log文件的大小，默认是48MB，通过 innodb_log_ﬁles_in_group可以指定日志文件的数量，默认就2个。
所以默认情况下，目录里就两个日志文件，分别为ib_logﬁle0和ib_logﬁle1，每个48MB，最多就这2个 日志文件，就是先写第一个，写满了写第二个。然后覆盖第一个继续写，重复往返。

# MySQL的 Redo Log(六)
大家应该都知道事务的ACID,即原子性、持久性、隔离性、一致性。
那么MySQL中的持久性是怎么实现的呢？答案：MySQL 使用重做日志（redo log）实现事务的持久性。

### 为什么需要Redo Log?
![-w795](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-06-10/16230491866161.jpg)

通过图我们可以知道，为了取得更好的读写性能，Mysql对数据的增删改查是在Buffer Poll中操作的，那么假设事务提交了，但是还未来得及更新磁盘文件，Mysql便宕机了，那么为了保证持久性，就需要一个能记录修改内容的东西，当故障发生导致内存数据丢失后，InnoDB会在重启时，通过重放Redo，将Page恢复到崩溃前的状态。这个就是Redo Log。

为什么不直接将修改内容写入磁盘，而是采用Redo Log呢？
1. 空间：
    * 因为直接将修改内容写入磁盘的话，就需要将一个个的数据页写入磁盘，每个数据页为16k,数据比较大。
    * redo log可能就占据几十个字节，写入速度快。
2. 速度：
    * 将数据页落入磁盘，需要的是随机读写，比较慢
    * Redo Log则采用顺序读写的方式，速度快。

    
### Redo log 
redo log的格式：
日志类型（就是类似MLOG_1BYTE之类的），表空间ID，数据页号，数据页中的偏移量，具体修改的数据

日志类型（就是类似MLOG_WRITE_STRING之类的），表空间ID，数据页号，数据页中的偏移量，修改数据长度，具体修改的数据


### Redo log block
Redo Log中也是有块的概念的，redo log不是一行行单独的直接写入日志文件中的，而是采用了redo log block来存放多个单行日志的。
一个redo log block是512字节，这个redo log block的512字节分为3个部分，一个是12字节的header块头，一个是 496字节的body块体，一个是4字节的trailer块尾。

![-w649](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-06-10/16230513960928.jpg)
所以一个个redo log先写入在内存中redo log block,然后redo log block写满之后，再将其刷新到磁盘上的日志文件中。

### redo log buffer
上面讨论了redo log会写入到redo log block后再把redo log block刷入到磁盘上，那么这个redo log block就是在redo log buffer上。

redo log buffer 会申请一片连续的空间(默认是16MB)，然后在里面划分出N多个redo log block。当你要写一条redo log的时候，就会先从第一个redo log block开 始写入，写满了一个redo log block，就会继续写下一个redo log block。

另外执行一个事务的过程中，每个事务会有多个增删改操作，那么就会有多个redo log，这多个redo log就是一组redo log，其实每次一组redo log都是先在别的地方暂存，然后都执行完了，再把一组redo log给写入到redo log buffer的block里去的。

##### 那什么时候redo log buffer里的redo log block写入磁盘呢？
1. 如果写入redo log buﬀer的日志已经占据了redo log buﬀer总容量的一半了，也就是超过了8MB 的redo log在缓冲里了，此时就会刷入到磁盘文件里去。

2. 一个事务提交的时候，必须把他的那些redo log所在的redo log block都刷入到磁盘文件里去。另外要注意的，是否提交事务就强行把redo log刷入物理磁盘文件中，这个需要设置对应的参数，也有可能刷入os cache。
3. 后台线程定时刷新，有一个后台线程每隔1秒就会把redo log buﬀer里的redo log block刷到磁盘文件里去。
4. MySQL关闭的时候，redo log block都会刷入到磁盘里去

##### 磁盘上的redo log怎么存储？
实际上默认情况下，redo log都会写入一个目录中的文件里，这个目录 可以通过show variables like 'datadir'来查看，可以通过innodb_log_group_home_dir参数来设置这个目录的。

通过innodb_log_ﬁle_size可以指定每个redo log文件的大小，默认是48MB，通过 innodb_log_ﬁles_in_group可以指定日志文件的数量，默认就2个。
所以默认情况下，目录里就两个日志文件，分别为ib_logﬁle0和ib_logﬁle1，每个48MB，最多就这2个 日志文件，就是先写第一个，写满了写第二个。然后覆盖第一个继续写，重复往返。

