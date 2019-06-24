
# 当前读

    更新语句时
    需要先读取指定的行记录然后修改
    这样的读取就是当前读  

# 原理
    
    当前读需要读取记录的最新的提交值，
    
    当前读对行记录添加的是排他行锁
    而且直到事务提交才会释放锁
    所以不会发生脏写、脏读和不可重复读
    同时为了避免幻读，
    innodb会对对记录添加间隙锁防止插入记录    


# 缺点
    
    加锁
    特别是 为了解决幻读而添加的间隙锁



# 语句

当前读语法

    select * from table where ? lock in share mode; 加共享锁
    select * from table where ? for update; 加排它锁

包含当前读

    update table set ? where ?;
    delete from table where ?;
    insert into table values (…) ??
   