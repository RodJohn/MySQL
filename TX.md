


使用 读提交隔离级别


长事务

除了对回滚段的影响，长事务还占用锁资源



# 语句

显式启动事务语句，
 begin 或 start transaction。配套的提交语句是commit，回滚语句是rollback。
 
 
set autocommit=0，这个命令会将这个线程的自动提交关掉。
意味着如果你只执行一个select语句，这个事务就启动了，而且并不会自动提交。
这个事务持续存在直到你主动执行commit 或 rollback 语句，或者断开连接。

MySQL默认的数据提交操作模式是自动提交模式（autocommit）。这就表示除非显式地开始一个事务，否则每个查询都被当做一个单独的事务自动执行。

begin/start transaction 命令并不是一个事务的起点，在执行到它们之后的第一个操作InnoDB表的语句，事务才真正启动。如果你想要马上启动一个事务，


# transaction id


InnoDB里面每个事务有一个唯一的事务ID，叫作transaction id。
它是在事务开始的时候向InnoDB的事务系统申请的，是按申请顺序严格递增的。



 