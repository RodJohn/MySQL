
# 

    考虑innodb
    
    
# 特点    

# 无锁读


    普通的SELECT语句

    READ UNCOMMITTED隔离级别下，
        不加锁，直接读取记录的最新版本，可能发生脏读、不可重复读和幻读问题。
    READ COMMITTED隔离级别下，
        不加锁，在每次执行普通的SELECT语句时都会生成一个ReadView，这样解决了脏读问题，但没有解决不可重复读和幻读问题。
    REPEATABLE READ隔离级别下，
        不加锁，只在第一次执行普通的SELECT语句时生成一个ReadView，这样把脏读、不可重复读和幻读问题都解决了。    
    SERIALIZABLE隔离级别下

# 锁定读

## 语句
    
    锁定读的语句
    我们把下边四种语句放到一起讨论：
    
    语句一：SELECT ... LOCK IN SHARE MODE;
    
    语句二：SELECT ... FOR UPDATE;
    
    语句三：UPDATE ...
    
    语句四：DELETE ...
    
    我们说语句一和语句二是MySQL中规定的两种锁定读的语法格式，
    而语句三和语句四由于在执行过程需要首先定位到被改动的记录并给记录加锁，也可以被认为是一种锁定读。

RU
RC

RR


## 全表扫描

RU
RC

    存储引擎每读取一条聚簇索引记录，
    就会为这条记录加锁一个S型next-key锁，然后返回给server层，
    如果server层判断条件成立则将其发送给客户端，否则会向InnoDB存储引擎发送释放掉该记录上的锁的消息，

RR

    存储引擎每读取一条聚簇索引记录，
    就会为这条记录加锁一个S型next-key锁，然后返回给server层，
    如果server层判断条件成立则将其发送给客户端，否则会向InnoDB存储引擎发送释放掉该记录上的锁的消息，
    不过在REPEATABLE READ隔离级别下，InnoDB存储引擎并不会真正的释放掉锁，所以聚簇索引的全部记录都会被加锁，并且在事务提交前不释放
    
示例

    
    


## 用主键进行等值查询


    
RC   

    加行锁

RR

    存在 
    加行锁
    
    但是如果我们要查询主键值不存在的记录
    间隙加锁 
    误伤
    
        
    
##  主键范围查询

RC

    满足条件的记录加行锁

RR

    SELECT * FROM hero WHERE number >= 8 LOCK IN SHARE MODE;
    
    因为要解决幻读问题，所以需要禁止别的事务插入number值符合number >= 8的记录，
    又因为主键本身就是唯一的，所以我们不用担心在number值为8的前边有新记录插入，只需要保证不要让新记录插入到number值为8的后边就好了，所以：
    
    为number值为8的聚簇索引记录加一个S型正经记录锁。
    为number值大于8的所有聚簇索引记录都加一个S型next-key锁（包括Supremum伪记录）。
    

## 普通二级索引等值查询



RC

    先通过二级索引idx_name定位到满足name = 'c曹操'条件的二级索引记录，然后进行回表操作。
    所以先要对二级索引记录加S型正经记录锁，然后再给对应的聚簇索引记录加S型正经记录锁，

RR



# 参考

    https://mp.weixin.qq.com/s/wSlNZcQkax-2KZCNEHOYLA
    https://mp.weixin.qq.com/s/ODbju9fjB5QFEN8IIYp__A
    https://mp.weixin.qq.com/s/9WWBXLNoUcTkS4DJnM5ViA
    
    http://hedengcheng.com/?p=771#_Toc374698307

                 