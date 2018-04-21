---

layout: post
title: 《mysql技术内幕》笔记2
category: 技术
tags: Architecture
keywords: mysql innodb

---

## 简介

## 索引

全文索引

### 红黑树

[红黑树深入剖析及Java实现](https://tech.meituan.com/redblack-tree.html)

基本要点：BST，Balanced BST，Red-Black Tree

1. 二叉树在插入的时候会导致树倾斜，不同的插入顺序会导致树的高度不一样，而树的高度直接的影响了树的查找效率。平衡树在插入和删除的时候，会通过旋转操作将高度保持在logN。具有代表性的平衡树分别为AVL树和红黑树。AVL树由于实现比较复杂，而且插入和删除性能差，在实际环境下的应用不如红黑树。
2. 二叉树通过旋转保持平衡，旋转算法是不同树的**具有区分性的特质**之一。红黑树通过引入颜色的概念，通过颜色这个约束条件的使用来保持树的高度平衡。

||插入维持平衡|删除维持平衡|
|---|---|---|
|二叉树|旋转|旋转|
|B+Tree|节点的分裂、元素的上浮|向兄弟节点借元素、合并节点、元素的下沉|

二叉树与高阶（一个节点多于2个叉）树不同在于，高阶树中每个节点元素的个数,为旋转算法提供了决策依据，体现在：

1. 是否需要分裂合并等操作
2. 大多数时候，一个插入和删除操作影响节点本身、父亲和兄弟节点，通过父节点元素个数，可以简单的判断是否波及其他节点。

而对于二叉树来说，这些都可以实现，大不了整棵树遍历一遍。但没有类似于“节点个数”（因为都是一个）这样直接的信息来提供决策，这也是为什么说“AVL树实现比较复杂”的原因吧。红黑树，节点的颜色标记，估计与“节点个数”有异曲同工的作用。有一句话说，“纠结的原因通常是因为掌握的信息不够（或信息获取比较难）”，其存在标识一种平衡状态，通过局部的直接判断，即可进行旋转决策。

## 锁

书中提到，在数据库中，锁有lock和latch，一般业务开发熟悉的锁对应的是latch，简单区别如下：

||对象|保护|持续时间|存在于|
|---|---|---|---|---|
|lock|事务|表、页、行|整个事务过程|lock manager的哈希表中|
|latch|线程|内存数据结构|很短|每个数据结构的对象中|

比如在java中，一个object内存结构就相应有锁的标记位，意味着任何一个object都有可能被竞争访问，如果object已经被锁住，则线程会被挂起。lock的主角是事务，是否意味着如果事务被挂起，执行事务的线程可以去执行其它事务呢？类似的，联系到go语言中，io和chain操作阻塞的对象是goroutine，不妨碍实际线程的执行。

其实，锁的标记信息存储在实际的数据结构上还是独立集中管理，都是一样的。

1. 在操作系统中，一个文件在磁盘上的存在形式是一个个磁盘块，在内存中的存在形式除了磁盘块载入内存的缓冲块外，还有一个文件表，表中的文件结构体有锁的标志位。文件是否被某个线程独占，并不属于文件的内容信息，存入磁盘中是不恰当的。如果锁的信息存入磁盘块对应的缓冲块，则破坏了缓冲块与磁盘块的直接对应关系。
2. 每个数据结构保有锁的标记信息有一个好处，即语言层面简化锁的使用，比如java的synchronized关键字， 比`lock unlock`方便多了。

## 事务

[理解事务 - MySQL 事务处理机制](https://juejin.im/entry/58f08b4cda2f60005d225a8e)基本要点（太经典，要低水平的复制粘贴了）：

重新理解一致性：在事务T开始时，此时数据库有一种状态，这个状态是所有的MySQL对象处于一致的状态，例如数据库完整性约束正确，日志状态一致等，当事务T提交后，这时数据库又有了一个新的状态，不同的数据，不同的索引，不同的日志等，但此时，**约束，数据，索引，日志等MySQL各种对象还是要保持一致性（正确性）。** 这就是 从一个一致性的状态，变到另一个一致性的状态。也就是事务执行后，并没有破坏数据库的完整性约束（一切都是对的）。

重新理解隔离性：即要达到这么一种效果：对于任意两个并发的事务T1和T2，在事务T1看来，T2要么在T1开始之前就已经结束，要么在T1结束之后才开始，这样每个事务都感觉不到有其他事务在并发地执行。


|事务|加解锁阶段|
|---|---|
|begin;|获取唯一自增的事务id等操作|
|insert...|加insert对应的锁|
|update...|加update对应的锁|
|delete...|加delete对应的锁|
|commit;|事务提交时，同时释放insert、update、delete对应的锁|

可以看到，begin和commit除了标记一个事务的开始与结束外，在数据库实现中，是有对应的操作意义的。

## 事务隔离级别与锁的关系

### 一堆琐碎

下面会列举一些概念，教科书式的一种罗列，害人不浅，但作为背景，还是要系统介绍一下

[Innodb中的事务隔离级别和锁的关系](https://tech.meituan.com/innodb-lock.html)基本要点:

1. 一致性和隔离性一般都通过加锁来实现
2. 不同的隔离性要求，加锁时机和类型都有所不同


|事务级别|脏读|不可重复读|幻读|加锁实现，具体的三种增删改写操作加的锁视情况不一样|rr：读取正在读取的行；rw 读取正在写的行；wr、ww类似|
|---|---|---|---|---|---|
|Read Uncommitted|可能|可能|可能|任何操作都不会加锁|rr,rw,wr,ww都允许|
|已提交读Read committed|不可能|可能|可能|读取不加锁，增删改加锁|rr wr允许;rw rw看是否是一致性锁定读；ww不允许|
|可重复读Repeatable read	|不可能|不可能|可能|悲观锁方案：读加共享锁，写加排他锁；乐观锁方案：xx；|rr允许；rw看是否是一致性锁定读；ww、wr不允许|
|可串行化 Serializable |不可能|不可能|不可能|悲观锁方案：读加共享锁，写加排他锁，读写互斥（读的时候不可以写操作，哪怕不是同一行记录）；乐观锁方案：不可行|除同一行严格要求外，不同行wr也不允许|

很多人容易搞混不可重复读和幻读，确实这两者有些相似。但不可重复读重点在于update和delete（对先后读取操作的影响），而幻读的重点在于insert。

||表现|
|---|---|---|
|悲观锁|读写加锁，写加排他锁，读加什么锁视情况|
|乐观锁|乐观锁其实并不是真正的锁机制，它只是一种思想。每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在提交更新的时候会判断一下在此期间别人有没有去更新这个数据。|

读操作，可以通过`select xx for update` 的方式对行加排他锁。

锁定读和非锁定读，是看待并发问题的另一维度。就像《Java并发编程实战》中提到的，解决并发问题的办法之一是：不要并发访问。还有副本、老版本的方式。

||读取已被排他锁占用的行|
|---|---|
|一致性锁定读|无法读取|
|一致性非锁定读|读取老版本数据|

以上都是从读的角度看，从写的角度看：事务隔离性要求，insert操作的记录只对事务本身可见，对其它事务不可见。

### 一些啰嗦

并发访问会带来哪些问题，有哪些控制手段？

我们日常代码的并发，若不采取措施，常见的就是数据错乱，另外，因为cpu缓存和内存一致性的缘故，会有一个可见性问题。一般业务synchronized关键字是够用的，从隔离级别上讲是Serializable。当然，在特定的场景下，java语言层面上AQS以及各种lock提供了许多优化手段。

本文是innodb的读书笔记，更宏观的看待并发问题请参考[腾讯云李海翔：数据库的并发控制技术深度探索](http://www.techweb.com.cn/news/2017-05-15/2523622.shtml)基本要点：

1. 数据库一共会发生11种异常现象，脏读、不可重复读、幻读只是其中三种。
2. 主流的并发控制技术

	* 两阶段锁
	* 基于时间戳
	* 基于有效性检查
	* MVCC，常与其它技术一起使用
	* SCO
3. 为什么并发控制技术能解决数据异常现象？

	* 基于封锁的并发控制技术，读写、写读、写写三种冲突情况用锁规则固化，不让它们并发进行。
	* 其他的并发控制技术，本质上都是定义了一些规则，用来约束并发的读写操作、提交顺序、并决定回滚哪些事务作为牺牲者等。

	小结一下，**所谓并发控制技术就是抑制并发，或者发现数据异常并处理。**目的是为了保证一致性，但绝对的一致性要损失性能，因此允许某些异常便带来隔离性的概念，调出几个常见的数据异常问题划分隔离性，总比不可重复读等说一堆，减少了沟通成本。
	
	数据库的并发安全，跟java语言层面的并发安全又有所不同，java语言层面就两个安全级别：安全，不安全。而数据库提出隔离性的概念，并将隔离性分级，一般mysql 引擎仅实现部分并发安全。除此之外，除非隔离性要求很高，否则尽量避免动用锁。
	
## 从redo log想到的

事务狭义的说，是一个sql操作序列，广义的说，异构的和分布式的操作序列都可以。redo log主要用于保证事务的持久性

1. 数据写入一般先保存在内存中，定期将内存数据写入到磁盘，以提高性能
2. 数据写磁盘一般是随机的，单次较慢，也不允许频繁写入
3. 磁盘的顺序写性能较高，所以先将redo log持久化到磁盘，便可以快速的保存操作信息。

在支付业务中，有一个用户账户表，还会有一个用户账户临时表，更新用户账户的金额数据时，经常先在临时表中先插入一条日志，因为只有插入操作，自然没有并发问题，然后再去更新用户账户。此时，临时表的作用就类似于redo日志。

## 数据库和文件系统（未完成）

书中有多处提到数据库和文件系统的关系：

1. 事务是数据库区别于文件系统的重要特性之一
2. 数据库的主要任务就是协调对数据记录的并发访问