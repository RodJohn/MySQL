
# 临时表

原理

  获取数据存入临时表，
  再进行排序和分组

示例

	环境

		SELECT VERSION();
		5.7.26

	SQL

		EXPLAIN SELECT a FROM test_order GROUP BY a


	explain

		Using temporary; Using filesort

	解析

		获取数据存入临时表，再进行排序和分组



# ORDER BY null 优化

原理

分析

	s省下了初步的排序，但是 对于 后续分组中的查找 不知道会不会有影响

示例

	环境

		同上
	
	仅groupby

		EXPLAIN SELECT a FROM test_order GROUP BY a
		Using temporary; Using filesort
		0
		1
		2
		3

	orderby null

		EXPLAIN SELECT a FROM test_order GROUP BY a ORDER BY null
		Using temporary

		321
		71
		23
		981

	select

		SELECT a FROM test_order

		321
		71
		23
		981
