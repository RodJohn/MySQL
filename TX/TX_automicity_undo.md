

# 分类

    是否涉及到对原先记录的修改分类 
      
    1.insert undo log
    指在insert操作中产生的undo log，因为insert操作的记录只对事务本身可见。因此该undo log在事务提交后直接删除，不需要进行purge操作。
    
    2.update undo log
    记录的是对delete和update操作产生的undo log，该log需要提供mvcc机制，因此不能在事务提交时就进行删除。提交时放入undo log链表，等待purge线程进行清除。
  


# undo页链

undo页链表

    undo日志被存放到undo页中
    不同类型的undo日志(insert/update)放在不同的undo页中  
    不同类型的undo页形成不同的undo链表
    每个事务单独分配相应的Undo页面链表
    （最多可能单独分配4个链表，临时表和物理表不通用）。



# 回滚段

    undo页链表都存放在回滚段中

回滚段数量决定最高事务数

    我们说一个事务执行过程中最多分配4个Undo页面链表，而一个回滚段里只有1024个undo slot，
    很显然undo slot的数量有点少啊。我们即使假设一个读写事务执行过程中只分配1个Undo页面链表，那1024个undo slot也只能支持1024个读写事务同时执行，
    
    
    话说在InnoDB的早期发展阶段的确只有一个回滚段，但是设计InnoDB的大叔后来意识到了这个问题，咋解决这问题呢？
    会议室不够，多盖几个会议室不就得了。所以设计InnoDB的大叔一口气定义了128个回滚段，也就相当于有了128 × 1024 = 131072个undo slot。
    假设一个读写事务执行过程中只分配1个Undo页面链表，那么就可以同时支持131072个读写事务并发执行（这么多事务在一台机器上并发执行，还真没见过呢～）。


# 删除

    当事务提交的时候，innodb不会立即删除undo log，因为后续还可能会用到undo log


# undo log 的删除

    为了支持快照读，对于delete mark操作来说，仅仅是在记录上打一个删除标记，并没有真正将它删除掉。

    当系统里没有比这个回滚日志更早的read-view的时候

    有一个后台运行的purge线程会把它们真正的删除掉

# 查看

show  ENGINE INNODB status

TRANSACTIONS

History list length 328




# 文件

当我们对数据进行操作的时候，就会产生undo记录，
Undo记录默认记录在系统表空间（ibdata）中，从MySQL 5.6开始，Undo使用的表空间可以分离为独立的Undo log文件。

如果开启了 innodb_file_per_table ，将放在每个表的.ibd文件中。



重用Undo页面

    但是这样也造成了一些问题，比如其实大部分事务执行过程中可能只修改了一条或几条记录，
    针对某个Undo页面链表只产生了非常少的undo日志，
    这些undo日志可能只占用一丢丢存储空间，每开启一个事务就新创建一个Undo页面链表

