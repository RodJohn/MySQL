

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






# 1 开启/提交

## 开启
    
BEGIN

    显式的开启一个事务
    
START TRANSACTION(

    显式的开启一个事务
    在存储过程中只能使用这个,因为BEGIN关键词重复
    
    
## 提交

COMMIT

    提交本次事务
    并使已对数据库进行的所有修改成为永久性的；

COMMIT WORK

    completion_type = 0时(默认);
        COMMIT WORK 用于提交事务并释放(release)
    completion_type = 1时;
        COMMIT WORK 用于提交事务并开启下一个事务(chain)
    completion_type = 2时;
        COMMIT WORK 用于提交事务并断开服务器连接



# 2 自动提交

作用

    在默认情况下MySQL开启的是autocommit模式，
    也就是每执行完一条SQL语句后都会自动执行COMMIT。
    如果显示开启事务,则在该事务内autocommit会失效

参数

    参看
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


# 语法    
    
设置隔离级别
	
	
	set session transaction isolatin level repeatable read;
	set global transaction isolation level repeatable read;
	
查看隔离级别
 
	select @@tx_isolation;
	select @@global.tx_isolation;




# 长事务



长事务意味着系统里面会存在很老的事务视图。由于这些事务随时可能访问数据库里面的任何数据，所以这个事务提交之前，数据库里面它可能用到的回滚记录都必须保留，这就会导致大量占用存储空间。


除了对回滚段的影响，长事务还占用锁资源，也可能拖垮整个库
