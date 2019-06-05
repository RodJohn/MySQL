
# COMPACT

	Compact行记录是在MySql 5.0中引入的，其设计目标是高效地存储数据。
	简单来说，一个页中存放的行数据越多，其性能就越高

# 格式

	Compact包括 变长字段长度列表、NULL值列表、头信息、隐藏列、用户数据

# 变长字段长度列表

	把所有非null的变长字段的真实数据占用的字节长度逆序存放


# NULL值列表

	将每个允许存储NULL的列对应一个二进制位，二进制位按照列的顺序逆序排列，
	二进制位的值为1时，代表该列的值为NULL。

# 头信息

	常用的头信息有delete_mask、next_record

完整如下

	delete_mask	1 	标记该记录是否被删除
	min_rec_mask	1	B+树的每层非叶子节点中的最小记录都会添加该标记
	n_owned	4 	表示当前记录拥有的记录数
	heap_no	13	 表示当前记录在记录堆的位置信息
	record_type	3	 表示当前记录的类型，0表示普通记录，1表示B+树非叶子节点记录，2表示最小记录，3表示最大记录
	next_record	16	 表示下一条记录的相对位置

# 隐藏列

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


# 用户数据

## NULL

	不管是char还是varchar类型，NULL不占该部分任何空间，
	即NULL除了占有NULL标志位，实际存储不占有任何空间。 

VARCHAR(M)

	M表示字符，不是字节
  	占用实际长度


## CHAR(M)

定长？变长

	通常的理解VARCHAR是存储变长长度的字符类型，CHAR是存储定长长度的字符类型。
	从MySQL 4.1开始，CHR（N）中的N指的是字符的长度，而不是之前版本的字节长度。
	此时，在不同的字符集下，CHAR的内部存储的不是定长的数据。
	如果采用变长字符集时，该列占用的字节数也会被加到变长字段长度列表。

最低长度

	变长字符集的CHAR(M)类型的列要求至少占用M个字节，
	比方说对于使用utf8字符集的CHAR(10)的列来说，该列存储的数据字节长度的范围是10～30个字节。
	即使我们向该列中存储一个空字符串也会占用10个字节，
	这是怕将来更新该列的值的字节长度大于原有值的字节长度而小于10个字节时，
	可以在该记录处直接更新，而不是在存储空间中重新分配一个新的记录空间，导致原有的记录空间成为所谓的碎片。


示例

	show table status like 'user'
		user	InnoDB	10	Compact	utf8mb4_unicode_ci
	desc user
		id	    int(11)	 NO	 PRI	
		name	char(2)	 YES		
	select name,LENGTH(name),CHAR_LENGTH(name) from user
		a	  1	  1
		好好	6	2
		aa	  2	  2
	最短长度的还得用其他的得工具查看





  
## 实例说明

参考

	https://blog.csdn.net/linux_ever/article/details/64124868
	https://www.cnblogs.com/wade-luffy/p/6289448.html
	
	

