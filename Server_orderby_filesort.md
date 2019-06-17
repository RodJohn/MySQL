# filesort

	filesort需要对数据进行全部排序
	filesort包括全字段排序和rowid排序两种
	
# 比较
	
	
	## 条件

	在上面这个算法过程里面，只对原表的数据读了一遍，剩下的操作都是在sort_buffer和临时文件中执行的。但这个算法有一个问题，就是如果查询要返回的字段很多的话，那么sort_buffer里面要放的字段数太多，这样内存里能够同时放下的行数很少，要分成很多个临时文件，排序的性能会很差。
	所以如果单行很大，这个方法效率不够好。
	那么，如果MySQL认为排序的单行长度太大会怎么做呢？
	接下来，我来修改一个参数，让MySQL采用另外一种算法。
	SET max_length_for_sort_data = 16;
	max_length_for_sort_data，是MySQL中专门控制用于排序的行数据的长度的一个参数。它的意思是，如果单行的长度超过这个值，MySQL就认为单行太大，要换一个算法。

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
	

  
	

# 全字段排序

### 原理

原理

	获取行的排序列、主键列、查询列，对排序列排序

流程

	1. 找到满足的行；
	2. 取出排序列、主键列、查询列的值，存入内存(sort_buffer)中；
	3. 对排序字段做快速排序；
	4. 如果要排序的数据量小于 sort_buffer_size，排序就在内存中完成。否则将利用磁盘临时文件辅助排序


### 测试

测试环境
	
	MySQL版本

		SELECT VERSION();
		5.7.26

	sort_buffer_size

		show variables like '%sort_buffer_size%';
		sort_buffer_size	262144 
	
	max_length_for_sort_data
	
		show variables like '%max_length_for_sort_data%';
		max_length_for_sort_data	1024

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

测试方案

	分析

		explain只能查看到使用Using filesort使用
		需使用optimizer_trace追踪SQL

	解析

		number_of_tmp_files

			表示的是，排序过程中使用的临时文件数。
			外部排序一般使用归并排序算法。
			sort_buffer_size越小，需要分成的份数越多，number_of_tmp_files的值就越大。

		sort_mode

测试纯内存排序

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

测试部分文件排序

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

### 原理

原理

	获取排序列和主键id，对排序列排序，然后回表获取需要的查询列

流程

	1. 找到满足的行；
	2. 取出排序列、主键列，存入内存(sort_buffer)中；
	3. 对排序字段做快速排序；
	4. 如果要排序的数据量小于 sort_buffer_size，排序就在内存中完成。否则将利用磁盘临时文件辅助排序
	5. 回表获取查询列（需要时）


### 测试

环境、方案

	同上



测试排序行宽度

	修改行宽度

		SET max_length_for_sort_data = 16;

	SQL

		SELECT * FROM test_order ORDER BY a ; 

	trace结果

		"filesort_summary": {
		  "rows": 12782,
		  "examined_rows": 12782,
		  "number_of_tmp_files": 0,
		  "sort_buffer_size": 262144,
		  "sort_mode": "<sort_key, rowid>"
		}

	解析

		sort_mode = sort_key rowid
			表示使用rowId排序
		examined_rows = 12782 
			表示需要对全部的数据排序
		number_of_tmp_files = 0
			表示没有使用文件排序
			这时候参与排序的行数虽然仍然是全表，但是每一行都变小了，需要的临时文件也相应地变少了

测试返回行类型

	重建表

		DROP TABLE IF EXISTS test.test_order;
		CREATE TABLE test.test_order(
		id int(10) not null auto_increment,
		a int(10) not null,
		b int(10) not null,
		c int(10) not null,
		PRIMARY key (`id`)
		)ENGINE INNODB DEFAULT CHARSET utf8 COMMENT '测试表';

	插入数据

		call insert_test_order(10,1000);

	修改行宽度

		SET max_length_for_sort_data = 1024;
		
	添加新列

		ALTER TABLE `test`.`test_order` 
		ADD COLUMN `t` text NULL AFTER `c`;
		
	测试一

		SELECT * FROM test_order ORDER BY a;

		"filesort_summary": {
		  "rows": 10,
		  "examined_rows": 10,
		  "number_of_tmp_files": 0,
		  "sort_buffer_size": 14560,
		  "sort_mode": "<sort_key, rowid>"
		 }
	
	解析
	
		返回列包含text类型数据，使用rowid排序
		  
	测试二

		select a,b,c from test_order order by a;

		"filesort_summary": {
		  "rows": 10,
		  "examined_rows": 10,
		  "number_of_tmp_files": 0,
		  "sort_buffer_size": 25480,
		  "sort_mode": "<sort_key, additional_fields>"
		}
