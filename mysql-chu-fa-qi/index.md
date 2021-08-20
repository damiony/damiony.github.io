# 【MySQL】触发器


## 什么是触发器

触发器是与数据表有关的数据库对象。在执行`INSERT`、`UPDATE`、`DELETE`之前或者之后，可以执行触发器中定义的`SQL`语句。

在触发器中，分别使用`OLD`和`NEW`来表示发生变化的记录。


## 触发器分类

- `INSERT`型触发器：
  
  `NEW`表示将要或者已经新增的数据。

- `UPDATE`型触发器：
  
  `OLD`表示修改之前的数据，`NEW`表示将要或已经修改的数据。

- `DELETE`型触发器：
  
  `OLD`表示将要或者已经删除的数据。


## 触发器语法

### 1. 创建触发器

```sql
CREATE TRIGGER trigger_name
BEFORE/AFTER INSERT/UPDATE/DELETE
ON 表名 [FOR EACH ROW]
BEGIN
    语句;
END
```

说明：

- `FOR EACH ROW`表示行级触发器，`MySQL`只支持行级触发器。

- 需要使用`DELIMITER`来设置触发器语句的结束标记。

- `BEGIN/END`中的`SQL`语句不予许返回结果集。

- 如果语句只有一行，可以省略`BEGIN/END`关键字。

### 2. 查看触发器

```sql
SHOW TRIGGERS;
```

### 3. 删除触发器

```sql
DROP TRIGGER 库名.触发器名;
```


## 示例

### 1. 记录数据的新增

```sql
DELIMITER $

CREATE TRIGGER trigger_test_insert
AFTER INSERT
ON t1 FOR EACH ROW
BEGIn
    INSERT INTO t2 VALUES("新增一行数据", NEW.Id, now());
END$
```
