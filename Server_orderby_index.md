
# 索引排序

## 原理


## 要求

	在排序之前，数据就是按照排序列顺序读取的，而且未发生改变

	如果有where where 中使用的相同索引命中
	如果有join 排序列必须在驱动表
	如果是二级索引 则要考虑回表代价 

	主键索引起效


# 主键排序


	EXPLAIN SELECT * FROM test_order  ORDER BY id 

	EXPLAIN SELECT * FROM test_order where a > 100  ORDER BY id 


# 二级索引


未添加索引

	EXPLAIN SELECT * FROM test_order ORDER BY a 
	显示使用filesort

添加二级索引

	ALTER TABLE `test`.`test_order` 
	ADD INDEX `idx_a`(`a`) USING BTREE;

	EXPLAIN SELECT * FROM test_order ORDER BY a 
	显示使用filesort

	回表成本太大

索引覆盖

	EXPLAIN SELECT a,b FROM test_order ORDER BY a
	
限制回表

	EXPLAIN SELECT * FROM test_order where a > 900 ORDER BY a 
	Using index condition  idx_a 

使用其他条件

	EXPLAIN SELECT * FROM test_order where b  > 300  ORDER BY a 
	显示使用filesort

使用其他条件（联合索引)

limit 

	EXPLAIN SELECT * FROM test_order  ORDER BY a limit 10
	Using index condition  idx_a 


# 联表


	DROP TABLE IF EXISTS test.test_code;
	CREATE TABLE test.test_code(
	id int(10) not null auto_increment,
	code int(10) not null,
	PRIMARY key (`id`)
	)ENGINE INNODB DEFAULT CHARSET utf8 ;


	EXPLAIN SELECT * FROM test_order o JOIN test_code c on o.id = c.id  ORDER BY o.id 


	| id | select_type | table | partitions | type   | key     | key_len | ref       | rows | filtered | Extra           
	|  1 | SIMPLE      | c     | NULL       | ALL    | NULL    | NULL    | NULL      |    4 |   100.00 | Using temporary; Using filesort 
	|  1 | SIMPLE      | o     | NULL       | eq_ref | PRIMARY | 4       | test.c.id |    1 |   100.00 | NULL           



	执行计划中出现了“Using temporary”，正是因为我们的排序操作需要在两个表Join之后才能进行


# 参考

https://blog.csdn.net/u011215669/article/details/82078812




