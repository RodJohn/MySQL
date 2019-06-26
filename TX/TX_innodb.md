

# 

只讨论innodb


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




# redolog

顺序写 顺序读

流程

内容

文件复用



# undolog

内容

                