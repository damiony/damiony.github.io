# 【MySQL】锁机制


## `MyISAM`锁

### 1. 显式加锁

`MyISAM`只支持表锁。

在执行查询语句时，会自动给涉及到的表加读锁。在执行增删改操作时，会自动给涉及到的表加写锁。

如果想显式加锁，语法为：

- 加读锁

  ```sql
  LOCK TABLE 表名 READ;
  ```

- 加写锁
  
  ```sql
  LOCK TABLE 表名 WRITE;
  ```

- 解锁
  
  ```sql
  UNLOCK TABLES;
  ```

## 查看表的使用情况

```sql
SHOW OPEN TABLES;
```

或者：

```sql
SHOW STATUS LIKE "Table%";
```


## `InnoDB`锁

### 1. 显式加锁

对于增删改操作，`InnoDB`会自动给记录添加排他锁。对于查询操作，`InnoDB`不会加任何锁。

如果想显式加锁，语法为：

- 加共享锁：
  
  ```sql
  SQL查询语句 LOCK IN SHARE MODE;
  ```

- 加排他锁：
  
  ```sql
  SQL查询语句 FOR UPDATE;
  ```

### 2. 优化建议

- 尽可能使用索引来完成检索，避免无索引时导致行锁升级为表锁。

- 合理设计索引，尽量缩小锁的范围。

- 尽量减少索引条件，缩小索引范围，从而避免间隙锁。

- 控制事务大小，减少锁定的资源量。

- 尽可能使用较低的事务隔离级别。
