# 原子

原子性，也就是事务中的操作要么全部完成，要么什么也不做
每当我们要对一条记录做改动
把回滚时所需的东西都给记下来

为了回滚而记录的这些东东称之为撤销日志，英文名为undo lo

为了实现事务的原子性，InnoDB存储引擎在实际进行增、删、改一条记录时，都需要先把对应的undo日志记下来。


# 事务id


给事务分配id的时机

，只有第一次对某个用户创建的临时表执行增、删、改操作时才会为这个事务分配一个事务id


事务id是怎么生成的
这个事务id本质上就是一个数字，它的分配策略和我们前边提到的对隐藏列row_id（当用户没有为表创建主键和UNIQUE键时InnoDB自动创建的列）的分配策略大抵相同

行记录trxId
InnoDB记录行格式的时候重点强调过：聚簇索引的记录除了会保存完整的用户数据以外，而且还会自动添加名为trx_id、roll_pointer的隐藏列


# roll_pointer

是一个指向记录对应的undo日志的一个指针




# INSERT操作对应的undo日志

插入，最终导致的结果就是这条记录被放到了一个数据页中。如果希望回滚这个插入操作，那么把这条记录删除就好了，

也就是说在写对应的undo日志时，主要是把这条记录的主键信息记上。


# DELETE操作对应的undo日志



删除的过程需要经历两个阶段

阶段一：仅仅将记录的delete_mask标识位设置为1

在删除语句所在的事务提交之前，被删除的记录一直都处于这种所谓的中间状态

阶段二：当该删除语句所在的事务提交之后，会有专门的线程后来真正的把记录删除掉。

所谓真正的删除就是把该记录从正常记录链表中移除，并且加入到垃圾链表中，



# UPDAh





TE操作对应的undo日志


在不更新主键的情况下，又可以细分为被更新的列占用的存储空间不发生变化和发生变化的情况。


就地更新（in-place update）

更新记录时，对于被更新的每个列来说，如果更新后的列和更新前的列占用的存储空间都一样大，

先删除掉旧记录，再插入新记录

在不更新主键的情况下，如果有任何一个被更新的列更新前和更新后占用的存储空间大小不一致，

那么就需要先把这条旧的记录从聚簇索引页面中删除掉，然后再根据更新后列的值创建一条新的记录插入到页面中。


更新主键的情况
在聚簇索引中，记录是按照主键值的大小连成了一个单向链表的，如果我们更新了某条记录的主键值，意味着这条记录在聚簇索引中的位置将会发生改变

将旧记录进行delete mark操作

根据更新后各列的值创建一条新记录，并将其插入到聚簇索引中（需重新定位插入的位置）



在一个事务执行过程中，可能混着执行INSERT、DELETE、UPDATE语句，也就意味着会产生不同类型的undo日志。但是我们前边又强调过，同一个Undo页面要么只存储TRX_UNDO_INSERT大类的undo日志，要么只存储TRX_UNDO_UPDATE大类的undo日志，反正不能混着存，所以在一个事务执行过程中就可能需要2个Undo页面的链表，一个称之为insert undo链表，另一个称之为update undo链表

多个事务中的Undo页面链表


# 版本链



# 

undo日志被存放到了类型为FIL_PAGE_UNDO_LOG的页面中  

Undo页面

之所以把undo日志分成两个大类，是因为类型为TRX_UNDO_INSERT_REC的undo日志在事务提交后可以直接删除掉，
95
​
96
而其他类型的undo日志还需要为所谓的MVCC服务，不能直接删除掉，对它们的处理需要区别对待。
97
​
98
如果有更多的事务，那就意味着可能会产生更多的Undo页面链表。


99 一般来说一个Undo页面链表只存储一个事务执行过程中产生的一组undo日志，


我们前边说为了能提高并发执行的多个事务写入undo日志的性能，设计InnoDB的大叔决定为每个事务单独分配相应的Undo页面链表（最多可能单独分配4个链表）。但是这样也造成了一些问题，比如其实大部分事务执行过程中可能只修改了一条或几条记录，针对某个Undo页面链表只产生了非常少的undo日志，这些undo日志可能只占用一丢丢存储空间，每开启一个事务就新创建一个Undo页面链表（


# 重用Undo页面








# 回滚段


回滚段数量决定最高事务数

多个回滚段
我们说一个事务执行过程中最多分配4个Undo页面链表，而一个回滚段里只有1024个undo slot，很显然undo slot的数量有点少啊。我们即使假设一个读写事务执行过程中只分配1个Undo页面链表，那1024个undo slot也只能支持1024个读写事务同时执行，再多了就崩溃了。这就相当于会议室只能容下1024个班长同时开会，如果有几千人同时到会议室开会的话，那后来的那些班长就没地方坐了，只能等待前边的人开完会自己再进去开。

话说在InnoDB的早期发展阶段的确只有一个回滚段，但是设计InnoDB的大叔后来意识到了这个问题，咋解决这问题呢？会议室不够，多盖几个会议室不就得了。所以设计InnoDB的大叔一口气定义了128个回滚段，也就相当于有了128 × 1024 = 131072个undo slot。假设一个读写事务执行过程中只分配1个Undo页面链表，那么就可以同时支持131072个读写事务并发执行（这么多事务在一台机器上并发执行，还真没见过呢～）。



