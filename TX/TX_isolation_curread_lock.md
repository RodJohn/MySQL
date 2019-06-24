

# 规则

    原则1：加锁的基本单位是next-key lock。
    
    
    最坏
    
    全表扫描
    非唯一索引


# 环境


环境

    SELECT VERSION()
        VERSION()
        5.7.26

    select @@global.tx_isolation;
        @@global.tx_isolation
        REPEATABLE-READ

数据

    DROP TABLE `test`.`curread`
    CREATE TABLE `test`.`curread`  (
      `id` int(0),
      `ui` int(0) NULL,
      `ni` int(255) NULL,
      `n` int(255) NULL,
      PRIMARY KEY (`id`),
      UNIQUE INDEX `idx_ui`(`ui`) USING BTREE,
      INDEX `idx_ni`(`ni`) USING BTREE
    );
    
    INSERT INTO `test`.`curread`(`id`, `ui`, `ni`, `n`) VALUES (0, 0, 0, 0);
    INSERT INTO `test`.`curread`(`id`, `ui`, `ni`, `n`) VALUES (5, 5, 5, 5);
    INSERT INTO `test`.`curread`(`id`, `ui`, `ni`, `n`) VALUES (10, 10, 10, 10);
    INSERT INTO `test`.`curread`(`id`, `ui`, `ni`, `n`) VALUES (15, 15, 15, 15);

查询被阻塞语句

    方法一
        show engine innodb status
        查看TRANSACATION部分
    
    方法二
        使用SQL查看
        SELECT b.trx_mysql_thread_id             AS 'blocked_thread_id' 
              ,b.trx_query                      AS 'blocked_sql_text' 
              ,c.trx_mysql_thread_id             AS 'blocker_thread_id'
              ,c.trx_query                       AS 'blocker_sql_text'
              ,( Unix_timestamp() - Unix_timestamp(c.trx_started) ) 
                                      AS 'blocked_time' 
        FROM   information_schema.innodb_lock_waits a 
            INNER JOIN information_schema.innodb_trx b 
                 ON a.requesting_trx_id = b.trx_id 
            INNER JOIN information_schema.innodb_trx c 
                 ON a.blocking_trx_id = c.trx_id 
        WHERE  ( Unix_timestamp() - Unix_timestamp(c.trx_started) ) > 4; 
        


# 全表扫描

## 特点

    RR级别下，
    全表扫描会给全部数据加上next-key锁，直到事务提交
    如果加的是排它锁，无法插入和修改，就相当于全表被锁死

## 原理

    全表扫描时，存储引擎每读取一条聚簇索引记录，
    就会为这条记录加锁一个next-key锁，然后返回给server层，
    如果server层判断条件成立则将其发送给客户端，否则会向InnoDB存储引擎发送释放掉该记录上的锁的消息，
    
    在REPEATABLE READ隔离级别下，
    InnoDB存储引擎并不会真正的释放掉锁，所以聚簇索引的全部记录都会被加锁，并且在事务提交前不释放
    
## 示例

SESSION 1

    START TRANSACTION;
    delete from `test`.`curread` where n = 3 ; 

SESSION 2

    START TRANSACTION;
    SELECT * from `test`.`curread` where n = 15 for update ;     
    此时语句会被阻塞，长时间没有响应
    查看命令show engine innodb status
        SELECT * from `test`.`curread` where n = 15 for update
        TRX HAS BEEN WAITING 41 SEC FOR THIS LOCK TO BE GRANTED:

    



# 主键等值查询

## 特点
    
RC   

    加行锁

RR

    指定行存在 
        为了防止数据被修改，对指定行加行锁
        由于主键索引存在，这个值不会被插入数据，不需要加间隙锁  
    指定行不存在 
        为了防止幻读，加间隙锁 

## 示例    
   
SESSION 1

    START TRANSACTION;
    delete from `test`.`curread` where id = 3 ; 

SESSION 2   

    START TRANSACTION;
    INSERT INTO `test`.`curread`(`id`, `ui`, `ni`, `n`) VALUES (4, 4, 4, 4);
        INSERT会被阻塞
    SELECT * from `test`.`curread` where id = 4 for update ; 
        SELECT不会被阻塞

   
# 普通索引等值查询

## 原理

RR

    指定行存在 
        为了防止数据被修改，对指定行加行锁
        为了防止幻读，加间隙锁，此时的间隙是值的两侧 
    指定行不存在 
        为了防止幻读，加间隙锁 
    同时还要考虑对应主键索引的
    
## 示例

SESSION 1

    START TRANSACTION;
    delete from `test`.`curread` where ni = 5 ; 

SESSION 2   

    START TRANSACTION;
    INSERT INTO `test`.`curread`(`id`, `ui`, `ni`, `n`) VALUES (3, 3, 3, 3);
        INSERT会被阻塞
    INSERT INTO `test`.`curread`(`id`, `ui`, `ni`, `n`) VALUES (8, 8, 8, 8);
        INSERT会被阻塞


     
    
#  主键范围查询
#  普通索引范围查询


# 参考

    https://mp.weixin.qq.com/s/wSlNZcQkax-2KZCNEHOYLA
    https://mp.weixin.qq.com/s/ODbju9fjB5QFEN8IIYp__A
    https://mp.weixin.qq.com/s/9WWBXLNoUcTkS4DJnM5ViA
    
    http://hedengcheng.com/?p=771#_Toc374698307
    
    MySQL实战45讲

                 