# 【MySQL】Order By执行流程


## `sort_buffer`

`MySQL`会给每个连接线程分配一块内存，称为`sort_buffer`。`order by`的所有排序算法，都需要依靠这块内存来完成。


## 全字段排序

假设有一条查询语句：

```sql
select city, name, age from t where city='杭州' order by name limit 1000;
```

其中，字段`city`上有普通索引。

通常情况下，`order by`的执行流程为：

1. 初始化`sort_buffer`，确定`name、city、age`三个字段。

2. 从索引`city`找到满足条件的第一行记录，获取主键`id`。

3. 从主键索引找到整行，取出`name`、`city`、`age`三个字段，放入`sort buffer`。

4. 在索引`city`中继续取下一个记录。

5. 对`sort_buffer`中的数据按照字段`name`做快速排序。

6. 取前`1000`行返回。


## `rowid`排序

如果单行数据太大，`MySQL`会采用`rowid`排序。

参数`max_length_for_sort_data`用来控制排序数据的长度。如果单行数据长度超过该值，`MySQL`就认为单行太大，转而使用`rowid`排序。

还是同样的查询语句：

```sql
select city, name, age from t where city='杭州' order by name limit 1000;
```

同样，字段`city`上有普通索引。`rowid`排序的流程为：

1. 初始化`sort_buffer`，确定`name, id`两个字段。

2. 从索引`city`获取第一个满足要求的记录，获取主键`id`。

3. 从主键索引取出整行，取`name`、`id`两个字段，存入`sort_buffer`。

4. 从索引`city`取下一个记录。

5. 对`sort_buffer`按`name`排序。

6. 取前`1000`行，按照`id`的值回原表取`city`、`name`和`age`三个字段返回。


## `sort_buffer_size`

排序操作可能在内存中完成，也可能使用外部排序。这取决于排序所需内存，和参数`sort_buffer_size`。

参数`sort_buffer_size`代表着`sort_buffer`的大小，如果待排序数据量小于该参数，排序就在内存中完成，否则就在临时文件中完成。

外部排序一般使用归并排序算法。


## 排序算法比较

如果内存够大，`MySQL`会优先选择全字段排序，因为`rowid`排序会多次回表，从而导致多次读磁盘。


