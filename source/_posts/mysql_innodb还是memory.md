---
title: innodb还是memory
date: 2020-03-25 09:32:53
tags: MySQL
categories: MySQL
---

>  都说innodb好、还要不要用memory引擎 ?

#### 内存表的数据组织结构
innodb表的数据就放在主键索引树上、主键索引是 B+ 树、主键索引上是有序存储的, 在执行select  * 时、会按照叶子节点从左到右扫描、得到的结果 0 出现在第一行

memory引擎的数据和索引是分开的. 主键id的索引里、存放的是每个数据的位置. 主键id是hash索引、索引上的key是乱序的. select * 也是全表扫描、也是顺序扫描数组、0就是最后一个被读到、并放入结果集的数据. 

innodb把数据放到主键索引上、其它索引保存的是主键id、是索引组织表(Index Organizied Table)
memory引擎采用的是数据单独存放、是堆组织表(Heap Organizied Table)

两者之间的差异:
1. innodb的数据总是有序存储的、内存表的数据是按照写入顺序存储的.
2. 当数据文件有空洞的时候、innodb表在插入新数据时、为了保证数据有序性、只能在固定位置写入新值, 而内存表找到空位置就可以插入新值.
3. 数据位置发生变化的时候、innodb表只需要修改主键索引、而内存表需要修改所有索引.s
4. innodb表用主键索引查询时、需要走一次索引查询、普通索引查询时、要走两次索引查找. 
   memory表无分别、所有的索引地位都是相同的.
5. innodb支持变长数据类型、不同记录的长度可能不同、内存表不支持Blob和Text字段、并且即使定义varchar(N)实际也是char(N), 即固定长度存储、所以: 每行数据长度相同.

所以: 内存表每行数据被删除以后、空出的位置可以被接下来要插入的数据复用.
注意: 内存表的索引是hash索引、范围查询 `select * from t1 where id<5` 是无法用到主键索引的, 会全表扫描.

#### hash索引 和 B-Tree索引
内存表也可以支持B-Tree索引

```sql
alter table t1 add index a_btree_index using btree(id);
```
内存表优势是速度快. 一方面它支持hash索引、另一方面、它的数据都在内存; 那么、为什么不建议生产环境使用内存表呢 ?
1. 锁粒度问题
2. 数据持久化问题.

#### 内存表的锁
内存表不支持行锁、只支持表锁、导致并发性太低.

#### 数据持久性问题
数据放在内存中是内存表的优势、也是劣势、因为DB重启时、所有的内存表都会被清空.

##### M-S 架构场景
1) 业务正常访问s主库
2) 备库硬件升级、备库重启、内存表t1内容被清空
3) 备库重启后、客户端发送一条update语句、修改表t1的数据行、此时备库应用线程就会找不到要更新的数据.

##### 双M架构
MySQL知道重启后、内存表的数据会丢失、所以担心主库重启后、出现主备不一致、在实现上、会在DB重启后写入一行 delete from t1的binlog记录.
备库重启时、备库的binlog里的delete语句就会传到主库、然后把主库内存表的内容删除, 出现主库内存表数据突然被清空的现象.

所以不适合在生产上使用.
1. 若选择内存表是因为更新量大、那么并发度是重要的参考指标、innodb支持行锁、并发度更好
2. 若考虑读性能、一个读QPS很高、且数据量不大的表、即使是innodb、数据也都是缓存在buffer pool的、因此innodb表性能也不差.

#### 一个适合用内存表的场景
内存临时表: 不会被其它线程访问、无并发问题; 重启需要删除、清空数据问题不存在; 备库的临时表不影响主库用户线程, 所以刚好可以无视内存表的两个不足.