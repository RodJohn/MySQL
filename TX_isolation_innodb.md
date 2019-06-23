

# MySQL

    MySQL中支持事务的存储引擎有innoDB和NDB。
    
# innoDB
 
 
## 实现

    “读未提交”隔离级别下直接返回记录上的最新值，没有视图概念；
    “串行化”隔离级别下直接用加锁的方式来避免并行访问。
    版本链和读取视图
    在实现上，数据库里面会创建一个视图，访问的时候以视图的逻辑结果为准。
    
    在“可重复读”隔离级别下，这个视图是在事务启动时创建的，整个事务存在期间都用这个视图。
    在“读提交”隔离级别下，这个视图是在每个SQL语句开始执行的时候创建的。
    
## 可重复读

    innoDB默认的隔离级别是RR，
    并且在RR的隔离级别下更进一步，通过多版本并发控制解决了脏读和不可重复读问题，加上间隙锁解决幻读问题。
    因此innoDB的RR隔离级别其实实现了串行化级别的效果，而且保留了比较好的并发性能。



# 语法    
    
设置隔离级别
	
	
	set session transaction isolatin level repeatable read;
	set global transaction isolation level repeatable read;
	
查看隔离级别
 
	select @@tx_isolation;
	select @@global.tx_isolation;




