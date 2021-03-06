# 深入垃圾回收器

首先进行垃圾回收是有专门的垃圾回收线程的，而且对不同的内存区域会有不同的垃圾回收器。
类似于垃圾回收线程和垃圾回收器配合起来，使用自己的垃圾回收算法，对指定的内存区域进行垃圾回收。

### 1、JVM的痛点：Stop the World
在正式讨论垃圾回收器之前，需要先知道Stop the World是什么？
* 问题1：GC的时候还能继续创建新的对象吗？

  这个问题主要是想问，在进行GC的时候，还能不能继续创建新的对象进行内存分配呢？
  我们现在假设还能继续创建对象，垃圾回收器在想办法把Eden和Survivor2里的存活对象标记出来转移到Survivor1去，然后还在想办法把Eden 和Survivor2里的垃圾对象都清理掉，结果这个时候系统程序还在不停的在Eden里创建新的对象，新创建的这些对象状态垃圾回收器是无法追踪的，就没办法处理了。
  所以在垃圾回收的过程中，同时还允许我们写的Java系统继续不停的运行在Eden里持续创建新的对象，目前来看是非常不合适的一个事情。
   
* 问题2：为什么说STW是痛点呢？
  在垃圾回收的时候，不能让Java系统继续运行，此时JVM会在后台直接进入“Stop the World”状态。此时所有的Java工作线程都停止工作了，直到垃圾回收结束了，才会恢复。
  假设Minor GC要运行100ms，那么可能就会导致系统直接停顿100ms不能处理任何请求，而Full GC是最慢的，有的时候甚至几十秒，就会导致系统停止几十秒中无法处理。
  
  
### 2、新生代垃圾回收器：ParNew
如果没有G1垃圾回收器的话，通常大家线上系统都是ParNew垃圾回收器作为新生代的垃圾回收器。即使有了G1，其实很多线上系统还是用的ParNew。

只有一个垃圾回收线程在工作确实太慢了，为了充分利用服务器的多核CPU的优势，例如4核CPU就可以支持4个垃圾回收线程并行执行，可以提升4倍的性能。ParNew默认情况下的垃圾回收线程数量是和CPU的核数是一样的。
所以新生代的ParNew垃圾回收器就是多线程垃圾回收机制，另外一种Serial垃圾回收器是单线程垃圾回收，他们俩都是回收新生代的，唯一的区别就是单线程和多线程的区别，但是垃圾回收算法是完全一样的。
![-w545](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-05-17/16210792272978.jpg)

使用“-XX:+UseParNewGC”选项，JVM启动之后对新生代进行垃圾回收的，就是 ParNew垃圾回收器了。


### 3、老年代垃圾回收器：CMS
一般老年代我们选择的垃圾回收器是CMS，CMS采用了4个阶段来垃圾回收，其中初始标记和重新标记，耗时很短，虽然会导致 “Stop the World”，但是影响不大。并发标记和并发清理，两个阶段耗时最长，但是是可以跟系统的工作线程并发运行的，所以对系统没太大影响。

CMS采用的是标记清理算法，把垃圾对象都标记出来，然后一次性把垃圾对象都回收掉，最大的问题，就是会造成很多内存碎片。

* CMS如何实现系统一边工作的同时进行垃圾回收？
  CMS在执行一次垃圾回收的过程一共分为4个阶段：初始标记、并发标记、重新标记、并发清理
  * 初始标记 ：进入“Stop the World”状态，标记出来所有GC Roots直接引用的对象，速度很快，影响不大。

  * 并发标记 ：对老年代所有对象进行GC Roots追踪，判断是否需要回收，其实是最耗时的，但是系统程序可以继续工作，不会对系统运行造成影响的。
  
  * 重新标记 ：由于并发标记时系统程序继续运行，导致系统可能会有很多存活对象和垃圾对象没标记出来的，再次进入“Stop the World”阶段，把这些对象标记出来。只有少量标记，运行速度很快。

  * 并发清理 : 清理掉之前标记为垃圾的对象，虽然耗时很长，但是也是跟系统程序并发运行的。

* CMS有什么问题呢？
    * 并发回收垃圾导致CPU资源紧张
        * 在并发标记和并发清理两个最耗时的阶段，垃圾回收线程和系统工作线程同时工作，会导致有限的CPU资源被垃圾回收线程占用了一部分。CMS默认启动的垃圾回收线程的数量是（CPU核数 + 3）/ 4。 2核的时候，CMS会有1个垃圾回收线程。

        
    * Concurrent Mode Failure问题（并发垃圾回收失败）
        * 在并发清理阶段，CMS只是回收之前标记好的垃圾对象，但是这个阶段系统一直在运行，就会让一些对象进入老年代，同时有的对象还变成垃圾对象，这种垃圾对象是“浮动垃圾”。需要等到下一次GC的时候才会回收。

        * 所以为了保证在CMS垃圾回收期间，还有一定的内存空间让一些对象可以进入老年代，一般会预留一些空间。CMS垃圾回收的触发时机，其中有一个就是当老年代内存占用达到一定比例了，就自动执行GC。使用XX:CMSInitiatingOccupancyFaction 设置，默认值是 92%。

        * 在CMS垃圾回收期间，如果此时放入老年代的对象大于老年代可用内存空间，这个时候会发生Concurrent Mode Failure，此时就会自动用“Serial Old”垃圾回收器替代CMS，就是直接强行把系统程序“Stop the World”，重新进行长时间的GC Roots追踪，标记出来全部垃圾对象，不允许新的对象产生，然后一次性把垃圾对象都回收掉，完事儿了再恢复系统线程。
        * 所以要尽量避免“Concurrent Mode Failure”问题。
    
    * 内存碎片问题
        * 老年代的CMS采用“标记-清理”算法，每次都是标记出来垃圾对象，然后一次性回收掉，这样会导致大量的内存碎片产生。太多的内存碎片实际上会导致更加频繁的Full GC。

        * CMS有一个参数是“-XX:+UseCMSCompactAtFullCollection”，默认打开。意思是在Full GC之后要再次进行“Stop the World”，停止工作线程，然后进行碎片整理，就是把存活对象挪到一起，空出来大片连续内存空间，避免内存碎片。

        * 还有一个参数是“-XX:CMSFullGCsBeforeCompaction”，这个意思是执行多少次Full GC之后再执行一次内存碎片整理的工作，默认 是0，意思就是每次Full GC之后都会进行一次内存整理。

    

