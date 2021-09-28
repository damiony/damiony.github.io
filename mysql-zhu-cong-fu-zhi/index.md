# 【MySQL】主从复制


## 主备架构

`MySQL`的高可用架构是从一主一备的的基础上演化过来的。

所谓的主备架构，即客户端从主节点读写数据，备节点是主节点的备份，和主节点的数据保持同步。进行切换时，备库被切成主库，主库被切成备库。

### readonly

通常情况下，会将备库设置为`readonly`模式。原因如下：

- 备库常被用于执行一些分析语句，设为只读可防止误操作。

- 防止切换过程中出现双写，进而导致主备不一致。

- 常用`readonly`状态来判断节点的角色。

注意，`readonly`配置对超级权限用户无效，进行同步的线程拥有超级权限。

### 同步流程

主备同步的流程如下：

1. 备库使用`change master`命令，设置主库的`IP`、端口、用户名、密码、`binlog`文件名和日志偏移量。

2. 在备库执行`start slave`命令。之后备库会启动两类线程，`io_thread`和`sql_thread`。

3. `sql_thread`和主库建立连接。

4. 主库校验完用户名和密码后，根据备库指定的`binlog`文件名和偏移量，从本地读取日志，然后发给备库。

5. 备库拿到`binlog`后，写到本地中转日志`relay log`。

6. `sql_thread`读取中转日志，然后进行解析执行。


## change master

配置主从复制的时候，需要在从库上执行`change master`语句，才能和主库保持同步。

### 语法

`change master`语法为：

```sql
change master to option [, option] ...
```

### 常见选项

常见的`option`参数如下：

- `MASTER_HOST`：主库的主机名或者`ip`。

- `MASTER_PORT`：主库的端口。

- `MASTER_USER`：连接到主库的用户名。

- `MASTER_PASSWORD`：连接到主库的密码。

- `MASTER_LOG_FILE`：指定主库的`binlog`文件位置。

- `MASTER_LOG_POS`：指定从`binlog`的该位置开始复制。

### 示例

```sql
change master to
master_host="127.0.0.1",
master_port=3306,
master_user="root",
master_password="123456";
```


## 查看 binlog

1. 查看`binlog`是否开启。

```sql
show variables like "log_bin%";
```

2. 查看第一个`binlog`内容。

```sql
show binlog events;
```

3. 查看`binlog`文件列表。

```sql
show binary logs;
```

4. 查看正在使用的`binlog`。

```sql
show master status;
```

5. 查看指定`binlog`内容。

```sql
show binlog events in 文件名;
```


## binlog 格式

`binlog`有三种格式，`statement`、`row`和`mixed`。

### statement

`binlog_format`设置为`statement`时，`binlog`里面记录的是`SQL`语句本身。

使用`statement`可能会导致主备数据不一致。因为日志记录的是`sql`语句，因此主库在执行时使用的索引，可能和备库执行时使用的索引不一样，从而可能会造成数据不一致。

### row

`row`格式的`binlog`记录的不是语句原文，而是“操作的哪张表”和“对该表进行的操作”。如下示例中的`Table_map`和`Delete_rows`就是被记录的`event`。

```sql
| binlog.000040 |  156 | Anonymous_Gtid | 1 | 235 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS' |
| binlog.000040 |  235 | Query          | 1 | 310 | BEGIN                                |
| binlog.000040 |  310 | Table_map      | 1 | 366 | table_id: 88 (test.t1)               |
| binlog.000040 |  366 | Delete_rows    | 1 | 406 | table_id: 88 flags: STMT_END_F       |
| binlog.000040 |  406 | Xid            | 1 | 437 | COMMIT /* xid=14 */                  |
```

如果想看到日志的详细内容，可以在终端使用如下命令：

```shell
mysqlbinlog -vv 日志路径 -start-position=310 --stop-position=437
```

使用`row`格式，不会造成主备数据不一致。


### mixed

如果使用`statement`，可能会导致主备不一致。如果使用`row`，生成的日志会占用大量的空间，消耗`io`资源，对语句性能造成影响。

由于上述的种种原因，从而有了`mixed`格式。

设置成`mixed`格式时，`MySQL`会判断语句是否会引起主备不一致。如果可能会，就用`row`格式，否则用`statement`格式。

### 数据恢复

因操作不当，导致数据出现错误时，还可以使用`row`格式的`binlog`将数据恢复到之前的状态。

### binlog 重放

`binlog`的一个重要使用场景是归档。使用某个时间点的备份，和该时间点之后的`binlog`日志，可以让数据恢复到指定时刻。

需要注意的是，`binlog`重放时，不能直接执行`statement`格式记录的语句，因为有些语句是依赖上下文环境的，如时间等。正确做法是，使用`mysqlbinlog`工具解析，然后将整个解析结果发给`MySQL`执行。

示例如下：

```shell
mysqlbinlog binlog.000040 --start-position=4 | mysql -uroot -p
```

## 双主架构

在双主架构中，客户端仍旧从单个节点读写数据，另一个节点与该节点保持数据同步，但是节点之间互为主备关系。在这种架构中，如果发生了切换，就不再需要修改主备关系。

如果将参数`log_slave_updates`设置为`on`，备库执行完`relay log`后，会生成`bin log`。

由于节点之间互为主备，因此可能会产生`bin log`循环复制的问题。为了解决这个问题，节点的`server id`必须互不相同。

备库收到`bin log`并且重放后，会生成新的`bin log`，新`bin log`的`server id`和原来`bin log`的`server id`一致。

备库收到主库发来的`bin log`时，会判断`server id`是否和自己的相等。如果相等，就表示这个日志是自己生成的，需要丢弃。

