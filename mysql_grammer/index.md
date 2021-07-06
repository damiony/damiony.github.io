# 【MySQL】基本语法


## 概念

1. `DB`
数据库（database)：存储数据的仓库。它保存了一系列有组织的数据。

2. `DBMS`
数据库管理系统（database management system）。数据库是通过`DBMS`创建和操作的容器。

3. `SQL`
结构化查询语言（structure query language）:专门用来与数据库通信的语言。


## 存储数据的特点

1. 一个数据库可以有多个表，每个表有一个名字。表名具有唯一性。
2. 表具有一些特性，这些特性定义了数据在表中如何存储。
3. 表由列组成，列也称为字段。
4. 表中的数据是按行存储的。


## `SQL`分类

1. `DDL`: 数据定义语言，用来定义数据库对象（库、表、列等）。
2. `DML`: 数据操作语言，用来定义数据库记录（增、删、改）。 
3. `DCL`: 数据控制语言，用来定义访问权限和安全级别。
4. `DQL`: 数据查询语言，用来查询记录。


## rpm安装mysql

```bash
rpm -ivh rpm软件名
```


## Mysql核心目录

#### 1. 分类

1. `/var/lib/mysql`：安装目录
2. `/usr/share/mysql`：配置文件
3. `/usr/bin`：命令目录（mysqladmin、mysqldump等）
4. `/etc/init.d/mysql`：启动脚本

#### 2. 配置文件

1. `my-huge.cnf`：高端服务器
2. `my-large.cnf`：中等规模
3. `my-medium.cnf`：一般
4. `my-small.cnf`：较小

以上配置文件默认不能识别，mysql5.5默认只能识别 `/etc/my.cnf`，mysql5.6默认识别 `/etc/mysql-default.cnf`。


## mysql字符编码

#### 1. 查看语法

```mysql
show variables like '%char%';
```

#### 2. 设置编码

修改`/etc/my.cnf`

```bash
[mysql]
default-character-set=utf8
   
[client]
default-character-set=utf8
   
[mysqld]
character_set_server=utf8
character_set_client=utf8
collation_server=utf8_general_co
```

修改编码，只对之后创建的数据库生效。


## 查看表的结构

#### 1. 语法

```mysql
DESC name;
```

或者：

```mysql
SHOW COLUMNS FROM tablename;
```

或者：

```mysql
SHOW CREATE TABLE tablename;
```


## 查询语句

#### 1. 语法

```mysql
select 查询列表 from 表名;
```

#### 2. 执行顺序：

- `from`子句

-  `select`子句

#### 3. 查询列表：

- 字段

```mysql
SELECT `last_name` FROM `employees`
```

- 表达式

```mysql
SELECT 100 % 3;
```

- 常量

```mysql
SELECT 100;
```

- 函数等

```mysql
SELECT DATABASE();
SELECT VERSION();
SELECT USER();
```

#### 4. 起别名

- 使用`as`关键字

```mysql
SELECT USER() as "username";
```

- 使用空格

```mysql
select user() username;
```

#### 5. mysql中+的作用

- 如果两个数都是数值型，则直接相加。

- 如果其中一个操作数为字符型，则将字符型转换成数值型。如果无法转换，则直接当作`0`处理。

- 其中一个操作数为null，结果为null。 

#### 6. distinct函数

```mysql
SELECT DISTINCT field FROM tablename
```

#### 7. ifnull函数

语法：

```mysql
ifnull(表达式1, 表达式2)
```

功能：

如果表达式1为`null`，则显示表达式2，否则显示表达式1。


## 条件查询

#### 1. 语法

```mysql
select 查询列表
from 表名
where 筛选条件；
```

#### 2. 执行顺序

- `from`子句。

- `where`子句。

- `select`子句。

#### 3. 筛选条件

- 按关系表达式筛选：`<`、`>`、`>=`、`<=`、`=`、`<>`，也可以使用 `!=` 但是不建议，`ANSI`标准中用的是`<>`。

- 按逻辑表达式筛选：`and`、`or`、`not`，也可以使用 `&&`、`||`、`!` 但是不建议。

- 模糊查询：`like`、`in`、`between ... and ...`、`is null`。

#### 4. 示例

- 查询部门编号不是 `50 ~ 100` 之间员工的姓名:

```mysql
select name from employees where department_id < 50 or department_id > 100;
```

- 查询姓名中包含字符`a`的员工。

```mysql
select * from employees where name like '%a%';
```

- 查询姓名最后一个字符为`a`的员工。

```mysql
select * from employees where name like '%a';
```

- 查询姓名第一个字符为`a`的员工。

```mysql
select * from employees where name like 'a%';
```

- 查询姓名中第三个字符为`a`的员工。

```mysql
select * from employees where name like '___a%'; # 有三个下划线_
```

- 查询姓名中第二个字符为`_`的员工信息。

```mysql
select * from employees where name like '_\_%';
# 或者
# select * from employees where name like '_$_%' escape '$';
```

- 查询部门编号是30/50/90的员工名

```mysql
select name from employees where department in (30, 50, 90);
```

- 查询部门编号是30～90之间的员工姓名

```mysql
select name from employees where department between 30 and 90;
```

- 查询部门编号不是30～90之间的员工姓名

```mysql
select name from employees where department not between 30 and 90;
```


## 排序查询

#### 1. 语法

```mysql
select 查询列表
from 表名
where 筛选条件
order by 排序列表
```

#### 2. 执行顺序

- `from`子句

- `where`子句

- `select`子句

- `order by`子句

#### 3. 说明

- 排序列表可以是单个字段、多个字段、表达式、函数、列数、以及以上的组合。

- 默认升序asc，降序desc

#### 4. 示例

- 将员工按照工资降序

```mysql
select * from employees order by salary desc;
```

- 按姓名长度进行升序

```mysql
select name from employees order by length(name);
```

- 查询员工信息，先按工资升序、再按部门将序

```mysql
select * from employees order by salary asc, department desc;
```

- 按照列数进行排序

```mysql
select * from employees order by 2 desc;
```


## 常见函数

#### 1. 分类

- 字符函数

- 数学函数

- 日期函数

- 流程控制函数

#### 2. 字符函数

- `concat`拼接字符

```mysql
select concat("hello", name) from user;
```

- `LENGTH`获取字节长度

```mysql
select length("hello,少年");
```

- `CHAR_LENGTH`获取字符长度

```mysql
SELECT CHAR_LENGTH("hello,少年");
```

- `SUBSTR`截取子串

```mysql
SELECT SUBSTR('少年你好骚啊',1,2);
# SELECT SUBSTR('你好骚啊少年',5);
```

- `INSTR`获取字符第一次出现的索引

```mysql
SELECT INSTR('孙悟空三打白骨精','白骨精');
```

- `TRIM`去除前后指定字符，默认去空格

```mysql
SELECT TRIM(' 空  格 ') as result;
```

```mysql
SELECT TRIM('x' FROM 'xxx空xxx格xxx') as result;
```

- `LPAD/RPAD`左填充/右填充

```mysql
# 左填充满10个字符
SELECT LPAD('木婉清', 10, 'a');
# 右填充满10个字符
SELECT RPAD('木婉清', 10, 'a');
```

- `UPPER/LOWER`变大写/变小写

- `STRCMP`比较两个字符大小

```mysql
SELECT STRCMP('abc','acb')
```

- `LEFT/RIGHT`截取子串

```mysql
SELECT LEFT('洪世贤',1);
```

#### 3. 数学函数

- `ABS`绝对值

```mysql
SELECT ABS(-1.1);
```

- `CELL`向上取整，返回大于等于该参数的最小整数

```mysql
SELECT CELL(1.09);
```

- `FLOOR`向下取整，返回小于等于该参数的最大整数

```mysql
SELECT FLOOR(-1.09);
```

- `ROUND`四舍五入

```mysql
SELECT ROUND(1.8765);
```

- `TRUNCATE`截断

```mysql
SELECT TRUNCATE(1.8765, 1); # 位数不够用0补齐
```

- `MOD`取余，实质上 `a % b = a - a/b*b`。

```mysql
SELECT MOD(-10, 3);
# SELECT -10%3;
# SELECT 10%3;
# SELECT -10%-3;
# SELECT 10%-3;
```

#### 4. 日期函数

- `NOW`

```mysql
SELECT NOW();
```

- `CURDATE`

```mysql
SELECT CURDATE();
```

- `CURTIME`

```mysql
SELECT CURTIME();
```

- `DATEDIFF`

```mysql
SELECT DATEDIFF('1998-7-16', '2019-7-13');
```

- `DATE_FORMAT`

```mysql
SELECT DATE_FORMAT('1998-7-16', '%Y年%M月%d日 %H小时%i分钟%s秒');
```

- `STR_TO_DATE`

```mysql
SELECT STR_TO_DATE('3/15 1998', '%m/%d %Y');
```

#### 5. 流程控制函数

- `IF`函数

```mysql
SELECT IF(100>9, "好", "坏");
```

- `CASE`函数

```mysql
CASE 表达式
WHEN 值1 THEN 结果1
WHEN 值2 THEN 结果2
...
ELSE 结果n
END
```

或者

```mysql
CASE
WHEN 条件1 THEN 结果1
WHEN 条件2 THEN 结果2
...
ELSE 结果N
END
```


## 分组函数

#### 1. 说明

往往用于实现将一组数据进行统计计算，最终得到一个值，又称为聚合函数或者统计函数。

#### 2. 列表

- `sum(字段名)`: 求和

- `avg(字段名)`: 求平均数

- `max(字段名)`: 求最大值

- `min(字段名)`: 求最小值

- `count(字段名)`: 计算非空字段值的个数

#### 3. 补充说明

- `count(*)`用于查询总行数。

- `count(1)`作用类似于`count(*)`，但是效率较低。

- `count()`搭配`distinct`做去重统计，如

```mysql
select count(distinct department) from employees;
```

#### 4. 示例

- 查询员工信息表中，所有员工的工资和、平均工资值、最低工资、最高工资、有工资的个数。

```mysql
select sum(salary), avg(salary), min(salary), max(salary), count(salary) from employees;
```


## 分组查询

需要使用`group by`子句。

#### 1. 语法

```mysql
select 查询列表
from 表名
where 筛选条件
group by 分组列表
having 分组后筛选
order by 排序列表;
```

#### 2. 执行顺序

- `from`子句

- `where`子句

- `group by`子句

- `having`子句

- `select`子句

- `order by`子句

#### 3. 说明

- 查询列表往往是分组函数和被分组的字段。

- 分组前筛选，基于原始表，使用`where`，位于`group by`前面。

- 分组后筛选，基于分组后的结果集，使用`having`，位于`group by`后面。

#### 4. 示例

- 查询每个工种员工平均工资

```mysql
select avg(salary), job_id from employees group by job_id;
```


## 连接查询

#### 1. 说明

又成多表查询，当查询的字段来自于多个表时，就会用到连接查询。

#### 2. sql连接查询分类

- 年代：
  - sql92标准：`mysql`仅仅支持内连接
  - sql99标准：`mysql`支持内连接+外连接（左外和右外）+交叉连接

- 功能：
  - 内连接：等值连接、非等值连接、自连接
  - 外连接：左外连接、右外连接、全外连接
  - 交叉连接


## sql92内连接

#### 1. 语法

```mysql
select 查询列表
from 表名1 别名1, 表名2 别名2...
where 连接条件
```

#### 2. 说明

- 为了解决多表字段重名，可以为表起别名。

- 表的顺序无要求。

- n表连接，至少需要n-1个连接条件。

#### 3. 示例

- 等值连接：查询部门编号`>100`的部门名和所在的城市名。

```mysql
SELECT `department_name`, `city`
FROM departments d,locations l
WHERE d.`location_id` = l.`location_id`
AND d.`department_id`>100;
```

- 非等值连接：查询员工的工资和工资级别。

```mysql
SELECT salary, grade_level
FROM employees e,job_grades g
WHERE salary BETWEEN g.`lowest_sal`
AND g.`highest_sal`;
```

- 自连接：查询员工名和上级的名称。

```mysql
SELECT e.employee_id, e.last_name, m.employee_id, m.last_name
FROM employees e, employees m
WHERE e.`manager_id`=m.`employee_id`;
```


## sql99内连接

#### 1. 语法

```mysql
SELECT 查询列表
FROM 表明1 别名
[INNER] JOIN 表明2 别名
ON 连接条件
WHERE 筛选条件
GROUP BY 分组列表
HAVING 分组后筛选
ORDER BY 排序列表;
```

#### 2. 说明

- 使用关键字`JOIN`代替了逗号，连接条件和筛选条件进行了分离。

#### 3. 示例

- 查询员工名和部门名

```mysql
SELECT last_name, department_name
FROM employees e
JOIN departments d
ON e.department_id=d.department_id;
```

- 查询部门编号`>100`的部门名和所在的城市名

```mysql
SELECT department_name, city
FROM departments d
JOIN locations l
ON d.`location_id` = l.`location_id`
WHERE d.`department_id`>100;
```

- 查询部门中员工个数`<10`的部门名，并按照员工个数降序

```mysql
SELECT COUNT(*) 员工个数, d.department_name
FROM employees e
JOIN departments d
ON e.`department_id`=d.`department_id`
GROUP BY d.`department_id`
HAVING 员工个数>10
ORDER BY 员工个数 DESC;
```

- 查询部门员工的工资级别，并按照级别进行分组

```mysql
SELECT COUNT(*) 个数, g.grade
FROM employees e
JOIN sal_grage g
ON e.`salary` BETWEEN g.`min_salary` AND g.`max_salary`
GROUP BY g.grade
```

- 查询员工名和对应的领导名

```mysql
SELECT e.`name`, m.`name`
FROM employees e
JOIN employees m
ON e.`manager_id`=m.`employee_id`
```


## SQL99外连接

#### 1. 说明

- 查询结果为主表中所有的记录，如果从表有匹配项，则显示匹配项；如果从表没有匹配项，则显示null。

- 一般用于查询主表中有但从表中没有的记录。

- 外连接主从表的顺序不可随意改变。

- 左连接的主表在左边。

- 右连接的主表在右边。

#### 2. 语法

```mysql
select 查询列表
from 表1 别名
left|right|full [outer] join 表2 别名
on 连接条件
where 筛选条件;
```


## 子查询

#### 1. 定义

当一个查询语句中，又嵌套了另一个完整的`select`语句，则被嵌套的语句称为子查询或内查询。

外面的语句称为主查询或外查询。

#### 2. 说明

- 子查询不一定必须出现在`select`子句中，只是这种情况比较常见。

- 子查询放在条件中，要求必须出现在条件右侧

- 子查询一般放在小括号中

- 单行子查询对应了单行操作符：`>, <, >=, <=, =, <>`

- 多行子查询对应了多行操作符：`any, some, all, in`（`some`和`any`的用法一致）

#### 3. 分类

按出现位置：

- `select`后面：子查询结果必须单行单列（标量子查询）
- `from`后面：子查询结果可以多行多列
- `where`或者`having`后面：子查询的结果必须为单列

#### 4. 示例

- 查询和Tom同部门的员工姓名和工资

```mysql
SELECT name, salary
FROM employees
WHERE department_id = (
	SELECT department_id
    FROM employees
    WHERE name = 'Tome'
);
```

- 查询部门编号是50的员工个数

```mysql
SELECT
(
    SELECT COUNT(*)
    FROM employees
    WHERE department_id = 50
) 个数;
```

- 查询有无名字叫"TOM"的员工信息

```mysql
SELECT EXISTS
(
    SELECT *
    FROM employees
    WHERE name = "TOM"
) 有无;
```


## sql99交叉连接

#### 1. 语法

```mysql
select 查询列表
from 表1 别名1
cross join 表2 别名2
where 筛选条件;
```

或者

```mysql
select 查询列表
from 表1 别名, 表2 别名, ...
where 筛选条件;
```


## 分页查询

#### 1. 语法

```mysql
select 查询列表
from 表1  别名
join 表2  别名
on 连接条件
where 筛选条件
group by 分组
having 分组后筛选
order by 排序列表
limit 起始条目索引，条目数
```

#### 2. 执行顺序

`from -> on -> join -> where -> group by -> having -> select -> order by -> limit`

#### 3. 说明

起始条目索引从0开始，缺省则默认为0

#### 4. 示例

- 查询员工信息表的前5条

```mysql
SEELCT * FROM employees LIMIT 5;
# 等价于
# SELECT * FROM employees LIMIT 0,5;
```


## 联合查询

#### 1. 说明

- 当查询结果来自于多张表，但多张表之间没有关联，这时可用联合查询，也称union查询。

- `union`自动去重

- `union all`可以支持重复项

- 多条待联合的查询语句，查询列数必须一致，类型、字段最好一致

#### 2. 语法

```mysql
select 查询列表 from 表1 where 筛选条件
union
select 查询列表 from 表2 where 筛选条件
```

#### 3. 示例

- 查询所有国家的年龄`>20`的用户信息

```mysql
SELECT * FROM chinese WHERE age > 20 UNION
SELECT * FROM usa WHERE uage > 20;
```

- 查询所有国家的用户姓名和年龄

```mysql
SELECT uname, uage FROM usa
UNION
SELECT age, `name` FROM chinese;
```


## DDL语言

#### 1. 说明

`DDL`: （Data Define Language）数据定义语言，用于对数据库和表的管理和操作。

#### 2. 库管理

- 创建数据库

```mysql
CREATE DATABASE 库名;
# CREATE DATEBASE IF NOT EXISTS 库名; # 如果不存在则创建
```

- 删除数据库

```mysql
DROP DATABASE 库名;
# DROP DATABASE IF EXISTS 库名; # 如果存在则删除
```

#### 3. 表管理

- 创建表

语法：

```mysql
CREATE TABLE [IF NOT EXISTS] 表名(
    字段名 字段类型 [字段约束]，
    ......
    字段名 字段类型 [字段约束],
    字段名 字段类型 [字段约束]
);
```

示例：

```mysql
CREATE TABLE stuinfo(
    stuid INT,
    stuname VARCHAR(20),
    stugender CHAR,
    email VARCHAR(20),
    age INT,
    majorid INT,
    CONSTRAINT fk_stuinfo_major FOREIGN KEY (majorid) REFERENCES major(id)
);
```

- 修改表

修改表名：

```mysql
ALTER TABLE 原表名 RENAME TO 新表名;
```

修改表字段：

```mysql
ALTER TABLE 表名 ADD|MODIFY|CHANGE|DROP COLUMN 字段名 字段类型 字段约束;
```

示例：

```mysql
ALTER TABLE stuinfo RENAME TO students;
```

```mysql
ALTER TABLE students ADD COLUMN dorndate TIMESTAMP NOT NULL;
```

```mysql
ALTER TABLE students CHANGE COLUMN borndate birthday DATETIME NULL;
```

```mysql
ALTER TABLE students MODIFY COLUMN birthday TIMESTAMP;
```

```mysql
ALTER TABLE students DROP COLUMN birthday;
```

- 删除表

```mysql
DROP TABLE IF EXISTS tablename;
```

- 复制表

```mysql
CREATE TABLE tablename1 LIKE tablename2; # 仅复制表结构
```

```mysql
CREATE TABLE tablename1 SELECT * FROM tablename2; # 复制结构和数据
```

示例：

```mysql
CREATE TABLE emp
SELECT name
FROM employees
WHERE 1=2;
```


## 常用字段类型

#### 1. int

整形：`TYNYINT, SMALLINT, INT, BIGINT`。

#### 2. double/float

浮点型，例如double(5, 2)表示最多5位，其中有且仅有2位小数，即最大值为999.99 

#### 3. decimal

浮点型，表示钱时使用该类型，不会出现精度问题

#### 4. char

固定长度字符串类型；内容范围是0~255字节；

`char(n)`：n参数默认为1，不管实际存储，开辟空间都是`n`个字符，效率高。

#### 5. varchar

可变长度字符串类型；内容范围为0~65535字节；

`varchar(n)`最大字符个数必选，根据实际存储空间决定开辟空间，效率低。

实际占用空间为：字符实际空间 + 1，且字符实际空间 + 1 <= n。

#### 6. text

字符串类型，通常存储较长文本。

#### 7. blob

字节类型；常用于`jpg, mp3, avi`。

#### 8. date

日期类型；格式为`yyyy-MM-dd`

#### 9. time

时间类型；格式为`hh:mm:ss`

#### 10. timestamp/datetime

时间戳类型；格式为`yyyyMMdd hhmmss`；

`timestamp`保存范围`1900-1-1~xxxx年`，占用8字节；

`datetime`保存范围`1970-1-1~2038-12-31`，占用4字节。


## 常见字段约束

#### 1. 说明

用于限制表中字段的数据；

#### 2. 列表

- `NOT NULL` 

  非空；用于限制该字段为必填项；

- `DEFAULT` 

  默认；如果该字段没有插入值，则显示默认值；

- `PRIMARY KEY` 

  主键；用于限制该字段的值不能重复；设为主键列的字段默认不能为空；

  ```mysql
  ALTER TABLE tbl_name ADD PRIMARY KEY (column_list);
  ```

- `UNIQUE` 

  唯一；限制该字段不能重复；该字段可以为空；

- `CHECK` 

  检查；限制该字段值必须满足指定条件；`mysql`不支持；

- `FOREIGN KEY` 

  外键；限制两个表的关系；外键列的值必须来自主表的关联列；主表关联列和从表关联列类型必须一致，意思一样，名称无要求；主表的关联列要求必须是主键；

  定义外键：

  ```mysql
  alter table teblename add constraint 外键名 foreign key(列名) references 表名（关联列）;
  ```

  或者：

  ```mysql
  create table teblename (
  	...
  	constraint 外键名 foreign key(列名) references 表名(关联列)
  );
  ```

  删除外键：

  ```mysql
  alter table tablename drop foreign key 键名
  ```


## 设置自增长列

#### 1. 关键字

`AUTO_INCREMENT`

#### 2. 说明

- 自增长列必须设置在一个键上，比如主键或唯一键。

- 自增长列要求数据类型为数值型。

- 一个表至多有一个自增长列。


## DML

`DML`: (Data Manipulation Language) 数据操作语言

   1. 插入数据
   2. 修改数据
   3. 删除数据   


## 插入数据

#### 1. 语法

- 插入单行

```mysql
insert into 表名(字段名1，字段名2，......) values(值1，值2，......);
```

- 插入多行

```mysql
insert into 表名（字段名1，字段名2，......) values(值1，值2，......),(值1，值2，......)...;
```

#### 2. 说明

- 字段和值列表一一对应。

- 数值型值不用单引号，非数值型值必须使用单引号。

#### 3. 示例

```mysql
INSERT INTO stuinfo (stuid, stuname, stugender, email, age, majorid)
VALUES(1, 'TOm', '男', 'tom@gmail.com', 12, 1);
```


## 修改数据

#### 1. 语法

- 修改单表：

```mysql
update 表名
set 列=新值, 列=新值,...
where 筛选条件;
```

- 修改多表，sql92语法：

```mysql
update 表名1 别名，表名2 别名
set 列=值, ...
where 连接条件
and 筛选条件;
```

- 修改多表，sql99语法：

```mysql
update 表1 别名
inner|left|right join 表2 别名
on 连接条件
set 列=值,...
where 筛选条件;
```


## 删除数据

#### 1. 语法

- delete语句：

```mysql
delete from 表名 where 筛选条件;
```

- truncate语句:

```mysql
truncate table 表名;
```

#### 2. delete和truncate的区别：

- delete可以添加where条件，truncate不能添加where条件，一次性清除所有数据。

- truncate的效率较高。

- 如果删除带自增长列的表；使用delete删除后，重新插入数据时，记录从断点处开始；使用truncate删除后，重新插入数据时，记录从1开始。

- delete删除数据时，会返回受影响的行数；truncate删除数据时，不会返回受影响的行数。

- delete删除数据时，可以支持事务回滚；truncate删除数据时，不支持事务回滚。


## 事务

#### 1. 概念：

一个事务是由一条或者多条sql语句构成，这一条或者多条sql语句，要么全部执行成功，要么全部执行失败。

#### 2. 四大特性：

- 原子性 (atomicity)：事务中所有操作是不可再分割的原子单位。事务中所有操作要么全部执行成功，要么全部执行失败。

- 一致性 (consistency)：事务执行后，数据库状态与业务规则保持一致。如转账业务，无论事务执行成功与否，参与转账的两个账号余额之和应该是不变的。

- 隔离性 (isolation)：在并发操作中，不同事务之间应该隔离开来，使每个并发中的事务不会相互干扰。

- 持久性 (durability)：一旦事务提交成功，事务中所有的数据操作都必须被持久化到数据库。即使提交事务后，数据库马上崩溃，在数据库重启时，也能通过某种机制恢复数据。

#### 3. 分类

   1. 隐式事务：没有明显的开启和结束标记

   2. 显式事务：具有明显的开启和结束标记

      步骤：取消隐式事务自动开启的功能 => 开启事务 => 编写事务语句 => 结束事务

#### 4. 示例

```mysql
#取消事务自动开启
SET autocommit = 0;
#开启事务 有的客户端不用加这句
START TRANSACTION;
#更新数据
UPDATE stuinfo SET balance=balance-5000 where stuid = 1;
UPDATE stuinfo SET balance=balabce+5000 where stuid = 2;
#提交
COMMIT;
#或者回滚
#ROLLBACK;
```


## 数据库的隔离级别

#### 1. 说明

对于同时运行的多个事务，当这些事务访问数据库中相同的数据时，如果没有采取必要的隔离机制，就会导致各种并发问题：

- 脏读：对于两个事务T1和T2，T1读取了已经被T2更新但还没有被提交的字段，若之后T2回滚，T1读取的内容就是临时且无效的。
- 不可重复读：对于两个事务T1和T2，T1读取了一个字段，然后T2更新了该字段，之后T1再次读取同一个字段，获得的值就不同了。
- 幻读：对于两个事务T1和T2，T1从一个表中根据某些条件读取出一些记录，然后T2向该表插入了符合条件的新纪录，T1再次读取时，就会把新纪录也读出来。

一个事务与其他事务隔离的程度称为隔离级别。不同隔离级别对应不同的干扰程度，隔离级别越高，数据一致性就越好，但并发性越弱。


## 数据的4种隔离级别

#### 1. 隔离级别说明

- 读未提交数据(READ UNCOMMITTED)：允许事务读取未被其他事务提交的变更。脏读，不可重复读和幻读的问题都会出现。

- 读已提交数据(READ COMMITTED)：只允许事务读取已经被其他事务提交的变更。可以避免脏读，但不可重复读和幻读问题仍然会出现。

- 可重复读(REPEATABLE READ)：确保事务可以多次从一个字段中读取相同的值，但这个事务持续期间，禁止其他事务对这个字段进行更新。可以避免脏读和不可重复读，但幻读的问题仍然存在。

- 串行化(SERIALIZABLE)：确保事务可以从一个表中读取相同的行，在这个事务持续期间，禁止其他事务对该表执行插入、更新和删除操作，所有并发问题都可以避免，但性能十分低下。

#### 2. Oracle支持的事务隔离级别：

- `READ COMMITED`（默认）

- `SERIALIZABLE`

#### 3. Mysql支持的事务隔离级别

- `READ UNCOMMITTED`

- `READ COMMITTED`

- `REPEATABLE READ`（默认）

- `SERIALIZABLE`


## MySql设置隔离级别

#### 1. 说明

每启动一个mysql客户端程序，就会获得一个单独的连接，每个连接都有一个变量`@@tx_isolation`表示当前的事务隔离级别。

#### 2. mysql查看当前连接的隔离级别

```mysql
select @@tx_isolation;
```

#### 3. mysql修改当前连接的隔离级别

当前连接的隔离级别不会影响到其他连接。

```mysql
set session transaction isolation level read uncommitted;
```

#### 4. mysql设置数据库系统全局的隔离级别

需要重新打开`mysql`会话，设置才会生效。

```mysql
set global transaction isolation level read committed;
```


## SAVEPOINT

#### 1. 说明

只能搭配`rollback`使用

#### 2. 示例

```mysql
SET autocommit=0;
DELETE FROM account WHERE id=25;
SAVEPOINT a;
DELETE FROM account WHERE id=28;
ROLLBACK TO a;
```


## 视图

#### 1. 含义

- 虚拟表，和普通表一样使用。

- `mysql5.1`版本之后出现的新特性。

- 行和列的数据，来自定义视图时的查询中使用到的表，并且是在使用视图时动态生成的。

- 只保存了sql逻辑，不保存查询结果。

#### 2. 应用场景

- 多个地方用到同样的查询结果。

- 该查询结果使用的sql语句较复杂。


## 创建视图

#### 1. 语法

```mysql
create view 视图名
as 查询语句;
```

#### 2. 示例

- 查询邮箱中包含a字符的员工名、部门名和工种信息。

```mysql
#创建视图
CREATE VIEW myv1
AS
SELECT name, department, title
FROM employees  e
JOIN departments  d ON e.department_id = d.department_id
JOIN jobs  j ON j.job_id = e.job_id;
   
#使用视图
SELECT * FROM myv1 WHERE name LIKE "%a%";
```

- 查询各部门的平均工资级别

```mysql
#创建视图
CREATE VIEW myv2
AS
SELECT AVG(salary) ag, department_id
FROM employees
GROUP BY department_id;

#使用
SELECT myv2.`ag`, g.grade_level
FROM myv2
JOIN job_grades g
ON myv2.`ag` BETWEEN g.`lowest_sal` AND g.`highest_sal`;
```


## 修改视图

#### 1. 语法

方式一：

```mysql
create or replace view 视图名
as
查询语句;
```

方式二：

```mysql
alter view 视图名
as
查询语句;
```


## 删除视图

#### 1. 语法

```mysql
drop view 视图名，视图名......;
```


## 查看视图

#### 1. 语法

```mysql
desc 视图名;
```

或者

```mysql
show create view 视图名;
```


## 视图内容的更新

#### 1. 插入

- 同表的插入语法一致。

- 会更新原始表。

#### 2. 修改

- 同表的修改语法一致。

- 会更新原始表。

#### 3. 删除

- 使用`delete`语法。

- 会更新原始表。

#### 4. 说明

视图的可更新性和视图中查询的定义有关系，以下类型的视图不能更新：

- 包含关键字：分组函数、distinct、group by、having、union、union all

- 常量视图

- select子句中包含子查询

- join（某些条件下可更新）

- from一个不能更新的视图

- where子句的子查询引用了from子句中的表


## 变量

#### 1. 系统变量

- 全局变量

- 会话变量

#### 2. 自定义变量

- 用户变量

- 局部变量


## 系统变量

#### 1. 含义

变量由系统提供，属于服务器层面。

#### 2. 语法

- 查看所有的系统变量

```mysql
show global | [session] variables;
```

- 查看满足条件的部分系统变量

```mysql
show global | [session] variables like '%char%';
```

- 查看指定的某个系统变量的值

```mysql
select @@global.|[session.]系统变量名;
```

- 为某个系统变量赋值

```mysql
set global | [session] 系统变量名 = 值; 
```

或者

```mysql
set @@global.|[session.]系统变量名 = 值;
```

#### 3. 注意

- 如果是全局级别，需要使用global

- 如果是会话级别，需要加session或者什么都不加

- 全局变量不能跨重启，即服务器重启后变量值恢复默认


## 自定义变量

#### 1. 含义

用户自定义的，不是系统提供的。

#### 2. 用户变量

- 作用域：当前会话（连接）有效。

- 声明并初始化：

`set @变量名=值;`

或者

`set @变量名:=值;`

又或者

`select @变量名:=值;`

- 赋值：

```mysql
set @变量名=值;
set @变量名:=值;
select @变量名:=值;
```

或者

```mysql
select 字段 into @变量名
from 表;
```

- 使用

```mysql
select @变量名;
```

#### 3. 局部变量

- 作用域：仅仅在定义它的begin end中有效

- 声明：

```mysql
declare 变量名 类型;
```

或者

```mysql
declare 变量名 类型 default 值;
```

- 赋值：

```mysql
set 变量名=值;
set 变量名:=值;
select @变量名:=值;
```

或者

```mysql
select 字段 into 变量名
from 表;
```

- 使用：

```mysql
select 变量名;
```

- 应用：

begin中的第一句话。


## 存储过程

#### 1. 好处

1. 提高了代码的重用性
2. 简化操作
3. 减少了编译次数，减少了和服务器的连接次数，提高了效率

#### 2. 含义

一组预先编译好的sql语句的集合。

#### 3. 创建

```mysql
create procedure 过程名(参数列表)
begin
存储过程体
end
```

注意：

- 参数列表包含三部分：参数模式、参数名、参数类型

- 参数模式：

  - IN：输入

  - OUT：输出

  - INOUT：输入和输出

- 如果存储过程体只有一句话，begin、end可以省略。

- 存储过程体每句都以分号结束。

- 存储过程体的结尾使用delimiter重新设置。

#### 4. 调用

```mysql
call 存储过程名(实参列表);
```

#### 5. 删除

```mysql
DROP PROCEDURE 过程名;
```

#### 6. 查看

```mysql
SHOW CREATER PROCEDURE 过程名
```

#### 7. 修改

存储过程无法修改。

#### 8. 示例

- 无参

```mysql
#定义
DELIMITER $
CREATE PROCEDURE myp1()
BEGIN
    INSERT INTO admin(username, `password`)
    VALUES('john1', '0000'), ('lily', '0000');
END $
#调用
CALL myp1()$
```

- IN

```mysql
DELIMITER $
CREATE PROCEDURE myp2(IN beautyName VARCHAR(20))
BEGIN
    SELECT bo.*
    FROM boys bo
    RIGHT JOIN beauty b ON bo.id = b.boyfriend_id
    WHERE b.name=beautyName
END $
```

- IN

```mysql
#定义
DELIMITER $
CREATE PROCEDURE myp4(IN username VARCHAR(20), IN PASSWORD VARCHAR(20))
BEGIN
    DECLARE result INT DEFAULT 0;
    SELECT COUNT(*) INTO result
    FROM admin
    WHERE admin.username = username
    AND admin.password = PASSWORD;      	
    SELECT IF(result>0,'成功','失败');
END $

#调用
CALL myp4('张飞', '8888')$
```

- OUT

```mysql
#定义
DELIMITER $
CREATE PROCEDURE myp5(IN beautyName VARCHAR(20), OUT boyName VARCHAR(20))
BEGIN
    SELECT bo.boyName INTO boyName
    FROM boys bo
    INNER JOIN beauty b ON bo.id = b.boyfriend_id
    WHERE b.name=beautyName;
END $

#调用
[SET @bName$]
CALL myp5('小昭', @bName)$
SELECT @bName$
```

- INOUT

```mysql
#定义
DELIMITER $
CREATE PROCEDURE myp6(INOUT a INT, INOUT b INT)
BEGIN
    SET a=a*2
    SET b=b*2
END $

#调用
SET @m=10$
SET @n=20$
CALL myp6(@m, @n)$
```


## 函数

函数和好处同存储过程。

#### 1. 说明

- 有且仅有一个返回。

#### 2. 创建

```mysql
CREATE FUNCTION 函数名（参数列表） RETURNS 返回类型
BEGIN
函数体
END
```

**注意**：

1. 参数列表包含两部分：参数名 参数类型。
2. 函数体中必须包含return。
3. 函数体仅有一句话，则可以省略begin和end。
4. 使用delimiter语句设置结束标记。

#### 2. 调用

```mysql
SELECT 函数名（参数列表）
```

#### 3. 查看函数

```mysql
SHOW CREATE FUNCTION 函数名;
```

#### 4. 删除函数

```mysql
DROP FUNCTION 函数名;
```

#### 5. 示例

- 无参有返回

```mysql
DELIMITER $
CREATE FUNCTION myf1() RETURNS INT
BEGIN
    DECLARE c INT DEFAULT 0;
    SELECT COUNT(*) INTO c
    FROM employees;
    RETURN c;
END $
      
SELECT myf1()$
```

- 有参有返回

```mysql
DELIMITER $
CREATE FUNCTION myf2(empName VARCHAR(20)) RETURNS DOUBLE
BEGIN
    SET @sal=0;
    SELECT salary INTO @sal
    FROM employees
    name=empName;
    RETURN @sal
END $
      
SELECT myf2('tom')$
```

## 流程控制结构

#### 1. 顺序结构

程序从上往下依次执行

#### 2. 分支结构

从多条路径中选择一条去执行

#### 3. 循环结构

在满足一定条件的基础上，重复执行一段代码

## 分支结构

#### 1. if函数

```mysql
IF(表达式1，表达式2，表达式3)
```

如果表达式1成立，则if函数返回表达式2的值，否则返回表达式3的值。

#### 2. case结构

```mysql
CASE 变量|表达式|字段
WHEN 要判断的值 THEN 返回的值1或语句;
WHEN 要判断的值 THEN 返回的值2或语句;
...
ELSE 要返回的值n或语句;
END;
```

或者

```mysql
CASE
WHEN 要判断的条件1 THEN 返回的值1或语句;
WHEN 要判断的条件2 THEN 返回的值2或语句;
...
ELSE 要返回的值n或语句;
END CASE;
```

#### 3. if结构

```mysql
if 条件1 then 语句1;
elseif 条件2 then 语句2;
...
else 语句n;
end if;
```

应用在`begin/end`中。

#### 4. 示例

- case

```mysql
DELIMITER $
CREATE PROCEDURE test_case(IN score INT)
BEGIN
    CASE
    WHEN score>=90 AND score<=100 THEN SELECT 'A';
    WHEN score>=80 THEN SELECT 'B';
    WHEN score>=60 THEN SELECT 'C';
    ELSE SELECT 'D';
    END CASE;
END $
```

- if结构

```mysql
DELIMITER $
CREATE FUNCTION test_if(score INT) RETURNS CHAR
BEGIN
    IF score>=90 AND score<=100 THEN RETURN 'A';
    ELSEIF score>=80 THEN RETURN 'B';
    ELSE RETURN 'D';
    END IF;
END $
```

## 循环结构

#### 1. 分类

- while

- loop

- repeat

#### 1. 循环控制语句

- iterate 类似于 continue
- leave 类似于 break

#### 2. 语法

```mysql
[标签:]while 循环条件 do
循环体;
end while [标签];
```

```mysql
[标签:]loop
循环体;
end loop [标签];
```

```mysql
[标签:]repeat
循环体;
until 结束循环的条件
end repeat [标签];
```

## MySQL语句处理顺序

```mysql
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

## join语句汇总

- `INNER JOIN`

```mysql
SELECT <select_list> FROM table A INNER JOIN table B ON A.key=b.key;
```

- `LEFT JOIN`

```mysql
SELECT <select_list> FROM table A LEFT JOIN table B ON A.key=B.key;
```

- `RIGHT JOIN`

```mysql
SELECT <select_list> FROM table A RIGHT JOIN table B ON A.key=B.key;
```

- `LEFT JOIN`

```mysql
SELECT <select_list> FROM table A LEFT JOIN table B ON A.key = B.key WHERE B.key IS NULL;
```

- `RIGHT JOIN`

```mysql
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


## 不需要创建索引的情况

1. 表记录太少
2. 频繁更新的字段不适合创建索引。
3. 如果某个数据列包含许多重复内容，建立索引则没有什么实际效果。索引的选择性是指索引列中不同值的数据与表中记录数的比。一个索引的选择性越接近1，这个索引的效率就越高。


## mysql分层和引擎

#### 1. 分层

- 连接层：提供与客户端连接的服务。
- 服务层：提供各种用户使用的接口；提供SQL优化器（mysql query optimizer）；
- 引擎层：提供了数据存储的方式（MyISAM，InnoDB等）；
- 存储层：存储数据。

#### 2. MyISAM与InnoDB

- MyISAM不支持主外键；InnoDB支持主外键。

- MyISAM不支持事务；InnoDB支持事务。

- MyISAM使用表锁，即使操作一条记录也会锁住整个表，不适合高并发操作；InnoDB使用行锁，操作时只锁一行，不对其他行有影响，适合高并发的操作。

- MyISAM只缓存索引，不缓存真实数据；InnoDB不仅缓存索引还缓存真实数据，对内存要求较高，并且内存大小对性能有决定性影响。

- MyISAM表空间小；InnoDB表空间大。

- MyISAM关注性能；InnoDB关注事务。

- MyISAM默认安装；InnoDB默认安装；（安装不代表默认使用）。

#### 3. 数据库引擎

支持哪些引擎：

```mysql
show engines;
```

正在使用的引擎：

```mysql
show variables like '%storage_engine%';
```

修改引擎：

```mysql
create table tb(
    id int(4) auto_increment,
    name varchar(5),
    dept varchar(5),
    primary key(id)
)engine=MyISAM auto_increment=1 default charset=utf8;
```


## sql优化

#### 1. 需要优化的原因

性能低、执行时间太长、等待时间太长、SQl语句欠佳（连接查询）、索引失效、服务器参数设置不对等。

#### 2. sql

编写过程：`select ... from ... join ... on ... where ... group by ... having ... order by ... limit`

解析过程：`from ... on ... join ... where ... group by ... having ... select ... order by limit ...`

#### 3. 优化

主要就是在优化索引。


## 索引

#### 1. 说明

相当于书的目录。

是帮助MySQL高效获取数据的数据结构。

即索引是数据结构（树：B树（默认）、Hash树。。。）

B树：小的节点放左边，大的节点在右边。一般都是指B+树，数据全部存放在叶节点中。B+树查询任意数据的次数都是n次（B+树的高度）。

#### 2. 索引弊端

- 索引本身很大，可以存放在内存/硬盘。
- 索引不是所有情况均适用：少量数据；频繁更新的字段；很少适用的字段等。
- 索引会降低增删改的效率。

#### 3. 索引好处

- 提高查询效率（降低IO使用率）。

- 降低CPU使用率（因为B树索引本身就是一个排好序的结构，因此在对数据排序时可以直接使用）。

#### 4. 分类

1. 主键索引：列值不能重复，且不能为null
2. 单值索引：单列构成的索引。一个表可以有多个单值索引。
3. 唯一索引：构成唯一索引的列数据不能重复。
4. 复合索引：多个列构成的索引。


## 创建索引

#### 1. 方式一

```mysql
create 索引类型 索引名 on 表（字段）
```

示例：

- 单值

```mysql
create index dept_index on tbl(dept);
```

- 唯一

```mysql
create unique index name_index on tbl(name);
```

- 复合

```mysql
create index dept_name_index on tbl(dept, name);
```

#### 2. 方式二

```mysql
alter table 表名 add 索引类型 索引名(字段) 
```

示例：

- 单值

```mysql
alter table tbl add index dept_index(dept);
```

- 唯一

```mysql
alter table tbl add unique index name_index (name);
```

- 多值

```mysql
alter table tbl add index dept_name_index(dept, name);
```

#### 3. 注意

如果一个字段是 primary key，则该字段默认就是主键索引。

## 删除索引

```mysql
drop index 索引名 on 表名;
```

示例：

```mysql
drop index name_index on tbl;
```


## 查看索引

```mysql
show index from 表名;
```

或者

```mysql
show index from 表名\G
```


## SQL性能问题

#### 1. 说明

- 主要是通过分析SQL执行计划，分析性能。

- 注意，SQL查询优化器会干扰我们的优化。


## SQL执行计划

#### 1. 语法

```mysql
explain + SQL语句
```


## 执行计划分析

#### 1. 表的执行顺序，因数量的改变而改变

原因：笛卡尔积。

数据量小的表，优先查询。

#### 1. id

id相同，从上往下顺序执行。

id不同，id值越大越优先查询（原因：在嵌套子查询时，先查内层，再查外层）。

id值有相同，又有不同，id值越大越优先，id值相同从上往下顺序执行。

#### 2. select_type

- `Primary`：包含子查询SQL中的主查询（最外层）。

- `Subquery`：包含子查询SQL中的子查询（非最外层）。

- `Simple`：简单查询（不包含子查询、union）。

- `derived`：衍生查询（使用到了临时表）；如在from子查询中只有一张表；又或者在from子查询中，如果有table1 union table2，则table1就是derived，table2就是union。

- `union`：说明见上例。

- `union result`：告知哪些表之间存在union查询。

#### 3. table

#### 4. type

`System > const > eq_ref > ref > range > index > all`

其中：`system`，`const`只是理想情况，实际能达到 `ref > range`。

优化前提：有索引。

- `system`（忽略）

  只有一条数据的系统表；或衍生表只有一条数据的主查询。

- `const`

  仅仅能查到一条数据的`SQL`，用于`primary key`或者`unique`索引（类型与索引类型有关）。

- `eq_ref`

  唯一性索引；对于每个索引键的查询，返回匹配的唯一行数据（有且只有1个，不能为0）；常见于唯一索引和主键索引。

- `ref`

  非唯一性索引，对于每个索引键的查询，返回匹配的所有行。

- `range`

  检索指定范围的行。

- `index`

  查询全部索引表中的数据。

- `all`

  查询表中的所有数据。

#### 5. Possible_keys

可能用到的索引，是一种预测。

#### 6. key

实际用到的索引。

#### 7. key_len

索引的长度。

用于判断复合索引是否被完全使用。

对于`utf8`，一个字符占3个字节。

如果索引字段可以为`null`，则会使用1个字节用于标识。

如果索引字段为`varchar`，使用2个字节标识可变长度。

#### 8. ref

注意与`type`中的`ref`值区分。

作用：指明当前表所参照的字段。

如：`select ... where a.c = b.x;`

#### 9. rows

被索引优化查询的数据个数（实际通过索引而查询到的数据个数）。

#### 10. extra

- `using filesort`

  性能消耗大；需要“额外”一次排序（查询）。常见于`order by`语句中。

  示例1：

  ```mysql
  explain select * from test where a1='' order by a2;
  ```

  小结：对于单索引，如果排序和查找是同一个字段，则不会出现`using filesort`；如果排序和查找不是同一个字段，则会出现`using filesort`。

  示例2：（注意存在a1, a2, a3, a4四个字段）

  ```mysql
  alter table test add index idx_a1_a2_a3 (a1, a2, a3);
  explain select * from test where a1='' order by a3; # using filesort
  explain select * from test where a2='' order by a3; # using filesort
  explain select * from test where a1='' order by a2;
  ```

  小结：对于复合索引，`where`和`order by`按照复合索引的顺序使用，不要跨列或无序使用。

- `using temporary`

  性能损耗大，用到了临时表。一般出现在`group by`语句中。

  示例：

  ```mysql
  explain select * from test where a1 in (1, 2, 3) group by a1;
  explain select * from test where a1 in (1, 2, 3) group by a2; # using temporary
  ```

- `using index`

  性能提升，常见于索引覆盖。

  只要使用到的列，全部都在索引中，就称为”索引覆盖“。

  不读取原文件，只从索引文件中获取数据（不需要回表查询）。
  
  索引覆盖会对`possible_keys`和`key`造成影响：如果没有`where`，则索引只出现在`key`中；如果有`where`，则索引可能出现在`key`和`possible_keys`中。
  
- `using where`

  回表查询时，会出现`using where`。

- `impossible where`

  `where`子句永远为`false`。
  
- `using join buffer`

  `mysql`引擎使用了连接缓存。


## SQL优化

SQL优化是一种概率层面的优化。

服务层有SQL优化器，可能会影响我们的优化。


## 单表优化

1. 最佳左前缀，保持索引的定义和使用的顺序一致性
2. 索引要逐步优化
3. 将含in的范围查询，放到where的最后，防止失效


## 双表优化

1. 小表驱动大表
2. 一般情况下，左外连接，给右表加索引；右外连接，给左表加索引。


## 三表优化

1. 小表驱动大表。
2. 索引建立在经常查询的字段上。


## 避免索引失效的一些原则

1. 复合索引，不要跨列或无序使用（最佳左前缀）。
2. 复合索引，尽量使用全索引匹配。
3. 不要在索引上进行任何操作（计算，函数，类型转换），否则索引失效。
4. 复合索引不能使用”不等于“，否则自身以及右侧所有全部失效。
5. 复合索引中如果有`>`，自身和右侧全失效。
6. 一般而言，范围查询之后的索引失效。
7. 尽量使用覆盖索引。
8. like尽量以“常量”开头，不要以"%"开头，否则索引失效。
9. 尽量不要使用类型转换。
10. 尽量不要使用`or`，否则索引失效。


## 其他的一些优化方法

#### order by

using filesort有两种算法：单路排序、双路排序。

双路排序扫描两次磁盘：从磁盘读取排序字段；读取其他字段。

单路排序扫描一次磁盘：读取全部字段（不一定真的是单次，有可能是多次IO）。

Mysql4.1之前默认使用双路排序；mysql4.1之前默认使用单路排序。

设置buffer的容量大小：`set max_length_for_sort_data = 1024`，单位byte。

如果`max_length_for_sort_data`值太低，mysql会自动从 单路切换成双路。

避免使用`select * ...`

复合索引不要跨列使用。

保证排序字段的排序一致性。


## 慢查询

mysql提供的一种日志记录，用于记录mysql响应超过阀值的语句，默认10秒。

默认是关闭的；建议调优时打开，最终部署时关闭。

检查是否打开了慢查询日志：

```mysql
show variables like '%slow_query_log%';
```

临时开启：

```mysql
set global slow_query_log = 1;
```

mysql服务器重启后会失效。

永久开启：

```bash
#/etc/my.cnf追加
[mysqld]
slow_query_log=1
slow_query_log_file=/var/lib/mysql/local-slow.log
```

慢查询阀值：

```mysql
show variables like "%log_query_time%";
```

设置慢查询阀值：

```mysql
set global lomg_query_time=5;
```

重新登陆后生效。

永久设置阀值：

```bash
#在/etc/my.cnf追加
[mysqld]
long_query_time=3
```

查询超过阀值的sql：

```mysql
show gloval status like '%slow_queries%';
```


## mysqldumpslow

用于查看慢SQL。


## 海量数据分析

#### 1. profiles

查看开启状态

```mysql
show variables like '%profiling%';
```

打开

```mysql
set profiling=on;
```

查看profiling打开之后，全部sql查询语句所花时间，不够精确

```mysql
show profiles;
```

#### 2. 精确分析

```mysql
show profile all for query 上一步查询的query_id;
```

或者

```mysql
show profile cpu, block io for query 上一步查询的query_id;
```

#### 3. 全局查询日志

记录开启之后的全部sql语句，仅仅在调优、开发过程中打开。

```mysql
show variables like '%general_log%'; # 查看状态

#记录在表中
set global general_log=1; # 开启
set global log_output='table'; # sql记录在表中

#或者记录在文件中
set global general_log=1; # 开启
set global log_output='file'; # 或者sql记录在文件中
set global general_log_file='/tmp/general.log'; # 或者sql记录在文件中
```


## 锁机制

解决因资源共享，造成的并发问题。

#### 分类

按照操作类型：

- 读锁（共享锁）：对同一个数据，多个读操作可以同时进行，互不干扰。

- 写锁（互斥锁）：如果当前写操作没有完毕，则无法进行其他的读、写。

按照操作范围：

- 表锁：一次性对一张表整体加锁。MyISAM使用表锁。开销小，加锁快，无死锁。但锁的范围大，容易发生锁冲突，并发度低。

- 行锁：一次性对一条数据加锁。InnoDB使用行锁。开销大，加锁慢，容易死锁。所得范围较小，不容易发生锁冲突，并发度高，很小概率发生高并发问题（脏读、幻读、不可重复读）。

- 页锁：


## 表锁

#### 给表加锁

```mysql
lock table 表1 read/write, 表2 read/write, ...;
```

#### 查看加锁的表

```mysql
show open tables;
```

#### 释放锁

```mysql
unlock tables;
```

#### 加读锁的机制

如果一个会话对A表加了read锁，则该会话可以对A表进行读操作、不能进行写操作；不能对其它表进行读写操作。其它会话可以对A表进行读操作、进行写操作得等待锁释放；对其他表可以进行读写操作。

#### 加写锁的机制

某个会话对表加写锁，之后可以对这张表进行任何操作，但是不能操作其它表。

其他会话对加写锁的表，进行增删改查时得等待锁释放。

#### MySQL表级锁的锁模式

1. MyISAM在执行查询语句前，会自动给涉及到的所有表加读锁；在执行更新操作前，会自动给涉及的表加写锁。
2. 对MyISAM表的读操作，不会阻塞其他进程（会话）对同一表的读请求，会阻塞对同一表的写请求。
3. 对MyISAm表的写操作，会阻塞其他进程（会话）对同一表的读和写操作，只有当锁释放时，才会进行其它进程的读写操作。

#### 分析表锁定

1. 查看表锁定状态

   ```mysql
   show open tables; # 1代表加了锁
   ```

2. 分析表锁定的严重程度

   ```mysql
   show status like 'table%';
   ```

   - `table_lock_immediate`：可能获取到的锁数。
   - `tale_locks_waited`：需要等待的表锁数，该值越大，锁竞争越大。

   当`table_locks_immediate / table_locks_waited` > 5000，建议采用InnoDB引擎。


## 行锁

InnoDB默认采用行锁。

研究前提：关闭了自动commit

关闭语句：

1. `set autocommit=0`
2. `start transection`
3. `begin`

#### 总结：

1. 如果”会话1“对“某条数据a”进行了DML操作，则其他会话需要等待“会话1”对“数据a”提交事务。
2. 行锁通过事务解锁。
3. 不同会话对不同行数据进行操作，互不影响。
4. 查询数据不会加锁。
5. 可以通过`for update`对`query`语句进行加锁。

#### 注意事项：

1. 如果没有索引，行锁会转换为表锁，需要通过`commit`进行解锁。

#### 行锁的一种特殊情况：

间隙锁：值在范围内，但却不存在。

如：`update ... where id > 1 and id < 9;`，如果`id=7`的数据不存在，则该行数据存在间隙锁。

如果有`where`，则实际加锁的范围是`where`后面的范围（不是实际的值）。

#### 缺点：

比表锁性能损耗大。

#### 优点：

并发能力强，效率高。

#### 行锁分析

1. 查看行锁状态

   ```mysql
   show status like '%innodb_row_lock%'
   ```

   - `Innodb_row_locl_current_waits`：当前正在等待锁的数量。
   - `Innodb_row_lock_time`：等待总时长，从系统启动到现在，一共等待的时间。
   - `Innodb_row_lock_time_avg`：平均等待时长，从系统启动到现在的平均等待时间。
   - `Innodb_row_lock_time_max`：最大等待时长，从系统启动到现在的最大等待时间。
   - `Innodb_row_lock_waits`：从系统启动到现在，等待的次数。



## 主从复制

#### 优点

1. 负载均衡
2. 失败迁移

#### 原理

1. master将改变的数据记录在本地的二进制日志中（binary log）；该过程称为“二进制日志事件”。
2. slave将master的binary log拷贝到relay log（中继日志文件）中。
3. slave将数据从relay log读取到自己的数据库中，称为中继日志事件。

#### 特点

1. 异步
2. 串行话
3. 有延迟

