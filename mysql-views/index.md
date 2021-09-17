# 【MySQL】视图


## 什么是视图

视图是一种虚拟表，不在数据库中实际存在。

视图行和列的数据，是在使用视图时动态生成的。


## 视图操作

### 1. 创建视图

```sql
CREATE view 视图名称 AS 查询语句; 
```

### 2. 修改视图

```sql
ALTER view 视图名 AS 查询语句;
```

或者：

```sql
CREATE OR REPLACE view 视图名 AS 查询语句;
```

### 3. 查看视图列表

使用`SHOW TABLES`不仅可以看到数据表，还可以看到被定义的视图。

### 4. 查看视图结构

```sql
DESC 视图名;
```

或者：

```sql
SHOW CREATE view 视图名;
```

### 4. 删除视图

```sql
DROP view 视图名...;
```


## 视图内容的更新

### 1. 插入

- 同表的插入语法一致。

- 会更新原始表。

### 2. 修改

- 同表的修改语法一致。

- 会更新原始表。

### 3. 删除

- 使用表的`delete`语法。

- 会更新原始表。

### 4. 说明

视图的可更新性和视图查询的定义有关系，以下类型的视图不能更新：

- 包含关键字：分组函数、distinct、group by、having、union、union all。

- 常量视图。

- select子句中包含子查询。

- join（某些条件下可更新）。

- from一个不能更新的视图。

- where子句的子查询引用了from子句中的表。


## 使用示例

- 查询邮箱中包含a字符的员工名、部门名和工种信息：

  创建视图：

  ```sql
  CREATE view test_v1
  AS
  SELECT name, department, title
  FROM employees e
  JOIN departments d ON e.department_id = d.department_id
  JOIN jobs j ON j.job_id = e.job_id;
  ```

  使用视图：

  ```sql
  SELECT * FROM test_v1 WHERE name LIKE "%a%";
  ```

- 查询各部门的平均工资级别：

  创建视图：
  
  ```sql
  CREATE view test_v2
  AS
  SELECT AVG(salary) ag, department_id
  FROM employees
  GROUP BY department_id;
  ```

  使用：

  ```sql
  SELECT * FROM test_v2;
  ```
