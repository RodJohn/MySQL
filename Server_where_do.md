


# 执行

建立了索引不一定使用索引，只有在二级索引 + 回表的代价比全表扫描的代价更低时才会使用索引。

怎样在单表中执行查询语句我们在前一章都唠叨过了，只需要选取代价最小的那种访问方法去执行单表查询语句就好了（就是说从const、ref、ref_or_null、range、index、all这些执行方法中选取代价最小的去执行查询）

# 索引合并

## Intersection合并

这里是说某个查询可以使用多个二级索引，将从多个二级索引中查询到的结果取交集，比方说下边这个查询：


示例

	SELECT * FROM single_table WHERE key1 = 'a' AND key3 = 'b';


	从idx_key1二级索引对应的B+树中取出key1 = 'a'的相关记录。
	从idx_key3二级索引对应的B+树中取出key3 = 'b'的相关记录。
	二级索引的记录都是由索引列 + 主键构成的，所以我们可以计算出这两个结果集中id值的交集。
	按照上一步生成的id值列表进行回表操作，也就是从聚簇索引中把指定id值的完整用户记录取出来，返回给用户。


要求

	情况一：二级索引列是等值匹配的情况
	SELECT * FROM single_table WHERE key1 = 'a' AND key_part = 'a';

	情况二：主键列可以是范围匹配
	SELECT * FROM single_table WHERE id > 100 AND key1 = 'a';


原理

二级索引的用户记录是由索引列 + 主键构成的，
二级索引列的值相同的记录可能会有好多条，这些索引列的值相同的记录又是按照主键的值进行排序的
此时求交集的过程就很easy

从idx_key1中获取到的主键值上直接运用条件id > 100过滤

## Union合并

SELECT * FROM single_table WHERE key1 = 'a' OR key3 = 'b'


# 索引起效

一般情况下只能利用单个二级索引执行查询，比方说下边的这个查询：

SELECT * FROM single_table WHERE key1 = 'abc' AND key2 > 1000;



# 索引失效

