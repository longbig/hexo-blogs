---
title: mysql索引基础
date: 2021-04-17 21:05:25
type: "_posts"
layout: "posts"
img: "https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/01.jpeg"
---

## 索引常见模型

### 三种数据结构

* 三种数据结构：哈希表，有序数组，搜索树；
* 哈希表结构适用于等值查询的场景
* 有序数组适用于静态存储引擎，查询多，改动少；
* 二叉搜索树:查找 O(logN)，插入也是O(logN)；

### Innodb的索引模型

1.每一个索引在 innodb里对应一棵B+树；

2.根据叶子节点内容分：主键索引和非主键索引：

* 主键索引：叶子节点存整行数据；
* 非主键索引：叶子节点存主键的值；

3.**提问：基于主键索引和普通索引的查询有什么区别？**

* 查询语句：`select * from T where ID = 1`主键查询方式，只搜索ID这棵B+树；
* 查询语句：`select * from T where k = 1`，普通索引方式，先找k索引B+树，找到对应的ID，再找ID索引树。**回表**； 

4.普通索引的查询会多扫描一棵索引树；

5.Innodb是一棵N叉树，N的值是1200；

6.提问：一张没有主键的表，只有1个普通索引，查询时怎么回表？

   Innodb默认会将RowId作为主键；

7.思考题：`老师你好：之前看过一遍文章，一直有疑惑：一个innoDB引擎的表，数据量非常大，根据二级索引搜索会比主键搜索快，文章阐述的原因是主键索引和数据行在一起，非常大搜索慢，我的疑惑是：通过普通索引找到主键ID后，同样要跑一遍主键索引，还望老师解惑。。。`

8. B+树的插入可能会引起数据页的分裂，删除可能会引起数据页的合并，二者都是比较重的IO消耗，所以比较好的方式是顺序插入数据，这也是我们一般使用自增主键的原因之一。



### 覆盖索引

1.回表：回到主索引树搜索的过程，普通索引的查找就需要；

2.示例：`select ID from T where k between 3 and 5`，需要查找的ID在k索引树上已经存在了，可以直接获取结果，不需要回表。简而言之，在**这个查询里索引k已经覆盖了查询需求，称为覆盖索引**

3.**重点：覆盖索引可以减少树的搜索次数，提升查询性能，所以使用覆盖索引是常用的性能优化手段**；

4.思考通过身份证号去查询姓名，在身份证号建立索引 和 （身份证号、姓名）联合索引对比？

   使用联合索引可用到覆盖索引，不需要回表查，减少了执行时间

### 最左前缀原则

1.B+ 树这种索引结构，可以利用索引的“最左前缀”，来定位记录，最左前缀可以是联合索引的最左N个字段，也可以是字符串索引的最左M个字符；

示例，查询（姓名，年龄）联合索引的表，找已张开头的数据`where name like '张%'`索引仍然有效

2.联合索引内的字段顺序如何安排？

   **重点：第一原则，如果通过调整顺序，可以少维护一个索引，那么这个顺序就是优先考虑的**，示例（a, b）联合索引，则不需要在a上建立索引了。

3.MySQL5.6之后引入了**索引下推**，在索引遍历过程中，对索引包含的字段进行判断，过滤掉不满足条件的记录，减少回表次数；

4.提问，什么情况下需要重建索引？

  索引因为删除，或者页分裂等原因，使得数据页有空洞，重建索引会创建新的索引，把数据按顺序插入，使得页面利用率最高。语句：`Alter table T engine=Innodb`

示例：`让我想到了我们线上的一个表, 记录日志用的, 会定期删除过早之前的数据. 最后这个表实际内容的大小才10G, 而他的索引却有30G. 在阿里云控制面板上看,就是占了40G空间. 这可花的是真金白银啊.后来了解到是 InnoDB 这种引擎导致的,虽然删除了表的部分记录,但是它的索引还在, 并未释放.只能是重新建表才能重建索引.`

### 提问F&Q

1.联合索引的技巧？（覆盖索引，最左前缀原则，索引下推）

2.**好问题：**老师，下面两条语句有什么区别，为什么都提倡使用2:
   `1.select * from T where k in(1,2,3,4,5)`
   `2.select * from T where k between 1 and 5`

   第1个树搜索5次，第2个树搜索1次。


# MySQL基础06-07

极客时间《MySQL实战45讲》

## 锁的分类

**按照加锁范围，分为：全局锁、表级锁、行级锁**

### 全局锁

1.对整个数据库实例加锁，MySQL提供了加全局读锁的方法，命令`flush tables with read lock`（FTWRL），让库处于只读状态；

2.应用场景：全库逻辑备份；

3.备份期间不加锁有什么问题？得到的备份库不是一个逻辑时间点的，这个视图是逻辑不一致的；

4.给整个数据库加只读锁，为什么不用`set global readonly=true`的方式呢？

 （a.readonly可能用于其他逻辑，比如判断是主库还是从库；b.异常处理机制上，如果客户端发生异常断开，FTWRL方式会自动释放全局锁，而设置readonly的方式，数据库会一直保持改状态）

5.备份方式，官方自带的逻辑备份工具mysqldump，使用参数`--single-transaction`时，会启动一个事务，确保拿到一致性视图。

### 表级锁

1.MySQL里有2种表级锁：**表锁、元数据锁**（meta data lock，MDL）

2.表锁语法`lock tables ... read/write`，可通过`unlock tables`主动释放表锁，

3.lock tables不仅限制别的线程读写，也限制本线程的操作；

4.MDL锁是系统默认会加的，**作用防止DDL和DML并发的冲突**，保证读写正确性，对一个表做增删改查时，加MDL读锁；对表结构变更时，加MDL写锁；即（MDL不需要显示使用，在访问一个表时自动加上）

5.MDL直到数据提交才会释放；

5.**思考：给一个小表加个字段，导致整个库挂了？**

​    原因：先查询，加了MDL读锁，再改表结构，加了MDL写锁，两个事务都没提交，导致后续操作会阻塞，如果客户端重试，库的线程很快爆满。

### 行锁

MySQL的行锁由各个存储引擎自己实现，如MyISAM不支持行锁，任何一个更新都会锁住整张表



两阶段锁协议：Innodb事务中，行锁在需要的时候才加上，需要等到事务结束时才释放，而不是不需要了立刻释放；

**死锁和死锁检测**

并发系统中，不同线程出现循环资源依赖，都在等待其他线程释放资源时，会进入无限等待状态，即死锁。



死锁有2种策略解决：

1.进入等待，直到超时，可通过参数`innodb_lock_wait_timeout`来设置，默认50s；

2.发起死锁检测，发现死锁后，主动回滚死锁链中的一个事务，使得其他事务能执行，设置参数`innodb_deadlock_detect`为on开启死锁检测；

**死锁检测带来的问题？**

`当一个事务被锁，就要看看它所依赖的线程有没有被别人锁住，如此循环，最后判断是否出现循环等待，即死锁`。假设1000个线程同时更新同一行，死锁检测的操作就是100万量级的，结果：CPU利用率很高，每秒执行事务却很少。

上述问题解决方式：

1.控制并发度（客户端控制可能不太行，因为客户端会有多个，如果有中间件，在中间件控制）

2.从业务设计上进行优化，将一行的改动逻辑分成多行，减少锁冲突；

**提问：**

1.死锁检测什么时候执行？ 在事务需要加锁访问的行上有锁，才要检测；一致性读不会加锁，故不需要死锁检测；

2.Innodb行级锁通过锁索引记录实现，如果update的列没建索引，即使update一条记录也会锁整张表吗？

  （隔离级别是RR，会的；隔离级别是RC，不会，MySQL做了优化的）

## 一致性读

事务查询数据，在这期间，即使数据被改过，但是事务看到的数据结果都是一致的。称为一致性读。

判断逻辑：

一个数据版本，对于一个事务视图来说，除了自己的更新总是可见以外，还有3种情况

`1.版本未提交，不可见`

`2.版本已提交，但是是在视图创建后提交的，不可见；`

`3.版本已提交，且是在视图创建前提交的，可见;`



**更新数据都是先读后写的，这个读只能读取当前值，即“当前读”**。（更新和查询的区别）

例如：事务B执行更新语句，这期间事务C已经更新k=k+1，那么事务B更新时读到的k=2，更新后k=3;

**5.为什么rr能实现可重复读而rc不能,分两种情况**
(1)快照读的情况下,rr不能更新事务内的up_limit_id,
  而rc每次会把up_limit_id更新为快照读之前最新已提交事务的transaction id,则rc不能可重复读
(2)当前读的情况下,rr是利用record lock+gap lock来实现的,而rc没有gap,所以rc不能可重复读
















