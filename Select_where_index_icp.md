


# 参考

# ICP

MySQL 索引条件下推 Index Condition Pushdown 出现在MySQL5.6及之后的版本中，能大幅提升查询效率

# 原理

索引比较是在InnoDB存储引擎层进行的。而数据表的记录比较first_name条件是在MySQL引擎层进行的。
开启ICP之后，包含在索引中的数据列条件(即上述SQL中的first_name LIKE %sal') 都会一起被传递给InnoDB存储引擎，这样最大限度的过滤掉无关的行。



# 联合索引示例

https://www.jianshu.com/p/bdc9e57ccf8b

set optimizer_switch='index_condition_pushdown=on';


# 主键示例





