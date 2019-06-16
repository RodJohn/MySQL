# 排序

filesort的效率未必比索引扫描排序低


# 索引排序


## 原理


## 要求

在排序之前，数据就是按照排序列顺序读取的，而且未发生改变

如果有where where 中使用的相同索引命中
如果有join 排序列必须在驱动表

主键索引起效 

二级索引 看数量  回表  limt 范围

配合示例 说明


单表排序
驱动表排序

# 参考

https://blog.csdn.net/u011215669/article/details/82078812


# 


DROP TABLE IF EXISTS test.test_order;
CREATE TABLE test.test_order(
id int(10) not null auto_increment,
a int(10) not null,
b int(10) not null,
c int(10) not null,
PRIMARY key (`id`)
)ENGINE INNODB DEFAULT CHARSET utf8 COMMENT '测试表';




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

call insert_test_order(10000,1000);


EXPLAIN SELECT * FROM test_order where a > 900 ORDER BY a 
EXPLAIN SELECT * FROM test_order  ORDER BY a limit 2


EXPLAIN SELECT * FROM test_order where a = 275

ALTER TABLE `test`.`test_order` 
ADD INDEX `idx_a`(`a`) USING BTREE;







# sort 

比较耗时的操作

索引
 需要维护索引 特定条件下才能用 
filesort
 使用内存或者文件进行


