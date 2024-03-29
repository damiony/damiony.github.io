# 【MySQL】Join原理


## Join 原理

常有人说，查询数据时不要使用`Join`，因为性能很差。然而，你是否好奇过，事实真的是这样吗？

为了能准确分析出`Join`语法的执行效率，接下来将会对`Join`的原理进行讨论。

### Index Nested-Loop Join

`Index Nested-Loop Join`简称为`NLJ`，算法的具体执行流程为：

1. 从表1中读取一行记录。

2. 取出这行记录的指定字段，去表2进行查找。

3. 从表2中取出满足条件的行，与步骤1取出的行组成结果集。

4. 重复执行上述步骤，直到读取完表1的记录。

上述算法的重要前提是，被驱动表可以使用索引。此时，因为使用到了表2的索引，所以`Join`的性能较好。

### Simple Nested-Loop Join

该算法的使用前提是，被驱动表无法使用索引。

因为无法使用索引，所以每次去被驱动表做匹配时，都需要进行全表扫描，性能很差。

注意，无索引情况下，`MySQL`使用的是`Block Nested-Loop Join`算法。

### Block Nested-Loop Join

`Block Nested-Loop Join`简称`BNL`，该算法也不会用到被驱动表的索引。

具体的执行流程为：

1. 将表1的数据全部读入`join_buffer`中。

2. 遍历表2，读取每一行数据。

3. 每从表2中读取一行数据，就去`join_buffer`中做匹配。如果满足条件，就放入结果集。

`BNL`与`Simple Nested-Loop Join`的区别在于，`BNL`的匹配操作是在内存中完成的，速度更快。

如果`join_buffer`内存放不下表1的所有数据，则执行流程为：

1. 扫描表1，顺序读取部分数据，放入`join_buffer`中。

2. 扫描表2，读取每一行数据。

3. 从表2中每读取一行数据，就跟`join_buffer`做对比。如果满足条件，就放入结果集。

4. 清空`join_buffer`。

5. 继续扫描表1，然后执行步骤2，直到数据扫描完毕。

`join_buffer`的大小受参数`join_buffer_size`控制，默认是256k。该参数设置的越大，可以容纳的表1数据就会越多，被驱动表的扫描次数越少，性能也会越好。


## Join 使用建议

### 1. 使用场景

如果能使用到被驱动表的索引，则可以使用`Join`。

如果不能使用到被驱动表的索引，查询时会扫描过多行数。在这种情况下，尽量不要使用`Join`。

### 驱动表的选择

对于`Index Nested-Loop Join`，应选择小表做驱动表。

对于`Simple Nested-Loop Join`，无论哪张表做驱动表，性能都是一样的。

对于`Block Nested-Loop Join`，如果`join_buffer_size`足够大，则任何一种选择都没差别，如果不够大，则选择小表做驱动表。

可以根据执行计划的`extra`字段，来判断`join`使用的是何种算法。

### 如何查找小表

查找小表的步骤如下：

1. 先将所有表按照条件过滤。

2. 然后计算参与`join`的各个字段的总数据量。

3. 数据量小的就是小表。


