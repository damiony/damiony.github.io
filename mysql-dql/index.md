# 【MySQL】DQL数据查询



## DQL

`DQL`：（Data Query Language）数据查询语言，用来查询记录。


## 查询语法

查询语句的基本语法如下：

```sql
SELECT 查询列表 FROM 表名;
```

常见的查询列表内容如下：

- 字段

  ```sql
  SELECT `last_name` FROM `employees`
  ```

- 表达式

  ```sql
  SELECT 100 % 3;
  ```

- 常量

  ```sql
  SELECT 100;
  ```

- 函数

  ```sql
  SELECT DATABASE();
  SELECT VERSION();
  SELECT USER();
  ```


## 查询说明

### 1. 别名

可以对查询的结果起别名，或者对被查询的表起别名。

- 使用`as`关键字

  ```sql
  SELECT USER() AS "username";
  ```

- 使用空格

  ```sql
  SELECT user() username;
  ```

### 2. `+`号

在`MySQL`中，使用`+`号时需注意如下规则：

- 如果两个数都是数值型，则直接相加。

- 如果其中一个操作数为字符型，则将字符型转换成数值型。如果无法转换，则当作`0`处理。

- 其中一个操作数为`null`，结果为`null`。 

### 3. `DISTINCT`去重

`DISINCT`子句可以消除重复字段。语法如下：

```sql
SELECT DISTINCT field FROM tablename
```

### 4. `IFNULL`函数

- 语法：

  ```sql
  IFNULL(表达式1, 表达式2)
  ```

- 功能：

  如果表达式1为`null`，则显示表达式2，否则显示表达式1。


### 5. 正则表达式

`SQL`语句可以使用正则表达式进行查询：

```sql
SELECT * FROM 表名 WHERE 字段 REGEXP 正则表达式;
```


## 常见函数

常见函数有如下几类：

- 字符函数

- 数学函数

- 日期函数

- 流程控制函数

- 聚合函数

### 1. 字符函数

- `CONCAT`拼接字符

  ```sql
  SELECT CONCAT("hello", name) FROM user;
  ```

- `LENGTH`获取字节长度

  ```sql
  SELECT LENGTH("hello,少年");
  ```

- `CHAR_LENGTH`获取字符长度

  ```sql
  SELECT CHAR_LENGTH("hello,少年");
  ```

- `SUBSTR`截取子串
  
  ```sql
  SUBSTR(string, start, end)
  ```
  
  截取从`start`到`end`（包含`end`）的所有字符，字符串从1开始计数。
  
  `start`可以为负数，表示从最后一个字符开始往前数。
  
  `end`可以省略，表示一直到结尾。

- `LEFT`从左边截取字符

  ```sql
  SELECT LEFT('洪世贤',1);
  ```

- `RIGHT`从右边截取字符

  ```sql
  SELECT RIGHT('少奶奶', 2);
  ```

- `INSTR`获取字符第一次出现的索引

  ```sql
  SELECT INSTR('孙悟空三打白骨精','白骨精');
  ```

- `TRIM`去除前后指定字符，默认去空格

  `TRIM`的语法为：
  
  ```sql
  TRIM ([BOTH | LEADING | TRAILING] [removed] [FROM] str)
  ```
  
  `BOTH`表示删除首尾字符，`LEADING`表示删除前导字符，`TRAILING`表示删除尾随字符。
  
  `BOTH`、`LEADING`和`TRAILING`都可以省略，省略后默认使用`BOTH`。
  
  `removed`是被删除的字符，可以省略，省略时默认删除空格。
  
  `FROM`可以省略，但是如果`BOTH | LEADING | TRAILING`存在，或者`removed`存在，则`FROM`不可省略。
  
  使用示例：

  ```sql
  SELECT TRIM(' 空  格 ') as result;
  ```
  
  ```sql
  SELECT TRIM('x' FROM 'xxx空xxx格xxx') as result;
  ```

- `LPAD`左填充

  左填充满10个字符：
  
  ```sql
  SELECT LPAD('木婉清', 10, 'a');
  ```

- `RPAD`右填充

  右填充满10个字符：
  
  ```sql
  SELECT RPAD('木婉清', 10, 'a');
  ```

- `UPPER`字符转大写

- `LOWER`字符转小写

- `STRCMP`比较两个字符串的字符大小

  ```sql
  SELECT STRCMP('abc','acb')
  ```

### 3. 数学函数

- `ABS`绝对值

  ```sql
  SELECT ABS(-1.1);
  ```

- `CEIL`向上取整，返回大于等于该参数的最小整数

  ```sql
  SELECT CEIL(1.09);
  ```

- `FLOOR`向下取整，返回小于等于该参数的最大整数

  ```sql
  SELECT FLOOR(-1.09);
  ```

- `ROUND`四舍五入

  ```sql
  SELECT ROUND(1.8765);
  ```

- `TRUNCATE`截断小数位

  ```sql
  SELECT TRUNCATE(1.8765, 1); # 1.8
  ```

- `MOD`取余

  实质上：`a % b = a - a/b*b`。

  ```sql
  SELECT MOD(-10, 3); # -1
  ```

  ```sql
  SELECT MOD(10, 3); # 1
  ```

  ```sql
  SELECT MOD(-10, -3); # -1
  ```

  ```sql
  SELECT MOD(10, -3); # 1
  ```

### 4. 日期函数

- `NOW`获取当前日期和时间

  ```sql
  SELECT NOW();
  ```

- `CURDATE/CURRENT_DATE`获取当前日期

  ```sql
  SELECT CURDATE();
  ```

- `CURTIME/CURRENT_TIME`获取当前时间

  ```sql
  SELECT CURTIME();
  ```

- `DATEDIFF`比较两个日期相差几天

  ```sql
  SELECT DATEDIFF('1998-7-16', '2019-7-13');
  ```

  ```sql
  SELECT DATEDIFF('19980716', '20190713');
  ```

- `TIMESTAMPDIFF`比较两个日期时间的差距

  语法：

  ```sql
  TIMESTAMPDIFF(时间类型, 时间1, 时间2)
  ```

  时间类型可以是天day、小时hour、分钟minute、秒second等。

  示例：

  ```sql
  SELECT TIMESTAMPDIFF(second, "1999-01-01 18:00:00", "1999-01-01 19:00:00");
  ```

- `YEAR`从给定时间中获取年份

  ```sql
  SELECT YEAR("2001-1-12");
  ```

- `MONTH`从给定时间中获取月份

  ```sql
  SELECT MONTH("2001-01-12");
  ```

- `DAY`从给定时间中获取日数

  ```sql
  SELECT DAY("2001-1-12");
  ```

- `HOUR`从给定时间中获取小时数

  ```sql
  SELECT HOUR("2001-1-12 15:04:05")
  ```

- `MINUTE`从给定时间中获取分钟数

  ```sql
  SELECT MINUTE("2001-1-12 15:04:05")
  ```

- `SECOND`从给定时间中获取秒钟数

  ```sql
  SELECT SECOND("2001-1-12 15:04:05")
  ```

- `DATE_FORMAT`

  ```sql
  SELECT DATE_FORMAT('1998-7-16', '%Y年%m月%d日 %H小时%i分钟%s秒');
  ```

- `STR_TO_DATE`

  ```sql
  SELECT STR_TO_DATE('3/15 1998', '%m/%d %Y');
  ```

### 5. 流程控制函数

- `IF`函数

  ```sql
  SELECT IF(100>9, "好", "坏");
  ```

### 6. 聚合函数

聚合函数用于对一组数据进行统计分析，并得到一个值。

- `sum(字段名)`：求和。

- `avg(字段名)`：求平均数。

- `max(字段名)`：求最大值。

- `min(字段名)`：求最小值。

- `count(字段名)`：计算非空字段值的个数。
  
  `count(*)`用于查询总行数。
  
  `count(1)`作用类似于`count(*)`，但是效率较低。
  
  `count()`搭配`distinct`做去重统计，如：

  ```sql
  SELECT count(distinct department) FROM employees;
  ```


## 条件查询

### 1. 语法

```sql
SELECT 查询列表
FROM 表名
WHERE 筛选条件；
```

### 2. 筛选条件

- 按关系表达式筛选：`<`、`>`、`>=`、`<=`、`=`、`<>`、`!=`。

- 按逻辑表达式筛选：`and`、`or`、`not`，也可以使用 `&&`、`||`、`!`。

- 模糊查询：`like`、`in`、`not in`、`is null`、`is not null`、`between ... and ...`、`not bwtween ... and ... `。

### 3. 示例

- 查询部门编号不是 `50 ~ 100` 之间员工的姓名:

```sql
SELECT name FROM employees WHERE department_id < 50 or department_id > 100;
```

- 查询姓名中包含字符`a`的员工。

```sql
SELECT * FROM employees WHERE name LIKE '%a%';
```

- 查询姓名最后一个字符为`a`的员工。

```sql
SELECT * FROM employees WHERE name LIKE '%a';
```

- 查询姓名第一个字符为`a`的员工。

```sql
SELECT * FROM employees WHERE name LIKE 'a%';
```

- 查询姓名中第三个字符为`a`的员工。

```sql
SELECT * FROM employees WHERE name LIKE '___a%'; # 有三个下划线_
```

- 查询姓名中第二个字符为`_`的员工信息。

```sql
SELECT * FROM employees WHERE name LIKE '_\_%';
```

或者：

```sql
SELECT * FROM employees WHERE name LIKE '_$_%' escape '$';
```

- 查询部门编号是30/50/90的员工名

```sql
SELECT name FROM employees WHERE department IN (30, 50, 90);
```

- 查询部门编号是30～90之间的员工姓名

```sql
SELECT name FROM employees WHERE department BETWEEN 30 AND 90;
```

- 查询部门编号不是30～90之间的员工姓名

```sql
SELECT name FROM employees WHERE department NOT BETWEEN 30 AND 90;
```


## 排序查询

### 1. 语法

```sql
SELECT 查询列表
FROM 表名
WHERE 筛选条件
ORDER BY 排序列表;
```

### 2. 说明

- 排序列表可以是单个字段、多个字段、表达式、函数、列数、以及以上的组合。

- 升序使用关键字`asc`，降序使用关键字`desc`。如果省略关键字，则默认升序。

### 3. 示例

- 将员工按照工资降序

```sql
SELECT * FROM employees ORDER BY salary DESC;
```

- 按姓名长度进行升序

```sql
SELECT name FROM employees ORDER BY length(name);
```

- 查询员工信息，先按工资升序、再按部门将序

```sql
SELECT * FROM employees ORDER BY salary ASC, department DESC;
```

- 按照第2列降序排列

```sql
SELECT * FROM employees ORDER BY 2 DESC;
```


## 分组查询

需要使用`group by`子句。

### 1. 语法

```sql
SELECT 查询列表
FROM 表名
WHERE 筛选条件
GROUP BY 分组列表
HAVING 分组后筛选
ORDER BY 排序列表;
```

### 2. 说明

- 查询列表往往是聚合函数和被分组的字段。

- 分组前的筛选，是基于原始表，使用`where`，位于`group by`前面。

- 分组后的筛选，是基于分组后的结果集，使用`having`，位于`group by`后面。

### 3. 示例

- 查询每个工种员工平均工资

```sql
SELECT avg(salary), job_id FROM employees GROUP BY job_id;
```


## 分页查询

### 1. 语法

```sql
SELECT 查询列表
FROM 表
WHERE 筛选条件
GROUP BY 分组
HAVING 分组后筛选
ORDER BY 排序列表
LIMIT 起始条目索引，条目数;
```

### 2. 说明

起始条目索引从0开始，缺省则默认为0。

### 3. 示例

- 查询员工信息表的前5条

```sql
SEELCT * FROM employees LIMIT 5;
```

等价于：

```sql
SELECT * FROM employees LIMIT 0, 5;
```


## 子查询

### 1. 定义

在一个查询语句中，又嵌套了另一个`select`语句，则被嵌套的语句称为子查询或内查询，外面的语句称为主查询或外查询。

### 2. 说明

- 子查询一般放在小括号中。

- 单行子查询对应了单行操作符：`>, <, >=, <=, =, <>, !=`。

- 多行子查询对应了多行操作符：`any, some, all, in`。

- `select`后面子查询的结果必须单行单列，`from`后面子查询的结果可以多行多列。

### 3. `any, some, all`

`any, some, all`是用于将子查询返回的单列结果与指定值做比较，使用时，需要在前面加`>, <, >=, <=, <>, !=`中的某个操作符。

`some`的含义是，指定值大于、小于、大于等于、小于等于、不等于单列结果集的所有值。

`any, some`的含义是，指定值大于、小于、大于等于、小于等于、不等于单列结果集的某个值。

### 4. 示例

- 查询和Tom同部门的员工姓名和工资

  ```sql
  SELECT name, salary
  FROM employees
  WHERE department_id = (
    SELECT department_id
    FROM employees
    WHERE name = 'Tome'
  );
  ```

- 查询部门编号是50的员工个数

  ```sql
  SELECT
  (
    SELECT COUNT(*)
    FROM employees
    WHERE department_id = 50
  ) 个数;
  ```

- 查询有无名字叫"TOM"的员工信息

  ```sql
  SELECT EXISTS
  (
    SELECT *
    FROM employees
    WHERE name = "TOM"
  ) 有无;
  ```


## 连接查询

### 1. 说明

又称多表查询，当查询的字段来自于多个表时，就会用到连接查询。

### 2. sql连接查询分类

- 按年代分类：

  - sql92标准：`mysql`仅仅支持内连接。
  
  - sql99标准：`mysql`支持内连接+外连接（左外和右外）+交叉连接。

- 按功能分类：

  - 内连接：等值连接、非等值连接、自连接
  
  - 外连接：左外连接、右外连接、全外连接（`MySQL`不支持）
  
  - 交叉连接


## sql92内连接

### 1. 语法

  ```sql
  SELECT 查询列表
  FROM 表名1 别名1, 表名2 别名2...
  WHERE 连接条件
  ```

### 2. 说明

- 为了解决多表字段重名，可以为表起别名。

- 表的顺序无要求。

- n表连接，至少需要n-1个连接条件。

### 3. 示例

- 等值连接：查询部门编号`>100`的部门名和所在的城市名。

  ```sql
  SELECT d.department_name, l.city
  FROM departments d, locations l
  WHERE d.location_id = l.location_id AND d.department_id>100;
  ```

- 非等值连接：查询员工的工资和工资级别。

  ```sql
  SELECT e.salary, g.grade_level
  FROM employees e, job_grades g
  WHERE e.salary BETWEEN g.lowest_sal AND g.highest_sal;
  ```

- 自连接：查询员工名和上级的名称。

  ```sql
  SELECT e.employee_id, e.last_name, m.employee_id, m.last_name
  FROM employees e, employees m
  WHERE e.manager_id = m.employee_id;
  ```


## sql99内连接

### 1. 语法

```sql
SELECT 查询列表
FROM 表明1 别名
[INNER] JOIN 表明2 别名
ON 连接条件
WHERE 筛选条件
GROUP BY 分组列表
HAVING 分组后筛选
ORDER BY 排序列表;
```

### 2. 说明

- 使用关键字`JOIN`代替了逗号，连接条件和筛选条件进行了分离。

### 3. `straight_join`

`straight_join`与内连接类似，不同的地方在于，`straight_join`会强制使用左边的表作为驱动表，使用右边的表作为被驱动的表。

上述特性，在某些特定情况下可用于调优。

### 4. 示例

- 查询员工名和部门名

  ```sql
  SELECT e.last_name, d.department_name
  FROM employees e
  JOIN departments d
  ON e.department_id = d.department_id;
  ```

- 查询部门编号`>100`的部门名和所在的城市名

  ```sql
  SELECT d.department_name, e.city
  FROM departments d
  JOIN locations l
  ON d.location_id = l.location_id
  WHERE d.department_id > 100;
  ```

- 查询部门中员工个数`<10`的部门名，并按照员工个数降序

  ```sql
  SELECT COUNT(*) 员工个数, d.department_name
  FROM employees e
  JOIN departments d
  ON e.department_id = d.department_id
  GROUP BY d.department_id
  HAVING 员工个数 < 10
  ORDER BY 员工个数 DESC;
  ```

- 查询员工名和对应的领导名

  ```sql
  SELECT e.name, m.name
  FROM employees e
  JOIN employees m
  ON e.manager_id = m.employee_id;
  ```


## sql99外连接

### 1. 说明

- 查询结果为主表中所有的记录，如果从表有匹配项，则显示匹配项；如果从表没有匹配项，则显示null。

- 外连接主从表的顺序不可随意改变。

- 左连接的主表在左边，右连接的主表在右边。

- MySQL不支持全外连接。

### 2. 语法

```sql
SELECT 查询列表
FROM 表1 别名
LEFT|RIGHT|FULL [OUTER] JOIN 表2 别名
ON 连接条件
WHERE 筛选条件;
```


## sql99交叉连接

交叉连接用于返回多个表的笛卡尔积。

### 1. 语法

```sql
SELECT 查询列表
FROM 表1 别名1
CROSS JOIN 表2 别名2
WHERE 筛选条件;
```

或者

```sql
SELECT 查询列表
FROM 表1 别名, 表2 别名, ...
WHERE 筛选条件;
```


## 联合查询

### 1. 说明

- 当查询结果来自于多张表，但多张表之间没有关联，这时可用联合查询，也称union查询。

- `union`自动去重

- `union all`可以支持重复项

- 多条待联合的查询语句，查询列数必须一致，类型、字段最好一致

### 2. 语法

```sql
SELECT 查询列表 FROM 表1 WHERE 筛选条件
UNION
SELECT 查询列表 FROM 表2 WHERE 筛选条件
```

### 3. 示例

- 查询所有国家的年龄`>20`的用户信息

  ```sql
  SELECT * FROM chinese WHERE age > 20
  UNION
  SELECT * FROM usa WHERE uage > 20;
  ```

- 查询所有国家的用户姓名和年龄

  ```sql
  SELECT uname, uage FROM usa
  UNION
  SELECT name, age FROM chinese;
  ```

## MySQL语句处理顺序

`MySQL`完整的查询语句如下：

```sql
SELECT DISTINCT 字段
FROM table1
[LEFT|RIGHT|INNER] JOIN table2
ON 连接条件
WHERE 筛选条件
GROUP BY 分组字段
HAVING 过滤条件
ORDER BY 排序字段
LIMIT 偏移量;
```

执行顺序如下：

```sql
FROM <left_table>
ON <join_condition>
<join_type> JOIN <right_table>
WHERE <where_condition>
GROUP BY <group_by_list>
HAVING <having_condition>
SELECT
DISTINCT <select_list>
ORDER BY <order_by_condition>
LIMIT <limit_number>
```

## `JOIN`语句汇总

- `INNER JOIN`

  ```sql
  SELECT <select_list> FROM table A INNER JOIN table B ON A.key=b.key;
  ```

- `LEFT JOIN`

  ```sql
  SELECT <select_list> FROM table A LEFT JOIN table B ON A.key=B.key;
  ```

- `RIGHT JOIN`

  ```sql
  SELECT <select_list> FROM table A RIGHT JOIN table B ON A.key=B.key;
  ```

- `LEFT JOIN`

  ```sql
  SELECT <select_list> FROM table A LEFT JOIN table B ON A.key = B.key WHERE B.key IS NULL;
  ```

- `RIGHT JOIN`

  ```sql
  SELECT <select_list> FROM table A RIGHT JOIN table B ON A.key = B.key WHERE A.key IS NULL;
  ```

- `FULL OUTER JOIN`

  ```sql
  SELECT <select_list> FROM table A FULL OUTER JOIN table B ON A.key = B.key;
  ```

- `FULL OUTER JOIN`

  ```sql
  SELECT <select_list> FROM table A FULL OUTER JOIN table B ON A.key = B.key WHERE A.key IS NULL or B.key IS NULL;
  ```
