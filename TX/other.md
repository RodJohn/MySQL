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



 WAL
 
 的全称是Write-Ahead Logging，它的关键点就是先写日志，再写磁盘
 



# 事务id


给事务分配id的时机

，只有第一次对某个用户创建的临时表执行增、删、改操作时才会为这个事务分配一个事务id


事务id是怎么生成的
这个事务id本质上就是一个数字，它的分配策略和我们前边提到的对隐藏列row_id（当用户没有为表创建主键和UNIQUE键时InnoDB自动创建的列）的分配策略大抵相同

行记录trxId
InnoDB记录行格式的时候重点强调过：聚簇索引的记录除了会保存完整的用户数据以外，而且还会自动添加名为trx_id、roll_pointer的隐藏列

