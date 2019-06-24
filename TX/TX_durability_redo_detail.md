

# 内容

概述

    redo日志本质上只是记录了一下事务对数据库做了哪些修改


最简格式

    存储表空间ID、页号、具体更改
    
    redo日志中只需要记录一下在某个页面的某个偏移量处修改了几个字节的值，具体被修改的内容是啥就好了，
    设计InnoDB的大叔把这种极其简单的redo日志称之为物理日志



复杂

    比如插入
    
    把一条记录插入到一个页面时需要更改的地方非常多
    
    表中包含多少个索引，一条INSERT语句就可能更新多少棵B+树。
    
    Page Directory
    
    
    还存在 页分裂
    


Mini-Transaction，

    简称mtr，比如上边所说的修改一次Max Row ID的值算是一个Mini-Transaction，向某个索引对应的B+树中插入一条记录的过程也算是一个Mini-Transaction。
    通过上边的叙述我们也知道，一个所谓的mtr可以包含一组redo日志，在进行奔溃恢复时这一组redo日志作为一个不可分割的整体
    
    log buffer中写入redo日志时不是一条一条写入的，而是以一个mtr生成的一组redo日志为单位进行写入的。

LSN

    每一组由mtr生成的redo日志都有一个唯一的LSN值与其对应，LSN值越小，说明redo日志产生的越早。
    



# 日志文件


查看

    使用SHOW VARIABLES LIKE 'datadir'查看

地址

    MySQL的数据目录下默认有两个名为ib_logfile开头


# 循环写入

循环写入

    由于redo日志文件组容量是有限的
    所以必须循环使用redo日志文件组中的文件


checkpoint

    checkpoint_lsn来代表当前系统中可以被覆盖的redo日志总量是多少，
    判断某些redo日志占用的磁盘空间是否可以覆盖的依据就是它对应的脏页是否已经刷新到磁盘里。
    buf_next_to_write的全局变量，标记当前log buffer中已经有哪些日志被刷新到磁盘中了

    当有新的redo日志写入到log buffer时，首先lsn的值会增长，但flushed_to_disk_lsn不变，
    随后随着不断有log buffer中的日志被刷新到磁盘上，flushed_to_disk_lsn的值也跟着增长。
    如果两者的值相同时，说明log buffer中的所有redo日志都已经刷新到磁盘中了。
    
    
    
# 删除


    flush链表中的LSN
    我们知道一个mtr代表一次对底层页面的原子访问，在访问过程中可能会产生一组不可分割的redo日志，
    在mtr结束时，会把这一组redo日志写入到log buffer中。除此之外，在mtr结束时还有一件非常重要的事情要做，
    就是把在mtr执行过程中可能修改过的页面加入到Buffer Pool的flush链表
    












