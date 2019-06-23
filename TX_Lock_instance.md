全局锁就是对整个数据库实例加锁。

当你需要让整个库处于只读状态的时候，

MySQL提供了一个加全局读锁的方法，命令是 Flush tables with read lock (FTWRL)。





全局锁的典型使用场景是，做全库逻辑备份

以前有一种做法，是通过FTWRL确保不会有其他线程对数据库做更新，然后对整个库做备份。



官方自带的逻辑备份工具是mysqldump

当mysqldump使用参数–single-transaction的时候，导数据之前就会启动一个事务，来确保拿到一致性视图。

所以，single-transaction方法只适用于所有的表使用事务引擎的库。如果有的表使用了不支持事务的引擎，那么备份就只能通过FTWRL方法。

这往往是DBA要求业务开发人员使用InnoDB替代MyISAM的原因之一。



