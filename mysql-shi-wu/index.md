# 【MySQL】事务


## 事务

### 1. 事务的概念

事务是数据库并发操作中，最小的控制单元，由一条或者多条sql语句构成，这些sql语句要么全部执行成功，要么全部执行失败。

事务在引擎层实现。有的引擎支持事务，如`InnoDB`，有的引擎不支持，如`MyISAM`和`Memory`。

### 2. 事务的特性

事务具备`ACID`特性。

- 原子性 (atomicity)：
  
  事务中所有操作是不可再分割的原子单位。事务的所有操作要么全部执行成功，要么全部执行失败。

- 一致性 (consistency)：
  
  事务执行后，数据库的状态仍与业务规则保持一致。
  
  如转账业务，无论事务执行成功与否，参与转账的两个账号余额之和应该是不变的。

- 隔离性 (isolation)：
  
  在并发操作中，不同事务之间保持隔离，不会相互干扰。

- 持久性 (durability)：
  
  事务一旦提交成功，所有数据操作都会被持久化到数据库。

### 3. 事务的分类

事务分为隐式事务和显式事务。

隐式事务没有开启和结束标记，一条`SQL`语句就是一个事务。使用隐式事务可能导致意外的长事务。

显式事务可以有开启标记，也可以没有开启标记，但是一定要有结束标记。

显式事务的执行步骤为：

1. 取消隐式事务自动开启的功能。

2. 开启事务，开启语句为`BEGIN`、`START TRANSACTION`或者什么也不写。

3. 编写事务语句。

4. 结束事务。

对于显式事务，如果不想每次都执行`begin`，可以使用`commit work and chain`，表示提交本事务后，自动开启下一个事务。

### 4. 使用示例

```sql
# 取消事务自动开启
SET autocommit = 0;

# 开启事务
START TRANSACTION;

# 更新数据
UPDATE userinfo SET balance = balance - 5000 WHERE uid = 1;
UPDATE userinfo SET balance = balabce + 5000 WHERE uid = 2;

# 提交
COMMIT;

# 或者回滚
# ROLLBACK;
```


## 隔离级别

### 1. 什么是隔离级别

一个事务与其他事务隔离的程度称为隔离级别。

隔离级别越高，数据一致性就越好，但并发性越弱。

### 2. 并发问题

当多个事务并发访问数据库中的相同数据时，如果没有使用合适的隔离级别，就会产生各种问题：

- 脏读。

  对于两个事务T1和T2，T1读取了已经被T2更新但还没有被提交的字段，若之后T2回滚，T1读取的内容就是临时且无效的。

- 不可重复读。

  对于两个事务T1和T2，T1读取了一个字段，然后T2更新了该字段，之后T1再次读取同一个字段，获得的值就不同了。

- 幻读。

  对于两个事务T1和T2，T1从一个表中根据某些条件读取出一些记录，然后T2向该表插入了符合条件的新记录。T1再次读取时，就会把新记录也读出来。



## 数据库的隔离级别

### 1. 隔离级别说明

- 读未提交数据（`READ UNCOMMITTED`）

  允许事务读取未被其他事务提交的变更。
  
  会出现脏读，不可重复读和幻读的问题。

- 读已提交数据（`READ COMMITTED`）

  简称`RC`。只允许事务读取已经被其他事务提交的变更。

  可以避免脏读，但不可重复读和幻读问题仍然会出现。

- 可重复读（`REPEATABLE READ`）

  简称`RR`。一个事务执行过程中看到的数据，总是跟它启动时看到的数据一致。
  
  可以避免脏读和不可重复读，但幻读的问题仍然存在。

- 串行化（`SERIALIZABLE`）

  在事务持续期间，禁止其他事务对该表执行插入、更新和删除操作。对一行记录，”写“会加”写锁“，读会加”读锁“。
  
  所有并发问题都可以避免，但性能十分低下。

### 2. Oracle支持的事务隔离级别：

- `READ COMMITED`（默认）

- `SERIALIZABLE`

### 3. Mysql支持的事务隔离级别

- `READ UNCOMMITTED`

- `READ COMMITTED`

- `REPEATABLE READ`（默认）

- `SERIALIZABLE`


## MySql设置隔离级别

### 1. 说明

每启动一个`MySQL`客户端程序，就会建立一个连接会话，每个连接会话都有一个变量`tx_isolation`表示当前的事务隔离级别。

注意，新版`MySQL`使用的是`transaction_isolation`变量。

### 2. 查看当前连接的隔离级别

```sql
SELECT @@tx_isolation;
```

### 3. 设置当前连接的隔离级别

当前连接的隔离级别不会影响到其他连接。设置语法如下：

```sql
set session transaction isolation level read uncommitted;
```

### 4. 设置全局的隔离级别

需要重新建立`MySQL`连接，全局设置才会生效。语法如下：

```sql
set global transaction isolation level read committed;
```


## SAVEPOINT

### 1. 什么是`SAVEPOINT`

`SAVEPOINT`是在事务中搭配`ROLLBACK`一起使用的某种机制，可以控制事务回滚的进度。

### 2. 示例

```sql
SET autocommit=0;

DELETE FROM account WHERE id=25;

SAVEPOINT a;

DELETE FROM account WHERE id=28;

ROLLBACK TO a;
```


## 长事务弊端

长事务提交前，它可能用到的回滚日志都必须保留，会占用大量存储空间。

`MySQL5.5`及以前的版本，回滚日志跟数据字典一起放在`ibdata`文件里，即使回滚段被清除，文件也不会变小。

长事务还会占用锁资源，从而拖垮整个库。

因此，尽量不要使用长事务。

### 查询长事务

如果查询持续时间超过`60s`的长事务，可以使用如下语句：

```shell
select * from information_schema.innodb_trx where TIME_TO_SEC(TIMEDIFF(now(), trx_started)) > 60;
```

