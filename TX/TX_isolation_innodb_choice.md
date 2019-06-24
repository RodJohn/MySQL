
# innodb选择哪个隔离级别    


# 不选择读未提交

缺点

    采用读未提交(Read UnCommitted),
    一个事务读到另一个事务未提交读数据，
    从逻辑上都说不过去！

# 不选择串行化

缺点

    采用串行化(Serializable)，每个次读操作都会加锁，快照读失效，一般是使用mysql自带分布式事务功能时才使用该隔离级别！
    (笔者从未用过mysql自带的这个功能，因为这是XA事务，是强一致性事务，性能不佳！互联网的分布式方案，多采用最终一致性的事务解决方案！)


# 不选择可重复读

## 实现

    innodb的可重复读
    是由undo日志为数据提供的版本链和事务期间的一致性视图实现的
    同时使用了nextkey lock来解决幻读问题
    效果相当于串行化

## 缺点

使用了nextkey lock

    在RR隔离级别下，存在间隙锁，导致出现死锁的几率比RC大的多！
    
    在RR隔离级别下，锁定读（UPDATE和DELETE）在条件列未命中索引会锁表！而在RC隔离级别下，只锁行
# 语义困惑

    语义 混乱
    同一个事务内快照读和当前读引发的歧义

## 示例

环境、数据

    SELECT VERSION()
        VERSION()
        5.7.26

    select @@global.tx_isolation;
        @@global.tx_isolation
        REPEATABLE-READ
        
    DROP TABLE `test`.`account`;
    CREATE TABLE `test`.`account`  (
      `id` int(0) NOT NULL,
      `money` int(255) NULL,
      PRIMARY KEY (`id`)
    );
    
    INSERT INTO `test`.`account`(`id`, `money`) VALUES (1, 100);
    
操作

    SESSION 1

    START TRANSACTION;
    SELECT * from `test`.`account` where id = 1
        id	money
        1	100
    update `test`.`account` SET money = money + 1 where id = 1 ; 
    SELECT * from `test`.`account` where id = 1
        id	money
        1	102

问题

    明明是加一，最终结果却是加二    

可能原因
    
    SESSION 2 在select之后update之前进行了下面的操作
    update `test`.`account` SET money = money + 1 where id = 1 ; 
    
    select是快照读，update是当前读




# 选择关闭间隙锁的可重复读
  
# 选择提交读


解决
 
    分析的问题都是在可重复读隔离级别下的，间隙锁是在可重复读隔离级别下才会生效的。
    所以，你如果把隔离级别设置为读提交的话，就没有间隙锁了。
    但同时，你要解决可能出现的数据和日志不一致问题，需要把binlog格式设置为row。这，也是现在不少公司使用的配置组合。
    务不需要可重复读的保证，这样考虑到读提交下操作数据的锁范围更小（没有间隙锁），这个选择是合理的
    
    然后，在备份期间，备份线程用的是可重复读，而业务线程用的是读提交。同时存在两种事务隔离级别，会不会有问题
  
  
# 参考

    https://mp.weixin.qq.com/s/JcJGXSYkko5O_YAM37LBXA  