


在数据修改的时候，不仅记录了redo，还记录了相对应的undo



undo log

    innodb的
    undo log有两个作用：提供回滚和多个行版本控制(MVCC)。