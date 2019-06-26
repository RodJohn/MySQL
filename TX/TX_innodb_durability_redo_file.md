# 日志文件


## 文件

查看

    使用SHOW VARIABLES LIKE 'datadir'查看

地址

    MySQL的数据目录下默认有两个名为ib_logfile开头

innodb_log_file_size：#事务日志的大小，默认5M
innodb_log_files_group =2：# 事务日志组中的事务日志文件个数，默认2个

必须修改 直接影响到



## 循环写入

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
  