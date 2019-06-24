
# 当前读

定义
    
    是针对修改提出的读取策略
    当前读就是读到记录的最新值
    ，读取的是记录的最新版本，并且，当前读返回的记录，都会加上锁，保证其他事务不会再并发修改这条记录。
    
# 语句

当前读

    select * from table where ? lock in share mode;
    select * from table where ? for update;

更新

    更新操作，包含当前读的部分
    更新数据都是先读后写的，而这个读，只能读当前的值，称为“当前读”（current read）。
    update之前要查询
    insert into table values (…);
    update table set ? where ?;
    delete from table where ?;


# 隔离性

    innodb使用两阶段提交的行锁、间隙锁来实行当前读的隔离级别        
 
     脏写 行锁锁住写不了
     脏读 行锁锁住写不了
     不可重复读  行锁锁住
     幻读  间隙锁锁住 插入不了
 
# 缺点
    
    加锁
    特别是 为了解决幻读而添加的间隙锁



   