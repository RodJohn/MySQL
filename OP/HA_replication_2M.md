# 双主复制

但实际生产上使用比较多的是双M结构

两台Master都可以读写，互为主备，
采用Keepalive等方案实现高可用（使用VIP对外提供服务），
默认只有一台（MasterA）负责数据写入，另一台（MasterB）备用。

节点A和B之间总是互为主备关系。这样在切换的时候就不用再修改主备关系。


优点

双主复制解决了主从复制的单点写故障问题，可以一定程度保障Master的高可用，
一台Master宕机后，可以在极短时间内自动切换到另一台Master。

通常会和proxy、keepalived等第三方软件同时使用，
即可以用来监控数据库的健康，又可以执行一系列管理命令。
如果主库发生故障，切换到备库后仍然可以继续使用数据库。

缺点：

资源利用率只有50%，高可用就是通过冗余实现，二者不可兼得
两台Master双写同步，数据可能冲突（如自增id同步冲突），需要解决冲突




