# 如何确定eden区的对象何时进入老年代？

在 JVM 内存模型的堆中，堆被划分为新生代和老年代，新生代又被进一步划分为 Eden区 和 Survivor区，Survivor 区由 From Survivor 和 To Survivor 组成

1. 当创建一个对象时，对象会被优先分配到新生代的 Eden 区，此时 JVM 会给对象定义一个对象年轻计数器（-XX:MaxTenuringThreshold）,默认值是15

2. 当 Eden 空间不足时，JVM 将执行新生代的垃圾回收（Minor GC）
JVM 会把存活的对象转移到 Survivor 中，并且对象年龄 +1 ，对象在 Survivor 中同样也会经历 Minor GC，每经历一次 Minor GC，对象年龄都会+1

1. 进入老年代的两种情况：
    * 对象年轻计数器达到参数 -XX:MaxTenuringThreshold 设置的值，默认是15，就会进入老年代
    
    * 当一个S区中，达到某个年龄的对象的比例 大于等于Desired survivor size(默认是50%)，则重新计算threshold，以该年龄和 MaxTenuringThreshold 两者的最小值为晋升老年代的阈值


