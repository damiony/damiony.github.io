# 【MySQL】DDL数据定义


## DDL解释

`DDL`: 数据定义语言`(Data Define Language)`，用于对数据库和表进行管理和操作。


## 库管理

### 1. 查看所有的数据库列表

```sql
SHOW DATABASES;
```

### 2. 创建数据库

详细语法为：

```sql
CREATE {DATABASE | SCHEMA} [IF NOT EXISTS] 库名 [[DEFAULT] CHARACTER SET [=] 字符集编码];
```

常见的字符集编码方式有`utf8`和`utf8mb4`。

`utf8`是`utf8mb3`的别名，使用该编码时，一个字符最多占3个字节。如果使用`utf8mb4`，一个字符最多占4个字节。推荐使用`utf8mb4`，方便存储`emoji`。

```sql
CREATE DATABASE 库名;
```

```sql
CREATE DATABASE IF NOT EXISTS 库名; # 如果不存在则创建
```

```sql
CREATE SCHEMA 库名 CHARACTER SET utf8mb4;
```

### 3. 查看建库语句

```sql
SHOW CREATE {DATABASE | SCHEMA} 库名;
```

### 4. 修改数据库

```sql
ALTER {DATABASE | SCHEMA} 库名 [[DEFAULT] CHARACTER SET [=] 字符编码]
```

### 5. 删除数据库

```sql
DROP {DATABASE | SCHEMA} [IF EXISTS] 库名;
```

如：

```sql
DROP DATABASE 库名;
```

```sql
DROP DATABASE IF EXISTS 库名; # 如果存在则删除
```

### 6. 打开数据库

```sql
USE 库名;
```

### 7. 查看所处数据库

```sql
SELECT DATABASE();
```


## 表管理

### 1. 创建表

语法：

```sql
CREATE TABLE [IF NOT EXISTS] 表名(
  字段名 字段类型 [字段约束],
  ......
  字段名 字段类型 [字段约束],
  字段名 字段类型 [字段约束]
)[ENGINE=引擎 [DEFAULT] CHARSET=字符编码];
```

示例：

```sql
CREATE TABLE test(
  id int,
  a int,
  b varchar(20)
)ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

### 2. 查看数据库的所有数据表

```sql
SHOW TABLES [FROM 数据库名];
```

除了数据表，该命令也会显示视图。

### 3. 查看数据表结构

```sql
DESC 表名;
```

或者：

```sql
SHOW COLUMNS FROM 表名;
```

或者：

```sql
SHOW CREATE TABLE 表名;
```

### 4. 修改表

修改表的属性：

```sql
ALTER TABLE 表名 属性名=属性值;
```

修改表名：

```sql
ALTER TABLE 原表名 RENAME TO 新表名;
```

修改表字段：

```sql
ALTER TABLE 表名 ADD|MODIFY|CHANGE|DROP [COLUMN] 字段名 字段类型 字段约束;
```

示例：

```sql
ALTER TABLE test engine=InnoDB;
```

```sql
ALTER TABLE stuinfo RENAME TO students;
```

```sql
ALTER TABLE students ADD COLUMN borndate TIMESTAMP NOT NULL;
```

```sql
ALTER TABLE students CHANGE COLUMN borndate birthday DATETIME NULL;
```

```sql
ALTER TABLE students MODIFY COLUMN birthday TIMESTAMP;
```

```sql
ALTER TABLE students DROP COLUMN birthday;
```

### 5. 删除表

```sql
DROP TABLE IF EXISTS tablename;
```

### 6. 复制表

```sql
CREATE TABLE tablename1 LIKE tablename2; # 仅复制表结构
```

```sql
CREATE TABLE tablename1 SELECT * FROM tablename2; # 复制结构和数据
```

示例：

```sql
CREATE TABLE emp
SELECT name
FROM employees
WHERE 1=2;
```

## 建表规范

1. 表名不能小写，不能太长，不能使用关键字，要和业务有关。

2. 必须设置存储引擎和字符集。

3. 数据类型在满足需求的基础上，要足够简短。

4. 必须有主键。

5. 每个列尽量设置为`not null`，并设定默认值。

6. 每个列需要加注释。

7. 列名规范同表名。


## 常用字段类型

### 1. 整数

- `tinyint`：1字节，有符号的范围是-128～127，无符号的范围是0~255。

- `smallint`：2字节，有符号的范围是-32768~32767，无符号的范围是0~65535。

- `mediumint`：3字节，有符号的范围是-8388608~8388607，无符号的范围是0~16777215。

- `int`：4字节，有符号的范围是-2147483648~2147483647，无符号的范围是0~4294967295。

- `bigint`：8字节。

### 2. 浮点数

- `float`: 4字节单精度浮点数

- `double`: 8字节双精度浮点数

例如：double(5, 2)表示最多5位，其中有且仅有2位小数，即最大值为999.99。

### 3. decimal

浮点型，不会出现精度问题。

### 4. char

固定长度字符串类型；内容范围是0~255字符；

`char(n)`：n参数默认为1，不管实际存储，开辟空间都是`n`个字节，效率高。`mysql5`之后，`n`指的是字符个数。

### 5. varchar

可变长度字符串类型；内容范围为0~65535字节；

`varchar(n)`最大字节个数`n`必选，根据实际存储空间决定开辟空间，效率低。`varchar`会消耗额外1到2个字节，存储字符长度。`mysql5`之后，`n`指的是字符。

### 6. text

字符串类型，通常存储较长文本。

### 7. blob

字节类型；常用于`jpg, mp3, avi`。

### 8. date

日期类型；格式为`yyyy-MM-dd`。

### 9. time

时间类型；格式为`hh:mm:ss`。

### 10. timestamp

用于保存日期和时间，保存格式为时间戳，共占用4个字节。

`timestamp`值的范围是`1970-01-01 00:00:01 UTC`到`2038-01-19 03:14:07 UTC`。

用命令行工具更新该值时，数据格式为“年月日时分秒”，“时分秒可以省略”。

### 10. datetime

用于保存日期和时间，格式为时间戳，占用5个字节。

`datetime`精度可以到微秒，如果需要保存毫秒和微秒，则需要消耗额外空间，最大可扩展到8字节。

注意，上述规则不适用于`MySQL5.6.4`之前版本。在该版本之前，`datetime`固定占用8字节。

取值范围是`0000-01-01 00:00:00`到`9999-12-31 23:59:59`。显示具体值时，只会精确到秒。


## 常见字段约束

### 1. 说明

用于限制表中字段的数据。

### 2. 列表

- `NOT NULL` 

  非空。用于限制该字段为必填项。

- `DEFAULT` 

  默认值。如果该字段没有插入值，则显示默认值。

- `PRIMARY KEY` 

  主键。被设为主键的列值不能重复，并且不能为空。

  ```sql
  ALTER TABLE tbl_name ADD PRIMARY KEY (column_list);
  ```

- `UNIQUE` 

  唯一键。限制字段不能重复，但是可以为空。

- `CHECK` 

  检查。限制该字段值必须满足指定条件，`MySQL`不支持。

- `FOREIGN KEY` 

  外键，用于限制两个表的关系。
  
  外键列的值必须来自主表的关联列。主表关联列和从表关联列必须类型一致，意思一致，但是名称无要求。主表的关联列要求必须是主键。

  定义外键：

  ```sql
  ALTER TABLE teblename ADD CONSTRAINT 外键名 FOREIGN KEY(列名) REFERENCES 表名（关联列）;
  ```

  或者：

  ```sql
  CREATE TABLE teblename (
  	...
  	CONSTRAINT 外键名 FOREIGN KEY(列名) REFERENCES 表名(关联列)
  );
  ```

  删除外键：

  ```sql
  ALTER TABLE tablename DROP FOREIGN KEY 键名
  ```

- `AUTO_INCREMENT`

  自增长列必须设置在一个键上，比如主键或唯一键。

  自增长列要求数据类型为数值型。

  一个表至多有一个自增长列。

