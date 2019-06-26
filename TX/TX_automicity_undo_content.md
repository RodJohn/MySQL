
undolog 是个 逻辑日志  


目的就是消除掉原先的修改

当执行rollback时，就可以从undo log中的逻辑记录读取到相应的内容并进行回滚。


# undo日志

    每条记录在更新的时候都会同时记录一条回滚操作。
    记录上的最新值，通过回滚操作，都可以得到前一个状态的值

#  INSERT

    如果希望回滚这个插入操作，那么把这条记录删除就好了，
    也就是说在写对应的undo日志时，主要是把这条记录的主键信息记上。
    
# DELETE

    记录主键信息

    在对一条记录进行delete mark操作前，
    需要把该记录的旧的trx_id和roll_pointer隐藏列的值都给记到对应的undo日志中来
    
    
# UPDATE

在不更新主键的情况下

    记录主键信息
    需要把该记录的旧的trx_id和roll_pointer隐藏列的值都给记到对应的undo日志中来
    记录更新前的数据
    
更新主键的情况
    
    其实就是一个DELETE的undolog+一个INSERT的undolog
  
  
