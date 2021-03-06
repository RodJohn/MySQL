# 索引

原理

	在嵌套循环中
	查找被驱动表的数据时使用索引  
	把对驱动表的全表扫描变为索引扫描
  
复杂度

	假设驱动表的行数是 N，被驱动表的行数是 M。
	执行过程就要扫描驱动表 N 行，
	查询被驱动表 N 次, 每次的复杂度近似复杂度是以 扇出数 为底的 M 的对数,扇出一般都很大
	时间复杂度 N + N * log(扇出数)M


# 示例

环境

	VERSION()
	5.7.26

数据

	同于blob示例


## 测试(选择有索引的作为被驱动表)

sql

	EXPLAIN	
	SELECT o.id,o.a,c.id,c.code
	FROM test_order o
	JOIN test_code c
	on o.a = c.id

	id	select_type	table	partitions	type	key		key_len	ref			rows	filtered	Extra
	1	SIMPLE		o					ALL									1100	100	
	1	SIMPLE		c					eq_ref	PRIMARY	4		test.o.a	1		100	

分析

	关联条件中code的id上有索引
	优化器选择code作为被驱动表
	这样就能使用索引加快关联

## 测试(同时有索引，选择小表被驱动表)

sql

	EXPLAIN	
	SELECT o.id,o.a,c.id,c.code
	FROM test_order o
	JOIN test_code c
	on o.id = c.id

	id	select_type	table	partitions	type	 key				key_len	ref			rows	filtered	Extra
	1	SIMPLE		c					ALL												9			100	
	1	SIMPLE		o					eq_ref PRIMARY				4		test.c.id	1			100	

分析

	关联条件中code的id和order的id都有索引
	优化器选择小表code作为被驱动表
	这样就能使用索引加快关联，也能减少循环次数


