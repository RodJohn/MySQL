



# 前提

    同一个innodb内的trxid是递增的
    所以可以通过比较trxid来比较事务发生的先后

# 版本链

作用

    undo日志之间的版本链来保存记录的历史值

结构


    在Innodb中，
    每次对记录进行改动后(不管有没有提交)
    记录的隐藏列存trxId会记录当前的事务id
    记录的隐藏列roll_pointer会指向本次的undo日志
    
    undo日志中可以获得记录原来的值，
    同时undo日志也有roll_pointer属性指向更早的版本的undo日志
    （INSERT操作对应的undo日志没有该属性，因为该记录并没有）
    记录和undo日志之间形成的链表就是数据的版本链
    

图例


# ReadView（一致性视图）

作用

    ReadView来设定快照的标识

结构

    ReadView中主要包含4个比较重要的属性
    creator_trx_id：表示生成该ReadView的事务的事务id
    m_ids：表示在生成ReadView时当前系统中活跃的读写事务的事务id列表。
    min_trx_id：表示在生成ReadView时当前系统中活跃的读写事务中最小的事务id，也就是m_ids中的最小值。
    max_trx_id：表示生成ReadView时系统中应该分配给下一个事务的id值。

# 判断标准

    通过ReadView判断版本链中的哪个版本是当前事务可见的
 
     对于事务中查询到的记录，从记录的开始判断数据
     
     如果被访问版本的trx_id = ReadView中的creator_trx_id
         意味着当前事务在访问它自己修改过的记录，所以该版本可以被当前事务访问。
     如果被访问版本的trx_id < ReadView中的min_trx_id
         表明生成该版本的事务在当前事务生成ReadView前已经提交，所以该版本可以被当前事务访问。
     如果被访问版本的trx_id > ReadView中的max_trx_id值，
         表明生成该版本的事务在当前事务生成ReadView后才开启，所以该版本不可以被当前事务访问。
     如果被访问版本的trx_id属性值在ReadView的min_trx_id和max_trx_id之间，
         那就需要判断一下trx_id属性值是不是在m_ids列表中，如果在，说明创建ReadView时生成该版本的事务还是活跃的，该版本不可以被访问；
         如果不在，说明创建ReadView时生成该版本的事务已经被提交，该版本可以被访问。
         
     图例        
 
     如果某个版本的数据对当前事务不可见的话，那就顺着版本链找到下一个版本的数据，
     继续按照上边的步骤判断可见性，依此类推，直到版本链中的最后一个版本。
     如果最后一个版本也不可见的话，那么就意味着该条记录对该事务完全不可见，查询结果就不包含该记录。
 
   

# 创建ReadView

    READ COMMITTED —— 每次读取数据前都生成一个ReadView
    REPEATABLE READ —— 在第一次读取数据时生成一个ReadView






    /**** TODO **/
    
    产生ReadView时不一定有 事务id  
    
    start stra 不一定会 修改数据
    


