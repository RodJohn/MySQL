# 索引分组

使用索引  获取的时候  排序

索引分组包括使用紧凑索引和松散索引

# 紧凑索引

原理

	利用索引排序和分组

示例

	环境

		ALTER TABLE `test`.`test_order` 
		ADD INDEX `idx_b_c`(`b`, `c`) USING BTREE;

	SQL(index用于排序)

		EXPLAIN SELECT b,c FROM test_order where a > 500 GROUP BY b,c

		id	select_type	table	type	key	key_len	ref	rows	Extra
		1	SIMPLE	test_order	index	idx_b_c	8		11152	Using where

	SQL(index用于搜索、排序、分组 索引覆盖)

		EXPLAIN SELECT b,c FROM test_order GROUP BY b,c
		
		id	select_type	table	type	key	key_len	ref	rows	Extra
		1	SIMPLE	test_order	index	idx_b_c	8		11152	Using index



# 松散索引


原理

	紧凑索引需要读取所有满足条件的索引键，然后再根据读取的数据来完成GROUP BY操作得到相应结果。
	这种方法只需要扫描索引中的少部分数据，而不是所有满足where条件的数据，所以这个方法叫做loose index scan
	(其实我还不是太懂)

示例

	环境

		同上
		注意
		松散索引扫描，此类查询的EXPLAIN输出显示Extra列的Using index for group-by
		在5.6版本下显示为Using index for group-by
		5.7版本显示为Using index 

	SQL

		EXPLAIN  SELECT b FROM test_order GROUP BY b
		
		id	select_type	table	type	key	key_len	ref	rows	Extra
		1	SIMPLE	test_order	range	idx_b_c	4		2231	Using index for group-by

		EXPLAIN  SELECT b,max(c) FROM test_order GROUP BY b
		
		id	select_type	table	type	key	key_len	ref	rows	Extra
		1	SIMPLE	test_order	range	idx_b_c	4		2231	Using index for group-by




