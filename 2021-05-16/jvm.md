# 深入JVM堆内存

### 1、年轻代、老年代、永久代

永久代：JVM里的永久代其实就是方法区，存放一些类信息的。

#### 基础概念
要来分析为什么有年轻代和老年代的，就必须要先明白短期和长期存活对象都是什么。
* 短暂存活的对象：生命周期很短，被使用完之后就成为垃圾的对象。
* 长期存活的对象：需要长期被使用，生命周期很长，需要长期存在，轻易不会被垃圾回收的。

好了我们进入讨论的正题 ，JVM堆的分代模型 新生代和老年代。

因为我们对象的生命周期不同，从垃圾回收的角度，所以JVM的堆实现了分代模型，将JVM的堆划分为 新生代和老年代。新生代存放短暂存活的对象，老年代存放长期存活的对象。

#### 对象怎么进入老年代？
上面叙述了 老年代存放长期存活的对象，那么这些对象难道是直接进入老年代的吗？显然这里是要分情况的：
* 第一种：正常对象是在新生代分配内存的，只有躲过多次新生代垃圾回收的对象，才会进入老年代。
* 第二种：大对象，是直接进入到老年代的。

##### 第一种情况：
正常对象(大对象除外)都是会在新生代进行内存分配的，躲过了多次垃圾回收，才会进入老年代。

这里需要注意几个点了：

1. 怎么判断哪些是垃圾？
    你说你是垃圾，怎么证明，所以JVM中使用了一种可达性分析算法来判定的，没有被GC Root引用着的对象就是垃圾。而这个GC Root就可以理解为 局部变量、静态变量。简单点就是只要对象被方法的局部变量、类的静态变量给引用了，就不会回收它。

1. 垃圾回收什么时候触发？
    新生代的垃圾回收 Minor GC , 只有在新生代去分配一个对象内存时，发现内存不够了，才会进行 Minor GC。把那些短期存活的已经变成垃圾的对象回收。
    
1. 躲过了多次到底是几次？
    这里又分为两种情况：
    * 如果一个实例对象在新生代中，成功的在15次垃圾回收之后，还是没被回收掉，就说明他已经15岁了，然后他会被转移到Java堆内存的老年代中去。当然了，这个15是通过JVM参数“-XX:MaxTenuringThreshold”来设置的，默认是15岁。
    
    * 动态年龄判断的规则：
      当前放对象的Survivor区域里，一批对象的总大小大于了这块Survivor区域的内存大小的50%，那么此时大于等于这批对象年龄的对象，就可以直接进入老年代了。
      ![-w528](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-05-16/16209906509206.jpg)    
      假设 存活对象1+存活对象2 >（100MB * 50%） ，那么Survivor1区中所有大于2岁的对象都要进入老年代。
      逻辑：年龄1+年龄2+年龄n的多个年龄对象总和超过了Survivor区 域的50%，此时就会把年龄n以上的对象都放入老年代。

##### 第二种情况：
大对象直接进入老年代，JVM中有一个参数，就是“-XX:PretenureSizeThreshold”，可以把他的值设置为字节数，比如“1048576”字节，就是1MB。
那么一个大于这个大小的对象就会直接进入老年代，不会在年轻代出现。这样做是避免新生代里出现那种大对象，然后屡次躲过GC，还得把他在两个Survivor区域里来回复制多次之后才能进入 老年代。

### 2、新生代的垃圾回收
针对新生代的垃圾回收算法，叫做复制算法

##### 新生代为什么使用复制算法进行回收？
如果采用 整理清除 的算法，就会产生大量的内存碎片，就会浪费大量的内存空间，虽然所有的内存碎片加起来其实有很大的一块内存，但是因为这些内存都是碎片式分散的，所以导致没有一块完整的足够的内存空间来分配新的对象。

##### 复制算法的缺点
所谓的“复制算法“，把新生代内存划分为两块相同大小的内存区域，然后只使用其中一块内存待那块内存快满的时候，就把里面的存活对象一次性转移到另外一块内存区域，保证没有内存碎片 接着一次性回收原来那块内存区域的垃圾对象，再次空出来一块内存区域。两块内存区域就这么重复着循环使用。

但是复制算法的缺点其实非常的明显，如果按照上述思路，假设我们给新生代1G的内存空间，那么只有 512MB的内存空间是可以用的，从始至终，就只有一半的内存可以用，这样的算法显然对内存的使用效率太低了。

##### 复制算法的优化
之前分析过，其实绝大多数的对象都是存活周期非常短的对象，可能被创建出来1毫秒之后就没人引用了，就是垃圾对象了。所以有可能一次垃圾回收后，99%的对象都被回收了，就1%的对象存活了下来。
所以实际上真正的复制算法会做出如下优化，把新生代内存区域划分为三块：
1个Eden区，2个Survivor区，其中Eden区占80%内存空间，每一块Survivor区各占10%内存空间，比如说Eden区有800MB内存，每 一块Survivor区就100MB内存。这么做最大的好处，就是只有10%的内存空间是被闲置的，90%的内存都被使用上了。

具体流程：
刚开始对象都是分配在Eden区内的，如果Eden区快满了，此时就会触发Minor GC，然后把Eden区存活的对象移动到其中一个空着的S区。
如果下次再次Eden区满，那么再次触发Minor GC，就会把Eden区和放着上一次Minor GC后存活对象的Survivor区内的存活对象，转移到另外一块Survivor区去。始终保持一块Survivor区是空着的。


##### 空间分配担保规则
如果新生代里有大量对象存活下来，确实是自己的Survivor区放不下了，必须转移到老年代去那么如果老年代里空间也不够放这些对象呢？

首先，在执行任何一次Minor GC之前，JVM会先检查一下老年代可用的可用内存空间，是否大于新生代所有对象的总大小。

为什么要判断检查这个呢？
假设 进入老年代的对象 > 老年代可用内存空间，那么你不能直接OOM额,你最起码做点什么，那就来一次Full GC,看看之后可不可以放到老年代去。

具体流程：
1. 执行 Minor GC **之前** ，JVM会先检查一下老年代的**可用内存空间**，是否大于新生代**所有对象的总大小**。

2. 如果老年代的可用内存大小 大于 新生代所有对象的，就会进行Minor GC。即使Minor GC之后所有对象都存活，Survivor区放不下了，也可以转移到老年代去。
3. 老年代的可用内存 小于 新生代的全部对象大小，就要看是否允许担保失败，需要通过 “XX:HandlePromotionFailure” 参数设置
    1. 允许担保失败：
        判断老年代的可用内存大小，是否大于之前每一次Minor GC后进入老年代的对象的平均大小。大于的话就会冒险 尝试一下Minor GC，失败的话 就会触发一次“Full GC”。
    2. 不允许担保失败：
       触发一次“Full GC”

![-w880](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-05-16/16209928074282.jpg)



### 3、老年代的垃圾回收
对老年代触发垃圾回收的时机，一般就是两个：
1. 在Minor GC之前，发现很可能Minor GC之后要进入老年代的对象太多了，老年代放不下，此时需 要提前触发Full GC然后再带着进行Minor GC；
2. 在Minor GC之后，发现剩余对象太多放入老年代都放不下了。

老年代的垃圾回收算法采用的 标记整理 ，会让这些存活对象在内存里进行移动，把存活对象尽量都挪动到一边去，让存活对象紧凑的靠在一起，避免垃圾 回收过后出现过多的内存碎片



### 4、案例
我们以一个案例，再来分析一个频繁Full GC的系统：

* 背景
   一台机器是4核8G的配置，JVM内存给了4G，其中新生代和老年代分别是1.5G的内存空间：
    ![-w436](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-05-16/16210678041864.jpg)
    
    每台机器大概每分钟负责执行100次任务，每次有1W条数据需要处理，每条数据大概20个字段，合计需要占用空间1KB，每次计算大概需要10秒。

* 多久新生代会触发Minor GC ?

  根据上述描述，每分钟大概会有 10000 * 1 * 100 = 1000MB 的数据进入新生代，基本上一分钟之后就会第一次Minor GC。
  
* 触发Minor GC的时候会有多少对象进入老年代？

  此时进行Minor GC前，先进行分配担保判断，第一次GC时，老年代可用空间 1.5G 肯定是大于新生代对象的，进行Minor GC , 因为每次计算需要10秒，此时大概还会有 200MB 的对象存活在着。
  每个S区的大小为150MB，那么Minor GC后存活了 200MB对象，肯定是放不到S区中，那么就会直接进入老年代。此时新生代空了，老年代中有200MB 的对象。
  
  
* 多久老年代会触发Full GC ？

   每分钟都要进行Minor GC,每次Minor GC都会有200MB的对象进入老年代。
   那么 2 分钟过去后，老年代中有 400MB 的对象，剩余可用空间为1.1G ，此时再次进行Minor GC时，老年代的可用内存空间 < 新生代对象大小。
   此时就要看是否允许担保失败了，如果不允许 就会触发Full GC；当然了一般都是允许担保失败的，此时就会检查老年代的可以空间 是否 大于历次Minor GC过后进入老年代的对象的平均大小，我们之前每次Minor GC后进入到老年代的对象大概为 200MB，所以可以推测，本次Minor GC后大概率还是有200MB对象进入老年代，1.1G可用空间是足够的。
   
  但是 1.5G /200MB ≈ 7 分钟后，7次Minor GC执行过后，大概1.4G对象进入老年代，老年代剩余空间就不到100MB，在第8分钟运行结束的时候，新生代又满了，执行Minor GC之前进行检查，此时发现老年代只有100MB内存空间了，比之前每次Minor GC后进入老年代的200MB对象要小，此时就会直接触发一次Full GC，接着就会执行Minor GC。
  
  所以我们可以推测每过 7、8分钟 就要进行一次Full GC,这个是不可接受的。
  
* 优化
   最大的问题，其实就是每次Survivor区域放不下存活对象。
   可以增加新生代的内存比例，3GB左右的堆内存，其中2GB分配给新生代，1GB留给老年代，这样Survivor区大概就是200MB，每次刚好能放得下Minor GC过后存活的对象了。
   
     只要每次Minor GC过后200MB存活对象可以放Survivor区域，那么等下一次Minor GC的时候，这个Survivor区的对象对应的任务早就结束了，都是可以回收的垃圾了。
    以此类推，基本上就很少对象会进入老年代中，老年代里的对象也不会太多的。把生产系统的老年代Full GC的频率从几分钟一次降低到了几个小时一次，大幅 度提升了系统的性能，避免了频繁Full GC对系统运行的影响。
  
  
  上述有个问题就是没有考虑动态年龄判定升入老年代的规则，就是如果Survivor区中的同龄对象大小超过 Survivor区内存的一半，就要直接升入老年代。这个其实可以调整比例来解决，问题根本就是 让每次Minor GC后的对象进入Survivor区中。