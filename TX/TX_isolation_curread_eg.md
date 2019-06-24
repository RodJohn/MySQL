



# 示例

数据

    mysql> CREATE TABLE `t` (
      `id` int(11) NOT NULL,
      `k` int(11) DEFAULT NULL,
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB;
    insert into t(id, k) values(1,1),(2,2);

操作    
 
![图片]()


结果

    事务B查到的k的值是3
    事务A查到的k的值是1
 
   