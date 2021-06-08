# 深入Buffer Poll(二)

### Buffer Poll简介
InnoDB中数据访问是以数据页(Page)为单位的，默认数据页的大小是16KB，而Buffer Poll则是用来管理和缓存这些数据页的。

Buffer Poll 是一片内存数据，可以划分成多个实例，每个实例大小相等，并且可以保证page只会在一个实例中。每个实例中都有自己的free、flush、lru等链表。

实际上在操作数据库的时候，不可能直接操作磁盘上数据，因为对磁盘进行随机读写操作非常耗时，而是针对内存里的Buffer Pool中的缓存数据进行(增删改查)的。

### 数据页
在mysql中的磁盘文件抽象出来了一个数据页的概念，把很多行数据放在了一个数据页里，默认数据页的大小是16KB，也就是说磁盘文件中就是会有很多的数据页，每一页数据里放了很多行数据。
当操作一行数据时，此时数据库会找到这行数据所在的数据页，然后从磁盘文件里把这行数据所在的数据页直接给加载到Buffer Pool里去。所以Buffer Pool中存放的是一个一个的数据页。
![-w447](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-06-06/16224534980676.jpg)


### 缓存页
数据页加载到Buffer Poll中叫做缓存页，每个缓存页都有一个对应的描述数据。这个描述数据包括：个数据页所属的表空间、数据页的编号、这个缓存页在Buffer Pool中的地址等一些数据。
Buffer Pool中的描述数据大概相当于缓存页大小的5%左右。也就是16k * 5% = 800字节。
![-w646](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-06-06/16224540750833.jpg)

### Free链表(空闲链表)

Free 链表中存放的都是Buffer Poll中未曾使用的空闲缓存页，InnoDB 需要数据页的时候从Free链表中获取，如果Free链表为空，即没有任何空闲缓存页，则会从LRU链表和Flush链表中通过 淘汰旧缓存页 和 Flush脏数据页 来回收。

在InnoDB初始化时，按照默认缓存页16k和800字节的描述数据去把内存划分出来。只不过这个时候，Buffer Pool中的缓存页都是空的，所以会将Buffer Poll中的所有缓存页加入到Free链表中。

![-w506](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-06-06/16224547083384.jpg)


###  磁盘上的页读取到Buffer Pool的缓存页中去？
从Free链表里获取一个描述数据块，然后就可以对应的获取到这个描述数据块对应的空闲缓存页。
然后把磁盘上的数据页读取到对应的缓存页里去，同时把相关的一些描述数据写入缓存页的描述数据块里去，最后把描述数据块从Free链表里去除。

### 如何找被缓存的数据页？
数据库有一个哈希表数据结构，会用表空间号+数据页号，作为一个key，然后缓存页的地址作为value。当要使用一个数据页的时候，通过“表空间号+数据页号”作为key去这个哈希表里查一下，如果没有就读取数据页，如果 已经有了，就说明数据页已经被缓存了。
![-w329](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-06-06/16224555361929.jpg)

### Flush链表
当缓冲池中的缓存页进行了修改，最后肯定是要刷回到磁盘中去的，那哪些缓存页是要被刷回磁盘的呢？
这里引入了另外一个Flush链表，所有被修改过且还没来得及被flush到磁盘上的Page（脏页），都会被保存在这个链表中。
所有保存在Flush List上的数据都会在LRU List中，但在LRU List中的数据不一定都在Flush List中。

和存储空闲缓存页的逻辑一样，Flush链表结构如下：
![-w545](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-06-06/16224562984064.jpg)

### LRU链表
如果Buffer Pool中的缓存页不够了怎么办？那么只能淘汰一部分缓存页数据，通过使用LRU链表来解决。

LRU链表，会被拆分为两个部分，一部分是热数据，一部分是冷数据，这个冷热数据的比例是由 innodb_old_blocks_pct参数控制的，他默认是37，也就是说冷数据占比37%。
。这样可以有效解决因为预读机制和全表扫描带来的大量不常用数据页加载到LRU链表头部问题。

![-w407](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-06-06/16227068931385.jpg)

LRU链表的三种情况：
1. 数据页第一次被加载到缓存：如果未能找到缓存页，则需要去数据文件中读取Page，并将其添加到冷数据的头部。
    ![-w646](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-06-06/16227071134306.jpg)
2. 如果在Buff Poll中找到了缓存页，并且Page在冷数据区域中。在读取完Page后会把它添加到热数据区的链表头部。

3. 如果在Buff Poll中找到了缓存页，并且Page在热数据区域中。并且只有Page处于热数据区域总长度大约1/4的位置之后，才会将其添加到热数据区域的链表头部。避免了频繁对 LRU 链表的调整


### 刷入磁盘

缓存页数量是有限的，那么肯定会在LRU中淘汰缓存页 或者 将脏页(flush链表)的数据刷入磁盘。

1. 所以有一个后台线程，会运行一个定时任务，这个定时任务每隔一段时间就会把LRU链表的冷数据区域的尾部的一些缓存页，刷入磁盘里去，清空这几个缓存页，把他们加入回free链表。

2. 另外这个后台线程同时也会找个时间把flush链表中的缓存页都刷入磁盘中，这样被修改过的脏页就会刷入磁盘。只要flush链表中的一波缓存页被刷入了磁盘，那么这些缓存页也会从flush链表和lru链表中移除，然后加入到free链表中去。

如果实在没有空闲缓存页了，此时就会从LRU链表的冷数据区域的尾部找到一个缓存页。然后把他刷入磁盘和清空，然后把数据页加载到这个腾出来的空闲缓存页里去。




