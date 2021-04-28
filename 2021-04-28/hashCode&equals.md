# hashcode 和 equals 方法的联系

当在HashSet, Hashtable, HashMap等等这些本质是散列表的数据结构中，用到了对应的类，那么这个类的“hashCode() 和 equals() ”是有关系的：

1. 如果两个对象相等，那么它们的hashCode()值一定相同。
    * 这里的相等是指，通过equals()比较两个对象时返回true。
2. 如果两个对象hashCode()相等，它们并不一定相等。
    * 因为在散列表中，hashCode()相等，即两个键值对的哈希值相等。然而哈希值相等，并不一定能得出键值对相等。“两个不同的键值对，哈希值相等”，这就是哈希冲突。此外，在这种情况下。若要判断两个对象是否相等，除了要覆盖equals()之外，也要覆盖hashCode()函数。否则，equals()无效。

当在HashSet, Hashtable, HashMap等等这些本质是散列表的数据结构中，没用到对应的类，那么这个类的“hashCode() 和 equals() ”是没有关系的。