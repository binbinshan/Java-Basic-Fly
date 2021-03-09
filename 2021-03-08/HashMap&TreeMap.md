

## 简述 HashMap 和 TreeMap 的实现原理以及常见操作的时间复杂度

HashMap：JDK1.8之前采用的是数组+链表的数据结构，JDK1.8之后改为数组+红黑树的数据结构
TreeMap：数组+红黑树的数据结构

HashMap的put、get、remove在选择合适的哈希函数的情况下，时间复杂度为O(1)
如果此时数组元素中是链表形式的，此时的时间复杂度为O(n)
如果此时数组元素中是红黑树，时间复杂度为O(logn)

TreeMap底层通过红黑树(Red-Black tree)实现，也就意味着containsKey(), get(), put(), remove()都有着log(n)的时间复杂度。
