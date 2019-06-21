# 数据修改

行锁  必须提交后释放

在innodb中，一个事物可以对一行数据进行多次修改，只有在


能不能在两个事务中交叉更新同一条记录呢？

哈哈，这不就是一个事务修改了另一个未提交事务修改过的数据，沦为了脏写了么？

InnoDB使用锁来保证不会有脏写情况的发生，也就是在第一个事务更新了某条记录后，就会给这条记录加锁，另一个事务再次更新时就需要等待第一个事务提交了，

把锁释放之后才可以继续更新。


# 版本链


每次对记录进行改动，都会记录一条undo日志，
roll_pointer：

每次对某条聚簇索引记录进行改动时，都会把旧的版本写入到undo日志中，然后这个隐藏列就相当于一个指针，可以通过它来找到该记录修改前的信息。

每条undo日志也都有一个roll_pointer属性（INSERT操作对应的undo日志没有该属性，因为该记录并没有更早的版本），

可以将这些undo日志都连起来，串成一个链表，


我们说insert undo在事务提交之后就可以被释放掉了，而update undo由于还需要支持MVCC，不能立即删除掉。



支持MVCC，对于delete mark操作来说，仅仅是在记录上打一个删除标记，并没有真正将它删除掉。


# ReadView
数据的变更记录就在版本连中
31
​
不同隔离级别


核心问题就是：需要判断一下版本链中的哪个版本是当前事务可见的




ReadView中主要包含4个比较重要的内容

creator_trx_id：表示生成该ReadView的事务的事务id

m_ids：表示在生成ReadView时当前系统中活跃的读写事务的事务id列表。

min_trx_id：表示在生成ReadView时当前系统中活跃的读写事务中最小的事务id，也就是m_ids中的最小值。

max_trx_id：表示生成ReadView时系统中应该分配给下一个事务的id值。

如果某个版本的数据对当前事务不可见的话，那就顺着版本链找到下一个版本的数据，继续按照上边的步骤判断可见性，依此类推，直到版本链中的最后一个版本。如果最后一个版本也不可见的话，那么就意味着该条记录对该事务完全不可见，查询结果就不包含该记录。



如果被访问版本的trx_id属性值与ReadView中的creator_trx_id值相同，意味着当前事务在访问它自己修改过的记录，所以该版本可以被当前事务访问。

如果被访问版本的trx_id属性值小于ReadView中的min_trx_id值，表明生成该版本的事务在当前事务生成ReadView前已经提交，所以该版本可以被当前事务访问。

如果被访问版本的trx_id属性值大于ReadView中的max_trx_id值，表明生成该版本的事务在当前事务生成ReadView后才开启，所以该版本不可以被当前事务访问。

如果被访问版本的trx_id属性值在ReadView的min_trx_id和max_trx_id之间，那就需要判断一下trx_id属性值是不是在m_ids列表中，如果在，说明创建ReadView时生成该版本的事务还是活跃的，该版本不可以被访问；如果不在，说明创建ReadView时生成该版本的事务已经被提交，该版本可以被访问。







接下来看一下READ COMMITTED和REPEATABLE READ所谓的生成ReadView的时机不同到底不同在哪里。


READ COMMITTED —— 每次读取数据前都生成一个ReadView



REPEATABLE READ —— 在第一次读取数据时生成一个ReadView






随着系统的运行，在确定系统中包含最早产生的那个ReadView的事务不会再访问某些update undo日志以及被打了删除标记的记录后，有一个后台运行的purge线程会把它们真正的删除掉



