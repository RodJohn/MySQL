


# 流程

# buffer 
redo log buffer的连续内存空间，翻译成中文就是redo日志缓冲区，

把通过mtr生成的redo日志都放在了大小为512字节的页中

# shuapan

redo日志刷盘时机
事务提交时

我们前边说过之所以使用redo日志主要是因为它占用的空间少，还是顺序写，在事务提交时可以不把修改过的Buffer Pool页面刷新到磁盘，但是为了保证持久性，必须要把修改这些页面对应的redo日志刷新到磁盘。


# 
flushed_to_disk_lsn

redo日志是首先写到log buffer中，之后才会被刷新到磁盘上的redo日志文件。
所以设计InnoDB的大叔提出了一个称之为buf_next_to_write的全局变量，标记当前log buffer中已经有哪些日志被刷新到磁盘中了。画



当有新的redo日志写入到log buffer时，首先lsn的值会增长，但flushed_to_disk_lsn不变，
随后随着不断有log buffer中的日志被刷新到磁盘上，flushed_to_disk_lsn的值也跟着增长。
如果两者的值相同时，说明log buffer中的所有redo日志都已经刷新到磁盘中了。





# 删除


flush链表中的LSN
我们知道一个mtr代表一次对底层页面的原子访问，在访问过程中可能会产生一组不可分割的redo日志，
在mtr结束时，会把这一组redo日志写入到log buffer中。除此之外，在mtr结束时还有一件非常重要的事情要做，
就是把在mtr执行过程中可能修改过的页面加入到Buffer Pool的flush链表




# 文件


redo日志文件

有一个很不幸的事实就是我们的redo日志文件组容量是有限的，我们不得不选择循环使用redo日志文件组中的文件

判断某些redo日志占用的磁盘空间是否可以覆盖的依据就是它对应的脏页是否已经刷新到磁盘里。

redo日志文件组
MySQL的数据目录（使用SHOW VARIABLES LIKE 'datadir'查看）下默认有两个名为ib_logfile0和ib_logfile1的文件，log buffer中的日志默认情况下就是刷新到这两个磁盘文件中。


# checkpoint

循环使用redo日志文件
一个全局变量checkpoint_lsn来代表当前系统中可以被覆盖的redo日志总量是多少，



# 

优化insert
顺序插入

不使用物理 delete 

行格式 紧凑 改新型 
变长字符
溢出数据  
	


# redo日志的写入过程















