---
title: 什么是MVCC | 数据库
cover_image: /img/cover.png
abbrlink: 44d2fb57
date: 2022-10-14 15:13:05
---

## 

## 什么是MVCC

全称Multi-Version Concurrency Control，即多版本并发控制，主要是为了提高数据库的并发性能。

同一行数据平时发生读写请求时，会上锁阻塞住。但mvcc用更好的方式去处理读—写请求，做到在发生读（指快照读）—写请求冲突时不用加锁。

## 当前读、快照读

### 当前读

每次读取数据库记录，都是当前最新的版本，会对当前读取的数据进行加锁，防止其他事务修改数据，是悲观锁。

如下操作都是当前读：

* select lock in share mode (共享锁)
* select for update (排他锁)
* update (排他锁)
* insert (排他锁)
* delete (排他锁)
* 串行化事务隔离级别
### 快照读

快照读的实现是基于多版本并发控制，即MVCC（快照读读到的数据不一定是当前最新的数据，有可能是之前历史版本的数据）

## 数据库并发场景

* 读-读：不存在任何问题，也不需要并发控制
* 读-写：有线程安全问题，可能会造成事务隔离性问题，可能遇到脏读，不可重复读，幻读
* 写-写：有线程安全问题，可能会存在更新丢失问题（第一类更新丢失，第二类更新丢失
## MVCC解决的问题

* 并发读-写时：可以做到读操作不阻塞写操作，同时写操作也不会阻塞读操作。
* 解决脏读、幻读、不可重复读等事务隔离问题，但不能解决上面的写-写 更新丢失问题。
MVCC不解决写写冲突，所以可以：

* MVCC + 悲观锁：MVCC解决读写冲突，悲观锁解决写写冲突
* MVCC + 乐观锁：MVCC解决读写冲突，乐观锁解决写写冲突
## MVCC的实现原理

mvcc用来解决读—写冲突的无锁并发控制，为事务分配单向增长的时间戳，为每个数据修改保存一个版本，版本与事务时间戳相关联。它的实现原理主要是版本链，undo日志 ，Read View来实现的

### 版本链

数据库中的每行数据有几个隐藏字段，分别是db_trx_id、db_roll_pointer、db_row_id。

* db_trx_id

6byte，最近修改(修改/插入)事务ID：记录创建这条记录/最后一次修改该记录的事务ID。

* db_roll_pointer（版本链关键）

7byte，回滚指针，指向这条记录的上一个版本,配合undo日志

* db_row_id

6byte，隐含的自增ID（隐藏主键），如果数据表没有主键，InnoDB会自动以db_row_id产生一个聚簇索引。

* 删除flag隐藏字段

记录被更新或删除并不代表真的删除，而是删除flag变了

### undo日志

Undo log 主要用于记录数据被修改之前的日志，在表信息修改之前先会把数据拷贝到undo log里。

当事务进行回滚时可以通过undo log 里的日志进行数据还原。每次对数据库记录进行改动，都会记录一条undo日志，每条undo日志都有一个roll_pointer属性.

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8Rp3KGC8icicxiaibATQm49PMqjiaLNNtyjjeYIW9czJ2MDXXhw7JUbQQic0r1RoaWTyKzVVMkkanyoMEgg/640?wx_fmt=png)Undo log 的用途

* 保证事务进行rollback时的原子性和一致性，当事务进行回滚的时候可以用undo log的数据进行恢复。
* 用于MVCC快照读的数据，在MVCC多版本控制中，通过读取undo log的历史版本数据可以实现不同事务版本号都拥有自己独立的快照数据版本。
undo log主要分为两种：

* insert undo log

代表事务在insert新记录时产生的undo log , 只在事务回滚时需要，并且在事务提交后可以被立即丢弃

* update undo log（主要）

事务在进行update或delete时产生的undo log ; 不仅在事务回滚时需要，在快照读时也需要；

### Read View(读视图)

事务进行快照读操作的时候会产生一个读视图(Read View)。

记录并维护系统当前活跃事务的ID(没有commit)，是系统中当前不应该被本事务看到的其他事务id列表。

Read View主要是用来做可见性判断的, 即当某个事务执行快照读的时候，会创建一个Read View读视图，把它比作条件用来判断当前事务能够看到哪个版本的数据，既可能是当前最新的数据，也有可能是该行记录的undo log里面的某个版本的数据。

Read View几个属性

* trx_ids: 当前系统活跃(未提交)事务版本号集合。
* low_limit_id: 创建当前read view 时“当前系统最大事务版本号+1”。
* up_limit_id: 创建当前read view 时“系统正处于活跃事务最小版本号”
* creator_trx_id: 创建当前read view的事务版本号；
### Read View可见性判断条件

* db_trx_id < up_limit_id || db_trx_id == creator_trx_id（显示）

数据事务ID小于read view中的最小活跃事务ID：该数据在当前事务启之前就已经存在,可以显示。

数据的事务ID等于creator_trx_id ：数据是当前事务自己生成的

* db_trx_id >= low_limit_id（不显示）

数据事务ID大于read view 中的当前系统的最大事务ID：数据是在当前read view 创建之后才产生的，数据不显示。

* db_trx_id是否在活跃事务（trx_ids）中

* 不存在：说明read view产生的时候事务已经commit了，数据则可以显示。
* 已存在：Read View生成时刻，事务还在活跃，没有Commit，不可见。
## MVCC和事务隔离级别

上面所讲的Read View用于支持RC（Read Committed，读提交）和RR（Repeatable Read，可重复读）隔离级别的实现。

### RR、RC生成时机

* RC隔离级别下，是每个快照读都会生成并获取最新的Read View；
* 而在RR隔离级别下，则是同一个事务中的第一个快照读才会创建Read View, 之后的快照读获取的都是同一个Read View，之后的查询就不会重复生成了，所以一个事务的查询结果每次都是一样的。
### 解决幻读问题

* 快照读：通过MVCC来进行控制的，不用加锁。按照MVCC中规定的“语法”进行增删改查等操作，以避免幻读。
* 当前读：通过next-key锁（行锁+gap锁）来解决问题的。

