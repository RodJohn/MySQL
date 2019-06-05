
# 概述

InnoDB用行的形式存储记录

# 分类

4种不同类型的行格式，分别是Compact、Redundant、Dynamic和Compressed行格式

Redundant格式是为了兼容之前版本而保留的  
在MySql 5.1版本中，默认设置为Compact行格式
Compact行记录是在MySql 5.0中引入的，其设计目标是高效地存储数据 


# 查看

 show table status like 'test'
 用户可以通过下面命令查看表使用的行格式，其中row_format属性表示当前所使用的行记录结构类型。


设置



