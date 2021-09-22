# 【MySQL】日志



`Mysql`的更新流程会涉及到两个重要日志模块：`redo log`（重做日志）和`bin log`（归档日志）

## `Redo log`

`redo log`是`InnoDB`引擎特有的日志，记录的是数据页的修改操作。日志大小固定，并以循环的方式写入.即从头开始写，写到末尾，然后重新回到开头。

### `crash-safe`

有了`redo log`，即使数据库异常重启，之前提交的记录也不会丢失，这种能力称为`crash-safe`。

### `WAL`技术

`WAL`全称是`Write-Ahead Logging`，是指先写日志，再更新磁盘的数据页。

如果每次更新的数据都直接写到磁盘，那么磁盘每次都需要查找对应记录，并且更新，`io`成本和查找成本太高。因为写入日志时，使用的是顺序写，速度较快，所以先写日志的效率更高。

### 执行流程

- 当有一条记录更新，`InnoDB`引擎会先把记录写到`redo log`，并更新内存。

- 系统比较空闲时，`InnoDB`引擎会将操作记录更新到磁盘。

- 如果日志文件写满了，也会将一部分操作更新到磁盘。

### `innodb_flush_log_at_trx_commit`

`innodb_flush_log_at_trx_commit`参数用于控制`redo log`的写入策略。

- 0: 每隔1秒，将`redo log buffer`中的日志刷新到磁盘。性能最好，会丢日志。

- 1: 每次提交事务时，将`redo log`日志全部刷新到磁盘。最慢。

- 2: 每次提交事务时，先将`redo log`日志写到操作系统的文件缓存，再每秒刷新到磁盘。操作系统崩溃时会丢日志。

参数为0时，可能会将未提交事务的`redo log`持久化到磁盘。

`redo log buffer`占用空间达到`innodb_log_buffer_size`一半的时候，后台线程会将`redo log`写入操作系统的文件缓存。

如果参数为1，事务A提交时，也会将事务B的`redo log`持久化到磁盘，即使事务B未提交。`redo log`在`prepare`阶段会持久化到磁盘，在`commit`阶段只会刷新到操作系统的文件缓存。


## `bin log`

`bin log`是`Server`层的日志，主要用于归档。

### `bin log`日志格式

使用如下`SQL`语句可以查看`bin log`的日志格式：

```sql
SHOW VARIABLES LIKE "binlog_format";
```

或者：

```sql
SELECT @@binlog_format;
```

- `statement`
  
  设置为该格式时，日志文件中记录的是`SQL`语句。
  
  对数据进行增、删、改的`SQL`语句，都会被记录在日志文件中，可以使用`mysqlbinlog`查看每条语句的内容。

- `row`
  
  设置为该日志格式时，日志记录的是每一行的数据变更，而不是`SQL`语句本身。

- `mixed`
  
  `mixed`混合了`statement`和`row`两种格式。默认情况采用`statement`格式，但是在某些情况下，会使用`row`来记录日志。

### 归档流程

以下介绍误删表时，`bin log`的归档流程：

- 找到最近的一次全备份，恢复到临时库。

- 从全备份的时间点开始，将`bin log`依次取出来，恢复到误删表之前的时刻。

- 从临时库中取出表数据，按需恢复到线上库。

### `binlog`的写入机制

写入机制为：

1. 事务执行过程中，先把日志写到`binlog cache`。
2. 事务提交时，将`binlog cache`中的完整事务写入到`binlog`文件中。
3. 清空`binlog cache`。

每个连接线程都有一个`binlog cache`，所占内存大小由`binlog_cache_size`控制。

所有的线程共用一个`binlog`文件，将内存数据持久化到`binlog`文件时，需要先写入到操作系统的文件缓存，然后再写入磁盘。

### `sync_binlog`

`sync_binlog`参数控制着日志写入到磁盘的频率：

- 0：每次提交事务时，只将日志写到文件缓存。

- 1：每提交一次事务，就将日志刷新到磁盘。最安全但性能最低。

- n：每次提交事务，都将日志写入文件缓存。提交`n`次事务后，将缓存中的日志刷新到磁盘。


## 二者的差异

1. `redo log`是`InnoDB`引擎特有的；`bin log`是`Server`层实现的，所有引擎都可用。

2. `redo log`是物理日志，记录了**在某个数据页上做了什么修改**；`bin log`是逻辑日志，记录的是**语句的原始逻辑**。

3. `redo log`是循环写，`bin log`是追加写。

4. `redo log`用于`crash-safe`，`bin log`用于归档。


## 二者的联系

- 在崩溃恢复的场景中，`InnoDB`如果判断一个数据页丢失了更新，会去读`redo log`。

- 如果`redo log`既有`prepare`状态，又有`commit`状态，则直接提交该事务。

- 如果只有`commit`状态，就从`redo log`中获取`XID`，然后去`bin log`查找对应事务。


## 更新时的日志状态

记录更新时的状态变化如下：

- 执行器找引擎取指定行数据。如果数据页位于内存，就直接返回给执行器；如果不在内存中，则先从磁盘加载再返回。

- 执行器拿到数据进行修改，调用引擎接口写入这行新数据。

- 引擎将新数据记录到内存，将更新操作记录到`redo log`，将`redo log`设为`prepare`状态，告知执行器执行完成。

- 执行器生成`bin log`并视情况写入磁盘。

- 调用引擎接口提交事务，引擎将`redo log`改为`commit`状态，流程结束。


## 两阶段提交

更新过程中使用的是两阶段提交，即先将`redo log`设为`prepare`状态，再生成`bin log`，最后将`redo log`设为`commit`状态。目的是为了让两份日志之间的逻辑一致。

如果不用两阶段提交，可能会导致主从数据库、或者主备数据库逻辑上的不一致：

- 如果先写`redo log`再写`bin log`，写`redo log`之后、`bin log`之前，系统`crash`，用`bin log`恢复出来的数据会比原库少。

- 如果先写`bin log`再写`redo log`，写`bin log`之后、`redo log`之前，系统`crash`，用`bin log`恢复出来的数据会比原库多。


