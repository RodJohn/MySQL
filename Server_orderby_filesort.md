# filesort

	filesort包括全字段排序和rowid排序两种

# 全字段排序

## 原理

获取行的排序列、主键列、查询列


## 流程

	1. 找到满足的行；
	2. 取出排序列、主键列、查询列的值，存入内存(sort_buffer)中；
	3. 对排序字段做快速排序；
	4. 如果要排序的数据量小于 sort_buffer_size，排序就在内存中完成。否则将利用磁盘临时文件辅助排序


## 测试

### 测试环境
	
MySQL版本

	SELECT VERSION();
	5.7.26
	
sort_buffer_size

	show variables like '%sort_buffer_size%';
	sort_buffer_size	262144 
	
表结构	

	DROP TABLE IF EXISTS test.test_order;
	CREATE TABLE test.test_order(
	id int(10) not null auto_increment,
	a int(10) not null,
	b int(10) not null,
	c int(10) not null,
	PRIMARY key (`id`)
	)ENGINE INNODB DEFAULT CHARSET utf8 COMMENT '测试表';

存储过程

	DROP PROCEDURE IF EXISTS insert_test_order;
	##num_limit 要插入数据的数量,rand_limit 最大随机的数值
	CREATE PROCEDURE insert_test_order(in num_limit int,in rand_limit int)
	BEGIN

	DECLARE i int default 1;
	DECLARE a int default 1;
	DECLARE b int default 1;
	DECLARE c int default 1;

	WHILE i<=num_limit do

	set a = FLOOR(rand()*rand_limit);
	set b = FLOOR(rand()*rand_limit);
	set c = FLOOR(rand()*rand_limit);
	INSERT into test.test_order values (null,a,b,c);
	set i = i + 1;

	END WHILE;

	END

插入数据

	call insert_test_order(1000,1000);

### 测试方案

分析

	explain只能查看到使用Using filesort使用
	需使用optimizer_trace追踪SQL

解析
	
	number_of_tmp_files

		表示的是，排序过程中使用的临时文件数。
		外部排序一般使用归并排序算法。
		sort_buffer_size越小，需要分成的份数越多，number_of_tmp_files的值就越大。
		
	sort_mode

### 测试纯内存排序

SQL

	SELECT * FROM test_order ORDER BY a ; 

trace结果

	"filesort_summary": {
	  "rows": 1001,
	  "examined_rows": 1000,
	  "number_of_tmp_files": 0,
	  "sort_buffer_size": 28032,
	  "sort_mode": "<sort_key, additional_fields>"
	}
		
解析

	sort_mode = sort_key
		表示使用全字段排序
	examined_rows = 1000 
		表示需要对全部的数据排序
    number_of_tmp_files = 0
		表示不使用文件排序

### 测试部分文件排序

插入数据

	call insert_test_order(11000,1000);

SQL

	SELECT * FROM test_order ORDER BY a ; 

trace结果

	"filesort_summary": {
	  "rows": 12782,
	  "examined_rows": 12782,
	  "number_of_tmp_files": 2,
	  "sort_buffer_size": 262136,
	  "sort_mode": "<sort_key, additional_fields>"
	}
		
解析

	sort_mode = sort_key
		表示使用全字段排序
	examined_rows = 12782 
		表示需要对全部的数据排序
    number_of_tmp_files = 2
		表示使用文件排序


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



# 对比



这也就体现了MySQL的一个设计思想：如果内存够，就要多利用内存，尽量减少磁盘访问。

对于InnoDB表来说，rowid排序会要求回表多造成磁盘读，因此不会被优先选择。

随机IO


	
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
	

	全字段排序是MySQL5.0以后
	
	
	
	
	
	# 内存排序
	
	
	# 使用临时表排序
	

  


