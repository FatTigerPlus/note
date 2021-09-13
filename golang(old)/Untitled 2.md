1. mysql：索引，锁，事务，日志，mvcc，优化，分库分表，主从， 事务隔离级别
2. redis：数据类型，LRU，LFU， 哨兵，集群，持久化（AOF和RDB），一致性相关
3. 网络：HTTP, UDP,TCP,HTTPS,IP,ICMP,ARP
4. OS：内存，存储，文件，线程进程（多线程多进程通讯，进程的调度和中断），寄存器，缓存，CPU，缓冲，IO多路复用，COW，零拷贝
5. 语言：基础语言相关问题，多线程相关，GMP， 多线程同步， 锁机制，源码层面，runtime，GC，GMM
6. 框架：设计和源码
7. 容器：
8. 算法

```go
mysql相关考点
1.索引：B+树，为什么选择树，他与hash和二叉平衡搜索树的相比有什么优势？
2.innodb和myisam的区别？
3.explian相关字段
4.事务的隔离性有几种？mysql的默认事务隔离级别？
5.redo，undo，binlog的区别？作用？
6.mysql查询语句和更新语句的执行的流程？
7.buffer pool？
8.表锁，行锁，库锁？悲观锁和乐观锁？线段锁等？
9.主从架构？sql_thread，io_thread?
10：mvcc原理
```

```go
redis相关考点：
1：redis的常见的数据类型
2：redis常见的数据类型的底层实现
3.持久化：AOF和RDB，
4.哨兵搭建，选举，心跳检测
5.LRU,LFU
```

```go
网络相关：
http：
1.http是啥
2.http常用状态码，http报文格式
3.cookie是啥，有啥用
4.什么是无状态协议
5.header相关
6.跨域问题
7.nginx和apache的区别，为什么使用nginx，nginx的优势何在
8.http各个版本的区别
9.常用的http的方式，，restful风格api
10.用户在浏览器输入url的一整个的操作流程（网络层面）


tcp：
1.四元组
2.三次握手四次挥手（为什么握手是三次，，挥手 是四次）
3.MTU
4.报文格式，每个字段的意思
5.tcp的几个flags分别代表什么
6. tcp的状态机
7.SYN攻击和半连接队列
8. tcp重试机制
9. 滑动窗口，流量控制
10.tcp和udp的区别
```

```go
go相关：
1.slice， map， channel，context
2.GMP调度
3.GMM，指令重排
```

