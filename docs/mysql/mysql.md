mysql

实操：
show open tables WHERE In_use >0; 查看是否锁表
show processlist;           //查看链接
kill processlist中的id        //kill链接
SELECT * from INFORMATION_SCHEMA.INNODB_LOCKS; //查看正在锁的事务
SELECT * from INFORMATION_SCHEMA.INNODB_LOCK_WAITS; //查看正在锁等待的事务

select 
table_schema as '数据库',
table_name as '表名',
table_rows as '记录数',
truncate(data_length/1024/1024, 2) as '数据容量(MB)',
truncate(index_length/1024/1024, 2) as '索引容量(MB)'
from information_schema.tables
WHERE table_name like 'print_bg_kdd_%'
order by data_length desc, index_length desc;


Mysql的utf8不是真正意义上的utf8，它只支持每个字符最多三个字节，而真正的UTF8每个字符最多四个字节。而utf8mb4才是真正的utf8.惊不惊喜，意不意外，祖传代码改不的，只好新增字符集兼容喽。
alter table test_flush_log CONVERT to character set utf8mb4 COLLATE utf8mb4_unicode_ci;

select @@global.tx_isolation,@@tx_isolation;  5.7.2以下。

//设置read uncommitted级别：
set session transaction isolation level read uncommitted;

//设置read committed级别：
set session transaction isolation level read committed;

//设置repeatable read级别：
set session transaction isolation level repeatable read;

//设置serializable级别：
set session transaction isolation level serializable;


ALTER TABLE  表名 DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;



脏读
读到了其他事务未提交的数据。通过行锁解决。

不可重复读
读到了本事务逻辑之后的事务提交的数据。
同一事务同一select两次执行的结果不一样。通过MVCC解决。

幻读
同一事务两次count的结果不一样。通过gap-lock+行锁解决。这里update和insert可能出现死锁。


隔离级别：
read uncommited. 		上面三种问题都无法解决
read committed     		解决了脏读
repeatable-read     		解决了脏读和不可重复读 部分解决了幻读(快照读不会幻读，当前读会出现幻读)		mvcc 当前读有nest-key锁
serializiable			能解决以上所有问题，但性能很差，锁竞争消耗大。


 show table status like 'table_name'

opt的本质是新建表，然后插入旧数据，然后删除旧表，这个过程很很长，需要注意对业务的影响。

show  table status like ‘…’
show index from table_name，列值Cardinality表明大致估算有多少个不同值

覆盖索引：就是索引中已经包含了查询的值
索引失效：
1、or
2、不符合最左
3、效率不如全表
4、隐式格式转换

MVCC：
1、每个事务begin或者start时(以下简称开启)，都会申请一个递增的trx_id。
2、每行记录版本有三个隐藏列DB_TRX_ID、DB_ROLL_PTR、DB_ROW_ID。分别表示当前版本的事务ID、上个版本的指针、主键id。
3、每个事务开启时，会有一个活跃事务数组，存储着当前事务的trx_id之前的所有活跃trx_id。比如事务A开启之前有一个trx_id为99的活跃事务。那么事务A的事务数组则是[99,100],事务B的数组则是[99,100,101]。以此类推。数组中的最小事务id成为低水位，最大事务id(也就是自己的id)为高水位
4、可见行规则如下：
	1、如果当前行的trx_id是比当前trx_id小的且不在数组里，则可见。
	2、如果当前行的trx_id是高水位大的，不可见
	3、如果当前行的trx_id是在活跃数组中的，除了自身的id，不可见
5、更新语句都是当前读，如果是当前行的trx_id比当前事务大，但已提交的，那么会是可见的，也就是update是以这个提交后的数据为准，是的。

在可重复读下，事务开始时创建一致性视图，之后其他查询共用这个视图。
在读提交下，每条语句执行前都重新计算出一个视图。

一致性读：
可重复读只读事务开启前的提交的数据
读已提交只读执行语句启动前提交的数据

当前读：总是读取已提交的最新版本


关联表最好用业务件作为关联主键
1、迁移数据时不会因为自增主键而出现数据不一致
2、很多业务场景可以不回表，提高性能



innodb的事务原理：
利用buffer pool 、log buffer、redo log、undo log.
以update为例，
1、先从buffer pool中读取到数据，然后修改，更新到buffer pool
2、记录一条redo log和undo log到log buffer
3、写bin log
4、事物提交，则将数据持久化，redo log持久化。详见刷盘策略
5、事物回滚或异常，则执行undo log.
涉及参数：
innodb_flush_log_at_trx_commit：redolog的刷盘策略 0是mysql线程每秒刷盘 1是每次都提交都刷盘 2是写入OS cache OS进程每秒刷盘 建议为2或1
sync_binlog：bin log的刷盘策略 0是写入操作系统缓存，让操作系统决定什么时候刷盘  n 每n次提交后直接刷盘。 建议为1
￼![三次握手](../img/mysql_query_full.png)
￼![三次握手](../img/mysql_two_commit.png)


￼

参数浏览：

innodb_autoinc_lock_mode：
插入分为三种：
1、simple insert 如insert into t(name) values(‘‘)
2、bulk insert 如load data | insert into ... select .... from ....
3、mixed insert 如insert into t(id,name) values(1,'a'),(null,'b'),(5,'c');
取值：
0:传统的模式，每次语句执行获得一个表级的自增锁。
1:默认 一次性获得多个连续值。获取是需要上述锁，获取完释放，然后再执行插入。
2:无锁 并发最高
binlog为row时，设置为2。性能最高，且安全，不会发生主从不一致问题。
binlog为statement时，设置为1。

索引下推：
5.2.6之前是带着满足索引的所有数据(包含不符合其他where的)去回表查询。5.2.6之后，会先把不符合其他where条件的数据过滤掉之后再去回表，减少回表次数。

主从同步：
主库将日志写入binlog,从库连接到主库之后，从库有个IO线程，不断同步主库的binlog到本地的relay中继日志里。从库的另一个SQL线程从中继日志中读取binlog执行
￼
由于主库是并发执行，从库执行是串行，所以从库的同步是有时延的。
- [ ] 那如果主库宕机，从库还没来得及同步，存在丢失问题，怎么解决？
问题根源是从库同步有执行效率差+网络不稳定
1、半同步复制
主库写入binlog之后，强制将数据传给从库，直到一个从库返回ACK之后才认为写完了。影响写的性能，但能解决丢失问题
2、并行复制
从库开启多个线程并行读取relay日志中不同库的日志，属于库级别的并行。但这也只是缓解丢失问题，如果主库命令还没到达relay日志就挂了，那还是会有丢失问题。


DDL:数据定义(definition)语句   新建表、新增字段
DML:数据操作语言 新增、修改、删除数据等
MDL:metadata lock 表级锁

online DDL
1、先获取MDL写锁
2、降级成MDL读锁
3、execute DDL
4、升级成MDL写锁
5、释放MDL锁
































