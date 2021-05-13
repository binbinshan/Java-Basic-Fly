# 每日上亿请求量的电商系统，如何优化新生代和老年代

昨天的 [每日百万交易的支付系统，如何设置堆内存？](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-05-12/heap.md) 讨论了关于正常百万交易支付系统的堆内存设置大小，提到了像双11这种瞬时压力情况，今天就来分析下这种情况下如何进行设置。

仍然以核心的订单系统为案例，还是 3台 4核8G 的服务器，每笔订单500字节，大促时每秒300个支付订单，处理1秒，每秒占用的内存空间大概为150kb。
加上其他业务对象再扩大20倍 ，还有订单系统关联的一些操作在扩大10倍，也就是每秒30mb的内存占用。但是一秒之后，这30mb就使用完毕了，成为垃圾了。

8G的服务器我们一般会给JVM分4G，剩余的留给操作系统使用，然后我们给堆内存分3G，新生代1.5G，老年代1.5G。每个虚拟机栈给1M，大概会用到几百M，再给永久代256M。

### 新生代被占满
我们现在知道了内存占用大小，继续分析，每秒30MB的对象会占据新生代，但是1秒之后机会变成垃圾，所以新生代1.5G的内存大约50秒就会被占满。

那么就会触发Minor GC,根据 老年代可用空间大小 和 历次Minor GC后进入老年代对象的平均大小 比较 是否可以通过检查，前期肯定是可以的通过的，所以进行Minor GC，除了最近一秒在处理的订单，其余的回收掉99%的新生代对象，此时新生代存活对象大概也就50MB左右。

### Survivor区够不够
要点：让每次Minor GC后的对象都留在 Survivor里，不要进入老年代

假设 -XX:SurvivorRatio 参数我们使用的是默认值(8)，那么Eden 1.2G ,两个Survivor 分别占据150mb。那么只需要40秒，Eden区1.2GB满了就要进行Minor GC了，然后Minor GC下存活的对象50MB对象就要进入S1区。
然后再次运行40秒，把Eden区又占满了，再次垃圾回收Eden和S1中的对象，存活对象可能还是有50MB左右会进入S2 区。

这里要讨论的就是Survivor区不够的情况，比如下面这两种情况：
* 系统处理慢，没办法1秒处理完，那么就存在可能，出现Minor GC过 后的对象无法放入Survivor中。
* 假设有75MB以上的对象躲过了Minor GC,进入Survivor区，因为这是一批同龄对象，直接超过了Survivor区空间的50%，此时也可能会导致对象进入老年代。

所有这里可以调整新生代和老年代的大小，很明显我们处理的对象都是短期存活，所以没有必要让对象进入老年代。
调整我们刚开始的(堆最小3G，堆最大3G，新生代1.5G ) -Xms3072M -Xmx3072M -Xmn1536M 的参数，设置为 (堆最小3G，堆最大3G，新生代2G )  -Xms3072M -Xmx3072M -Xmn2048M。

此时Eden为1.6G，每个Survivor为200MB，Survivor区域变大，就降低了对象直接进入老年代的问题。


### 晋升老年代年龄
每40秒触发一次Mionr GC,那么假设使用默认的15作为晋升年龄，那么在新生代存活了10分钟的对象，也该进入老年代了。
那么有没有必要调大这个年龄呢,我认为没有必要，既然15次Mionr Gc还存活的对象，大概率是长期存活的，这种对象应该也不多，所以调整这个参数也无非是让对象在新生代多待几分钟，没有实际意义。我们需要让它快点进到老年代里，下调这个参数 -XX:MaxTenuringThreshold= 10。

### 多大的对象直接进入老年代
-XX:PretenureSizeThreshold 默认值是0，默认所有对象都在新生代分配内存，结合我们的订单系统每个订单才500字节，所以-XX:PretenureSizeThreshold设置个1MB即可。

### 指定垃圾回收器
使用命令设置-XX:+UseParNewGC XX:+UseConcMarkSweepGC。
新生代使用ParNew，老年代使用CMS。
ParNew垃圾回收器的核心参数其实我们不用特别关心，配置好新生代内存大小、Eden和Survivor的比例，动态年龄判断等参数就没问题。

### 什么时候对象会进入老年代？
1. 我们设置了 -XX:MaxTenuringThreshold= 10 ，所以大约 6、7分钟躲过Minor GC的对象就会进入老年代。这种长期被GC Roots引用的对象一般不会太多。
2. 还要就是-XX:PretenureSizeThreshold = 1MB，大于1MB的对象也会直接进入老年代。
3. Minor GC过后可能存活的对象超过200MB放不下Survivor了，或者是一下子占到超过Surviovr的50%，此时 会有一些对象进入老年代中。虽然我们优化了新生代的内存，但是无法保证不出现该类情况。

### 那老年代多久触发一次full GC呢？
每隔10分钟会在Minor GC之后有100MB左右的大小的一批对象进入老年代。
Full GC触发的情况：
1. Minor GC之前，都检查一下“老年代可用内存空间” < “历次Minor GC后升入老年代的平均对象大小”
2. 某次Minor GC后要升入老年代的对象比较多，但是老年代可用空间不足了
3. 设置了“-XX:CMSInitiatingOccupancyFaction”参数，老年代空间使用超过这个参数比例了。

情况1就目前情况看了，要很多次Minor GC之后才可能有一两次碰巧会有100MB对象升入老年代，所以触发Full GC的情况很小。
所以很可能是在系统运行1小时之后，才会有接近 1GB的对象进入老年代，满足了上述情况，才会触发了Full GC。1个小时之后，双11的热点也已经过了，此时才可能会触发一次Full GC，随着高峰期过去，大概几个小时才会触发一次。

### 总结
Full GC优化的前提是Minor GC的优化，Minor GC的优化的前提是合理分配内存空间，合理分 配内存空间的前提是对系统运行期间的内存使用模型进行预估。尽量 让Minor GC之后的存活对象留在Survivor里不要去老年代，然后其余的GC参数不做太多优化。


-Xms3072M -Xmx3072M -Xmn2048M -Xss1M -XX:PermSize=256M -XX:MaxPermSize=256M XX:SurvivorRatio=8 -XX:MaxTenuringThreshold=10 -XX:PretenureSizeThreshold=1M -XX:+UseParNewGC XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFaction=95 -XX:+UseCMSCompactAtFullCollection -XX:CMSFullGCsBeforeCompaction=0





