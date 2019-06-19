
# Block Nested-Loop Join

原理

	驱动表数据批量匹配被驱动表
	减少被驱动表的循环次数

流程

	1. 把驱动表的数据读入线程内存 join_buffer 中
	（如果放不下，那就分段放）
	2. 遍历被驱动表的每一行，跟 join_buffer 中的数据做对比
	3. 满足匹配条 件的，作为结果集的一部分返回


复杂度

	假设驱动表的行数是 N，被驱动表的行数是 M。
	分成K段放入内存中
	执行过程就要扫描驱动表 N 行，扫描被驱动表 K * M 行
	在内存中的判断次数是 M*N 次 内存操作，速度上会快很多
	时间复杂度 N + K * M = N + N/S * M 

缺点

	BNL 算法在大表 join 的时候性能就差多了，比较次数等于两个表参与 join 的行数 的乘积	
	也就是说，BNL 算法对系统的影响主要包括三个方面：
	1. 可能会多次扫描被驱动表，占用磁盘 IO 资源； 
	2. 判断 join 条件需要执行 M*N 次对比（M、N 分别是两张表的行数），如果是大表就会 占用非常多的 CPU 资源； 
	3. 可能会导致 Buffer Pool 的热数据被淘汰，影响内存命中率


设置

	设置
	set optimizer_switch = 'block_nested_loop=off'
	查看
	show variables like '%optimizer_switch%' 
	
	join_buffer 的大小是由参数 join_buffer_size 设定的，默认值是 256k。
	join_buffer_size 越大，一次可以放入 的行越多，分成的段数也就越少，对被驱动表的全表扫描次数就越少。


# 测试

环境

	VERSION()
	5.7.26
	
	show variables like '%optimizer_switch%' 
	block_nested_loop=on

数据

	DROP TABLE IF EXISTS test.test_order;
	CREATE TABLE test.test_order(
	id int(10) not null auto_increment,
	a int(10) not null,
	b int(10) not null,
	c int(10) not null,
	PRIMARY key (`id`)
	)ENGINE INNODB DEFAULT CHARSET utf8 COMMENT '测试表';
	
	DROP TABLE IF EXISTS test.test_code;
	CREATE TABLE test.test_code(
	id int(10) not null auto_increment,
	code int(10) not null,
	PRIMARY key (`id`)
	)ENGINE INNODB DEFAULT CHARSET utf8 ;

	
	DROP PROCEDURE IF EXISTS insert_test_order;
	CREATE PROCEDURE insert_test_order(in num_limit int,in rand_limit int)
	BEGIN

	DECLARE i int default 1;
	DECLARE a int default 1;
	DECLARE b int default 1;
	DECLARE c int default 1;

	WHILE i<=num_limit do

	set a = FLOOR(rand()*rand_limit);
	set b = FLOOR(rand()*rand_limit);
	set c = FLOOR(rand()*rand_limit);
	INSERT into test.test_order values (null,a,b,c);
	set i = i + 1;

	END WHILE;

	END

	
	DROP PROCEDURE IF EXISTS insert_test_code;
	CREATE PROCEDURE insert_test_code(in num_limit int,in rand_limit int)
	BEGIN

	DECLARE i int default 1;
	DECLARE a int default 1;

	WHILE i<=num_limit do

	set a = FLOOR(rand()*rand_limit);
	INSERT into test.test_code values (null,a);
	set i = i + 1;

	END WHILE;

	END

	call insert_test_order(1100,1000);
	call insert_test_code(10,100);


SQL 

	EXPLAIN	
	SELECT o.id,o.a,c.id,c.code
	FROM test_code c
	JOIN test_order o
	on o.a = c.code

	id	select_type	table	type	key	key_len	ref	rows	filtered	Extra
	1	SIMPLE		c		ALL						9		100	
	1	SIMPLE		o		ALL						1100	10	Using where; Using join buffer (Block Nested Loop)

分析

	使用joinbuffer减少被驱动表的循环次数











	
