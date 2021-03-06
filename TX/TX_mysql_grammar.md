

# 启动、提交

## 显式

显式启动事务语句

    begin 或 start transaction。
    配套的提交语句是commit，
    回滚语句是rollback。


## 自动提交

作用
    
    自动提交
    也就是每执行完一条SQL语句后都会自动执行COMMIT。
    如果显示开启事务,则在该事务内autocommit会失效

参数

    参看
    在默认情况下MySQL开启的是autocommit模式，
        select @@autocommit;
            autocommit = 1 开启自动提交
    设置
        set @@autocommit = 0 ;
            之后需要手动COMMIT. 



示例

    ```
    set autocommit = 0; //挂起自动提交
    insert into t_cart values(10001, 10001, 1, 10001, 0);
    insert into t_cart values(10001, 10002, 1, 10001, 0);
    COMMIT; //提交事务
    set autocommit = 1; //恢复自动提交
    ```
    
    ```
    BEGIN; //开始事务，挂起自动提交
    insert into t_cart values(10001, 10001, 1, 10001, 0);
    insert into t_cart values(10001, 10002, 1, 10001, 0);
    COMMIT; //提交事务，恢复自动提交
    ```
    
# 3 ROLLBACK

## SAVEPOINT

SAVEPOINT identifier
    
    SAVEPOINT允许在事务中创建一个保存点，一个事务中可以有多个SAVEPOINT;
    
RELEASE SAVEPOINT identifier

    删除一个事务的保存点，当没有指定的保存点时，执行该语句会抛出一个异常；

## ROLLBACK

ROLLBACK

    结束用户的事务，
    并撤销该事务内的全部修改；    
        
ROLLBACK TO identifier；

    把事务回滚到标记点；    
    但是不结束事务


## 出错和回滚

原理

    innodb存储引擎中的事务都是原子性的，说明以下2种情况：
    构成事务的每条语句都会commit，否则事务的每条语句都会rollback，
    这种保护还会涉及到单调的语句。一条语句要不完成成功要么完全回滚，
    但是一条语句失败并不会导致前一条执行的语句自动回滚，他们的工作会保留，需要你手动commit或者rollback。

示例

    mysql> create table t(a int, primary key (a))engine=innodb;
    Query OK, 0 rows affected (0.01 sec)
    
    mysql> begin;
    Query OK, 0 rows affected (0.00 sec)
    
    mysql>  insert into t select 1;
    Query OK, 1 row affected (0.00 sec)
    Records: 1  Duplicates: 0  Warnings: 0
    
    mysql>  insert into t select 1;
    ERROR 1062 (23000): Duplicate entry '1' for key 'PRIMARY'

    mysql>  select * from t;
    +---+
    | a |
    +---+
    | 1 |
    +---+
    1 row in set (0.00 sec)
    
    插入第二条记录的时候，db抛出了1062错误，但是并没有自动回滚，能查出前一条insert的记录



# 链式



COMMIT WORK

    completion_type = 0时(默认);
        COMMIT WORK 用于提交事务并释放(release)
    completion_type = 1时;
        COMMIT WORK 用于提交事务并开启下一个事务(chain)
    completion_type = 2时;
        COMMIT WORK 用于提交事务并断开服务器连接



# 隔离级别   
    
设置隔离级别
	
	
	set session transaction isolatin level repeatable read;
	set global transaction isolation level repeatable read;
	
查看隔离级别
 
	select @@tx_isolation;
	select @@global.tx_isolation;



