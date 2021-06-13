# MySQL 预读机制


## 预读机制

预读机制是相对于`Innodb`存储引擎而言的。

所谓预读机制，就是在加载数据页时，可能会将相邻数据页一起预先加载到`buffer pool`中，之后读取时就可以直接从内存获取，不必从磁盘读入。

使用`Innodb`的预读机制，可以减少磁盘IO次数，提高读写性能。

## 预读模式

`Innodb`一共提供了两种预读模式：

- 线性预读：判断当前extend中的数据是否被连续访问，从而决定是否把下一个extend从磁盘中读出来，加载到buffer pool。
- 随机预读：判断当前extend中读取page的个数，从而决定是否把当前extend的数据加载到buffer pool中。

## 线性预读

`MySQL5.5`之后，默认使用的是线性预读模式，与之有关的配置为`innodb_read_ahead_threshold`，这个参数表示，当前extend中，如果有超过`innodb_read_ahead_threshold`个page被连续访问，就预先加载下一个extend的数据。

`innodb_read_ahead_threshold`的默认参数是56，最大为64，最小为0。如果为0，表示关闭线性预读。即使关闭了线性预读，只要随机预读没有显式打开，就不会使用它。

`innodb`组织数据的顺序为`tablespace, segment, extend, page, row`，因为`extend`固定为`1Mb`，`page`默认为`16Kb`，所以一个`extend`能容纳64个`page`，所以`innodb_read_ahead_threshold`最大为64。

### 预读原理

- 连续访问，是根据page的最新一次访问时间来判断的。

- 如果当前page是extend的第一个，则看后面的63个page，如果访问时间连续递减的个数，大于等于`innodb_read_ahead_threshold`，就加载上一个extend。

- 如果当前page是extend的最后一个，则看前面的63个page，如果访问时间连续递增的个数，大于等于`innodb_read_ahead_threshold`，就加载下一个extend。

## 随机预读

`MySQL5.5`之后，随机预读是默认关闭的。

与随机预读有关的配置为`innodb_random_read_ahead`。

