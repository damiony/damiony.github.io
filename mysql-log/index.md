# 【MySQL】日志



## `bin log`日志格式

使用如下`SQL`语句可以查看`bin log`日志的格式：

```sql
SHOW VARIABLES LIKE "binlog_format";
```

或者：

```sql
SELECT @@binlog_format;
```

### `statement`

设置为该格式时，日志文件中记录的都是`SQL`语句。

对数据进行增删改的`SQL`语句，都会被记录在日志文件中，通过`mysqlbinlog`可以查看每条语句的内容。

### `row`

该日志格式，记录的时每一行的数据变更，而不是`SQL`语句本身。

### `mixed`

`mixed`混合了`statement`和`row`两种格式。默认情况采用`statement`格式，但是在某些情况下，会使用`row`来记录日志。
