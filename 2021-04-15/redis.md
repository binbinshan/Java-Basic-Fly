实际生产使用当中，我们不可能采取单机的形式使用Redis，肯定是多台机器来保证Redis的高并发；并且你不能说一台Redis机器挂了整个Redis集群就不能工作了，这就需要哨兵来实现高可用。

* 高并发
    1. Redis主从架构
    单机的Redis能承载的QPS大概在上万到几万，对于缓存来说一般都是需要支持高可用，所以Redis支持主从架构，一主多从，主负责写，并将数据复制到其他从节点。从节点负责读，所有的读请求都走从节点。
    
    1. Redsi主从复制的核心机制
        1. Redis采用异步方式，主节点每次接收到写命令之后，先在内部写入数据，然后异步发送给 slave node
        2. 一个主节点可以配置多个从节点
        3. 一个从节点可以复制另一个从节点
        4. 从节点在做复制的时候，不会阻塞主节点，也不会阻塞自己的查询操作，会使用旧版的数据提供服务，直到复制完成后，删除旧数据，加载新数据，这个时候会阻塞自己的查询操作
        5. 从节点主要是支持横向扩展，提高吞吐量

    2. Redis主从复制的核心原理
        当启动一个slave node的时候, 会发送一个PSYNC命令给master node, 如果这时重新连接master node, 那么master node仅会复制给slave部分缺少的数据; 否则如果是salve 第一次时, 那么会触发一次full resynchronization开始进行全量的复制, 此时的master会在后台启动一个进程, 生成一份rdb文件, 同时还会将从客户端收到的命令缓存在内存中, rdb文件生成完毕之后, master会将这个rdb发送给slave, slave会先写入本地磁盘, 然后再从本地磁盘加载到内存, 然后master会将内存中缓存的写命令发送给slave, slave也会同步这些数据.

    1. 主从复制的断点续传
        如果在主从复制的时候网络连接断掉了, 那么可以接着场次复制的地方, 继续复制下去, 而不是从头开始复制一份。这里实现原理是依靠主节点上的backlog，主节点会在内存中维护一份backlog，里面保存了replica offset，如果 master 和 slave 网络连接断掉了，slave 会让 master 从上次 replica offset 开始继续复制，如果没有找到对应的 offset，那么就会执行一次 全量复制。
        
        
    1. 过期 key 处理
        slave不会过期key, 只会等待master过期key, 如果master过期了一个key, 或者通过LRU淘汰了一个key, 那么会模拟一条del命令发送给slave
        
    1. Redis复制全流程
        ![](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-04-15/16180431742965.jpg)




* 高可用
    Redis 的高可用架构，叫做 failover 故障转移，也可以叫做主备切换。master node 在故障时，自动检测，并且将某个 slave node 自动切换为 master node 的过程，叫做主备切换。这个过程，实现了 redis 的主从架构下的高可用。
    
    * ① 哨兵介绍
        sentinel，中文名是哨兵。哨兵用于实现 redis 集群的高可用，本身也是分布式的，作为一个哨兵集群去运行，互相协同工作。

        * 集群监控：负责监控 redis master 和 slave 进程是否正常工作。
        * 消息通知：如果某个 redis 实例有故障，那么哨兵负责发送消息作为报警通知给管理员。
        * 故障转移：如果 master node 挂掉了，会自动转移到 slave node 上
        * 配置中心：如果故障转移发生了，通知 client 客户端新的 master 地址

    * ② 哨兵核心知识
        1. 哨兵至少需要 3 个实例，来保证自己的健壮性。

            * 哨兵集群中有两个参数，quorum表示认为主机宕机的哨兵数量，majority表示授权进行主从切换的最少的哨兵数量(至少一半)。只有quorum数量的节点认为主机宕机，且majority个数量的节点同意授权进行切换才可以进行故障转移。
            * 如过只有两个实例，一个哨兵实例挂掉后，另一个实例发现了主机故障，但是许执行故障转移时无法达到majority数量，就无法进行转移。
        2. 哨兵 + redis 主从的部署架构，是不保证数据零丢失的，只能保证 redis 集群的高可用性。
            
    * ③ 哨兵主备切换导致数据丢失的问题
        3. 异步复制导致的数据丢失
            因为 master->slave 的复制是异步的，所以可能有部分数据还没复制到 slave，master 就宕机了，此时这部分数据就丢失了。

        1. 脑裂导致的数据丢失
            脑裂是指某个master主机脱离了网络，哨兵进行了选举故障转移，这个时候集群就会出现两个master。
            因为脑裂的问题，客户端还在向旧的master写入数据，当旧master再次恢复的时候，就会作为从节点，重新从新主节点复制数据，就会导致脑裂之后写入旧master的数据丢失。

    * ④ 哨兵主备切换导致数据丢失的问题的解决方案
        进行如下配置：
    ```
    //至少有 1 个 slave，数据复制和同步的延迟不能超过 10 秒。
    min-slaves-to-write 1
    min-slaves-max-lag 10
    ```
    一旦所有的 slave，数据复制和同步的延迟都超过了 10 秒钟，那么这个时候，master 就不会再接收任何请求了。
        1. 减少异步复制数据的丢失
        有了 min-slaves-max-lag 这个配置，就可以确保说，一旦 slave 复制数据和 ack 延时太长，就认为可能 master 宕机后损失的数据太多了，那么就拒绝写请求，这样可以把 master 宕机时由于部分数据未同步到 slave 导致的数据丢失降低的可控范围内。

        2. 减少脑裂的数据丢失
        如果一个 master 出现了脑裂，跟其他 slave 丢了连接，那么上面两个配置可以确保说，如果不能继续给指定数量的 slave 发送数据，而且 slave 超过 10 秒没有给自己 ack 消息，那么就直接拒绝客户端的写请求。因此在脑裂场景下，最多就丢失 10 秒的数据。
        
        
    * ⑤ sdown(主观宕机)和odown(客观宕机)
        1. sdown 是主观宕机，如果一个哨兵 ping 一个 master，超过了 is-master-down-after-milliseconds 指定的毫秒数之后，那么就是主观宕机
        2. odown 是客观宕机，如果一个哨兵在指定时间内，收到了 quorum 数量的其它哨兵也认为那个 master 是 sdown 的，那么就是客观宕机
    
    * ⑥ 哨兵集群的自动发现机制
        * 哨兵互相之间的发现，是通过 redis 的 pub/sub 系统实现的，每个哨兵都会往 __sentinel__:hello 这个 channel 里发送一个消息，这时候所有其他哨兵都可以消费到这个消息，并感知到其他的哨兵的存在。

        * 哨兵完成切换之后，会在自己本地更新生成最新的 master 配置，然后同步给其他的哨兵，就是通过之前说的 pub/sub 消息机制。新的 master 配置是跟着新的 version 号的。其他的哨兵都是根据版本号的大小来更新自己的 master 配置的。

    * ⑦ slave->master 选举算法
         * 如果一个 master 被认为 odown 了，而且 majority 数量的哨兵都允许主备切换，那么某个哨兵就会执行主备切换操作，此时首先要选举一个 slave 来，会考虑 slave 的一些信息
             * 跟 master 断开连接的时长：如果一个 slave 跟 master 断开连接的时间已经超过了 down-after-milliseconds 的 10 倍
             * slave 优先级：按照 slave 优先级进行排序，slave priority 越低，优先级就越高。
             * 复制 offset：如果 slave priority 相同，那么看 replica offset，哪个 slave 复制了越多的数据，offset 越靠后，优先级就越高。
             * run id：如果上面两个条件都相同，那么选择一个 run id 比较小的那个 slave。

    * ⑧ configuration epoch
        * 执行切换的那个哨兵，会从要切换到的新 master（salve->master）那里得到一个 configuration epoch，这就是一个 version 号，每次切换的 version 号都必须是唯一的。
        * 如果第一个选举出的哨兵切换失败了，那么其他哨兵，会等待 failover-timeout 时间，然后接替继续执行切换，此时会重新获取一个新的 configuration epoch，作为新的 version 号。
