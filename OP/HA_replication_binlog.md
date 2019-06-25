
# binlog

binlog的特性确保了在备库执行相同的binlog，可以得到与主库相同的状态。

# 格式


binlog_format有三种可选配置：STATEMENT、ROW、MIXED，相应地，
基于这三种模式的Replication分别称为SBR(STATEMENT BASED Replication)、RBR、MBR。
 同时，我们也知道，MySQL Replication可以支持比较灵活的binlog规则，可以设置某些库、某些表记录或者忽略不记录。

binlog有两种格式，一种是statement，一种是row。
可能你在其他资料上还会看到有第三种格式，
叫作mixed，其实它就是前两种格式的混合。


# statement


由于statement格式下，记录到binlog里的是语句原文
，因此可能会出现这样一种情况：在主库执行这条SQL语句的时候，用的是索引a；
而在备库执行这条SQL语句的时候，却使用了索引t_modified。因此，MySQL认为这样写是有风险的

# ROW

mysqlbinlog工具，用下面这个命令解析和查看binlog中的内容。

当binlog_format使用row格式的时候，binlog里面记录了真实删除行的主键id，这样binlog传到备库去的时候，就肯定会删除id=4的行，不会有主备删除不同行的问题


现在越来越多的场景要求把MySQL的binlog格式设置成row。这么做的理由有很多，我来给你举一个可以直接看出来的好处：恢复数据。



# MIXED

    
    因为有些statement格式的binlog可能会导致主备不一致，所以要使用row格式。
    但row格式的缺点是，很占空间。
    所以，MySQL就取了个折中方案，也就是有了mixed格式的binlog。
    mixed格式的意思是，MySQL自己会判断这条SQL语句是否可能引起主备不一致，
    如果有可能，就用row格式，否则就用statement格式。
    也就是说，mixed格式可以利用statment格式的优点，同时又避免了数据不一致的风险。
    
    因此，如果你的线上MySQL设置的binlog格式是statement的话，那基本上就可以认为这是一个不合理的设置。你至少应该把binlog的格式设置为mixed。




通常地，我们强烈建议不要设置这些规则，默认都记录就好，在Slave上也是如此，默认所有库都进行Replicate，不要设置DO、IGNORE、REWRITE规则。







