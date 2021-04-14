# redis 的过期策略都有哪些？内存淘汰机制都有哪些？手写一下 LRU 代码实现？

首先问题就是，Redis都是存储在内存上的，假设我们内存是10G，你放入了20G的数据到Redis里，那肯定会有10G的数据丢失，那么怎么判断丢弃哪10G呢？这就需要Redis的过期策略。

### Redis的过期策略
redis 过期策略是：定期删除+惰性删除。
* 定期删除：
    * 指的是 redis 默认是每隔 100ms 就随机抽取一些设置了过期时间的 key，检查其是否过期，如果过期就删除。
    * 这里Redis肯定不能扫描全部的key进行判断，否则CPU消耗很大，实际上 redis 是每隔 100ms 随机抽取一些 key 来检查和删除的。

* 惰性删除：
    * 定期删除可能会导致很多过期 key 到了时间并没有被删除掉，那咋整呢？所以就是惰性删除了。
    * 惰性删除就是获取 key 的时候，如果此时 key 已经过期，就删除，不会返回任何东西。

### 内存淘汰机制都有哪些？
我们上面说了一大堆，比如有些情况下，定期删除漏掉了很多过期 key，然后也没及时去查，也就没走惰性删除，此时会怎么样？如果大量过期 key 堆积在内存里，导致 redis 内存块耗尽了。这个时候就需要内存淘汰机制。

redis 内存淘汰机制有以下几个：
* noeviction: 当内存不足以容纳新写入数据时，新写入操作会报错，这个一般没人用吧，实在是太恶心了。
* allkeys-lru：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的 key（这个是最常用的）。
* allkeys-random：当内存不足以容纳新写入数据时，在键空间中，随机移除某个 key，这个一般没人用吧，为啥要随机，肯定是把最近最少使用的 key 给干掉啊。
* volatile-lru：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，移除最近最少使用的 key（这个一般不太合适）。
* volatile-random：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，随机移除某个 key。
* volatile-ttl：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，有更早过期时间的 key 优先移除。

上面的淘汰机制很多，常用的就是allkeys-lru，也就是内存不够写入新数据时，就把**最近最少**使用的key给干掉。


### 手写一下 LRU 代码实现？
我们就简单说下一个简单的LRU实现过程。
1. 使用linkedHashMap，定义KV结构的缓存对象
2. 按照访问顺序来进行排序，最近访问的放在头部，最老访问的放在尾部。
3. 当 map中的数据量大于指定的缓存个数的时候，就自动删除最老的数据。