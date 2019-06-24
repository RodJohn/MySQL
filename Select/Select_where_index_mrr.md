

# 多范围读


在次级索引上使用范围扫描读取行可能会导致在表格较大并且未存储在存储引擎的高速缓存中时对基表进行多次随机磁盘访问。通过磁盘扫描多范围读取（MRR）优化，MySQL尝试通过首先扫描索引并收集相关行的密钥来减少范围扫描的随机磁盘访问次数。然后对密钥进行排序，最后使用主键的顺序从基表检索行。磁盘扫描MRR的动机是减少随机磁盘访问的次数，从而对基表数据进行更顺序的扫描。

使用MRR时，EXPLAIN输出中的Extra列显示使用MRR。

一、什么是MRR
MMR全称是Multi-Range Read，是MYSQL5.6优化器的一个新特性，在MariaDB5.5也有这个特性。优化的功能在使用二级索引做范围扫描的过程中减少磁盘随机IO和减少主键索引的访问次数。将随机IO转换为顺序IO

而MRR的优化在于，并不是每次通过辅助索引就回表去取记录，而是将其rowid给缓存起来，然后对rowid进行排序后，再去访问记录，这样就能将随机I/O转化为顺序I/O，从而大幅地提升性能


使用限制
MRR 适用于range、ref、eq_ref的查询



# 适用场景
#辅助索引key_part1，查询key_part1在1000到2000范围内的数据

SELECT * FROM t WHERE key_part1 >= 1000 AND key_part1 < 2000

不使用MRR：先通过二级索引的key_part1字段取出满足条件的key_part1,pk_col order by key_part1.然后通过pk_col去表中取出满足条件的数据，此时，因为取出的pk_col是乱序的，而表又是pk_col存放数据的，当去表中取数据时，则会产生大量的随机IO

使用MRR：先通过二级索引的key_part1字段取出满足条件的key_part1,pk_col order by key_part1.放到缓存中（read_rnd_buffer_size），当对应的缓冲满了以后，将这部分key值按照pk_col排序，最后再按照排序后的reset去取表中数据，此时pk_col1是顺序的，将随机IO转化为顺序IO，多页数据记录可一次性读入或根据此次的主键范围分次读入，以减少IO操作，提高查询效率




