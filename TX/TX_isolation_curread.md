


# 当前读

    更新数据都是先读后写的，而这个读，只能读当前的值，称为“当前读”（current read）。
    当前读。其实，除了update语句外，select语句如果加锁，也是当前读。
    select * from t where id=1修改一下，加上lock in share mode 或 for update，

    当前读的规则，就是要能读到所有已经提交的记录的最新值。
    如果没提交 就等待
    
    锁实现
    

# 隔离性

脏写 行锁锁住写不了
脏读 行锁锁住写不了
不可重复读  行锁锁住
幻读  间隙锁锁住 插入不了

# 语句

当前读：特殊的读操作，插入/更新/删除操作，属于当前读，需要加锁。
select * from table where ? lock in share mode;
select * from table where ? for update;
insert into table values (…);
update table set ? where ?;
delete from table where ?;




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
 
   