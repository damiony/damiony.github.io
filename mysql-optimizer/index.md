# 【MySQL】优化


## SQL性能问题

### 1. 说明

- 主要是通过分析SQL执行计划，分析性能。

- 注意，SQL查询优化器会干扰我们的优化。


## SQL执行计划

### 1. 语法

```mysql
explain SQL语句
```


## 执行计划分析

### 1. id

### 2. select_type

- `Primary`：包含子查询的SQL中的主查询（最外层）。

- `Subquery`：包含子查询的SQL中的子查询（非最外层）。

- `Simple`：简单查询（不包含子查询、union）。

- `derived`：衍生查询（使用到了临时表）；如在from子查询中只有一张表；又或者在from子查询中，如果有table1 union table2，则table1就是derived，table2就是union。

- `union`：说明见上例。

- `union result`：告知哪些表之间存在union查询。

### 3. table

### 4. type

`System > const > eq_ref > ref > range > index > all`

其中：`system`，`const`只是理想情况，实际能达到 `ref > range`。

优化前提：有索引。

- `system`（忽略）

  只有一条数据的系统表；或衍生表只有一条数据的主查询。

- `const`

  仅仅能查到一条数据的`SQL`，用于`primary key`或者`unique`索引（类型与索引类型有关）。

- `eq_ref`

  唯一性索引；对于每个索引键的查询，返回匹配的唯一行数据（有且只有1个，不能为0）；常见于唯一索引和主键索引。

- `ref`

  非唯一性索引，对于每个索引键的查询，返回匹配的所有行。

- `range`

  检索指定范围的行。

- `index`

  查询全部索引表中的数据。

- `all`

  查询表中的所有数据。

### 5. Possible_keys

可能用到的索引，是一种预测。

### 6. key

实际用到的索引。

### 7. key_len

索引的长度。

用于判断复合索引是否被完全使用。

对于`utf8`，一个字符占3个字节。

如果索引字段可以为`null`，则会使用1个字节用于标识。

如果索引字段为`varchar`，使用2个字节标识可变长度。

### 8. ref

注意与`type`中的`ref`值区分。

作用：指明当前表所参照的字段。

如：`select ... where a.c = b.x;`

### 9. rows

被索引优化查询的数据个数（实际通过索引而查询到的数据个数）。

### 10. extra

- `using filesort`

  性能消耗大；需要“额外”一次排序（查询）。常见于`order by`语句中。

  示例1：

  ```mysql
  explain select * from test where a1='' order by a2;
  ```

  小结：对于单索引，如果排序和查找是同一个字段，则不会出现`using filesort`；如果排序和查找不是同一个字段，则会出现`using filesort`。

  示例2：（注意存在a1, a2, a3, a4四个字段）

  ```mysql
  alter table test add index idx_a1_a2_a3 (a1, a2, a3);
  explain select * from test where a1='' order by a3; # using filesort
  explain select * from test where a2='' order by a3; # using filesort
  explain select * from test where a1='' order by a2;
  ```

  小结：对于复合索引，`where`和`order by`按照复合索引的顺序使用，不要跨列或无序使用。

- `using temporary`

  性能损耗大，用到了临时表。一般出现在`group by`语句中。

  示例：

  ```mysql
  explain select * from test where a1 in (1, 2, 3) group by a1;
  explain select * from test where a1 in (1, 2, 3) group by a2; # using temporary
  ```

- `using index`

  性能提升，常见于索引覆盖。

  只要使用到的列，全部都在索引中，就称为”索引覆盖“。

  不读取原文件，只从索引文件中获取数据（不需要回表查询）。
  
  索引覆盖会对`possible_keys`和`key`造成影响：如果没有`where`，则索引只出现在`key`中；如果有`where`，则索引可能出现在`key`和`possible_keys`中。
  
- `using where`

  回表查询时，会出现`using where`。

- `impossible where`

  `where`子句永远为`false`。
  
- `using join buffer`

  `mysql`引擎使用了连接缓存。


## SQL优化

SQL优化是一种概率层面的优化。

服务层有SQL优化器，可能会影响我们的优化。


## 单表优化

1. 最佳左前缀，保持索引的定义和使用的顺序一致性
2. 索引要逐步优化
3. 将含in的范围查询，放到where的最后，防止失效


## 双表优化

1. 小表驱动大表
2. 一般情况下，左外连接，给右表加索引；右外连接，给左表加索引。


## 三表优化

1. 小表驱动大表。
2. 索引建立在经常查询的字段上。


## 避免索引失效的一些原则

1. 复合索引，不要跨列或无序使用（最佳左前缀）。
2. 复合索引，尽量使用全索引匹配。
3. 不要在索引上进行任何操作（计算，函数，类型转换），否则索引失效。
4. 复合索引不能使用”不等于“，否则自身以及右侧所有全部失效。
5. 复合索引中如果有`>`，自身和右侧全失效。
6. 一般而言，范围查询之后的索引失效。
7. 尽量使用覆盖索引。
8. like尽量以“常量”开头，不要以"%"开头，否则索引失效。
9. 尽量不要使用类型转换。
10. 尽量不要使用`or`，否则索引失效。


## 其他的一些优化方法

#### order by

using filesort有两种算法：单路排序、双路排序。

双路排序扫描两次磁盘：从磁盘读取排序字段；读取其他字段。

单路排序扫描一次磁盘：读取全部字段（不一定真的是单次，有可能是多次IO）。

Mysql4.1之前默认使用双路排序；mysql4.1之前默认使用单路排序。

设置buffer的容量大小：`set max_length_for_sort_data = 1024`，单位byte。

如果`max_length_for_sort_data`值太低，mysql会自动从 单路切换成双路。

避免使用`select * ...`

复合索引不要跨列使用。

保证排序字段的排序一致性。


## 慢查询

mysql提供的一种日志记录，用于记录mysql响应超过阀值的语句，默认10秒。

默认是关闭的；建议调优时打开，最终部署时关闭。

检查是否打开了慢查询日志：

```mysql
show variables like '%slow_query_log%';
```

临时开启：

```mysql
set global slow_query_log = 1;
```

mysql服务器重启后会失效。

永久开启：

```bash
#/etc/my.cnf追加
[mysqld]
slow_query_log=1
slow_query_log_file=/var/lib/mysql/local-slow.log
```

慢查询阀值：

```mysql
show variables like "%log_query_time%";
```

设置慢查询阀值：

```mysql
set global lomg_query_time=5;
```

重新登陆后生效。

永久设置阀值：

```bash
#在/etc/my.cnf追加
[mysqld]
long_query_time=3
```

查询超过阀值的sql：

```mysql
show gloval status like '%slow_queries%';
```


## mysqldumpslow

用于查看慢SQL。


## 海量数据分析

#### 1. profiles

查看开启状态

```mysql
show variables like '%profiling%';
```

打开

```mysql
set profiling=on;
```

查看profiling打开之后，全部sql查询语句所花时间，不够精确

```mysql
show profiles;
```

#### 2. 精确分析

```mysql
show profile all for query 上一步查询的query_id;
```

或者

```mysql
show profile cpu, block io for query 上一步查询的query_id;
```

#### 3. 全局查询日志

记录开启之后的全部sql语句，仅仅在调优、开发过程中打开。

```mysql
show variables like '%general_log%'; # 查看状态

#记录在表中
set global general_log=1; # 开启
set global log_output='table'; # sql记录在表中

#或者记录在文件中
set global general_log=1; # 开启
set global log_output='file'; # 或者sql记录在文件中
set global general_log_file='/tmp/general.log'; # 或者sql记录在文件中
```


## 锁机制

解决因资源共享，造成的并发问题。

#### 分类

按照操作类型：

- 读锁（共享锁）：对同一个数据，多个读操作可以同时进行，互不干扰。

- 写锁（互斥锁）：如果当前写操作没有完毕，则无法进行其他的读、写。

按照操作范围：

- 表锁：一次性对一张表整体加锁。MyISAM使用表锁。开销小，加锁快，无死锁。但锁的范围大，容易发生锁冲突，并发度低。

- 行锁：一次性对一条数据加锁。InnoDB使用行锁。开销大，加锁慢，容易死锁。所得范围较小，不容易发生锁冲突，并发度高，很小概率发生高并发问题（脏读、幻读、不可重复读）。

- 页锁：


## 表锁

#### 给表加锁

```mysql
lock table 表1 read/write, 表2 read/write, ...;
```

#### 查看加锁的表

```mysql
show open tables;
```

#### 释放锁

```mysql
unlock tables;
```

#### 加读锁的机制

如果一个会话对A表加了read锁，则该会话可以对A表进行读操作、不能进行写操作；不能对其它表进行读写操作。其它会话可以对A表进行读操作、进行写操作得等待锁释放；对其他表可以进行读写操作。

#### 加写锁的机制

某个会话对表加写锁，之后可以对这张表进行任何操作，但是不能操作其它表。

其他会话对加写锁的表，进行增删改查时得等待锁释放。

#### MySQL表级锁的锁模式

1. MyISAM在执行查询语句前，会自动给涉及到的所有表加读锁；在执行更新操作前，会自动给涉及的表加写锁。
2. 对MyISAM表的读操作，不会阻塞其他进程（会话）对同一表的读请求，会阻塞对同一表的写请求。
3. 对MyISAm表的写操作，会阻塞其他进程（会话）对同一表的读和写操作，只有当锁释放时，才会进行其它进程的读写操作。

#### 分析表锁定

1. 查看表锁定状态

   ```mysql
   show open tables; # 1代表加了锁
   ```

2. 分析表锁定的严重程度

   ```mysql
   show status like 'table%';
   ```

   - `table_lock_immediate`：可能获取到的锁数。
   - `tale_locks_waited`：需要等待的表锁数，该值越大，锁竞争越大。

   当`table_locks_immediate / table_locks_waited` > 5000，建议采用InnoDB引擎。


## 行锁

InnoDB默认采用行锁。

研究前提：关闭了自动commit

关闭语句：

1. `set autocommit=0`
2. `start transection`
3. `begin`

#### 总结：

1. 如果”会话1“对“某条数据a”进行了DML操作，则其他会话需要等待“会话1”对“数据a”提交事务。
2. 行锁通过事务解锁。
3. 不同会话对不同行数据进行操作，互不影响。
4. 查询数据不会加锁。
5. 可以通过`for update`对`query`语句进行加锁。

#### 注意事项：

1. 如果没有索引，行锁会转换为表锁，需要通过`commit`进行解锁。

#### 行锁的一种特殊情况：

间隙锁：值在范围内，但却不存在。

如：`update ... where id > 1 and id < 9;`，如果`id=7`的数据不存在，则该行数据存在间隙锁。

如果有`where`，则实际加锁的范围是`where`后面的范围（不是实际的值）。

#### 缺点：

比表锁性能损耗大。

#### 优点：

并发能力强，效率高。

#### 行锁分析

1. 查看行锁状态

   ```mysql
   show status like '%innodb_row_lock%'
   ```

   - `Innodb_row_locl_current_waits`：当前正在等待锁的数量。
   - `Innodb_row_lock_time`：等待总时长，从系统启动到现在，一共等待的时间。
   - `Innodb_row_lock_time_avg`：平均等待时长，从系统启动到现在的平均等待时间。
   - `Innodb_row_lock_time_max`：最大等待时长，从系统启动到现在的最大等待时间。
   - `Innodb_row_lock_waits`：从系统启动到现在，等待的次数。



## 主从复制

#### 优点

1. 负载均衡
2. 失败迁移

#### 原理

1. master将改变的数据记录在本地的二进制日志中（binary log）；该过程称为“二进制日志事件”。
2. slave将master的binary log拷贝到relay log（中继日志文件）中。
3. slave将数据从relay log读取到自己的数据库中，称为中继日志事件。

#### 特点

1. 异步
2. 串行话
3. 有延迟

