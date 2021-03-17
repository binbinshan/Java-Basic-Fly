# mysql索引实现原理？如何使用索引？

1. 使用索引的缺点和优点是什么？
2. MySQL索引的原理和数据结构能介绍一下吗？
3. b+树和b-树有什么区别？
4. myIsam和InnoDb实现索引的区别？
5. MySQL聚簇索引和非聚簇索引的区别是什么？他们分别是如何存储的？
6. 什么是回表查询？非聚簇索引一定会回表查询吗？
7. 使用MySQL索引都有哪些原则？
8. MySQL复合索引如何使用？


### 1、使用索引缺点和优点是什么？
优点：通过使用索引，可以大大提高数据的检索速度。
缺点：索引需要物理空间存储，并且需要动态的去维护。


### 2、MySQL索引的原理和数据结构能介绍一下吗？
Mysql索引原理其实就是把一个表中的某一列放到一个数据结构中，当对该列进行查询的时候，就可以不用全表扫描，只需要查询特定的数据结构找到那一列中对应的值，然后在找到其所在行的地址，或者所在行的数据。

Mysql中的索引实现为b+树。


### 3、b+树和b-树的区别？
b-(b树，不要读b减树)
1. B树的所有节点既存放 键(key) 也存放 数据(data);而 B+树只有叶子节点存放 key 和 data，其他内节点只存放 key。
2. B 树的叶子节点都是独立的;B+树的叶子节点有一条引用链指向与它相邻的叶子节点。
3. B+树的检索效率稳定，任何查找都是从根节点到叶子节点的过程。


### 4、myIsam和InnoDb实现索引的区别？
myisam存储引擎的索引中，每个叶子节点的data存放的是数据行的物理地址，比如0x07之类的东西，然后根据这个物理地址去获取数据，myIsam最大的特点就是索引和数据是分开存储的。

innodb的数据文件本身就是个索引文件，叶子节点的key就是主键，然后叶子节点的data就是数据。


### 5、MySQL聚簇索引和非聚簇索引的区别是什么？他们分别是如何存储的？
聚簇索引：innndb存储引擎是要求表必须有主键的，然后会根据这个主键创建一个默认索引，这个索引中叶子节点的值就是主键key所在行的数据。
非聚簇索引：是指对某个非主键的字段创建索引，该索引中叶子结点存的值就是主键的值，需要根据主键再去聚簇索引中根据key查询到数据。

这里就说明了，为啥innodb下不要用UUID生成的超长字符串作为主键？
因为所有的非聚簇索引的data都是那个主键值，进行回表查询，最终导致索引会变得过大，浪费很多磁盘空间。建议使用自增主键，这样主键是单调自增，不会导致索引B+树分裂，提高效率。


### 6、什么是回表查询？非聚簇索引一定会回表查询吗？
当使用非聚簇索引的时候，这时查询得到的值是主键，然后再根据主键去查询聚簇索引，称为回表查询。

非聚簇索引一定会回表查询吗？ 
答案是否定的，有一种情况覆盖索引(索引覆盖)，当一个索引包含了所有需要查询的字段的值，就称之为“覆盖索引”。这时就不需要回表查询了。
例如：创建了索引(name,age)，查询语句为：
```
//只查询了name.age  此时数据都在就不要回表查询了。
select name,age from table where name = 1 and age=10;
```


### 7、使用MySQL索引都有哪些原则？
最左前缀原则:
MySQL中的索引可以以一定顺序引用多列，这种索引叫作联合索引。如User表的name和city加联合索引就是(name,city)，而最左前缀原则指的是，如果查询的时候查询条件精确匹配索引的左边连续一列或几列，则此列就可以被用到。如下：
```
    select * from user where name=xx and city=xx ; //可以命中索引
    select * from user where name=xx ; // 可以命中索引
    select * from user where city=xx ; // 无法命中索引    
```        
这里需要注意的是，查询的时候如果联合索引中所有条件都用上了，但是顺序不同，如 city= xx and name ＝xx，那么现在的查询引擎会自动优化为匹配联合索引的顺序，这样是能够命中索引的。



### 8、MySQL复合索引如何使用？
1. 全列匹配
这个就是说，你的一个sql里，正好where条件里就用了这3个字段，那么就一定可以用到这个联合索引的： select * from product where shop_id=1 and product_id=1 and gmt_create=’2018-01-01 10:00:00’

2. 最左前缀匹配
这个就是说，如果你的sql里，正好就用到了联合索引最左边的一个或者几个列表，那么也可以用上这个索引，在索引里查找的时候就用最左边的几个列就行了：select * from product where shop_id=1 and product_id=1，这个是没问题的，可以用上这个索引的

3. 最左前缀匹配了，但是中间某个值没匹配
这个是说，如果你的sql里，就用了联合索引的第一个列和第三个列，那么会按照第一个列值在索引里找，找完以后对结果集扫描一遍根据第三个列来过滤，第三个列是不走索引去搜索的，就是有一个额外的过滤的工作，但是还能用到索引，所以也还好，例如：
select * from product where shop_id=1 and gmt_create=’2018-01-01 10:00:00’
就是先根据shop_id=1在索引里找，找到比如100行记录，然后对这100行记录再次扫描一遍，过滤出来gmt_create=’2018-01-01 10:00:00’的行

4. 没有最左前缀匹配
那就不行了，那就在搞笑了，一定不会用索引，所以这个错误千万别犯
select * from product where product_id=1，这个肯定不行

5. 前缀匹配
这个就是说，如果你不是等值的，比如=，>=，<=的操作，而是like操作，那么必须要是like ‘XX%’这种才可以用上索引，比如说
select * from product where shop_id=1 and product_id=1 and gmt_create like ‘2018%’

6. 范围列匹配
如果你是范围查询，比如>=，<=，between操作，你只能是符合最左前缀的规则才可以范围，范围之后的列就不用索引了 select * from product where shop_id>=1 and product_id=1


7. 包含函数
如果你对某个列用了函数，比如substring之类的东西，那么那一列就不用索引
select * from product where shop_id=1 and 函数(product_id) = 2

8. 索引不会包含有 NULL 值的列
只要列中包含有 NULL 值都将不会被包含在索引中，复合索引中只要有一列含有 NULL 值，那么这一列对于此复合索引就是无效的。所以在数据库设计时不要让字段的默认值为 NULL

