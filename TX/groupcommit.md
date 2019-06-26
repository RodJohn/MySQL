
但是一次刷一个事务的日志性能较低，特别是事务集中在某一时刻时事务量非常大的时候。innodb提供了group commit功能，可以将多个事务的事务日志通过一次fsync()刷到磁盘中。

因为事务在提交的时候不仅会记录事务日志，还会记录二进制日志，但是它们谁先记录呢？二进制日志是MySQL的上层日志，先于存储引擎的事务日志被写入。



在MySQL5.6以前，
当事务提交(即发出commit指令)后，MySQL接收到该信号进入commit prepare阶段；
进入prepare阶段后，立即写内存中的二进制日志，写完内存中的二进制日志后就相当于确定了commit操作；
然后开始写内存中的事务日志；最后将二进制日志和事务日志刷盘，它们如何刷盘，
分别由变量 sync_binlog 和 innodb_flush_log_at_trx_commit 控制。

但因为要保证二进制日志和事务日志的一致性，在提交后的prepare阶段会启用一个prepare_commit_mutex锁来保证它们的顺序性和一致性。
但这样会导致开启二进制日志后group commmit失效，特别是在主从复制结构中，几乎都会开启二进制日志。


在MySQL5.6中进行了改进。
提交事务时，在存储引擎层的上一层结构中会将事务按序放入一个队列，队列中的第一个事务称为leader，
其他事务称为follower，leader控制着follower的行为。虽然顺序还是一样先刷二进制，
再刷事务日志，但是机制完全改变了：删除了原来的prepare_commit_mutex行为，也能保证即使开启了二进制日志，group commit也是有效的。


https://www.cnblogs.com/f-ck-need-u/p/9010872.html#auto_id_16
