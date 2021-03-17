# mysql中的事务

1. 事务的特点是哪几个？
2. mysql的事务隔离级别？
3.  mysql中可能出现的事务问题
4. mysql的默认事务隔离级别？如何实现的？

###  1、事务的特点是哪几个？
事务具有四个特性，原子性、一致性、隔离性、持久性
原子性(Atomic):一个事务中要么全部提交成功，要么全部失败回滚，不能只有一部分提交。
一致性(consistency):一个事务执行前执行后的，都必须保证正确性。
隔离性(isolation):多个事务在执行的时候，不能互相干扰。
持久性(durability)：一个事务提交之后，对数据的修改必须永远有效。

### 2、mysql的事务隔离级别？
mysql的事务隔离级别共四种：
1. 读未提交
    是指一个事务在执行过程中，查询时读到了另一个事务修改完的但是还没有提交事务的数据。
2. 读已提交
    是指一个事务在执行过程中，查询时读到了另一个事务修改完且已经提交事务的数据。
3. 可重复读
    是指一个事务在执行过程中，无论别的事务是否对查询数据修改，查询到的都是进入事务时的数据。
4. 串行化
    所有的事务都是串行执行，不允许同时执行多个事务。可以解决幻读的情况。
    
    

### 3、mysql中可能出现的事务问题
|  问题 | 释义  | 出现的隔离级别 | 解决的隔离级别 |
|---|---|---|---|
| 脏读（dirty read） | 事务在执行过程中读到了别的事务未提交的数据  | 读未提交  |  读已提交 |
| 不可重复读（non-repeatable read）  | 在一个事务A中多次操作数据，在事务操作过程中(未最终提交)，事务B也才做了处理，并且该值发生了改变，这时候就会导致A在事务操作的时候，发现数据与第一次不一样了。 就是不可重复读。  | 读未提交、读已提交  | 可重复读  |
| 幻读（phantom read）  |  第一个事务对一个表中的数据进行查询所有行。第二个事务是向表中插入“一行新数据”。那么以后第一个事务就会发现表中还有新的数据行，就好象发生了幻觉一样. | 读未提交、读已提交、可重复读  | 串行化  |

### 4、mysql中的事务隔离级别
mysql的默认事务是 可重复读，就是每个数据都会开启一个对要操作的数据的一个快照。事务期间读的都是这个快照，就保证了可重复读。

#### 4.1、MySQL的可重复读实现原理
通过MVCC机制来实现的，就是多版本并发控制。

当我们使用innodb存储引擎，会在每行数据的最后加两个隐藏列，一个保存行的创建时间，一个保存行的删除时间，但是存放的不是时间，而是事务id（事务ID在mysql内部是全局唯一递增的）。

在一个事务内查询的时候，mysql只会查询创建时间的事务id小于等于当前事务id的行，这样可以确保这个行是在当前事务中创建，或者是之前创建的；
同时一个行的删除时间的事务id要么没有定义（就是没删除），要么是必当前事务id大（在事务开启之后才被删除）；满足这两个条件的数据都会被查出来。

##### 4.1.1、删除情况下的原理
如下图：
![image](https://user-images.githubusercontent.com/23192002/111483490-c9b2f400-876f-11eb-92ce-7bfa3f8f9c3f.png)

 
select * from table where id=1；
当事务id=100的事务，查询id=1的这行数据时，一定会找到 创建事务ID <= 当前事务ID的 那行数据。
![image](https://user-images.githubusercontent.com/23192002/111483532-d1729880-876f-11eb-9bad-8da9ae9e1a1b.png)

当事务id=102的事务，将id=1的这一行给删除了，此时就会将id=1的行的删除事务id设置成102。
此时事务id=100的事务，再次查询id=1的那一行，还是可以查到的，因为 事务ID 100 符合要求 创建事务id <= 当前事务id，当前事务id < 删除事务id，也就是100 <= 100 ，100 < 102

##### 4.1.2、更新情况下的原理

如果某个事务执行期间，别的事务更新了一条数据呢？
![image](https://user-images.githubusercontent.com/23192002/111483553-d6374c80-876f-11eb-80a6-0e5b4cee0321.png)

当 创建事务=103 查询id=2这一行数据，此时把ID=2的name改成小B，那么就会变成如下这样：
![image](https://user-images.githubusercontent.com/23192002/111483563-da636a00-876f-11eb-8fc0-9b3229118195.png)

创建事务ID=104的ID，新插入了一条，将原来记录的删除事务ID设置为104。

在innodb中，是新插入了一行记录，然后将新插入的记录的创建时间设置为新的事务的id，同时将这条记录之前的那个版本的删除时间设置为新的事务的id。
这样的话，你的这个事务其实对某行记录的查询，始终都是查找的之前的那个快照，因为之前的那个快照的创建时间小于等于自己事务id，然后删除时间的事务id比自己事务id大，所以这个事务运行期间，会一直读取到这条数据的同一个版本。
 