
 # 刷新过程
 
    innodb把redo log 从 redolog buffer写日志到磁盘时
    实际上是把redolog提交给操作系统的buffer
    操作系统具体什么时候写入磁盘是不知道的
    只有调用fsync() 才能强制将OS buffer中的日志刷到磁盘上的log file
    
 # 参数
 
    redolog刷新到redolog文件中的
    由innodb_flush_log_at_trx_commit确定
    
innodb_flush_log_at_trx_commit = 1，

    是Innodb 的默认设置。
    每次事务的提交都会写入redolog并调用fsync()强制将日志写到磁盘
    这个设置是最安全的设置，
    能够保证不论是MySQL Crash 还是OS Crash 或者是主机断电都不会丢失任何已经提交的数据。

innodb_flush_log_at_trx_commit = 2，

    每次事务提交的时候会写入redolog，但是不调用fsync()
    所以，当设置为2 的时候，MySQL Crash 并不会造成数据的丢失，
    但是OS Crash 或者是主机断电后可能丢失的数据量就完全控制在文件系统上了

innodb_flush_log_at_trx_commit = 0，

    每次事务的结束（commit 或者是rollback）并不会触发将log buffer 中的数据写入文件。
    /**TODO 刷新**/
    所以，当设置为0 的时候，当MySQL Crash 和OS Crash 或者主机断电之后，最极端的情况是丢失1 秒时间的数据变更。
 
    /***TODO 这个参数 是redo undo 都用吗 **/ 
    
# 测试

环境

    机械硬盘
    
    VERSION()
    5.6.17
 
数据
        
    drop table if exists test_flush_log;
    create table test_flush_log(id int,name char(50))engine=innodb;
    
    drop procedure if exists proc_flush_log;
    delimiter $$
    create procedure proc_flush_log(i int)
    begin
        declare s int default 1;
        declare c char(50) default repeat('a',50);
        while s<=i do
            start transaction;
            insert into test_flush_log values(null,c);
            commit;
            set s=s+1;
        end while;
    end$$
    delimiter ;

测试

    set @@global.innodb_flush_log_at_trx_commit=1; 
    call proc_flush_log(1000)
    > OK
    > 时间: 0.339s
    set @@global.innodb_flush_log_at_trx_commit=2; 
    truncate test_flush_log;
    call proc_flush_log(1000)
    > OK
    > 时间: 0.236s
    set @@global.innodb_flush_log_at_trx_commit=0; 
    truncate test_flush_log;
    call proc_flush_log(1000)
    > OK
    > 时间: 0.24s

    set @@global.innodb_flush_log_at_trx_commit=1; 
    call proc_flush_log(10000)
    > OK
    > 时间: 2.981s
    set @@global.innodb_flush_log_at_trx_commit=2; 
    truncate test_flush_log;
    call proc_flush_log(10000)
    > OK
    > 时间: 1.904s
    set @@global.innodb_flush_log_at_trx_commit=0; 
    truncate test_flush_log;
    call proc_flush_log(10000)
    > OK
    > 时间: 1.81s
    
结论

    1：普通硬盘情况下，1和2的情况，差异几乎到了1倍。
    3：硬盘的IO性能对性能的影响是巨大的。

# 选择

    刷新机制很重要
    直接关系着
    事务的并发受到IO的影响
    和持久性
    但这个时候事务的
    使用默认选项
    刷新策略直接关系到TPS
    可以有丢失选择2，例如im系统
    严格的默认选择1，转账
     
