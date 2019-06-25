

# 实现 

一致性
    undolog 破坏了约束 ，就回滚
    
持久性
    redo log 避免bufferpool中的更新丢失
    
原子性

    出错 回滚    

隔离性

    锁
    当前读 undolog
    
# WAL



是不是只有 redo log buffer 

调整

innodb_flush_log_at_trx_commit 三种取值 0 、1、2 对性能有很大影响


可能丢失1秒的事务数据。
log buffer

http://blog.sina.com.cn/s/blog_53b13d9501011uma.html
https://blog.csdn.net/joeyon1985/article/details/44828329
https://www.cnblogs.com/yuyue2014/p/3735035.html

# redolog

流程

内容

文件复用



# undolog

内容

                