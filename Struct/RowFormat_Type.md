
# COMPACT


	Compact行记录是在MySql 5.0中引入的，其设计目标是高效地存储数据。
	简单来说，一个页中存放的行数据越多，其性能就越高

格式

	Compact包括 变长字段长度列表、NULL值列表、头信息、隐藏列、用户数据

## 变长字段长度列表

	把所有非null的变长字段的真实数据占用的字节长度逆序存放


## NULL值列表

	将每个允许存储NULL的列对应一个二进制位，二进制位按照列的顺序逆序排列，
	二进制位的值为1时，代表该列的值为NULL。

## 头信息

	常用的头信息有delete_mask、next_record

完整如下

	delete_mask	1 	标记该记录是否被删除
	min_rec_mask	1	B+树的每层非叶子节点中的最小记录都会添加该标记
	n_owned	4 	表示当前记录拥有的记录数
	heap_no	13	 表示当前记录在记录堆的位置信息
	record_type	3	 表示当前记录的类型，0表示普通记录，1表示B+树非叶子节点记录，2表示最小记录，3表示最大记录
	next_record	16	 表示下一条记录的相对位置

## 隐藏列

	隐藏列包括row_id、transaction_id、roll_pointer
	
row_id	

	row_id是MySQL在特定情况下生成的主键

	InnoDB表对主键的生成策略：
	优先使用用户自定义主键作为主键，
	如果用户没有定义主键，则选取一个Unique键作为主键，
	如果表中连Unique键都没有定义的话，则InnoDB会为表默认添加一个名为row_id的隐藏列作为主键。


transaction_id

	事务ID
	
roll_pointer

	回滚指针


# Redundant

	Redundant行格式是MySQL5.0之前用的一种行格式，也就是说它已经非常老了


字段长度偏移列表

	一个字段长度偏移列表
	按照两个相邻数值的差值来计算各个列值的长度的意思就是：



# Dynamic和Compressed


	MySQL版本是5.7，它的默认行格式就是Dynamic，


	这俩行格式和Compact行格式挺像，只不过在处理行溢出数据时有点儿分歧，它们不会在记录的真实数据处存储字段真实数据的前768个字节，
	而是把所有的字节都存储到其他页面中，只在记录的真实数据处存储其他页面的地址

	Compressed行记录格式的另一个功能就是，存储在其中的行数据会以zlib的算法进行压缩，因此对于BLOB、TEXT、VARCHAR这类大长度类型的数据能进行非常有效的存储。


  
## 实例说明

参考

	https://blog.csdn.net/linux_ever/article/details/64124868
	https://www.cnblogs.com/wade-luffy/p/6289448.html
	
	

