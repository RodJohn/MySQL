

# 用法

作用

注意


# 原理

原理

	获取数据放入临时表，
	分组，
	从每组中只取出一条记录
	和GROUP BY的操作非常相似

类型

	执行方法和groupby类似 分为使用临时表，使用索引（松散索引，紧凑索引）

# 示例

	环境
		
		SELECT VERSION()
		5.6.17
		
		 索引bc
		

	临时表

		EXPLAIN SELECT distinct(a) FROM test_order 

		id	select_type	table	type	key	key_len	ref	rows	Extra
		1	SIMPLE	test_order	ALL			11152	Using temporary


	紧凑索引

		EXPLAIN SELECT DISTINCT b,c FROM test_order

		id	select_type	table	type	possible_keys	key	key_len	ref	rows	Extra
		1	SIMPLE	test_order	index	idx_b_c	idx_b_c	8		11152	Using index

	松散索引

		EXPLAIN  SELECT DISTINCT b FROM test_order GROUP BY b

		id	select_type	table	type	key	key_len	ref	rows	Extra
		1	SIMPLE	test_order	range	idx_b_c	4		2231	Using index for group-by

		松散索引显示的是Using index for group-by


# 参考

https://www.cnblogs.com/ggjucheng/archive/2012/11/18/2776449.html
