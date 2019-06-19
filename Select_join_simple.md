
# Simple Nested-Loop Join

原理

	使用嵌套循环来匹配数据

流程

1. 从驱动表中读入一行数据，并获取关联值
2. 遍历被驱动表，获得全部满足条件的被驱动行
3. 获取驱动行和被驱动行的笛卡尔积，作为结果集的一部分；
4. 重复执行步骤 1 到 3，直到驱动表的末尾循环结束。

示例

select * from t1 straight_join t2 on t1.a= t2.a 

1. 从表 t1 中读入一行数据 R； 
2. 从数据行 R 中，取出 a 字段到表 t2 里去查找； 
3. 取出表 t2 中满足条件的行，跟 R 组成一行，作为结果集的一部分； 
4. 重复执行步骤 1 到 3，直到表 t1 的末尾循环结束。

复杂度

假设驱动表的行数是 N，被驱动表的行数是 M。
执行过程就要扫描驱动表 N 行，扫描被驱动表 N * M 行
时间复杂度 N + N * M  


怎么选择驱动表
如果直接使用 join 语句，MySQL 优化器可能会选择表 t1 或 t2 作为驱动表
N 对扫描行数的影响更大，因此应该让小表来做驱动表



# 优化

提前过滤

驱动表为小表
外链接

# 缺点


# 示例

set optimizer_switch = 'block_nested_loop=off'

EXPLAIN	
SELECT o.id,o.a,c.id,c.code
FROM test_code c
JOIN test_order o
on o.a = c.code


id	select	table	type	key	key_len	rows	filtered	Extra
1	SIMPLE	c		ALL					9		100	
1	SIMPLE	o		ALL					1100	10	Using where


call insert_test_code(200,100);

EXPLAIN	
SELECT o.id,o.a,c.id,c.code
FROM test_order o
JOIN test_code c
on o.a = c.code

id	select_type	table	type	key		ref	rows	filtered	Extra
1	SIMPLE		o		ALL					100		100	
1	SIMPLE		c		ALL					200	10	Using where

自动选择小表



LEFT JOIN


EXPLAIN	
SELECT o.id,o.a,c.id,c.code
FROM test_code c
LEFT JOIN test_order o
on o.a = c.code

id	select_type	table	partitions	type	possible_keys	key	key_len	ref	rows	filtered	Extra
1	SIMPLE	c		ALL					200	100	
1	SIMPLE	o		ALL					100	100	Using where












  

