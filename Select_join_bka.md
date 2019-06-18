
# Batched Key Access

原理

BKA相当于块循环算法和索引

MRR算法可以将随机的二级索引回表转为顺序回表
在


流程


# 示例

环境

	set optimizer_switch='mrr=on,mrr_cost_based=off,batched_key_access=on';
	前两个参数的作用是要启用MRR。这么做的原因是，BKA算法的优化要依赖于MRR。


数据

	同于blob的结构和数据

	ALTER TABLE `test`.`test_code` 
	ADD COLUMN `msg` varchar(50) NULL AFTER `code`;

	ALTER TABLE `test_code`
	ADD INDEX `idx_code`(`code`) USING BTREE;

SQL

	EXPLAIN	
	SELECT o.id,o.a,o.b,c.id,c.code,c.msg
	FROM test_order o
	JOIN test_code c
	on o.a = c.code

	id	select_type	table	type	key	key_len	ref	rows	filtered	Extra
	1	SIMPLE		o		ALL						1100	100	
	1	SIMPLE		c		ref	idx_code	4	test.o.a	1	100		Using join buffer (Batched Key Access)

分析

	code是二级索引
	要返回的msg字段需要回表
	使用BKR将随机IO转为顺序IO
	显示USING BKA
	



