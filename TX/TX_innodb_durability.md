
# 持久性

    持久化的要求是
	事务一旦提交，数据的修改就是永久的.
	即使系统崩溃，提交的修改也不应该丢失。
    innodb认为
    把数据的修改保存到了磁盘中，也就完成了持久化
    
# 脏页

    innodb修改数据时，为了提高修改效率
    如果数据页在bufferpool中，会直接修改bufferpool中的数据页
    如果数据页不在bufferpool中，会在bufferpool中的changebuffer中记录下数据的修改
    然后，innodb会定期将修改写入到磁盘上
    
    对于事务已经提交，修改却没有写入到磁盘中的页被称为脏页
    由于脏页保存在内存中，如果系统崩溃，已提交的修改就会丢失
    
 # redolog机制           
 
    innodb使用redolog机制为了防止脏页的丢失  
    
    在修改数据时，
    innodb使用redolog记录相应的物理修改日志
    redolog会先保存到redo log buffer中，
    默认情况下，
    redolog会在事务提交前写入到redolog文件中
    
    在innodb启动时，
    不管数据库上次是否正常关闭
    都会尝试使用redolog恢复数据

# 特点

    为了提高修改效率，
    修改记录会缓存在bufferpool，不用每次提交都写磁盘
    但是，默认情况下
    修改对应的redolog要求在事务提交前写入磁盘
    这样岂不是多此一举？
    
    实际上
    1.redolog是顺序写，实际的修改是随机写，
    2.redolog刷新到策略可以修改
    
        


	




