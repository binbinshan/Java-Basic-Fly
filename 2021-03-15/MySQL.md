# MySQL有哪些存储引擎? (MyISAM、InnoDB)

mysql支持的存储引擎有很多种,比如MyISAM、InnoDB,目前国内基本上都是使用InnoDB,而且这个也是mysql 5.5之后的默认存储引擎。

## InnoDB
为什么都会使用InnoDB呢?
主要是InnoDB生态太好了，它支持事务，走聚簇索引，强制主键，支持外键，另外针对高可用可以做主备切换，针对高并发可以做读写分离，针对大数据量可以做分库分表。

## MyIasm
MyIasm主要是不支持事务，不支持外键约束，索引和数据文件分开，所以内存里可以放更多的缓存，对查询的性能会更好，适用少量插入，大量查询的场景。
就比如报表系统，每天都会产生T+1的数据然后插入到mysql中，之后就是纯查询了，这样的场景就不需要事务，只需要明天跑批产生数据插入mysql即可。但是如果数据量过大的话还是建议用kylin或者elasticsearch，不建议用mysql存储，因为mysql单表推荐控制在500w以内的数据量，数据量过大容易把mysql搞崩。