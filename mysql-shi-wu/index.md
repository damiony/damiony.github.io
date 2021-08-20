# 【MySQL】事务


## 事务

### 1. 概念

事务由一条或者多条sql语句构成，这些sql语句要么全部执行成功，要么全部执行失败。

### 2. 事务特性

事务具备`ACID`特性。

- 原子性 (atomicity)：事务中所有操作是不可再分割的原子单位。事务中所有操作要么全部执行成功，要么全部执行失败。

- 一致性 (consistency)：事务执行后，数据库的状态仍与业务规则保持一致。如转账业务，无论事务执行成功与否，参与转账的两个账号余额之和应该是不变的。

- 隔离性 (isolation)：在并发操作中，不同事务之间应该隔离开来，使每个并发中的事务不会相互干扰。

- 持久性 (durability)：一旦事务提交成功，事务中所有的数据操作都必须被持久化到数据库。即使提交事务后，数据库马上崩溃，在数据库重启时，也能通过某种机制恢复数据。

### 3. 分类

- 隐式事务：
  
  没有明显的开启和结束标记。

- 显式事务：
  
  具有明显的开启和结束标记。

  执行步骤为，取消隐式事务自动开启的功能 => 开启事务 => 编写事务语句 => 结束事务

### 4. 示例

```sql
# 取消事务自动开启
SET autocommit = 0;

# 开启事务 有的客户端不用加这句
START TRANSACTION;

# 更新数据
UPDATE stuinfo SET balance=balance-5000 where stuid = 1;
UPDATE stuinfo SET balance=balabce+5000 where stuid = 2;

# 提交
COMMIT;

# 或者回滚
# ROLLBACK;
```


## 数据库的隔离级别

### 1. 说明

一个事务与其他事务隔离的程度称为隔离级别。不同隔离级别对应不同的干扰程度，隔离级别越高，数据一致性就越好，但并发性越弱。

当多个事务同时运行、并访问数据库中的相同数据时，如果没有采取必要的隔离机制，就会导致各种并发问题：

- 脏读。

  对于两个事务T1和T2，T1读取了已经被T2更新但还没有被提交的字段，若之后T2回滚，T1读取的内容就是临时且无效的。

- 不可重复读。

  对于两个事务T1和T2，T1读取了一个字段，然后T2更新了该字段，之后T1再次读取同一个字段，获得的值就不同了。

- 幻读。

  对于两个事务T1和T2，T1从一个表中根据某些条件读取出一些记录，然后T2向该表插入了符合条件的新记录。T1再次读取时，就会把新记录也读出来。



## 数据库的隔离级别

### 1. 隔离级别说明

- 读未提交数据(READ UNCOMMITTED)

  允许事务读取未被其他事务提交的变更。脏读，不可重复读和幻读的问题都会出现。

- 读已提交数据(READ COMMITTED)

  只允许事务读取已经被其他事务提交的变更。可以避免脏读，但不可重复读和幻读问题仍然会出现。简称`RC`。

- 可重复读(REPEATABLE READ)

  确保事务可以多次从一个字段中读取相同的值，但这个事务持续期间，禁止其他事务对这个字段进行更新。可以避免脏读和不可重复读，但幻读的问题仍然存在。简称`RR`。

- 串行化(SERIALIZABLE)

  确保事务可以从一个表中读取相同的行，在这个事务持续期间，禁止其他事务对该表执行插入、更新和删除操作。所有并发问题都可以避免，但性能十分低下。

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

每启动一个mysql客户端程序，就会建立一个连接会话，每个连接都有一个变量`tx_isolation`表示当前的事务隔离级别。注意，新版`mysql`使用的是`transaction_isolation`变量。

### 2. mysql查看当前连接的隔离级别

```mysql
select @@tx_isolation;
```

### 3. mysql修改当前连接的隔离级别

当前连接的隔离级别不会影响到其他连接。

```mysql
set session transaction isolation level read uncommitted;
```

### 4. mysql设置数据库系统全局的隔离级别

需要重新打开`mysql`会话，设置才会生效。

```mysql
set global transaction isolation level read committed;
```


## SAVEPOINT

### 1. 说明

只能搭配`rollback`使用

### 2. 示例

```mysql
> SET autocommit=0;
> DELETE FROM account WHERE id=25;
> SAVEPOINT a;
> DELETE FROM account WHERE id=28;
> ROLLBACK TO a;
```
