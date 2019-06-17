


索引失效  

数据转换  函数 隐式转换  


# 函数

对索引字段做函数操作，可能会破坏索引值的有序性，因此优化器就决定放弃走树搜索功能

B+树提供的这个快速定位能力，来源于同一层兄弟节点的有序性。
但是，如果计算month()函数的话，你会看到传入7的时候，在树的第一层就不知道该怎么办了。

示例

select count(*) from tradelog where month(t_modified)=7;

# 隐式类型转换



mysql> select * from tradelog where tradeid=110717;
交易编号tradeid这个字段上，本来就有索引，但是explain的结果却显示，这条语句需要走全表扫描。

你可能也发现了，tradeid的字段类型是varchar(32)，而输入的参数却是整型，所以需要做类型转换。


在MySQL中，字符串和数字做比较的话，是将字符串转换成数字。



就知道对于优化器来说，这个语句相当于：mysql> select * from tradelog where  CAST(tradid AS signed int) = 110717;





