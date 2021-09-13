php连接mysql8.0x出现问题的时候。可以在my.ini（windows下）或者my.cnf(linux下)添加以下语句：

```sql
[mysqld]
default_authentication_plugin=mysql_native_password
```

查看mysql现在使用的加密方式：

```sql
 SELECT user, authentication_string, plugin, host FROM mysql.user;
```

<img src="http://akatsuke.com/image-20201030221217459.png" alt="image-20201030221217459" style="zoom: 67%;" />

mysql的语句分类：

> 1. DML（Data Definition Languages）：数据定义语言，这些语句定义了不同的数据段、数据库、表、列、索引等数据库对象的定义。常用的语句关键字主要包括 create、drop、alter等。
> 2. DDL（Data Manipulation Language）：数据操纵语句，用于添加、删除、更新和查询数据库记录，并检查数据完整性，常用的语句关键字主要包括 insert、delete、udpate 和select 等。(增添改查）
> 3. DCL（Data Control Language）：数据控制语句，用于控制不同数据段直接的许可和访问级别的语句。这些语句定义了数据库、表、字段、用户的访问权限和安全级别。主要的语句关键字包括 grant、revoke 等。

三范式：

> 1. 每个列都不能再拆分
> 2. 在第一范式的基础上，非主键列完全依赖于主键，且不能依赖于主键的一部分
> 3. 在第二范式的基础上，非主键只依赖于主键，不依赖于其他非主键

### mysql主从复制原理：

> 通过在主库记录二进制日志，在备库重放日志的方式来实现数据的复制。

复制的步骤：

1. 主库把数据更改记录到二进制日志(bin log)中
2. 备库将主库的日志复制到自己的中继日志(relay log)中
3. 备库读取中继日志中的事件，将其重放到备库的数据之上

<img src="http://akatsuke.com/image-20201030220030066.png" alt="image-20201030220030066" style="zoom:67%;" />

创建用户语句;

```go
create user username identified by 'password';
```

## 原则：尽可能使用 not null

非`null`字段的处理要比`null`字段的处理高效些！且不需要判断是否为`null`。

`null`在MySQL中，不好处理，存储需要额外空间，运算也需要特殊的运算符。如`select null = null`和`select null <> null`（`<>`为不等号）有着同样的结果，只能通过`is null`和`is not null`来判断字段是否为`null`。

如何存储？MySQL中每条记录都需要额外的存储空间，表示每个字段是否为`null`。因此通常使用特殊的数据进行占位，比如`int not null default 0`、`string not null default ‘’`