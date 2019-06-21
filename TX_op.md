

# MySQL的事务启动方式有以下几种：


显式启动事务语句， begin 或 start transaction。配套的提交语句是commit，回滚语句是rollback。

set autocommit=0，这个命令会将这个线程的自动提交关掉。意味着如果你只执行一个select语句，这个事务就启动了，而且并不会自动提交。

这个事务持续存在直到你主动执行commit 或 rollback 语句，或者断开连接。
特别是 spring的时候 controlle层 使用 Transcation

有些客户端连接框架会默认连接成功后先执行一个set autocommit=0的命令。这就导致接下来的查询都在事务中，如果是长连接，就导致了意外的长事务。

或者 read committed

但是有的开发同学会纠结“多一次交互”的问题。对于一个需要频繁使用事务的业务，第二种方式每个事务在开始时都不需要主动执行一次 “begin”，减少了语句的交互次数。如果你也有这个顾虑，我建议你使用commit work and chain语法。

# 

你可以在information_schema库的innodb_trx这个表中查询长事务，比如下面这个语句，用于查找持续时间超过60s的事务。select * from information_schema.innodb_trx where TIME_TO_SEC(timediff(now(),trx_started))>60




# 长事务



长事务意味着系统里面会存在很老的事务视图。由于这些事务随时可能访问数据库里面的任何数据，所以这个事务提交之前，数据库里面它可能用到的回滚记录都必须保留，这就会导致大量占用存储空间。


除了对回滚段的影响，长事务还占用锁资源，也可能拖垮整个库
