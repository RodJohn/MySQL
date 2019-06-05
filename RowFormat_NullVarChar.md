
  
# Null

Redundant

	对于varchar类型的NULL值，Redundant行记录格式同样不占用任何存储空间
	而char类型的NULL值需要占用空间。

Compact

	compact拥有null标识位	
	不管是char还是varchar类型，NULL不占该部分任何空间，
	即NULL除了占有NULL标志位，实际存储不占有任何空间。 




# CHAR(M)

## Redundant

	占用的真实数据空间就是该字符集表示一个字符最多需要的字节数和M的乘积。
	比方说使用utf8字符集的CHAR(10)类型的列占用的真实数据空间始终为30个字节，
	使用gbk字符集的CHAR(10)类型的列占用的真实数据空间始终为20个字节。
	由此可以看出来，使用Redundant行格式的CHAR(M)类型的列是不会产生碎片的。


## Compact

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

