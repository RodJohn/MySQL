

# 示例

## 环境

环境

    SELECT VERSION()
        VERSION()
        5.7.26

    select @@global.tx_isolation;
        @@global.tx_isolation
        REPEATABLE-READ

数据

    DROP TABLE `test`.`mvcc`;
    CREATE TABLE `test`.`mvcc`  (
      `id` int(0) NOT NULL,
      `age` int(0) NULL,
      PRIMARY KEY (`id`)
    );

    INSERT INTO `test`.`mvcc`(`id`, `age`) VALUES (1, 11);
    INSERT INTO `test`.`mvcc`(`id`, `age`) VALUES (2, 12);

## 流程


SESSEION 1

    START TRANSACTION;
    SELECT * FROM `test`.`mvcc`
        id	age
        1	11
        2	12

SESSEION 2

	START TRANSACTION;
	SELECT * FROM `test`.`mvcc`
        id	age
        1	11
        2	12
	UPDATE `test`.`mvcc` SET `age` = 110 WHERE `id` = 1;
	SELECT * FROM `test`.`mvcc`
        id	age
        1	110
        2	12

SESSEION 1

    SELECT * FROM mvcc
        id	age
        1	11
        2	12
        分析:
            session2的修改未提交，
            mvcc快照读不会发生脏读

SESSEION 2

	COMMIT;
	
	
SESSEION 1

    SELECT * FROM `test`.`mvcc`
        id	age
        1	11
        2	12
        分析:
            session2的修改提交晚于session1的ReadView建立
            mvcc快照读不会发生不可重复读


SESSEION 3

    START TRANSACTION;
    SELECT * FROM `test`.`mvcc`
        id	age
        1	110
        2	12
         分析:
             session2的修改提交早于session3的ReadView建立
                                
    INSERT INTO `test`.`mvcc`(`id`, `age`) VALUES (3, 13);
    SELECT * FROM `test`.`mvcc`
        id	age
        1	110
        2	12
        3	13
    COMMIT;        
        
SESSEION 1
    
    SELECT * FROM `test`.`mvcc`
        id	age
        1	11
        2	12
        分析:
            session3的修改提交晚于session1的ReadView建立
            mvcc快照读不会发生幻读
    COMMIT;

SESSEION 4

    SELECT * FROM `test`.`mvcc`
        id	age
        1	110
        2	12
        3	13


## 结论

    对于普通的select语句
    innodb采用mvcc实现的快照读方式
    可以实现完全的隔离级别
    也是完全无锁的读取