
# 

bufferpool

（1）MySQL数据存储包含内存与磁盘两个部分；

（2）内存缓冲池(buffer pool)以页为单位，缓存最热的数据页(data page)与索引页(index page)；


https://mp.weixin.qq.com/s/nA6UHBh87U774vu4VvGhyw

# changebuffer

在MySQL5.5之前，叫插入缓冲(insert buffer)，只针对insert做了优化；现在对delete和update也有效，叫做写缓冲(change buffer)。

它是一种应用在非唯一普通索引页(non-unique secondary index page)不在缓冲池中，
对页进行了写操作，并不会立刻将磁盘页加载到缓冲池，而仅仅记录缓冲变更(buffer changes)，
等未来数据被读取时，再将数据合并(merge)恢复到缓冲池中的技术。写缓冲的目的是降低写操作的磁盘IO，提升数据库性能。


读数据随机IO


https://mp.weixin.qq.com/s/PF21mUtpM8-pcEhDN4dOIw


# 

情况一

假如要修改页号为4的索引页，而这个页正好在缓冲池内。
（1）直接修改缓冲池中的页，一次内存操作；
脏页 刷新
（2）写入redo log，一次磁盘顺序写操作；

什么时候缓冲池中的页，会刷到磁盘上呢？

定期刷磁盘，而不是每次刷磁盘，能够降低磁盘IO，提升MySQL的性能。

情况二

假如要修改页号为40的索引页，而这个页正好不在缓冲池内。

（1）先把需要为40的索引页，从磁盘加载到缓冲池，一次磁盘随机读操作；

（2）修改缓冲池中的页，一次内存操作；

（3）写入redo log，一次磁盘顺序写操作；

没有命中缓冲池的时候，至少产生一次磁盘IO，对于写多读少的业务场景，是否还有优化的空间呢？

这即是InnoDB考虑的问题，又是本文将要讨论的写缓冲(change buffer)。

加入写缓冲优化后，流程优化为：

（1）在写缓冲中记录这个操作，一次内存操作；

（2）写入redo log，一次磁盘顺序写操作；

其性能与，这个索引页在缓冲池中，相近。

画外音：可以看到，40这一页，并没有加载到缓冲池中。




（2）写缓冲不只是一个内存结构，它也会被定期刷盘到写缓冲系统表空间；

（3）数据读取时，有另外的流程，将数据合并到缓冲池；

    （1）载入索引页，缓冲池未命中，这次磁盘IO不可避免；
    
    （2）从写缓冲读取相关信息；
    
    （3）恢复索引页，放到缓冲池LRU里；
 
 # 写缓冲优化，仅适用于非唯一普通索引页呢
 
 
 如果索引设置了唯一(unique)属性，在进行修改操作时，InnoDB必须进行唯一性检查。
 也就是说，索引页即使不在缓冲池，磁盘上的页读取无法避免(否则怎么校验是否唯一？)，
 此时就应该直接把相应的页放入缓冲池再进行修改，而不应该再整写缓冲这个幺蛾子。
 
 
 


# 修改

insert

    这条记录被放到了一个数据页中
    内存
    
    每当新插入记录时，
    首先判断PAGE_FREE指向的头节点代表的已删除记录占用的存储空间是否足够容纳这条新插入的记录，
    如果不可以容纳，就直接向页面中申请新的空间来存储这条记录


delete 

    数据页中的正常记录会根据记录中的next_record属性组成一个单向链表，也就是记录链表
    被删除的记录也会根据记录中的next_record属性组成一个单向链表，也就是垃圾链表
    
    删除的过程需要经历两个阶段
    阶段一：仅仅将记录的delete_mask标识位设置为1
    在删除语句所在的事务提交之前，被删除的记录一直都处于这种所谓的中间状态
      
    阶段二：当该删除语句所在的事务提交之后，会有专门的线程后来真正的把记录删除掉。
    所谓真正的删除就是把该记录从正常记录链表中移除，并且加入到垃圾链表中，
    也叫做purge

update

    在不更新主键的情况下
    
    
        就地更新（in-place update）
        更新记录时，对于被更新的每个列来说，如果更新后的列和更新前的列占用的存储空间都一样大，
        
        先删除掉旧记录，再插入新记录
        在不更新主键的情况下，如果有任何一个被更新的列更新前和更新后占用的存储空间大小不一致，
        那么就需要先把这条旧的记录从聚簇索引页面中删除掉，然后再根据更新后列的值创建一条新的记录插入到页面中。
        这里如果新创建的记录占用的存储空间大小不超过旧记录占用的空间，那么可以直接重用被加入到垃圾链表中的旧记录所占用的存储空间，
        否则的话需要在页面中新申请一段空间以供新记录使用，
        如果本页面内已经没有可用的空间的话，那就需要进行页面分裂操作，然后再插入新记录
        （频繁更新的表 也可能是随机IO了）
    
    
    更新主键的情况
    
        在聚簇索引中，记录是按照主键值的大小连成了一个单向链表的
        如果我们更新了某条记录的主键值，意味着这条记录在聚簇索引中的位置将会发生改变
        将旧记录进行delete mark操作
        根据更新后各列的值创建一条新记录，并将其插入到聚簇索引中（需重新定位插入的位置）
        (相当于DELETE加INSERT)
       
    
    更新主键的情况
    其实就是删除插入
    将旧记录进行delete mark操作
    根据更新后各列的值创建一条新记录，并将其插入到聚簇索引中（需重新定位插入的位置）。
    
    
    数据量越大  修改越频繁  
        频繁的页修改 会在同一个页
        ？？相邻id的数据可能不相邻  可能不在同一个页  随机IO越大
    
# 日志

在数据修改的时候，不仅记录了redo，还记录了相对应的undo

    

