
# 分组

GROUP BY相当于
先排序，在分组，最后处理聚合


# 索引


MySQL在进行GROUP BY操作的时候要想利用索引，
必须满足GROUP BY的字段必须同时存放于同一个索引中，且该索引是一个有序索引（如Hash索引就不能满足要求）。
而且，并不只是如此，是否能够利用索引来实现GROUP BY还与使用的聚合函数也有关系。

优化Group By最有效的办法是当可以直接使用索引来完全获取需要group的字段






# 优化

对于上面三种MySQL处理GROUP BY的方式，我们可以针对性的得出如下两种优化思路：

1.尽可能让MySQL可以利用索引来完成GROUP BY操作，当然最好是松散索引扫描的方式最佳。在系统允许的情况下，我们可以通过调整索引或者调整Query这两种方式来达到目的；

2.当无法使用索引完成GROUP BY的时候，由于要使用到临时表且需要filesort，所以我们必须要有足够的sort_buffer_size来供MySQL排序的时候使用，而且尽量不要进行大结果集的GROUP BY操作，因为如果超出系统设置的临时表大小的时候会出现将临时表数据copy到磁盘上面再进行操作，这时候的排序分组操作性能将是成数量级的下降；

 

至于如何利用好这两种思路，还需要大家在自己的实际应用场景中不断的尝试并测试效果，最终才能得到较佳的方案。
此外，在优化GROUP BY的时候还有一个小技巧可以让我们在有些无法利用到索引的情况下避免filesort操作，也就是在整个语句最后添加一个以null排序（ORDER BY null）的子句，大家可以尝试一下试试看会有什么效果。

 
 https://www.cnblogs.com/ggjucheng/archive/2012/11/18/2776449.html
 
