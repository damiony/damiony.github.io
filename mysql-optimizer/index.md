# 【MySQL】优化



## SQL优化

主要是通过分析SQL执行计划，分析性能。

服务层的SQL优化器，可能会影响我们的优化。


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


## 慢查询日志

慢查询日志可以记录响应时间超过阀值的`SQL`语句，默认阈值为10秒。

该日志默认关闭，建议调优时打开，最终部署时关闭。

### 1. 检查是否打开了慢查询日志

```sql
SHOW VARIABLES LIKE '%slow_query_log%';
```

### 2. 临时开启

```sql
SET GLOBAL slow_query_log = 1;
```

`MySQL`服务器重启后会失效。

### 3. 永久开启

```bash
#/etc/my.cnf追加
[mysqld]
slow_query_log=1
slow_query_log_file=/var/lib/mysql/local-slow.log # 日志文件路径
```

### 4. 查看慢查询阀值

```sql
SHOW VARIABLES LIKE "%long_query_time%";
```

或者：

```sql
SELECT @@long_query_time;
```

### 5. 设置慢查询阀值

```sql
SET GLOBAL long_query_time=5;
```

重新连接数据库后才生效。

### 6. 永久设置阀值

```bash
#在/etc/my.cnf追加
[mysqld]
long_query_time=3
```

### 7. 查询超过阀值的sql个数

```sql
SHOW GLOBAL STATUS LIKE '%slow_queries%';
```

### 8. mysqldumpslow

可以使用`mysqldumpslow`工具查看慢查询日志，用法为：

```bash
mysqldumpslow 日志文件
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



## 查看服务器状态

可以使用如下命令查看服务器状态信息：

```sql
SHOW [SESSION | GLOBAL] STATUS;
```

如果想查看`SQL`语句执行频率的统计信息，可以使用：

```sql
SHOW [SESSION | GLOBAL] STATUS LIKE "Com_______";
```

如果想查看`InnoDB`数据行的统计信息，可以使用：

```sql
SHOW [SESSION | GLOBAL] STATUS LIKE "InnoDB_rows_%";
```


## 排查低效率语句

排查低效率`SQL`语句，有以下两种方法：

- 慢查询

- `show processlist`


## 执行计划字段分析

获取执行计划的语法为：`EXPLAIN SQL语句`。

### 1. id

`select`查询的序列号，表示查询中执行`select`子句的顺序。

`id`相同时，查询顺序从上往下。

`id`不同时，值越大，越先被执行。

### 2. select_type

`SELECT`的类型。

- `SIMPLE`：简单查询，不包含子查询和`UNION`。

- `PRIMARY`：包含子查询的SQL语句中的主查询（最外层）。

- `SUBQUERY`：在`SELECT子`句或者`WHERE`子句中，包含的子查询。

- `DERIVED`：衍生查询，使用到了临时表。FROM子句中的子查询会被标记为`DERIVED`，又或者`UNION`包含在`FROM`子句的子查询中，外层`SELECT`将会被标记为`DRIVED`。

- `UNION`：UNION中的第二个或者之后的查询语句。

- `UNION RESULT`：对`UNION`结果进行的查询。

### 3. table

查询的表。

### 4. type

表示表的访问类型。

连接类型的性能由好到坏，依次是：`NULL`、`system`、`const`、`eq_ref`、`ref`、`fulltext`、`ref_or_null`、`index_merge`、`unique_subquery`、`index_subquery`、`range`、`index`和`all`。

`system`，`const`很难遇见，最好能达到`ref`和`range`级别。

- `NULL`
  
  不访问任何表，直接返回结果。

- `system`（忽略）

  只有一条数据的系统表，或衍生表只有一条数据的主查询。

- `const`

  仅仅能查到一条数据的`SQL`，用于`primary key`或者`unique`索引。

- `eq_ref`

  用于连接查询。
  
  对于每个索引键的查询，返回匹配的唯一行数据，有且只有1个，不能为0。
  
  常见于唯一索引和主键索引。

- `ref`

  非唯一性索引查询，返回匹配的所有行。

- `range`

  检索指定范围的行，需要使用一个索引。

- `index`

  对索引表进行遍历。

- `all`

  对数据表进行遍历。

### 5. Possible_keys

可能用到的索引。

### 6. key

实际用到的索引。

### 7. key_len

索引使用的字节数，其值为索引字段最大可能长度，越短越好。

可用于判断复合索引是否被完全使用。

对于`utf8`，一个字符占3个字节。如果索引字段可以为`null`，则会使用1个字节用于标识。如果索引字段为`varchar`，使用2个字节标识可变长度。

### 8. ref

用于指明当前表所参照的字段，注意跟`type`中的`ref`值区分开。

### 9. rows

实际通过索引查询到的数据个数。

### 10. extra

执行情况的额外说明。

- `using filesort`

  需要额外进行一次排序，而不是使用索引的顺序，性能消耗大。常见于`order by`语句中。

  示例1：

  ```sql
  explain select * from test where a1='' order by a2;
  ```

  小结：对于单索引，如果排序和查找是同一个字段，则不会出现`using filesort`；如果排序和查找不是同一个字段，则会出现`using filesort`。

  示例2：（注意存在a1, a2, a3, a4四个字段）

  ```sql
  alter table test add index idx_a1_a2_a3 (a1, a2, a3);
  explain select * from test where a1='' order by a3; # using filesort
  explain select * from test where a2='' order by a3; # using filesort
  explain select * from test where a1='' order by a2;
  ```

  小结：对于复合索引，`where`和`order by`按照复合索引的顺序使用，不要跨列或无序使用。

- `using temporary`

  用到了临时表，性能损耗大。一般出现在`order by`和`group by`语句中。

  示例：

  ```sql
  explain select * from test where a1 in (1, 2, 3) group by a1;
  explain select * from test where a1 in (1, 2, 3) group by a2; # using temporary
  ```

- `using index`

  性能较好，常见于索引覆盖。

  只要使用到的列，全部都在索引中，就称为”索引覆盖“。

  这时只需要从索引文件中获取数据，不需要读取数据文件，即不回表查询。
  
- `using where`

  回表查询时，会出现`using where`。

- `impossible where`

  `where`子句永远为`false`。
  
- `using join buffer`

  `mysql`引擎使用了连接缓存。


## 语句分析

### 1. profiles

- 查看是否支持`profiles`。
  
  ```sql
  SHOW VARIABLES LIKE "%have_profiling%";
  ```

  或者：

  ```sql
  SELECT @@have_profiling;
  ```

- 查看`profiles`开启状态。
  
  ```sql
  SHOW VARIABLES LIKE "profiling";
  ```

  或者：

  ```sql
  SELECT @@profiling;
  ```

  返回的结果中，`OFF`或者`0`表示关闭，`ON`或者`1`表示开启。

- 打开`profiles`。
  
  ```sql
  SET profiling=ON;
  ```

  或者：

  ```sql
  SET profiling=1;
  ```

  `proflie`开启之后，会记录所有查询语句所花的时间，但是不是很精确。

- 关闭`profiles`。
  
  ```sql
  SET profiling=OFF;
  ```

  或者：

  ```sql
  SET profiling=0;
  ```

- 查看`profiles`统计信息
  
  ```sql
  SHOW profiles;
  ```

- 查看某个查询的性能
  
  ```sql
  SHOW PROFILE [type...] FROM QUERY query_id;
  ```

  `query_id`可以通过上一个命令获取。

  `type`如果省略，则只显示时间消耗。常见的`type`参数有`ALL`、`BLOCK IO`、`CPU`、`IPC`等，表示需要进行分析的具体内容。


### 2. `trace`

`MySQL5.6`开始提供了针对`SQL`语句的`TRACE`，通过`TRACE`文件，可以知道执行计划的生成过程。

- 查看`trace`的开启情况。
  
  ```sql
  SHOW VARIABLES LIKE "optimizer_trace";
  ```

  或者：

  ```sql
  SELECT @@optimizer_trace;
  ```

- 打开`trace`。
  
  ```sql
  SET optimizer_trace="enabled=on,one_line=off";
  ```

- 设置`trace`能使用的内存大小。
  
  ```sql
  SET optimizer_trace_max_mem_size=1048576;
  ```

- 查看分析结果
  
  ```sql
  SELECT * FROM information_schema.optimizer_trace\G
  ```

### 3. 记录全局日志

记录开启之后的全部sql语句，建议只在调优和开发过程中打开。

- 查看状态
  
  ```sql
  SHOW VARIABLES LIKE '%general_log%';
  SHOW VARIABLES LIKE "%log_output%";
  ```

- 开启，并将sql记录在表里

  ```sql
  SET GLOBAL general_log=1;
  SET GLOBAL log_output='table';
  ```

- 开启，并将sql记录在文件里

  ```sql
  SET GLOBAL general_log=1;
  SET GLOBAL log_output='file';
  SET GLOBAL general_log_file='/tmp/general.log';
  ```


## 批量导入数据的优化

### 1. 批量插入数据的命令

```sql
LOAD DATA LOCAL infile 数据文件路径 INTO TABLE 表名 FIELDS TERMINATED BY ',' LINES TERMINATED BY '\n';
```

对于数据文件中的每行数据，以`,`号来分隔每个字段。

### 2. 优化建议

- 按照主键顺序插入。

- 关闭对唯一性索引的校验：
  
  ```sql
  SET UNIQUE_CHECKS=0;
  ```

- 关闭事务的自动提交：
  
  ```sql
  SET AUTOCOMMIT=0;
  ```


## `INSERT`优化

- 如果对一张表插入多条数据，尽量将多条数据放在一个`INSERT`语句中，能降低客户端和数据库的连接、关闭消耗，效率会比分开执行的`INSERT`语句快。

- 将多个插入放在一个事务中。

- 按照主键的顺序插入。


## `group by`优化

`group by`除了会进行分组外，还会进行排序操作。如果分组时使用了聚合函数，则也会进行聚合函数的计算。

如果想避免排序造成的性能消耗，可以在`group by`子句之后，使用`order by null`来取消排序。

除了取消排序，还可以对分组字段建立索引。


## 子查询优化

使用子查询，可以一次性完成需要多个步骤才能完成的工作。

子查询的优化准则为，尽量使用连接查询代替子查询。


## `or`优化

对于包含`OR`的查询语句，如果要使用到索引，则`OR`两侧的每个条件都必须用到索引。

优化方式除了增加索引外，还可以使用`union`替代`or`。


## 分页查询优化

1. 在索引上完成排序分页操作，然后回数据表查询所需的其他字段。

2. 将`limit`查询转换成对某个位置的查询，适用于主键自增的表，即不能出现主键断层。如`id > 10000 limit 10`。

3. 如果想优化`count()`，可以新建一张表，记录总数。


## 人为控制索引

### 1. `USE INDEX`

使用`USE INDEX(索引名)`，可以让`MySQL`只参考指定的索引，但不一定会使用。

### 2. `IGNORE INDEX`

可以使用`IGNORE INDEX(索引名)`，来让`MySQL`忽略一个或多个索引。

### 3. `FORCE INDEX`

通过使用`FORCE INDEX(索引名)`，可以强制要求`MySQL`使用指定的索引。


## 应用层面的优化 

### 1. 使用连接池

对于数据库来说，建立连接的代价比较昂贵，因此，如果频繁的建立和关闭连接，会消耗较多资源。建立连接池，可以减少连接次数，提高访问性能。

### 2. 避免对数据进行重复查询

尽量一次性获取所需数据，减少无用的重复请求。

### 3. 增加缓存

可以在应用层和数据库之间，增加缓存层。查询数据时，直接访问缓存层，即可以提高查询速度，也可以降低数据库的压力。

### 4. 对查询进行分流

通过`MySQL`的主从复制，可以实现读写分离。

增删改操作由主结点执行，查询操作则分发给从结点，从而可以降低单台服务器的读写压力。

### 5. 分布式数据库架构

将数据分布在多台服务器，可以很好实现负载均衡，从而解决大数据量和高负载问题。


## `MySQL`并发参数

### 1. `max_connections`

`max_connections`用于控制`MySQL`数据库的最大连接数，默认151，查看变量值的命令为：

```sql
SHOW VARIABLES LIKE "max_connections";
```

或者：

```sql
SELECT @@global.max_connections;
```

注意，`max_connections`是全局变量。

当连接数达到最大值时，后续的连接请求可能会失败。查看失败的连接请求数的命令为：

```sql
SHOW STATUS LIKE "connection_errors_max_connections";
```

设置最大连接数时，需要考虑多个因素的影响。如操作系统的内存大小、连接的负荷、cpu性能、期望的响应时间等。

### 2. `back_log`

如果`MySQL`的连接数达到`max_connections`，新来的请求会被放在堆栈中，`back_log`参数就是该堆栈能容纳的数量。

如果等待连接的数量超过`back_log`，新的连接请求就会报错。

### 3. `thread_cache_size`

`MySQL`数据库会缓存一些服务线程，当又客户端请求到来时，会将其中的服务线程分配给连接会话使用。

`thread_cache_size`用于控制该线程数量。

### 4. `innodb_lock_wait_timeout`

该参数，主要用于控制`InnoDB`事务等待行锁的时间。
