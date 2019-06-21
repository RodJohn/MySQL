

# 格式

redo日志本质上只是记录了一下事务对数据库做了哪些修改


最简

存储表空间ID、页号、具体更改

redo日志中只需要记录一下在某个页面的某个偏移量处修改了几个字节的值，具体被修改的内容是啥就好了，
设计InnoDB的大叔把这种极其简单的redo日志称之为物理日志



复杂

比如插入

把一条记录插入到一个页面时需要更改的地方非常多

表中包含多少个索引，一条INSERT语句就可能更新多少棵B+树。

Page Directory


还存在 页分裂


组

所谓的页分裂操作，也就是新建一个叶子节点，然后把原先数据页中的一部分记录复制到这个新的数据页中，然后再把记录插入进去，把这个叶子节点插入到叶子节点链表中，最后还要在内节点中添加一条目录项记录指向这个新创建的页面。很显然，这个过程要对多个页面进行修改，也就意味着会产生多条redo日志，我们把这种情况称之为悲观插入

Mini-Transaction，

简称mtr，比如上边所说的修改一次Max Row ID的值算是一个Mini-Transaction，向某个索引对应的B+树中插入一条记录的过程也算是一个Mini-Transaction。
通过上边的叙述我们也知道，一个所谓的mtr可以包含一组redo日志，在进行奔溃恢复时这一组redo日志作为一个不可分割的整体

log buffer中写入redo日志时不是一条一条写入的，而是以一个mtr生成的一组redo日志为单位进行写入的。

LSN

每一组由mtr生成的redo日志都有一个唯一的LSN值与其对应，LSN值越小，说明redo日志产生的越早。



# 

优化insert
顺序插入

不使用物理 delete 

行格式 紧凑 改新型 
变长字符
溢出数据  
	














