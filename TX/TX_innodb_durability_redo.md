

 # 内容
 
 概述
 
    redo日志是物理日志
    记录了事务对数据库做了哪些修改

## 格式
     
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
 
## Mini-Transaction，
 
     简称mtr，比如上边所说的修改一次Max Row ID的值算是一个Mini-Transaction，向某个索引对应的B+树中插入一条记录的过程也算是一个Mini-Transaction。
     通过上边的叙述我们也知道，一个所谓的mtr可以包含一组redo日志，在进行奔溃恢复时这一组redo日志作为一个不可分割的整体
     log buffer中写入redo日志时不是一条一条写入的，而是以一个mtr生成的一组redo日志为单位进行写入的。
 
## LSN
 
     每一组由mtr生成的redo日志都有一个唯一的LSN值与其对应，LSN值越小，说明redo日志产生的越早。
     LSN称为日志的逻辑序列号(log sequence number)，在innodb存储引擎中，lsn占用8个字节。LSN的值会随着日志的写入而逐渐增大。
     




  
 
 
# 恢复

    在innodb启动时，
    不管数据库上次是否正常关闭
    都会尝试使用redolog恢复数据

    checkpoint表示已经刷新到硬盘的修改
    所以，只需要恢复大于checkpoint的redolog

    而且redolog是物理日志，恢复起来快
 
# 摘抄

    https://www.cnblogs.com/f-ck-need-u/p/9010872.html
    https://juejin.im/book/5bffcbc9f265da614b11b731/section/5c7522daf265da2de165acc3










