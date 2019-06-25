


# 流程

    redo日志是首先写到log buffer中，之后才会被刷新到磁盘上的redo日志文件。
    （提高效率）

## redo log buffer

    redo log buffer的连续内存空间，翻译成中文就是redo日志缓冲区，
    把通过mtr生成的redo日志都放在了大小为512字节的页中

## 刷盘

刷盘

    但是为了保证持久性，必须要把修改这些页面对应的redo日志刷新到磁盘。

刷新策略

    innodb_flush_log_at_trx_commit
    
    提交刷新    
        在事务提交完成之前把该事务所修改的所有页面都刷新到磁盘，
        性能
  
    后台刷新  
        后台有一个线程，大约每秒都会刷新一次log buffer中的redo日志到磁盘。
    


    一般设置为 提交刷新



