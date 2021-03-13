

## String，StringBuffer，StringBuilder 之间有什么区别？

1. 三者都是用来处理字符串的，都是final类，不允许继承。
2. 执行速度 StringBuilder > StringBuffer > String。
3. String的长度是不可变的，StringBuilder和StringBuffer是长度可变的。
4. StringBuffer是线程安全的，StringBuilder不是线程安全的。

### String
String是不可变的对象，因此每次在对String类进行改变的时候都会生成一个新的string对象，然后将指针指向新的string对象，所以经常要改变字符串长度的话不要使用string，因为每次生成对象都会对系统性能产生影响，特别是当内存中引用的对象多了以后，JVM的GC就会开始工作，性能就会降低；

### StringBuffer
使用StringBuffer类时，每次都会对StringBuffer对象本身进行操作，而不是生成新的对象并改变对象引用，所以多数情况下推荐使用StringBuffer，特别是字符串对象经常要改变的情况；

### StringBuilder
StringBuilder是5.0新增的，此类提供一个与 StringBuffer 兼容的 API，但不保证同步。该类被设计用作 StringBuffer 的一个简易替换，用在字符串缓冲区被单个线程使用的时候。如果可能，建议优先采用该类，因为在大多数实现中，它比 StringBuffer 要快。两者的方法基本相同
