
# 不选择读未提交级别

    采用读未提交(Read UnCommitted),
    一个事务读到另一个事务未提交读数据，
    语义不对
    
# 不选择串行化级别

    采用串行化(Serializable)，
    每次读写操作都加排他锁，效率太低


# 不选择可重复读级别

## 使用间隙锁

    innodb的可重复读级别使用了间隙锁解决幻读问题

    当前读在条件列未命中索引会锁表！而在RC隔离级别下，只锁行
    
    当前读在，存在间隙锁，导致出现死锁的几率比RC大的多！
    比如
    
## 需要保存更长的undo日志

    
    
## 语义歧义

    当前读读取到的永远都是已提交的最新值
    快照读会根据视图的区别选择记录版本
    在可重复读级别下可能产生语义的混乱

### 示例

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


## 不选择关闭间隙锁的可重复读   

 
  
# 选择提交读级别


优点

    没有间隙锁
    加快undolog清理
    快照读和当前读一致的语义

问题
 
    解决备份问题
    

  
# 参考

    https://mp.weixin.qq.com/s/JcJGXSYkko5O_YAM37LBXA  