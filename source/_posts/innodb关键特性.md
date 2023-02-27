---
title: innodb关键特性
top: false
cover: false
toc: true
mathjax: true
date: 2021-04-27 21:49:16
password:
summary:
tags:
categories:
---
# Innodb关键特性

## 插入缓冲

### Insert Buffer

概念：对于**非聚集索引的插入或更新操作**，先判断插入的非聚集索引页是否在缓冲池中，在则直接插入。不在，先放入到insert buffer对象中，在一定情况下对insert buffer和辅助索引页子节点merge操作。 （可将多个插入合并到一个操作中，大大提高了对非聚集索引的插入性能）

使用需要满足2个条件：

* 索引是辅助索引
* 索引不是唯一的 （因为insert buffer不查找索引页判断插入记录唯一性）

通过`show engine innodb status`查看Innodb引擎状态，找到`INSERT BUFFER AND ADAPTIVE HASH INDEX`

```shell
-------------------------------------
INSERT BUFFER AND ADAPTIVE HASH INDEX
-------------------------------------
Ibuf: size 1, free list len 0, seg size 2, 0 merges
merged operations:
 insert 0, delete mark 0, delete 0
discarded operations:
 insert 0, delete mark 0, delete 0
Hash table size 138389, node heap has 0 buffer(s)
Hash table size 138389, node heap has 1 buffer(s)
Hash table size 138389, node heap has 0 buffer(s)
Hash table size 138389, node heap has 1 buffer(s)
Hash table size 138389, node heap has 1 buffer(s)
Hash table size 138389, node heap has 1 buffer(s)
Hash table size 138389, node heap has 1 buffer(s)
Hash table size 138389, node heap has 76 buffer(s)
0.00 hash searches/s, 0.00 non-hash searches/s
---
```



### change buffer

Innodb可对DML操作进行缓冲，包括：INSERT、DELETE、UPDATE，对应`insert buffer`、`delete buffer`、`purge buffer`；

适用对象是：非唯一的辅助索引；

参数`innodb_change_buffering`可设置缓冲类型：insert、delete、purges、changes、all等等；

对一条记录的UPDATE过程：

* 先将记录标记已删除  （对应delete buffer，状态的delete mark）
* 真正删除记录  （对应purge buffer，状态的delete）



## 两次写

doublewrite用于提升Innodb的数据页可靠性。

在应用重做日志前，用户需要一个页的副本，当写入失效时，先通过页的副本还原该页，再进行重做，即doublewrite；

**写过程**：在对缓冲池刷脏页时，不是直接写磁盘，而是将脏页复制到内存中的doublewrite buffer，之后doublewrite buffer分2次，每次1MB顺序写入共享表空间的物理磁盘上，最后同步磁盘。



## 自适应哈希索引

Innodb会监控表上个索引页的查询，如果观察到建立哈希索引可以提升性能，Innodb会自己建立哈希索引，即自适应哈希索引（Adaptive Hash Index，AHI）；

使用要求：

* 等值查询
* 以该模式访问100次
* 页通过该模式访问了N次，N=页中记录/16



## 异步IO

AIO，用户发出一个IO请求后，可直接发下一个IO请求，不用等待响应。AIO还可将多个IO请求merge为1个IO，提高了IOPS性能；



## 刷新邻接页

刷脏页时，Innodb检测该页所在区（extent）的所有页，如果是脏页，一并刷新。固态硬盘已经有很强的IO能力，可关闭该功能；











