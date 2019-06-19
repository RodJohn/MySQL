# 子查询转连接

分析
	将IN子查询转换为连接查询时
	子查询的结果可能会重复，而导致连接结果重复
	所以IN子查询和两表连接之间并不完全等价。
	
示例

	子查询
	SELECT * FROM s1 WHERE key1 IN (SELECT common_field FROM s2 WHERE key3 = 'a');
	连接查询
	SELECT s1.* FROM s1 INNER JOIN s2 
		ON s1.key1 = s2.common_field WHERE s2.key3 = 'a';
	分析


# 半连接

定义

	半连接（semi-join）的意思就是
	对于驱动表的某条记录来说，
	只判断在驱动表中是否存在与之匹配的记录是否存在，而不关心具体有多少条记录与之匹配，
	最终的结果集中只保留驱动表的记录

	将子查询转换为连接可以充分发挥优化器的作用
	

策略

	IN子查询符合转换为semi-join的5种策略
	Table pullout
	DuplicateWeedout
	LooseScan
	Materialization
	FirstMatch

条件



# Table pullout 

## 原理

原理

	子查询中的表上拉
	转为连接查询

说明

	当子查询的查询列表处只有主键或者唯一索引列时，
	可以直接把子查询中的表上拉到外层转化为子查询
	因为主键或者唯一索引列中的数据本身就是不重复，不会匹配出重复数据


## 示例

环境

	VERSION()
	5.7.26

数据

SQL

	EXPLAIN EXTENDED
	SELECT * 
	FROM test_order
	WHERE a in 
	(SELECT id FROM test_code );

	+----+-------------+------------+--------+---------+---------+-------------------+------+----------+-------------+
	| id | select_type | table      | type   | key     | key_len | ref               | rows | filtered | Extra       |
	+----+-------------+------------+--------+---------+---------+-------------------+------+----------+-------------+
	|  1 | SIMPLE      | test_order | ALL    | NULL    | NULL    | NULL              |  100 |   100.00 | NULL        |
	|  1 | SIMPLE      | test_code  | eq_ref | PRIMARY | 4       | test.test_order.a |    1 |   100.00 | Using index |
	+----+-------------+------------+--------+---------+---------+-------------------+------+----------+-------------+

	2 rows in set, 2 warnings (0.00 sec)

	mysql> show warnings;
	+---------+------+----------------------------------------------------------------------------------------------------------------------------------
	| Level   | Code | Message                                                                                                                          
	+---------+------+----------------------------------------------------------------------------------------------------------------------------------
	| Warning | 1681 | 'EXTENDED' is deprecated and will be removed in a future release.                                                                
	| Note    | 1003 | 
	/* select#1 */ select `test`.`test_order`.`id` AS `id`,`test`.`test_order`.`a` AS `a`,`test`.`test_order`.`b` AS `b`,`test`.`test_order`.`c` AS `c` 
	from `test`.`test_code` join `test`.`test_order` where (`test`.`test_code`.`id` = `test`.`test_order`.`a`) |
	+---------+------+----------------------------------------------------------------------------------------------------------------------------------
	2 rows in set (0.00 sec)

分析

	子查询被转化为了连接查询

# Materialization

## 原理

原理

	将子查询的结果保存到临时表（Materialize）
	在临时表中建立唯一索引
	建立驱动表和临时表的连接查询


特点

## 示例

环境

	VERSION()
	5.7.26

数据


SQL

	EXPLAIN EXTENDED
	SELECT * 
	FROM test_order
	WHERE a in 
	(SELECT code FROM test_code )

	+----+--------------+-------------+--------+------------+---------+-------------------+------+----------+-------------+
	| id | select_type  | table       | type   | key        | key_len | ref               | rows | filtered | Extra       |
	+----+--------------+-------------+--------+------------+---------+-------------------+------+----------+-------------+
	|  1 | SIMPLE       | test_order  | ALL    | NULL       | NULL    | NULL              |  100 |   100.00 | Using where |
	|  1 | SIMPLE       | <subquery2> | eq_ref | <auto_key> | 4       | test.test_order.a |    1 |   100.00 | NULL        |
	|  2 | MATERIALIZED | test_code   | ALL    | NULL       | NULL    | NULL              |  200 |   100.00 | NULL        |
	+----+--------------+-------------+--------+------------+---------+-------------------+------+----------+-------------+
	3 rows in set, 2 warnings (0.00 sec)

	mysql> show warnings;
	+---------+------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
	| Level   | Code | Message                                                                                                                                                                      
	+---------+------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
	| Warning | 1681 | 'EXTENDED' is deprecated and will be removed in a future release.                                                                                                            
	| Note    | 1003 | /* select#1 */ select `test`.`test_order`.`id` AS `id`,`test`.`test_order`.`a` AS `a`,`test`.`test_order`.`b` AS `b`,`test`.`test_order`.`c` AS `c` from `test`.`test_order`
						semi join (`test`.`test_code`) where (`<subquery2>`.`code` = `test`.`test_order`.`a`) 
	+---------+------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
	2 rows in set (0.00 sec)

分析

	将子查询结果生成了临时表，
	并建立了唯一索引
	在主表和临时表之间建立了半连接



