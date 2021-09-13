binlog（二进制日志）：

二进制日志（binary log）记录了对mysql数据库执行更改的所有操作，不包含select， show这种操作，如果一个操作本身没有导致数据库发生改变，那么该操作不会写入到二进制文件中。

binlog的作用：

> 1. 恢复（recovery）：一些数据的恢复需要二进制日志，在一个数据库全备文件恢复后，用户可以通过二进制文件进行point-in-time的恢复。
> 2. 复制（replication）：通过复制与执行二进制日志使一台远程的mysql数据库（slave）与一台mysql数据库（master）进行实时同步。
> 3. 审计（audit）：可以通过二进制日志中的信息来进行审计，判断是否有对数据库进行注入的攻击。

查看binlog日志是否已经开启：

```sql
show variables like '%log_bin%'；
```

如果未开启：

![image-20201031001233498](http://akatsuke.com/image-20201031001233498.png)

开启binlog需要在配置文件中加入以下命令：

```sql
log-bin=mysql
server-id=1
```

成功开启的时候：

![image-20201031004023255](http://akatsuke.com/image-20201031004023255.png)

查看binlog的日志文件：

![image-20201031004447799](http://akatsuke.com/image-20201031004447799.png)

可以通过命令行查看binlog的记录：

```sql
show binlog events;
```

![image-20201031004728605](http://akatsuke.com/image-20201031004728605.png)



binlog的格式：

1. row格式
2. statement格式
3. mixed格式

mysql是如何知道binlog是完整的：

1. 在statement格式的binlog的情况下， 最后会有commit
2. 在row格式的binlog的情况下，最后会有一个xid event

在 MySQL 5.6.2 版本以后，还引入了 binlog-checksum 参数，用来验证 binlog 内容的正确性。对于 binlog 日志由于磁盘原因，可能会在日志中间出错的情况，MySQL 可以通过校验 checksum 的结果来发现。



redo log和binlog如何关联的：

redo log和bin log都有一个相同的字段XID，在崩溃恢复 的时候，会按照顺序先找redo log，如果relog的日志中的事务既有prepare又有commit则直接提交事务，，但是如果遇到只有prepare没有commit的事务的情况的时候，会拿着XID去binlog中找对应的事务，这个过程是这样的：

1. 如果binlog存在且完整，则提交那个redo log中只有prepare状态的事务
2. 如果binlog中不存在这个事务，则这条事务回滚