

# 用法

作用

注意


# 原理

DISTINCT实际上和GROUP BY的操作非常相似，只不过是在GROUP BY之后的每组中只取出一条记录而已。
所以，DISTINCT的实现和GROUP BY的实现也基本差不多，没有太大的区别。
同样可以通过松散索引扫描或者是紧凑索引扫描来实现，当然，在无法仅仅使用索引即能完成DISTINCT的时候，MySQL只能通过临时表来完成。
但是，和GROUP BY有一点差别的是，DISTINCT并不需要进行排序。
也就是说，在仅仅只是DISTINCT操作的Query如果无法仅仅利用索引完成操作的时候，MySQL会利用临时表来做一次数据的“缓存”，但是不会对临时表中的数据进行filesort操作。
当然，如果我们在进行DISTINCT的时候还使用了GROUP BY并进行了分组，并使用了类似于MAX之类的聚合函数操作，就无法避免filesort了。




https://www.cnblogs.com/ggjucheng/archive/2012/11/18/2776449.html
