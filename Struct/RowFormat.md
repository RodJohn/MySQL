
# 行格式

	InnoDB用行的形式存储用户数据

# 种类

	InnoDB现有4种不同类型的行格式，分别是Compact、Redundant、Dynamic和Compressed


	Compact行记录是在MySql 5.0中引入的，其设计目标是高效地存储数据 
	Null Char 对比redundant
	行溢出 对比
	

# 查看

 	用户可以通过命令查看表使用的行格式，其中row_format属性表示当前所使用的行记录结构类型。
	 show table status like 'table_name'




