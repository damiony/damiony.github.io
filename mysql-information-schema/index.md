# 【MySQL】information_schema


## `information_schema`介绍

`information_schema`库中，存放着多个视图，主要用来统计数据库相关的元数据。常用的有：

- `TABLES`。


## `TABLES`

### 1. 说明

保存了所有表的数据信息。

### 2. 常用字段说明

- `TABLE_SCHEMA`：表所在的库。

- `TABLE_NAME`：表名。

- `ENGINE`：表使用的引擎。

- `TABLE_ROWS`：表的数据行，非实时。

- `AVG_ROW_LENGTH`：表行长度的平均值。

- `DATA_LENGTH`：表的存储空间大小，非实时。

- `INDEX_LENGTH`：索引占用空间大小。

- `DATA_FREE`：表中是否有碎片。

### 3. 示例

1. 统计每个库中，所有表的个数，以及所有表名。

```sql
SELECT TABLE_SCHEMA, count(TABLE_NAME), group_concat(TABLE_NAME)
FROM information_schema.TABLES
GROUP BY TABLE_SCHEMA;
```


