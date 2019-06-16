# filesort





# 全字段排序

## 原理



## 流程

1. 初始化 sort_buffer，确定放入 name、city、age 这三个字段；
2. 从索引 city 找到第一个满足 city='杭州’条件的主键 id，也就是图中的 ID_X；
3. 到主键 id 索引取出整行，取 name、city、age 三个字段的值，存入 sort_buffer 中；
4. 从索引 city 取下一个记录的主键 id；
5. 重复步骤 3、4 直到 city 的值不满足查询条件为止，对应的主键 id 也就是图中的
ID_Y；
6. 对 sort_buffer 中的数据按照字段 name 做快速排序；

图中“按 name 排序”这个动作，可能在内存中完成，也可能需要使用外部排序，这取决
于排序所需的内存和参数 sort_buffer_size。
sort_buffer_size，就是 MySQL 为排序开辟的内存（sort_buffer）的大小。如果要排序
的数据量小于 sort_buffer_size，排序就在内存中完成。但如果排序数据量太大，内存放
不下，则不得不利用磁盘临时文件辅助排序



## 示例

SQL



查看



	来确定一个排序语句是否使用了临时文件


/* 打开optimizer_trace，只对本线程有效 */
SET optimizer_trace='enabled=on'; 

/* @a保存Innodb_rows_read的初始值 */
select VARIABLE_VALUE into @a from  performance_schema.session_status where variable_name = 'Innodb_rows_read';

/* 执行语句 */
select city, name,age from t where city='杭州' order by name limit 1000; 

/* 查看 OPTIMIZER_TRACE 输出 */
SELECT * FROM `information_schema`.`OPTIMIZER_TRACE`\G

/* @b保存Innodb_rows_read的当前值 */
select VARIABLE_VALUE into @b from performance_schema.session_status where variable_name = 'Innodb_rows_read';

/* 计算Innodb_rows_read差值 */
select @b-@a;
这个方法是通过查看 OPTIMIZER_TRACE 的结果来确认的，你可以从 number_of_tmp_files中看到是否使用了临时文件。



number_of_tmp_files

	表示的是，排序过程中使用的临时文件数。
	外部排序一般使用归并排序算法。
	sort_buffer_size越小，需要分成的份数越多，number_of_tmp_files的值就越大。


接下来，我再和你解释一下图4中其他两个值的意思。
我们的示例表中有4000条满足city='杭州’的记录，所以你可以看到 

examined_rows

	表示参与排序的行数是4000行。
	sort_mode 里面的packed_additional_fields的意思是，排序过程对字符串做了“紧凑”处理。即使name字段的定义是varchar(16)，在排序过程中还是要按照实际长度来分配空间的。
同时，最后一个查询语句select @b-@a 的返回结果是4000，表示整个执行过程只扫描了4000行。





# rowid排序


## 原理


新的算法放入sort_buffer的字段，只有要排序的列（即name字段）和主键id。


## 过程

初始化sort_buffer，确定放入两个字段，即name和id；
从索引city找到第一个满足city='杭州’条件的主键id，也就是图中的ID_X；
到主键id索引取出整行，取name、id这两个字段，存入sort_buffer中；
从索引city取下一个记录的主键id；
重复步骤3、4直到不满足city='杭州’条件为止，也就是图中的ID_Y；
对sort_buffer中的数据按照字段name进行排序；
遍历排序结果，取前1000行，并按照id的值回到原表中取出city、name和age三个字段返回给客户端。


## 示例


SQL 

检测


从OPTIMIZER_TRACE的结果中，你还能看到另外两个信息也变了。
sort_mode变成了<sort_key, rowid>，表示参与排序的只有name和id这两个字段。
number_of_tmp_files变成10了，是因为这时候参与排序的行数虽然仍然是4000行，但是每一行都变小了，因此需要排序的总数据量就变小了，需要的临时文件也相应地变少了。



## 条件

在上面这个算法过程里面，只对原表的数据读了一遍，剩下的操作都是在sort_buffer和临时文件中执行的。但这个算法有一个问题，就是如果查询要返回的字段很多的话，那么sort_buffer里面要放的字段数太多，这样内存里能够同时放下的行数很少，要分成很多个临时文件，排序的性能会很差。
所以如果单行很大，这个方法效率不够好。
那么，如果MySQL认为排序的单行长度太大会怎么做呢？
接下来，我来修改一个参数，让MySQL采用另外一种算法。
SET max_length_for_sort_data = 16;
max_length_for_sort_data，是MySQL中专门控制用于排序的行数据的长度的一个参数。它的意思是，如果单行的长度超过这个值，MySQL就认为单行太大，要换一个算法。



filesort
	 使用内存或者文件进行 慢
	 使用内存或者文件进行
	
	优缺点
	
	
	
	# FileSort
	
	排序缓存 
	  设置值
	排序算法
	  一次扫描  * 
	  两次扫描  旧的 
	  选择 一次 除非 blob text  数据过大
	
	# 排序算法
	
	所以filesort是否会使用磁盘取决于它操作的数据量大小。
	
	总结来说就是，filesort按排序方式来划分 分为两种：
	
	1.数据量小时，在内存中快排
	2.数据量大时，在内存中分块快排，再在磁盘上将各个块做归并
	
	数据量大的情况下涉及到磁盘io，所以效率会低一些。
	
	根据回表查询的次数，filesort又可以分为两种方式：
	
	1.回表读取两次数据(two-pass)：两次传输排序
	2.回表读取一次数据(single-pass)：单次传输排序
	
	
	
	## 全字段排序
	
	## rowId排序
	
	## 对比
	
	全字段排序是MySQL5.0以后
	
	
	
	
	
	# 内存排序
	
	
	# 使用临时表排序
	
	## 示例
	
	SQL
	
	EXPLAIN
  


